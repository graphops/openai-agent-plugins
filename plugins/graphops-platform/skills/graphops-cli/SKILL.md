---
name: graphops-cli
description: Act on the GraphOps platform for a signed-in person with the GraphOps CLI (graphops), and run code with stored GraphOps API keys through graphops exec. Use when the graphops command is installed and a task deploys, stages or promotes a Managed Subgraph, manages Endpoint Aliases, API keys or service accounts, reads usage, credits or billing, or needs a GraphOps API key for code that runs on its own.
metadata:
  version: 5fa5813031e34e5f
  source: https://docs.graphops.xyz/.well-known/agent-skills/graphops-cli/SKILL.md
---

# GraphOps CLI

`graphops` is the command-line interface to the GraphOps platform. You use it as the person who
signed it in: their organization roles apply, the platform refuses what they can't do, and every
change is recorded in their name. It never prompts, prints one JSON document with `-o json`, and
exits with a code you can act on.

## Before you start

- Check that `graphops` is on the `PATH`. If it isn't, tell the person what it adds (platform
  actions, and API keys that never enter your transcript) and continue without it. Never install
  it or any other software yourself.
- Run `graphops -o json agent-skills check`. Exit 1 means installed GraphOps skills are out of date
  or a new one is published. Tell the person, and suggest `graphops agent-skills update` for
  out-of-date skills or the command in `installCommands` for new ones. Don't update or install your
  own instructions in the middle of a task.
- Run `graphops -o json auth status`. When `valid` is `false` and `tokenSource` is
  `environment`, `GRAPHOPS_TOKEN` holds a rejected token that takes precedence over any sign-in:
  ask the person to replace or unset it. Otherwise, when `valid` is `false`, run
  `graphops auth login`, show the person the sign-in address and code it prints, and wait for it
  to finish. See [Sign in to the CLI](https://docs.graphops.xyz/platform/cli/sign-in.md).
- Pick the organization explicitly: `graphops -o json orgs list`, then pass
  `--organization <id-or-slug>` to every command instead of relying on the stored selection.
  `graphops -o json --organization <id> orgs show` describes its plan, credits and access.

## Rules

- Always pass `-o json` before the command, as in `graphops -o json subgraphs list`, and act on
  the exit code and `error.code`, not on the wording of a message.
- Stop on exit 3 (sign-in) or 5 (the platform refused) and tell the person. Don't retry them.
- Never run `graphops auth token`, `graphops substreams env`, `key show --reveal` or `--reveal`
  on any command, and never use `auth login --with-token`. Never print the environment inside
  `graphops exec`, and never put a credential in a file, a message or a command line.
- Ask the person before any command marked "ask first" in the reference below. Changes include
  `subgraphs deploy`, `subgraphs create`, `promote` and `unstage`, any `key create`, and changes to
  Endpoint Aliases, API keys and service accounts. Ask before Substreams runs beyond a small block
  range: they consume credits for every block.
- A removal also needs `--confirm` with the resource's name or ID. Put it on the command only after
  the person agrees to that specific removal.

## Run code with an API key

When code you write needs to call a GraphOps data service on its own:

1. `graphops -o json <product> key show` (`rpc`, `subgraphs` or `substreams`) says whether a key is
   stored. Exit 4 means none is.
2. With the person's agreement, `graphops --organization <id> <product> key create --name <label>`
   creates one and stores it without printing it.
3. Run the code with `graphops exec --product <product> -- <command> [args…]`. It sets
   `GRAPHOPS_<PRODUCT>_API_KEY` for that process only. Substreams also gets `SUBSTREAMS_API_KEY`;
   add `--network <network>` so it also gets the GraphOps endpoint, and pass that endpoint to the
   tool with `-e "$(graphops substreams endpoint --network <network>)"`. `graphops substreams
   run|gui|sink` supply the key and endpoint themselves.
4. `exec` exits with the command's own code. 125 means `graphops` failed before running it (read
   `error.code`, for example `missing_key`), 126 that the command can't run, and 127 that it wasn't
   found.

