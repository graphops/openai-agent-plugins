# GraphOps Subgraphs

Query indexed blockchain entities with GraphOps Subgraphs: read schemas, freshness and query results through the Subgraphs MCP server, and write paginated GraphQL code.

## Install

```text
codex plugin marketplace add graphops/openai-agent-plugins
codex plugin add graphops-subgraphs@graphops
```

## Set up

Set `GRAPHOPS_API_KEY` to a GraphOps query key that includes Subgraphs in the environment where Codex and your code run, or run code with `graphops exec` from the GraphOps CLI, which sets it from your organization's stored key. Create a key for the environment your agent works in, with a name that says so, so you can revoke it without affecting your applications. The skill reads the key from there and never asks you to paste it into the conversation.

Codex plugins can't pass an API key to an MCP server, so connect the server yourself. It reads the query key from `GRAPHOPS_API_KEY` in Codex's environment, and its requests use credits like any other request. Use a key without limits on networks or Subgraphs: the MCP server refuses a key with limits, because a tool call names its target in its arguments.

```text
codex mcp add graphops-subgraphs --url https://subgraphs.graphops.xyz/v1/mcp --bearer-token-env-var GRAPHOPS_API_KEY
```

To have the GraphOps CLI supply the key instead of setting the variable yourself, start Codex with `graphops exec --product subgraphs codex`. It sets `GRAPHOPS_API_KEY` from your organization's stored key for that Codex session only, after checking the key includes the service.

The [Subgraphs MCP server](https://docs.graphops.xyz/subgraphs/mcp) page describes its tools, results, errors and limits, and https://docs.graphops.xyz/ai-agents#query-data-through-mcp covers other agents.

## Contents

- Skill `graphops-subgraphs`, published at https://docs.graphops.xyz/.well-known/agent-skills/graphops-subgraphs/SKILL.md
- Skill `graphops`, published at https://docs.graphops.xyz/.well-known/agent-skills/graphops/SKILL.md

The skills contain instructions and links to the live documentation, never scripts. They are generated from the GraphOps documentation; see https://docs.graphops.xyz/ai-agents for other ways to use GraphOps with AI agents.
