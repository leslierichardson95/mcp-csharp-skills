# MCP C# Skills Collection

A comprehensive set of GitHub Copilot skills for building Model Context Protocol (MCP) servers using C# and .NET.

## Overview

These skills guide you through the complete lifecycle of MCP server development - from project creation to cloud deployment. MCP servers enable LLMs to interact with external services through well-designed tools, prompts, and resources.

## Skills

| Skill | Description |
|-------|-------------|
| **[mcp-csharp](mcp-csharp/SKILL.md)** | Main orchestrator for C# MCP development. Use as your entry point to navigate the complete workflow. |
| **[mcp-csharp-create](mcp-csharp-create/SKILL.md)** | Create new MCP servers using the official C# SDK and Microsoft templates. Covers stdio and HTTP transports. |
| **[mcp-csharp-debug](mcp-csharp-debug/SKILL.md)** | Run and debug MCP servers locally. Covers VS/VS Code configuration, MCP Inspector, and GitHub Copilot Agent Mode testing. |
| **[mcp-csharp-test](mcp-csharp-test/SKILL.md)** | Test MCP servers with unit tests, integration tests, and LLM effectiveness evaluations. |
| **[mcp-csharp-publish](mcp-csharp-publish/SKILL.md)** | Publish and deploy MCP servers to NuGet (stdio), Docker, or Azure (HTTP). |

## Quick Start

### Prerequisites

- .NET 10.0 SDK or later
- Visual Studio 2022+ or VS Code with C# Dev Kit
- GitHub Copilot (optional, for Agent Mode testing)

### Create Your First MCP Server

```bash
# Install the template
dotnet new install Microsoft.McpServer.ProjectTemplates

# Create a stdio server (local/CLI)
dotnet new mcpserver -n MyMcpServer

# Or create an HTTP server (remote/web)
dotnet new mcpserver -n MyMcpServer --transport http
```

## Development Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Create    │ →  │    Debug    │ →  │    Test     │ →  │   Publish   │
│  mcp-csharp │    │  mcp-csharp │    │  mcp-csharp │    │  mcp-csharp │
│   -create   │    │   -debug    │    │   -test     │    │  -publish   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

1. **Create** - Scaffold your project, add tools with `[McpServerTool]` attributes
2. **Debug** - Run locally, configure IDE, test with MCP Inspector
3. **Test** - Write unit/integration tests, create LLM evaluations
4. **Publish** - Deploy to NuGet (stdio) or Docker/Azure (HTTP)

## Transport Options

| Transport | Use Case | Publish To | Run Command |
|-----------|----------|------------|-------------|
| **stdio** | Local/CLI integrations | NuGet.org | `dnx YourPackage@1.0.0` |
| **HTTP** | Remote/web services | Docker/Azure | Container URL |

## Installation

Copy these skill folders to your Copilot skills directory:

- Windows: `%USERPROFILE%\.copilot\skills\`
- macOS/Linux: `~/.copilot/skills/`

## License

See individual LICENSE.txt files in each skill folder.
