---
layout: post
title: Detecting Android WebView Vulnerable Configurations with Semgrep Rules - Part 1
categories: [security, android, android-security]
tags: [security, android]
description: Android Webview has multiple security configuration that may lead to security vulnerabilities. <br /> We'll take a deep dive into those webview configs, breakdown vulnerable configs and leverage semgrep to identify those pattern.
comments: true
---

Android WebView widget provides APIs that help developers seamlessly integrate webpage contents within Android application. Advancement in Webview & Chrome Custom Tabs lead to [exponential growth in webview based mobile development](https://tomtunguz.com/mobile-only-saas/) platforms such as `Ionic framework`, `JQuery Mobile`, `Adobe Phonegap` later open-sourced as `Cordova Project`, `React Native`. However the race to capture the mobile development market, immature WebView APIs and lack of security guidance lead to multiple vulnerabilities and exploits. In today's blog post, we'll deep dive into multiple WebView vulnerability configurations and leverage semgrep to detect those configuration real time.

As a sidenote, Static Analysis Tools such as semgrep, CodeQL can only detect patterns and alert security engineers, researchers. However, You may want a full-fledged tool such as [`DEVAA`](https://uwspace.uwaterloo.ca/handle/10012/17034?show=full) to perform Dynamic Analysis to verify the exploit.

### Quick Overview

Outside of Browser ecosystem, implementing web related components particularly in security context is more challenging for a developer getting all things right. For instance, WebView exposes powerful APIs to interact with web components and any misconfiguration would lead to security issues. In Browser & Web ecosystem, there are RFCs, testcases, security researchers and fuzzing techniques to monitor those regression or behavioural changes.

### Vulnerable Configuration

As discussed before, here are few powerful WebView APIs `setAllowFileAccess`, `setAllowFileAccessFromFileURLs`, `setAllowFileAccessFromFileURLs`, `setAllowUniversalAccessFromFileURLs`, `setAllowContentAccess`, `onReceivedSslError`, `setWebContentsDebuggingEnabled`.

#### setAllowFileAccess

WebView allows `main window` access to File Protocol i.e `file://` and render based on the content type. Using `setAllowFileAccess` config doesn't cause any vulnerability but combination of Cross Site Scripting with File Access enabled config would leak any sandbox files within the Android application.

```yaml
rules:
  - id: CWE-259
    patterns:
      - pattern-either:
        - pattern: $X.setAllowFileAccess(...)
    message: Semgrep found a match
    languages: [java]
    severity: WARNING
  ```

#### setAllowFileAccessFromFileURLs

WebView allows `main window` access to File Protocol i.e `file://` via `Ajax Calls using JavaScript`. Using `setAllowFileAccessFromFileURLs` config doesn't cause any vulnerability but combination of Cross Site Scripting with File Access enabled config would leak any sandbox files within the Android application.

```yaml
rules:
  - id: CWE-259
    patterns:
      - pattern-either:
        - pattern: $X.setAllowFileAccessFromFileURLs(...)
    message: Semgrep found a match
    languages: [java]
    severity: WARNING
  ```

#### setAllowUniversalAccessFromFileURLs

WebView allows `main window` access to Any Protocol i.e `file://`, `res://`, `content://` via `Ajax Calls using JavaScript` & via subframe as Iframe. Using `setAllowFileAccessFromFileURLs` config doesn't cause any vulnerability but combination of Cross Site Scripting with Universal Access enabled config would leak any sandbox files within the Android application.

```yaml
rules:
  - id: CWE-259
    patterns:
      - pattern-either:
        - pattern: $X.setAllowUniversalAccessFromFileURLs(...)
    message: Semgrep found a match
    languages: [java]
    severity: WARNING
  ```

#### setAllowContentAccess

WebView allows `main window` access to File Protocol i.e `content://` and render based on the content type. Using `setAllowContentAccess` config doesn't create any cause any vulnerability but combination of Cross Site Scripting with File Access enabled config would leak any sandbox files within the Android application.

```yaml
rules:
  - id: CWE-259
    patterns:
      - pattern-either:
        - pattern: $X.setAllowContentAccess(...)
    message: Semgrep found a match
    languages: [java]
    severity: WARNING
  ```

#### onReceivedSslError

WebView programmatically allows to skip any SSL/TLS Errors which may be quite dangerous if you're doing it without knowing the risk. The below rule might be helpful in detecting `onReceivedSslError` method implementation changes.

```yaml
rules:
  - id: CWE-200
    patterns:
      - pattern-either:
        - pattern: |
            class $X extends WebViewClient {
              ...
              public void onReceivedSslError(...) { 
                handler.proceed(); 
              }
            }
    message: Semgrep found a match
    languages: [java]
    severity: WARNING
  ```

#### setWebContentsDebuggingEnabled

Webview allows to connect remotely and debug via [Chrome Remote Console](https://developer.chrome.com/docs/devtools/remote-debugging/) which helps developers to interact with webview directly from console and leverage dev console. However, this doesn't cause any serious threat since it requires physical device attached to ADB Interface and authorized by entering the Security Key. However, If you were able to get hold of physical device, One might be able to execute javascript code in the context of the Android app.

```yaml
rules:
  - id: CWE-489-Webview-debugging
    patterns:
      - pattern-either:
        - pattern: $X.setWebContentsDebuggingEnabled(...)
    message: Semgrep found a match
    languages: [java]
    severity: INFO
```

### What about JavaScriptInterface ?

JavaScript Interface is one of those powerful API exposed by WebView to webpage to execute Java Methods from Javascript. This have previously escalated to [Remote Code Execution](https://labs.withsecure.com/publications/webview-addjavascriptinterface-remote-code-execution) in prior versions of Android JellyBean (API 16). It warrants a seperate blog post on how to detect, exploit and mitigate JavaScript Interface related vulnerability.

### Source and Reference:

1. [The First Mobile-Only SaaS Company](https://tomtunguz.com/mobile-only-saas/)
2. [JavaScriptInterface Based Vulnerability Publication](https://labs.withsecure.com/publications/webview-addjavascriptinterface-remote-code-execution)

### Closing Note:

Though this static analysis rule only helps in detecting patterns across Android repositories, However we could always improve the detection mechanism by leveraging [taint tracking](https://semgrep.dev/docs/writing-rules/data-flow/taint-mode/) to prevent false positives and use dynamic analysis to test the webview based vulnerability in real time virtual machine. In the Notion of keeping blog post short, I'll cover those taint tracking and dynamic analysis methods in upcoming posts. Meanwhile, my thesis on [Detecting Exploitable Vulnerabilities in Android Applications](https://uwspace.uwaterloo.ca/handle/10012/17034) can be more helpful to give an overview of the dynamic analysis part. 

I'm yet to publish Mobile Based Semgrep/CodeQL Static Analysis ruleset to detect vulnerable patterns in Mobile Source Code. Meanwhile, I hope this post is helpful for vulnerability researcher 🔍 & code reviewers, For bugs,hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.
