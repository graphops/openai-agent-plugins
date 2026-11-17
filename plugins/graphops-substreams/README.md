# GraphOps Substreams

Stream ordered blockchain data with GraphOps Substreams: run packages, backfill block ranges and build sinks that resume from cursors and handle reorgs.

## Install

```text
codex plugin marketplace add graphops/openai-agent-plugins
codex plugin add graphops-substreams@graphops
```

## Set up

Set `SUBSTREAMS_API_KEY` to a GraphOps Substreams API key in the environment where the `substreams` CLI and your sinks run. The skill reads the key from there and never asks you to paste it into the conversation. With the GraphOps CLI installed, `graphops substreams key create` stores a key and `graphops substreams run` uses it.

## Contents

- Skill `graphops-substreams`, published at https://docs.graphops.xyz/.well-known/agent-skills/graphops-substreams/SKILL.md
- Skill `graphops`, published at https://docs.graphops.xyz/.well-known/agent-skills/graphops/SKILL.md

The skills contain instructions and links to the live documentation, never scripts. They are generated from the GraphOps documentation; see https://docs.graphops.xyz/ai-agents for other ways to use GraphOps with AI agents.
