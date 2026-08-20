# Wick &amp; Ledger

A single-file costing and inventory ledger for a small candle studio. One HTML document, no build step, no server, no account. Open it in a browser and it runs; your data lives in a JSON file you save and load yourself.

Built to run [Poe's Apothecary](#), but nothing in it is specific to that shop.

---

## What problem it solves

Most candle makers cost their product once, in a spreadsheet, and never revisit it. Then wax prices move, a jar supplier changes, labor creeps up, and the margin quietly disappears.

Wick &amp; Ledger keeps three things in one place and keeps them honest with each other:

- **What a candle costs to make** — broken into wax, fragrance, wick, dye, container, packaging, labor, and overhead.
- **What you actually have** — raw materials, work in progress, and finished candles, each moving at the moment the real-world event happens.
- **What you actually sold it for** — recorded against the cost of the specific candles that left the shelf, so margin is history rather than hope.

## Features

**Costing.** Every recipe produces a per-candle cost and a suggested price from a markup multiplier. Wax quantity comes either from the container's volume and a fill ratio, or from a straight weight you enter. Fragrance load is a percentage of wax weight. Overhead and markup can be overridden per recipe when one product doesn't behave like the rest.

**The melt bar.** Each recipe renders its cost as a single stacked bar, one segment per component, each segment a different engraved fill rather than a different color. It answers "where is the money going" at a glance and survives being printed in black and white.

**Bulk cost entry.** Enter what a 50 lb case cost and how much came in it; the per-ounce figure falls out. Or enter the unit cost directly. Either way the recipe reads the same number.

**Batch scaling, three ways.** Size a run by candle count, by total wax you intend to pour, or by *max from stock* — which walks every requirement, divides by what's on the shelf, and tells you which material runs out first.

**Two-step inventory.** Starting a batch deducts materials immediately, because the wax is gone the moment it's in the pot. Candles only reach finished stock when you mark the batch complete. Cancelling an open batch returns everything to the shelf. This split is deliberate: it keeps poured wax out of raw inventory without pretending the candles already exist.

**Sales ledger.** Log a sale against finished stock and it deducts units, records the price you actually charged, and snapshots the unit cost of those candles at that moment. Rolls up into revenue, cost of goods, margin, and sell-through per recipe, plus revenue by channel. Sales can be reversed. Write-offs — breakage, samples, gifts — leave stock without touching revenue, so they can't flatter your margin.

**Low stock warnings.** Optional per-item threshold, surfaced on the dashboard.

## Getting started

1. Download `wick-and-ledger.html`.
2. Open it in any modern browser. There is nothing to install.
3. Add a container, then your wax and fragrance, then build a recipe.
4. **Click Save file.** The app does not autosave and does not use browser storage — if you close the tab without saving, the work is gone. It will warn you.

Load the JSON back with **Load file** to pick up where you left off.

Keeping the saved ledger in a synced folder or committing it to a private repo gives you version history for free.

## The data file

One JSON document. Human-readable, diff-able, and yours.

```
{
  "schema": 3,
  "settings":    { laborRate, minutesPerCandle, overhead, markup, fillRatio },
  "containers":  [ { id, name, volume, cost, stock, lowAt, notes } ],
  "ingredients": [ { id, name, category, costMode, unitCost | bulkQty + bulkCost, stock, lowAt } ],
  "recipes":     [ { id, name, containerId, fillMode, fillRatio | waxOz, waxId,
                     fragranceId, fragrancePct, dyeId, dyeAmt, wicks[], packaging[],
                     minutes, overheadOverride, markupOverride } ],
  "batches":     [ { id, recipeId, count, status, startedAt, completedAt, unitCost, lines[] } ],
  "finished":    [ { recipeId, qty, totalCost } ],
  "sales":       [ { id, recipeId, qty, unitPrice, unitCost, channel, date, notes } ]
}
```

Older files load without complaint — missing fields are backfilled with current defaults on import.

## Two numbers to check before trusting the output

**Fill ratio** (default `0.86`) is ounces of wax by weight per fluid ounce of container volume. That figure is typical for soy in glass, but it varies by wax, vessel shape, and how high you fill. Pour one test candle, weigh the wax, and set the real number — every cost in the app scales off it.

**Minutes per candle** (default `6`) drives the labor line. Time yourself through one honest batch including cleanup, not just the pour.

## Under the hood

Vanilla JavaScript, no dependencies, no bundler. State lives in a single object; views re-render from it. Roughly 2,400 lines including styles.

The test suite (`test.js`, jsdom) runs the full workflow — container through ingredient, recipe, batch, completion, sale, reversal, write-off, save/load round trip, and legacy file import — and asserts the arithmetic at each step.

```bash
npm install jsdom
node test.js
```

## Known limitations

- Reversing a sale returns candles at their original cost. If batches have completed in between, the running average shifts slightly. Fine in normal use, imprecise if you undo a sale weeks later.
- Finished stock uses a weighted average, not FIFO or lot tracking.
- No multi-currency, no tax handling, no shipping cost allocation.
- Single user, single file. That is the point, not an oversight.

## License

MIT.
