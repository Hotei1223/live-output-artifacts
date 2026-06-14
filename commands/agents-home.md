---
description: Build the Rockstar Agent Command Center — a live, per-user artifact for tracking, playbooks, a journey map, an AI roadmap, and every agent's skills and tools
---

Render the **Rockstar Agent Command Center** as a LIVE Cowork artifact.

## Do exactly this — do not improvise the UI
1. **Find the EXACT tool name.** In THIS session, find the available tool whose name ends
   in `__rockstar_agents` — it looks like `mcp__<serverId>__rockstar_agents` where
   `<serverId>` is an opaque UUID (e.g. `mcp__5689d4cc-…__rockstar_agents`). Call it once
   to confirm it returns `{ agents: [...] }`.
2. Read the file `skills/rockstar-agents-home/templates/agents-home.html`.
3. Copy its contents as the HTML body, but **replace the token `__ROCKSTAR_AGENTS_TOOL__`
   with the exact tool name from step 1** (the `const TOOL_AGENTS` line). Change nothing else.
4. Call `mcp__cowork__create_artifact` with that HTML body and
   `mcp_tools: ["<exact tool name from step 1>"]` (required — the page can only call tools
   listed here).

## What the artifact includes
The template renders a full command center with a top nav: **Dashboard** (completion ring,
AI "Your next move", activity heatmap + streak, Pinned, Continue), **Playbooks** (guided
multi-skill sequences), **Journey Map** (Mermaid flow of the playbooks), **Goal Roadmap**
(type a goal, the AI builds an ordered path), and **All Skills** (search, filter, launch,
favorites, per-skill notes + output links, mark-done). Usage is tracked **per user,
device-local** (browser localStorage) — private to each user, not synced across devices.
The AI panels use `window.cowork.askClaude` and fall back gracefully if it's unavailable.
No extra setup or `mcp_tools` beyond the one `__rockstar_agents` tool.

## Tool-name rule — THIS is what was breaking it
In Cowork live artifacts, MCP calls route by the **fully-qualified UUID name**
`mcp__<serverId>__rockstar_agents`, NOT the friendly `mcp__ir-mcp__rockstar_agents`.
Using the friendly name makes the call fail INSIDE Cowork before it reaches the server
(no server log, artifact shows "Failed to load agents: Tool returned an error"). The
serverId UUID changes per install, so it must be read live every time — never hardcode it.

## Hard rules (the last attempt broke these)
- **Do NOT write your own HTML/CSS/layout.** Use the template as-is — it is already designed.
- **Do NOT paste any of these instructions, the procedure text, or any skill prose into the
  artifact.** The page must contain ONLY the template markup — no visible prompt/instruction text.
- **Do NOT print the catalog as chat text.** It MUST be one live artifact.
- **Do NOT hardcode the data** or add your own reload button — the template fetches live and the
  artifact header already has Reload.
- Use ONLY the real catalog data the tool returns; never invent agents, skills, or tools.
