---
name: graphops-substreams
description: Stream ordered blockchain data with GraphOps Substreams over gRPC, using the Substreams CLI, sinks or a gRPC client authenticated with a GraphOps API key. Use when code needs every block or a Substreams module's output in order, a backfill over a block range, a sink that writes to a database, or a consumer that must resume from cursors and handle reorgs through *.substreams.graphops.xyz.
metadata:
  version: 9fac094481b92e5e
  source: https://docs.graphops.xyz/.well-known/agent-skills/graphops-substreams/SKILL.md
---

# GraphOps Substreams

GraphOps Substreams streams the output of Substreams packages, block by block and in order, over
gRPC. Use it for indexers, sinks and backfills. If the task only needs a few reads of chain state or
indexed entities, see the `graphops` skill to choose another service.

## Connect

- Read the key from `SUBSTREAMS_API_KEY`, which the `substreams` CLI and its sinks send as the
  `x-api-key` header. Raw gRPC clients send `x-api-key` request metadata themselves. If the key
  isn't set, ask the person to create a key with Substreams access and set the variable; see
  [API keys](https://docs.graphops.xyz/platform/api-keys.md). If the GraphOps CLI is installed,
  `graphops substreams run|gui|sink` supply the organization's stored key themselves and
  `graphops exec --product substreams --network <network> -- <command>` sets it, with the GraphOps
  endpoint, for any other command such as a sink; pass that endpoint with `-e` too. See the
  `graphops-cli` skill.
- The key never goes in an endpoint URL or a query string.
- EVM endpoint: `https://<CHAIN_ID>.evm.substreams.graphops.xyz`, for example
  `https://1.evm.substreams.graphops.xyz` for Ethereum mainnet. Look up chain IDs and registry
  network names on [Supported networks](https://docs.graphops.xyz/substreams/networks.md) or with the docs MCP
  `list_networks` tool.

```sh
substreams run \
  -e "https://1.evm.substreams.graphops.xyz" \
  <PACKAGE> <MODULE> \
  --start-block 18000000 \
  --stop-block +10 \
  -o jsonl
```

## Bound every stream

- Give every exploratory run a `--start-block` and a short `--stop-block` (for example `+10`)
  until the output is right. Every block streamed uses credits.
- Split large backfills into ranges and checkpoint progress. See
  [Backfill a range](https://docs.graphops.xyz/substreams/guides/backfill-a-range.md).
- Bound the number of concurrent streams, and how fast you open them. Use backoff when reconnecting.
- Inspect a package's modules and output types before running it. See
  [Packages and modules](https://docs.graphops.xyz/substreams/concepts/packages-and-modules.md) and
  [Discover Substreams](https://docs.graphops.xyz/substreams/guides/discover-packages.md).

## Write consumers that survive restarts and reorgs

- Save each block's cursor in the same transaction as the output it follows, and resume from the
  last committed cursor after a disconnect.
- On a `BlockUndoSignal`, revert everything written after the last valid block it names, then save
  its cursor.
- Never claim exactly-once results without both rules.

Read [Cursors and reorgs](https://docs.graphops.xyz/substreams/concepts/cursors-and-reorgs.md) before writing a consumer, then
[Resume with cursors](https://docs.graphops.xyz/substreams/guides/resume-with-cursors.md) and
[Run a sink](https://docs.graphops.xyz/substreams/guides/run-a-sink.md).

## When a stream fails

| Symptom                                                   | What to do                                                                                                                                                                                                                                 |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Authentication fails                                      | Stop and ask the person to check `SUBSTREAMS_API_KEY` and that it is a Substreams key.                                                                                                                                                     |
| Refused with `credits_exhausted` or `usage_access_paused` | Stop and tell the person. Don't reconnect.                                                                                                                                                                                                 |
| Refused with a valid key                                  | Read [Stream is refused or ends with a valid key](https://docs.graphops.xyz/substreams/troubleshooting.md#stream-is-refused-or-ends-with-a-valid-key).                                                                                     |
| Refused with `RESOURCE_EXHAUSTED`                         | Read [Stream refused with `RESOURCE_EXHAUSTED`](https://docs.graphops.xyz/substreams/troubleshooting.md#stream-refused-with-resource_exhausted). For `request_rate_limited`, wait for `retryAfterSeconds` before opening the stream again. |
| Package or module not found                               | Check the package reference and module name against the package.                                                                                                                                                                           |
| Disconnects                                               | Reconnect with backoff from the last committed cursor.                                                                                                                                                                                     |

[Troubleshooting](https://docs.graphops.xyz/substreams/troubleshooting.md) covers each case in detail.

## Pages to read for the task

- First stream: [Get started](https://docs.graphops.xyz/substreams/getting-started.md)
- Stream fields and messages for gRPC clients: [API reference](https://docs.graphops.xyz/substreams/api.md)
- Writing to a database or webhook: [Sinks and materialization](https://docs.graphops.xyz/substreams/concepts/sinks-and-materialization.md)
- Credits and organization limits: [Usage and limits](https://docs.graphops.xyz/substreams/usage-and-limits.md)

## Keep these instructions current

These instructions are published at https://docs.graphops.xyz/.well-known/agent-skills/graphops-substreams/SKILL.md. At the start of a task, fetch that URL and compare
its `metadata.version` with the version of this copy. When they differ, tell the person that
their GraphOps skills are out of date, and follow the published version for the task.
