# Flow Schema Reference

> **Formal definition**: `packages/core/src/schemas/flowSchema.ts`  
> **Types**: `packages/core/src/types/flow.ts`  
> **Schema version**: 1 (added in wave-client v0.x)

This document describes the Wave **flow** file format — the persisted shape of a visual
request-orchestration document — covering every field, the connector condition
semantics used during execution, and a practical authoring guide for developers (and
AI agents) generating flows programmatically.

---

## Table of Contents

1. [Overview](#overview)
2. [Execution Semantics](#execution-semantics)
3. [Schema Reference](#schema-reference)
   - [Flow](#flow)
   - [FlowNode](#flownode)
   - [FlowConnector](#flowconnector)
   - [ConnectorCondition](#connectorcondition)
4. [Validation](#validation)
5. [Referencing Upstream Output](#referencing-upstream-output)
6. [Annotated Example](#annotated-example)
7. [Authoring Tips for AI Agents](#authoring-tips-for-ai-agents)

---

## Overview

A **flow** is a JSON file (`*.json`) stored in the `flows/` directory. It defines a
dependency-aware chain of HTTP requests laid out on a canvas:

- **Nodes** reference existing collection requests by `requestId`. Each node carries a
  user-defined **alias** used to reference its output downstream.
- **Connectors** are directed edges between nodes. Each connector carries a
  **condition** that decides whether the target node runs based on the source node's
  result.

The flow file is **used directly by the UI** (canvas layout lives in each node's
`position`) and by the flow runner at execution time.

---

## Execution Semantics

At run time the flow runner:

1. Starts from **starting nodes** — nodes with no incoming connectors.
2. Executes a node, producing a result (HTTP response + validation outcome).
3. For each outgoing connector, evaluates its [condition](#connectorcondition) against
   the source node's result. If satisfied, the target node becomes eligible to run
   once **all** its incoming dependencies have completed.
4. Nodes whose connector condition is not met are **skipped**.
5. Before a node runs, `{{alias.…}}` references in its request are resolved from the
   responses of completed upstream nodes (see
   [Referencing Upstream Output](#referencing-upstream-output)).

Flows must be **acyclic** — a cycle makes execution order undefined and is rejected by
[`validateFlow`](#validation).

---

## Schema Reference

### Flow

Top-level object stored to disk.

| Field           | Type               | Required | Description                                          |
|-----------------|--------------------|----------|------------------------------------------------------|
| `id`            | `string`           | ✅        | Unique flow identifier (used as filename)            |
| `name`          | `string` (min 1)   | ✅        | Display name — must be unique across flows           |
| `description`   | `string`           |          | Optional free-text description                       |
| `nodes`         | `FlowNode[]`       | ✅        | Requests placed on the canvas; may be empty          |
| `connectors`    | `FlowConnector[]`  | ✅        | Directed edges between nodes; may be empty           |
| `defaultAuthId` | `string`           |          | Default auth profile applied to all requests         |
| `defaultEnvId`  | `string`           |          | Default environment for variable resolution          |
| `createdAt`     | `string` (ISO 8601)| ✅        | Creation timestamp                                   |
| `updatedAt`     | `string` (ISO 8601)| ✅        | Last modification timestamp                          |

---

### FlowNode

A single request placed on the canvas.

| Field       | Type                       | Required | Description                                                        |
|-------------|----------------------------|----------|--------------------------------------------------------------------|
| `id`        | `string`                   | ✅        | Unique within the flow                                             |
| `alias`     | `string`                   | ✅        | Reference handle for this node's output (e.g., `{{getUser.body.id}}`) |
| `requestId` | `string`                   | ✅        | `"collectionFile.json:requestId"` or just `"requestId"`            |
| `name`      | `string`                   | ✅        | Display name (cached from the referenced request)                 |
| `method`    | `string`                   | ✅        | HTTP method (cached for quick display)                            |
| `url`       | `string`                   |          | Request URL (cached for hover-card display)                       |
| `position`  | `{ x: number; y: number }` | ✅        | Canvas coordinates                                                 |

> `name`, `method`, and `url` are **caches** of the referenced request, kept on the node
> for fast rendering without resolving the collection. `url` is optional; `name` and
> `method` are always written.

---

### FlowConnector

A directed edge between two nodes.

| Field          | Type                 | Required | Description                                       |
|----------------|----------------------|----------|---------------------------------------------------|
| `id`           | `string`             | ✅        | Unique within the flow                            |
| `sourceNodeId` | `string`             | ✅        | The node that must complete first                 |
| `targetNodeId` | `string`             | ✅        | The node that runs after the source, if eligible  |
| `condition`    | `ConnectorCondition` | ✅        | Gate that decides whether the target runs         |

---

### ConnectorCondition

A string enum. The target node runs only when the source node's result satisfies it:

| Value             | Satisfied when…                                                                 |
|-------------------|---------------------------------------------------------------------------------|
| `"any"`           | Always — runs unconditionally after the source completes                        |
| `"success"`       | Source succeeded with an HTTP status in the `200`–`399` range                   |
| `"failure"`       | Source failed (error) or returned a status outside `200`–`399`                  |
| `"validation_pass"` | The source response's validation rules **all passed**                         |
| `"validation_fail"` | The source response ran validation and **at least one rule failed**           |

> ⚠️ These are **plain string values**, not objects. `"success"`, not `{ "type": "success" }`.

---

## Validation

Flows are checked at two distinct layers:

1. **Structural schema validation** — `validateFlowSchema` (in `flowSchema.ts`) checks
   the persisted shape: required fields, field types, and that every connector
   `condition` is one of the five allowed strings. It is **validate-only**: on success
   it returns the original object unchanged (unknown extra fields survive load/save
   round-trips). Invalid files are logged and **skipped** at load time by
   `FlowService.loadOne` so one bad file doesn't break the Flows pane.

2. **Graph validation** — `validateFlow` (in `flowUtils.ts`, distinct from the schema)
   is a runtime check that returns human-readable errors for graph-level problems:
   - the flow has no nodes,
   - duplicate node aliases,
   - duplicate connectors between the same node pair,
   - self-referencing connectors,
   - connectors referencing non-existent nodes,
   - **cycles** (circular dependencies).

   Schema validity does **not** imply graph validity — a structurally valid flow can
   still contain a cycle. The editor surfaces these errors before a run.

---

## Referencing Upstream Output

Each node's `alias` is the handle for its response in downstream requests. Use
`{{alias.<path>}}` in any field of a downstream request (URL, headers, query params,
body) to splice in a value resolved from a completed upstream node:

```
{{getUser.body.id}}              # JSON body field via dot path
{{getUser.body.items[0].name}}   # array indexing
{{getUser.status}}               # response status code
{{getUser.headers.Location}}     # response header
```

References resolve from the responses of **upstream** nodes (those the current node
transitively depends on). A reference that can't be resolved is left intact and the
node's parameters are reported as unresolved.

---

## Annotated Example

A two-step flow: create a user, then fetch it by the id returned from step one.

```json
{
  "id": "flow-user-onboarding",
  "name": "User Onboarding",
  "description": "Create a user, then read it back by id",
  "defaultEnvId": "env-staging",
  "nodes": [
    {
      "id": "node-create",
      "alias": "createUser",
      "requestId": "users-collection.json:req-create-user",
      "name": "Create User",
      "method": "POST",
      "url": "{{BaseUrl}}/users",
      "position": { "x": 50, "y": 50 }
    },
    {
      "id": "node-get",
      "alias": "getUser",
      "requestId": "users-collection.json:req-get-user",
      "name": "Get User",
      "method": "GET",
      "url": "{{BaseUrl}}/users/{{createUser.body.id}}",
      "position": { "x": 350, "y": 50 }
    }
  ],
  "connectors": [
    {
      "id": "conn-1",
      "sourceNodeId": "node-create",
      "targetNodeId": "node-get",
      "condition": "success"
    }
  ],
  "createdAt": "2026-06-16T00:00:00.000Z",
  "updatedAt": "2026-06-16T00:00:00.000Z"
}
```

`getUser` runs only if `createUser` returns a `2xx`/`3xx` status (`condition: "success"`),
and its URL pulls the new id from `createUser`'s response body.

---

## Authoring Tips for AI Agents

When generating flows programmatically, follow these rules to produce valid, runnable
flows:

1. **Always include required fields.** `id`, `name`, `nodes`, `connectors`,
   `createdAt`, `updatedAt` at the flow level; `id`, `alias`, `requestId`, `name`,
   `method`, `position` at the node level; `id`, `sourceNodeId`, `targetNodeId`,
   `condition` at the connector level.

2. **Generate unique IDs.** Use a UUID generator for `id` values. Node and connector
   ids must be unique within the flow.

3. **Keep aliases unique.** Two nodes may not share an alias (case-insensitive) — it's
   the key used for `{{alias.…}}` references and is rejected by graph validation.

4. **Use string conditions.** `condition` is one of `"any"`, `"success"`, `"failure"`,
   `"validation_pass"`, `"validation_fail"` — a bare string, never an object.

5. **Keep the graph acyclic.** Connectors describe dependencies; a cycle is rejected.
   Connector `sourceNodeId`/`targetNodeId` must both reference existing node ids, and a
   connector may not point a node at itself.

6. **Cache `name`/`method` (and `url` if known)** from the referenced request so the UI
   renders nodes without resolving the collection. `url` may be omitted.

7. **Reference upstream output with dot paths.** Splice values from earlier nodes using
   `{{alias.body.field}}`, `{{alias.status}}`, `{{alias.headers.Name}}`, etc. Only
   reference nodes that are upstream of the consuming node.

8. **Set `position` sensibly.** Coordinates are persisted canvas pixels; spread nodes
   left-to-right (increasing `x`) along the execution order so the saved layout is
   readable. The editor can also auto-layout by depth.

9. **Omit optional fields you don't need.** `description`, `defaultAuthId`,
   `defaultEnvId`, and node `url` are optional — leave them out rather than writing
   empty values.
