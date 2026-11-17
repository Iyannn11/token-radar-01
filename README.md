# Token Radar — entry #1

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/Iyannn11/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/Iyannn11/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

A token launch radar for make.fun's token-radar-01 bounty. It reads a sealed snapshot of every protocol DefiLlama tracks, finds the protocols that still have no token and are not a wrapped or staked product of one that trades, ranks them by what their launch would be worth, and hands in a web dashboard showing the board, a detail view for each protocol, and the steps to farm it.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

Node 22 or newer, standard library only, no network. The program reads the snapshot, writes `out/board.json` (the ranked board), `out/data.js` (the dashboard's data), copies `dashboard/` and `screenshots/` to `out/`, and writes `out/dossier.json` last. Run it twice and the outputs are byte for byte the same.

Open `out/dashboard/index.html` in a browser — it renders with no network, straight from disk.

## The model (radar-2)

```
implied = TVL × category multiple × growth multiplier × maturity multiplier
```

- **Category multiple.** For each category, the median market-cap-to-TVL ratio of the protocols in that category whose token already trades (at least three of them). A category with too few live comps falls back to the global median across all live-token protocols. The multiple is recomputed from the snapshot on every run — no protocol name is ever special-cased, and no multiple is hardcoded. Multiples are clamped to 0.02–2.00.
- **Growth multiplier.** 1 + (7-day TVL change, clamped to −60%…+120%) ÷ 100 × 0.5.
- **Maturity multiplier.** 0.6 + min(age in days, 1095) ÷ 1095 × 0.4. Unknown age counts as 365 days.
- **Coverage.** The board covers the whole watchlist (40 protocols above the $50M floor) plus every other rankable pre-token protocol the snapshot carries — 198 rows in total on the dev snapshot. Live-token protocols are set aside and shown at the market's own value, never mixed into the ranking.

The dashboard states the formula and prints the full calibration table, so the model can be checked from the page itself. Each protocol's detail view breaks its implied value down into the three multipliers and recomputes it in the page, showing that it agrees with `board.json` exactly.

## Farm plans

Every row carries six steps, derived from the protocol's own category, chains, TVL and age — never from a list of names. Each step states what it costs and how long it takes: a fresh wallet, the category's action (supply, stake, bridge, provide liquidity, deposit into a vault, …) with a capital sized by TVL tier, a weekly qualifying action, a hold of 30–60 days by age, a weekly check of the protocol's own rewards page, and a farming log.

## Files

- `main.ts` — the entry program (Node 22, stdlib only).
- `lib/model.ts` — the calibrated model, the board builder, the farm plans, the dashboard data.
- `vendor/makefun-board.ts` — the make.fun dossier library, byte for byte as the kit ships it.
- `dashboard/` — the static page: `index.html`, `app.css`, `app.js`.
- `screenshots/` — the board, a protocol detail view, and a filtered view.
- `LICENSE` — MIT.

## License

MIT — see `LICENSE`.
