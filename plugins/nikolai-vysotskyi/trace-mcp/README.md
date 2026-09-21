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
- The `trace-mcp` executable on `PATH`, version 3.28.0 or newer (`npm install -g trace-mcp@3.28.0` — this package was validated against 3.28.0). `mcp.json` starts it as a stdio server with no arguments. Note the host cannot verify which binary answers to a bare `trace-mcp` name: a stale global install or any other same-named `PATH` entry wins silently, so keep the pinned version installed and, if your setup allows it, check the install with `npm audit signatures` (every trace-mcp release ships with Sigstore provenance).
- macOS, Linux, or Windows.
- No account, no paid service, no API key.

## Data and network (default configuration)

- The code index is built and kept on the user's machine. With default settings, source code never leaves it.
- Two opt-in features change that, and neither is enabled by default: cloud embedding providers sit behind an explicit consent gate (`~/.trace/consent.json`, granted per provider) and send code excerpts to the configured provider; OTLP/Langfuse export sends spans to a backend you configure. Enabling either is a deliberate step outside this package's defaults.
- At most one anonymous usage ping per day, sent to Google's GA4 Measurement Protocol endpoint: a persistent locally-generated UUID (stored in `~/.trace/telemetry-state.json`), the trace-mcp version and previous version, install/upgrade signal, Node major, OS platform, timezone country, MCP client name and the model it mostly drove, number of indexed repositories, machine class (arch, cores, RAM in whole GB, kernel version), tool preset and advertised tool count, aggregate tool-call/saved-token deltas, and daemon start/crash counters. No code, no paths, no file names, no query content, no IP. Full field list: https://trace-mcp.com/privacy.html. Turn it off with `TRACE_MCP_TELEMETRY=off`, or with `"telemetry": { "usage_ping": false }` in `~/.trace/.config.json`. Suppressed automatically in CI.
- No other network access with default settings. No credentials in the package.

## Writes

Two of the server's tools modify the user's local checkout, nothing else: `apply_rename` rewrites a definition plus every reference in one operation, and `apply_codemod` applies pattern rewrites with a dry-run preview as the default — applying requires `dry_run: false`, and changes touching more than 20 files additionally require `confirm_large: true`.

## Skills and MCP

Skills (each directory matches its frontmatter `name`): `trace-mcp` (routing: call trace-mcp instead of reading files when exploring a codebase), `trace-mcp-refactoring` (risk assessment and cross-file renames), `trace-mcp-codemod` (bulk mechanical edits), `trace-mcp-pre-commit` (security, quality-gate, and antipattern checks before committing).

MCP: one stdio server, `trace-mcp`. Upstream counts: 182 tools, 81 languages, 88 framework integrations.

## License

MIT. See [LICENSE](LICENSE).
