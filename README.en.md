# OpenReel Agent Plugin

English · [简体中文](./README.md)

Connect an external agent directly to a running
[OpenReel Studio](https://github.com/yutianxiao6/openreel-studio).
Projects, nodes, edges, and reusable assets use direct tools, while complex or
uncommon canvas operations are loaded through
`search → describe → execute`.

This repository contains a general local stdio MCP bridge plus the manifest
and operating skill used for a complete Codex installation. Codex can install
the bundle from a marketplace. Claude Code, Cursor, VS Code/Copilot, Gemini
CLI, Windsurf, and other stdio MCP-capable clients can configure the same
bridge directly. OpenReel's built-in agent remains an independent path.

## Complete Codex installation

Add this repository as a Codex marketplace, then install the plugin:

```bash
codex plugin marketplace add https://github.com/yutianxiao6/openreel-agent-plugin.git
codex plugin add openreel-studio@openreel-agent
```

Start a new Codex session after installation or an update so it loads the
latest tools and skill.

## Other agents

Clone this repository and configure
`plugins/openreel-studio/scripts/openreel-mcp.mjs --stdio` as a local MCP
server. Other clients receive the same OpenReel tools but do not automatically
install the Codex `.codex-plugin` manifest or skill. See the
[plugin guide](plugins/openreel-studio/README.en.md) for configuration examples
and compatibility limits.

## Use

Start OpenReel Studio, then make the connection request explicit:

> Connect to my local OpenReel, list projects, select “Product Demo,” and
> summarize the current canvas.

Installed desktop and source builds are discovered on verified localhost
ports. Docker and remote deployments use an explicit address and optional
authentication. A verified connection is saved in the current user's private
configuration. Projects can be switched within an agent session, and a newly
created project becomes the current target automatically.

## Repository layout

```text
.agents/plugins/marketplace.json       Agent plugin marketplace
plugins/openreel-studio/               OpenReel Agent plugin
  .codex-plugin/plugin.json            Codex installation manifest
  .mcp.json                            MCP server entry
  scripts/openreel-mcp.mjs             OpenReel control bridge
  skills/openreel-director/SKILL.md    Codex operating guidance
```

The public name is **OpenReel Agent Plugin**. `.codex-plugin` is the
installation format required by Codex; it does not limit the underlying MCP
tools to Codex.

## Local verification

```bash
node --check plugins/openreel-studio/scripts/openreel-mcp.mjs
node --test plugins/openreel-studio/tests/*.test.mjs
node plugins/openreel-studio/scripts/openreel-mcp.mjs --check
```

## License

MIT
