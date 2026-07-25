# OpenReel Studio Agent Plugin

English · [简体中文](./README.md)

This optional integration lets an external host agent analyze and orchestrate
work while the MCP bridge performs OpenReel project, canvas, node, edge,
upload, asset-library, and node-run operations. OpenReel's built-in agent and
chat surface remain available as an independent path.

## Quick start

1. Start an OpenReel desktop or source installation.
2. Install the complete plugin in Codex, or configure the local stdio MCP
   bridge in another client.
3. Ask the agent to connect, list projects, and inspect the current canvas.
4. The agent verifies the service, selects a project, reads the minimum state,
   and performs the requested operation.

An installed desktop build starts FastAPI on `127.0.0.1` using a dynamic port
beginning at `7860`. The bridge scans supported local ports and accepts only a
health response whose application identity is `openreel-studio`. Source builds
use the same discovery path. No connection variables are required for an
unauthenticated local service.

## Client setup

Codex uses the complete marketplace installation:

```bash
codex plugin marketplace add https://github.com/yutianxiao6/openreel-agent-plugin.git
codex plugin add openreel-studio@openreel-agent
```

Claude Code, Cursor, Gemini CLI, Windsurf, and other clients that accept an
`mcpServers` root can clone the repository and configure the local bridge:

```json
{
  "mcpServers": {
    "openreel-studio": {
      "command": "node",
      "args": [
        "/absolute/path/openreel-agent-plugin/plugins/openreel-studio/scripts/openreel-mcp.mjs",
        "--stdio"
      ]
    }
  }
}
```

VS Code uses a `servers` root in `.vscode/mcp.json`:

```json
{
  "servers": {
    "openreel-studio": {
      "type": "stdio",
      "command": "node",
      "args": [
        "/absolute/path/openreel-agent-plugin/plugins/openreel-studio/scripts/openreel-mcp.mjs",
        "--stdio"
      ]
    }
  }
}
```

A direct MCP connection receives the same tools but does not automatically
load the Codex `openreel-director` skill. `.codex-plugin` is the installation
format required by Codex; it is not the product name or a client restriction
on the underlying MCP bridge.

## Docker and remote deployments

Set the service address and matching credentials before starting the agent
client:

```bash
export OPENREEL_BASE_URL="https://example.com/studio"
export OPENREEL_USERNAME="your-user"
export OPENREEL_PASSWORD="your-password"
```

For bearer authentication:

```bash
export OPENREEL_BASE_URL="https://example.com/studio"
export OPENREEL_TOKEN="your-token"
```

`OPENREEL_BASE_URL` can point to the site root or the `/studio` root. After a
successful health check, the bridge stores the complete connection profile in
the current user's private configuration:

- Linux: `~/.config/openreel-agent-plugin/connection.json`
- macOS: `~/Library/Application Support/OpenReel/agent-connection.json`
- Windows: `%APPDATA%\OpenReel\agent-connection.json`

Linux and macOS files use mode `0600`; Windows uses the current user's
application-data access controls. Tool results expose only the configuration
source, persistence status, and authentication type. A newly verified explicit
profile replaces the previous profile as one unit, so an address and its
credentials cannot be mixed across services.

Profiles saved before the rename under `openreel-codex-plugin` or
`codex-connection.json` are still read for compatibility. Newly saved profiles
use the Agent Plugin paths.

An unauthenticated remote service needs only `OPENREEL_BASE_URL`. Advanced
variables:

- `OPENREEL_DISCOVERY_PORTS`: local scan range, default
  `7860-7920,8000-8020`.
- `OPENREEL_DISCOVERY_TIMEOUT_MS`: timeout for one probe, default 15 seconds,
  configurable from 1 to 60 seconds.
- `OPENREEL_REQUEST_TIMEOUT_MS`: timeout for one request, default 20 minutes.
- `OPENREEL_REMEMBER_CONNECTION=0`: use explicit configuration for this
  process without persisting it.
- `OPENREEL_CONFIG_PATH`: choose a managed connection-file path.

Clear saved connection profiles:

```bash
node scripts/openreel-mcp.mjs --forget-connection
```

Check connectivity:

```bash
node scripts/openreel-mcp.mjs --check
```

## Project selection

The server connection and selected project are separate state. Connection
configuration persists across sessions; project selection belongs to the
current agent process.

