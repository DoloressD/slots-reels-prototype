# Web-Based Flexible Reels Prototype Engine
**Design Doc + Prototype**

---

## 1. Prototype Goal

The engine's job in its simplest useful form is to let a team spin a configurable grid of reels, see a result, and evaluate whether a win condition was met — all without touching core code between prototypes.

The problem it solves is iteration speed. When a designer wants to test a 4x3 layout with a new symbol set, or when leadership wants to see a feature trigger in action, the current alternative is usually rebuilding from scratch or hacking an existing prototype until it breaks. A shared reels engine gives everyone a stable foundation to push against.

Iteration speed matters because it keeps the focus on improving the experience rather than rebuilding infrastructure. Without a shared foundation, prototype work tends to become rushed and inconsistent. Every new idea starts from zero instead of building on something stable!

The engine is not trying to be a shipping game. It is trying to answer one question quickly: *does this idea feel worth developing further?*

---

## 2. System Breakdown

The engine is split into six distinct layers. Keeping these layers separate is what makes the engine reusable.

**Math Layer**
Owns the outcome. Selects symbols from reel strips using weighted or sequential logic, evaluates paylines or win conditions, and returns a structured result object. This layer has no knowledge of the display.

**Config Layer**
A single JSON object that describes everything variable about a game: reel count, row count, reel strips, symbol definitions, paytable, payline patterns, feature trigger rules, and spin behavior settings. Swapping the config file swaps the game.

**Reel Layout**
A data structure that defines the grid — how many reels, how many rows visible per reel, and how symbols map to positions. Rows and reels are both driven by config, not hardcoded.

**Symbol Definitions**
Each symbol has an ID, a display label or asset reference, a weight (used during weighted random selection), and optional metadata flags such as `isWild`, `isScatter`, or `isBonus`. Art assets are swappable without touching logic.

**Spin Behavior**
Controls how the spin animates. Options include: instant (no animation, useful for rapid math testing), staggered (reels stop one at a time), and simultaneous. Duration and easing are configurable per prototype.

**Feature Hooks**
Lightweight callback points that fire on specific events: `onSpinStart`, `onReelStop`, `onResultEvaluated`, `onFeatureTrigger`. These are where prototype-specific behavior — free spins, multipliers, expanding wilds — plugs in without modifying core logic.

**Debug / Testing Panel**
A collapsible overlay showing: the current config name, last spin result as raw JSON, forced result input (to test specific outcomes), spin counter, and a basic win frequency display over N spins.

---

## 3. Flexible Design Approach

Flexibility in a prototype engine comes from one rule: **if you might want to change it between prototypes, it belongs in config, not in code.**

**Different reel sizes**
`reelCount` and `rowCount` are top-level config values. The layout renders dynamically from these numbers. Nothing about the grid is hardcoded.

**Different symbol sets**
Symbols are defined as an array in config. The engine does not know what symbols exist until it reads the config. Adding or removing symbols requires only a config edit.

**Weighted symbols and reel strips**
Each reel can have its own strip — an ordered array of symbol IDs that gets sampled during a spin. Weights can be applied per symbol globally or overridden per reel. Both weighted-random and reel-strip models are supported.

**Different paylines, ways, or evaluation styles**
The result evaluator accepts a strategy parameter. Built-in strategies include `paylines` (fixed line patterns defined in config), `ways` (all adjacent matching symbols count), and `scatter` (symbol appears anywhere). New evaluation strategies are added as standalone functions without touching the core.

**Feature triggers**
A trigger is a condition check registered in config: `{ "type": "scatter", "symbol": "BONUS", "count": 3, "fires": "freeSpins" }`. The engine evaluates triggers after every spin result and fires the registered hook if conditions are met. Prototype-specific feature behavior lives in the hook, not the engine.

**Placeholder and AI-generated art**
Symbol rendering accepts either a CSS color block with a label (the default placeholder), a local image path, or a URL. Swapping from placeholder to real art is a one-line change per symbol in config.

