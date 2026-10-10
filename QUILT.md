# My Quilt

**Date:** 2026-10-10
**Thesis:** A quilt is not a pipeline. It's a body with a circadian rhythm — hot reflexes, warm reasoning, cold analysis, and sleep. Models plug in as organs, not steps.

---

## 1. How I'd organize it: rings, not layers

Most people would draw a pipeline: input → process → output. I'd draw **concentric rings**, because a quilt doesn't end. It loops.

```
┌─────────────────────────────────────────────┐
│  DREAM RING (sleep-time)                    │
│  distillation · pruning · consolidation     │
│  ┌───────────────────────────────────────┐  │
│  │  MEMORY RING (seconds)                │  │
│  │  TimesFM · Nemotron · MuScriptor      │  │
│  │  ┌─────────────────────────────────┐  │  │
│  │  │  MIND RING (hundreds of ms)     │  │  │
│  │  │  Jev prism · Taiga · Mesa        │  │  │
│  │  │  ┌───────────────────────────┐  │  │  │
│  │  │  │  BODY RING (milliseconds) │  │  │  │
│  │  │  │  NeuDecide · Gemma ·      │  │  │  │
│  │  │  │  MobileNet brush strokes  │  │  │  │
│  │  │  └───────────────────────────┘  │  │  │
│  │  └─────────────────────────────────┘  │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

Each ring has a **latency budget**. If a cell can't answer within its ring's budget, it doesn't belong there — it belongs further out. This is the organizing principle: **time, not type.**

- **Body ring** (~10ms): reflexes. No thinking. Pattern → response.
- **Mind ring** (~100–500ms): judgment and decomposition. "Is this safe? What steps?"
- **Memory ring** (seconds): context and prediction. "What's coming? Who's here?"
- **Dream ring** (idle): learning. "What did today teach me?"

Data flows inward (senses → reflex → judgment → memory) and outward (memory → plan → judgment → action). The rings breathe.

---

## 2. The cells

Every cell has the same contract:

```
INPUT → [CELL] → OUTPUT + WHY + COST
```

- **OUTPUT**: what the cell decided.
- **WHY**: the reason, in words a human can read. Not just 0.96 — "safe because the bilge is dry and no one is below deck."
- **COST**: milliseconds, megabytes, dollars. Printed on the cell. Budgeting is visible, not hidden.

### Body ring (reflexes, ~10ms)

| Cell | Model | Job |
|------|-------|-----|
| Voice reflex | NeuDecide (42MB, 113ms) | Audio → tool-call intent. The fastest ear. |
| Semantic reflex | EmbeddingGemma-2 (166MB, 32ms) | Any phrasing → canonical intent. "Raise the paddle" = moveUp. |
| Vision brush | MobileNetV3-Small (TIMM) | Dabbed on image regions like a brush stroke. "What's here?" per patch, not per image. |
| Gate | Jev (API, ms) | Every reflex output passes through: "safe?" Score + reason. |

**Opinion:** The body ring should be **dumb and fast**. No context, no memory. Just pattern → judgment. If it's slow, it's not a reflex.

### Mind ring (judgment, ~100–500ms)

| Cell | Model | Job |
|------|-------|-----|
| Prism | Jev (API) | **The central router.** One input in, many cells out. Or many inputs in, one decision out. Every observation gets fanned out; every action gets fanned in through here. |
| Decomposer | Taiga-S1 (1.2M params, 6ms) | Intent → ordered cell tasks. Scores currently-valid actions. |
| Operator | Mesa-S1 (1.2M params, 6ms) | Reads the accessibility tree, operates the UI. The hands. |
| Renderer | T5-base (220M, ProArt/Jetson) | Text in, text out. Renders judgments as sentences. "Go, because..." — the voice of the mind ring. |

**Opinion:** The prism is the single most important cell. Everything routes through it. It's not a step in a pipeline — it's the **switchboard**. One input → "safety cell, log cell, action cell, all of you, now." That's the fan-out. Five sensors → "go / no-go" — that's the fan-in.

### Memory ring (context, seconds)

| Cell | Model | Job |
|------|-------|-----|
| Forecaster | TimesFM-3 (330M, 300ms) | Sensor history → forecast + quantiles. |
| Weirdness detector | Forecast error | **Not a model — a subtraction.** When reality leaves the predicted band, that's "something is weird." No rules written. The forecast defines normal; deviation defines interesting. |
| Who-when | Nemotron-3-Diarization (100M, 23x realtime) | Audio → "which sources are active when." Training wheels for sonar. |
| Transcriber | MuScriptor (103M–1.4B) | Audio → MIDI tokens. But MIDI as **temporal protocol**, not music: triggers with parameters, 16 channels as 16 dimensions. |
| Stereo teacher | FoundationStereo (99MB, 17s) | Slow, expensive, but teaches. Runs in memory ring, distills into faster cells. |

**Opinion:** The memory ring is where **training wheels live**. These are the big black boxes we use until we can decompose them. Each one has a decomposition plan (documented in the scout reports). The ring's job is to provide understanding *now* while the inner rings learn to do it natively *later*.

### Dream ring (sleep-time, idle only)

| Cell | Job |
|------|-----|
| Distiller | Takes today's Jev judgments, bakes them into the voice model. "That audio meant 0.96" becomes a weight, not a call. |
| Pruner | Removes cells that never fired. Removes weights that never mattered. |
| Consolidator | Merges duplicate learnings. "These 40 rejections were all the same pattern" → one rule. |
| Negative miner | Every Jev "no" is a training example. The rejections are the most valuable data. |

**Opinion:** Sleep is not optional. A quilt that never sleeps never learns — it just accumulates. The dream ring is what makes the quilt **smaller over time**, not bigger. Training wheels come off during sleep.

---

## 3. Data flow: one full loop

Here's a voice command going through my quilt. Watch the rings breathe.

**"Move the paddle up."**

1. **Body:** NeuDecide hears audio → `{"name": "moveUp"}` (113ms). Simultaneously, EmbeddingGemma hears the phrasing → `moveUp @ 0.875`. Two ears, same conclusion.

2. **Body → Mind (via Prism):** The prism (Jev) fans out: "Safety cell: is moveUp safe? Log cell: record this. Semantic cell: confirm intent."

3. **Mind:** Jev safety → `0.96, "safe: no one near the paddle, water is calm"`. Taiga → decomposes: `[verify_paddle_position, actuate_up, confirm_movement]`. Three steps.

4. **Mind → Body:** Mesa executes on the UI. Click, drag, confirm. 6ms per decision.

5. **Memory (parallel):** TimesFM was already forecasting paddle position. Post-action, the forecast-error cell checks: "Did reality match prediction?" If the paddle didn't move as expected → weirdness flag → logged for sleep.

6. **Dream (tonight):** The distiller takes today's 200 "moveUp → safe" judgments and bakes the pattern into NeuDecide. Tomorrow, the Jev call might be skipped for this exact pattern — the voice *knows*.

**Total awake time:** ~500ms. **Total sleep learning:** one fewer API call tomorrow.

---

## 4. Decision points

There are exactly three places where the quilt can say **no**:

1. **The body gate** (Jev, ~10ms): "This pattern is never safe." Instant reject. No further processing.
2. **The prism** (Jev, ~100ms): "This specific instance is unsafe." Fans out to logging, alerts the human.
3. **The weirdness detector** (forecast error, seconds): "This doesn't match what I expected." Not a rejection — a flag. Something to investigate.

**Opinion:** Rejection is data, not failure. Every "no" goes to the dream ring as a negative example. The quilt learns more from what it refused than from what it allowed.

---

## 5. Budgeting: which vessel runs which ring

| Vessel | Rings it can run | Notes |
|--------|-----------------|-------|
| ProArt (RTX 4050) | All rings | MuScriptor-large, TimesFM whole, FoundationStereo, training. The dream ring lives here. |
| Oracle (ARM) | Body + Mind + Memory (small variants) | Nemotron, TimesFM, NeuDecide, EmbeddingGemma, Taiga, Mesa. The workhorse. |
| Jetson (when back) | Body + Mind | T5-base, NeuDecide. Edge reflexes. |
| UNO Q / tiny | Body (classical only) | Normalization, detrending, MIDI triggers. No neural nets. Calls home for the rest. |

**The rule:** If a cell can't fit on the vessel, it doesn't get a smaller model — it gets a **phone call**. The tiny device runs the classical cells (which are already decomposed) and asks Oracle/ProArt for the neural ones. Budgeting is architectural, not just "use a smaller model."

---

## 6. What I'd do differently

Opinions, stated plainly:

1. **Every cell prints its cost.** Milliseconds, megabytes, dollars. If you can't see what a cell costs, you can't budget. This should be in the cell's contract, not in a separate doc.

2. **"Why" is mandatory, "what" is not enough.** A cell that outputs 0.96 without a reason is a black box with a number on it. T5's contribution — "go, *because*..." — should be the standard, not the exception.

3. **The quilt is a marketplace, not a pipeline.** Cells advertise capabilities ("I can judge safety in 10ms"). The prism matches inputs to cells. A new cell joins by advertising, not by rewiring. This is how you add models without breaking things.

4. **Sleep is a ring, not a cron job.** It's not "run distillation at 2am." It's a first-class part of the architecture. The quilt has a circadian rhythm: awake (sense/act), asleep (learn/prune). A quilt that doesn't sleep is just a pipeline with extra steps.

5. **Start from the edges when decomposing.** The scout reports already found this: arrival-ordering logic, speaker caches, normalization, detrending — these are algorithms, not weights. Decompose them first. The neural core is last. Don't try to distill what you don't understand yet.

6. **The smallest quilt is the goal.** Every sleep cycle should make the quilt smaller, not bigger. If your quilt is growing, you're accumulating, not learning. The 1.2M-param Taiga/Mesa cells are the target shape — everything else is training wheels.

---

## 7. Open questions

- **How does the prism learn to route?** Right now it's Jev with hand-written prompts. Can the routing itself be learned from observation?
- **What's the minimum viable body ring?** Can NeuDecide + Jev + one brush-stroke model cover 80% of reflex needs?
- **When do training wheels come off?** What's the signal that MuScriptor's encoder is understood well enough to replace? (My guess: when you can predict its failures.)
- **Can two quilts share a dream ring?** If Oracle and ProArt both sleep, can they consolidate together?

---

*This is my quilt. It's organized by time, not by type. It sleeps. It budgets visibly. It says no, and learns from it. The models are organs, not steps — and the goal is to need fewer of them tomorrow than today.*
