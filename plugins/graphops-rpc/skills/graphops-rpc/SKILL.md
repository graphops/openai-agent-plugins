---
name: graphops-rpc
description: Read EVM and Bitcoin chain data with GraphOps RPC, JSON-RPC over HTTPS and WebSocket authenticated with a GraphOps API key. Use when code or a question needs blocks, transactions, receipts, balances, contract calls (eth_call), event logs (eth_getLogs), archive reads, live subscriptions or transaction submission through rpc.graphops.xyz.
metadata:
  version: a3ae04c24daa45b6
  source: https://docs.graphops.xyz/.well-known/agent-skills/graphops-rpc/SKILL.md
---

# GraphOps RPC

GraphOps RPC serves standard JSON-RPC for EVM networks and Bitcoin. Use it for chain state and
event logs. If the task needs indexed entities or an ordered stream of every block, see the
`graphops` skill to choose another service.

## Connect

- Read the key from `GRAPHOPS_API_KEY`. If the GraphOps CLI is installed, run the code with
  `graphops exec --product rpc -- <command>`, which sets it for that process only; see the
  `graphops-cli` skill. Otherwise, if it isn't set, ask the person to create a key with RPC access
  and set the variable; see [API keys](https://docs.graphops.xyz/platform/api-keys.md).
- EVM endpoint: `https://rpc.graphops.xyz/v1/evm/<CHAIN_ID>`, where `<CHAIN_ID>` is the decimal
  chain ID, for example `1` for Ethereum mainnet.
- Send `Authorization: Bearer $GRAPHOPS_API_KEY` and `Content-Type: application/json`.
- Look up the chain ID, Bitcoin network names and archive coverage on
  [Supported networks](https://docs.graphops.xyz/rpc/networks.md) or with the docs MCP `list_networks` tool. Don't assume a
  network is supported.
- For WebSocket clients that can't set headers, use the route-token URL from
  [Authentication](https://docs.graphops.xyz/rpc/authentication.md#route-token-authentication). It contains the key, so never
  log or print it.

```sh
curl -sS "https://rpc.graphops.xyz/v1/evm/1" \
  -H "Authorization: Bearer $GRAPHOPS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
```

## Use the MCP tools when they're connected

If the `graphops-rpc` MCP server is connected, answer one-off questions with its tools instead of
writing code. Code the application runs still calls the endpoint above.

- `evm_rpc` and `bitcoin_rpc` run one read-only method, `{ "method": ..., "params": [...] }` with
  positional parameters, and return the node's result with the block it reflects.
- `network` is the CAIP-2 ID, such as `eip155:1`, or `bip122:000000000019d6689c085ae165831e93` for
  Bitcoin mainnet. Read it from [Supported networks](https://docs.graphops.xyz/rpc/networks.md) or the docs MCP
  `list_networks` tool's `id`.
- A state read with a tag reports `{ "requested": "latest" }`, not a number. Pin a block number
  when results must agree, as below.
- Filters, subscriptions, block and range traces and signing methods aren't tools; use the endpoint
  for them. Bitcoin address balances and history aren't available at all.
- A node refusal, such as `execution reverted`, comes back as `rpcError`, unbilled. A result over the
  size limit fails with `upstream_response_too_large`: narrow the range or filter by address and
  topics.
- A tool error's `action` says whether to fix the arguments, ask for less, retry later or stop.
- Send a transaction with `evm_send_raw_transaction` or `bitcoin_send_raw_transaction` only as the
  next section allows. Check a Bitcoin transaction with `testmempoolaccept` first.

[MCP server](https://docs.graphops.xyz/rpc/mcp.md) is the reference for every tool.

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
- Retry a `429` only after the seconds in its `Retry-After` header. Retry `429`, `502`, `503` and
  `504` with exponential backoff and jitter, and cap total retry time. See
  [Retries and reorgs](https://docs.graphops.xyz/rpc/guides/retries-and-reorgs.md).
- Check that a method is supported before relying on it: [Methods](https://docs.graphops.xyz/rpc/methods.md).

## Change chain state only when asked

Submit a transaction only when the person asked for it and gave you the signed transaction or
the means to sign it. If a submission fails with a retryable error, resend the same signed
transaction; never change and re-sign it to retry. See
[Send a raw transaction](https://docs.graphops.xyz/rpc/guides/send-raw-transaction.md).

## When a request fails

| Response                                                | What to do                                                                                                                                                                                                                                             |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `401`                                                   | The key is missing or invalid. Stop and ask the person to check `GRAPHOPS_API_KEY`.                                                                                                                                                                    |
| `403` with `credits_exhausted` or `usage_access_paused` | Stop and tell the person. Don't retry.                                                                                                                                                                                                                 |
| `403` with another code                                 | Read the code's entry in [Error reference](https://docs.graphops.xyz/platform/errors.md).                                                                                                                                                              |
| `429`                                                   | `request_rate_limited`: wait for `Retry-After`, then retry with backoff and jitter. The limit counts the whole organization, so reduce request rate if it repeats. See [Request rate](https://docs.graphops.xyz/rpc/usage-and-limits.md#request-rate). |
| Timeout or response too large                           | Narrow the block range or the request, then retry.                                                                                                                                                                                                     |
| JSON-RPC `method not found`                             | The method isn't supported on this network. Check [Methods](https://docs.graphops.xyz/rpc/methods.md).                                                                                                                                                 |

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