1. `openreel_list_projects` returns session identities, titles, and
   `_agent_selected`. `_codex_selected` remains only as a legacy field.
2. `openreel_select_project` accepts an exact UUID or a unique exact title.
3. Project-scoped tools default to the selected project.
4. `openreel_create_project` creates and selects a project, then asks an
   already-open OpenReel page to navigate and reload.
5. `openreel_update_project` renames a session; deleting the current project
   clears selection.

The plugin selection determines the agent's operation target. The visible
OpenReel browser or desktop window remains controllable from the left project
rail.

## Call sequence

Use this sequence in each new agent session:

1. Run `openreel_connection_info`.
2. Run `openreel_list_projects`, then select the exact target when needed.
3. Read `openreel_get_canvas` once for graph-level work, or use
   `openreel_get_nodes` for known nodes.
4. Use direct tools for common project, node, edge, asset, upload, and single
   node-run operations.
5. Use `search → describe → execute` for dynamic, complex, or uncommon
   operations.
6. Treat complete persistence results and terminal runs as verification. Read
   the canvas again only when a later operation depends on the new topology.

## Capability layers

Direct tools cover:

- Connection and project list, selection, creation, reading, rename, and
  authorized deletion.
- Canvas and node reads; node creation, update, movement, and authorized
  deletion.
- Edge creation, update, and authorized deletion.
- Single-node execution, event-driven terminal waiting, media upload to an
  existing node, and host-generated image publication.
- Asset-library query by character, scene, storyboard, category, or keyword;
  saving a local file or existing node into the library.
- Search, description, normal execution, and authorized destructive execution
  for deferred capabilities.

The deferred catalog covers uncommon operations such as node copying, media
history selection, mixed canvas patches, batch runs, snapshot restoration, and
batch deletion or restoration. Search returns compact summaries. Describe
returns the exact input schema, safety class, executor, and `schema_ref`.
Execution validates input against that schema.

## Asset library

`openreel_list_assets` filters the selected project's reusable library by
`kind`, category, plain text, or regular expression, with `offset` / `limit`
pagination. Returned `path` values can be placed directly in node
`fields.references`.

`openreel_upload_asset` accepts exactly one source:

- `file_path`: upload a local file readable by the agent host, then save it to
  the library.
- `node_id`: save an existing canvas result to the library.

Choose `kind=character|scene|storyboard`. `category` preserves the user's own
language and defaults to uncategorized.

## Host image generation

When the host has a local image-generation tool and the user has not selected
an OpenReel provider:

1. Generate one final local file with the host tool.
2. Call `openreel_publish_generated_image` to import it as one complete image
   node.

The publish tool records `generation_backend=agent_host` by default; a host can
provide a more specific backend identifier when one is available. A client
should publish only when it can produce a local file readable by the MCP host.
Otherwise, read the dynamic image-node contract and use the OpenReel
`create → run` path. The host-image call audit is documented in
[`docs/image-generation-call-analysis.md`](docs/image-generation-call-analysis.md).

## Video generation

Video always uses the OpenReel dynamic node contract. The bridge reads the
available provider, mode, reference limits, duration, aspect ratio, resolution,
and native-audio capability, then creates and runs the node.

Plugin-created video defaults explicitly to `720p`. Use another resolution
only when the user asks for it. `generate_audio` is valid only when the selected
target declares `supports_native_audio=true`; omit it otherwise.

The bridge does not assemble provider requests, upload upstream references,
poll provider tasks, or parse provider responses. OpenReel delegates those
operations to Universal Model Adapter. Put each media source once in
`fields.references`; do not duplicate it into `reference_images` or
`depends_on`.

The node-run tool holds one event-driven wait for up to 20 minutes. OpenReel
and UMA perform provider polling, persist the terminal state, and end that
wait. A timeout means only that the wait did not see a terminal event; the task
may still be running. Continue with `openreel_wait_for_node` on the same node
instead of starting a duplicate billable run.

## Safety boundaries

- Project deletion requires explicit authorization and a `confirm_title` that
  exactly matches the current title.
- Node and edge deletion requires explicit authorization and `confirm=true`.
- Snapshot restoration and composite destructive operations use the
  destructive executor.
- Authentication data stays in the user's private connection file; tool
  results return only safe metadata.
- The plugin calls atomic OpenReel APIs. OpenReel `/api/chat` remains the
  independent built-in-agent path.
