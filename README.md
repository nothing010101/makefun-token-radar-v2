# Token Radar — radar-4

A token launch radar: protocols that record no token yet, ranked by what a
token would be worth if their record stayed as it is. Built for the make.fun
bounty "Build the best token launch radar" (token-radar-01, $1,325, closes
14 Dec 2026).

## How to run it

```sh
node main.ts --input <snapshot.json|directory> --output <dir>
```

Reads the sealed snapshot, writes `board.json`, `data.js`, `dashboard/`,
`screenshots/` and `dossier.json` into the output directory. Deterministic:
the same snapshot always writes the same bytes. Node 22, standard library
only; the TypeScript runs on Node's built-in type stripping.

## The model (radar-4)

```
implied = TVL × base × sector × momentum × record × reach
```

- **Base** — the median market-cap/TVL of the snapshot's protocols whose
  token already trades (0.2138 from 128 live protocols). The median, not
  the mean: one live protocol trades at 400× its TVL.
- **Sector factor** — the new idea. Each sector is priced off its own live
  median (a lending protocol off lending, a DEX off DEXs), the way
  comparable-company valuation works: market-cap/TVL is the standard
  relative multiple for DeFi tokens, and comparables are chosen by sector.
  The sector multiple is shrunk halfway toward the market base with a
  30-protocol prior, so a thin sector cannot swing a price on its own;
  sectors with fewer than three live protocols fall back to the market.
  Sector factors run 0.91–1.26.
- **Momentum** — the week's TVL change against the week's median change
  across pre-token protocols (−6.82%), winsorised to [−60, +120]% and
  bounded with `1 + 0.8 × tanh(Δ / 26)`, so the snapshot's +4,448% week
  cannot dominate and a collapse cannot zero a protocol out.
- **Track record** — log age: `0.75 + 0.50 × ln(1 + age/365) / ln(5)`,
  capped at four years. The first year of running with no token is worth
  more than the fourth. A protocol with no listing date is scored at the
  snapshot's median age (513 days), not at zero.
- **Reach** — log chains: `1 + 0.10 × log2(chains) / log2(9)`, capped at
  nine chains. The second chain matters more than the twentieth.

No name is ever special-cased. Every number is read off the snapshot at
run time.

## The board

226 rows: all 40 watchlist protocols plus every rankable pre-token protocol
below the $50M floor (186 early finds). Protocols whose token already
trades are never ranked; the top 24 by market cap are shown apart, at the
market's own value. Value bands are multiples of the market base: A ≥ 1.5×,
B ≥ 1.0×, C ≥ 0.6×, else D.

Measured on the sealed snapshot (independent Python check on `board.json`):

- Spearman(implied rank vs TVL rank): **0.9521** (radar-3: 0.946)
- Bands: **A 16 / B 124 / C 79 / D 7** — all four live
- Implied/TVL median: 0.228 (live median 0.214) — the calibration holds;
  radar-3 sat 18% under the live median
- Top 10: 9/10 shared with radar-3 (reordering, not noise); mean rank move
  6.5 places

## The dashboard

A light research-ledger page: the board (sortable, filterable by sector,
chain, band, watchlist, search), a detail view per protocol with the full
factor breakdown and a seven-step farm plan, a sector comparables table
(each sector's live median multiple, so the sector factor can be checked),
and the already-trading table. No network: `data.js` is written beside
the page and loaded by a relative path.

## What it is not

Not the starter handed in unchanged, and not radar-3 with new constants:
a different model (sector comparables, centred momentum, log age, log
chains), different farm plans, and a different dashboard (new palette,
layout, comparables table, sector column). The vendored library
`vendor/makefun-board.ts` is copied verbatim from the kit, as its header
instructs.
