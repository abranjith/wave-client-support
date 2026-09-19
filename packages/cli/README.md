# `wvc` — Wave Client CLI

[![npm](https://img.shields.io/npm/v/@abranjith/wave-client-cli.svg)](https://www.npmjs.com/package/@abranjith/wave-client-cli)
[![node](https://img.shields.io/node/v/@abranjith/wave-client-cli.svg)](https://nodejs.org)

A cross-platform command-line client for the [Wave Client](../../README.md) workspace. It reads the
same on-disk workspace (`WAVE_DATA_DIR`, default `~/.waveclient`) as the VS Code extension and the
web app — no server, no UI.

Built for humans (colors, spinners, examples in `--help`) and for agents (`--json` output, stable
exit codes).

> **Full documentation:** [docs/platforms/cli.md](../../docs/platforms/cli.md) — install, the complete
> command reference, and the agent guide (`--json` shapes + exit codes). Agents should start with
> `wvc docs --json`.

## Install

```bash
npm install -g @abranjith/wave-client-cli
```

## Quick start

```bash
# Send a saved request by name (or id, or unique prefix)
wvc send "Get User"

# Scope to a collection, show response headers
wvc send "Get User" -c "Users API" -i

# Machine-readable output for scripts and agents
wvc send "Get User" --json | jq .status
```

## Ad-hoc requests

`wvc send` has two modes. A target that starts with `http://` or `https://` is sent as a one-off
URL request; anything else is resolved as a saved request reference.

```bash
# Plain GET
wvc send https://api.example.com/users

# JSON POST
wvc send https://api.example.com/users -d '{"name":"Ada"}' --data-json

# Multipart form upload
wvc send https://api.example.com/upload -F note=hello -F file=@report.pdf

# Environment variables and query params
wvc send https://{{host}}/users --query page=1 -e staging

# Agent-friendly output
wvc send https://api.example.com/users --json | jq .status
```

Ad-hoc URL mode still uses Wave's environment variable resolution, saved auth profiles
(`--auth <ref|none>`), and saved proxy/certificate profiles (`--proxy <ref|off>`,
`--cert <ref|off>`). Inline Basic auth is available with `-u user:pass` and is never saved.
When a process is spawned with a pre-tokenized argv (task runners, CI configs,
programmatic invocation) that preserves literal outer single quotes, `--data-json`
accepts and strips them, so a `-d '{"value":"test"}' --data-json` argument still sends
`{"value":"test"}`. This does **not** help when typing the same command directly into
`cmd.exe` or PowerShell: both strip embedded `"` characters from arguments before Node
sees them, corrupting the JSON no matter how it's quoted. On those shells, use
`-d @file.json` or pipe the body via `-d @-` instead (see below).

## Read surface

```bash
wvc collection list --json
wvc collection show "Users API" --tree
wvc request show "Get User" -c "Users API"
wvc search "post users" --limit 5 --json
wvc env show staging --json
wvc store auth show "Token" --json
wvc settings get requestTimeoutSeconds
wvc history list --limit 1 --json
```

Environment secrets are masked unless `env show --reveal` is passed. Wave Store auth, proxy, and
certificate profile credentials are always masked, including in `--json`.

## Runners and reports

Run saved collections, flows, and test suites headlessly from the terminal:

```bash
wvc collection run "Users API" --folder Admin
wvc collection run "Users API" --request "Get User" --json | jq .progress
wvc flow run "Login flow" --sequential
wvc suite run "Smoke suite"
wvc collection run "Users API" --report
wvc suite run "Smoke suite" --report reports/smoke.html
```

Run progress is written to stderr and summaries are written to stdout, so shell pipelines can keep
the data channel clean. A run with any failed request, flow node, suite item, or failed validation
exits `3`. `--report [file]` writes the same self-contained HTML report built by Wave Client's UI
report builders; omitting the file uses the standard `wave-...html` report filename in the current
directory.

## Mutating the workspace

Mutation commands use the same shared Wave services as the UI, so saved files keep the normal Wave
schema versions, ids, and validation behavior.

```bash
wvc collection create "Users API"
wvc request add -c "Users API" --name "Get User" --method GET --url https://api.example.com/users/1
wvc request edit "Get User" -c "Users API" --url https://api.example.com/users/42
wvc collection rename "Users API" "Public Users API"
wvc collection delete "Public Users API" --yes
```

Structured payloads accept exactly one input channel: `--file <path>`, `--stdin`, or `--edit`.
`--edit` opens `$VISUAL`, then `$EDITOR`, then `notepad` on Windows or `vi` elsewhere. Agents and
CI should prefer `--file` or `--stdin`; destructive commands should pass `--yes` in non-interactive
runs.

```bash
cat flow.json | wvc flow create --stdin --json
wvc flow edit "Login flow" --edit
wvc suite create --file suite.json
wvc suite add-test "Smoke suite" --request "Get User"
wvc suite remove-test "Smoke suite" "Get User" --yes
```

Collection import/export supports Wave JSON, Postman JSON, OpenAPI/Swagger, and HTTP files:

```bash
wvc collection import ./postman.json --format postman
wvc collection import ./openapi.yaml --name "Public API"
wvc collection export "Users API" --format postman -o users.postman_collection.json
```

Environment and settings mutations:

```bash
wvc env create staging
wvc env set staging API_URL=https://api.example.com
wvc env set staging API_KEY=abc --secret
wvc env unset staging API_KEY
wvc env export -o envs.json
wvc settings set requestTimeoutSeconds 5
```

`env show` masks secret variables by default; pass `--reveal` only when you explicitly need the
plain value in your terminal or script.

## For AI agents

Use `wvc docs --json` as the machine-readable command reference. It walks the registered command
tree and returns each command's name, summary, usage, flags, examples, and subcommands without
touching the workspace, so it is safe to run with a missing or placeholder `WAVE_DATA_DIR`.

## Exit codes

| Code | Meaning |
|------|---------|
| `0`  | Success |
| `1`  | Internal error |
| `2`  | Usage / invalid input |
| `3`  | Execution failure (transport error, validation failed, `--fail` with HTTP ≥ 400) |
| `4`  | Not found / ambiguous reference |

## Development

```bash
pnpm --filter @abranjith/wave-client-cli build   # bundle -> dist/wvc.js
pnpm --filter @abranjith/wave-client-cli test    # unit + integration tests
node packages/cli/dist/wvc.js --version
```

The build is a single-file esbuild ESM bundle: `@wave-client/core/headless` and
`@wave-client/shared` are inlined, third-party dependencies stay external. The build fails if
React-family packages appear in the runtime external list or if any external dependency is missing
from `dependencies`.
