---
name: agentic-diagrams
description: Use this skill when the user is creating, authoring, drafting, editing, validating, or fixing a `.agentic.yaml` / `.agentic.yml` file; when they ask to diagram, model, map, or architect an agentic system, multi-agent workflow, AI agent pipeline, tool/router/guardrail/memory topology, or agent-to-agent handoff; when they ask to diagram an existing codebase, repo, or service from its source; or whenever the request mentions "Agentic Diagrams", "agentic schema", "agenticdiagrams.com", the `@agenticdiagrams/schema` npm package, or the agentic.yaml spec (v0.1 or later). Covers authoring from a natural-language description, authoring from source-code inspection, validating existing files, and fixing schema errors.
---

# agentic-diagrams

Helps users author and validate `.agentic.yaml` files — the open schema for describing agentic system architectures (agents, tools, routers, guardrails, memory, flows, scenarios).

This skill is **fully self-contained**. Everything needed to author and validate a diagram ships with it — no network access, no package installation, no scripts required.

## Bundled references

Read these files from the skill's own directory (paths relative to this SKILL.md):

| File | What it is |
|---|---|
| `references/spec.md` | The complete human-readable spec: all node types, edge types, scenarios, layout, defaults, design decisions |
| `references/agentic.schema.json` | The machine-readable JSON Schema (draft 2020-12) — the authority on valid fields, enums, and shapes |
| `references/examples/simple-agent.agentic.yaml` | A small complete example (agent + tool + model + scenario) |
| `references/examples/multi-agent-with-guardrails.agentic.yaml` | A larger example (multi-agent, guardrails, groups) |

Bundled spec version: **0.1** (vendored from `@agenticdiagrams/schema` v0.1.7). Before authoring or validating, read `references/spec.md` — do not work from memory of other diagram formats, and do not invent node types, edge types, or field names that are not in the bundled spec.

## Authoring workflow

1. **Understand the system.** Identify:
   - Actors (human users, agents, sub-agents, orchestrators)
   - Tools, APIs, and MCP servers the agents call
   - Models (LLMs, embedding models) and prompts
   - Data stores (vector DBs, knowledge bases, memory, caches)
   - Routers, guardrails, and policy checks
   - The flow or user journey that brings the pieces together
2. **Read `references/spec.md`** and pick node and edge types from it — never invent them.
3. **Draft the YAML** with:
   - `agentic: '0.1'` at the top (quoted — it is a string, not a number)
   - A `diagram:` block with `name` and optional `description`
   - A `nodes:` map keyed by stable, **kebab-case** ids (`support-agent`, `kb-search`, `stripe-api`)
   - Edges either inline on each node (`edges: [{ to: other-node }]`) or in a top-level `edges:` array — both are valid and can be mixed; inline is terser, top-level suits responses and cross-cutting edges
   - An optional `scenarios:` block when the user describes a specific flow or user journey, so the diagram animates the steps
   - No `layout:` block unless the user asks for one — the editor auto-arranges
4. **Prefer clarity over cleverness.** Labels default to the node id, so well-named ids produce a readable diagram with no extra config.
5. **Never guess types.** If something the user describes does not map cleanly to a type in the spec, say so and propose the closest match rather than inventing a new `type` or `sub_type`.

### Diagramming an existing codebase

When asked to diagram a repo, service, or file from its source, inspect the code first and map what you find onto spec types:

- **Entry points and triggers** — HTTP handlers, queue consumers, cron jobs, webhooks → `trigger`, `gateway`, or `channel` nodes; end users → `user` or `human`
- **Agent loops** — code that calls an LLM in a loop with tools → `agent` (set `is_looping: true` where applicable); the model itself → `model` with `provider`
- **Tool calls** — functions/APIs/MCP servers the agent invokes → `tool` with a `sub_type` (`mcp`, `api`, `function`, `custom`)
- **State** — vector stores, databases, caches, session state → `memory` (with `ttl`/`scope`) or `knowledge_base`; other backing services → `backend`
- **Control flow** — branching/dispatch logic → `router`; validation/safety/policy checks → `guardrail`; fan-in → `aggregator`
- **Boundaries** — deployment or trust boundaries → `group` nodes with children pointing at them via `group:`

