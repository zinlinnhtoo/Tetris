# Tetris (Python / Pygame-CE)

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![Game Engine](https://img.shields.io/badge/engine-pygame--ce-informational.svg)](https://pyga.me/)
[![Tests](https://img.shields.io/badge/tests-pytest-green.svg)](https://docs.pytest.org/)
[![License](https://img.shields.io/badge/license-MIT-lightgrey.svg)](LICENSE)

An enhanced, modular desktop implementation of classic Tetris featuring dual gameplay modes, dynamic item mechanics, comprehensive accessibility support (color-blind friendly palettes), persistent leaderboards, and a headless core engine designed for high test coverage.

---

## Table of Contents

- [Features](#features)
- [Game Modes](#game-modes)
  - [Classic Mode](#classic-mode)
  - [Item Mode](#item-mode)
- [Architecture & Design](#architecture--design)
- [Directory Structure](#directory-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Running the Game](#running-the-game)
- [Testing](#testing)
- [Building a Standalone Executable](#building-a-standalone-executable)
- [Configuration & Data Storage](#configuration--data-storage)

---

## Features

- **Decoupled Engine Core**: Game rules, matrix updates, collision detection, and rotation wall-kicks are isolated from presentation logic, allowing 100% headless unit testing.
- **Scene-Based UI**: Clean state machine powering transitions across Main Menu, Game View, Settings, Pause Dialog, Scoreboard, and Game Over screens.
- **Accessibility Modes**: Native color-blind themes for **Protanopia**, **Deuteranopia**, and **Tritanopia** in addition to the standard palette.
- **Configurable Controls**: Dynamic key rebinding support (e.g., WASD vs. Arrow Keys, custom drop keys).
- **Difficulty Presets**: Easy, Normal, and Hard modes adjusting tick rates and gravity drop progression.
- **Persistent Data**: JSON-backed storage for player records, leaderboards, and settings.

---

## Game Modes

### Classic Mode
Adheres strictly to traditional competitive Tetris mechanics:
- Standard $10 \times 20$ grid.
- Standard 7-bag randomizer ($I, J, L, O, S, T, Z$).
- Semi-transparent **Ghost Piece** for landing previews.
- Next-piece queue and standard line-clear scoring rules.

### Item Mode
Introduces tactical arcade blocks that manipulate the board upon landing:

| Item Icon | Item Name | Effect & Behavior |
| :---: | :--- | :--- |
| **B** | **Bomb** | Detonates on impact, clearing a $5 \times 5$ radius around its landing coordinate. |
| **W** | **Weight / Drill** | Ignores obstacles, drilling vertically straight down through blocks to the floor. |
| **L** | **Line Clear** | Instantly clears the entire horizontal row where it locks. |
| **G** | **Golden Piece** | Awards a **$5\times$ score multiplier** for all line clears made using this piece. |

---

## Architecture & Design

The project enforces a strict **Separation of Concerns (SoC)**:

```
+-------------------------------------------------------------------+
|                        src/ui/ (Pygame-CE)                        |
|  - SceneManager (Menu, Game, Settings, Scoreboard, GameOver)      |
|  - Renderer (Grid, Tetrominoes, Ghost Piece, Text & Palettes)     |
+---------------------------------+---------------------------------+
                                  | Reads State / Forwards Input
                                  v
+-------------------------------------------------------------------+
|                        src/core/ (Pure Python)                    |
|  - GameEngine (Ticks, gravity, drop timers, state dispatch)       |
|  - Board (10x20 grid, collision checks, line clears)              |
|  - Tetromino & PieceQueue (7-bag randomizer, SRS wall-kicks)      |
|  - src/items/ (Bomb, Weight, Line Clear, Golden Piece effects)    |
+---------------------------------+---------------------------------+
                                  | Read / Write
                                  v
+-------------------------------------------------------------------+
|                     src/services/ & data/                         |
|  - settings.py <--> data/settings.json (Keybinds, Palettes, Res)  |
|  - scoreboard.py <--> data/scores.json (Top 10 Classic / Item)    |
+-------------------------------------------------------------------+
```

Because `src/core/` does not import `pygame`, unit tests can run instantly in continuous integration (CI) environments without creating a display window.

---

## Directory Structure

```text
tetris_python/
├── assets/                  # Audio, sound effects, TTF fonts, sprites
├── data/
│   ├── scores.json          # Persistent high scores
│   └── settings.json        # Player preferences & keybindings
├── src/
│   ├── core/                # Headless core logic (NO pygame imports)
│   │   ├── board.py         # 10x20 matrix, collision, line clearing
│   │   ├── tetromino.py     # 7 standard shapes + rotations + kicks
│   │   ├── piece_queue.py   # 7-bag randomizer
│   │   └── game_engine.py   # State machine, gravity tick, drop & lock
│   ├── items/               # Item mode logic
│   │   ├── item_types.py    # Bomb, Weight, LineClear, Golden definitions
│   │   └── item_effects.py  # Effect trigger handlers on lock
│   ├── services/
│   │   ├── settings.py      # Color-blind palettes, keymaps, resolution
│   │   └── scoreboard.py    # Score persistence & ranking logic
│   ├── ui/                  # Pygame rendering & Scene management
│   │   ├── scene_manager.py # Screen state switcher
│   │   ├── scenes/          # Scene controllers
│   │   │   ├── menu_scene.py
│   │   │   ├── game_scene.py
│   │   │   ├── settings_scene.py
│   │   │   ├── score_scene.py
│   │   │   └── gameover_scene.py
│   │   └── renderer.py      # Grid, ghost piece, and UI drawing
│   └── main.py              # Application entry point
├── tests/                   # Pytest automated test suites
│   ├── test_board.py
│   └── test_engine.py
├── requirements.txt
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.10 or higher
- `pip` package manager

### Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/your-username/tetris-python.git
   cd tetris-python
   ```

2. **Create and activate a virtual environment** (recommended):
   ```bash
   # macOS/Linux
   python3 -m venv venv
   source venv/bin/activate

   # Windows
   python -m venv venv
   venv\Scripts\activate
   ```

3. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

### Running the Game

Launch the application via the main entry point:
```bash
python src/main.py
```

---

## Testing

Run the test suite using `pytest`. The decoupled architecture allows tests to execute headlessly:

```bash
# Run all tests
pytest tests/

# Run tests with coverage report
pytest tests/ --cov=src
```

---

## Building a Standalone Executable

To compile the application into a single executable without requiring a local Python runtime:

```bash
pip install pyinstaller
pyinstaller --onefile --windowed --name Tetris src/main.py
```

The resulting binary will be output to the `dist/` folder.

---

## Configuration & Data Storage

Settings and high scores are managed in human-readable JSON files located in `data/`:

- **`data/settings.json`**:
  ```json
  {
    "palette": "STANDARD",
    "difficulty": "NORMAL",
    "keybindings": {
      "LEFT": "K_LEFT",
      "RIGHT": "K_RIGHT",
      "ROTATE": "K_UP",
      "SOFT_DROP": "K_DOWN",
      "HARD_DROP": "K_SPACE"
    }
  }
  ```

- **`data/scores.json`**:
  ```json
  {
    "classic": [
      { "name": "AAA", "score": 12500, "date": "2026-10-28" }
    ],
    "item_mode": [
      { "name": "ALX", "score": 15400, "date": "2026-10-28" }
    ]
  }
  ```
