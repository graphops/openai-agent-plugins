---
name: graphops-subgraphs
description: Query indexed blockchain entities with GraphQL through GraphOps Subgraphs, by Deployment CID, Graph Network Subgraph ID or Endpoint Alias, authenticated with a GraphOps API key. Use when code or a question needs Subgraph data from subgraphs.graphops.xyz, pagination over entities, indexing freshness from _meta, or a choice of which Subgraph identifier an application should query.
metadata:
  version: d11cf3c01a0291c1
  source: https://docs.graphops.xyz/.well-known/agent-skills/graphops-subgraphs/SKILL.md
---

# GraphOps Subgraphs

GraphOps Subgraphs serves GraphQL queries over indexed entities. Use it when a Subgraph already
indexes the data the task needs. If the task needs raw chain state or every block in order, see the
`graphops` skill to choose another service.

## Use the MCP tools when they're connected

If the GraphOps Subgraphs MCP server (`https://subgraphs.graphops.xyz/v1/mcp`) is connected,
answer questions with its tools instead of writing requests: `subgraph_get_meta` for indexing
freshness, `subgraph_get_schema` before writing any query, and `subgraph_query` to run one. Each
result states the block it reflects; report it. A failed call returns an `action`:
`fix_arguments`, `narrow_request`, `retry_later` or `stop`; follow it. Write code against the
endpoints below when the person needs code, not an answer.

## Connect

- Read the key from `GRAPHOPS_SUBGRAPHS_API_KEY`. If it isn't set, ask the person to create a
  Subgraphs key and set the variable; see [API keys](https://docs.graphops.xyz/platform/api-keys.md).
- If the GraphOps CLI is installed, run the code with
  `graphops exec --product subgraphs -- <command>`, which sets the variable for that process only;
  see the `graphops-cli` skill.
- Send a GraphQL `POST` with `Authorization: Bearer $GRAPHOPS_SUBGRAPHS_API_KEY` to one of:
- `https://subgraphs.graphops.xyz/v1/deploymentCID/<DEPLOYMENT_CID>` for one exact version,
- `https://subgraphs.graphops.xyz/v1/subgraphId/<SUBGRAPH_ID>` to follow a Graph Network
  Subgraph as it publishes new versions,
- `https://subgraphs.graphops.xyz/v1/alias/<ALIAS>` for an Endpoint Alias the key's organization
  owns.
- Which identifier an application should use decides how it behaves when a new version is
  published. Read [Identifiers and Endpoint Aliases](https://docs.graphops.xyz/subgraphs/concepts/identifiers-and-aliases.md)
  before choosing one for the person's code.

```sh
curl -sS "https://subgraphs.graphops.xyz/v1/deploymentCID/$DEPLOYMENT_CID" \
  -H "Authorization: Bearer $GRAPHOPS_SUBGRAPHS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query":"{ _meta { block { number hash } hasIndexingErrors } }"}'
```

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
- Retry `429`, `502`, `503` and `504` with exponential backoff and jitter, and cap total retry
  time.

## Leave Managed Subgraphs to the person

Don't create, change, promote or remove Managed Subgraphs or Endpoint Aliases on your own. With
the GraphOps CLI installed and the person's agreement, the `graphops-cli` skill describes how to
deploy, stage, promote and retarget through it as the person. Otherwise they manage those in the
GraphOps platform, and you can explain the steps from
[Managed Subgraphs](https://docs.graphops.xyz/subgraphs/managed-subgraphs.md) and
[Deploy with Graph CLI](https://docs.graphops.xyz/subgraphs/guides/deploy-with-graph-cli.md).

## When a query fails

| Response                                                | What to do                                                                                                                                     |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `401`                                                   | The key is missing or invalid. Stop and ask the person to check `GRAPHOPS_SUBGRAPHS_API_KEY`.                                                  |
| `403` with `credits_exhausted` or `usage_access_paused` | Stop and tell the person. Don't retry.                                                                                                         |
| `404`                                                   | The identifier doesn't resolve, or the Endpoint Alias belongs to another organization. Check the identifier.                                   |
| Subgraph unavailable                                    | No indexer is serving that deployment. Read the matching section of [Troubleshooting](https://docs.graphops.xyz/subgraphs/troubleshooting.md). |
| GraphQL validation errors                               | The query doesn't match the schema. Re-read the schema; don't guess field names.                                                               |
| `429`                                                   | Back off and retry with jitter. Reduce concurrency if it repeats.                                                                              |

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
