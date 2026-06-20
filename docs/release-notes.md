# Release Notes

> Wave Client is in **public beta**. This page highlights what each release brings. For exact change history, see source control.

---

## Beta update — auth schemes, structured test authoring, placeholders & result accuracy

A feature‑heavy pass that expands authentication, turns the Test Lab into a structured authoring surface, adds per‑request placeholder overrides, makes collection import/move workflows complete, and makes run results accurate and consistent everywhere.

### Authentication
- **Three new auth types** — **OAuth2 Client Credentials**, **OAuth2 Authorization Code (PKCE, manual code/URL paste)**, and **HMAC** request signing, configured end‑to‑end. See [Auth](features/auth.md).
- **Cleaned‑up config** — Digest now configures only username/password (all other parameters come from the server challenge); the misleading "Base64 Encode Credentials" option and unused fields were removed. OAuth2 types now require an explicit **client auth method**, and a **no‑assumed‑defaults** policy means you choose every value that affects how a credential is computed (challenge method, HMAC algorithm/encoding, …).

### Requests & Variables
- **Request placeholders** — a new **Placeholders** tab lets you preview and **temporarily override** the `{{variables}}` a request uses (URL, params, headers, body), without touching environments. Overrides are per‑tab, in‑memory only, take precedence over environment values, and turn overridden variables green in the editor. See [Variables](features/variables.md).

### Collections
- **Import into an existing collection** — merge an imported file into any collection or folder (all‑or‑nothing on name conflicts), or create a new collection with an explicit, validated name (no more silent overwrites).
- **New Folder** — create validated folders in place from the collection/folder menus.
- **Folder Move** — Move now works on folders too, carrying the whole subtree; all moves are atomic (validate‑first, with rollback on cross‑collection failure).
- **Import‑time replacements** — apply `find → replace` rules (literal or regex) to request URLs and bodies during import, e.g. swap a Swagger host for `{{ApiBaseUrl}}`. Swagger specs with no server URL now generate a `{{<Name>Url}}` placeholder instead of a fake host. See [Collections](features/collections.md).
- **Save write‑back fixes** — saved‑tab breadcrumbs no longer duplicate/drop path segments, wizard‑saved tabs link correctly to their collection, the dirty dot clears reliably, and both apps show one identical success notification.

### Test Lab
- **Structured test case editor** — per‑case overrides for variables, headers, query params, and **body** (None / Raw / File / Form URL‑Encoded / Multipart) now have proper table editors, replacing the raw‑JSON sections. Detected request placeholders prepopulate the variables table.
- **Validation overrides in the UI** — add or link assertions per test case with an **inline** rule editor (no nested modal).
- **Richer display & a formal schema** — the suite list shows full names, untruncated descriptions, and an override summary; test suites now have a formal, validated schema documented in the [Test Suite Schema Reference](test-suite-schema.md).

### Results & reporting
- **Friendly errors** — DNS, connection, timeout, and TLS failures show a clear message and hint instead of `HTTP 0` / `Unknown Error`.
- **Validation‑aware status** — a request that *expects* a non‑2xx status (e.g. an "expect 400" rule) now counts as **Passed**; a 200 with a failing validation counts as **Failed**. A failing test case rolls its item and suite up to Failed.
- **Sent request everywhere** — run cards and HTML reports show the actual on‑wire request (the "Sent" snapshot), consistently across the collection runner, flows, test suites, and reports. See [Reporting](features/reporting.md).

### UI
- Flow and Test Suite toolbars regroup editing vs. running controls; Test Suite **Run Settings** is now a self‑contained collapsible section in the editor body; results side‑panels widen responsively on larger screens.

---

## Beta update — import fidelity, cancellation & collection integrity

A stabilization pass across collections, environments, requests, and history, plus formalized file schemas and versioning docs.

