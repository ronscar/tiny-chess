# tiny-chess-01 entry

A compact alpha-beta chess engine in a single `main.py` (Python 3.13,
standard library only), written from scratch for the make.fun
"strongest chess engine in 4 KB of Python" bounty.

## Design

- **Board:** 0x88 (128 squares), pieces 1-6 white / -1--6 black.
- **Search:** iterative deepening to a fixed node budget (60,000
  nodes/move), alpha-beta with a transposition table (mate-score
  ply adjustment), two killer moves per ply, MVV-LVA + promotion
  move ordering, and a transposition-table move first.
- **Pruning:** null-move pruning (R=2, depth >= 3, only with
  non-pawn material and not in check), late-move reductions
  (quiet moves after the first 3, depth >= 3, re-searched at full
  depth if they beat alpha), and delta pruning in quiescence.
- **Quiescence:** captures and promotions, all replies when in
  check; returns mate score when no legal reply exists.
- **Evaluation:** material (100/320/330/500/900) + six
  hand-written piece-square tables (mirrored to 32 values each)
  + bishop-pair bonus. Zobrist hashing; repetition via the
  game history replayed from the start position each turn.
- **Determinism:** fixed node budget, no clock, no unseeded
  randomness (Zobrist keys from a fixed seed). Replays play the
  same moves. No network, no model calls, stdlib only.

## Size

`main.py` is an lzma + base85 packed form of the readable golfed
source (packing is explicitly allowed by the rules; the house
baseline is packed the same way). Packed size is 4,045 bytes,
under the 4,096-byte limit (lzma FORMAT_ALONE + base85). The readable source, the pack script,
and the full test harness live outside this repo in the author's
working directory.

## Run

The program talks to the env at `MAKEFUN_ENV_URL`:

    MAKEFUN_ENV_URL=http://127.0.0.1:8100 python3 main.py

It asks for the game in play (`POST /game`), sends its moves in
UCI (`POST /move`), and stops when the env reports `done`.

## Test

See the kit: run `program/env/server.mjs` for a practice trial,
then score with `program/scorer/score.mjs`.

## Results (practice trials, local, real kit env + scorer)

- Kit starter (1-ply greedy), trial 0: 0W 0D 8L, Elo 0.
- This engine, 40,000 nodes, trial 0: 2W 1D 5L, Elo 1,173.
- This engine, 40,000 nodes + LMR, trials 0/1/2:
  1,173 / 1,087 / 989, mean 1,083.
- This engine, 60,000 nodes + LMR, trials 0/1/2:
  1,253 / 1,087 / 1,087, mean 1,142 (drew vs the 100k-node
  house engine on trial 0; beat it on trial 2).
- v11 (this entry): same 60k budget, improved move ordering
  (stronger MVV-LVA, promotion bonus on captures, killers on
  captures) + merged attack check. Trials 0/1/2: 1,326 / 1,253 /
  1,253, mean 1,277. Max move 1.99s; max wall 508s locally.
- House baseline (sunfish 4K, 50,000 nodes/move): mean Elo 1,325
  across the three practice trials.

The v11 mean (1,277) clears the 1,154 practice floor by 123 Elo,
48 below sunfish. LMP (trial-0 Elo 871) and PVS were tried and
dropped; they hurt.

## License

MIT. See LICENSE.
