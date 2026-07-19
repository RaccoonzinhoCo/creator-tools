# creator-tools

Vendored MCP servers + Creator-specific skills, for [Creator](https://github.com/RaccoonzinhoCo/Creator) (an autonomous app/book/game generator) to configure and run.

## What's here

### `mcp_servers/`

Six MCP servers, vendored (full source copy, not a submodule/reference) from the official [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) repository, so Creator's "sync MCP server library" action can pull a known-good local copy without depending on npm/PyPI registry availability at runtime:

| Server | Runtime | Run without a build step? |
|---|---|---|
| `fetch` | Python (`uv`) | Yes — `uv run --directory mcp_servers/fetch mcp-server-fetch` |
| `git` | Python (`uv`) | Yes — `uv run --directory mcp_servers/git mcp-server-git` |
| `time` | Python (`uv`) | Yes — `uv run --directory mcp_servers/time mcp-server-time` |
| `filesystem` | TypeScript (Node) | No — run `npm install && npm run build` in the folder first |
| `memory` | TypeScript (Node) | No — run `npm install && npm run build` in the folder first |
| `sequentialthinking` | TypeScript (Node) | No — run `npm install && npm run build` in the folder first |

Each server folder contains a `.vendored-from-source` (the upstream URL) and `.vendored-from-commit` (the exact upstream commit this copy was taken from) — see [Staying in sync](#staying-in-sync).

### `skills/`

Plain-markdown skill files Creator injects into a generation prompt right before writing a file of the matching type — see each `SKILL.md` for what it covers. `godot_gdscript` lives in the main [Creator](https://github.com/RaccoonzinhoCo/Creator) repo (`library_packages/`) since it predates this repo; new skills land here going forward.

## Staying in sync

True real-time sync with upstream isn't possible — GitHub doesn't let an external repo subscribe to another repo's pushes without the upstream owner configuring that themselves, and this repo has no such arrangement with `modelcontextprotocol/servers`.

Instead, [`.github/workflows/sync-upstream.yml`](.github/workflows/sync-upstream.yml) runs daily (and can be triggered manually from the Actions tab) and:
1. Shallow-clones the current `modelcontextprotocol/servers` `main` branch.
2. Diffs each vendored server folder here against the upstream copy.
3. If anything changed, copies the new files in, updates that server's `.vendored-from-commit`, and opens a PR (never pushes directly to `main`) so a human reviews upstream changes before they land.

## License

The vendored `mcp_servers/` content is upstream's own code, under the license terms in [`UPSTREAM_LICENSE.md`](UPSTREAM_LICENSE.md) (MIT/Apache-2.0, per [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)) — see that file and each server's own `LICENSE`/`pyproject.toml`/`package.json` for specifics. `skills/` content is original, written for Creator.