**Fast iteration**
The config is loaded at runtime from a JSON file, not bundled at build time. Changing a config value and refreshing the browser is enough to test a new idea. No rebuild required.

---

## 4. Phased Execution Plan

### Phase 1 — Math Core
**What:** Config loader, reel strip sampler, weighted random selection, basic result object output.
**Why it matters:** Everything else depends on this being correct and cleanly separated from display.
**Proof of usefulness:** Running a function in the browser console that returns `{ positions: [[0,2,1],[1,0,3],[2,1,0]], symbols: [["A","B","C"],["B","A","D"],["C","B","A"]] }` from a config input.
**Left out intentionally:** All rendering, animation, win evaluation.

### Phase 2 — Grid Renderer + Spin Animation
**What:** DOM or Canvas grid that reads layout from config, renders symbols in cells, and plays a configurable spin animation.
**Why it matters:** Stakeholders need to see something spin. A working visual makes the prototype real enough to gather feedback.
**Proof of usefulness:** A 5x3 grid spins and lands on a valid result driven by Phase 1 math.
**Left out intentionally:** Win highlighting, feature triggers, paytable.

### Phase 3 — Win Evaluation + Payline Display
**What:** Evaluator that checks result against config-defined paylines or ways, returns win data, highlights winning lines in the UI.
**Why it matters:** Without win feedback, it is not a slot game — it is a spinning grid. Win display is the first moment the game reads as a game.
**Proof of usefulness:** A winning combination highlights correctly. A non-win spin shows no highlights.
**Left out intentionally:** Payout math, credit balance, sound.

### Phase 4 — Feature Hook System + Debug Panel
**What:** `onFeatureTrigger` callback system, scatter detection, a basic free-spin stub, and the collapsible debug overlay.
**Why it matters:** This is what makes the engine a *prototype tool* rather than a one-game demo. The debug panel lets non-engineers force results and test edge cases without code changes.
**Proof of usefulness:** Forcing 3 scatter symbols via the debug panel fires the feature callback and logs to the panel. Swapping to a new config file changes the layout and symbol set with no code edits.
**Left out intentionally:** Full feature implementations, sound design, credit system, server integration.

---

## 5. AI Usage

AI is a collaborator on this project, not an autocomplete tool. Here is specifically where it helps and where judgment takes over.

**Where AI accelerates:**
- *Architecture planning:* Used to pressure-test the layer separation model. Specifically, prompting with "what breaks if the math layer knows about the DOM?" surfaces coupling risks early.
- *Scaffolding:* Generating the initial config schema, symbol data structure, and evaluator skeleton. AI produces a useful first draft in seconds; the review pass catches assumptions that don't match the design.
- *Sample data:* Generating realistic-looking reel strips, symbol weight distributions, and payline pattern arrays for a 5x3 grid.
- *Debugging:* Pasting unexpected output into a prompt and asking what the result object would look like given specific input — faster than reading through evaluation logic manually.
- *Edge case discovery:* Prompting "what edge cases should a weighted reel strip sampler handle?" reliably surfaces things like zero-weight symbols, empty strips, and single-symbol reels that are easy to miss.
- *Documentation:* Generating config schema docs and inline comments from the working code, rather than writing them during development.

**Where judgment matters more:**
- Deciding what stays in config versus what gets hardcoded. AI will often suggest config options for things that should be constants, and constants for things that will absolutely vary. That call requires understanding how designers and engineers will actually use the tool.
- Evaluating whether Phase 4 is worth building at all given a 2-hour window. AI can't make that tradeoff — it will always say "yes, build it."
- Reviewing generated evaluator logic for correctness. AI-generated win evaluation code often handles the happy path correctly and misses edge cases like wild substitutions on multi-symbol lines.
- Deciding the right visual fidelity for the prototype. AI defaults to either too minimal or too polished; the right level depends on who the audience is.

