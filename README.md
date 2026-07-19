# creator-tools

Vendored MCP servers + Creator-specific skills, for [Creator](https://github.com/RaccoonzinhoCo/Creator) (an autonomous app/book/game generator) to configure and run.

## What's here

### `mcp_servers/`

Eight MCP servers, vendored (full source copy, not a submodule/reference) so Creator's "sync MCP server library" action can pull a known-good local copy without depending on npm/PyPI registry availability at runtime. Six from the official [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) repository, plus two chosen specifically to close gaps in Creator's own generation quality:

| Server | Source | Runtime | Run without a build step? |
|---|---|---|---|
| `fetch` | modelcontextprotocol/servers | Python (`uv`) | Yes — `uv run --directory mcp_servers/fetch mcp-server-fetch` |
| `git` | modelcontextprotocol/servers | Python (`uv`) | Yes — `uv run --directory mcp_servers/git mcp-server-git` |
| `time` | modelcontextprotocol/servers | Python (`uv`) | Yes — `uv run --directory mcp_servers/time mcp-server-time` |
| `filesystem` | modelcontextprotocol/servers | TypeScript (Node) | No — `npm install && npm run build` first |
| `memory` | modelcontextprotocol/servers | TypeScript (Node) | No — `npm install && npm run build` first |
| `sequentialthinking` | modelcontextprotocol/servers | TypeScript (Node) | No — `npm install && npm run build` first |
| `context7` | [upstash/context7](https://github.com/upstash/context7) (`packages/mcp`) | TypeScript (Node) | No — `npm install && npm run build` first. Live, current documentation lookup for any library/package — directly targets hallucinated/stale API calls in generated Flutter and Godot code. |
| `playwright` | [microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp) | TypeScript (Node) | No — `npm install && npm run build` first. Real browser automation — lets generation/review actually load and inspect a generated Flutter web build instead of guessing whether it renders correctly. |

Each server folder contains a `.vendored-from-source` (the upstream URL) and `.vendored-from-commit` (the exact upstream commit this copy was taken from) — see [Staying in sync](#staying-in-sync).

### `skills/`

Plain-markdown skill files Creator injects into a generation prompt right before writing a file of the matching type — see each `SKILL.md` for what it covers. `godot_gdscript`, `flutter_widgets`, and `book_prose` also live (byte-identical) in the main [Creator](https://github.com/RaccoonzinhoCo/Creator) repo's `library_packages/` — that's the checksum-tested copy Update Library actually serves and the one bundled into the app for offline-from-first-launch use; this repo is where they're authored/reviewed.

Current skills: `godot_gdscript`, `godot_sprite_animation`, `godot_game_mechanics`, `godot_ui_ux` (game craft); `flutter_widgets`, `flutter_state_management`, `flutter_theming_design`, `flutter_animations`, `flutter_testing` (Flutter craft); `book_prose`, `genre_conventions`, `prose_rhythm_pacing` (book craft); `web_development` (authored ahead of a website/SaaS pipeline existing — not yet wired into any Creator generation call).

## Staying in sync

True real-time sync with upstream isn't possible — GitHub doesn't let an external repo subscribe to another repo's pushes without the upstream owner configuring that themselves, and this repo has no such arrangement with any of the upstream sources above.

Instead, [`.github/workflows/sync-upstream.yml`](.github/workflows/sync-upstream.yml) runs daily (and can be triggered manually from the Actions tab) and, for each vendored server:
1. Shallow-clones that server's own upstream repo.
2. Diffs the vendored folder here against the upstream copy.
3. If anything changed, copies the new files in, updates that server's `.vendored-from-commit`, and opens a PR (never pushes directly to `main`) so a human reviews upstream changes before they land.

## License

The vendored `mcp_servers/` content is upstream's own code, under the license terms in [`UPSTREAM_LICENSE.md`](UPSTREAM_LICENSE.md) (MIT/Apache-2.0, per [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)) — see that file and each server's own `LICENSE`/`pyproject.toml`/`package.json` for specifics. `skills/` content is original, written for Creator.
