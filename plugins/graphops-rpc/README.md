# GraphOps RPC

Read EVM and Bitcoin chain data with GraphOps RPC: query blocks, balances, contract calls and event logs through the RPC MCP server, and write bounded, recoverable JSON-RPC code.

## Install

```text
codex plugin marketplace add graphops/openai-agent-plugins
codex plugin add graphops-rpc@graphops
```

## Set up

Set `GRAPHOPS_RPC_API_KEY` to a GraphOps RPC API key in the environment where Codex and your code run. Create one for the environment your agent works in, with a name that says so, so you can revoke it without affecting your applications. The skill reads the key from there and never asks you to paste it into the conversation.

Codex plugins can't pass an API key to an MCP server, so connect the server yourself. It reads the key from the same variable, and its requests use credits like any other request:

```text
codex mcp add graphops-rpc --url https://rpc.graphops.xyz/v1/mcp --bearer-token-env-var GRAPHOPS_RPC_API_KEY
```

## Contents

- Skill `graphops-rpc`, published at https://docs.graphops.xyz/.well-known/agent-skills/graphops-rpc/SKILL.md
- Skill `graphops`, published at https://docs.graphops.xyz/.well-known/agent-skills/graphops/SKILL.md

The skills contain instructions and links to the live documentation, never scripts. They are generated from the GraphOps documentation; see https://docs.graphops.xyz/ai-agents for other ways to use GraphOps with AI agents.
