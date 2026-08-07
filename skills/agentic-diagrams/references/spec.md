# Agentic YAML Specification v0.1

A YAML format for describing agentic system diagrams as human-readable text.

File extension: `.agentic.yaml` or `.agentic.yml`

## Minimum valid document

```yaml
agentic: '0.1'
```

An empty diagram. Name defaults to `"Untitled"`, no nodes, no edges.

A practical minimum:

```yaml
agentic: '0.1'
nodes:
  my-agent:
    type: agent
```

Label defaults to the node id. Edge type defaults to `request`.

## Defaults

| Property           | Default      | Notes                               |
| ------------------ | ------------ | ----------------------------------- |
| `diagram.name`     | `"Untitled"` | Name of the diagram                 |
| Node `label`       | node id      | `my-agent` → displays as "my-agent" |
| Edge `type`        | `request`    | Most common in agent diagrams       |
| Step `type`        | `call`       | Most common in flows                |
| `layout.direction` | `TB`         | Top to bottom                       |

## Top-level structure

| Key         | Type   | Required | Description                      |
| ----------- | ------ | -------- | -------------------------------- |
| `agentic`   | string | yes      | Spec version (`"0.1"`)           |
| `diagram`   | object | no       | Diagram metadata                 |
| `nodes`     | map    | no       | Node definitions (id → node)     |
| `edges`     | array  | no       | Connections between nodes        |
| `scenarios` | map    | no       | Flow definitions (id → scenario) |
| `layout`    | object | no       | Visual/rendering hints           |

## Diagram

```yaml
agentic: '0.1'
diagram:
  name: My Diagram
  description: Optional description
  type: system # system | component
  active_scenario: flow-1 # default scenario to display
  parent_diagram: parent-id # parent diagram (drill-down hierarchy)
  created: '2026-03-10T21:24:34.728Z'
  updated: '2026-04-10T12:00:00.000Z'
```

All fields are optional. `name` defaults to `"Untitled"`.

`parent_diagram` identifies the parent diagram when this file represents a drilled-down sub-system.

## Nodes

Nodes are a map of **id → definition**. The id is a stable, human-readable identifier used to reference the node in edges and scenarios. When `label` is omitted, the id is used as the display name.

```yaml
nodes:
  orchestrator:
    type: agent
    is_looping: true

  pantry-server:
    type: tool
    sub_type: mcp
```

### Common properties

| Property         | Type   | Required | Description                            |
| ---------------- | ------ | -------- | -------------------------------------- |
| `type`           | string | yes      | Node type (see list below)             |
| `label`          | string | no       | Display name (defaults to node id)     |
| `sub_title`      | string | no       | Secondary display text below label     |
| `description`    | string | no       | Longer description text                |
| `sub_type`       | string | no       | Type-specific variant                  |
| `url`            | string | no       | External link                          |
| `group`          | string | no       | Parent group node id                   |
| `linked_diagram` | string | no       | Drill-down target (diagram id or path) |
| `edges`          | array  | no       | Inline edge definitions (see below)    |

### Type-specific properties

These properties are meaningful for specific node types but accepted on any node.

| Property           | Type    | Relevant types           | Description                                                            |
| ------------------ | ------- | ------------------------ | ---------------------------------------------------------------------- |
| `is_looping`       | boolean | agent                    | Runs in a loop                                                         |
| `auth`             | string  | agent, backend           | Auth mechanism (OAuth, JWT, etc)                                       |
| `auth_detail`      | string  | agent, backend           | Auth subtitle/detail                                                   |
| `provider`         | string  | model                    | Model provider (OpenAI, etc)                                           |
| `content`          | string  | prompt, note, text_label | Text content                                                           |
| `example_response` | string  | agent                    | Example agent response                                                 |
| `ttl`              | string  | memory                   | Retention lifetime (e.g. `session`, `24h`, `permanent`, or freeform)   |
| `scope`            | string  | memory                   | Audience scope (e.g. `per-user`, `per-session`, `global`, or freeform) |

#### Memory node retention fields

`ttl` and `scope` describe data lifecycle for memory nodes, aiding security and compliance reviews.

```yaml
nodes:
  session-cache:
    type: memory
    sub_type: vector
    ttl: 24h
    scope: per-user

  shared-knowledge:
    type: memory
    sub_type: sql
    ttl: permanent
    scope: global

  temp-context:
    type: memory
    ttl: session # cleared when the session ends
    scope: per-session
```

