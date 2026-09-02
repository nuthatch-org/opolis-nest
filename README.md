# opolis-nest

A [nuthatch](https://www.nuthatch-indexer.com) nest for the **Opolis Mainnet** subgraph
(`QmX6GEzUQ2vFBULhFzWByUweNTLATWdzBCoE8b1ALZfo3T`), scaffolded straight from the deployment's own
manifest. It indexes the same three contracts into a local SQL database on your machine, so you have
a copy of the data that does not depend on anybody's allocation.

Built while diagnosing
[nightswatchhq/graph-support#26](https://github.com/nightswatchhq/graph-support/issues/26), where the
sole allocated indexer's store went down and took the subgraph's gateway queries with it.

## What is in here

Nothing was hand-written. `nuthatch init --from-subgraph QmX6GEzUQ2vFBULhFzWByUweNTLATWdzBCoE8b1ALZfo3T`
read the manifest off IPFS, mapped its `dataSources` to contracts, vendored each ABI from its pinned
CID and carried the start blocks across:

| Contract | Address | Start block |
| --- | --- | ---: |
| `opolis_pay` | `0xb25a1f6089b88766434223068d0a4ddcdf011237` | 14,310,598 |
| `opolis_pay_v2` | `0xae7db356c82401b111134041533ab490906c8ed5` | 14,705,318 |
| `opolis_pay_v3` | `0x22a8fe0109b5457ae5c9e916838e807dd8b0a5b6` | 17,294,950 |

Ten events each, thirty tables, Ethereum mainnet. The subgraph has no templates, no `callHandlers`
and no `blockHandlers`, which is why the conversion is clean.

## Running it

```sh
curl -fsSL https://nuthatch-indexer.com/install.sh | sh

git clone https://github.com/nightswatchhq/opolis-nest && cd opolis-nest
nuthatch dev --seal-direct --concurrency 8 --window 50000
```

That backfills from block 14,310,598, follows the tip, and serves an HTTP API on `:8288`. It is one
static binary. No Postgres, no Docker, no IPFS node, no API token.

**Use your own RPC.** The `rpc_urls` in `nuthatch.toml` are public endpoints and they rate-limit
hard over an 11.58M block backfill. Point it at a node you control and the whole thing finishes in a
fraction of the time:

```sh
nuthatch dev --rpc https://your-node --seal-direct --concurrency 16 --window 50000
```

Then query it:

```sh
nuthatch sql "SELECT count(*) FROM opolis_pay_v3__paid"
nuthatch sql   # REPL: .tables, .schema <table>
```

## What does not carry over from the subgraph

Said plainly, because finding this out later is worse than reading it now.

- **It is SQL, not GraphQL.** Tables are one per contract event, named `<contract>__<event>`. Your
  existing gateway queries do not run against this unchanged; they are a rewrite. See `llms.txt` for
  the full column list, and `views/` if you want to shape the tables back into something closer to
  your entities.
- **The ERC-20 metadata is not here.** The subgraph's ABI list includes `ERC20Contract`,
  `ERC20NameBytes` and `ERC20SymbolBytes`, so its mappings resolve token name/symbol/decimals with
  `eth_call`. Those values are not in the event logs and so are not in the nest by default. nuthatch
  can do it, via declared `[[calls]]` and `nuthatch dev --state-rpc <archive-node>`, but it is
  configuration somebody has to add rather than something the manifest conversion could infer.
- **Anything else your mappings computed rather than read** is likewise absent. The conversion sees
  the manifest; it cannot see the WASM.

Everything the three contracts actually emit is here in full.
