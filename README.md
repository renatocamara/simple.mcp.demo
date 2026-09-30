# Dad Joke MCP Server Lab

This fork is maintained by [Renato Camara](https://github.com/renatocamara).

![.NET 10](https://img.shields.io/badge/.NET-10-512BD4?style=for-the-badge&logo=dotnet)
![MCP Lab](https://img.shields.io/badge/MCP-Lab-0A7A5E?style=for-the-badge)
![Transports](https://img.shields.io/badge/Transports-StdIO%20%7C%20SSE-2E3A59?style=for-the-badge)
![Open For Laughs](https://img.shields.io/badge/Open_for-Laughs-F4A261?style=for-the-badge)

Welcome to a small MCP lab with one serious goal: learn fast, then ship your own tools.

This repository is a small, hands-on lab for learning how to build a simple MCP server in .NET 10.

> [!IMPORTANT]
> This is a demo for learning. It is not intended to be a production blueprint.

> [!TIP]
> For a security-focused journey, see the [MCP Security Summit Workshop](https://azure-samples.github.io/sherpa/).

## What You Will Build

* Build and run an MCP server with practical tool examples
* Choose your transport: StdIO or SSE over HTTP
* Run locally from source or inside Docker
* Extend the server with your own tool methods

## What Is Included

### Core components

* MCP server powered by `ModelContextProtocol` (`0.1.0-preview.2`)
* StdIO transport for local client integrations
* SSE transport at `http://localhost:3001` by default

### Included tools

* `GetDadJoke`
* `GetDadJokeCategories`
* `GetDadJokesByCategory`
* `Echo`

## Architecture Snapshot

```text
+----------------------+        +-------------------------+
|  MCP Client/Host     | -----> |  Dad Joke MCP Server    |
|  (VS Code, etc.)     |        |  (.NET 10)              |
+----------------------+        +-----------+-------------+
                                            |
                                            v
                                +-------------------------+
                                |  DadJokeService         |
                                |  Data/Jokes.json        |
                                +-------------------------+
```

## Quick Start

### Prerequisites

* .NET 10 SDK or later
* Basic familiarity with MCP
* Docker Desktop (optional)

### Build once

```powershell
git clone https://github.com/renatocamara/simple.mcp.demo.git
Set-Location simple.mcp.demo
dotnet build
```

### Connect with `mcp.json` using StdIO

```json
{
    "servers": {
        "dad-jokes-stdio-local": {
            "type": "stdio",
            "command": "dotnet",
            "args": [
                "run",
                "--project",
                "C:\\path\\to\\simple.mcp.demo\\DadJokeMCPStdIO\\DadJokeMCPStdIO.csproj"
            ]
        }
    }
}
```

Update the project path for your machine.

## Choose How To Run It

| Option | Best for | Why choose it | Trade-off |
|---|---|---|---|
| Hosted SSE URL | Shared dev endpoint | No local startup required | Needs reachable service |
| Local source with `dotnet run` | Active development | Fast debug and edit loop | Requires local SDK setup |
| Docker with `docker run` | Consistent environments | Reproducible runtime | Requires Docker images |

Simple guide:

* Use hosted SSE when you already have a running endpoint.
* Use local source when you are actively changing code.
* Use Docker when you want consistent behavior across machines.

## MCP Configuration Examples

### Hosted endpoint with SSE

```json
{
    "servers": {
        "dad-jokes-hosted": {
            "type": "sse",
            "url": "https://your-host.example.com/mcp"
        }
    }
}
```

Local hosted equivalent:

```json
{
    "servers": {
        "dad-jokes-hosted-local": {
            "type": "sse",
            "url": "http://localhost:3001/mcp"
        }
    }
}
```

### Run from local source

StdIO project:

```json
{
    "servers": {
        "dad-jokes-stdio-local": {
            "type": "stdio",
            "command": "dotnet",
            "args": [
                "run",
                "--project",
                "C:\\path\\to\\simple.mcp.demo\\DadJokeMCPStdIO\\DadJokeMCPStdIO.csproj"
            ]
        }
    }
}
```

SSE project:

```json
{
    "servers": {
        "dad-jokes-sse-local": {
            "type": "sse",
            "url": "http://localhost:3001/mcp",
            "command": "dotnet",
            "args": [
                "run",
                "--project",
                "C:\\path\\to\\simple.mcp.demo\\DadJokeMCPSSE\\DadJokeMCPSSE.csproj"
            ]
        }
    }
}
```

### Run with Docker

Build images:

```powershell
docker build -f DadJokeMCPStdIO/Dockerfile -t dadjokemcp:local .
docker build -f DadJokeMCPSSE/Dockerfile -t dadjokemcp-sse:local .
```

StdIO container:

```json
{
    "servers": {
        "dad-jokes-stdio-docker": {
            "type": "stdio",
            "command": "docker",
            "args": [
                "run",
                "-i",
                "--rm",
                "dadjokemcp:local"
            ]
        }
    }
}
```

SSE container:

```json
{
    "servers": {
        "dad-jokes-sse-docker": {
            "type": "sse",
            "url": "http://localhost:3001/mcp",
            "command": "docker",
            "args": [
                "run",
                "--rm",
                "-p",
                "3001:3001",
                "dadjokemcp-sse:local"
            ]
        }
    }
}
```

## Add A New Tool In 3 Steps

1. Create a class with `[McpServerToolType]`.
2. Add methods with `[McpServerTool]`.
3. Add `[Description]` attributes for discoverability.

```csharp
[McpServerToolType]
public static class CustomTool
{
    [McpServerTool, Description("Returns a custom response")]
    public static string ToolMethod(string param) => $"Result: {param}";
}
```

## Project Layout

* `DadJokeMCPStdIO` contains the StdIO server implementation.
* `DadJokeMCPSSE` contains the SSE over HTTP implementation.
* `DadJokeMCP.Shared` contains shared services, models, and joke data.

## Dependencies

* `Microsoft.Extensions.Hosting` (`9.0.3`)
* `ModelContextProtocol` (`0.1.0-preview.2`)
* `System.Text.Json` (`9.0.3`)

## SSE Notes

`DadJokeMCPSSE` runs as an ASP.NET Core app and keeps the same tools and joke service behavior as the StdIO project.

For transport hardening, read [MCP SSE security recommendations](https://modelcontextprotocol.io/docs/concepts/transports#security-warning%3A-dns-rebinding-attacks).

## License

MIT
