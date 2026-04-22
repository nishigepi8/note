---
title: MCP：Model Context Protocol 入门与实践
description: 从概念、配置到跨仓库协作，系统理解 MCP 在 AI 开发工作流中的作用。
author: ga666666
date: 2026-01-12
updated: 2026-03-12
keywords: MCP, Model Context Protocol, Claude Code, Cursor, AI Agent, 上下文工程
tags: [AI, MCP, Agent, 上下文工程]
---

# MCP：Model Context Protocol 入门与实践

> 发布日期：2026-01-12
> 最后更新：2026-03-12
> 说明：涉及 Claude Code、Cursor 与 MCP Server 的配置项时，请以各产品当期官方文档为准。

本文介绍 **Model Context Protocol (MCP)** 是什么、为什么重要、如何配置与使用，以及在与 [跨仓库 AI 协作方案](./OpenSpec专题/跨仓库AI协作方案-Spec与上下文工程.md) 结合时的实践建议。

---

## 一、MCP 是什么

### 1.1 定义与目标

**Model Context Protocol (MCP)** 是由 **Anthropic** 提出的开放标准，用于让 **AI 模型（如 Claude）** 以统一方式连接 **外部工具、数据源与系统**。

可以把它理解为「**AI 的通用接口**」：  
不针对每个工具各写一套集成，而是由 **MCP Server** 暴露能力，**MCP Client**（如 Claude、Cursor）通过同一协议调用，从而：

- 突破「仅限当前对话与已打开文件」的限制
- 访问数据库、API、文档、Git、云服务等
- 在多次对话、多个会话间复用同一套工具能力

### 1.2 架构：Client–Server

```mermaid
graph LR
    subgraph Client 端
        Claude[Claude / Claude Code]
        Cursor[Cursor]
      end

    subgraph MCP 协议
        Protocol[MCP 协议]
      end

    subgraph Server 端
        Postgres[(Postgres MCP)]
        FS[Filesystem MCP]
        GitHub[GitHub MCP]
        Custom[自定义 MCP Server]
      end

    Claude -->|MCP Client| Protocol
    Cursor -->|MCP Client| Protocol
    Protocol -->|MCP Server| Postgres
    Protocol -->|MCP Server| FS
    Protocol -->|MCP Server| GitHub
    Protocol -->|MCP Server| Custom
```

- **MCP Server**：对外提供「能力」——如查数据库、读文件、调 API、执行 Git 操作。可本地运行（stdio）或远程（HTTP/SSE）。
- **MCP Client**：内置于 Claude、Cursor 等产品，负责发现 Server、发送请求、把结果交给模型做推理。
- **协议**：规定请求/响应格式、能力描述方式（如 tools、resources），实现 Client 与 Server 的解耦。

### 1.3 没有 MCP vs 有 MCP

| 维度 | 没有 MCP | 有 MCP |
|------|----------|--------|
| 数据来源 | 仅对话中粘贴的内容、已打开文件 | 数据库、API、文档、Git、监控等 |
| 实时性 | 依赖人工复制最新数据 | Agent 按需查询最新状态 |
| 集成方式 | 各产品各自对接，不统一 | 统一协议，换 Client 或 Server 即可复用 |
| 跨会话 | 每次重新粘贴 | Server 常驻，多会话共享 |

---

## 二、MCP 能做什么

典型能力包括（具体取决于每个 Server 的实现）：

- **数据**：连接 Postgres、MySQL、SQLite 等，查 schema、跑查询、分析数据。
- **文件与文档**：读本地或远程文件、目录结构、配置文件、OpenSpec specs。
- **开发与协作**：Git 操作、GitHub/GitLab issue/PR、代码搜索。
- **网络**：调用 HTTP API、抓取网页内容（fetch）。
- **自定义**：内部 API、工单系统、监控告警等，通过自建 MCP Server 暴露。

在 [跨仓库 AI 协作方案](./OpenSpec专题/跨仓库AI协作方案-Spec与上下文工程.md) 中，MCP 的典型用法是：

- 前端 Agent 通过 MCP 读取 **API 文档 / OpenAPI / specs**；
- 后端 Agent 通过 MCP 读取 **数据库 schema** 或 **OpenSpec specs**；
- 避免频繁复制粘贴、减少「信息过期」带来的错误。

---

## 三、怎么用：配置与连接

### 3.1 Claude Desktop / Claude Code

Claude 官方支持通过配置添加 MCP Server，常见方式：

**1）stdio（本地进程）**

本地启动一个进程，通过标准输入/输出与 Claude 通信：

```bash
# 示例：添加一个本地的 Postgres MCP Server
claude mcp add --transport stdio postgres -- npx -y @modelcontextprotocol/server-postgres postgres://user:pass@localhost:5432/dbname
```

**2）HTTP（推荐，适合远程）**

Server 以 HTTP 服务形式运行，Claude 通过 URL 连接：

```bash
claude mcp add --transport http my-server https://your-mcp-server.example.com
```

**3）配置文件**

在 Claude 的配置目录中编辑 MCP 配置（路径因系统而异），例如：

