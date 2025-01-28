---
layout: post
title: "How I Use AI to Streamline/Assist My Work"
categories: [llm, ai]
tags: [productivity]
description: "A short blog post on how I leverage LLMs (AI) to streamline or assist my work"
comments: false
---

## Intro

While there's a lot of skepticism about using AI to automate tasks, I've found AI tools to be invaluable allies that enhance my results and handle niche tasks.

### 🤖 Reflecting on My LLM Usage

I used to pay for OpenAI & Anthropic Claude API access and regularly automated several tasks until recently when DeepSeek-v3 was released, cutting costs by at least 50% while maintaining the same response quality. Here are a few tasks I found useful after attempting more than 30+ workflows to incorporate and derive value.

#### Automated & Detailed Brief About New Concepts

Using GPT-4 model to generate detailed briefs of concepts such as GoLang, technical concepts, bookmarks, and weekly newsletter links. Once you set up integration between Notion and [GraphRAG](https://microsoft.github.io/graphrag/), you can automate and query daily to populate Notion pages for learning. GraphRAG is an amazing tool to index and question your private content.

#### Automated & Detailed Brief About Coding Problems and Solutions

Using Claude Sonnet-3.5 model to generate detailed briefs solving coding problems with naive to optimized solutions, step-by-step breakdowns, edge cases, and unit test cases. Having used it for the past 6 months, the Claude-3.5 model is trained on most coding challenges and solutions, and now DeepSeek-v3 can do that at a fraction of the cost.

#### Monitor Latest Trends in AppSec, InfoSec News in Hacker News

As an active HN news follower, I recently set up ChatGPT-4 tasks to monitor HN front page (30 results) to filter out security and AppSec related submissions. Prior to ChatGPT's native task scheduler, I used Claude Sonnet-3.5 with curl to summarize HN news and show submissions to populate my Notion pages. (Notion markdown API sucks)

#### Automated Security Code Reviewer (Built on Top of Sourcegraph Cody AI)

[Sherlock](https://shivasurya.me/security-reviews/sast/2024/06/27/automate-security-code-reviews-with-cody-ai.html) - Automate security code reviews with Sourcegraph Cody AI. The core focus is more on uncovering low-hanging vulnerabilities rather than proving unreliable one-off high-impact findings. The bar for building such agents can be:

1. LLM + Code Reviews - Low bar (anyone can build it)
2. LLM + Custom rules + Code Reviews - Medium bar (anyone can build it)
3. LLM + Codebase Context + Code Reviews - High bar
4. LLM + Codebase Context + SAST + Customization + Code Reviews - Very High bar

I speculate that codebase context, integration with engineering tools, and customization is where the secret sauce lives.

#### Brainstorming New Ideas & ChatGPT Project Structure

ChatGPT projects is an amazing use case for having custom instruction prompts, file attachments, and organized chat history. You can simply switch context better and pick up chats where you left off previously instead of documenting what you were doing before.

Personally, I use ChatGPT projects to brainstorm ideas, create artifacts, generate task lists, or even summarize work.

#### Miscellaneous Tasks

##### Assistance with Writing Emails and Chats

For anything that requires more than 5 lines of email or chat message, ChatGPT does an amazing job customizing tone and generating replies.

##### ELI5 on Anything That I Don't Understand

Claude 3.5-Sonnet or DeepSeek-v3 can help in understanding complex topics or jargon by breaking them down just by adding the keyword "ELI5" (explain like I'm 5)

### Closing Note

I hope you find this blog post useful. For bugs or hugs & discussion, DM me on [X](https://x.com/sshivasurya). Opinions are my own and not the views of my employer.