In conclusion, AI is a great tool to challenge and use, not to blindly trust. I made the decisions on how the prototype should be structured, evaluated the output, and redirected when something was wrong.

---

## 6. Example Implementation Artifact

See the accompanying `index.html` file for a working single-file demo. The demo includes three swappable configs (Classic 3×3, Standard 5×3, Wide 6×4), staggered spin animation, payline evaluation with win highlighting, scatter detection, feature toast notifications, a live debug panel with forced-result input, and win rate tracking — all driven by the schema below with no code changes between configs.

Below is the config schema the demo is built against. Any prototype built on this engine starts by filling out a config object in this shape:

```json
{
  "name": "Classic 5x3",
  "reelCount": 5,
  "rowCount": 3,
  "spinBehavior": {
    "mode": "staggered",
    "durationMs": 600,
    "staggerDelayMs": 120
  },
  "symbols": [
    { "id": "A", "label": "♠", "weight": 5, "isWild": false },
    { "id": "B", "label": "♥", "weight": 5, "isWild": false },
    { "id": "C", "label": "♦", "weight": 4, "isWild": false },
    { "id": "D", "label": "♣", "weight": 4, "isWild": false },
    { "id": "E", "label": "★", "weight": 2, "isWild": false },
    { "id": "W", "label": "🃏", "weight": 1, "isWild": true },
    { "id": "S", "label": "⚡", "weight": 1, "isScatter": true }
  ],
  "reelStrips": [
    ["A","B","C","A","D","B","W","A","C","E","B","S","D"],
    ["B","A","D","C","B","A","E","D","C","B","A","W","C"],
    ["C","D","A","B","C","D","A","S","B","C","D","A","E"],
    ["A","B","C","D","E","A","B","C","W","A","B","D","C"],
    ["D","C","B","A","D","C","B","A","S","D","C","B","E"]
  ],
  "paylines": [
    [1,1,1,1,1],
    [0,0,0,0,0],
    [2,2,2,2,2],
    [0,1,2,1,0],
    [2,1,0,1,2]
  ],
  "paytable": {
    "A": [0, 0, 5, 20, 50],
    "B": [0, 0, 5, 20, 50],
    "C": [0, 0, 8, 30, 80],
    "D": [0, 0, 8, 30, 80],
    "E": [0, 0, 15, 60, 150],
    "W": [0, 0, 25, 100, 500]
  },
  "featureTriggers": [
    {
      "type": "scatter",
      "symbol": "S",
      "count": 3,
      "fires": "freeSpins",
      "params": { "count": 8 }
    }
  ]
}
```

---

## 7. Risks to Watch For

**Hidden coupling between math and display**
The most common failure mode. If the spin animator starts reaching into the result object to decide what to show, or the evaluator starts checking DOM state, you lose the ability to test math independently. The fix is to enforce a clean result contract: the math layer returns data, the display layer reads data.

**Hardcoded reel assumptions**
Classic examples: `if (reelIndex === 2)` logic in the evaluator, or CSS that assumes exactly 5 reels. These break silently when config changes. Every grid assumption should be derived from config at runtime.

**Confusing configuration**
A config schema that requires deep knowledge of the engine to use correctly is not a prototype tool — it is a burden. The schema should be reviewable by a designer without explanation. Field names like `reelStrips` and `paylines` are clear; field names like `stripSampleMode_v2_final` are not.

**Poor debug visibility**
If forcing a specific result requires editing source code, non-engineers cannot test edge cases. A forced-result input in the debug panel pays for itself immediately.

**Overbuilding too early**
The strongest pull in this kind of exercise is toward completeness. A credit balance, a bet selector, sound effects, and a lobby screen are all reasonable slot game features — and all of them are out of scope for a prototype engine. Every hour spent on features that belong in a shipping game is an hour not spent on the flexibility that makes this tool worth using.

**Results that are difficult to reproduce**
If the math uses `Math.random()` directly with no seed option, testing a specific outcome requires luck. A seeded RNG mode — even just `forcedResult` override in debug — makes QA and stakeholder demos reliable.