The service skills (`graphops-rpc`, `graphops-subgraphs`, `graphops-substreams`) describe how to
call each service with that variable.

## Command reference

Each command lists what it does, whether to ask first, and the permissions a service account
would need. "Person-only" commands refuse service account credentials.

### auth

- `graphops auth login [--with-token] [--open]`: Sign in with a device code, or with an access token read from stdin. _local; person-only._
- `graphops auth status`: Show the profile, organization and session. _read._
- `graphops auth logout`: Sign out of this profile. _local; person-only._
- `graphops auth token [--id-token]`: Print the access token. The API accepts a device session's access token only together with its ID token. _local; person-only._

### orgs

- `graphops orgs list`: List your organizations. _read; person-only._
- `graphops orgs use <ORGANIZATION>`: Select the organization for this profile. _local; person-only._
- `graphops orgs show`: Show the organization's plan, credits, API keys and service access. _read; permissions: organization.read._
- `graphops orgs products`: List the data services the organization can use, with their active API key counts. _read; permissions: dataServices.read._

### subgraphs

- `graphops subgraphs deploy [--create] [--replace-next] [--accept-schema-changes] [--cid CID] <NAME> [<GRAPH_ARGS>…]`: Build and deploy with `graph deploy`, then create or stage the Managed Subgraph. _cost: always ask first; permissions: managedSubgraphs.read, managedSubgraphs.create, managedSubgraphs.stage, managedSubgraphs.unstage._
- `graphops subgraphs create <NAME> [--cid CID] [--subgraph-id SUBGRAPH_ID]`: Create a Managed Subgraph from an existing Deployment CID or Graph Network Subgraph ID. _cost: always ask first; permissions: managedSubgraphs.read, managedSubgraphs.create, managedSubgraphs.reconfigure._
- `graphops subgraphs list [--all]`: List Managed Subgraphs with their status. _read; permissions: managedSubgraphs.read._
- `graphops subgraphs status <TARGET>`: Show the current and Next version of a Managed Subgraph. _read; permissions: managedSubgraphs.read._
- `graphops subgraphs history <TARGET>`: List a Managed Subgraph's versions: Next, current, then previous versions, newest first. _read; permissions: managedSubgraphs.read._
- `graphops subgraphs wait <TARGET> [--for synced|promotable] [--timeout TIMEOUT]`: Wait until a Managed Subgraph is synced or its Next version can be promoted. _read; permissions: managedSubgraphs.read._
- `graphops subgraphs promote <TARGET> [--expect-next CID]`: Promote the Next version. _change: ask first; permissions: managedSubgraphs.read, managedSubgraphs.promote._
- `graphops subgraphs unstage <TARGET>`: Cancel the staged Next version. _change: ask first; permissions: managedSubgraphs.read, managedSubgraphs.unstage._
- `graphops subgraphs key create [--name NAME] [--replace]`: Create a query key for the organization and store it in this profile. _change: ask first; person-only; needs a recent sign-in; permissions: apiKeys.createQueryKey._
- `graphops subgraphs key set`: Store a key read from stdin, replacing any stored key. _local._
- `graphops subgraphs key show [--reveal]`: Show the stored key ID, organization and last four characters. _local._
- `graphops subgraphs rename <TARGET> <NAME>`: Rename a Managed Subgraph. _change: ask first; permissions: managedSubgraphs.read, managedSubgraphs.rename._
- `graphops subgraphs target-type <TARGET> [--subgraph-id SUBGRAPH_ID] [--current-cid]`: Change a Managed Subgraph's Target type: follow a Graph Network Subgraph ID, or stay on the current Deployment CID. _change: ask first; permissions: managedSubgraphs.read, managedSubgraphs.reconfigure._
- `graphops subgraphs delete <TARGET> --confirm ID_OR_NAME`: Delete a Managed Subgraph. Its indexing stops and it stops being billed. _removal: always ask first; permissions: managedSubgraphs.read, managedSubgraphs.delete._
- `graphops subgraphs browse [--search SEARCH] [--network NETWORK] [--limit LIMIT] [--offset OFFSET]`: Browse published Subgraphs on The Graph Network. _read; permissions: subgraphs.read._
- `graphops subgraphs explore [--subgraph-id SUBGRAPH_ID] [--cid CID]`: Show a Subgraph's name, network and manifest summary. _read; permissions: subgraphs.read._
- `graphops subgraphs metadata [--subgraph-id SUBGRAPH_ID] [--cid CID]`: Show a Deployment CID's or Graph Network Subgraph ID's network, start block and spec version. _read; permissions: subgraphs.read._
- `graphops subgraphs schema [--subgraph-id SUBGRAPH_ID] [--cid CID]`: Print a deployment's GraphQL schema (SDL). _read; permissions: subgraphs.read._
- `graphops subgraphs entities [--subgraph-id SUBGRAPH_ID] [--cid CID]`: List a deployment's entities and enums with their fields. _read; permissions: subgraphs.read._
- `graphops subgraphs manifest [--subgraph-id SUBGRAPH_ID] [--cid CID]`: Print a deployment's manifest YAML as published. _read; permissions: subgraphs.read._

