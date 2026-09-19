# CLI (`wvc`)

`wvc` is the Wave Client command line — a third platform alongside the [VS Code extension](vscode.md) and the [web app](web-app.md). It reads and writes the **same on-disk workspace** (`~/.waveclient` by default) as the other two, with no server and no UI. It is built to be equally good for humans (colors, spinners, examples in `--help`) and for AI agents and scripts (`--json` output, stable exit codes, a self-documenting `wvc docs` command).

The npm package is [`@abranjith/wave-client-cli`](https://www.npmjs.com/package/@abranjith/wave-client-cli); the installed binary is `wvc`.

> **Agents:** jump to [For AI agents](#for-ai-agents). The single best starting point is `wvc docs --json`.

---

## Install

```bash
npm install -g @abranjith/wave-client-cli
wvc --version
```

Or run it without installing:

```bash
npx @abranjith/wave-client-cli send https://api.example.com/health
```

Requires Node.js **≥ 18.18.0**. The published package is a single self-contained bundle — Wave's own code is inlined and only a handful of third-party runtime dependencies are installed.

---

## Where your data lives

`wvc` operates on the **existing** Wave workspace — the same collections, environments, flows, test suites, store profiles, settings, and history the VS Code extension and web app use. The workspace directory is resolved in this order:

1. `--data-dir <path>` (global flag)
2. `WAVE_DATA_DIR` environment variable
3. the persisted **Data Storage Location** setting (when no explicit override is set)
4. `~/.waveclient` (default)

A relative `--data-dir ./ws` is resolved against your **current working directory**, not your home directory.

> **This precedence applies to `wvc` only.** The VS Code extension and web app always use their configured **Data Storage Location**; a `WAVE_DATA_DIR` in your environment does not redirect them. Headless clients (`wvc`, the web-app server, the MCP server) opt into the override explicitly, so setting the variable for CLI work cannot silently move the GUI's workspace.

An explicit `WAVE_DATA_DIR` always overrides the GUI's Data Storage Location setting. If the resolved directory is missing or empty, human-mode commands print a warning naming its absolute path instead of silently implying the workspace has no data.

> **Plain-JSON only (encrypted workspaces).** Like the MCP server and the HTTP server, `wvc` reads and writes the workspace as **plain JSON and never decrypts**. If a workspace was encrypted in the VS Code extension, `wvc` cannot read those files and falls back to defaults (an empty-looking workspace) rather than failing. Use `wvc` against an unencrypted workspace. See [Settings → Security](../features/settings.md#security-settings).
>
> **Encrypted files are never modified.** An encrypted collection is invisible to `wvc`, but it is never overwritten by it: the collection fails to load, so references to it cannot resolve (exit `4`), and creating a collection whose filename would collide writes a suffixed file (`secret_api_1.json`) instead of replacing the original. Pointing `wvc` at an encrypted workspace is safe — it just will not show you anything.

---

## Command reference

Every command has `--help` with an **Examples** block, and `wvc docs` prints the whole tree at once. Machine-readable form: `wvc docs --json`.

### Sending requests

```bash
wvc send <request|url> [options]
```

`wvc send` is **dual-mode**. A target beginning with `http://` or `https://` is sent as a one-off **ad-hoc URL**; anything else is resolved as a **saved request** reference (by id, name, or unique prefix).

- **Saved-request flags:** `-c, --collection <ref>` (scope resolution), `--no-validate`, `-i/--include`, `--full`, `-o/--output <path>`, `--fail`.
- **Ad-hoc URL flags:** `-X/--method`, repeatable `-H/--header`, `-d/--data` (`@file` or `@-` for stdin), `--data-json`, repeatable `-F/--form` (`key=value` or `key=@file`), repeatable `--query key=value`, `-u/--user user:pass` (inline Basic auth), `--proxy <ref|off>`, `--cert <ref|off>`.
- **Shared execution flags** (both modes): `-e/--env`, `--auth`, `--var`, `--timeout`, `--insecure`.

Mixing modes is an error: URL-only flags with a saved-request target, or `-c`/`--no-validate` with a URL target, exit `2` and name the offending flag. A saved-ref miss that looks like a bare host (e.g. `example.com`) exits `4` with a `did you mean https://example.com?` hint.

An HTTP 4xx/5xx status is a **successful send** (curl semantics) unless `--fail` is passed. Only transport failures and failed validation are execution failures (exit `3`).

`-o/--output <path>` saves the response body instead of printing it. A path that names a directory (trailing slash, or an existing directory) saves into that directory under a filename derived from the response — the `Content-Disposition` header if present, otherwise a timestamped name using the content type's extension (e.g. `response_2026-07-12T16-08-09.png`) — since a response is new data, not an update to a file whose name you already know. Any other path is used as the exact file to write and is overwritten if it already exists.

```bash
wvc send "Get User"                                   # saved, by name
wvc send "Get User" -c "Users API" -i                 # scoped, show headers
wvc send https://api.example.com/users --json         # ad-hoc URL
wvc send https://api.example.com/users -d '{"n":1}' --data-json
wvc send https://api.example.com/upload -F file=@report.pdf
```

### Collections and requests

```bash
wvc collection list [--json]
wvc collection show <ref> [--tree]
wvc collection create <name>
wvc collection rename <ref> <newName>
wvc collection delete <ref> [--yes]
wvc collection import <file> [--format wave|postman|openapi|http] [--name <name>]
wvc collection export <ref> [--format wave|postman] [-o <file>]
wvc collection run <ref> [--folder <path>] [--request <ref>]... [--concurrency <n>] [--delay <ms>] [--stop-on-failure] [--report [file]]

wvc request show <ref> [-c <coll>]
wvc request add [-c <coll>] (flags or JSON)
wvc request edit <ref> [-c <coll>] (flags or JSON)
wvc request delete <ref> [-c <coll>] [--yes]

wvc search <query> [-c <coll>] [--limit <n>]
```

Collection references also match the file name (with or without `.json`) and the collection's `waveId`. Import auto-detects the format from content when `--format` is omitted (Wave, Postman, OpenAPI/Swagger, or `.http`). Export emits Wave or Postman JSON to stdout or `-o`.

### Flows

```bash
wvc flow list [--json]
wvc flow show <ref>
wvc flow run <ref> [--sequential] [--report [file]]
wvc flow create (--file|--stdin|--edit)
wvc flow edit <ref> (--file|--stdin|--edit)
wvc flow delete <ref> [--yes]
```

Flows run in parallel by default; `--sequential` opts into ordered execution. Nodes missing canvas coordinates get a default grid layout before validation.

### Test suites

```bash
wvc suite list [--json]
wvc suite show <ref>
wvc suite run <ref> [--report [file]]
wvc suite create (--file|--stdin|--edit)
wvc suite edit <ref> (--file|--stdin|--edit)
wvc suite delete <ref> [--yes]
wvc suite add-test <suiteRef> (--request <ref> | --flow <ref>)
wvc suite remove-test <suiteRef> <itemRef> [--yes]
wvc suite add-case <itemRef> [-s <suite>] (flags or JSON)
wvc suite edit-case <itemRef> <caseRef> [-s <suite>] (flags or JSON)
wvc suite remove-case <itemRef> <caseRef> [-s <suite>] [--yes]
wvc suite list-cases <itemRef> [-s <suite>] [--json]
```

Test-case commands are **item-anchored**: `<itemRef>` resolves by id, exact name, or unique
substring across every suite. Item ids are globally unique and need no suite reference; pass
`-s/--suite <ref>` only when the same item name appears in multiple suites. `list-cases --json`
emits the complete, lossless case array so an agent can inspect a case, modify it, and send it back
through `edit-case --stdin`.

Simple flags cover variables (`--var` and `--empty-var`) plus request-only headers, query params,
body, and auth. These fields follow the merge/replace behavior in the
[Test Suite Schema Reference](../test-suite-schema.md); use `--file`, `--stdin`, or `--edit` for
full JSON and validation rules. Flow cases accept variable overrides only—request-only fields or
validation fail with exit `2`. Case names must be unique within their item, ignoring case and
surrounding whitespace. Invalid payloads and duplicate names exit `2`; missing or ambiguous
item/case references exit `4`.

### Environments

```bash
wvc env list [--json]
wvc env show <ref> [--reveal]
wvc env create <name>
wvc env delete <ref> [--yes]
wvc env set <ref> <key=value>... [--secret]
wvc env unset <ref> <key>...
wvc env import <file>
wvc env export [-o <file>]
```

`env show` masks `type: secret` variable values unless `--reveal` is passed. `env set` splits on the first `=`, so `env set staging TOKEN=a=b` stores the value `a=b`.

### Wave Store (read-only)

```bash
wvc store auth list [--json]
wvc store auth show <ref>
wvc store proxy list [--json]
wvc store proxy show <ref>
wvc store cert list [--json]
wvc store cert show <ref>
```

Store profiles are **read-only** in the CLI (create/edit them in the VS Code extension or web app). Credential material — auth secrets, proxy passwords, certificate keys — is **always masked**, including under `--json`. There is no `--reveal` for store profiles.

### Settings and history

```bash
wvc settings list [--json]
wvc settings get <key>
wvc settings set <key> <value>
wvc history list [--limit <n>]
wvc history show <id>
```

`settings set` updates existing dot-path keys only, coercing the value to the current setting's type. An unknown key or an invalid type exits `2`. Credential-like settings are always masked by `settings list` and `settings get`, including with `--json`; there is no reveal override. `encryptionKeyEnvVar` remains visible because it is only an environment-variable name.

### Self-documentation

```bash
wvc docs           # human-readable command reference
wvc docs --json    # machine-readable command tree (agent entry point)
```

`wvc docs` and every `--help` are pure — they **never touch the workspace**, so they are safe to run with a missing, placeholder, or encrypted `WAVE_DATA_DIR`.

---

## Global flags

Available on every command:

| Flag | Meaning |
| --- | --- |
| `--data-dir <path>` | Workspace directory (overrides `WAVE_DATA_DIR` and the `~/.waveclient` default) |
| `--json` | Emit a single JSON document on stdout; implies `--quiet` and `--no-color` |
| `-q, --quiet` | Suppress non-essential decoration on stderr |
| `--verbose` | Print diagnostics to stderr (data dir, resolved ids, timing — never secrets) |
| `--no-color` | Disable ANSI color |
| `-V, --version` | Print the version and exit |
| `-h, --help` | Show help (with examples) for any command |

## Shared execution flags

Accepted by `send` and the `run` commands:

| Flag | Meaning |
| --- | --- |
| `-e, --env <nameOrId>` | Environment to resolve variables against (`global` for none) |
| `--auth <nameOrId\|none>` | Auth profile to use (`none` sends unauthenticated) |
| `--var <key=value>` | Set/override a variable (repeatable) |
| `--timeout <seconds>` | Per-request timeout (`0` = no timeout) |
| `--insecure` | Skip TLS certificate validation |
| `-o, --output <path>` | Write the response body to a file, or into a directory (trailing slash, or an existing directory) under a filename derived from the response |
| `--report [file]` | (`run` only) Write an HTML run report; default filename when no value |

---

## Streams and exit codes

**Stream discipline.** Command **data** goes to **stdout**; everything else — status lines, spinners, progress, validation `✓`/`✗` lines, prompts, diagnostics — goes to **stderr**. So `wvc send X > body.json` yields exactly the response body, and a run pipeline keeps its data channel clean.

**Exit codes** are a stable contract:

| Code | Meaning |
| --- | --- |
| `0` | Success |
| `1` | Internal error |
| `2` | Usage / invalid input (bad flags, wrong mode, unknown setting key, invalid JSON payload) |
| `3` | Execution failure — transport error, failed validation, run failures, or `--fail` with HTTP ≥ 400 |
| `4` | Not found / ambiguous reference (with candidates listed) |

---

## For AI agents

**Start here:** `wvc docs --json`. It walks the registered command tree and returns each command's `name`, `summary`, `usage`, `options`, `examples`, and `subcommands` — the full surface, without bootstrapping or touching disk.

### `--json` output shapes

`--json` emits **one JSON document on stdout** with no envelope, and implies `--quiet` + `--no-color`. Bodies are decoded to text when textual, otherwise base64 with `isBase64: true`. Masking (secret env vars, store credentials) is applied to JSON output exactly as it is to human output.

- **`send` (both saved and ad-hoc URL modes share one shape):**
  ```json
  {
    "status": 200,
    "statusText": "OK",
    "headers": { "content-type": "application/json" },
    "body": "{\"id\":1}",
    "elapsedTime": 132,
    "size": 9,
    "validationResult": { "allPassed": true, "results": [] }
  }
  ```
  Binary bodies come back base64-encoded with `"isBase64": true`.
- **`list` commands** (`collection list`, `env list`, `flow list`, …) emit a **JSON array** of objects.
- **`show` commands** emit a single object for the resolved entity.
- **`run` commands** (`collection run`, `flow run`, `suite run`) emit a run-summary document. Collection results include `name`, `method`, and `folderPath`; unsupported WS/SSE entries appear in `skipped` and are included in `progress.total` without causing failure. Suite items include `name`.

  Two invariants make the counters safe to reason about:

  - `progress.completed + progress.skipped === progress.total`. Use that sum — **not `completed` alone** — to decide whether a run finished, since `completed` never reaches `total` when anything was skipped.
  - `progress.skipped === skipped.length`. `progress.skipped` counts only entries the runner never attempted (unsupported protocols), and those are exactly the entries listed in `skipped`. Requests abandoned by `--stop-on-failure` are a different thing: they were selected and attempted-in-principle, so they appear in `results` with `"status": "skipped"` and are counted under `completed`.
- **`search`** results include an actionable collection-item `id`; use `wvc search pet --method POST --json` and pass an id to `wvc send <id> -c <collection>`.

### Error envelope

On failure, `--json` writes an error envelope to **stderr** (stdout stays empty) and sets the exit code:

```json
{ "error": { "code": 4, "message": "Ambiguous reference \"Ambiguous\"", "candidates": ["Users API › Ambiguous", "Orders API › Ambiguous"] } }
```

`code` matches the process exit code. `candidates` appears only for ambiguous references. Under `--json`, stderr is either empty or exactly this one error JSON document; shared-service diagnostics are suppressed unless `--verbose` is also set.

### Non-interactive rules

- Prefer `--file <path>` or `--stdin` over `--edit` for structured mutations — `--edit` needs an interactive editor.
- Pass `--yes` to destructive commands (`delete`, `remove-test`). Without a TTY and without `--yes`, they exit `2` rather than hang on a prompt.
- Rely on **exit codes**, not stderr text, for control flow. `0` = success; treat `3` as an execution failure and `4` as a lookup problem.
- Parse **stdout** only; keep stderr for human/log consumption.

---

## Troubleshooting

### Editor for `--edit` (Windows)

`--edit` opens `$VISUAL`, then `$EDITOR`, then a platform fallback (`notepad` on Windows, `vi` elsewhere). On Windows:

- Point `EDITOR`/`VISUAL` at a real path. Editors shipped as batch shims (`code`, `codium` → `code.cmd`) are launched through `cmd.exe` automatically; `.exe` editors and full paths are launched directly. Quoted paths with arguments work: `set "EDITOR="C:\Program Files\Microsoft VS Code\bin\code.cmd" --wait"`.
- If your editor returns immediately (VS Code, Sublime), add its **wait** flag (`--wait` / `-w`) so `wvc` reads the file after you finish editing, not before.
- Agents and CI should avoid `--edit` entirely — use `--file` or `--stdin`.

### Piping and stdin (PowerShell / cmd)

- Read a body from stdin with `-d @-`:
  - PowerShell: `Get-Content .\body.json -Raw | wvc send https://api.example.com -d @- --data-json`
  - cmd: `type body.json | wvc send https://api.example.com -d @- --data-json`
- Inline `--data-json` also accepts payloads that arrive with literal outer single quotes when a process is spawned with a pre-tokenized argv (task runners, CI configs, programmatic invocation) that preserves them as bytes. This does **not** apply to PowerShell or cmd typed interactively — both strip embedded `"` characters from arguments before Node ever sees them, no matter how the value is quoted, so `-d '{"value":"test"}' --data-json` typed directly at the prompt still fails. Use `-d @file.json` or the `-d @-` stdin form above instead.
- Feed a mutation payload with `--stdin`: `Get-Content .\flow.json -Raw | wvc flow create --stdin --json`.
- Color is disabled automatically when stdout is redirected or piped (non-TTY), and by `NO_COLOR`. A redirected file (`wvc ... > out.json`) never contains ANSI escape codes.

### Colors and terminals

Colors and spinners are verified on Windows Terminal and the VS Code integrated terminal. Force color off with `--no-color` or `NO_COLOR=1`; `--json` and `--quiet` also suppress decoration.

---

## Security notes

- **Settings masking.** `settings list` and `settings get` always mask credential-like values, including under `--json`; there is no reveal override.
- **Headless OAuth.** OAuth2 Authorization Code and PKCE flows cannot be authorized headlessly. Refresh Token and Client Credentials flows work.
- **Protocol runs.** WebSocket and SSE requests are reported as skipped by `collection run`; they do not make an otherwise successful HTTP run fail.

- **No decryption.** `wvc` never decrypts an encrypted workspace; it reads plain JSON only and falls back to defaults otherwise.
- **Masking.** `env show` masks secret variables unless `--reveal`; Wave Store auth/proxy/cert credentials are **always** masked (no override), including under `--json`.
- **No secrets in logs.** `--verbose` prints operation context (data dir, resolved ids, timing) but never secrets, tokens, or resolved auth headers. The "Sent" snapshot respects the same redaction rules.
- **No side-effect log files.** A `wvc` invocation writes diagnostics to stderr behind `--verbose` only; it never writes a log file.

---

## Related

- [Build Your Own Client](../build-your-own-client.md) — the headless-client pattern `wvc` is built on
- [VS Code extension](vscode.md) · [Web app](web-app.md) — the other two platforms sharing this workspace
- [Reporting](../features/reporting.md) — the HTML run reports `--report` produces
- [Versioning](../versioning.md) — the CLI's version track and release checklist
- npm: [`@abranjith/wave-client-cli`](https://www.npmjs.com/package/@abranjith/wave-client-cli)