### Requests
- **Cancel in-flight requests** — while a request is processing, **Send** becomes **Cancel**; aborting stops the actual outbound call on both apps and settles the viewer in a distinct "Request cancelled" state.
- **Ctrl+S / Cmd+S** saves the active request from the request editor (direct save, or the Save dialog for unsaved requests) — see [keyboard shortcuts](features/requests.md#keyboard-shortcuts).
- **Response download fixed** on both apps — downloads now honor the real response bytes, a sensible filename, and the response content type.
- **History refreshes in real time** — the History pane updates immediately after each send, no reload needed.
- Tighter request-editor spacing, with Params/Headers **Copy/Paste** buttons relocated into the table header.

### Collections
- **Full `.http` / `.rest` import syntax** — `###` separators, `#` and `//` comments, `# @name` directives, file-variable lines, optional methods, multi-line URLs, and automatic unique request naming. See [Collections](features/collections.md).
- **Content-based format auto-detection** on import — Postman, OpenAPI/Swagger, `.http`, and Wave files are recognized from their content, with the dropdown as a manual override.
- **Reliable collection operations** — rename, delete, and move now persist dependably; item identity is stable across rename/move/duplicate; naming rules (non-empty, sibling-unique) are enforced everywhere.
- **Clearer Move & Save dialog** — shows the request's current location, offers a single searchable destination picker listing every collection and folder (any depth), and blocks moves that would overwrite an existing item.

### Environments
- **Postman environment import** with an auto-detected, overridable **Environment Type** dropdown; secret-typed Postman variables import as Wave secrets. See [Environments](features/environments.md).

### Schemas & versioning
- The persisted collection and environment formats are now formal, versioned schemas (both `0.0.1`), validated at import and load — documented field by field in [Wave Schemas](schemas.md).
- How Wave Client versions evolve is now documented in [Versioning](versioning.md): four independent tracks with per-track semver semantics and bump checklists.

---

## Beta — Initial public release

The first public beta of Wave Client, available as both a **VS Code extension** and a **standalone web app** with feature parity between them.

### Requests & Protocols
- HTTP requests with all common methods, visual editors for **params, headers, and body**.
- Body types: **None, Raw** (JSON / XML / HTML / Text / CSV), **URL‑encoded, multipart Form‑data** (text + file fields), and **File**.
- **WebSocket** and **Server‑Sent Events (SSE)** connections with live Messages/Events panels.
- A request‑side **"Sent"** view showing the exact on‑wire request (final URL, headers, display‑safe body).
- A default `User-Agent` is added when you don't supply one.
- See [Requests](features/requests.md).

### Collections & Import
- Nested **collections/folders**, with **rename, delete (confirmed), move, and duplicate**.
- Import from **Postman (v2.1.0)**, **OpenAPI 3.x / Swagger 2.0** (JSON or YAML), and **HTTP** files; export support.
- Run collections with a **result explorer**.
- See [Collections](features/collections.md).

### Environments & Variables
- **Environments** with global and environment‑specific values, import/export.
- `{{variable}}` resolution plus **dynamic `_fn_` functions** (UUIDs, random numbers/strings, dates/times, names, addresses, contacts).
- See [Environments](features/environments.md) and [Variables](features/variables.md).

### Authentication & Wave Store
- Auth types: **API Key, Basic, Digest, OAuth2 (refresh token)**, with domain filters.
- **Wave Store** for reusable **cookies, auth, proxies, and client certificates (mTLS)**.
- See [Auth](features/auth.md) and [Wave Store](features/wave-store.md).

### Validations
- Response validation by **status, headers, and body**, using full **JSONPath** and **JSON Schema (draft‑07)**; defaults to 2xx success when no rules are set.
- See [Validations](features/validations.md).

### Flows
- Visual **flow canvas** chaining requests, with **conditional connectors** (including validation pass/fail) and **readable aliases** for JSONPath data passing between steps.
- See [Flows](features/flows.md).

### Test Lab
- **Test suites** that run multiple requests with assertions and report pass/fail.
- See [Test Lab](features/tests.md).

### Reporting
- Interactive, self‑contained **HTML run reports** for collections, flows, and test suites (search, status filtering, expand/collapse).
- See [Reporting](features/reporting.md).

### AI / Wave Arena
- In‑app **AI assistant** for web‑standards questions and tool‑backed workspace help, with slash commands and confirmation‑gated run actions.
- An **MCP server** to expose your workspace to external AI tools.
- See [AI & Wave Arena](features/ai-arena.md).

### Settings & Security
- Settings wizard for **data storage**, **encryption** (with recovery key), and **AI** configuration.
- See [Settings](features/settings.md).

---

## How releases are logged

Going forward, each release adds a new section at the top of this page summarizing notable additions and changes, grouped by area. This page reflects highlights, not an exhaustive changelog — use git history for the complete record.
