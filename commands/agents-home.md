---
description: Build the Rockstar Agents home — a live artifact listing every agent with its skills and tools
---

Render the **Rockstar Agents home** as a LIVE Cowork artifact.

## Do exactly this — do not improvise the UI
1. Read the file `skills/rockstar-agents-home/templates/agents-home.html`.
2. Call `mcp__cowork__create_artifact` and pass that file's contents as the HTML body
   **VERBATIM** — copy it byte-for-byte. Set
   `mcp_tools: ["mcp__ir-mcp__rockstar_agents"]` (required — the page can only call
   tools listed here).
3. The template already: fetches on load via
   `window.cowork.callMcpTool("mcp__ir-mcp__rockstar_agents", {})`, unwraps the response,
   renders the agent with its skills (each with a **Start chat** button) and its tools,
   has a live search box, and refreshes every time it opens.

## Hard rules (the last attempt broke these)
- **Do NOT write your own HTML/CSS/layout.** Use the template as-is — it is already designed.
- **Do NOT paste any of these instructions, the procedure text, or any skill prose into the
  artifact.** The page must contain ONLY the template markup — no visible prompt/instruction text.
- **Do NOT print the catalog as chat text.** It MUST be one live artifact.
- **Do NOT hardcode the data** or add your own reload button — the template fetches live and the
  artifact header already has Reload.
- Use ONLY the real catalog data the tool returns; never invent agents, skills, or tools.
