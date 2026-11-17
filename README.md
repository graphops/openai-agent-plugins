# GraphOps plugins for Codex

Agent plugins for GraphOps blockchain data services. They give the agent the instructions to use GraphOps RPC, Subgraphs and Substreams correctly, the docs MCP server, and the GraphOps CLI workflow to manage platform resources. They use the portable Agent Plugins layout.

## Install

```text
codex plugin marketplace add graphops/openai-agent-plugins
```

Then install the plugins your project needs, for example:

```text
codex plugin add graphops-rpc@graphops
```

## Plugins

| Plugin | What it does |
| --- | --- |
| [`graphops`](plugins/graphops) | Choose between GraphOps RPC, Subgraphs and Substreams, handle GraphOps API keys and error codes, and search the GraphOps documentation through its MCP server. |
| [`graphops-rpc`](plugins/graphops-rpc) | Read EVM and Bitcoin chain data with GraphOps RPC: query blocks, balances, contract calls and event logs through the RPC MCP server, and write bounded, recoverable JSON-RPC code. |
| [`graphops-subgraphs`](plugins/graphops-subgraphs) | Query indexed blockchain entities with GraphOps Subgraphs: read schemas, freshness and query results through the Subgraphs MCP server, and write paginated GraphQL code. |
| [`graphops-substreams`](plugins/graphops-substreams) | Stream ordered blockchain data with GraphOps Substreams: run packages, backfill block ranges and build sinks that resume from cursors and handle reorgs. |
| [`graphops-platform`](plugins/graphops-platform) | Deploy and promote Managed Subgraphs, manage Endpoint Aliases, API keys and service accounts, read usage and credits, and run code with stored API keys, through the GraphOps CLI signed in as you. |

## About this repository

Every file here is generated from the GraphOps documentation each time it is deployed, so the skills match https://docs.graphops.xyz exactly. Changes made here directly are overwritten. To report a problem or suggest a change, contact GraphOps at https://graphops.xyz/contact/.
