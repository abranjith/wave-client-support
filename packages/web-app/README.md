# Wave Client

A modern, local-first **REST API client** — build and send HTTP / WebSocket / SSE
requests, organize them into collections, parameterize with environments, validate
responses, chain requests into flows and test suites, and use a built-in AI assistant.

This package is the **standalone web app**: one command starts a local server and
opens the Wave Client UI in your browser. Your data is stored locally on your
machine (by default under `~/.waveclient`).

> Also available as a [VS Code extension](https://marketplace.visualstudio.com/).

## Requirements

- Node.js **18.18+** (LTS recommended)

## Quick start

> This package is published under your npm scope. Replace `@your-scope` below
> with the actual scope/name you publish as. The installed command is always
> `wave-client`.

Run it directly with npx (no install):

```bash
npx @abranjith/wave-client
```

…or install it globally:

```bash
npm install -g @abranjith/wave-client
wave-client
```

This starts the app and opens `http://127.0.0.1:3456` in your browser.

## Usage

```
wave-client [options]

  -p, --port <port>      Port to listen on            (default: 3456, env: WAVE_PORT)
      --host <host>      Interface to bind            (default: 127.0.0.1, env: WAVE_HOST)
      --data-dir <dir>   Directory for Wave data      (default: ~/.waveclient, env: WAVE_DATA_DIR)
      --no-open          Do not open the browser automatically
  -v, --version          Print the version and exit
  -h, --help             Show this help and exit
```

Examples:

```bash
wave-client --port 8080                 # run on a different port
wave-client --data-dir ./my-wave-data   # keep data in a project folder
wave-client --no-open                   # start without launching a browser
```

The server and UI are served from a **single local port**. By default it binds to
loopback (`127.0.0.1`) only. You can pass `--host 0.0.0.0` to expose it on your
network, but do so **only on trusted networks** — there is no authentication.

## License

See the LICENSE file. © Wave Client.
