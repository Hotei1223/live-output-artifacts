---
name: rockstar-agents-home
description: >
  Use when the user wants to browse or see the available Rockstar / IR agents —
  e.g. "show me the agents", "what agents do I have", "open the agents home",
  "list the orchestrators", "what can Rockstar do", "agent directory/menu". Renders
  a LIVE Cowork artifact listing every agent with the skills and tools inside each,
  re-fetching from the server every time the user reopens it.
---

# Rockstar Agents home (live)

Render a LIVE Cowork artifact that lists every Rockstar / IR agent the user can
access, with the skills and tools inside each. Each skill has a **Start chat**
button that opens a new Cowork chat for that skill.

## Procedure
1. Read `templates/agents-home.html` (in this skill folder).
2. Call `mcp__cowork__create_artifact` with that file's contents as the HTML body
   **VERBATIM** (byte-for-byte) and `mcp_tools: ["mcp__ir-mcp__rockstar_agents"]`
   (required — the page can only call tools listed here).
3. That's it. The template fetches live via `window.cowork.callMcpTool`, unwraps the
   response, renders the agent/skills/tools, has a search box, wires every Start chat
   button, and refreshes on reopen.

## Hard rules
- **Do NOT write your own HTML/CSS/layout** — the template is already designed; use it as-is.
- **Do NOT paste this skill's text, the procedure, or any instructions into the artifact.**
  The page must contain ONLY the template markup — no visible prompt text.
- **Do NOT print the catalog as chat text** — it MUST be one live artifact.
- **Do NOT hardcode the data** and do NOT add your own reload button (the header has one).
- Use ONLY the real catalog data the tool returns; never invent agents, skills, or tools.
- One "Rockstar Agents" home — update the existing artifact instead of duplicating.
