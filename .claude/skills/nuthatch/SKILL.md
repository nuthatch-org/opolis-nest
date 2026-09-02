---
name: nuthatch
description: Query this self-hosted nuthatch nest on mainnet - decoded events, balances, and read-only SQL. Use when asked about on-chain activity for these contracts.
---

# Querying the nuthatch nest

Contracts indexed on mainnet:
- `opolis_pay` = 0xb25a1f6089b88766434223068d0a4ddcdf011237
- `opolis_pay_v2` = 0xae7db356c82401b111134041533ab490906c8ed5
- `opolis_pay_v3` = 0x22a8fe0109b5457ae5c9e916838e807dd8b0a5b6

Data is local - never call an external API for it.

## Preferred: MCP
If a `nuthatch` MCP server is configured, use its tools. Call `schema` first to learn the
data model, then `sql` / `entity` / `balance` / `top_balances`.

## Fallback: HTTP (a `nuthatch dev` must be running)
- Recent rows:  `curl localhost:8288/entities?limit=20`
- Read-only SQL: `curl -G localhost:8288/sql --data-urlencode 'q=SELECT count(*) FROM transfers'`

`sql` sees finalized data only; balances/entity cover the live tip.
