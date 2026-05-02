# Cognitive Assembly Loop — Architecture Analysis

## Origin: PentestGPT

PentestGPT (USENIX 2024, GreyDGL) achieved 86.5% benchmark success by separating reasoning (GPT-4) from tool execution. Its core insight: LLMs lose context across a long pentest unless a structured session object accumulates knowledge between calls.

penligent extends this into a **multi-agent loop** where six agents with distinct epistemological roles each see a shared `CognitiveState` object and contribute their specialist perspective before passing it on.

---

## The Six Agents and Their Epistemic Roles

| Agent | Reasoning Style | Model | Role in the Loop |
|---|---|---|---|
| Recon | Inductive | claude-sonnet-4-6 | Generalise from port/banner observations → surface topology |
| Hypothesis | Probabilistic | claude-opus-4-7 | Bayesian hypothesis tree, novelty flagging |
| Execution | Procedural | claude-sonnet-4-6 | Select best hypothesis, simulate tool call, capture flags |
| Analyst | Abductive | claude-opus-4-7 | Find best explanation for surprising findings, detect loops |
| Adversarial | Adversarial | claude-sonnet-4-6 | Devil's Advocate gate, evasion constraints, blue team model |
| Memory | Analogical | claude-sonnet-4-6 | Retrieve proven chains from episodic memory, warn on patterns |

---

## Three Loop Configurations

The same six agents run in different orders. The order is the hypothesis — different epistemologies create different emergent behaviours.

```
ALPHA  (Rationalist / Intelligence-First)
  MEMORY → RECON → HYPOTHESIS → EXECUTION → ANALYST → ADVERSARIAL
  Rationale: ground the run in analogical memory before gathering new data

BETA   (Empiricist / Strike-First)
  EXECUTION → ANALYST → ADVERSARIAL → RECON → MEMORY → HYPOTHESIS
  Rationale: act immediately on prior knowledge, analyse what surfaces

GAMMA  (Pragmatist / Stealth-First)
  ADVERSARIAL → MEMORY → HYPOTHESIS → RECON → EXECUTION → ANALYST
  Rationale: set evasion rules before any action is taken
```

---

## Anti-Loop Mechanisms

All of these are **architectural**, not prompt engineering:

### 1. Semantic Brake
Word-overlap similarity between consecutive state summaries. If > 0.92 → stagnation detected → anomaly injection triggered.

### 2. Anomaly Engine (Stochastic Resonance)
Injects controlled noise into the evidence stream. Seven anomaly types:
- COUNTERFACTUAL — "what if this port were not open?"
- PERSPECTIVE_INVERSION — see through the defender's eyes
- MAGNITUDE_SHIFT — what if the scope were 100× larger?
- DOMAIN_CROSSING — what attack techniques from adjacent domains apply?
- FALSE_EVIDENCE — fabricated finding (haiku-generated) to surface weak signals
- TEMPORAL_DISPLACEMENT — what would this attack look like in 6 months?
- CONSTRAINT_REMOVAL — remove one evasion constraint and reason freely

Anomalies have learned effectiveness weights updated each iteration.

### 3. Devil's Advocate Gate
The Adversarial agent raises ≥3 objections per turn. If they cannot be rebutted, `allowed_to_proceed=false` blocks the next Execution step entirely. This is structurally equivalent to Prolog's `\+` (negation-as-failure).

### 4. Cognitive Temperature Scheduling
Simulated annealing: temperature = 1.0 → 0.5 over iterations. Stagnation spikes it by +0.3. Explore early, converge late, restart on stuck.

### 5. Dead-End Memory
The Analyst agent identifies conclusively failed paths. The Runner never revisits them. The Memory agent warns on pattern repetitions.

---

## The Game Layer: Snakes and Ladders

Game mechanics are the cognitive architecture, not decoration. The board creates the incentive gradient that makes original thinking the fastest path to winning.

### Why This Works: Variable Ratio Reinforcement
Slot machine psychology. Unpredictable reward timing creates maximum exploration pressure. The agent cannot settle into a routine because routines don't earn ladders.

