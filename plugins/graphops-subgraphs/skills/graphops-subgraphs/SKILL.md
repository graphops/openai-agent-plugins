---
name: graphops-subgraphs
description: Query indexed blockchain entities with GraphQL through GraphOps Subgraphs, by Deployment CID, Graph Network Subgraph ID or Endpoint Alias, authenticated with a GraphOps API key. Use when code or a question needs Subgraph data from subgraphs.graphops.xyz, pagination over entities, indexing freshness from _meta, or a choice of which Subgraph identifier an application should query.
metadata:
  version: 02631f8cc54579d5
  source: https://docs.graphops.xyz/.well-known/agent-skills/graphops-subgraphs/SKILL.md
---

# GraphOps Subgraphs

GraphOps Subgraphs serves GraphQL queries over indexed entities. Use it when a Subgraph already
indexes the data the task needs. If the task needs raw chain state or every block in order, see the
`graphops` skill to choose another service.

## Connect

- Read the key from `GRAPHOPS_API_KEY`. If it isn't set, ask the person to create a key with
  Subgraphs access and set the variable; see [API keys](https://docs.graphops.xyz/platform/api-keys.md).
- If the GraphOps CLI is installed, run the code with
  `graphops exec --product subgraphs -- <command>`, which sets the variable for that process only;
  see the `graphops-cli` skill.
- Send a GraphQL `POST` with `Authorization: Bearer $GRAPHOPS_API_KEY` to one of:
- `https://subgraphs.graphops.xyz/v1/deploymentCID/<DEPLOYMENT_CID>` for one exact version,
- `https://subgraphs.graphops.xyz/v1/subgraphId/<SUBGRAPH_ID>` to follow a Graph Network
  Subgraph as it publishes new versions,
- `https://subgraphs.graphops.xyz/v1/alias/<ALIAS>` for an Endpoint Alias the key's organization
  owns.
- Which identifier an application should use decides how it behaves when a new version is
  published. Read [Identifier types](https://docs.graphops.xyz/subgraphs/concepts/identifier-types.md)
  before choosing one for the person's code.

```sh
curl -sS "https://subgraphs.graphops.xyz/v1/deploymentCID/$DEPLOYMENT_CID" \
  -H "Authorization: Bearer $GRAPHOPS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query":"{ _meta { block { number hash } hasIndexingErrors } }"}'
```

## Use the MCP tools when they're connected

If the `graphops-subgraphs` MCP server is connected, explore a Subgraph with its tools instead of
writing code. Code the application runs still calls the endpoint above.

- Name the Subgraph with `target`, holding exactly one of `deploymentId`, `subgraphId` or `alias`.
  Every result's `deployment` is the Deployment CID that answered.
- Call `subgraph_get_schema` before writing a query. Its `note` explains the filter, ordering and
  `block` arguments the compact schema leaves out.
- `subgraph_get_meta` returns the indexed block and `hasIndexingErrors`.
- `subgraph_query` takes one `query` operation, without introspection, and returns `data`, `errors`,
  `indexedHead` (the block the data reflects) and, for full pages ordered by `id`, `pages` with the
  next page's arguments.
- GraphQL `errors` come back beside `data`, not as a tool error. A tool error's `action` says
  whether to fix the arguments, ask for less, retry later or stop.

The bounds below still apply. [MCP server](https://docs.graphops.xyz/subgraphs/mcp.md) is the reference for every
tool.

## Make bounded, fresh queries

- Learn the schema once with introspection in development, not on every request. See
  [Schema reference](https://docs.graphops.xyz/subgraphs/schema-reference.md).
- Give every list field a `first`, and request only the fields the task needs.
- Page long scans with `orderBy: id` and `where: { id_gt: $lastId }`, not large `skip` values.
  Stop when a page returns fewer rows than `first`. See
  [Paginate results](https://docs.graphops.xyz/subgraphs/guides/paginate-results.md).
- For a consistent multi-page or multi-query answer, pass the same `block: { number: N }` to every
  query.
- Read `_meta { block { number } hasIndexingErrors }` and report the indexed block with the
  answer. If `hasIndexingErrors` is `true`, say that the data may be incomplete. See
  [Sync and reorgs](https://docs.graphops.xyz/subgraphs/concepts/sync-and-reorgs.md).
- A GraphQL response can return `200` with an `errors` array. Check it before using `data`.
- Retry a `429` only after the seconds in its `Retry-After` header. Retry `429`, `502`, `503` and
  `504` with exponential backoff and jitter, and cap total retry time.

## Leave Managed Subgraphs to the person

Don't create, change, promote or remove Managed Subgraphs or Endpoint Aliases on your own. With
the GraphOps CLI installed and the person's agreement, the `graphops-cli` skill describes how to
deploy, stage, promote and retarget through it as the person. Otherwise they manage those in the
GraphOps platform, and you can explain the steps from
[Managed Subgraphs](https://docs.graphops.xyz/subgraphs/managed-subgraphs.md) and
[Deploy your own Subgraph](https://docs.graphops.xyz/subgraphs/guides/deploy-your-subgraph.md).

## When a query fails

| Response                                                | What to do                                                                                                                                                                                                                                             |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `401`                                                   | The key is missing or invalid. Stop and ask the person to check `GRAPHOPS_API_KEY`.                                                                                                                                                                    |
| `403` with `credits_exhausted` or `usage_access_paused` | Stop and tell the person. Don't retry.                                                                                                                                                                                                                 |
| `404`                                                   | The identifier doesn't resolve, or the Endpoint Alias belongs to another organization. Check the identifier.                                                                                                                                           |
| Subgraph unavailable                                    | No indexer is serving that deployment. Read the matching section of [Troubleshooting](https://docs.graphops.xyz/subgraphs/troubleshooting.md).                                                                                                         |
| GraphQL validation errors                               | The query doesn't match the schema. Re-read the schema; don't guess field names.                                                                                                                                                                       |
| `429`                                                   | `request_rate_limited`: wait for `Retry-After`, then retry with backoff and jitter. The limit counts the whole organization, so reduce query rate if it repeats. See [Query rate](https://docs.graphops.xyz/subgraphs/usage-and-limits.md#query-rate). |

## Pages to read for the task

- First query: [Get started](https://docs.graphops.xyz/subgraphs/getting-started.md)
- Endpoint, request envelope and error format: [API reference](https://docs.graphops.xyz/subgraphs/api.md)
- Finding a Subgraph for a protocol or dataset: [Discover public Subgraphs](https://docs.graphops.xyz/subgraphs/guides/discover-public-subgraphs.md)
- Indexing health: [Monitor sync health](https://docs.graphops.xyz/subgraphs/guides/monitor-sync-health.md)
- Credits and organization limits: [Usage and limits](https://docs.graphops.xyz/subgraphs/usage-and-limits.md)

## Keep these instructions current

These instructions are published at https://docs.graphops.xyz/.well-known/agent-skills/graphops-subgraphs/SKILL.md. At the start of a task, fetch that URL and compare
its `metadata.version` with the version of this copy. When they differ, tell the person that
their GraphOps skills are out of date, and follow the published version for the task.
