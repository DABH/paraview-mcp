<p align="center">
  <img src="Documentation/Images/paraview-mcp-logo.png" width="200" alt="ParaView MCP">
</p>

# ParaView MCP

[![CI](https://github.com/failed33/paraview-mcp/actions/workflows/ci.yml/badge.svg)](https://github.com/failed33/paraview-mcp/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/paraview-mcp-server)](https://pypi.org/project/paraview-mcp-server/)
[![CodeQL](https://github.com/failed33/paraview-mcp/actions/workflows/codeql.yml/badge.svg)](https://github.com/failed33/paraview-mcp/actions/workflows/codeql.yml)
[![codecov](https://codecov.io/gh/failed33/paraview-mcp/graph/badge.svg?token=imlWhPGAgh)](https://codecov.io/gh/failed33/paraview-mcp)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Connect [ParaView](https://www.paraview.org/) to LLM assistants through the [Model Context Protocol](https://modelcontextprotocol.io/).

`paraview-mcp-server` has two runtime parts:

- a **ParaView plugin** (C++/Qt) that exposes a TCP bridge inside the ParaView GUI
- a **Python MCP server** that connects to the plugin and serves tools to any MCP client

## Prerequisites

- [ParaView 5.13.3, 6.0.1, or 6.1.1](https://www.paraview.org/download/) with the matching MCP plugin loaded (see [Plugin Setup](#set-up-the-paraview-plugin))
- [uv](https://docs.astral.sh/uv/)

## Quick Start

Add to Claude Code in one command:

```bash
claude mcp add paraview -- uvx paraview-mcp-server
```

Then [set up the ParaView plugin](#set-up-the-paraview-plugin) and you're ready to go.

## Set Up the ParaView Plugin

Download a pre-built plugin binary from the [latest GitHub Release](https://github.com/failed33/paraview-mcp/releases/latest). Releases provide this matrix:

| Platform | Architecture | ParaView versions | Archive |
| --- | --- | --- | --- |
| Linux | x86_64 | 5.13.3, 6.0.1, 6.1.1 | `.tar.gz` |
| macOS | arm64 (Apple Silicon) | 5.13.3, 6.0.1, 6.1.1 | `.tar.gz` |
| Windows | x64 | 5.13.3, 6.0.1, 6.1.1 | `.zip` |

Choose the archive that names your exact ParaView version and platform, verify it with the adjacent `.sha256` file, then extract it and follow the included `INSTALL.md`. Pull requests also produce the same binaries as short-lived GitHub Actions artifacts; GitHub Releases are the permanent distribution channel.

Alternatively, build the plugin from source against a ParaView 5.13 or newer SDK. See [CONTRIBUTING.md](CONTRIBUTING.md) for full build instructions. Binary compatibility is release-series specific, so use a plugin built for your ParaView major.minor version.

Once installed:

1. Open **Tools > Manage Plugins** in ParaView.
2. Click **Load New...** and select `ParaViewMCP.so` (Linux/macOS) or `ParaViewMCP.dll` (Windows) from the plugin directory.
3. Enable **Auto Load**.
4. Open **Tools > ParaView MCP**.
5. Click **Start Server**.

The dock widget shows the connection status. Non-loopback binds require an auth token.

## Configure Your MCP Client

### Claude Code (CLI)

```bash
claude mcp add paraview -- uvx paraview-mcp-server
```

### Claude Desktop

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "paraview": {
      "command": "uvx",
      "args": ["paraview-mcp-server"]
    }
  }
}
```

### Cursor

Add to `.cursor/mcp.json` in your project root:

```json
{
  "mcpServers": {
    "paraview": {
      "command": "uvx",
      "args": ["paraview-mcp-server"]
    }
  }
}
```

## Configuration

The server connects to the ParaView plugin using these environment variables:

| Variable              | Default     | Required          | Description                                          |
| --------------------- | ----------- | ----------------- | ---------------------------------------------------- |
| `PARAVIEW_HOST`       | `127.0.0.1` | No                | Host where the ParaView plugin is listening          |
| `PARAVIEW_PORT`       | `9877`      | No                | TCP port for the plugin bridge                       |
| `PARAVIEW_AUTH_TOKEN` | —           | Non-loopback only | Authentication token (must match the plugin setting) |

Defaults work for a standard local setup. Override these when connecting to ParaView on a remote machine or non-standard port:

```json
{
  "mcpServers": {
    "paraview": {
      "command": "uvx",
      "args": ["paraview-mcp-server"],
      "env": {
        "PARAVIEW_HOST": "192.168.1.10",
        "PARAVIEW_PORT": "9877",
        "PARAVIEW_AUTH_TOKEN": "your-token"
      }
    }
  }
}
```

## Available Tools

| Tool                            | Description                                            |
| ------------------------------- | ------------------------------------------------------ |
| `execute_paraview_code(code)`   | Execute Python code inside the active ParaView session |
| `get_pipeline_info()`           | Return a JSON snapshot of the current pipeline         |
| `get_screenshot(width, height)` | Capture the active render view as a PNG image          |

## Design and Differences from Paraview_MCP

This project follows the approach of [Blender-MCP](https://github.com/ahujasid/blender-mcp) and [Slicer-MCP](https://github.com/pieper/SlicerMCP), both of which give LLMs direct code execution inside their respective application runtimes. The existing [Paraview_MCP](https://github.com/llnl/paraview_mcp) implementation[^1] takes a different approach, exposing a fixed set of high-level tools without access to the underlying Python runtime, which limits flexibility for custom workflows.

We instead provide an `execute_paraview_code` tool that runs arbitrary Python inside the ParaView session, giving the AI agent the same level of control a human scripter would have.

There is also an architectural difference. [Paraview_MCP's own disclaimer](https://github.com/LLNL/paraview_mcp/blob/30242a0a6768eaf4192529bb78096cfee3292c73/README.md#disclaimer) says its connection relies on synchronization between `pvserver` and the ParaView client, a feature deprecated in recent ParaView versions, and warns that this can cause incorrect application views and general stability issues. This project instead runs a plugin inside the interactive ParaView process and communicates with it through a TCP bridge, avoiding that `pvserver`/client synchronization path.

[^1]: S. Liu, H. Miao, and P.-T. Bremer, "Paraview-MCP: Autonomous Visualization Agents with Direct Tool Use," in _Proc. IEEE VIS 2025 Short Papers_, IEEE, 2025.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for build instructions, development setup, and pull request guidelines.

## License

[MIT](LICENSE) — see [THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt) for dependency licenses.