### substreams

- `graphops substreams run [<ARGS>…]`: `substreams run` with the GraphOps endpoint and API key. _cost: always ask first._
- `graphops substreams gui [<ARGS>…]`: `substreams gui` with the GraphOps endpoint and API key. _cost: always ask first._
- `graphops substreams sink <KIND> [<ARGS>…]`: `substreams sink noop|protojson|webhook` with the GraphOps endpoint and API key. _cost: always ask first._
- `graphops substreams env [--network NETWORK]`: Print shell exports for standalone sinks and other Substreams tools. _read._
- `graphops substreams endpoint [--network NETWORK]`: Print the endpoint for a network, or every supported network. _read._
- `graphops substreams key create [--name NAME] [--replace]`: Create a query key for the organization and store it in this profile. _change: ask first; person-only; needs a recent sign-in; permissions: apiKeys.createQueryKey._
- `graphops substreams key set`: Store a key read from stdin, replacing any stored key. _local._
- `graphops substreams key show [--reveal]`: Show the stored key ID, organization and last four characters. _local._
- `graphops substreams packages list [--network NETWORK]`: List popular Substreams packages on supported networks. Needs no sign-in. _read._
- `graphops substreams packages show <SLUG> <VERSION>`: Show a package's modules, output types and network. _read; permissions: substreams.read._

### rpc

- `graphops rpc key create [--name NAME] [--replace]`: Create a query key for the organization and store it in this profile. _change: ask first; person-only; needs a recent sign-in; permissions: apiKeys.createQueryKey._
- `graphops rpc key set`: Store a key read from stdin, replacing any stored key. _local._
- `graphops rpc key show [--reveal]`: Show the stored key ID, organization and last four characters. _local._

### exec

- `graphops exec --product rpc|subgraphs|substreams… [--network NETWORK] <COMMAND>…`: Run a command with stored API keys in its environment, and nowhere else. _local._

### api-keys

- `graphops api-keys list [--all]`: List the organization's API keys: query keys and service account API keys. _read; permissions: apiKeys.read._
- `graphops api-keys show <ID>`: Show one API key: its products and limits, or its service account. _read; permissions: apiKeys.read, managedSubgraphs.read._
- `graphops api-keys create [--name NAME] [--product PRODUCT…] [--grants FILE] [--reveal] [--secret-file FILE]`: Create a query key for one or more data services. _change: ask first; person-only; needs a recent sign-in; permissions: managedSubgraphs.read, apiKeys.createQueryKey._
- `graphops api-keys update <ID> [--product PRODUCT…] [--grants FILE]`: Replace a query key's products and limits. The key and its secret stay the same. _change: ask first; person-only; needs a recent sign-in; permissions: managedSubgraphs.read, apiKeys.reconfigure._
- `graphops api-keys rename <ID> <NAME>`: Rename an API key. _change: ask first; permissions: apiKeys.rename._
- `graphops api-keys rotate <ID> --confirm ID [--name NAME] [--reveal] [--secret-file FILE]`: Replace an API key with a new secret. The old key stops working at once. _removal: always ask first; person-only; needs a recent sign-in; permissions: apiKeys.rotate._
- `graphops api-keys revoke <ID> --confirm ID`: Revoke an API key. It stops working at once; revoking cannot be undone. _removal: always ask first; permissions: apiKeys.read, apiKeys.revoke._

