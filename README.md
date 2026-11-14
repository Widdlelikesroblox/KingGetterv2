# Kingmaker

A checkers coach for American checkers (8x8, captures are mandatory, kings move both ways).

Pick Black or Red, set the pieces up the way they sit on your real board, and record every move
that is played, yours and your opponent's. Kingmaker searches the position and shows the strongest
move for whoever is to play, with an arrow on the board, a win/loss bar and the likely line.

## Quick install

Linux or macOS: `./install.sh`, then `./kingmaker-run`.
Windows: double-click `install.bat`, then `Kingmaker.bat`.

The installer checks Python and Tkinter, builds the C engine and creates the launcher.

## Run it by hand

Needs Python 3.10 or newer with Tkinter (included with the python.org installers; on Debian or
Ubuntu run `sudo apt install python3-tk`). There are no Python packages to install.

```bash
python build_native.py   # compiles the fast C engine (needs gcc or clang)
python run.py            # or: python -m kingmaker
```

The window shows which engine is running. Without the compiled library the app falls back to the
pure Python engine, which works but searches about 12 times shallower.

## Using it

- **Play tab:** click a piece, then its destination, to record a move. Multi-jumps are clicked one landing square at a time.
- **Play this move** applies the suggested move for you. **Undo** steps back.
- **Set up tab:** place or erase men and kings for both sides, choose who moves next, then press *Analyze this position*. Use this to match a game that is already in progress.
- **Think time:** longer means stronger. Deep (12 s) is the default.

### Styles and settings (change them at any time, even mid-game)

The **Style** buttons on the move card, and the **Settings** button at the top, control how your moves are chosen.

| Style | What it does |
| --- | --- |
| Balanced | Plays the strongest move it can find. |
| Safe | Takes no chances. It values a draw as a good result, keeps a sturdier back rank, and among near-equal moves picks the one that gives your opponent the fewest forcing replies. |
| Aggressive | Avoids draws, pushes pieces forward, and among near-equal moves picks the one with the best position. Use it when you are ahead or must win. |

Other settings: switch to Safe **automatically when you are behind** (you choose how far behind), show the **other options** with quick scores, and choose how many **CPU threads** to use. Settings are saved in `.kingmaker.json` in your home folder.

Your opponent is always assumed to play its best, whatever style you choose. Safe, Aggressive and extra threads need the C engine.

### Draw rules

The engine knows that a position that repeats, or 80 plies (40 moves each) without a capture or man move, is a draw. It uses this to avoid shuffling when it is winning and to head for a draw when it is losing. The app tracks your game so these rules also apply to moves already played.

## Use the engine in your own code

```python
from kingmaker import engine as E

board = E.start_board()                 # 64 ints: 1 black man, 2 black king, -1 red man, -2 red king
result = E.analyze(board, E.BLACK, ms=3000)
print(E.notation(result.best), result.score, result.depth)
board = E.apply_move(board, result.best)
```

## Tests

```bash
python -m unittest discover -s tests -v
KINGMAKER_PURE_PYTHON=1 python -m unittest discover -s tests   # force the Python engine
```

The tests check move generation against the known node counts for the starting position and
cover mandatory captures, multi-jumps, crowning, and basic search behaviour.

## Build a standalone app

The **Build apps** workflow in `.github/workflows/build.yml` builds Windows, macOS and Linux apps
with PyInstaller. It compiles the C engine on each system and packs it inside the app. Run it from the Actions tab, or push a version tag to attach the files to a release:

```bash
git tag v1.0.0 && git push origin v1.0.0
```

## Strength

The engine is an iterative-deepening principal-variation search with alpha-beta pruning, a shared
lock-free transposition table, Lazy SMP multi-threading, late move reductions, killer and history move
ordering, capture extensions, draw rules and a hand-tuned evaluation.

| Engine | Speed (1 thread) | Depth in 3 s from the start |
| --- | --- | --- |
| C (`native/kingmaker.c`) | about 5 million positions per second | 15 plies |
| Pure Python (`engine.py`) | about 60 thousand positions per second | 9 plies |

Measured against the previous version of the C engine, at equal time (100 ms per move, 60 games with
colours swapped, 6 random opening moves), version 2 scored 18 wins, 16 losses and 26 draws. That is not a
measurable difference, so the gains in version 2 are the settings, the draw rules and multi-threading
rather than raw playing strength. An evaluation term for runaway men looked sensible but lost clearly
in testing, so it is switched off. Checkers is a draw with perfect play, so no program can promise a win
against every opponent.

## Ideas to build on

- Tune the weights in `evaluate()` (in `native/kingmaker.c`) by playing versions against each other. This is the biggest remaining gain, and it needs thousands of games to see small improvements.
- Add an endgame database for positions with six or fewer pieces, and an opening book.
- Add a position import and export format.