Preset `ttl` values: `session`, `24h`, `permanent`. Any freeform string (e.g. `7d`, `30m`, `1y`) is also valid.

Preset `scope` values: `per-user`, `per-session`, `global`. Any freeform string (e.g. `per-team`, `per-org`) is also valid.

### Node types

Core types for agentic systems:

| Type             | Description                    | Common sub_types                                           |
| ---------------- | ------------------------------ | ---------------------------------------------------------- |
| `agent`          | AI agent or orchestrator       | —                                                          |
| `tool`           | External tool / integration    | `mcp`, `custom`, `api`, `function`                         |
| `model`          | LLM or embedding model         | `llm`, `embedding`, `vision`, `custom`                     |
| `memory`         | Persistent memory store        | `vector`, `sql`, `file`                                    |
| `knowledge_base` | Knowledge / RAG source         | `documents`, `embeddings`, `web_search`, `structured_data` |
| `prompt`         | Prompt definition              | `system`, `user`, `few_shot`, `template`                   |
| `guardrail`      | Safety filter / policy         | `output_filter`, `input_filter`, `policy`                  |
| `human`          | Human actor / approval point   | —                                                          |
| `trigger`        | Event trigger / webhook        | —                                                          |
| `router`         | Message / request router       | —                                                          |
| `gateway`        | API gateway / message broker   | —                                                          |
| `skill`          | Agent skill / capability       | —                                                          |
| `aggregator`     | Data aggregator / collector    | —                                                          |
| `observability`  | Monitoring / logging / tracing | —                                                          |
| `channel`        | Communication channel          | `web`, `email`, `mobile`, `chat`, `app`                    |

Structural types:

| Type          | Description                 | Common sub_types                            |
| ------------- | --------------------------- | ------------------------------------------- |
| `system`      | System with drill-down      | —                                           |
| `component`   | Software component          | —                                           |
| `backend`     | Backend service / datastore | —                                           |
| `group`       | Visual grouping container   | `zone`, `team`, `vpc`, `boundary`, `custom` |
| `user`        | User / customer entity      | —                                           |
| `output`      | System output / response    | —                                           |
| `environment` | Deployment environment      | —                                           |
| `generic`     | Untyped node                | —                                           |

Visual types (typically generated by the editor, not hand-written):

| Type         | Description              |
| ------------ | ------------------------ |
| `note`       | Freeform comment         |
| `text_label` | Simple text label        |
| `block`      | Visual block / container |
| `image`      | Image display            |
| `divider`    | Visual divider line      |
| `waypoint`   | Invisible routing point  |

## Edges

Edges can be defined in two ways that are equivalent and can be mixed freely. Edge `type` defaults to `request` when omitted.

### Inline edges (on nodes)

Edges defined directly on a node infer `from` as that node. This co-locates a node's connections with its definition:

```yaml
nodes:
  orchestrator:
    type: agent
    edges:
      - to: inventory-agent
        label: Check stock
      - to: recipe-agent
      - to: streaming-service
        type: streaming
        animated: true
```

### Top-level edges

Edges in the top-level `edges` array specify both `from` and `to` explicitly. Useful for responses or when you prefer a flat structure:

```yaml
edges:
  - from: inventory-agent
    to: orchestrator
    type: response
```

Both forms produce identical results. On import, inline edges are merged with top-level edges. On export, the editor may use either form.

### Edge properties

| Property        | Type    | Required       | Default   | Description                                 |
| --------------- | ------- | -------------- | --------- | ------------------------------------------- |
| `from`          | string  | top-level only | —         | Source node id (inferred when inline)       |
| `to`            | string  | yes            | —         | Target node id                              |
| `type`          | string  | no             | `request` | Edge type (see below)                       |
| `label`         | string  | no             | —         | Edge label                                  |
| `animated`      | boolean | no             | `false`   | Show animated flow on connection            |
| `arrows`        | string  | no             | `end`     | Arrowheads: `none`, `start`, `end`, `both`  |
| `path`          | string  | no             | `bezier`  | Path rendering hint                         |
| `source_handle` | string  | no             | —         | Connection point on source node (see below) |
| `target_handle` | string  | no             | —         | Connection point on target node (see below) |
| `metadata`      | object  | no             | —         | Presentation hints for the edge (see below) |

### Edge metadata

`metadata` carries freeform presentation hints set by the editor. Known keys are typed; unknown keys are allowed so the editor can stash additional cosmetics without a schema change.

