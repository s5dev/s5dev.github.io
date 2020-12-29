---
layout: post
title: Securing an ExpressJS server against CSRF Attacks - Part 2
categories: [security, server, nodejs]
tags: [security, waf]
description: Strategies for securing expressjs server.
---

Cross-site request forgery (also known as CSRF) is a web security vulnerability that allows an attacker to induce users to perform actions that they do not intend to perform. It allows an attacker to partly circumvent the same origin policy, which is designed to prevent different websites from interfering with each other.

### Detection Techniques
Developers love to use libraries for abstraction and better code management. Npm provides excellent support for packaging and distributing libraries as modules for developers. One of the most common software security and vulnerability incidents revolve around the outdated third party dependencies or vulnerabilities within the library.

As codebase tends to grow when features grow, the developers tend to keep the same dependencies that don't break the build or existing features. But this comes with the cost of hidden security vulnerabilities in the code. The famous Equifax breach with several million stolen credit information is due to by a vulnerability in the Java Framework, which has an outdated dependency. With the patch available for more than three months, Equifax failed to apply and secure the webserver.

#### Mitigation:
1. Developers should often update the libraries and adopted framework.
2. Subscribe for security newsletter or updates from the developer of the library.

### Takeaways:
1. Be aware of libraries, modules within the application
2. Check for transitive dependencies
3. Use tools like snyk.io to detect, fix vulnerable libraries in the development environment
4. Subscribe to the mailing list, updates newsletter for receiving updates on security issues or notice from developers.

For bugs/hugs, contact via [Twitter](https://twitter.com/sshivasurya).
