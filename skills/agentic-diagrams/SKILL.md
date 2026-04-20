---
name: agentic-diagrams
description: Use this skill when the user is creating, authoring, drafting, editing, validating, or fixing a `.agentic.yaml` / `.agentic.yml` file; when they ask to diagram, model, or architect an agentic system, multi-agent workflow, AI agent pipeline, tool/router/guardrail/memory topology, or agent-to-agent handoff; or whenever the request mentions "Agentic Diagrams", "agentic schema", "agenticdiagrams.com", the `@agenticdiagrams/schema` npm package, or the agentic.yaml spec (v0.1 or later). Covers authoring from a natural-language description, validating existing files, and fixing schema errors.
---

# agentic-diagrams

Helps users author and validate `.agentic.yaml` files — the open schema for describing agentic system architectures (agents, tools, routers, guardrails, memory, flows, scenarios).

## Hard rule: always fetch the live spec first

Before writing or validating any `.agentic.yaml`, fetch the live spec:

- **URL:** `https://agenticdiagrams.com/docs/spec/`
- **Also fetch, as needed:** `/docs/spec/nodes`, `/docs/spec/edges`, `/docs/spec/scenarios`, `/docs/spec/layout`
- **Machine-readable schema:** `https://agenticdiagrams.com/schemas/agentic/0.1.json`

The schema is versioned (currently `0.1`) and evolves. Node types, edge types, and field names may change between versions. **Do not rely on memory or training data** — always fetch the current spec before authoring or validating. If the fetch fails, tell the user and stop; do not guess.

## Authoring workflow

1. **Understand the system.** From the user's description, identify:
   - Actors (human users, agents, sub-agents, orchestrators)
   - Tools, APIs, and MCP servers the agents call
   - Data stores (vector DBs, knowledge bases, memory, caches)
   - Routers, guardrails, and policy checks
   - The flow or user journey that brings the pieces together
2. **Fetch the live spec** (see above) and pick node and edge types **from the spec** — do not invent them.
3. **Draft the YAML** with:
   - `agentic: '0.1'` at the top (use the version from the live spec)
   - A `diagram:` block with `name` and optional `description`
   - A `nodes:` map keyed by stable, **kebab-case** ids (`support-agent`, `kb-search`, `stripe-api`)
   - Edges expressed either inline on each node (`edges: [{ to: other-node }]`) or as a top-level `edges:` array — match what the live spec recommends
   - An optional `scenarios:` block when the user describes a specific flow or user journey, so the diagram animates the steps
4. **Prefer clarity over cleverness.** Labels default to the node id, so well-named ids produce a readable diagram with no extra config.
5. **Never guess types.** If something the user describes does not map cleanly to a type in the spec, say so and propose the closest match rather than inventing a new `type` or `sub_type`.

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
    sub_type: rest
```

Always verify each `type` and `sub_type` against the freshly fetched spec — the example above is illustrative, not authoritative.

## Validation workflow

1. Check whether `@agenticdiagrams/schema` is installed. If not, install it:
   ```bash
   npm install @agenticdiagrams/schema
   ```
   Use the project's package manager if different (`pnpm add`, `yarn add`, `bun add`).
2. Parse the YAML and run the exported `validate()` function:
   ```ts
   import * as yaml from 'js-yaml';
   import { validate } from '@agenticdiagrams/schema';

   const doc = yaml.load(yamlString);
   const result = validate(doc);
   if (!result.valid) console.error(result.errors);
   ```
3. For each error, report:
   - The YAML path (e.g. `nodes.support-agent.type`)
   - The approximate line number when possible
   - A concrete fix drawn from the live spec (valid values, required fields, correct shape)
4. Re-run `validate()` after applying fixes until `result.valid` is `true`.

## Output

- Write the file to disk. Default to `[diagram-name].agentic.yaml` in the current working directory unless the user specifies a different path or filename.
- Use the `.agentic.yaml` (or `.agentic.yml`) extension so editors and tooling recognize it.
- After writing, tell the user:

  > You can import this file at **https://app.agenticdiagrams.com** to render, animate, present, and share the diagram.

## Do not

- Do not invent node types, edge types, `sub_type` values, or field names. If the user's concept does not fit the spec, name the closest existing type and flag the mismatch.
- Do not hardcode the spec from memory. Fetch it every run — it changes.
- Do not skip validation after authoring. Always run `validate()` before handing the file back.
- Do not rename or reshape the user's existing file silently when fixing errors. Explain each change.
