# quilt-cells

The quilt cell specification. A seed.

## What is a cell?

A cell is a unit of visible logic. It takes an input, produces an output, and
tells you **why** — in words a human can read — and what it **cost**.

```
INPUT → [CELL] → OUTPUT + WHY + COST
```

- **OUTPUT**: what the cell decided.
- **WHY**: the reason. Not just `0.96` — *"safe because the bilge is dry
  and no one is below deck."* A score without a reason is a black box
  with a number.
- **COST**: milliseconds, megabytes, dollars. Budgeting is visible, not hidden.

## Cells are seeds, not constraints

The cell contract is a starting point for building, not a box to fit into.
If something doesn't fit the contract, the contract bends. Not the thing.

## The prism

Some cells act as prisms: one thing in, many things out — or many in, one out.
A tiny decision model (Jev, a pruned judge) routing a voice command to the
safety cell *and* the action cell *and* the log cell. The prism renders one
cell's output out to the others, or collects many cells into a single
decision.

## Known cells (seeded from experiments, 2026-10)

| Cell | Model | Job |
|------|-------|-----|
| Voice reflex | NeuDecide (42MB) | Audio → tool-call intent |
| Semantic reflex | EmbeddingGemma-2 (166MB) | Any phrasing → canonical intent |
| Judge | Jev / Typesafe | "Is this safe?" — the prism |
| Decomposer | Taiga-S1 (1.2M params) | Goal → ordered cell tasks |
| Operator | Mesa-S1 (1.2M params) | Reads the Quilt's own UI |
| Forecaster | TimesFM 3.0 (330M) | History → forecast + quantiles |
| Separator | Nemotron-3 (100M) | "Who spoke when" — source separation |
| Transcriber | MuScriptor (103M–1.4B) | Audio → MIDI tokens |
| Renderer | T5-base (220M) | Context → "go, *because*..." |

## License

MIT
