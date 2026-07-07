# live-output-artifacts

A Claude plugin that bundles your MCP server with a skill so that, whenever your
output-generating tool runs, Claude renders the result as a **live, re-fetching
Cowork artifact** instead of plain chat text.

## What's inside

```
live-output-artifacts/
├── .claude-plugin/
│   ├── plugin.json          # plugin manifest
│   └── marketplace.json     # marketplace listing (for publishing)
├── .mcp.json                # ir-mcp server (remote HTTP)
├── skills/
│   └── render-live-artifact/
│       ├── SKILL.md         # drives the auto-artifact behavior
│       └── templates/
│           └── live-artifact.html
└── README.md
```

The **skill** is what makes the behavior automatic; the **plugin** packages the
skill + server together so it installs in one step. The plugin adds no new
capability beyond Cowork's existing `create_artifact` — it just makes the setup
repeatable and shippable.

## Already configured

- Server: `ir-mcp` at `aidistrict agents` (`.mcp.json`).
- The skill triggers on **any** tool from the `ir-mcp` server, so new
  orchestrators you expose later are covered automatically — no per-tool edits.

## Ready to publish

Repo: `https://github.com/Sunil3696/live-output-artifacts`. Nothing left to fill
in. (In the artifact template, `TOOL` is a placeholder Claude substitutes with the
actual orchestrator name at artifact-creation time — you don't hardcode it.)

If your server uses SSE rather than streamable HTTP, change `"type": "http"` to
`"type": "sse"` in `.mcp.json`. (HTTP is recommended where supported.)

## Publish it

1. Push this folder to a GitHub repo (it can be both the plugin and the
   marketplace, since `marketplace.json` lists itself).
2. Validate locally: `claude plugin validate --plugin-dir ./live-output-artifacts`

## Install it (what your users run)

```
/plugin marketplace add Sunil3696/live-output-artifacts
/plugin install live-output-artifacts
/reload-plugins
```

After install, the MCP server connects and the skill activates automatically —
when any `ir-mcp` orchestrator produces output, Claude renders it as a live artifact.

## A note on "live" + LLM output

Live artifacts re-run your tool every time the page is opened. If your tool
regenerates LLM output on each call, the page will produce different results on
each reload. The skill handles this: if the output isn't meaningfully refreshable,
it falls back to a one-time static artifact. Decide which fits your tool.

## Host portability (Cowork, Codex, and beyond)

Every runtime template ships with a small **host adapter** at the top (look for
`HOST ADAPTER`). It lets the same artifact run across hosts instead of hard-binding
to Cowork's `window.cowork` global — and, just as important, it **degrades to a
clear message instead of throwing on an undefined global** when opened with no host.

Resolution order for every `callMcpTool` / `askClaude` the templates make:

1. **Claude Cowork** — native `window.cowork.callMcpTool` / `askClaude` (untouched when present).
2. **OpenAI / Codex Apps** — `window.openai.callTool`, and `sendFollowUpMessage` for "launch a skill".
3. **Generic MCP host** — `window.{craftAgent|codex|mcpHost|host}.callMcpTool` / `callTool` / `tools.call`.
4. **Authenticated HTTP fallback** — a minimal MCP Streamable-HTTP client
   (`initialize → notifications/initialized → tools/call`, SSE-aware).

The HTTP fallback is **opt-in** and reads config from globals or `<meta>` tags an
embedder can inject before the artifact loads:

| Config | `<meta>` equivalent | Purpose |
| --- | --- | --- |
| `window.__MCP_ENDPOINT` | `<meta name="mcp-endpoint">` | MCP Streamable-HTTP URL |
| `window.__MCP_TOKEN` | `<meta name="mcp-token">` | Bearer token for that URL |
| `window.__LOA_NEW_CHAT_URL` | — | "launch a skill" URL base (default `claude://cowork/new?q=`) |

Notes:
- When a host bridge (1–3) answers, the HTTP path is never used.
- The HTTP fallback is best-effort: the target server must permit the artifact's
  origin via **CORS**, and most production MCP servers (including `ir-mcp`) require
  a bearer token — supply one via `__MCP_TOKEN` or rely on a host bridge instead.
- No host and no endpoint → the UI shows *"No MCP host detected…"* rather than a
  blank/broken page. AI panels fall back to deterministic suggestions when no
  `askClaude`/host LLM is available.