Then model the main request path as a scenario: one step per hop, `type: return` for responses, so the diagram can be played back. Name nodes after the real things in the code (file, service, or class names in kebab-case) so the diagram is recognizable to the people who work on it.

### Minimal example

```yaml
agentic: '0.1'

diagram:
  name: Support Bot

nodes:
  user:
    type: user
    edges:
      - to: support-agent

  support-agent:
    type: agent
    edges:
      - to: kb-search
      - to: ticket-api

  kb-search:
    type: tool
    sub_type: mcp

  ticket-api:
    type: tool
    sub_type: api
```

The bundled `references/examples/` files show fuller diagrams, including scenarios.

## Validation workflow

Validate by checking the YAML against `references/agentic.schema.json` yourself — read the schema and verify the document conforms. No tooling is required. Work through this checklist; it covers the real failure modes:

1. **Version:** `agentic: '0.1'` is present and is a quoted string (unquoted `0.1` parses as a number and fails).
2. **No invented keys.** The schema sets `additionalProperties: false` almost everywhere — every field name must appear in the schema for that object. A typo'd or guessed key is an error, not a no-op.
3. **Node types:** every node's `type` is one of the schema's `nodeType` enum values (29 of them — check the enum, not memory).
4. **Enums generally:** edge `type`, step `type`, `path`, `arrows`, `fragment.type`/`position`, `note.position`, `layout.direction`, `diagram.type` all have closed enum values in the schema.
5. **References resolve** — the JSON Schema *cannot* check these; verify them explicitly:
   - Every edge `to` (and top-level `from`) names an existing node id
   - Every scenario step's `from` and `to`, or every id in its `nodes` list, names an existing node id
   - Every node `group:` names an existing container node (usually type `group`)
   - `diagram.active_scenario` names an existing scenario id
6. **Handles:** `source_handle`/`target_handle` match the pattern `^(top|right|bottom|left)(-\d+(-t)?)?$` (e.g. `bottom`, `bottom-20`, `top-80-t`). Preserve any `-t` suffixes when editing an existing file.
7. **Shapes:** `nodes` and `scenarios` are maps keyed by id; `edges` and `steps` are arrays. Each step is either a `from`/`to` pair (a message) or a `nodes` list (nodes highlighted together: at least one id, no duplicates), never both. Layout `positions` values are `[x, y]`, `sizes` values are `[width, height]` or `[width]` alone, and `viewport` is `[x, y, zoom]`.

Report each problem with its YAML path (e.g. `nodes.support-agent.type`), the approximate line, and a concrete fix drawn from the spec. Re-check after fixing until the document is clean.

Optional: if the user has a Node.js project and wants programmatic validation, the `@agenticdiagrams/schema` npm package exports `validate()`. Offer it only if asked — it is never required for this workflow.

## Output

- Write the file to disk. Default to `[diagram-name].agentic.yaml` in the current working directory unless the user specifies a different path or filename.
- Use the `.agentic.yaml` (or `.agentic.yml`) extension so editors and tooling recognize it.
- After writing, tell the user:

  > You can import this file at **https://app.agenticdiagrams.com** to render, animate, present, and share the diagram.

## Staying current

The live spec and schema are published at `https://agenticdiagrams.com/docs/spec/` and `https://agenticdiagrams.com/schemas/agentic/0.1.json`. Consult them only if the user explicitly asks to check for a newer spec version, or if a file declares a spec version newer than the bundled `0.1`. Inability to reach them is never a blocker — the bundled references are sufficient.

## Do not

- Do not invent node types, edge types, `sub_type` values, or field names. If the user's concept does not fit the spec, name the closest existing type and flag the mismatch. A node property the spec doesn't model goes under that node's `metadata` (string, number or boolean values), never as a new top-level key.
- Do not fetch anything from the network or install any package to author or validate — the bundled references are sufficient and authoritative for spec 0.1.
- Do not skip validation after authoring. Always run the validation checklist before handing the file back.
- Do not rename or reshape the user's existing file silently when fixing errors. Explain each change.
