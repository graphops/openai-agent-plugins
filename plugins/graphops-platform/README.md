# GraphOps Platform

Deploy and promote Managed Subgraphs, manage Endpoint Aliases, API keys and service accounts, read usage and credits, and run code with stored API keys, through the GraphOps CLI signed in as you.

## Install

```text
codex plugin marketplace add graphops/openai-agent-plugins
codex plugin add graphops-platform@graphops
```

## Set up

This plugin drives the `graphops` CLI, so it needs a shell and works in Codex. Install the CLI yourself first; the plugin never installs software.

The first time a task needs the platform, the agent runs `graphops auth login` and shows you a sign-in address and code. Approve it in your browser. The CLI then acts as you, with your role in each organization, and the platform records every change under your account. The agent asks before any command that deploys, promotes, creates or removes a key or alias, or spends credits. `graphops exec` passes a stored API key to the code the agent runs without the key entering the conversation.

## Contents

- Skill `graphops-cli`, published at https://docs.graphops.xyz/.well-known/agent-skills/graphops-cli/SKILL.md
- Skill `graphops`, published at https://docs.graphops.xyz/.well-known/agent-skills/graphops/SKILL.md

The skills contain instructions and links to the live documentation, never scripts. They are generated from the GraphOps documentation; see https://docs.graphops.xyz/ai-agents for other ways to use GraphOps with AI agents.
