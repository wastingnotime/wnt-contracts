# Agent Runtime v1

The [job schema](./job.schema.json) describes a request for bounded agent
execution. The [outcome schema](./outcome.schema.json) describes one durable
terminal result. `schemaVersion` selects the compatibility version.

## Rules

- `jobId` is stable across retries. Consumers use it to avoid duplicate PRs
  and terminal outcomes.
- `agent` selects a named runtime registration. A worker rejects unknown names;
  the name does not grant authority by itself. `jobType` describes the task.
- `repository.revision` pins the source revision used as the job starting
  point. A worker checks it before editing.
- `authority` is a request ceiling, not a permission grant. The runtime checks
  an independent repository, tool, credential, and network policy and may
  reject a broader request.
- `maxRuntimeSeconds` is a hard job limit, including sandbox startup. The
  transport may have a lower limit.
- `maxTokens` and `maxCostUsd` are optional request ceilings for runtimes that
  can enforce them. A worker must reject a supplied ceiling it cannot enforce;
  it must not silently treat a measured-afterward value as a hard limit.
- `inputs` and outcome references point to durable, access-controlled
  artifacts. Do not put credentials, tokens, personal data, source archives,
  or unrestricted prompts in queue messages or evidence fields.
- `succeeded` means the declared success conditions were met and the artifact
  or PR is linked. `escalated` means a semantic or authority question needs a
  human decision. `failed` means execution failed. `blocked` means a named
  dependency prevented progress.
- A consumer records the terminal outcome before acknowledging a delivery.
- Validators must assert the JSON Schema `format` values for UUIDs, URIs, and
  timestamps; a parser that only treats `format` as documentation is not
  sufficient at the submission or execution boundary.

The first producer and worker live in `infra-platform`. Queue addresses,
credential delivery, Codex options, and deployment details are outside this
contract.
