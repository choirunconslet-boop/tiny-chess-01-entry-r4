# tiny-chess-01 — round 4

An original 4 KB Python chess engine for the make.fun `tiny-chess-01`
competition ("Build the strongest chess engine in 4 KB of Python").

## Engine

`main.py` (4,087 bytes) is a self-contained, pure-stdlib engine:

- **Board:** 0x88, incremental material + piece-square evaluation
  (classic-style PSTs; no bishop-pair term — measured ~0 Elo).
- **Search:** alpha-beta with transposition table, killer moves,
  null-move pruning, late-move reductions, check extension,
  depth-1 futility pruning, and quiescence (captures, promotions,
  all evasions when in check). Iterative deepening with aspiration
  windows (±25 cp) under a fixed 100,000-node cap per move.
- **Root ordering:** root moves are pre-ordered by static evaluation,
  so developing moves (Nf3/Nc3/d4/e4) are tried before quiet
  waiting moves — this matters because a PST-only eval ties most
  opening moves at shallow depth.
- **Speed:** ~20,000–32,000 nodes/sec locally (CPython 3.12);
  a 100k-node move takes ~3–5 s, inside the 10 s/move limit.
- **Correctness:** perft-verified (startpos d4 197281, Kiwipete d3
  97862, EP + promotion positions) and eval/unmake-consistent over
  random games. Full legal move generation: castling, en passant,
  promotion. The driver replays the game from the start position,
  validates its move against the environment's legal list, and falls
  back to the best capture (then the first legal move) if the search
  ever disagrees with the environment.

## Why round 4

Round 3 (a sunfish MTD(f) fork, 60k node cap, no quiescence) scored
708 Elo live (0W/1D/5L) — below the 1,154 gate. Measured locally it
ran only ~7,100–9,300 nps, so a 60k-node move took 6.5–8.5 s, within
~2–3 s of the 10 s/move limit; on a slower grading CPU moves time
out. It also had an incremental-eval sign bug for black movers
(the score drifted after every black move) that its test never
caught, because the test unmade every move instead of playing a
real game. Round 4 is a from-scratch rewrite that fixes both:
~3× faster, correct incremental eval, and quiescence search.

Local practice trials (kit harness, trial 0, 6 games vs the house
gauntlet depth-2 … nodes-500k), run 18 Nov 2026:

- Round 3: **1,364 Elo** (3W/1D/2L, spare_plies 241) — above the
  1,154 gate locally; its live 708 was a speed/timeout artifact.
- Round 4, 100k-node cap: **1,099 Elo** (1W/2D/3L,
  spare_plies 58), max 6.2 s/move, avg 3.5 s/move.
- Round 4, 125k-node cap: **997 Elo** (1W/1D/4L) — the bigger cap
  did not help (6-game trials are noisy; ±100 Elo or more).

So round 4 is faster and more robust against move timeouts than
round 3, but weaker on the board and below the 1,154 gate in local
testing. It is a correct, original engine; whether it is worth
submitting is a judgement call for the entrant, not a claim made
here.

## Files

- `main.py` — the packed engine (the submission).
- `LICENSE` — MIT.

Build notes (dev source, tests, packer) are kept out of this repo;
the submission is just `main.py` + `LICENSE`.
