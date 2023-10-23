---
layout: post
title: "Building Inter-procedural Source Sink Analysis from Scratch - Part 2"
categories: [static-analysis, sast]
tags: [programming, productivity]
description: "Building Inter-procedural Source Sink Analysis from Scratch - Part 2"
comments: true
---

This is the second part of the blog post series on building inter-procedural source sink analysis from scratch. In the first part, we have built the [intra-procedural source sink analysis](https://shivasurya.me/static-analysis/sast/2023/08/27/building-simple-source-sink-analysis-from-scratch-part-1.html). In this blog post, we will be building the inter-procedural source sink analysis.

### Plan

We'll be parsing whole java project source code and generate AST using JavaParser. While traversing the AST, we will be collecting the method declaration and method invocation. We will be using graph theory algorithm to find the path from source to sink. The source is the method declaration and the sink is the method invocation. The method declaration is the node and the method invocation is the edge. While classes may contain duplicate method names with different signatures, we will be using the fully qualified method name to uniquely identify the method. The fully qualified method name is the class name + method name + method arguments. The method arguments are used to differentiate the method overloading. The method declaration is the key and method invocation is the value in the hashmap. The hashmap is used to build the graph and find the path from source to sink.


### Parsing Java Project

Let's parse the source code and generate AST using JavaParser. We will be using [Signal's Android opensource codebase](https://sourcegraph.com/github.com/signalapp/Signal-Android) as an example. The JavaParser.parse method takes in the source code as string and returns the ParseResult object. The ParseResult object contains the source code tree aka compilation unit. Iterate through repo directory and filter for `java` files and parse the source code.

```java
private void resolveClassAndExtractMethod(String projectRoot, String className) throws IOException {
    File root = new File(projectRoot);
    File[] list = root.listFiles();
    if (list == null) return;
    for (File f : list) {
        if (f.isDirectory() && !f.getName().equals("build") && !f.getName().equals(".git")) {
            resolveClassAndExtractMethod(f.getAbsolutePath(), className);
        }
        else {
            if (f.getName().endsWith(".java")) {
                //System.out.println(f.getAbsoluteFile());
                Path path = f.toPath();
                String sourceCode = new String(Files.readAllBytes(path));
                JavaParser parser = new JavaParser();
                ParseResult<CompilationUnit> cu = parser.parse(sourceCode);

                // check parsed file contains the class we are looking for
                if (cu.isSuccessful() && cu.getResult().isPresent()) {
                    CompilationUnit compiledSource = cu.getResult().get();
                    if (compiledSource.findFirst(ClassOrInterfaceDeclaration.class, c -> c.getNameAsString().equals(className)).isPresent()) {
                        // Create a visitor to extract method invocations
                        if(cu.getResult().isPresent()) {
                            CompilationUnit compilationUnit = cu.getResult().get();
                            findMethodCalls(compilationUnit);
                        }
                    }
                }
            } else if (f.getName().endsWith(".kt")) {
                // parse Kotlin files
            }
        }
    }
}
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

### Building Call Graph

In previous post, we just utilized methods from single class. But, in real world, we need to utilize methods from other classes. We need to build the call graph to find the path from source to sink. The method declaration is the key and method invocation is the value in the hashmap. The hashmap is used to build the graph and find the path from source to sink. The method declaration is the node and method invocation is the edge. While classes may contain duplicate method names with different signatures, we will be using the fully qualified method name to uniquely identify the method. The fully qualified method name is the class name + method name. **I know this isn't still perfect, what if there are classes in different package with similar class name, methods with same name, etc. I will be covering those edge cases in the next blog post.**

```java
private void extractAndPrintMethodCalls(CompilationUnit cu) {
    cu.findAll(MethodDeclaration.class).forEach(method -> {
        System.out.println("---------------------------");
        System.out.println("Method: " + method.getNameAsString());

        String DeclaratorClass = "UnknownClass";
        if(method.findAncestor(ClassOrInterfaceDeclaration.class).isPresent())
        {
            DeclaratorClass = method.findAncestor(ClassOrInterfaceDeclaration.class).get().getNameAsString();
        }
        System.out.println("Class: " + DeclaratorClass);

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

            for (Expression arg : call.getArguments()) {
                System.out.println("    Argument: " + arg.toString());
            }

            System.out.println(DeclaratorClass + "::" + method.getNameAsString() + " -> " + className + "::" + call.getNameAsString());
            analysis.addMethodInvocation(DeclaratorClass + "::" + method.getNameAsString(), className + "::" + call.getNameAsString());
        }

    });
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

A quick example of Inter-procedural source sink analysis. The source method is `MainActivity::onCreate` and the sink method is `Util::execShellCommand`. The `execShellCommand` is reachable from `MainActivity::onCreate` method. The `execShellCommand` method is reachable from `MainActivity::onCreate` method because `MainActivity::onCreate` method calls `MainActivity::doSomething` method and `MainActivity::doSomething` method calls `Util::execShellCommand` method.

```java
SourceSinkAnalysis analysis = new SourceSinkAnalysis();

// Assuming you have information about method invocations & declarations from JavaParser
analysis.addMethodInvocation("MainActivity::onCreate", "Util::writeToFile");
analysis.addMethodInvocation("MainActivity::onCreate", "MainActivity::doSomething");
analysis.addMethodInvocation("MainActivity::doSomething", "Util::execShellCommand");

String sourceMethod = "MainActivity::onCreate";
String sinkMethod = "Util::execShellCommand";

boolean isReachable = analysis.isReachable(sourceMethod, sinkMethod);
System.out.println("Is " + sinkMethod + " reachable from " + sourceMethod + "? " + isReachable);
```

### Closing Note:

There are lot of edge cases to handle like `this` and `super` keyword, static method call, etc. I will be covering those edge cases in the next blog post. I hope this post is helpful for developers & engineers 💻. For bugs or hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.
