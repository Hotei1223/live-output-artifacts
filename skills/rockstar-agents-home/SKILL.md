---
name: rockstar-agents-home
description: >
  Use when the user wants to browse their Rockstar / IR agents or track their
  progress — e.g. "show me the agents", "what agents do I have", "open the agents
  home / command center", "what should I run next", "what have I used", "my
  progress". Renders a LIVE, per-user Cowork artifact: a command center with usage
  tracking, an AI "next move", guided playbooks, a journey map, a goal roadmap, and
  every agent's skills and tools — re-fetching the catalog each time it's reopened.
---

# Rockstar Agent Command Center (live)

Render a LIVE Cowork artifact: a per-user command center over every Rockstar / IR
agent the user can access. It tracks which skills they've used vs. pending, suggests
the next move, and lets them launch any skill. Each skill has a **Start chat** button
that opens a new Cowork chat for that skill.

## Procedure
1. **Find the EXACT tool names — there are TWO.** Look at the tools available in THIS
   session and find:
   - the one whose name ends in `__rockstar_agents` (e.g. `mcp__<serverId>__rockstar_agents`)
   - the one whose name ends in `__my_outputs` (e.g. `mcp__<serverId>__my_outputs`)
   `<serverId>` is an opaque UUID, the SAME for both. Call `rockstar_agents` once to confirm
   it returns `{ agents: [...] }`. **These exact names are critical** — see the Tool-name rule.
2. Read `templates/agents-home.html` (in this skill folder).
3. Copy that file's contents as the HTML body, then replace BOTH tokens (do not change
   anything else in the markup):
   - `__ROCKSTAR_AGENTS_TOOL__` → the exact `__rockstar_agents` tool name (the `const TOOL_AGENTS` line)
   - `__MY_OUTPUTS_TOOL__` → the exact `__my_outputs` tool name (the `const TOOL_OUTPUTS` line)
4. Call `mcp__cowork__create_artifact` with that HTML body and
   `mcp_tools: ["<rockstar_agents name>", "<my_outputs name>"]` (required — BOTH, the page can
   only call tools listed here). Use the SAME UUID names here, NOT the friendly `ir-mcp` names.
5. That's it. The template fetches the catalog + the user's outputs live via
   `window.cowork.callMcpTool`, renders the full command center, and refreshes on reopen.

## What the template renders
A command center with a top nav:
- **Dashboard** — completion ring (done/pending/total), an AI "Your next move" panel, an
  activity heatmap + day streak, a Pinned row, and a Continue row.
- **Playbooks** — guided multi-skill sequences (e.g. "Launch a new offer") with progress
  and a highlighted next step.
- **Journey Map** — a Mermaid flow of the playbook chains, colored by done/pending.
- **Goal Roadmap** — the user types a goal and the AI builds an ordered path from the catalog.
- **All Skills** — search, category filters, launch, favorites (★), per-skill notes + output
  links, and mark-done.
- **Outputs** — the user's saved deliverables (live, via `my_outputs`): summary stats, search,
  filter chips for **Client / Skill / Agent**, and cards that open the FULL deliverable in a
  modal (lazy-loaded the first time the tab is opened).

Progress, favorites, notes and activity are stored **per user, device-local** (browser
localStorage) — private to each user, not synced across devices. AI panels use
`window.cowork.askClaude` and fall back to deterministic suggestions if it's unavailable.

## Tool-name rule (this is what breaks if you get it wrong)
- In Cowork live artifacts, MCP tool calls route by the **fully-qualified UUID name**
  `mcp__<serverId>__rockstar_agents`, NOT the friendly connector name
  `mcp__ir-mcp__rockstar_agents`. Using the friendly name makes the call fail INSIDE
  Cowork before it reaches the server — the artifact shows "Failed to load agents:
  Tool returned an error". Always use the exact loaded tool name.

## Hard rules
- **Do NOT write your own HTML/CSS/layout** — the template is already designed; use it as-is
  apart from the one `__ROCKSTAR_AGENTS_TOOL__` substitution above.
- **Do NOT paste this skill's text, the procedure, or any instructions into the artifact.**
  The page must contain ONLY the template markup — no visible prompt text.
- **Do NOT print the catalog as chat text** — it MUST be one live artifact.
- **Do NOT hardcode the data** and do NOT add your own reload button (the header has one).
- Use ONLY the real catalog data the tool returns; never invent agents, skills, or tools.
- One "Rockstar Agents" home — update the existing artifact instead of duplicating.
