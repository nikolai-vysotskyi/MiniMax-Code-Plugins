# trace-mcp

Code intelligence for MiniMax Code: one MCP tool call returns callers, callees, and framework edges across the repo, so the agent stops reading files one by one to answer "what uses this".

Upstream source: https://github.com/nikolai-vysotskyi/trace-mcp

## Try it

```text
Find every caller of `validateHostedPluginDirectory` in this repo and tell me what breaks if I change its signature.
```

Expected result: the agent calls the trace-mcp search and impact tools and answers with the call sites plus the blast radius, without opening each file. On an unindexed project the agent runs `trace init` once first, then answers the same way.

## Requirements

- Node.js 22 or newer on `PATH`.
- The `trace-mcp` executable on `PATH` (`npm install -g trace-mcp`). `mcp.json` starts it as a stdio server with no arguments.
- macOS, Linux, or Windows.
- No account, no paid service, no API key.

## Data and network

- The code index is built and kept on the user's machine. Source code never leaves it.
- At most one anonymous usage ping per day (version, OS, MCP client, aggregate counts; no code, no paths, no per-install identifier beyond a locally generated UUID). Turn it off with `TRACE_MCP_TELEMETRY=off`, or with `"telemetry": { "usage_ping": false }` in `~/.trace/.config.json`.
- No other network access. No credentials in the package.

## Skills and MCP

Skills (each directory matches its frontmatter `name`): `trace-mcp` (routing: call trace-mcp instead of reading files when exploring a codebase), `trace-mcp-refactoring` (risk assessment and cross-file renames), `trace-mcp-codemod` (bulk mechanical edits), `trace-mcp-pre-commit` (security, quality-gate, and antipattern checks before committing).

MCP: one stdio server, `trace-mcp`. Upstream counts: 182 tools, 81 languages, 88 framework integrations.

## License

MIT. See [LICENSE](LICENSE).