### service-accounts

- `graphops service-accounts list [--all]`: List the organization's service accounts. _read; permissions: serviceAccounts.read._
- `graphops service-accounts show <ACCOUNT>`: Show a service account with its permissions and keys. _read; permissions: serviceAccounts.read, managedSubgraphs.read._
- `graphops service-accounts create <NAME> [--description DESCRIPTION] [--grant PERMISSION…] [--grants FILE]`: Create a service account with the permissions it acts with. _change: ask first; person-only; needs a recent sign-in; permissions: managedSubgraphs.read, serviceAccounts.create._
- `graphops service-accounts update <ACCOUNT> [--name NAME] [--description DESCRIPTION] [--clear-description] [--add-grant PERMISSION…] [--remove-grant PERMISSION…] [--grants FILE]`: Change a service account's name, description or permissions. _change: ask first; person-only; needs a recent sign-in; permissions: serviceAccounts.read, managedSubgraphs.read, serviceAccounts.reconfigure._
- `graphops service-accounts delete <ACCOUNT> --confirm ID_OR_NAME`: Delete a service account. Its keys are revoked at once, and it is listed as archived. _removal: always ask first; person-only; needs a recent sign-in; permissions: serviceAccounts.read, serviceAccounts.delete._
- `graphops service-accounts keys create <ACCOUNT> --name NAME [--reveal] [--secret-file FILE]`: Create a service account API key (`gsk_…`) for automation such as CI. _change: ask first; person-only; needs a recent sign-in; permissions: serviceAccounts.read, serviceAccounts.createKey._

### aliases

- `graphops aliases list`: List the organization's Endpoint Aliases. _read; permissions: endpointAliases.read._
- `graphops aliases show <ALIAS>`: Show an Endpoint Alias: its target, what it serves and its serving evidence. _read; permissions: endpointAliases.read._
- `graphops aliases create <ALIAS> [--cid CID] [--managed-subgraph MANAGED_SUBGRAPH] [--subgraph-id SUBGRAPH_ID] [--behavior current|next|latest-available|latest]`: Create an Endpoint Alias. _change: ask first; permissions: managedSubgraphs.read, endpointAliases.create._
- `graphops aliases retarget <ALIAS> [--cid CID] [--managed-subgraph MANAGED_SUBGRAPH] [--subgraph-id SUBGRAPH_ID] [--behavior current|next|latest-available|latest] [--force]`: Point an Endpoint Alias at another target. Applications using it switch at once. _change: ask first; permissions: managedSubgraphs.read, subgraphs.read, endpointAliases.read, endpointAliases.retarget, endpointAliases.reconfigure._
- `graphops aliases update <ALIAS> --behavior current|next|latest-available|latest`: Change how an Endpoint Alias follows its Managed Subgraph or Graph Network Subgraph ID. _change: ask first; permissions: endpointAliases.read, endpointAliases.reconfigure._
- `graphops aliases delete <ALIAS> --confirm ALIAS`: Delete an Endpoint Alias. Its query URL stops working at once. _removal: always ask first; permissions: endpointAliases.delete._

### usage

