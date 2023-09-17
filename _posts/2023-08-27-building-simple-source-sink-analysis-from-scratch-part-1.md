---
layout: post
title: "Building A Simple Source-Sink Analysis in Java from Scratch - Part 1"
categories: [static-analysis, sast]
tags: [programming, productivity]
description: Building a simple source sink analysis in Java from scratch.
comments: true
---


### Overview of Source Sink Analysis

Source Sink Analysis is a type of basic static analysis that detects the flow of information from a source to a sink. A source is a place where the information is coming from and a sink is a place where the information is going to. For example, a source can be a user input and a sink can be a database query. If the user input is not sanitized, it can lead to SQL Injection. Apart from source sink techniques, there are other techniques like taint analysis, control flow graph, data flow analysis, etc. which are used to detect vulnerabilities in the code. In this blog post, we will be building a simple source sink analysis in Java from scratch.

![data flow analysis](/assets/media/attachments/dataflowpath.webp)

### Getting Started

To understand the source code, one could visualize it as Tree datastructure where each node is a statement and the edges are the control flow. Abstract Syntax Tree is perfect candidate to represent the source code as tree datastructure. We will be using [JavaParser](https://javaparser.org/) to parse the source code and build the AST. JavaParser is a Java library for parsing Java code. It provides a simple and easy to use API to parse, modify, and generate Java code. 

![AST Java source example](/assets/media/attachments/ast-example.png)

Add javaparser-core dependency to your pom.xml file.

```xml
<dependency>
    <groupId>com.github.javaparser</groupId>
    <artifactId>javaparser-core</artifactId>
    <version>3.25.4</version>
</dependency>
```

### Parsing Source Code

Let's parse the source code and generate AST using JavaParser. We will be using Signal's Android opensource codebase [MainActivity.java](https://sourcegraph.com/github.com/signalapp/Signal-Android/-/blob/app/src/main/java/org/thoughtcrime/securesms/MainActivity.java) as an example. The JavaParser.parse method takes in the source code as string and returns the ParseResult object. The ParseResult object contains the source code tree aka compilation unit.

```java
 File f = new File("src/main/java/com/MainActivity.java");
 Path path = f.toPath();
 String sourceCode = new String(Files.readAllBytes(path));
 JavaParser parser = new JavaParser();
 ParseResult<CompilationUnit> cu = parser.parse(sourceCode);
```

### Traversing AST for Method Declaration

Now that we have the compilation unit, we can traverse the AST to get the method declaration. We will be using the findAll method to get all the method declarations. The findAll method takes in the class type and returns the list of nodes. In this case, we are looking for the method declaration, so we will be passing MethodDeclaration.class as an argument. The findAll method returns the list of nodes, so we will be iterating over the list of nodes to get the method declaration.

```java
compilationUnit.findAll(MethodDeclaration.class).forEach(method -> {
    String declaratorClass = "UnknownClass";
    if (method.findAncestor(ClassOrInterfaceDeclaration.class).isPresent()) {
        declaratorClass = method.findAncestor(ClassOrInterfaceDeclaration.class).get().getNameAsString();
    }
    // sometimes there might be static inner class
    System.out.println("Class: " + declaratorClass);
    System.out.println("Declared Method: " + method.getNameAsString());
    
    // get declared method arguments type
    ArrayList<String> declaredMethodArguments = new ArrayList<>();
    for (Parameter parameter : method.getParameters()) {
        declaredMethodArguments.add(parameter.getType().toString());
    }
});
```

### Traversing AST for Method Invocation

Now that we have the method declaration, we can traverse the AST to get the method invocation. We will be using the findAll method to get all the method invocations. The findAll method takes in the class type and returns the list of nodes. In this case, we are looking for the method invocation, so we will be passing MethodCallExpr.class as an argument. The findAll method returns the list of nodes, so we will be iterating over the list of nodes to get the method invocation. The `method.findAll*` is object of type `MethodDeclaration` and `call.findAll*` is object of type `MethodCallExpr`. That way we could scope the method call to the method declaration. There are lot of edge cases to handle like `this` and `super` keyword, static method call, etc. I will be covering those edge cases in the next blog post.

```java
 List<Method> calledMethods = new ArrayList<>();

List<MethodCallExpr> methodCalls = method.findAll(MethodCallExpr.class);
for (MethodCallExpr call : methodCalls) {
    String objectName = call.getScope().map(s -> s.toString()).orElse("UnknownClass");
    String className = "UnknownClass";
    List<FieldDeclaration> fieldDeclarations = cu.findAll(FieldDeclaration.class);

    for (FieldDeclaration fieldDeclaration : fieldDeclarations) {
        if (fieldDeclaration.getVariables().get(0).getNameAsString().equals(objectName)) {
            className = fieldDeclaration.getElementType().toString();
            break; // Assuming you only need to find one matching field
        }
    }

    // Handle super and this
    if (objectName.equals("this")) {
        className = method.findAncestor(ClassOrInterfaceDeclaration.class).get().getNameAsString();
    }

    System.out.println("  Class: " + className);
    System.out.println("  Method Call: " + call.getNameAsString());

    List<String> methodCallArguments = new ArrayList<>();
    for (Expression arg : call.getArguments()) {
        System.out.println("    Argument: " + arg.toString());
        methodCallArguments.add(arg.toString());
    }

    Method calledMethod = new Method(className, call.getNameAsString(), methodCallArguments);
    calledMethods.add(calledMethod);

    System.out.println(declaratorClass + "::" + method.getNameAsString() + " -> " + className + "::" + call.getNameAsString());
}
```

### Leveraging Graph Theory Algorithm

Now that we have the method declaration and method invocation, we can leverage graph theory algorithm to find the path from source to sink. We will be using BFS algorithm to find the path from source to sink. The BFS algorithm takes in the source method and sink method as an argument and returns true if the sink is reachable from the source. The BFS algorithm uses the method declaration and method invocation to build the graph. The method declaration is the node and method invocation is the edge. The method declaration is the key and method invocation is the value in the hashmap. The BFS algorithm uses the hashmap to build the graph and find the path from source to sink.

```java
 public boolean isReachable(String sourceMethod, String sinkMethod) {
        Set<String> visited = new HashSet<>();
        Queue<String> queue = new LinkedList<>();
        queue.offer(sourceMethod);
  
        while (!queue.isEmpty()) {
            String currentMethod = queue.poll();
            visited.add(currentMethod);

            System.out.println("currentMethod: " + currentMethod);
           // System.out.println("sinkMethod: " + sinkMethod);
            if (currentMethod.equals(sinkMethod)) {
                return true; // Sink is reachable from the source
            }

            List<String> callees = methodInvocations.getOrDefault(currentMethod, Collections.emptyList());
            for (String callee : callees) {
                if (!visited.contains(callee)) {
                    queue.offer(callee);
                }
            }
        }

        return false; // Sink is not reachable from the source
}

```

A quick example of Intra-procedural source sink analysis. The source method is `onCreate` and the sink method is `execShellCommand`. The `execShellCommand` is reachable from `onCreate` method. This is variant of classic [Yelp Interview question](https://leetcode.com/problems/reconstruct-itinerary/) known as Reconstruct Itinerary problem. The itinerary problem is to find the path from source to sink. The source is the departure airport and the sink is the arrival airport. The BFS/DFS algorithm is used to find the path from source to sink.

![Intra-procedural source sink analysis](/assets/media/graph.png)

```java
SourceSinkAnalysis analysis = new SourceSinkAnalysis();

// Assuming you have information about method invocations & declarations from JavaParser
analysis.addMethodInvocation("onCreate", "writeToFile");
analysis.addMethodInvocation("onCreate", "doSomething");
analysis.addMethodInvocation("doSomething", "execShellCommand");

String sourceMethod = "onCreate";
String sinkMethod = "execShellCommand";

boolean isReachable = analysis.isReachable(sourceMethod, sinkMethod);
System.out.println("Is " + sinkMethod + " reachable from " + sourceMethod + "? " + isReachable);
```

### Scaling from inter-procedural to intra-procedural

The above example is an intra-procedural analysis where we could detect the source and sink reachability within single Java file. The inter-procedural analysis is where we could detect the source and sink reachability across multiple Java files. The inter-procedural analysis is more complex than intra-procedural analysis. The inter-procedural analysis requires the knowledge of call graph. The call graph is a directed graph that represents calling relationships between methods in a program. I'll be covering the inter-procedural analysis in the next blog post.

### Closing Note:

I'll be talking about how I built the inter-procedural analysis in the next blog post. I hope this post is helpful for developers & engineers 💻. For bugs or hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.
