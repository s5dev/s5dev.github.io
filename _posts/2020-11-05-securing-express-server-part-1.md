---
layout: post
title: Securing an ExpressJS server - Part 1
categories: [security, server, nodejs]
tags: [security, waf]
description: Strategies for securing expressjs server.
---

As Javascript programming language popularity increases, platforms have already started adopting it from native desktop apps, mobile, browser to server-side, giving rise to exciting frameworks, style guides, tools.

To JavaScript—you weren't born with a silver spoon in your mouth, but you've outclassed every language that's challenged you in the browser.

ExpressJS is not an exception that powers [2.31% of the top 1 million websites](https://trends.builtwith.com/framework/Express) which runs on top of NodeJS and provides excellent features to develop web-based applications. So, let's jumpstart with a few basics, and this particular series will cover a lot more aspects of securing, maintaining and deploying production-grade expressjs server.

### Maintaining and security third party dependencies
Developers love to use libraries for abstraction and better code management. Npm provides excellent support for packaging and distributing libraries as modules for developers. One of the most common software security and vulnerability incidents revolve around the outdated third party dependencies or vulnerabilities within the library.

As codebase tends to grow when features grow, the developers tend to keep the same dependencies that don't break the build or existing features. But this comes with the cost of hidden security vulnerabilities in the code. The famous Equifax breach with several million stolen credit information is due to by a vulnerability in the Java Framework, which has an outdated dependency. With the patch available for more than three months, Equifax failed to apply and secure the webserver.

#### Mitigation:
1. Developers should often update the libraries and adopted framework.
2. Subscribe for security newsletter or updates from the developer of the library.

#### Tools:
There are a lot of tools available in the market to help developers to keep informed about the vulnerabilities. [Snyk.io](https://snyk.io) provides exceptional support to scan vulnerabilities and outdated libraries from the command line in no time for busy developers who run behind the deadlines. Integrate snyk with your CI/CD pipeline will help in assisting the fix for vulnerable libraries before deploying to production.

### Takeaways:
1. Be aware of libraries, modules within the application
2. Check for transitive dependencies
3. Use tools like snyk.io to detect, fix vulnerable libraries in the development environment
4. Subscribe to the mailing list, updates newsletter for receiving updates on security issues or notice from developers.

For bugs/hugs, contact via [Twitter](https://twitter.com/sshivasurya).