```json
{
  "mcpServers": {
    "postgres": {
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgres://localhost:5432/mydb"]
    },
    "filesystem": {
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"]
    }
  }
}
```

具体字段名、路径以 [Claude 官方 MCP 文档](https://docs.anthropic.com/en/docs/claude-code/mcp) 为准。

### 3.2 Cursor

Cursor 支持在设置中配置 MCP Server，从而让 Cursor Agent 使用同一批能力：

- 打开 **Settings → MCP**（或对应入口）。
- 添加 Server：选择 **stdio** 或 **HTTP**，填写命令或 URL。
- 保存后，在对话或 Agent 任务中即可使用该 Server 暴露的工具。

配置形态与 Claude 类似（stdio 命令 + 参数，或 HTTP URL）。

### 3.3 常用官方/社区 MCP Server 示例

| Server | 用途 | 典型命令/包 |
|--------|------|-------------|
| **Postgres** | 查 schema、执行只读/写 SQL | `@modelcontextprotocol/server-postgres` |
| **Filesystem** | 读指定目录下的文件 | `@modelcontextprotocol/server-filesystem` |
| **GitHub** | 仓库、issue、PR、搜索 | `@modelcontextprotocol/server-github` |
| **Fetch** | HTTP 请求、抓取网页 | `@modelcontextprotocol/server-fetch` |
| **Git** | 本地 Git 仓库操作 | 对应 Git MCP 实现 |
| **Slack / Google Drive** | 协作与文档 | 官方或社区实现 |

安装与运行方式多为 `npx -y <package名> [参数]`，具体见各 Server 的 README。

---

## 四、跨仓库与跨项目中的 MCP 实践

### 4.1 用 MCP 暴露「共享 Spec」

在 [跨仓库 AI 协作方案](./OpenSpec专题/跨仓库AI协作方案-Spec与上下文工程.md) 中，多个仓库共享同一套 OpenSpec 时：

- 可单独起一个 **MCP Server**，对外暴露 **openspec-shared** 的 `specs/`（或整个 openspec 目录）。
- 前端仓库、后端仓库的 Agent 都连接这个 Server，实时读取最新 API/UI spec，无需手动复制或等 submodule 同步。

```mermaid
graph LR
    FE[Cursor / Claude 前端] -->|MCP| SpecServer[Spec MCP Server]
    BE[Cursor / Claude 后端] -->|MCP| SpecServer
    SpecServer --> FS[openspec-shared/specs]
```

实现方式可以是：一个简单的 **Filesystem MCP Server** 限定到 `openspec-shared` 的路径，或自写一个小型 HTTP Server 封装「按模块/文件名读取 spec」的接口。

### 4.2 数据库与 API 文档

- **后端开发**：为 Agent 配置 **Postgres/MySQL MCP**，连接开发库，便于查表结构、写安全查询、排查数据问题。
- **前端/全栈**：配置 **Fetch MCP** 或自建「API 文档 MCP」，让 Agent 读取 OpenAPI/Swagger 或内部文档站，生成或校验请求代码。

### 4.3 安全与权限

- **最小权限**：数据库账号建议只读或仅限开发库；Filesystem 只开放必要目录。
- **敏感信息**：连接串、API Key 不要写进文档，使用环境变量或本地密钥管理。
- **生产**：生产库、生产 API 一般不直接对 MCP 开放；必要时通过只读副本或专门网关。

---

## 五、自建 MCP Server 简要

若现有 Server 不满足需求，可自建：

1. **实现 MCP 协议**：按 [MCP 规范](https://modelcontextprotocol.io/) 实现 Server（暴露 tools/resources）。
2. **传输方式**：stdio（本地子进程）或 HTTP（远程服务）。
3. **注册到 Claude/Cursor**：在各自配置中添加该 Server 的命令或 URL。

语言不限（Node、Python、Go 等均可），只要符合协议即可被任意 MCP Client 使用。

---

## 六、实践清单

- [ ] 在 Claude 或 Cursor 中至少添加一个 MCP Server（如 filesystem 或 postgres），验证「对话中可用」。
- [ ] 跨仓库场景下，用 MCP 暴露共享 specs 或 API 文档，让前后端 Agent 统一读取。
- [ ] 数据库与文件路径遵循最小权限；敏感信息用环境变量。
- [ ] 需要时查阅官方文档： [Claude Code MCP](https://docs.anthropic.com/en/docs/claude-code/mcp)、[MCP 官网](https://modelcontextprotocol.io/)。

---

**AI 协作系列**  
- [AI 驱动的端到端开发协作标准](./AI驱动的端到端开发协作标准.md)  
- [跨仓库 AI 协作方案：Spec 与上下文工程](./OpenSpec专题/跨仓库AI协作方案-Spec与上下文工程.md)  
- [Agent 执行流程与规范：AGENTS.md 与规则文档](./Agent执行流程与规范-AGENTS与规则文档.md)  
- [Agent Skill：是什么、怎么用与推荐](./Agent-Skill是什么怎么用与推荐.md)  
- [如何开发 AI Agent](./如何开发AI-Agent.md)