- `graphops usage hourly --from FROM [--to TO] [--by-attribution] [--product rpc|subgraphs|substreams] [--api-key API_KEY] [--resource-type RESOURCE_TYPE] [--resource-id RESOURCE_ID] [--event-type EVENT_TYPE]`: Credits consumed per hour (up to 90 days). _read; permissions: usage.read._
- `graphops usage daily --from FROM [--to TO] [--by-attribution] [--product rpc|subgraphs|substreams] [--api-key API_KEY] [--resource-type RESOURCE_TYPE] [--resource-id RESOURCE_ID] [--event-type EVENT_TYPE]`: Credits consumed per UTC day (up to 730 days). _read; permissions: usage.read._
- `graphops usage windows [--from FROM] [--to TO] [--limit LIMIT] [--offset OFFSET] [--product rpc|subgraphs|substreams] [--api-key API_KEY] [--resource-type RESOURCE_TYPE] [--resource-id RESOURCE_ID] [--event-type EVENT_TYPE]`: Usage windows: the metered records billing settles, one page at a time. _read; permissions: usage.read._

### credits

- `graphops credits show`: Show available, plan and purchased credits, and when plan credits expire. _read; permissions: billing.read._
- `graphops credits ledger [--limit LIMIT]`: List the newest credit ledger entries. _read; permissions: billing.read._

### billing

- `graphops billing show`: Show the plan, invoices, top-ups, automatic top-ups and service access. _read; permissions: billing.read._
- `graphops billing top-ups [--limit LIMIT] [--cursor CURSOR]`: List top-up and plan increase orders, newest first. _read; permissions: billing.read._
- `graphops billing notifications`: Show who receives billing notifications and at which credit thresholds. _read; permissions: billing.read._
- `graphops billing catalog [--public]`: Show the plan's prices and the credits each data service consumes. _read; permissions: billing.read._

### agent-skills

- `graphops agent-skills list`: List the published skills and where each is installed. _local._
- `graphops agent-skills install [<SKILLS>…] [--agent claude-code|codex|cursor|copilot|gemini|opencode|windsurf…] [--project]`: Download skills from the GraphOps docs, verify them and install them for coding agents. _local._
- `graphops agent-skills update`: Reinstall every installed skill that differs from the published one. _local._
- `graphops agent-skills check`: Exit 1 when an installed skill differs from the published one. _local._
- `graphops agent-skills remove [<SKILLS>…] [--agent claude-code|codex|cursor|copilot|gemini|opencode|windsurf…] [--project]`: Remove skills this CLI installed. _local._

### doctor

- `graphops doctor`: Check the configuration, platform, session, credentials and tools. _read._

### Exit codes

| Exit code | Meaning                                                                                                                                                                                                                   |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0         | Success.                                                                                                                                                                                                                  |
| 1         | Failure: network, unexpected API response, wrapped tool failed without an outcome.                                                                                                                                        |
| 2         | Usage error, refused argument, unsupported network, a `--confirm` that does not match (`confirmation_mismatch`).                                                                                                          |
| 3         | Not signed in, sign-in denied or expired, session expired, sign in again for a person-only action (`reauthentication_required`), service account credential not accepted, a person-only command run as a service account. |
| 4         | Missing product key (RPC, Subgraphs or Substreams).                                                                                                                                                                       |
| 5         | The platform refused the operation; `error.code` names it: not found, conflict, schema incompatible, not eligible, quota or credits.                                                                                      |
| 6         | `subgraphs wait` timed out.                                                                                                                                                                                               |
| 125       | `exec` only: the CLI failed before running the command (`error.code` names why, such as `missing_key` or `usage`).                                                                                                        |
| 126       | `exec` only: the command exists but cannot run (`command_not_executable`).                                                                                                                                                |
| 127       | Wrapped tool, or the command `exec` runs, not found (`tool_not_found`).                                                                                                                                                   |

## Keep these instructions current

These instructions are published at https://docs.graphops.xyz/.well-known/agent-skills/graphops-cli/SKILL.md. At the start of a task, fetch that URL and compare
its `metadata.version` with the version of this copy. When they differ, tell the person that
their GraphOps skills are out of date, and follow the published version for the task.
