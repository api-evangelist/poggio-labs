---
name: Track a durable goal with Goalkeeper
description: >-
  Create a durable team goal in Goalkeeper, keep reporting progress against it across
  agent runs, and close it out — using the REST API or the equivalent MCP tools, with the
  idempotency and revision rules that make repeated agent writes safe.
api: openapi/poggio-labs-goalkeeper-openapi.json
mcp: mcp/poggio-labs-goalkeeper-mcp.yml
operations: [listGoalLabels, createGoal, listGoals, getGoal, updateGoal, createGoalUpdate,
  listGoalUpdates]
generated: '2026-08-13'
method: generated
source: >-
  openapi/poggio-labs-goalkeeper-openapi.json (v0.5.0), docs.gkeeper.ai concepts pages,
  conventions/poggio-labs-goalkeeper-conventions.yml
---

# Track a durable goal with Goalkeeper

Goalkeeper is Poggio Labs' Apache-2.0 record of long-running work: the goal, the
definition of done, and the progress so far. It does not run agents — it gives every
agent and person the same record when work moves between sessions and runtimes.

**Before you start.** Goalkeeper is self-hosted. There is no vendor endpoint; use the base
URL of your organization's deployment (the spec ships `http://localhost:3001` for local
development) and the MCP endpoint your operator publishes as `PUBLIC_MCP_URL`.

## Authentication

Send an opaque Goalkeeper API token as `Authorization: Bearer <token>`. The token is
bound to one organization and to the scopes selected at creation:

| Need | Scope |
|---|---|
| read your own goals | `goals:read` |
| create/update/report on your own goals | `goals:write` |
| read every visible goal | `goals:read:all` |
| write every writable goal | `goals:write:all` |
| read labels | `labels:read` |
| manage labels | `labels:write` |

Write scopes do **not** imply read. Tokens expire (1–365 days, default 90) and are
revocable immediately, and the owner's live membership role is re-checked on every call —
a token that worked yesterday can legitimately return `401` or `403` today. Token
management itself (`createApiToken`, `revokeApiToken`) requires an interactive browser
session and is not callable with a token.

## 1. Find the labels you should tag the goal with

`listGoalLabels` — `GET /v1/goal-labels` (MCP: `list_goal_labels`). Labels are
organization-wide; reuse an existing one rather than minting duplicates. Create one with
`createGoalLabel` only if `labels:write` is granted and nothing fits.

## 2. Create the goal

`createGoal` — `POST /v1/goals` (MCP: `create_goal`).

Required: `detailedDescription` (long-form, Markdown allowed) and `timeframe`.
Optional: `title` (≤200 chars), `ownerUserId`, `labelIds` (≤20), `criteria` (≤100).

Write the definition of done into `criteria`. That is what later evaluations are scored
against, and it is the difference between a goal and a task list.

## 3. Report progress — the part that has to be idempotent

`createGoalUpdate` — `POST /v1/goals/{goalId}/updates` (MCP: `report_goal_update`).

Every report **must** carry:

- `status` — the status the goal advances to
- `summary` (≤500 chars) and `details` (long-form)
- `expectedRevision` — the goal `revision` you read before writing
- `idempotencyKey` — your own key (≤200 chars), stable for this logical report

Optional: `health` (omit or send `null` to retain the current value; completed and
archived statuses clear it) and `evaluation`.

The rules that matter for an agent:

- **Re-read before you write.** `getGoal` returns `revision`; pass it as
  `expectedRevision`. If another agent reported in between, you get `409` — re-read,
  re-decide, retry. Do not blind-retry with a stale revision.
- **Reuse the key on retry.** On a network failure or a `409`, retry with the *same*
  `idempotencyKey` so a partially-applied report is not duplicated. Mint a new key only
  for a genuinely new report.
- **Updates are append-only.** History is never rewritten; `listGoalUpdates`
  (`GET /v1/goals/{goalId}/updates`) is the audit trail.

## 4. Change the goal itself, not its status

`updateGoal` — `PATCH /v1/goals/{goalId}` (MCP: `update_goal`) changes description,
timeframe, criteria, ownership or labels, and needs at least one field. Status, health and
evaluation never travel here — they go through step 3.

## 5. Close it out

Report a terminal `status` through `createGoalUpdate`. **Archive rather than delete**:
the MCP surface ships no goal-delete tool on purpose. `deleteGoal`
(`DELETE /v1/goals/{goalId}`) exists in REST only, and an agent should not reach for it.

## Errors you will actually hit

| Status | Meaning | Do |
|---|---|---|
| `400` | The request is invalid | Fix the payload against the schema; do not retry unchanged |
| `401` | Not authenticated | Token expired or revoked — get a new one |
| `403` | Credential lacks the required scope or authority | Missing scope, or the owner lost the role that conferred it |
| `404` | Not found **or not visible to you** | Never treat as proof the goal does not exist |
| `409` | Conflicts with existing state | Optimistic-concurrency rejection — re-read `revision` and retry with the same key |

The error body is Goalkeeper's own `{"error": "...", "message": "..."}` object, not
RFC 9457 problem+json. There is no `Retry-After` and no rate-limit headers — Goalkeeper
publishes no rate limits, so back off on your own schedule.

## Attribution

Goalkeeper separates the authorizing user from the immediate actor on every update: an
API-token or OAuth call attributes the actor to the verified credential, a
provider-verified agent binding takes precedence when present, and self-reported MCP
client name/version are descriptive metadata that never establish identity. Do not try
to write attribution fields yourself.
