# machin-game-2048

The **2048** sliding-puzzle as a real **native desktop app** — written in **[machin](https://github.com/javimosch/machin)** (MFL) and drawn with [raylib](https://www.raylib.com/) through machin's C FFI. A real OpenGL window, not a terminal UI: arrow keys (or WASD) slide the tiles, equal tiles merge, a new `2` or `4` spawns each move, the score climbs, **R** restarts, **Esc** quits.

Part of [**awesome-machin**](https://github.com/javimosch/awesome-machin) — the machin ecosystem.

> **Agents:** [`SKILL.md`](SKILL.md) covers the build (incl. no-root raylib), the FFI mapping, and the board mechanics.

```
+-----------------------------+
|  2048           score 24    |
|  +-----+-----+-----+-----+  |
|  |     |  2  |     |     |  |
|  +-----+-----+-----+-----+  |
|  |  4  |  4  |     |     |  |
|  +-----+-----+-----+-----+  |
|  |  8  |     |  2  |     |  |
|  +-----+-----+-----+-----+  |
|  |     |     |     |  2  |  |
|  +-----+-----+-----+-----+  |
+-----------------------------+
```

## Why it exists

The machin north star is "build real things." Two of the ecosystem's games stress different ends of the language: [machin-game-snake](https://github.com/javimosch/machin-game-snake) drove the terminal-input builtins; this one goes the *other* way — a **GUI desktop app** — to exercise machin's **C FFI** on a second real graphics program. The whole game logic is pure MFL over a flat `[]int` board; only rendering and input cross into raylib (scalars + one by-value `Color` struct — the same FFI surface as the [examples/gui menu](https://github.com/javimosch/machin/tree/main/examples/gui)). It composes what's there rather than adding a builtin — the FFI was already enough to drive a real graphics library.

## Build

Needs the `machin` compiler, a C compiler, **raylib**, and a display (X11/desktop). A GUI binary links the system graphics stack (`libGL`/`libX11`), so — unlike machin's headless tools — it is **not** a no-dependency binary.

```bash
./build.sh            # → ./machin-game-2048
./machin-game-2048
```

`build.sh` uses a **system raylib** if one is installed (`sudo apt-get install libraylib-dev`, `brew install raylib`, …). If not, it **vendors raylib's prebuilt static release** into `vendor/` automatically — no root required — and links that. Override the compiler with `MACHIN=/path/to/machin ./build.sh`.

## Play

| key | action |
|-----|--------|
| `←` `↑` `→` `↓` or `A` `W` `D` `S` | slide all tiles |
| `R` | restart |
| `Esc` | quit |

Slide to push every tile as far as it goes; two tiles with the same number merge into their sum and add it to your score. Each move spawns a new `2` (90%) or `4` (10%). When the board fills with no merges left, it's game over — press `R`.

## How it works

- **Board.** A flat `[]int` of 16 (row-major); `0` is empty.
- **One slide for four directions.** `lane_index(dir, lane, pos)` maps a direction + lane + travel-position to a board index, so a single `move_lane` (compact non-zeros → merge each equal pair once → pad) handles left/right/up/down. It mutates the shared board slice in place and reports score gained + whether anything moved.
- **Spawn / game-over.** `rand_bytes` picks a random empty cell; `has_moves` checks for any empty cell or mergeable neighbor.
- **Render.** raylib immediate mode: `ClearBackground`, `DrawRectangle` per tile (classic 2048 palette), `MeasureText` + `DrawText` to center each number (font shrinks as the number widens), and a translucent overlay on game over.

See [`game2048.src`](game2048.src). `build.sh` runs `machin encode` to produce the canonical `game2048.mfl`, then `machin build`.

## License

MIT
