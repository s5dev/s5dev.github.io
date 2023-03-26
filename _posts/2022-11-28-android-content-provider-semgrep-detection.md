---
layout: post
title: Detecting Android Content Provider APIs with Semgrep Rules
categories: [security, android, android-security]
tags: [security, android]
description: Content provider APIs are powerful way to expose data to internal or external apps within Android ecosystem. However, there are lot of ways these APIs are implemented with flaws that leads to serious data leakage and even Remote code execution.
comments: true
---

Content Provider is one of the powerful APIs which helps Android developers programmatically expose resource content within Android ecosystem via Intents. One could easily write those queries easily by extending the `ContentProvider` class and implementing those methods and accessing via URI (example: `android://com.zoho.example/database/:_data`). Though these Content Provider is a cupcake for developers, Unfortunately there are lot of vulnerabilities hidden within those APIs and with implementation part. 

The main intent for writing this blog post were Semgrep and the recent blog post from `project zero` regarding [Analysis of a Samsung in-the-wild exploit chain](https://googleprojectzero.blogspot.com/2022/11/a-very-powerful-clipboard-samsung-in-the-wild-exploit-chain.html). I've been using semgrep for a while to tweak my findings instead of naive grep, CodeQL and the Samsung exploit chain may look trivial but how a simple permission bypass can affect system level apps in the Android phone. Later this year, I have added semgrep to my mobile pentesting suite which helps me to run these scripts over large Android projects, decompiled projects in automated way which pings me on Slack 🤖.

### Quick Overview

Content providers has various vulnerability patterns (not an exhaustive list) that includes,

1. arbitrary read & write into app sandbox 
2. arbitrary SQL queries
3. exported content providers
4. exposed content provider <= API 16 (Jelly Bean) 
5. exposed content provider > API 23 when compiled against API 16 (reported by me around 2019) 

and even more 🔥

In this post, we'll take a look at `arbitrary read & write into app sandbox` which means a exported content provider allows arbitrary file read and write within the app sandbox.

### Source and Sink

The source would be `OpenFile(Uri, String)` and sink is `openFileHelper(Uri, String)`. Here is the quick overview about openFile and openFileHelper APIs if you're not familiar about them,

1. `openFile(Uri, String)` - openFile abstract method helps in serving file request via Content provider intent

2. `openFileHelper(Uri, String)` - openFileHelper is one of the inbuild method helps in accessing files from application sandbox with specified mode (read, write, append)

### Semgrep Rule

```yaml
rules:
- id: android-content-provider-path-traversal-issue
  patterns:
    - pattern: openFileHelper(...)
    - pattern-inside: |
        @Override
        public ParcelFileDescriptor openFile(...) {
          ...
        }
  message: Semgrep found a match
  languages: [java]
  severity: WARNING
```

### Results

You can headover to Semgrep playground and play around with the semgrep rule and sample code [here](https://semgrep.dev/s/oJwn) 🎉

### Source and Reference:

1. Project Zero Blog: [A Very Powerful Clipboard: Analysis of a Samsung in-the-wild exploit chain](https://googleprojectzero.blogspot.com/2022/11/a-very-powerful-clipboard-samsung-in-the-wild-exploit-chain.html)
2. [Semgrep Join Mode](https://semgrep.dev/docs/writing-rules/experiments/join-mode/overview/)

### Closing Note:

Though this static analysis rule only helps in detecting patterns across Android repositories, However we could always improve the detection mechanism by leveraging [taint tracking](https://semgrep.dev/docs/writing-rules/data-flow/taint-mode/) to prevent false positives and use dynamic analysis to test the traversal vulnerability in real time virtual machine. In the Notion of keeping blog post short, I'll cover those tain tracking and dynamic analysis methods in upcoming posts. Meanwhile, my thesis on [Detecting Exploitable Vulnerabilities in Android Applications](https://uwspace.uwaterloo.ca/handle/10012/17034) can be more helpful to give an overview of the dynamic analysis part.

I hope this post is helpful for vulnerability researcher 🔍 & code reviewers, For bugs,hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.
