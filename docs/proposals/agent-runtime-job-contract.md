# Agent Runtime Job Contract Proposal

## Consumers

- First real consumer: the `infra-platform` Agent Runtime dispatcher and worker
  planned by [campaign #692](https://github.com/wastingnotime/infra-platform/issues/692).
- Planned consumers in the [WNT Agentic Software Lifecycle](https://linear.app/wastingnotime/initiative/wnt-agentic-software-lifecycle-091a03ae7e4d): Air remediation and autonomous release campaigns. They do not yet consume this contract.

## Invariant

Every WNT agent job needs an immutable identity, repository revision, bounded
objective, context references, explicit authority and limits, success and
escalation conditions, and lineage. Every terminal run needs a structured
outcome and evidence references. These meanings are shared across workflows;
the AWS queue, sandbox, and GitHub implementation remain local to their owners.

## Public Surface

`contracts/agent-runtime/v1/job.schema.json` and `outcome.schema.json` define
the v1 JSON wire format. Unknown fields are rejected. A breaking change gets
a new version directory. Additive optional fields may enter v1 only after the
first consumer demonstrates the need and confirms older readers ignore them.

The schemas carry references, not credentials, source bundles, or secrets.
They express the requested authority; the runtime must enforce its own
allowlist and may narrow or reject any request. `productionChange` is fixed to
`false` in v1.

## Why Shared Now

The initiative explicitly defines a common job contract as the first delivery
step, before Air and campaign workers migrate. A single wire format is a
WNT-wide execution invariant, and the `wnt-contracts` repository owns JSON
schemas. Keeping it here makes independent consumers validate the same
messages without copying an infrastructure-specific implementation.

## Migration

1. Review and merge the v1 schemas in `wnt-contracts`.
2. Pin the first `infra-platform` producer and worker to a reviewed contract
   revision; validate on submission and again before execution.
3. Add Air and release campaign consumers separately after their runtime
   behavior is observed.
4. Preserve v1 readers for queued v1 jobs during any later schema transition;
   a new version must not reinterpret an existing queued payload.

Rollback of the first consumer stops new submissions, retains queued messages,
and leaves the schemas available for inspection and replay after repair.
