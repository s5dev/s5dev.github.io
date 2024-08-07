---
layout: post
title: "Sherlock: Automate security code reviews with Cody AI"
categories: [security-reviews, sast]
tags: [programming, productivity]
description: "This blog post will discuss about semi-autonomous way to perform security code reviews"
comments: true
---

### Intro

### Need for semi-autonomous security code reviews

My job as a security engineer (application security context) is to read source code and perform security reviews. Most of the time, mainly corelate the source code with frameworks & libraries, understand context where the code executes and enumerate all security risks. While there are lot of second generation SAST scanning tools in the market which is good at identifying patterns, eliminate false positive, executes and brings up results in minutes. I believe,

- First generation SAST tools are based on symbolic execution and takes a lot of time to execute with lot of false postives
- Second generation SAST tools are really fast, identify patterns, tune rules, reduce false positives and still human review is required
- *Next generation should be semi-autonomous, understand context and perform security reviews that escalate to human reviews*

### Second generation SAST requires lot of manual effort

Having worked as software engineer & security engineer for more than 5 years now, I can confidently say that manual effort is the biggest bottleneck in the second generation SAST tools.

For example, SAST tooling & experts boldly claim using `dangerouslySetInnerHTML` is a security risk in reactjs based application. I can 100% say this is **false positive** because,

#### What SAST can do:
- Software Engineers and Developers are smart enough to know that `dangerouslySetInnerHTML` is a security risk but business usecases demand rendering html
- Though the html is santized and rendered, SAST tooling has to perform source sink analysis to identify the flow.

#### What SAST may do:
- What if source sink analysis has to run through multiple files, projects
- Perform intra-procedural (single file) data flow analysis and identify all possible paths & conditions

#### What SAST can't do:

While the first pattern matching, source sink analysis (intra-procedural analysis) are easier to perform but inter-procedural analysis with data flow analysis is more difficult. If it was easy, experts would have published the ruleset to the community. (except CodeQL which has really steep learning curve)

- TLDR: Inter-procedural (projects/multiple files) data flow analysis is hard. Writing generic rule to capture them has steep learning curve.

#### Effects
Now, the human security engineer has to triage the `dangerouslySetInnerHTML` finding and go over multiple files, identify flow, understand context only to finally identify and mark it as `false positive`. This indeed requires time, manual effort.

### Capabilities of LLM vs classic SAST

Unlike calling GPT APIs, I have attempted to train a model from scratch, and loaded open weights. After performing Instruction fine-tuning and classification surprised me that LLMs are better at pattern matching and better classifiers when context is known. Thanks to @rasbt for writing the book [Building Large Language Models from Scratch](https://www.manning.com/books/build-a-large-language-model-from-scratch).

| Code Review Task  | Capability  | Instruction fine-tuned LLM | SAST Tools  |
|:--------|:-------------|:-------------:|:-------------:|
| Find vulnerable pattern | Pattern matching  | ✅ | ✅ |
| Understand vulnerability class | Understand context | ✅ | ❌ |
| Identify false positive | Better classifier  | ✅  | ❌  |
| Reading through projects, multiple files | Fetch context  | ❌  | ❌  |

### Why Cody AI?

Context is King 👑 . No other assistant knows your codebase better than [Cody AI](https://sourcegraph.com/cody). Combining capabilities of LLM (kind of NLP tasks) and finding perfect context from the codebase can help in performing security code reviews.

### Show me the code 🧑‍💻

Cody has experimental CLI support for chat, autocomplete by specifying repo as context.

> npm install -g @sourcegraph/cody

```bash
# Ask Cody a question (with Sourcegraph Enterprise repository context):
cody cli chat --context-repo github.com/sourcegraph/sourcegraph --show-context -m 'how is authentication handled in sourcegraph?'
```

### How I integrate Cody AI into my workflow?

While i use Cody regularly for code completion and chat, I have integrated Cody into my workflow SAST tooling to perform additional security code reviews.

![Sherlock powered by Cody AI](/assets/media/sherlock.png)

### Closing Note:

For bugs or hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.