### Board Structure
```
100 squares, 5 zones:
  Zone 1 (1-20):   Discovery — ladders for novel surface findings
  Zone 2 (21-40):  Enumeration — ladders for service fingerprinting
  Zone 3 (41-60):  Exploitation — ladders for confirmed vulns, snakes for noise
  Zone 4 (61-80):  Lateral movement — ladders for pivot chains
  Zone 5 (81-100): Exfiltration — ladders for flags, snakes for detection
```

### Ladder Conditions (selected)
| Condition | Move | Why |
|---|---|---|
| novel_hypothesis | +12 | Probabilistic agent found something not in memory |
| divergence_finding | +20 | Analyst flagged high divergence_score |
| flag_captured | +30 | Execution captured a flag |
| memory_shortcut | +8 | Memory agent found a proven chain |

### Snake Conditions (selected)
| Condition | Move | Why |
|---|---|---|
| semantic_brake | -10 | Stagnation detected |
| detection_event | -12 | Blue team triggered |
| pattern_gravity | -15 | Repeated same attack vector |

### Luck System: More Carrot Than Stick
Luck rolls happen at the start of every turn (d20):
- 1-5: Fortune Event (+15–30, real vulnerability found by luck)
- 6-10: Free CHANCE card draw
- 11-15: Small Gift (+3–8)
- 16-19: Jackpot (+20)
- 20: LEGENDARY (2× multiplier, named in hall of fame)

**Pity Timer**: After 3 dry turns → +5. After 5 → free card. After 8 → snake immunity for 2 turns. The pity timer prevents demoralisation.

### Game Outcomes
```
WIN_PERFECT   — board ≥95, flags, clean (×2.5 score)
WIN_CLEAN     — board ≥80, low noise (×1.8)
WIN_LUCKY     — board ≥70, luck_score > 40% of total (×1.5)
WIN_STANDARD  — board ≥60 (×1.0)

LOSE_BURNED       — detection_events > 5
LOSE_EXHAUSTED    — iterations maxed, board < 40
LOSE_TIMED_OUT    — wall clock exceeded
LOSE_BANKRUPT     — score went negative
LOSE_NOISE_LIMIT  — noise_budget exhausted

TRY_PIVOT   — board 40-60, retry keeping dead_ends + nodes
TRY_COOL    — 2+ detection events, retry with tighter constraints
TRY_FRESH   — score 20-50, retry with cleared hypothesis tree
TRY_ANGLE   — 3+ failed hypotheses, retry with new attack vectors
TRY_LUCKY   — luck_score > 30, retry retaining all luck bonuses
```

**TRY AGAIN is a checkpoint, not a reset.** Acquired knowledge (dead ends, discovered nodes, captured evidence) is preserved. Only execution state resets.

---

## Prolog Game Rules Engine

`penligent/loop/game_rules.pl` declares all game logic as pure logical facts. This is not incidental — it's structurally correct:

- Ladder/snake conditions ARE logical facts → `ladder_condition/3`, `snake_condition/3`
- Devil's Advocate IS `\+` (negation-as-failure)
- Hypothesis exploration IS Prolog backtracking
- Win/lose evaluation IS rule unification

Python queries the Prolog engine via `pyswip`. The separation keeps game logic auditable, modifiable without Python knowledge, and formally verifiable.

---

## Results Analysis

`ResultsAnalyzer` compares ALPHA/BETA/GAMMA across six metrics:
- `final_score` — overall effectiveness
- `detection_events` — stealth (lower is better)
- `total_noise` — execution discipline
- `anomaly_injections` — loop stagnation tendency
- `ladder_moves` — original thinking rate
- `luck_score` — pity timer distribution

Divergence > 20% on any metric is reported. Flags captured by only one configuration are flagged as **order-dependent** — they require that specific epistemological sequencing and cannot be found by the others.

---

## Key Insight

The ordering of agents is not administrative. It determines:
- Which knowledge is present when hypotheses are formed
- Whether evasion constraints precede or follow execution
- Whether memory primes or post-processes the run
- Which findings emerge vs. which are structurally invisible

Running all three in parallel on the same target is not redundancy. It is triangulation.
