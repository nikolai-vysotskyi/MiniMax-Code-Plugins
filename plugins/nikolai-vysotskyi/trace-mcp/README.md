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
- `npx` on `PATH`. `mcp.json` starts the server as `npx -y trace-mcp@3.31.5` with no other arguments, so the exact validated version is resolved from the npm registry on every launch: a stale global install or any other same-named `PATH` entry is never consulted, and npm verifies the tarball integrity on download. Every trace-mcp release additionally ships with Sigstore provenance (`npm audit signatures`). Offline fallback: a pre-installed `trace-mcp` binary works if `trace-mcp --version` prints 3.31.5 or newer — minus the registry guarantee above, so prefer the `npx` form.
- macOS, Linux, or Windows.
- No account, no paid service, no API key.

## Data and network (default configuration)

- The code index is built and kept on the user's machine. With default settings, source code never leaves it.
- Two opt-in features change that, and neither is enabled by default: cloud embedding providers sit behind an explicit consent gate (`~/.trace/consent.json`, granted per provider) and send code excerpts to the configured provider; OTLP/Langfuse export sends spans to a backend you configure. Enabling either is a deliberate step outside this package's defaults.
- At most one anonymous usage ping per day, sent to Google's GA4 Measurement Protocol endpoint: a persistent locally-generated UUID (stored in `~/.trace/telemetry-state.json`), the trace-mcp version and previous version, install/upgrade signal, Node major, OS platform, timezone country, MCP client name and the model it mostly drove, number of indexed repositories, machine class (arch, cores, RAM in whole GB, kernel version), tool preset and advertised tool count, aggregate tool-call/saved-token deltas, and daemon start/crash counters. No code, no paths, no file names, no query content, no IP. Full field list: https://trace-mcp.com/privacy.html. Turn it off with `TRACE_MCP_TELEMETRY=off`, or with `"telemetry": { "usage_ping": false }` in `~/.trace/.config.json`. Suppressed automatically in CI.
- No other network access with default settings. No credentials in the package.

## Writes

Six of the server's tools modify the user's local checkout; everything else is read-only. All six preview with `dry_run: true` by default and write nothing until re-called with `dry_run: false`:

- `apply_rename` — renames a definition plus every reference in one operation, after collision detection. Past 20 files the apply fails closed (`success: false`, `Rename affects N files (>20). Pass confirm_large: true to proceed.`) and returns the preview so the call can be re-issued deliberately.
- `apply_codemod` — pattern rewrites (AST-aware on TypeScript/JavaScript, regex fallback elsewhere). Same >20-file `confirm_large` gate as rename, same fail-closed shape.
- `extract_function` — extracts a line range into a named helper (single file, TypeScript/JavaScript). Rejects multi-return slices with a structured error instead of guessing; lowers `confidence` on shadowed-variable cases.
- `apply_move` — moves a symbol between files, or renames/moves a file, updating imports.
- `change_signature` — adds, removes, renames, or reorders parameters and updates call sites.
- `remove_dead_code` — deletes one symbol after verifying it is dead (multi-signal detection or zero incoming edges), and warns about orphaned imports.

`plan_refactoring` previews any rename/move/extract/signature change without touching files — the read-only way to review blast radius first.

Two limits, stated plainly: applied edits are not rolled back automatically (a failed type-check after the fact is reported, not reverted — review the preview, or version-control the checkout), and every file argument is confined to the indexed project root (out-of-root paths are rejected before any write; writes addressed through symlinks pointing outside the root are refused at write time — verified against the pinned 3.31.5).

The refactoring and codemod skills need these tools visible: under the server's default preset they are hidden (`Tool "apply_rename" is not available in this session's tool preset`), so load the `dev` preset first (`load_tools`).

## Skills and MCP

Skills (each directory matches its frontmatter `name`): `trace-mcp` (routing: call trace-mcp instead of reading files when exploring a codebase), `trace-mcp-refactoring` (risk assessment and cross-file renames), `trace-mcp-codemod` (bulk mechanical edits), `trace-mcp-pre-commit` (security, quality-gate, and antipattern checks before committing).

MCP: one stdio server, `trace-mcp`. Upstream counts: 182 tools, 81 languages, 88 framework integrations.

## License

MIT. See [LICENSE](LICENSE).
