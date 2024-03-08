---
layout: post
title: "Defining Boundaries & Sinks for Inter-procedural Source Sink Analysis - Part 3"
categories: [static-analysis, sast]
tags: [programming, productivity]
description: "Defining Boundaries & Sinks for Inter-procedural Source Sink Analysis - Part 3"
comments: true
---

This is the third part of the blog post series on building inter-procedural source sink analysis from scratch. In the first part, we have built the [intra-procedural source sink analysis](https://shivasurya.me/static-analysis/sast/2023/08/27/building-simple-source-sink-analysis-from-scratch-part-1.html). In this blog post, I'll discuss about defining boundaries, configs and sinks for inter-procedural analysis. ✨ This idea of defining boundaries and sinks is inspired from the [CodeQL](https://codeql.github.com/) library and while discussing with my colleague at [SWAG lab @ uwaterloo](https://www.swag.uwaterloo.ca/).

### Plan

While tools like CodeQL has well-defined support for libraries and framework such as [Android CodeQL](https://codeql.github.com/codeql-standard-libraries/java/semmle/code/java/frameworks/android/Android.qll/module.Android.html) these libraries has predefined boundaries and sinks. But, start from scratch, we need to define our own boundaries and sinks. The boundaries are the entry points and sinks are the exit points.


### Defining Boundaries via Config

A good example of boundary is the entry point of the application. I would prefer basic Android application as an example. Source key can contain the source code and the routes key can contain the routes of the application. Dependencies key can contain the dependencies of the application and boundaries key can contain the entry points of the application.

```yaml
application:
  - source:
    - src/
    - lib/
  - routes:
    - src/AndroidManifest.xml
  - dependencies:
    - src/build.gradle
  - system_boundaries:
    - Activity::onCreate
    - Activity::onStart
    - Activity::onResume
    - Activity::onPause
    - Activity::onStop
    - Activity::onDestroy
    - Activity::onRestart
    - Activity::onBackPressed
    - Activity::onRequestPermissionsResult
    - Activity::onActivityResult
    - Activity::onNewIntent
    - Activity::onCreateContextMenu
    - Activity::onContextItemSelected
    - Activity::onContextMenuClosed
    - Activity::onCreateOptionsMenu
    - Activity::onOptionsItemSelected
    - Activity::onPrepareOptionsMenu
    - Activity::onCreateDialog
    - Activity::onCreateView
    - Service::onCreate
    - Service::onStartCommand
    - Service::onBind
    - Service::onUnbind
    - Service::onRebind
    - Service::onDestroy
    - BroadcastReceiver::onReceive
    - ContentProvider::onCreate
    - ContentProvider::query
    - ContentProvider::insert
    - ContentProvider::update
    - ContentProvider::delete
    - ContentProvider::getType
    - Application::onCreate
    - Application::onTerminate
    - Application::onLowMemory
    - Application::onTrimMemory
    - Application::onConfigurationChanged
  - system_source:
    - Intent::getExtras
    - Intent::getData
    - Intent::getAction
    - Intent::getScheme
    - Intent::getType
    - Intent::getCategories
    - Intent::getComponent
    - Intent::getPackage
    - Intent::getSelector
    - Intent::getClipData
    - Intent::getFlags
    ...

```

By defining the system source and boundaries, we can easily identify the entry points and exit points of the application. Because of the nature of the Android application, the entry points are the lifecycle methods of the activity, service, broadcast receiver, content provider and application. The system source are the methods that are used to get the data from the system. For example, `Intent::getExtras` is used to get the extras from the intent. The system source and boundaries are the entry points and exit points of the application. Mostly application code will be in the middle of the system source and boundaries. This gives clear separation of the application code and the system code.

By classifying the system source and boundaries for popular frameworks and libraries, this could be more helpful for security engineers to pick the framework and start writing the queries for security analysis. The abstraction layer can be used to define the system source and boundaries while the implementation layer can be used to define the application level source and boundaries.

For example similar to CodeQL Android library, `AndroidIntentRedirectionQuery` has already defined the system source and boundaries for Android application. reference: [Github QL](https://github.com/github/codeql/blob/main/java/ql/src/Security/CWE/CWE-940/AndroidIntentRedirection.ql)

```ql
import java
import semmle.code.java.security.AndroidIntentRedirectionQuery
import IntentRedirectionFlow::PathGraph

from IntentRedirectionFlow::PathNode source, IntentRedirectionFlow::PathNode sink
where IntentRedirectionFlow::flowPath(source, sink)
select sink.getNode(), source, sink,
  "Arbitrary Android activities or services can be started from a $@.", source.getNode(),
  "user-provided value"
```

Thus defining the boundaries and system source is a long way to go. But, it is a good start to define the boundaries and system source for the popular frameworks and libraries. CodeQL is a good example of defining the boundaries and system source for the popular frameworks and libraries.

### Closing Note:

For bugs or hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.