| Key         | Type   | Description                                                                                              |
| ----------- | ------ | -------------------------------------------------------------------------------------------------------- |
| `label_pos` | number | Label position along the edge as a fraction of its length (0 = source end, 1 = target end). Default 0.5. |

```yaml
edges:
  - from: gw
    to: plan
    type: data
    label: issue
    metadata:
      label_pos: 0.72 # slide the label 72 % along the edge
```

### Edge types

| Type        | Meaning                       |
| ----------- | ----------------------------- |
| `request`   | Synchronous request (default) |
| `response`  | Response to a request         |
| `async`     | Asynchronous message          |
| `streaming` | Streaming data flow           |
| `event`     | Event-driven trigger          |
| `error`     | Error / exception path        |
| `default`   | Generic untyped connection    |

### Path types

| Value           | Description              |
| --------------- | ------------------------ |
| `bezier`        | Curved line (default)    |
| `straight`      | Straight line            |
| `step`          | Right-angle steps        |
| `smooth_step`   | Smooth right-angle steps |
| `simple_bezier` | Simple bezier curve      |

### Handle positions

`source_handle` and `target_handle` accept two forms:

**Plain side name** — connects at the centre of that side (default behavior):

```yaml
source_handle: bottom
target_handle: top
```

**Slot form** — places the anchor at a specific percentage along the side, in 10 % increments (10–90):

```yaml
source_handle: bottom-20 # 20 % from the left edge of the bottom side
target_handle: top-80-t # 80 % from the left edge of the top side (-t marks a target slot)
```

The `-t` suffix distinguishes target slots from source slots when both are present on the same side. It is added automatically by the editor on export and must be preserved when hand-editing.

When handles are omitted, the editor uses direction-aware defaults (`bottom`/`top` for TB, `right`/`left` for LR, etc.).

## Scenarios

Scenarios define flow sequences that can be played back step-by-step. Step `type` defaults to `call` when omitted.

```yaml
scenarios:
  dinner-prep:
    name: Dinner Preparation
    steps:
      - from: chef
        to: orchestrator
        label: What can we cook?
        payload: '{"request": "dinner suggestions"}'

      - from: orchestrator
        to: inventory-agent
        label: Check ingredients
```

### Scenario properties

| Property      | Type   | Required | Description          |
| ------------- | ------ | -------- | -------------------- |
| `name`        | string | yes      | Display name         |
| `description` | string | no       | Scenario description |
| `steps`       | array  | no       | Ordered flow steps   |

### Step properties

| Property   | Type   | Required | Default | Description                                            |
| ---------- | ------ | -------- | ------- | ------------------------------------------------------ |
| `from`     | string | yes      | —       | Source node id                                         |
| `to`       | string | yes      | —       | Target node id                                         |
| `type`     | string | no       | `call`  | Step type: `call`, `return`, `async`, `event`, `error` |
| `label`    | string | no       | —       | Short label shown during playback                      |
| `payload`  | string | no       | —       | Detail data shown in panel                             |
| `duration` | number | no       | —       | Custom playback duration (ms)                          |
| `fragment` | object | no       | —       | Sequence diagram fragment                              |
| `note`     | object | no       | —       | Sequence diagram note                                  |

### Fragment

Fragments model sequence diagram constructs (alt/opt/loop/par).

```yaml
- from: orchestrator
  to: inventory-agent
  label: Check stock
  fragment:
    type: loop
    label: 'Retry up to 3 times'
    position: start

- from: inventory-agent
  to: orchestrator
  type: return
  label: Stock levels
  fragment:
    type: loop
    position: end
```

| Property   | Type   | Required | Values                      |
| ---------- | ------ | -------- | --------------------------- |
| `type`     | string | yes      | `alt`, `opt`, `loop`, `par` |
| `label`    | string | no       | Condition or description    |
| `position` | string | yes      | `start`, `else`, `end`      |

### Note

Notes attach to steps for sequence diagram annotations.

```yaml
- from: orchestrator
  to: model
  label: Generate response
  note:
    text: 'Uses GPT-4 with temperature=0.7'
    position: right
```

| Property   | Type   | Required | Values                  |
| ---------- | ------ | -------- | ----------------------- |
| `text`     | string | yes      | Note content            |
| `position` | string | yes      | `left`, `right`, `over` |

## Layout

The layout section contains visual rendering hints. It is optional and typically generated by the editor. Hand-written files can omit it entirely — the editor will auto-layout using `direction` (default `TB`).

When `positions` are provided, they take precedence over auto-layout.

