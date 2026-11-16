[![make.fun](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/jihoonbaee/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/jihoonbaee/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01)

# Token Radar

A dashboard that ranks pre-token protocols by implied value — what a protocol's token
would be worth if its record stayed as it is — and shows a reader how to farm each one.
Built for the [token-radar-01 bounty on make.fun](https://make.fun/bounty/token-radar-01).

## What it does

- Reads the sealed DefiLlama snapshot (758 protocols, 40 on the watchlist).
- Ranks every genuine pre-token protocol: the full watchlist plus every genuine
  pre-token protocol below the floor. Wrapped, staked and bridged products of a token
  that already trades are left out — they have no token of their own to launch.
- Prices each row at its own category's market rate: the median market-cap / TVL of
  the live protocols in that category, calibrated from the snapshot at run time
  (market-wide median where a category has fewer than three live comparables).
- Writes a 7-step farm plan per protocol, each step with its cost and time, derived
  from the protocol's category, chains, TVL and age.
- Serves a static dashboard (no network, no framework): a sortable, filterable board,
  a detail view per protocol, a live-token view set apart, and a model sandbox that
  re-ranks the board live.

## The model (radar-2)

```
implied = round( tvl_usd x multiple x growth x maturity x breadth )
multiple = median mcap/TVL of live protocols in the category (min 3 comps),
           else the market-wide live median, clamped to [0.05, 1.00]
growth   = 1 + clamp(change_7d, -60, +120) / 100 x 0.5
maturity = 0.6 + min(age_days, 1095) / 1095 x 0.4   (unknown age: 365 days)
breadth  = 1 + 0.02 x min(chains - 1, 5)
```

No protocol name is special-cased. The same function scores every row, and the
dashboard recomputes each row in the page to prove it.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Node 22.18 or later, standard library only. The program is deterministic: two runs
write the same bytes.

## Layout

- `main.ts` — the entrypoint: snapshot in, board + dashboard + dossier out.
- `lib/model.ts` — the model, the board builder and the farm plans.
- `dashboard/` — the static page (`index.html`, `app.css`, `app.js`).
- `screenshots/` — three screenshots of the dashboard, taken from its own code, stored
  base64-encoded (`*.jpg.b64`, one line each) because the tool this entry was uploaded
  with carries text only; `main.ts` decodes them into `/output/screenshots` byte for
  byte, so the gate and the dossier see the real JPEGs.
- `vendor/makefun-board.ts` — make.fun's dossier library, unchanged from the kit.

## License

MIT. See [LICENSE](LICENSE).
