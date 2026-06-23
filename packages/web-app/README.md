<h1>
  &nbsp;@wave-client/web-app
</h1>

> **🚧 Not yet published — coming soon.** The npm command below is a preview. For now, run it from source in dev mode (Contributors only).

The Wave Client **web app** — runs the shared Wave Client UI in the browser.

## Purpose

This package delivers Wave Client as a standalone web application. It reuses the platform‑agnostic UI from [`@wave-client/core`](../core/README.md) and bridges it to a local backend ([`@wave-client/server`](../server/README.md)) through a web platform adapter.

## What you can do

- **Requests beyond HTTP** — HTTP, **WebSocket**, and **SSE**, with rich body editors and a "Sent" view of the exact outgoing request.
- **Organize** — nested [collections](../../docs/features/collections.md) with import (Postman, OpenAPI/Swagger, HTTP) and export.
- **Parameterize** — [environments](../../docs/features/environments.md), `{{variables}}`, and dynamic [`_fn_` functions](../../docs/features/variables.md).
- **Authenticate** — API Key, Basic, Digest, OAuth2 (Refresh, Client Credentials, Authorization Code/PKCE), and HMAC, plus a reusable [Wave Store](../../docs/features/wave-store.md) for cookies, auth, proxies, and certificates.
- **Validate** — response checks via [JSONPath and JSON Schema](../../docs/features/validations.md).
- **Automate** — [flows](../../docs/features/flows.md), [test suites](../../docs/features/tests.md), and exportable [run reports](../../docs/features/reporting.md).
- **AI built in** — [Wave Arena](../../docs/features/ai-arena.md) and an MCP server for external AI tools.

## Documentation

See the full documentation at [`docs/`](../../docs/README.md) — start with the [Web app guide](../../docs/platforms/web-app.md). For how the adapter works, see [Design & Architecture](../../docs/design.md).

## Feedback & Community

Wave Client is in public beta and your input directly shapes what gets built next.

**Found a bug?** [Open a bug report](https://github.com/abranjith/wave-client-support/issues/new?template=bug_report.md) — the more detail you include (environment, steps to reproduce, a HAR file if relevant), the faster it gets fixed.

**Have an idea?** [Open a feature request](https://github.com/abranjith/wave-client-support/issues/new?template=feature_request.md) — new platform clients (CLI, desktop, …) are explicitly welcome, not just improvements to the existing ones.

**Browse existing issues** before opening a new one — upvoting an existing issue is the best signal that something matters.