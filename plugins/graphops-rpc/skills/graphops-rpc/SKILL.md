---
name: graphops-rpc
description: Read EVM and Bitcoin chain data with GraphOps RPC, JSON-RPC over HTTPS and WebSocket authenticated with a GraphOps API key. Use when code or a question needs blocks, transactions, receipts, balances, contract calls (eth_call), event logs (eth_getLogs), archive reads, live subscriptions or transaction submission through rpc.graphops.xyz.
metadata:
  version: 4b2682dad28cc13f
  source: https://docs.graphops.xyz/.well-known/agent-skills/graphops-rpc/SKILL.md
---

# GraphOps RPC

GraphOps RPC serves standard JSON-RPC for EVM networks and Bitcoin. Use it for chain state and
event logs. If the task needs indexed entities or an ordered stream of every block, see the
`graphops` skill to choose another service.

## Use the MCP tools when they're connected

If the GraphOps RPC MCP server (`https://rpc.graphops.xyz/v1/mcp`) is connected, answer questions
about chain data with its tools instead of writing requests: `evm_rpc` for read-only EVM methods,
`bitcoin_rpc` for read-only Bitcoin methods, and `evm_send_raw_transaction` or
`bitcoin_send_raw_transaction` only under the rules in
"Change chain state only when asked" below. Tools take the network
as a CAIP-2 ID, such as `eip155:1` for Ethereum mainnet; look it up with the docs MCP
`list_networks` tool. Each result states the block it reflects. A failed call returns an `action`:
`fix_arguments`, `narrow_request`, `retry_later` or `stop`; follow it. Write code against the
endpoint below when the person needs code, not an answer.

## Connect

- Read the key from `GRAPHOPS_RPC_API_KEY`. If the GraphOps CLI is installed, run the code with
  `graphops exec --product rpc -- <command>`, which sets it for that process only; see the
  `graphops-cli` skill. Otherwise, if it isn't set, ask the person to create an RPC key and set the
  variable; see [API keys](https://docs.graphops.xyz/platform/api-keys.md).
- EVM endpoint: `https://rpc.graphops.xyz/v1/evm/<CHAIN_ID>`, where `<CHAIN_ID>` is the decimal
  chain ID, for example `1` for Ethereum mainnet.
- Send `Authorization: Bearer $GRAPHOPS_RPC_API_KEY` and `Content-Type: application/json`.
- Look up the chain ID, Bitcoin network names and archive coverage on
  [Supported networks](https://docs.graphops.xyz/rpc/networks.md) or with the docs MCP `list_networks` tool. Don't assume a
  network is supported.
- For WebSocket clients that can't set headers, use the route-token URL from
  [Authentication](https://docs.graphops.xyz/rpc/authentication.md#route-token-authentication). It contains the key, so never
  log or print it.

```sh
curl -sS "https://rpc.graphops.xyz/v1/evm/1" \
  -H "Authorization: Bearer $GRAPHOPS_RPC_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
```

## Make bounded, consistent requests

- Pin a block for any answer built from more than one call: read the block number once, then pass
  it as the block parameter to every call, and report it. See
  [Consistency and finality](https://docs.graphops.xyz/rpc/concepts/consistency-and-finality.md).
- Always give `eth_getLogs` an explicit `fromBlock` and `toBlock`, and filter by address and
  topics. Split long ranges into chunks, and halve a chunk that times out or returns a
  too-large error. See [Fetch logs](https://docs.graphops.xyz/rpc/guides/fetch-logs.md) and
  [Backfill historical events](https://docs.graphops.xyz/rpc/guides/backfill-historical-events.md).
- Bound concurrency. Keep JSON-RPC batches small, match responses by `id`, and retry only the
  items that failed with a retryable error.
- Retry `429`, `502`, `503` and `504` with exponential backoff and jitter, and cap total retry
  time. See [Retries and reorgs](https://docs.graphops.xyz/rpc/guides/retries-and-reorgs.md).
- Check that a method is supported before relying on it: [Methods](https://docs.graphops.xyz/rpc/methods.md).

## Change chain state only when asked

Submit a transaction only when the person asked for it and gave you the signed transaction or
the means to sign it. If a submission fails with a retryable error, resend the same signed
transaction; never change and re-sign it to retry. See
[Send a raw transaction](https://docs.graphops.xyz/rpc/guides/send-raw-transaction.md).

## When a request fails

| Response                                                | What to do                                                                                             |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `401`                                                   | The key is missing or invalid. Stop and ask the person to check `GRAPHOPS_RPC_API_KEY`.                |
| `403` with `credits_exhausted` or `usage_access_paused` | Stop and tell the person. Don't retry.                                                                 |
| `403` with another code                                 | Read the code's entry in [Error reference](https://docs.graphops.xyz/platform/errors.md).              |
| `429`                                                   | Back off and retry with jitter. Reduce concurrency if it repeats.                                      |
| Timeout or response too large                           | Narrow the block range or the request, then retry.                                                     |
| JSON-RPC `method not found`                             | The method isn't supported on this network. Check [Methods](https://docs.graphops.xyz/rpc/methods.md). |

[Troubleshooting](https://docs.graphops.xyz/rpc/troubleshooting.md) covers each case in detail.

## Pages to read for the task

- First request: [Get started](https://docs.graphops.xyz/rpc/getting-started.md)
- Request and batch format, error responses: [API reference](https://docs.graphops.xyz/rpc/api.md)
- Contract reads: [Read contract state](https://docs.graphops.xyz/rpc/guides/read-contract-state.md)
- Historical state: [Archive reads](https://docs.graphops.xyz/rpc/concepts/archive-reads.md)
- Live events: [WebSocket subscriptions](https://docs.graphops.xyz/rpc/guides/websocket-subscriptions.md)
- Credits and organization limits: [Usage and limits](https://docs.graphops.xyz/rpc/usage-and-limits.md)

## Keep these instructions current

These instructions are published at https://docs.graphops.xyz/.well-known/agent-skills/graphops-rpc/SKILL.md. At the start of a task, fetch that URL and compare
its `metadata.version` with the version of this copy. When they differ, tell the person that
their GraphOps skills are out of date, and follow the published version for the task.
