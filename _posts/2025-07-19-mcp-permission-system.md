---
layout: post
title: "Rethinking MCP or Tool Calling Through Permission Based System"
categories: [llm, ai]
tags: [mcp, security]
description: "Explore a permission-based security model for MCP and Tool Calling in LLMs, inspired by Android's runtime permissions, to protect sensitive data while maintaining functionality."
comments: true
---

Model Context Protocol (MCP) and Tool Calling are revolutionizing the application layer of Large Language Models (LLMs), enabling AI to autonomously operate tools and MCP servers to complete tasks. While these capabilities are typically distributed as npm packages or hosted remotely, this distribution method poses potential security risks through malicious code.

Despite these concerns, Tool Calling and MCP add significant value to AI applications. For instance, Windsurf IDE demonstrates excellent integration by leveraging various tools for file operations, diff viewing, and command execution. Users can configure their own MCP packages and servers, allowing models to control these tools effectively.

A particularly impressive use case involves test automation, where models can select appropriate test commands, interpret terminal output, run tests, verify coverage, and write code in a continuous feedback loop.

However, security concerns arise when dealing with sensitive data. Consider a scenario where an LLM assists with immigration forms by accessing personal documents from Dropbox. While efficient, using third-party MCP servers or npm packages with access to private data (like personal documents, financial information, or PII) presents significant risks.

The core issue lies in the potential for untrusted tokens or commands from external sources to manipulate the model into extracting sensitive information. This vulnerability necessitates a more secure approach to tool calling and MCP implementation.

A possible solution could involve implementing a trusted mediator system between the model and tools, similar to runtime or on-demand Android's permission dialog system. Key aspects would include:

![Android Based Permission system](/assets/media/android-permission-compare-mcp.gif)

1. A permission-based system for tool access where:
    - Tools are categorized by security levels (low, medium, high)
    - Sensitive operations require explicit user approval
    - Permissions can be granular (read-only, write, execute)
    - Users can revoke access at any time

2. A secure mediator layer that:
    - Validates all tool calls before execution
    - Maintains an audit log of operations
    - Enforces rate limits and usage quotas
    - Sanitizes inputs and outputs

3. Runtime security controls:
    - Sandboxed execution environments
    - Encrypted communication channels
    - Token-based authentication
    - Session management and timeouts

This architecture would provide fine-grained control over tool access while maintaining security and usability. *However this doesn't solve or detect the origin of malicious prompts but effective in gate-keeping sensitive resource.*



![LLM Tooling permission model](/assets/media/llm-tooling-permission.png)

### Limitations

While this permission-based approach enhances security, some key limitations remain:

1. Request Origin Ambiguity
    - The mediator cannot definitively verify the true origin of tool requests
    - Malicious tools could potentially masquerade as legitimate ones to access sensitive resources

2. Context Manipulation Risks
    - Once sensitive data is loaded into the model's context
    - Malicious tools could attempt to extract this information through carefully crafted prompts
    - The system cannot fully prevent context-based information leakage

Despite these limitations, the permission system provides essential control gates for sensitive resource access, allowing users to make informed decisions about tool access requests.

### Closing Note

The permission-based approach to MCP and tool calling represents a significant step toward more secure AI applications. While not a complete solution to all security challenges, it provides a practical framework for managing tool access and protecting sensitive data. As these technologies continue to evolve, further security enhancements will be crucial for building trustworthy AI systems that can safely handle sensitive operations.
