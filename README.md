# 🎮 Tank Battle Simulator

> Advanced C++ tank battle simulator with dynamic .so loading, tournament framework, BFS pathfinding, and parallel execution. Test and compare AI algorithms automatically.

[![C++](https://img.shields.io/badge/C++-17-blue.svg)](https://en.cppreference.com/w/cpp/17)
[![CMake](https://img.shields.io/badge/CMake-3.10+-green.svg)](https://cmake.org/)

![Tank Battle Demo](docs/demo.gif)
*AI tanks battling it out on the grid*

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Architecture](#-architecture)
- [Installation](#-installation)
- [Usage](#-usage)
- [Tank AI Logic](#-tank-ai-logic)
- [Project Structure](#-project-structure)
- [Technical Highlights](#-technical-highlights)
- [Contributing](#-contributing)

---

## 🌟 Overview

Tank Battle Simulator is a sophisticated C++ framework for developing, testing, and comparing autonomous tank AI algorithms. The simulator runs automated tournaments where different AI strategies compete on various maps, providing comprehensive statistics to identify the best-performing algorithms.


---

## ✨ Key Features

### 🔌 Plugin Architecture
- Dynamic .so loading at runtime
- No recompilation needed for new algorithms
- Macro-based automatic registration

### ⚡ High Performance
- Multi-threaded parallel execution
- Configurable thread pools
- Efficient BFS pathfinding

### 🏆 Flexible Testing Modes

**Comparative Mode:**
- Test two algorithms across multiple game managers
- Evaluate robustness to different rule implementations

**Competition Mode:**
- Full tournament with multiple algorithms on multiple maps
- Smart pairing system ensures different matchups across maps
- Complete rankings with win/loss/tie statistics

---

## 🏗️ Architecture

The project consists of three main components that work together:

### High-Level Structure

```
┌─────────────────────────────────────────────────────┐
│                  Simulator                          │
│  • Loads algorithms and game managers (.so files)  │
│  • Builds GameContainers for each matchup          │
│  • Runs containers in parallel using threads       │
│  • Aggregates results into tournaments             │
└──────────────────┬──────────────────────────────────┘
                   │
                   │ creates multiple
                   ▼
        ┌─────────────────────┐
        │   GameContainer     │
        │                     │
        │  Contains:          │
        │  • GameManager      │
        │  • Player 1 & 2     │
        │  • Map/Board info   │
        │  • Algorithm names  │
        │                     │
        │  Self-contained     │
        │  game instance      │
        └──────────┬──────────┘
                   │
                   │ runs
                   ▼
        ┌─────────────────────┐
        │  Single Game        │
        │  (See diagram below)│
        └─────────────────────┘
```

**How GameContainers Work:**

The simulator doesn't run games directly. Instead, it:
1. **Builds GameContainers** - Each container packages everything needed for one complete game (map, two players, game manager, algorithm factories)
2. **Distributes to threads** - GameContainers are distributed across the thread pool
3. **Independent execution** - Each container runs its game independently and returns results
4. **Collects results** - Simulator gathers all GameResults and generates tournament statistics

This design enables efficient parallel execution since each GameContainer is self-contained.

### Single Game Architecture

This is how one game works internally:

```
                    ┌──────────────────┐
                    │   GameManager    │
                    │  (Game Engine)   │
                    └────┬─────────┬───┘
                         │         │
              ┌──────────┘         └──────────┐
              │                                │
              ▼                                ▼
        ┌──────────┐                    ┌──────────┐
        │ Player 1 │                    │ Player 2 │
        └────┬─────┘                    └────┬─────┘
             │                                │
             │ manages                        │ manages
             ▼                                ▼
    ┌─────────────────┐              ┌─────────────────┐
    │ TankAlgorithm 1 │              │ TankAlgorithm 2 │
    │ TankAlgorithm 2 │              │ TankAlgorithm 3 │
    │      ...        │              │      ...        │
    └─────────────────┘              └─────────────────┘
```

### How Information Flows During a Turn

```
┌─────────────────────────────────────────────────────────────┐
│ EXAMPLE: Tank requests battle information                  │
└─────────────────────────────────────────────────────────────┘

┌────────────────┐         ┌────────────────┐         ┌──────────┐
│  GameManager   │         │ TankAlgorithm  │         │  Player  │
└───────┬────────┘         └───────┬────────┘         └────┬─────┘
        │                          │                       │
   [1]  │─── getAction() ────────→ │                       │
        │                          │                       │
   [2]  │   ← GetBattleInfo ───────│                       │
        │                          │                       │
   [3]  │─── updateTankWithBattleInfo(tank, satelliteView) →│
        │                          │                       │
   [4]  │                          │ ←─ updateBattleInfo ──│
        │                          │    (battleInfo)       │
        │                          │                       │
        │  [Turn complete - tank now has updated info]     │
        │                          │                       │
```

**What happens:**
1. GameManager asks tank for an action
2. Tank returns `GetBattleInfo` (this is the action for this turn)
3. GameManager calls Player's `updateTankWithBattleInfo()`, passing the tank and a SatelliteView
4. Player creates a BattleInfo object and calls `tank.updateBattleInfo()` to inform the tank
5. Turn ends - the tank will use this information in future turns to make decisions

### Component Responsibilities

| Component | Role | Key Functions |
|-----------|------|---------------|
| **GameManager** | Game engine that runs the match | Manages turns, physics, collisions, victory conditions |
| **Player** | Coordinator for one team's tanks | Decides which tank acts and when to update info, creates BattleInfo, implements update strategy (round-robin or every 3 turns) |
| **TankAlgorithm** | Individual tank's AI brain | Makes movement/combat decisions based on BattleInfo |
| **BattleInfo** | Information container | Abstract class - you create a derived class with game state data |
| **SatelliteView** | Map observer | Provides `getObjectAt(x,y)` to see what's on each tile |

---

## 🚀 Installation

### Prerequisites
- **C++17** compatible compiler (GCC 7+, Clang 5+)
- **CMake** 3.10 or higher
- **Linux** environment (Ubuntu 20.04+ recommended)

### Build Instructions

```bash
# Clone the repository
git clone https://github.com/yourusername/tank-battle-simulator.git
cd tank-battle-simulator

# Build everything
cmake .
make

# Executables and .so files are generated in their respective directories
```

**Generated Files:**
- `Simulator/simulator_322213836_212054837` - Main executable
- `Algorithm/Algorithm_322213836_212054837.so` - Example algorithm
- `GameManager/GameManager_322213836_212054837.so` - Game manager

---

## 📖 Usage

### Comparative Mode
Test algorithm robustness across different game manager implementations:

```bash
cd Simulator
./simulator_322213836_212054837 -comparative \
  game_map=maps/simple.map \
  game_managers_folder=../GameManager/ \
  algorithm1=../Algorithm/Algorithm_A.so \
  algorithm2=../Algorithm/Algorithm_B.so \
  num_threads=4 \
  -verbose
```

**Output:** `comparative_results.txt` showing which algorithm wins more consistently

### Competition Mode
Full tournament with all algorithms competing on all maps:

```bash
./simulator_322213836_212054837 -competition \
  game_maps_folder=maps/ \
  game_manager=../GameManager/GameManager_322213836_212054837.so \
  algorithms_folder=algorithms/ \
  num_threads=8 \
  -verbose
```

**Output:** `competition_results.txt` with complete tournament standings

### Command-Line Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `-comparative` | flag | Run comparative mode |
| `-competition` | flag | Run competition mode |
| `game_map=<path>` | string | Path to .map file (comparative) |
| `game_maps_folder=<path>` | string | Folder with maps (competition) |
| `game_manager=<path>` | string | Path to game manager .so |
| `game_managers_folder=<path>` | string | Folder with game managers |
| `algorithm1=<path>` | string | First algorithm .so (comparative) |
| `algorithm2=<path>` | string | Second algorithm .so (comparative) |
| `algorithms_folder=<path>` | string | Folder with algorithms (competition) |
| `num_threads=<n>` | int | Number of threads (default: 1) |
| `-verbose` | flag | Enable detailed logging |

---

## 🤖 Tank AI Logic

Our reference implementation uses a dual-mode strategy:

### 🛡️ Evader Mode (Defensive)
**Triggered when:** Shell heading toward tank OR out of ammo near enemy

**Strategy:**
1. Simulate all shell trajectories
2. Identify safe positions
3. Move/rotate to avoid impact
4. If facing threat with ammo: shoot to intercept

### ⚔️ Chaser Mode (Offensive)
**Triggered when:** No immediate threats detected

**Strategy:**
1. Find nearest enemy using BFS pathfinding
2. Calculate optimal route
3. Rotate toward next waypoint
4. Move along path
5. Shoot when enemy in line of fire

### 🎯 Information Update Strategy

**Smart Update Logic:**
- **Multiple tanks (2+):** Each tank requests battle info on its turn in round-robin order - ensures always up-to-date information
- **Single tank:** Requests info once every 3 turns instead - reduces overhead when coordination isn't needed

This adaptive approach balances information freshness with performance.

---

## 📁 Project Structure

```
tank-battle-simulator/
├── Algorithm/                      # Your tank AI implementation
│   ├── TankAlgorithm_*.cpp/h      # AI logic (BFS, threat detection)
│   ├── Player_*.cpp/h             # Player coordinator
│   └── ConcreteBattleInfo.cpp/h   # Battle state management
│
├── GameManager/                    # Game engine implementation
│   ├── GameManager_*.cpp/h        # Main game loop
│   ├── GameBoard.cpp/h            # Map representation
│   ├── Logger.cpp/h               # Action logging
│   └── objects/                   # Game entities
│       ├── Tank.cpp/h             # Tank object
│       ├── Shell.cpp/h            # Projectile
│       ├── Wall.h                 # Obstacle
│       └── Mine.h                 # Hazard
│
├── Simulator/                      # Core framework
│   ├── main.cpp                   # Entry point
│   ├── Simulator.cpp/h            # Base simulator
│   ├── ComparativeSimulator.*     # Comparative mode
│   ├── CompetitiveSimulator.*     # Competition mode
│   ├── AlgorithmRegistrar.*       # Algorithm storage
│   ├── GameManagerRegistrar.*     # GM storage
│   ├── GameContainer.*            # Game setup
│   └── SOHandle.h                 # RAII .so wrapper
│
├── common/                         # Shared interfaces
│   ├── AbstractGameManager.h      # GM interface
│   ├── Player.h                   # Player interface
│   ├── TankAlgorithm.h            # Algorithm interface
│   ├── *Registration.h            # Registration macros
│   └── ...                        # Other interfaces
│
├── UserCommon/                     # Shared utilities
│   └── utils/
│       ├── Position.h             # Coordinate handling
│       ├── Direction.h            # Movement vectors
│       ├── MapUtils.*             # BFS pathfinding
│       └── DirectionUtils.*       # Rotation logic
│
└── CMakeLists.txt                 # Build configuration
```

---

## 💡 Technical Highlights

### Design Patterns & Architecture

| Pattern | Application | Benefit |
|---------|-------------|---------|
| **Factory** | Algorithm creation via lambdas | Dynamic instantiation without recompilation |
| **Singleton** | AlgorithmRegistrar & GameManagerRegistrar | Single global storage for all factories |
| **Strategy** | Swappable TankAlgorithm implementations | Easy to test different AI approaches |
| **Template Method** | AbstractGameManager interface | Consistent game loop across implementations |
| **RAII** | SOHandle for .so management | Automatic resource cleanup, exception-safe |
| **Plugin Architecture** | Dynamic .so loading with dlopen | Hot-swappable components at runtime |

### Key Technologies

- **C++17** features (std::function, unique_ptr, lambdas)
- **Multi-threading** with std::jthread and atomic operations
- **Dynamic loading** via dlopen/dlclose
- **CMake** build system for modular compilation
- **BFS pathfinding** for optimal route planning
- **Grid-based collision detection** for efficient physics


```
