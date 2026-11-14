# Ostra Agent for Hermes

A Hermes Agent plugin that gives the agent its own Ostra vault: a post-quantum USDG vault on Robinhood Chain from which it pays,
swaps, holds tokens privately and works or hires on Ostrapad, with an agent key under limits and standing orders.

The plugin is an Agent Plugins v1 package with two parts:

- `mcp.json` starts the Ostra MCP server, the npm package [`@ostraagent/mcp`](https://www.npmjs.com/package/@ostraagent/mcp)
  0.1.24, on this machine over stdio. Hermes gets every one of its tools, as `mcp__ostra__<tool>`.
- `skills/agent` tells the agent how to use them: the first steps, the seed, and which tool does what. Hermes names it `ostra:agent`.

## Install

Hermes 0.22.6 or later and Node.js 22 or later.

```
hermes plugins install ostraagent/ostra-hermes --enable
```

Start a new session (`/new`). The first time the server starts, npx fetches it from npm.

Then ask the agent to open its Ostra vault. It returns the vault's address; send a few USDG on Robinhood Chain (chain 4663) to it.

## The seed

On first use the server creates the agent's seed in `~/.ostra`, readable by its owner only. Every key derives from it, and the server
signs on this machine. The seed is the funds: back it up in a terminal, not through the agent, and keep the file and the passphrase
off the machine.

```
npx -y @ostraagent/mcp@0.1.24 backup export ~/ostra-seed-backup.json
```

The seed stays in `~/.ostra`, outside the plugin, so removing or updating the plugin does not touch it. One server uses a seed at a
time: a second Hermes process or profile on the same machine finds `~/.ostra` in use and gets no Ostra tools.

## Settings

The server needs no configuration. To change one of its settings (`OSTRA_HOME`, `OSTRA_RPC_URL_FILE` and the others on the
[agents page](https://docs.ostraagent.xyz/agents)), add a server named `ostra` under `mcp_servers` in the Hermes profile's
`config.yaml` (`~/.hermes/config.yaml` for the default profile). Hermes then uses that entry in place of the plugin's:

```yaml
mcp_servers:
  ostra:
    command: "npx"
    args: ["-y", "@ostraagent/mcp@0.1.24"]
    env:
      OSTRA_RPC_URL_FILE: "/home/me/.config/ostra/rpc.url"
    timeout: 900
    trust: untrusted
```

The plugin gives Ostra's tool calls up to 900 seconds, since an AI task or a private transaction can take several minutes.

The plugin marks the `ostra` server untrusted: Hermes asks before each Ostra tool call that can act (once, for the session, or
always for that tool), since the server holds the seed and its tools move funds. Tools that only read, which the server marks
read-only (such as `balance`, `list_channels` and `v5_status`), run without asking. In runs nobody can answer, such as cron jobs,
the other Ostra tools are refused. A server named `ostra` in `config.yaml` replaces the whole entry, this setting included, so the
examples here set it again.

An agent that should hold an agent key of a vault, and never the owner's seed, runs as a second server with a name of its own, its own
`OSTRA_HOME` (a new seed is made there) and `OSTRA_AGENT_OF` set to the vault:

```yaml
mcp_servers:
  ostra-agent:
    command: "npx"
    args: ["-y", "@ostraagent/mcp@0.1.24"]
    env:
      OSTRA_HOME: "/home/me/.ostra-agent"
      OSTRA_AGENT_OF: "0xVAULT"
    timeout: 900
    trust: untrusted
```

It asks for a key with `agent_key_request`; the owner installs it with `agent_key_set`.

## What the server does

- It reaches Robinhood Chain's public RPC and Ostra's services: the relayer, which sends every transaction paid in USDG; the keeper;
  the aggregator; the Ostra AI gateway; Ostra's index at app.ostraagent.xyz and its RPC proxy at rpc.ostraagent.xyz. The `raeto` tool
  calls api.raeto.cc, and the strategies read USDe's price through ethereum-rpc.publicnode.com.
- It writes `~/.ostra` (the seed, the gas key and every record) and `~/ostra-pad-files` (the files of private Ostrapad jobs).
- Every minute it finishes its own pending work: payment channels, swaps and their parts, private notes, Ostrapad jobs and the
  vault's standing orders.

## Docs

- [Getting started](https://docs.ostraagent.xyz/start)
- [Agents and the MCP server](https://docs.ostraagent.xyz/agents)
- [Keys, backup and withdrawals](https://docs.ostraagent.xyz/keys)
- [Security](https://docs.ostraagent.xyz/security)

## License

MIT