```yaml
layout:
  direction: LR
  viewport: [418.5, -19, 2]
  positions:
    orchestrator: [350, 0]
    inventory-agent: [100, 200]
  sizes:
    orchestrator: [240, 117]
  node_styles:
    orchestrator:
      borderless: true
      icon_url: 'https://example.com/icon.png'
      icon_layout: left
```

### Layout properties

| Property      | Type   | Default | Description                                     |
| ------------- | ------ | ------- | ----------------------------------------------- |
| `direction`   | string | `TB`    | Auto-layout flow: `TB`, `LR`, `BT`, `RL`        |
| `viewport`    | array  | —       | `[x, y, zoom]` — initial camera position        |
| `positions`   | map    | —       | Node id → `[x, y]` coordinates                  |
| `sizes`       | map    | —       | Node id → `[width, height]` measured dimensions |
| `node_styles` | map    | —       | Node id → visual style overrides (see below)    |

### Direction values

| Value | Meaning                 |
| ----- | ----------------------- |
| `TB`  | Top to bottom (default) |
| `LR`  | Left to right           |
| `BT`  | Bottom to top           |
| `RL`  | Right to left           |

### Node style properties

| Property      | Type    | Description                                     |
| ------------- | ------- | ----------------------------------------------- |
| `borderless`  | boolean | Hide node border                                |
| `icon_url`    | string  | Custom icon URL                                 |
| `icon_layout` | string  | Icon position: `left`, `right`, `top`, `bottom` |
| `image_url`   | string  | Background image (image nodes)                  |

## Design decisions

**Smart defaults** — Edge type defaults to `request` (most common in agent diagrams), step type to `call`, label to the node id, direction to `TB`. A file with just `agentic: "0.1"` is valid.

**Node IDs as map keys** — Avoids repeating `id:` on every node. IDs must be unique, URL-safe strings. When importing, the key becomes the node's internal ID.

**`sub_type` not `tool_type`/`memory_type`/etc.** — One universal field instead of 15 type-specific fields. The node's `type` already provides context for what `sub_type` means.

**`from`/`to` not `source`/`target`** — Consistent between edges and scenario steps. More readable.

**Edges as array, not map** — Edge IDs aren't meaningful to humans. Auto-generated on import.

**Inline or top-level edges** — Edges can be defined on the node (`edges:` array, `from` inferred) or in the top-level `edges:` array (explicit `from`/`to`). Both forms produce identical results and can be mixed. Inline form is more terse and co-locates connections with the node; top-level form is better for responses and cross-cutting edges.

**Layout is optional** — The core spec (nodes, edges, scenarios) fully describes the diagram's meaning. Layout can be omitted when hand-writing and the editor will auto-arrange using `direction`.

## Type name mapping

Node types that differ between YAML and internal representation:

| YAML spec        | Internal (editor) |
| ---------------- | ----------------- |
| `knowledge_base` | `knowledgeBase`   |
| `text_label`     | `textLabel`       |

All other types are identical in both formats.

Property name mapping:

| YAML spec          | Internal (editor)                               |
| ------------------ | ----------------------------------------------- |
| `diagram`          | (top-level metadata)                            |
| `sub_type`         | `toolType`, `memoryType`, etc. (type-dependent) |
| `sub_title`        | `subTitle`                                      |
| `is_looping`       | `isLooping`                                     |
| `auth`             | `authType`                                      |
| `auth_detail`      | `authSubTitle`                                  |
| `provider`         | `modelProvider`                                 |
| `example_response` | `exampleResponse`                               |
| `linked_diagram`   | `linkedDiagramId`                               |
| `ttl`              | `memoryTtl`                                     |
| `scope`            | `memoryScope`                                   |
| `group`            | `parentId` (on Node)                            |
| `parent_diagram`   | `parentId` (on Diagram)                         |
| `from` / `to`      | `source` / `target` (edges)                     |
| `path`             | `data.pathType` (edges)                         |

Edge type bidirectional mapping:

| YAML        | Internal  | Notes                                               |
| ----------- | --------- | --------------------------------------------------- |
| _(omitted)_ | `request` | YAML omitted → imports as `request`                 |
| `default`   | `default` | Distinct style (grey/no arrow), preserved on export |
| `request`   | `request` | Same on both sides                                  |

Path type bidirectional mapping:

| YAML            | Internal       |
| --------------- | -------------- |
| `smooth_step`   | `smoothStep`   |
| `simple_bezier` | `simpleBezier` |
| `bezier`        | `bezier`       |
| `straight`      | `straight`     |
| `step`          | `step`         |
