---
name: machin-game-2048
description: Build, run, and modify machin-game-2048 — the 2048 puzzle as a native raylib desktop app written in machin (MFL). Use when working on this repo, or as a worked example of driving a GUI / graphics library from machin through the C FFI (window, rectangles, text, keyboard).
---

# machin-game-2048

The 2048 sliding-puzzle as a real native desktop window, written in [machin](https://github.com/javimosch/machin) (MFL) and rendered with [raylib](https://www.raylib.com/) over machin's C FFI. It is the reference example for **GUI / graphics** programs in machin (the terminal counterpart is [machin-game-snake](https://github.com/javimosch/machin-game-snake)).

## Build & run

```bash
./build.sh                 # machin encode game2048.src -> game2048.mfl, then machin build -> ./machin-game-2048
./machin-game-2048
```

Needs `machin`, a C compiler, **raylib**, and a display.

- **System raylib** (`apt-get install libraylib-dev`, `brew install raylib`, …): `build.sh` detects it via `pkg-config`/headers and builds the committed system-style source directly.
- **No root:** `build.sh` downloads raylib's prebuilt **static** release into `vendor/` and injects `cflags "-I… -L…"` + `link ":libraylib.a"` into a throwaway copy of the encoded `.mfl` — the committed `game2048.src` stays system-style.
- `MACHIN=/path/to/machin ./build.sh` to use a specific compiler.

Controls: `←↑→↓` or `WASD` to slide, `R` restart, `Esc` quit.

## How it maps to the FFI

The whole `extern "raylib" { … }` block is the only thing crossing into C; everything else is pure MFL. It uses just FFI Phases 1–2 — scalars and one by-value struct — exactly like `machin/examples/gui`:

| MFL | raylib C | FFI feature |
|-----|----------|-------------|
| `cstruct Color { r u8 g u8 b u8 a u8 }` | `Color{ unsigned char r,g,b,a; }` | by-value struct |
| `fn DrawText(string,i32,i32,i32,Color)` | `void DrawText(const char*,int,int,int,Color)` | scalars + struct arg |
| `fn MeasureText(string,i32) i32` | `int MeasureText(const char*,int)` | scalar return (text centering) |
| `fn IsKeyPressed(i32) bool` | `bool IsKeyPressed(int)` | scalar return (input) |
| `link "raylib" link "GL" …` | `-lraylib -lGL …` (in order) | multi-lib linking |

raylib is immediate-mode and polls input via functions, so no opaque handles or callbacks are needed. raylib key codes used: `263/262/265/264` = Left/Right/Up/Down, `65/68/87/83` = A/D/W/S, `82` = R. (Esc is raylib's default window-close key, caught by `WindowShouldClose()`.)

## Patterns worth copying

- **One slide routine, four directions.** `lane_index(dir, lane, pos)` maps direction + lane + travel-position → flat board index; `move_lane` then compacts non-zeros, merges each equal pair once, and pads. No four-way copy-paste.
- **Mutate a slice in place across calls.** `move_lane`/`spawn`/`reset` take the `board` slice and write `board[i] = …`; the writes hit the shared backing array, so the caller sees them. They return only *summaries* (gained, moved, ok).
- **No `str(bool)`.** machin's `str` is numeric only — stringifying a bool is a type error. Keep bools in control flow (`if moved { … }`), never `str(moved)`.
- **Random without a PRNG builtin.** `rand_bytes(2)` then `% len(empties)` picks a cell; a second byte `< 26` gives the ~10% chance of a 4.
- **Center text.** `MeasureText(s, fs)` then `x + (CELL - tw)/2`; shrink the font as the number widens so 4-digit tiles still fit.

## Modifying

- **Board size** is wired to 4×4 (the `% 4` / `* 4` indexing and the 16-cell loops); changing it touches `lane_index`, `has_moves`, and the render loop.
- **Window / tile geometry:** the `WIN_W/WIN_H/BX/BY/GAP/CELL` helper funcs.
- **Colors:** `tile_color(v)` (per-value palette) and `text_color(v)`.
- **Win detection** (reaching 2048) isn't enforced — play continues; add a check in the move handler if you want a win banner.
- After any edit to `game2048.src`, re-run `./build.sh` (never hand-edit `game2048.mfl` — it is generated).
