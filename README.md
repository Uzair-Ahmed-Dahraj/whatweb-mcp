# WhatWeb MCP Server: web technology fingerprinting for AI agents

[![Docker Hub](https://img.shields.io/docker/v/devkind/whatweb-mcp?sort=semver&label=docker%20hub)](https://hub.docker.com/r/devkind/whatweb-mcp)
[![CI](https://github.com/devkindhq/whatweb-mcp/actions/workflows/ci.yml/badge.svg)](https://github.com/devkindhq/whatweb-mcp/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

WhatWeb MCP server lets Claude Code, Cursor, and any MCP client fingerprint websites and identify their CMS, web server, frameworks, and analytics tools. It runs WhatWeb in a hardened Docker container and returns a structured report in one tool call.

Sales and prospecting teams use it to qualify leads; security teams use it for recon.

The server has three [tools](#tools) for scanning a site and reading results. Scans run in a sandboxed container with a non-root user, memory and CPU limits, and flag-injection validation on every target.

## Quickstart (Docker)

The image is multi-arch (linux/amd64 and linux/arm64), so the same tag works on Apple Silicon, x86 servers, and ARM cloud instances.

```bash
docker pull devkind/whatweb-mcp:latest
```

Ready-paste config for Claude Code or Cursor, saved as `.mcp.json` in your project root (or `~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "whatweb": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "--memory=512m", "--cpus=0.5", "--pids-limit=100", "devkind/whatweb-mcp:latest"]
    }
  }
}
```

Then ask your agent: *"What technologies does example.com run?"*

### Install in your client

Requirements:

- Docker
- Claude Desktop, Claude Code, Cursor, VS Code, Windsurf, or any other MCP client
- No API keys

**Standard config** works in most MCP clients:

```json
{
  "mcpServers": {
    "whatweb": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--memory=512m",
        "--cpus=0.5",
        "--pids-limit=100",
        "devkind/whatweb-mcp:latest"
      ]
    }
  }
}
```

<details>
<summary>Claude Desktop</summary>

Follow the MCP install [guide](https://modelcontextprotocol.io/quickstart/user) and add the standard config above to your `claude_desktop_config.json`.

</details>

<details>
<summary>Claude Code</summary>

Run:

```bash
claude mcp add whatweb -- docker run -i --rm --memory=512m --cpus=0.5 --pids-limit=100 devkind/whatweb-mcp:latest
```

Or add the standard config above to `.mcp.json` in your project root.

</details>

<details>
<summary>Cursor</summary>

#### Click the button to install:

[<img src="https://cursor.com/deeplink/mcp-install-dark.svg" alt="Install in Cursor">](https://cursor.com/en/install-mcp?name=WhatWeb%20MCP&config=eyJjb21tYW5kIjoiZG9ja2VyIiwiYXJncyI6WyJydW4iLCItaSIsIi0tcm0iLCItLW1lbW9yeT01MTJtIiwiLS1jcHVzPTAuNSIsIi0tcGlkcy1saW1pdD0xMDAiLCJkZXZraW5kL3doYXR3ZWItbWNwOmxhdGVzdCJdfQ%3D%3D)

#### Or install manually:

Go to `Cursor Settings` -> `MCP` -> `Add new MCP Server`. Name it "WhatWeb MCP", use `command` type with the command `docker run -i --rm --memory=512m --cpus=0.5 --pids-limit=100 devkind/whatweb-mcp:latest`.

</details>

<details>
<summary>VS Code</summary>

Follow the MCP install [guide](https://code.visualstudio.com/docs/copilot/chat/mcp-servers#_add-an-mcp-server) and use the standard config above. You can also install with the VS Code CLI:

```bash
code --add-mcp '{"name":"whatweb","command":"docker","args":["run","-i","--rm","--memory=512m","--cpus=0.5","--pids-limit=100","devkind/whatweb-mcp:latest"]}'
```

</details>

<details>
<summary>Windsurf</summary>

Follow the Windsurf MCP [documentation](https://docs.windsurf.com/windsurf/cascade/mcp) and use the standard config above.

</details>

<details>
<summary>Goose</summary>

Go to `Advanced settings` -> `Extensions` -> `Add custom extension`. Name it "WhatWeb MCP", use type `STDIO`, and set the `command` to `docker run -i --rm --memory=512m --cpus=0.5 --pids-limit=100 devkind/whatweb-mcp:latest`. Click "Add Extension".

</details>

<details>
<summary>LM Studio</summary>

Go to `Program` in the right sidebar -> `Install` -> `Edit mcp.json`. Use the standard config above.

</details>

<details>
<summary>Warp</summary>

Go to `Settings` -> `AI` -> `Manage MCP Servers` -> `+ Add` to [add an MCP Server](https://docs.warp.dev/knowledge-and-collaboration/mcp#adding-an-mcp-server). Use the standard config above, or use the slash command `/add-mcp` in the Warp prompt and paste it.

</details>

## Tools

| Tool | What it does |
| --- | --- |
| `whatweb_scan` | Scan a URL or domain. Arguments: `target` (required), `aggression` (default `normal`), `timeout` in seconds. |
| `get_scan_results` | Return a previous scan by `scan_id`. |
| `list_active_scans` | List the scans that are running now. |

## Example

```json
{ "tool": "whatweb_scan", "arguments": { "target": "https://example.com", "aggression": "normal" } }
```

Illustrative output:

```json
{
  "scan_id": "a1b2c3d4",
  "target": "https://example.com",
  "status": "completed",
  "http_status": 200,
  "stats": { "total_technologies": 3 },
  "technologies": [
    { "name": "nginx", "version": "1.25.3", "string": null },
    { "name": "WordPress", "version": "6.4.3", "string": "wp-content" },
    { "name": "jQuery", "version": "3.7.1", "string": "jquery.js" }
  ],
  "error": null
}
```

Aggression levels map to WhatWeb's `-a` flag: `stealthy`/`passive` (1, one request), `polite` (2), `normal` (3), `aggressive` (4, potentially intrusive, so use only on targets you are authorized to test).

## Configuration

Pass these environment variables with `docker run -e NAME=value`. All are optional.

| Variable | Default | Purpose |
| --- | --- | --- |
| `WHATWEB_TIMEOUT` | `300` | Per-scan timeout in seconds, when the tool call does not set `timeout`. |
| `WHATWEB_MAX_CONCURRENT` | `2` | Maximum number of scans that run at the same time. |
| `WHATWEB_OUTPUT_DIR` | `/app/output` | Directory for WhatWeb JSON logs inside the container. |

## How it works

The container installs WhatWeb from the official upstream repository on an Alpine Ruby base, then runs a Python MCP server (`mcp` SDK, stdio transport). Each scan shells out with an argument array (no shell interpolation), writes WhatWeb's JSON log to an ephemeral `/app/output`, and parses it into typed results. Up to 100 scan results are kept in memory for follow-up questions in the same session. Also installable from source without Docker: `pip install -r requirements.txt && python3 server.py` (requires WhatWeb on your PATH).

## Claude Code plugin

The repo root is a Claude Code plugin. It adds two things:

- The WhatWeb MCP server, pinned to a release image with the resource limits above ([`.mcp.json`](.mcp.json)).
- The [`tech-detector`](skills/tech-detector/SKILL.md) skill. The skill tells Claude when to scan a site, how to call `whatweb_scan`, and how to turn the result into a stack summary for lead qualification or competitor research.

The repo is also its own plugin marketplace. In Claude Code 2.1.275 or later, install the plugin with:

```text
/plugin install whatweb --marketplace devkindhq/whatweb-mcp
```

The plugin will also be in the Claude plugin directory once it is listed.

## Roadmap

- [x] Official MCP Registry listing (`io.github.devkindhq/whatweb-mcp`)
- [ ] Docker MCP Registry listing
- [ ] Batch scanning (multiple targets in one call)
- [ ] Optional CSV/JSON export tool
- [ ] Scheduled re-scans with diffing (alert when a target's stack changes)

## Development

```bash
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt pytest
.venv/bin/python -m pytest
```

The tests do not need the WhatWeb binary. See [CONTRIBUTING.md](CONTRIBUTING.md) for checks, commit style and the release flow.

## Built by

Built by [Devkind](https://devkind.com.au). Maintained by Kazim Ali.

## License

The code in this repo is [MIT](LICENSE). See [CONTRIBUTING.md](CONTRIBUTING.md) and [SECURITY.md](SECURITY.md).

The Docker image also installs [WhatWeb](https://github.com/urbanadventurer/WhatWeb) v0.6.4 from upstream, unmodified. WhatWeb is licensed under the [GNU GPL v2](https://github.com/urbanadventurer/WhatWeb/blob/v0.6.4/LICENSE) or later.

Scan only targets you are authorized to scan.

## Qualifying leads by their tech stack?

Once you know what a site runs, Devkind can help you build on it. We work with [Shopify](https://devkind.com.au/services/shopify), [Swell](https://devkind.com.au/services/swell) and [headless commerce](https://devkind.com.au/services/headless-ecommerce) stacks, and build custom MCP servers and AI agent workflows. [Talk to Devkind](https://devkind.com.au/services) or email [hello@devkind.com.au](mailto:hello@devkind.com.au).
