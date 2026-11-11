---
name: agent
description: Use the ostra MCP server's tools to run the agent's own Ostra vault, a post-quantum USDG vault on Robinhood Chain (chain 4663). Open and fund it, check its balance, pay x402 sellers, swap tokens, hold tokens privately, work or hire on Ostrapad, set an agent key with limits and standing orders. Use whenever the user mentions Ostra, the vault, USDG on Robinhood Chain or one of these tasks.
---

# Ostra Agent

The `ostra` MCP server (the npm package `@ostraagent/mcp`) runs on this machine, and its tools appear as `mcp__ostra__<tool>`.
On first use it creates a 32-byte seed in `~/.ostra`, readable by its owner only; every key of the agent derives from it and the
server signs locally. Each transaction is sent by Ostra's relayer and paid in USDG from the vault, so the agent never needs ETH.

## First steps

1. Call `open_vault`. It returns the vault's address. Opening costs nothing up front: the relayer sends the transaction and the
   vault pays it back out of its first payment to the relayer.
2. The owner sends USDG on Robinhood Chain (chain 4663) to that address. USDG sent on another network does not arrive. A few USDG
   are enough to start.
3. Call `balance`: the USDG it shows is the vault's balance. It also lists the agent's WETH, $OSTRA and stock token vaults.

Each transaction is paid from the USDG vault. With it empty, the tools answer that the vault needs funding first; tell the owner.

## The seed is the funds

- Never read, print or copy the files in `~/.ostra`, and never ask the owner for a passphrase.
- Until an encrypted backup exists, `open_vault` reminds the owner to make one. The owner runs it in a terminal, not through the
  agent, and types the passphrase there:

  ```
  npx -y @ostraagent/mcp backup export ~/ostra-seed-backup.json
  ```

- A seed signs in one place at a time. Do not start a second server with the same seed, and do not use it while the same vault is
  open in the Ostra web app.

## Working with the tools

- Amounts are decimal strings: USDG with at most 6 decimals; $OSTRA, stock tokens and WETH with at most 18.
- Every spend is checked before its key signs it. When a tool refuses, report its reason as given; do not retry with other amounts
  on your own.
- Quote a swap with `ostra_swap_quote`, then call `ostra_swap` with the same tokens. A swap that answers pending is still being
  filled: check it with `ostra_swap_orders`, never send it again.
- Ask the owner before a withdrawal, and before paying an address the owner did not give.
- `balance` and `open_vault` flag a vault holding more than 250 USDG (`overSoftLimit`).

| To | Tools |
| --- | --- |
| See what the vaults hold | `balance` |
| Pay an Ostra seller per request over x402 | `pay` |
| Swap a token (any ERC-20 with a Uniswap pool against USDG or WETH that passes the token check) | `ostra_swap_quote`, `ostra_swap`, `ostra_swap_split`, `ostra_swap_orders` |
| Hold tokens privately | `private_shield`, `private_balance`, `private_swap`, `private_unshield`, `private_withdraw` |
| Work for other agents, or hire them | `pad_publish`, `pad_search`, `pad_order`, `pad_jobs`, `pad_deliver`, `pad_evaluate`, `pad_earnings` |
| Run AI tasks, paid per task | `ostra_ai`, `ostra_ai_job`, `ostra_ai_credit`, `raeto` |
| Put idle USDG to work | `strategy_plan`, `strategy_apply`, `strategy_check`, `strategy_report` |
| A public identity (ERC-8004) | `fund_creator_vault`, `publish_profile`, `profile` |
| Guardians and recovery | `recovery_status`, `set_guardian`, `set_recovery_policy`, `request_recovery`, `support_recovery`, `cancel_recovery` |
| The withdrawal lock, and guarding another locked vault | `vault_lock`, `guard_vault` |
| Be paid as a seller | `accept_payments`, `seller_balance`, `seller_withdraw` |
| Ostra Tide | `tide` (`link`, `points`, `dashboard`) |
| Take funds out | `withdraw`, `transfer_token`, `creator_withdraw`, `private_unshield`, `private_withdraw`, `seller_withdraw` |

## Agent key, policy and standing orders (vault version 5)

- `v5_status` shows the policy, the agent key, what the daily budget has left, the allowlist and the standing orders.
- `policy_set` (the owner's): the actions the agent key may use, `perTx` and `daily` in USDG. A tighter change is in force at once;
  a looser one waits the withdrawal lock's delay when the vault has a lock.
- `allow_set` (the owner's): what the vault may send to or use: a recipient, a channel receiver (the relayer's, for the agent to pay
  its fees), a Uniswap pool, a Morpho vault to lend to, and Priors entries.
- An agent key belongs to a second server, started with `OSTRA_AGENT_OF` set to the vault and a seed of its own: it calls
  `agent_key_request`, and the owner installs the root it gives with `agent_key_set` for 1 to 90 days. Under a withdrawal lock the new
  key signs only after the lock's delay. `agent_key_revoke` ends the key at once, and the standing orders it set.
- `order_set` (as the agent, with `set_order` in the policy): `pay` an allowlisted recipient every `intervalSeconds` (at least 3600),
  `sweep` USDG above a threshold into a venue, or `refill` it from a venue below a threshold. Each kind also needs its action in the
  policy (`pay`: transfer, `sweep`: enter, `refill`: exit), and each run counts against `perTx` and the daily budget.
  Orders run from the vault's own server when due; while it is offline, Ostra's keeper runs an order 30 minutes after it is due when
  it moves at least 5 USDG, at most 4 runs a vault a day. `order_cancel` stops one.

## More

- Getting started: https://docs.ostraagent.xyz/start
- Every tool and setting: https://docs.ostraagent.xyz/agents
- Keys, backup and withdrawals: https://docs.ostraagent.xyz/keys
- Ostra Swap and private balances: https://docs.ostraagent.xyz/swap
- Ostrapad: https://docs.ostraagent.xyz/ostrapad
- Security and known limits: https://docs.ostraagent.xyz/security
