# agenticdiagrams/skills

Agent skills for [agenticdiagrams.com](https://agenticdiagrams.com) — the open schema for diagramming agentic systems (agents, tools, routers, guardrails, memory, flows, scenarios).

Currently ships one skill, `agentic-diagrams`, which teaches your coding agent to author and validate `.agentic.yaml` files. The skill always fetches the live spec from agenticdiagrams.com before authoring, then validates the output with `@agenticdiagrams/schema` so every diagram is guaranteed to parse.

## Install

### Claude Code (native plugin)

```
/plugin marketplace add agenticdiagrams/skills
/plugin install agentic-diagrams@agentic-diagrams
```

### Cursor, VS Code / Copilot, Windsurf, Codex, and 40+ other tools

Uses the [Agent Skills spec](https://agentskills.io/specification) via Vercel's `skills` CLI:

```
npx skills add agenticdiagrams/skills
```

The CLI installs the skill into your editor's native skills directory (for example, `~/.cursor/skills/` for Cursor or `~/.codeium/windsurf/skills/` for Windsurf). Run `npx skills list` to confirm.

## Use

Once installed, ask your agent to diagram something:

> *"make an agent diagram of this file"*
> 
> *"draw the flow for our support agent that calls a KB and a ticket API"*
> 
> *"validate my diagram.agentic.yaml"*

The skill activates automatically. The output lands as a `.agentic.yaml` file you can import at [app.agenticdiagrams.com](https://app.agenticdiagrams.com) to render, animate, present, and share.

## Repo layout

```
.
├── .claude-plugin/
│   ├── marketplace.json     # Claude Code marketplace catalog
│   └── plugin.json          # Claude Code plugin manifest
└── skills/
    └── agentic-diagrams/
        └── SKILL.md         # canonical skill — discovered by both install paths
```

One canonical `SKILL.md` serves both distribution channels:

- Claude Code finds it via the plugin manifest (default `skills/` directory).
- The Vercel `skills` CLI finds it via the repo root's `skills/` directory (and also honors `.claude-plugin/marketplace.json`).

## License

MIT
