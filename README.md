# Sim Cryptid Emergence

![Language](https://img.shields.io/badge/language-C%2B%2B17-blue)
![Platform](https://img.shields.io/badge/platform-Windows%20x64-lightgrey)
![GUI](https://img.shields.io/badge/GUI-wxWidgets%203.2-green)
![Build](https://img.shields.io/badge/build-Visual%20Studio%202022-purple)

A fully-featured **Conway's Game of Life** simulation built with **C++17 and wxWidgets**. Configurable grid sizes up to 1000 cells, two universe boundary modes, full file persistence, live HUD overlay, customizable colors, and a toolbar-driven simulation loop — all rendered through a double-buffered wxWidgets drawing panel.

> Part of the Squatchworks portfolio. Developed by [Jacob Blackburn](https://github.com/Squatchworks).

---

## Features

**Simulation**
- Full Conway's Game of Life rule implementation — underpopulation, overpopulation, survival, and reproduction
- Sandbox/scratchpad double-buffer pattern — next generation calculated in a separate array then swapped in
- Two boundary modes switchable at runtime:
  - **Finite** — cells at the universe edge treat out-of-bounds neighbors as dead
  - **Toroidal** — edges wrap, connecting left↔right and top↔bottom into a seamless loop
- Configurable simulation interval (milliseconds per generation via Settings dialog)

**Controls**
- Toolbar: Play, Pause, Next (step one generation), Clear
- Left-click any cell to toggle it alive/dead
- Randomize with a time-based seed or a user-entered seed (0–1000)
- New, Open, Save, Save As, Import — full `.cells` file format support
- Reset Settings to defaults

**View**
- Toggleable cell grid lines
- Toggleable 10×10 bold grid overlay for navigation
- Toggleable neighbor count display — live neighbor tally drawn inside each cell
- Toggleable HUD overlay — generation count, living cell count, boundary mode, and universe size
- Fully configurable living cell color, dead cell color, and grid line color via color picker

**Persistence**
- Settings serialized to `settings.bin` using raw binary `fwrite` — loaded automatically on startup
- Game states saved and loaded as `.cells` files (`*` = alive, `.` = dead, one row per line)
- Import merges a `.cells` file into the current universe without resizing

---

## Architecture
Got everything. This is a much more substantial project than the brick breaker — wxWidgets GUI, settings persistence, toroidal/finite universe modes, file I/O, HUD overlay, the works. Writing the README now.

Paste this directly into the GitHub README editor, replacing everything:

text
# Sim Cryptid Emergence

![Language](https://img.shields.io/badge/language-C%2B%2B17-blue)
![Platform](https://img.shields.io/badge/platform-Windows%20x64-lightgrey)
![GUI](https://img.shields.io/badge/GUI-wxWidgets%203.2-green)
![Build](https://img.shields.io/badge/build-Visual%20Studio%202022-purple)

A fully-featured **Conway's Game of Life** simulation built with **C++17 and wxWidgets**. Configurable grid sizes up to 1000 cells, two universe boundary modes, full file persistence, live HUD overlay, customizable colors, and a toolbar-driven simulation loop — all rendered through a double-buffered wxWidgets drawing panel.

> Part of the Squatchworks portfolio. Developed by [Jacob Blackburn](https://github.com/Squatchworks).

---

## Features

**Simulation**
- Full Conway's Game of Life rule implementation — underpopulation, overpopulation, survival, and reproduction
- Sandbox/scratchpad double-buffer pattern — next generation calculated in a separate array then swapped in
- Two boundary modes switchable at runtime:
  - **Finite** — cells at the universe edge treat out-of-bounds neighbors as dead
  - **Toroidal** — edges wrap, connecting left↔right and top↔bottom into a seamless loop
- Configurable simulation interval (milliseconds per generation via Settings dialog)

**Controls**
- Toolbar: Play, Pause, Next (step one generation), Clear
- Left-click any cell to toggle it alive/dead
- Randomize with a time-based seed or a user-entered seed (0–1000)
- New, Open, Save, Save As, Import — full `.cells` file format support
- Reset Settings to defaults

**View**
- Toggleable cell grid lines
- Toggleable 10×10 bold grid overlay for navigation
- Toggleable neighbor count display — live neighbor tally drawn inside each cell
- Toggleable HUD overlay — generation count, living cell count, boundary mode, and universe size
- Fully configurable living cell color, dead cell color, and grid line color via color picker

**Persistence**
- Settings serialized to `settings.bin` using raw binary `fwrite` — loaded automatically on startup
- Game states saved and loaded as `.cells` files (`*` = alive, `.` = dead, one row per line)
- Import merges a `.cells` file into the current universe without resizing

---

## Architecture
GameOfLife/
├── Main entry
│ └── App.cpp / App.h # wxApp entry point — bootstraps MainWindow
│
├── MainWindow.cpp / MainWindow.h # Core controller — game loop, neighbor counting,
│ # generation stepping, file I/O, all menu/toolbar events
│
├── DrawingPanel.cpp / DrawingPanel.h # wxPanel subclass — double-buffered paint,
│ # cell toggle on click, HUD, neighbor count overlay
│
├── Settings.h # POD struct — all runtime settings with Save/Load
│ # via binary file I/O and RefreshSettings() reset
│
├── SettingsDialog.cpp / .h # wxDialog — grid size, interval, and color pickers
│ # writes directly back to MainWindow::mSettings on OK
│
└── Header.h # Reserved

`MainWindow` owns the `mGameBoard` (`vector<vector<bool>>`) and `mNeighborCounts` (`vector<vector<int>>`), both passed to `DrawingPanel` by reference. `MainWindow::IncrementGameBoardOneGeneration()` runs the full simulation step: allocate sandbox, apply rules via `CountLiveNeighbors()`, swap boards, recount neighbors, update status bar. `DrawingPanel::OnPaint()` uses `wxAutoBufferedPaintDC` and `wxGraphicsContext` to eliminate flicker during grid redraws.

---

## Neighbor Counting

Two implementations selectable at runtime:

**Finite (`FiniteCellCountMethod`)** — 3×3 loop centered on the target cell, skips any index below 0 or above grid size. Edge cells have fewer than 8 neighbors; out-of-bounds is treated as dead.

**Toroidal (`TorodialCellCountMethod`)** — same 3×3 loop but passes each index through `CheckIndice()`, which maps `-1` to `gridSize - 1` and `gridSize` to `0`, wrapping the universe on both axes.

---

## How to Build

### Prerequisites

- Visual Studio 2022 with the **Desktop development with C++** workload
- [vcpkg](https://github.com/microsoft/vcpkg) installed and integrated (`vcpkg integrate install`)
- wxWidgets installed via vcpkg: `vcpkg install wxwidgets:x64-windows`

### Build steps

1. Clone the repo: git clone https://github.com/Squatchworks/Sim_Cryptid_Emergence.git
2. Open `Game Of Life.sln` in Visual Studio 2022.
3. Set configuration to **Release | x64**.
4. **Build → Build Solution** (`Ctrl+Shift+B`).

If wxWidgets is not found, verify your vcpkg integration or set the include/lib paths manually in the project properties.

---

## Lessons Learned

- **`wxAutoBufferedPaintDC` eliminates flicker** — using a plain `wxPaintDC` during full-grid redraws produces visible tearing; the buffered variant composites the frame off-screen before blitting.
- **Binary settings serialization is fragile across struct changes** — writing the raw `Settings` struct with `fwrite` is fast but will silently corrupt on any field reorder or size change. A future version should use a text or JSON format.
- **Passing board state by reference** — `DrawingPanel` holds `rGameBoard` and `rNeighborCounts` as references to `MainWindow`'s vectors rather than copies. This keeps rendering and simulation in sync without data duplication, but couples the two classes tightly.
- **Toroidal wrapping via index clamping** — the `CheckIndice()` helper cleanly separates the wrap logic from the neighbor counting loop, making it easy to switch boundary modes without duplicating the iteration code.

---

## Roadmap

### Current build
- Full Conway's rules with finite and toroidal boundary modes
- wxWidgets GUI with toolbar, menu bar, status bar, and settings dialog
- `.cells` file save/load/import
- Binary settings persistence
- Live HUD, neighbor count overlay, configurable colors

### Planned — "Emergence: Cryptid Edition"
A visual and thematic reskin targeting the Cryptid Forge Studios aesthetic:

- Paranormal-themed cell states — spores, organisms, cryptid entities
- Atmospheric color palette and animated cell transitions
- Procedural pattern library with named cryptid-themed starting configurations
- Sound design — ambient tone that shifts with living cell density
- Export simulation frames as image sequences
- Settings persistence migrated from binary to a readable format

Currently in the branding and design backlog.

---

## Author

**Jacob Blackburn** — Gameplay programmer and C++ developer based in Russells Point, Ohio.
- GitHub: [@Squatchworks](https://github.com/Squatchworks)
- LinkedIn: [linkedin.com/in/squatchworks](https://linkedin.com/in/squatchworks)
