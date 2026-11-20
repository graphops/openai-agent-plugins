# GraphOps RPC

Read EVM and Bitcoin chain data with GraphOps RPC: query blocks, balances, contract calls and event logs through the RPC MCP server, and write bounded, recoverable JSON-RPC code.

## Install

```text
codex plugin marketplace add graphops/openai-agent-plugins
codex plugin add graphops-rpc@graphops
```

## Set up

Set `GRAPHOPS_API_KEY` to a GraphOps query key that includes RPC in the environment where Codex and your code run, or run code with `graphops exec` from the GraphOps CLI, which sets it from your organization's stored key. Create a key for the environment your agent works in, with a name that says so, so you can revoke it without affecting your applications. The skill reads the key from there and never asks you to paste it into the conversation.

Codex plugins can't pass an API key to an MCP server, so connect the server yourself. It reads the query key from `GRAPHOPS_API_KEY` in Codex's environment, and its requests use credits like any other request. Use a key without limits on networks or Subgraphs: the MCP server refuses a key with limits, because a tool call names its target in its arguments.

```text
codex mcp add graphops-rpc --url https://rpc.graphops.xyz/v1/mcp --bearer-token-env-var GRAPHOPS_API_KEY
```

To have the GraphOps CLI supply the key instead of setting the variable yourself, start Codex with `graphops exec --product rpc codex`. It sets `GRAPHOPS_API_KEY` from your organization's stored key for that Codex session only, after checking the key includes the service.

The [RPC MCP server](https://docs.graphops.xyz/rpc/mcp) page describes its tools, results, errors and limits, and https://docs.graphops.xyz/ai-agents#query-data-through-mcp covers other agents.

## Contents

- Skill `graphops-rpc`, published at https://docs.graphops.xyz/.well-known/agent-skills/graphops-rpc/SKILL.md
- Skill `graphops`, published at https://docs.graphops.xyz/.well-known/agent-skills/graphops/SKILL.md

The skills contain instructions and links to the live documentation, never scripts. They are generated from the GraphOps documentation; see https://docs.graphops.xyz/ai-agents for other ways to use GraphOps with AI agents.
