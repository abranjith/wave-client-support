<h1>
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/abranjith/wave-client-support/main/packages/vscode/assets/logos/wave-client-logo-dark.png">
    <img src="https://raw.githubusercontent.com/abranjith/wave-client-support/main/packages/vscode/assets/logos/wave-client-logo-light-32.png" alt="Wave Client logo" width="28" height="28" align="absmiddle">
  </picture>
  &nbsp;@wave-client/vscode
</h1>

The Wave Client **VS Code extension** — runs the shared Wave Client UI inside VS Code.

## Purpose

This package delivers Wave Client as a native VS Code extension, so you can build and send API requests right next to your code. It reuses the platform‑agnostic UI from [`@wave-client/core`](https://github.com/abranjith/wave-client-support/blob/main/packages/core/README.md) and bridges it to VS Code through a platform adapter.

## What you can do

- **Requests beyond HTTP** — HTTP, **WebSocket**, and **SSE**, with rich body editors and a "Sent" view of the exact outgoing request.
- **Organize** — nested [collections](https://github.com/abranjith/wave-client-support/blob/main/docs/features/collections.md) with import (Postman, OpenAPI/Swagger, HTTP) and export.
- **Parameterize** — [environments](https://github.com/abranjith/wave-client-support/blob/main/docs/features/environments.md), `{{variables}}`, and dynamic [`_fn_` functions](https://github.com/abranjith/wave-client-support/blob/main/docs/features/variables.md).
- **Authenticate** — API Key, Basic, Digest, OAuth2 (Refresh, Client Credentials, Authorization Code/PKCE), and HMAC, plus a reusable [Wave Store](https://github.com/abranjith/wave-client-support/blob/main/docs/features/wave-store.md) for cookies, auth, proxies, and certificates.
- **Validate** — response checks via [JSONPath and JSON Schema](https://github.com/abranjith/wave-client-support/blob/main/docs/features/validations.md).
- **Automate** — [flows](https://github.com/abranjith/wave-client-support/blob/main/docs/features/flows.md), [test suites](https://github.com/abranjith/wave-client-support/blob/main/docs/features/tests.md), and exportable [run reports](https://github.com/abranjith/wave-client-support/blob/main/docs/features/reporting.md).
- **AI built in** — [Wave Arena](https://github.com/abranjith/wave-client-support/blob/main/docs/features/ai-arena.md) and an MCP server for external AI tools.

## Documentation

See the full documentation at [`docs/`](https://github.com/abranjith/wave-client-support/blob/main/docs/README.md) — start with the [VS Code platform guide](https://github.com/abranjith/wave-client-support/blob/main/docs/platforms/vscode.md). To understand the overall architecture, see [Design & Architecture](https://github.com/abranjith/wave-client-support/blob/main/docs/design.md).

## Feedback & Community

Wave Client is in public beta and your input directly shapes what gets built next.

**Found a bug?** [Open a bug report](https://github.com/abranjith/wave-client-support/issues/new?template=bug_report.md) — the more detail you include (environment, steps to reproduce, a HAR file if relevant), the faster it gets fixed.

**Have an idea?** [Open a feature request](https://github.com/abranjith/wave-client-support/issues/new?template=feature_request.md) — new platform clients (CLI, desktop, …) are explicitly welcome, not just improvements to the existing ones.

**Browse existing issues** before opening a new one — upvoting an existing issue is the best signal that something matters.