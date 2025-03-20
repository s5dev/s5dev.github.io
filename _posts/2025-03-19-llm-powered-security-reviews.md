---
layout: post
title: "LLM-Powered Security Reviews: Insights and Challenges"
categories: [llm, ai]
tags: [productivity, security]
description: "Exploring the potential and challenges of LLM-assisted security reviews"
comments: true
---

### Introduction

In a previous post on the [Sherlock blog](https://shivasurya.me/security-reviews/sast/2024/06/27/automate-security-code-reviews-with-cody-ai.html), I discussed leveraging large language models (LLMs) to assist with security code reviews. There’s no doubt that LLMs outperform traditional static application security testing (SAST) tools in several ways, enhancing the security review process by:

- Reducing false positive rates  
- Increasing the accuracy of findings  
- Uncovering previously unidentified edge cases  

When used in conjunction with SAST tools, LLMs can significantly boost the effectiveness of security reviews.

As I continue to explore research publications, delve into the internals of LLMs, and conduct various experiments, I find myself reflecting on a few key questions to validate these results mathematically:

- **RQ1. How can we mathematically prove the reliability of LLM findings?**
- **RQ2. Can LLMs fail to identify certain edge cases or vulnerabilities? How can we mitigate this?**
- **RQ3. What techniques are being used to provide additional context for security reviews?**
- **RQ4. If LLMs are so effective, why aren’t they widely used to find vulnerabilities in open-source software (OSS)?**

Below, I’ll share my thoughts on these questions and how I approach leveraging LLMs for security reviews.

---

### Diving Deeper 🤿

#### RQ1: How can we mathematically prove the reliability of LLM findings?

To the best of my knowledge, there is currently no direct way to mathematically validate the reliability of LLM findings using internal metrics. The most effective approach is to assess the quality of LLM outputs through benchmarking. This involves:

- **Building benchmarks** using existing vulnerability datasets from OSS or internal sources and evaluating LLM responses against them.  
    - For example, the Purple Llama project has developed an interesting [CyberSecEval benchmark](https://arxiv.org/abs/2312.04724).  
    - Similarly, the IRIS project has curated the [CWE-Bench-Java dataset](https://github.com/iris-sast/cwe-bench-java).  
- **Understanding LLM internals** and exploring the underlying mathematics.  
    - [Building LLM from Scratch by Sebastian](https://www.manning.com/books/build-a-large-language-model-from-scratch) is an excellent resource for breaking down the complex math behind LLMs.  
    - Gaining this understanding has significantly boosted my confidence in LLM results and their ability to perform a variety of NLP tasks.

---

#### RQ2: Can LLMs fail to identify certain edge cases or vulnerabilities? How can we mitigate this?

Yes, LLMs can sometimes miss vulnerable patterns or edge cases. This is often attributed to their non-deterministic nature, which stems from floating-point calculations and inherent randomness. While there’s no definitive mathematical explanation for these misses, here are some ways to mitigate them:

- **Providing additional context** to the LLM.  
- **Incorporating more example data during fine-tuning**, though this risks overfitting the model.  

---

#### RQ3: What techniques are being used to provide additional context for security reviews?

This is a fascinating area where many companies and individuals are employing similar strategies. Here are some common techniques:

![mr-bean](/assets/media/mr-bean-copying-meme.jpg)

1. **Abstract Syntax Tree (AST) parsing, Language Server Protocol (LSP), or code navigation** to traverse source code and provide extra context.  
    - [Windsurf](https://codeium.com/blog/using-code-syntax-parsing-for-generative-ai) uses this approach.  
    - [Nuanced.dev](https://www.nuanced.dev/blog/initial-launch), a YC company, has done interesting work for Python code.  
    - [CodeQL](https://github.com/github/codeql/tree/main/java/ql/automodel/src) employs sophisticated queries to enable LLM-based classification tasks.  
2. **Dumping entire codebases into LLMs** using tools like code2prompt.  
    - Based on my experience with Gemini models, this approach works well for flagging vulnerabilities but often requires multi-shot prompts to iterate and improve accuracy.  
3. **Brute force and filtering.**  
    - Asking the LLM to list all potential vulnerabilities in the code.  
    - Iteratively refining results by providing additional context, such as method definitions, references, predefined prompts, or rules.  
4. **Using LLMs for basic classification tasks.**  
    - This is particularly effective for reducing false positives.  
    - Example: If a piece of code is flagged as vulnerable but uses a specific library, the LLM can classify it as a false positive and suppress the alert.

---

#### RQ4: If LLMs are so effective, why aren’t they widely used to find vulnerabilities in OSS?

This is a question I’m surprised more people aren’t asking. Many seem to dismiss the idea of combining LLMs with security living in their own bubble. So, here are few evidence that it's working:

- **XBOW Security**, founded by former CodeQL builders, appears to be quietly hunting low-hanging vulnerabilities. [Source](https://github.com/advisories?query=credit%3Axbow-security)  
    - I’m curious about the sophistication of their automation and token usage.  
- Google’s Project Zero has explored this space in their blog post, [From Naptime to Big Sleep: Using Large Language Models To Catch Vulnerabilities In Real-World Code](https://googleprojectzero.blogspot.com/2024/10/from-naptime-to-big-sleep.html).  

---

### Interesting Research and Resources

Here are some interesting research papers and resources related to LLM-assisted security reviews:

- **LLM-Assisted Static Analysis for Detecting Security Vulnerabilities** - [Arxiv](https://arxiv.org/abs/2405.17238)  
- **Resolving Code Review Comments with ML** - [Google Research](https://research.google/blog/resolving-code-review-comments-with-ml/)  
- **Purple Llama CyberSecEval: A Secure Coding Benchmark for Language Models** - [Arxiv](https://arxiv.org/abs/2312.04724)  
- **CWE-Bench-Java Benchmark Dataset** - [GitHub](https://github.com/iris-sast/cwe-bench-java)
- <I'll keep updating if I come across interesting stuffs>  

---

This post reflects my ongoing journey of exploring how LLMs can revolutionize security reviews. While there are still challenges to overcome, the potential is undeniable. I hope you find this blog post useful. For bugs or hugs & discussion, DM me on [X](https://x.com/sshivasurya). Opinions are my own and not the views of my employer.