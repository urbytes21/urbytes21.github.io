---
author: "Phong Nguyen"
title: "Model Context Protocol (MCP)"
date: "2026-02-10"
description: "Model Context Protocol (MCP)"
tags: ["mcp"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 15    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
## 1. Introduce
- Modern applications are integrating with Large Language Models (LLMs) to build AI solution.
- The AI model's response will be more context-ware if we provide it external source like databases, file systems, pictures ,...
- MCP provides a standard way to connect AI-powered applications with external data sources.
  - ![image](/images/mcp_overview.png)<br>
<br>

## 2. Model Context Protocol
- Components overview:
  - ![image](/images/mcp_overview_diagram.png)<br>
- MCP follows a client-server architecture, that included:
  - **MCP Host:** is the main application that integrates with an LLMs and required to connect with external data source
  - **MCP Client:** are components that establish and maintain 1-1 connection with **MCP servers**
  - **MCP Servers:** are components that integrate with external data sources and expose apis to interact with them
  - MCP provides **two transport channels**, with many transport types:
    - `stdio`: to enable communication through standard I/O streams
    - `SSE`: HTTP

## 3. Java Example - Creating an MCP Host
Building an application (MCP Host):
 - act as a chatbot using a Claude LLMs model.
 - enable to use a local LLM

### 3.1. Dependency
- Create an java maven / jdk 21 project.
- Add the following dependencies to `pom.xml`: 
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-anthropic</artifactId>
    <version>1.0.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-client</artifactId> 
    <version>1.0.1</version>
</dependency>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.0.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
- Notes:
  - `spring-ai-starter-model-anthropic`: to interact with the Claud model
  - `spring-ai-starter-mcp-client`: to configure clients in the application to maintain 1-1 connection with the MCP servers.

- Navigate to [Claud](https://platform.claude.com/settings/keys) and create an API key.
-  