**Mismatch between internal data layout and human mental model**
The engine stores the grid as `grid[reel][row]` — column-first, because that maps cleanly to reel strips. But humans read a slot grid row-first, left to right. This gap caused the debug force-result input to accept input in the wrong orientation, so a forced result that looked correct visually produced the wrong grid internally and never evaluated as a win. Any tool that exposes grid data to non-engineers — debug panels, config editors, logging — needs to translate to row-first display, or it will cause confusion every time.

**Evaluation logic that doesn't scale to new game types**
A payline evaluator written specifically for 5-line classic slots will break when someone wants to prototype a Megaways-style game. The evaluator strategy pattern described in Section 3 exists precisely to prevent this.

---

## 8. Output Quality Check

**What is strong:**
The layer separation is the right call and the design decision I am most confident in. Keeping math, config, display, and feature hooks as distinct concerns means any one of them can change without destabilizing the others. The config schema is also intentionally narrow — it covers what varies between prototypes without exposing implementation details.

The phased plan reflects real prioritization: Phase 1 and 2 produce something visible and testable; Phase 3 makes it feel like a game; Phase 4 makes it a tool. Each phase has a concrete proof of usefulness rather than a vague milestone. I started with the math core because it defines the game, everything else is built around it. Phase 2 moves it from something only visible in a console to something anyone in the room can react to, and it forces you to verify that the visual output actually matches the math. Phase 3 is the moment it becomes a game rather than a spinning grid. Phase 4 came last because it adds depth and gives non-engineers a way to test edge cases themselves, without needing to touch code.

**Caught during testing:**
During prototype review, the win evaluator had two bugs that only surfaced through active testing. First, the paytable was misconfigured so that 2-of-a-kind paid out on a 3-reel game, producing an unrealistic win rate above 90%. Zeroing out sub-threshold paytable entries looked like a fix, but the real lesson was that the evaluator itself needed to enforce full-line matching in logic — not just rely on config values being correct. Second, an off-by-one error in the paytable index lookup meant that even valid 3-of-a-kind wins returned zero payout — the kind of bug that's invisible until you specifically test a winning outcome and see nothing fire.

Both bugs are a direct argument for a simulation mode that runs hundreds of spins automatically and reports actual win rates. Manual testing over a handful of spins doesn't surface either of these reliably.

After adding the simulation mode and running 10,000 spins, the 5×3 config settled at approximately 10% hit frequency. This is intentionally on the looser end of a realistic range (typical base games target 5–8%) — for a prototype where the primary goal is showing wins clearly to stakeholders, a slightly higher hit rate makes the demo more readable in a room. Tightening it toward production math would be a config-only change to the reel strips.

**What is weak:**
The feature hook system in Phase 4 is underspecified. In practice, free spins alone surface a lot of complexity — accumulating spins, re-triggering, return-to-base behavior. A stub is fine for a prototype engine, but the interface between core and feature code needs more thought before someone builds a second feature on top of it.

The config schema also doesn't address reel-level symbol weight overrides yet. Global weights are enough for most prototypes, but per-reel weighting is how most real games are tuned, and adding it later will require a schema change.

**What would improve with more time:**
A simulation mode was added to the debug panel in the final pass — running 100, 1,000, or 10,000 spins instantly and reporting win rate, feature trigger rate, wins by symbol, and hits by payline. This is the kind of tool that pays dividends when leadership asks "how often does the bonus fire?" during a review, and it is the clearest demonstration that the math layer is genuinely decoupled from display — the simulation calls the math functions directly with no rendering involved.

I would also want to explore adding a credit system and a minigame-within-a-game. Credits matter because they tie player decisions to outcomes — without them, every spin feels the same weight. A minigame triggered by the bonus gives leadership something concrete to react to, which is ultimately what a prototype is for.

---

*Prototype demo: see `index.html`*
