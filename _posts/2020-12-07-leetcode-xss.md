---
layout: post
title: Cross-Site Scripting attack on Leetcode
categories: [security, server, client-security]
tags: [security, waf, xss]
description: DOM Cross-Site Scripting attack on leetcode.com.
---

Reflected XSS (Cross-Site Scripting) attack is my favorite vulnerability category as it's relatively easy to exploit by checking for params as the source and rendering DOM as the sink.

### Problem

The core problem of the Reflected Cross-Site scripting attack is appending the URL parameter values in the DOM without validation or filtering. Though the reflected XSS requires user interaction by visiting the page or clicking on links in real-life attacks, people should think about Iframe tags that don't need any interaction to load them on other third party web pages.

Cutting to the chase, the Leetcode submission page has an exciting parameter in the URL as "from," which contains URI used by the problem statement button. This button helps in navigating the user back to the problem statement seamlessly. The problem is that leetcode developers appended the from URL parameter value in the href tag without validation. This triggered XSS on \*.leetcode.com.

payload : https://leetcode.com/submissions/detail/415518712/?from=javascript:alert(document.domain)///explore/learn/card/data-structure-tree/134/traverse-a-tree/9298/

#### Mitigation:

1. A simple fix would be constructing the URL with HTTP/HTTPS and block javascript or custom URI origin.
2. [Attribute based encoding](https://portswigger.net/web-security/cross-site-scripting/contexts) to prevent XSS.

### Timeline:

1. Reported - Nov 2, 2020
2. Acknowledged on Nov 3, 2020
3. Fixes rolled out on ~Dec 5, 2020

### Reward:

- 300 Leet coins :) 🥇. Thanks to the Leetcode engineering team for the support 🙏.

For bugs/hugs, contact via [Twitter](https://twitter.com/sshivasurya).
