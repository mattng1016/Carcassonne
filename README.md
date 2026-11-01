# Carcassonne

A Carcassonne-inspired tile-placement game built with **Godot 4.5** and **GDScript** as a collaborative CMPT course project.

Build a shared landscape of roads, cities, monasteries, and fields. Rotate and place tiles, claim features with meeples, and compete for points against computer opponents or other players over a local network.

## Features

- **Single-player against bots:** Play against one to four computer opponents, with three selectable bot types.
- **LAN multiplayer:** Host or join a two-to-five-player match using an IP address and port, with LAN discovery support.
- **Tile placement and rotation:** Snap tiles to a grid, match neighbouring edges, and highlight legal placement positions.
- **Meeple placement and scoring:** Claim connected features, award points to majority owners, return meeples from completed features, and score remaining features at the end of the game.
- **Game interface:** Player score cards, remaining-tile counts, placement animations, sound effects, and camera pan/zoom.
- **Automated test suite:** GUT unit and integration tests for board coordinates, tile rotation, placement rules, and scoring helpers.

## Getting started

### Requirements

- [Godot 4.5](https://godotengine.org/download/) — the standard build supports this GDScript project.
- Git, or a downloaded ZIP of this repository.

The project is configured to use Godot's **Forward+** renderer.

### Run in the editor

1. Clone the repository:

   ```sh
   git clone https://github.com/mattng1016/Carcassonne.git
   cd Carcassonne
   ```

2. Open Godot, select **Import**, and choose `project.godot`.
3. Let Godot finish importing assets and scanning scripts.
4. Press **F6** with `scenes/menu/mainMenu.tscn` open, or **F5** to run the configured main scene.
5. Choose **Single-Player** to configure bots, or **MultiPlayer** to open the LAN lobby.

**GUT setup note:** The repository includes GUT scripts, but `addons/gut/plugin.cfg` is missing even though it is enabled in `project.godot`. If Godot reports a missing plugin, install a complete Godot 4-compatible [GUT](https://github.com/bitwes/Gut) release into `addons/gut`. To play without the editor plugin, remove the stale GUT entry from the `[editor_plugins]` section of `project.godot`; this does not repair the test installation.

## How to play

1. A tile is drawn for the current player.
2. Move the tile preview to an empty cell next to the existing landscape.
3. Rotate the tile until its edges match every neighbouring tile, then place it.
4. Optionally place a meeple on an available feature of the newly placed tile, or skip.
5. Completed features are scored and eligible meeples return to their owners. Play continues until the deck is exhausted, followed by end-game scoring.

### Controls

| Action | Control |
| --- | --- |
| Move the tile preview | Mouse |
| Place a tile / select a meeple position | Left click |
| Rotate clockwise | Right click or **E** |
| Rotate counterclockwise | **Q** |
| Skip meeple placement | **Space** or the skip button |
| Toggle a meeple-placement hint during the meeple phase | **H** |
| Pan the camera | **WASD** or arrow keys |
| Zoom | Mouse wheel or supported trackpad gestures |
| Undo the current tile before placing a meeple, in local play | Undo button |

### LAN multiplayer

1. Connect the participating computers to the same local network.
2. On the host, open **MultiPlayer**, enter a username, choose the player count and port, and select **Host LAN**.
3. On each client, enter a username, the host's local IP address, and the same port, then select **Join LAN**.
4. Once the players have joined, the host selects **Start Match**.

The default game port is **7000**. The networking code uses Godot's `ENetMultiplayerPeer`. If a connection fails, check that the host address and port match and that the local firewall allows the game. This workflow is intended for LAN play; internet matchmaking is not included.

## Implementation highlights

### Board and tile representation

The board stores placed tiles in a dictionary keyed by `Vector2i` grid coordinates. Placement validation checks that a cell is empty, touches an existing tile, and matches all neighbouring edges. Tile resources define edge types, internal feature connections, and meeple positions; rotation updates that data alongside the visual orientation.

### Connected-feature scoring

`ScoreSystem.gd` uses an explicit stack and visited set to traverse connected road and city segments. It tracks unique tiles, open ends, and shields to determine completion and scoring. Fields use half-edge connectivity, while monasteries are checked through surrounding tiles. Majority ownership supports tied players receiving points.

### Computer opponents

`BotController.gd` generates legal placements across all four tile rotations and selects between greedy evaluation, a shallow minimax strategy, and the expert bot. `ExpertBot.gd` considers possible future tile draws and meeple choices. `FeatureDSU.gd` provides a disjoint-set structure with checkpoints and rollback for evaluating hypothetical moves.

### Multiplayer coordination

Networking is separated into session state, LAN discovery, and turn coordination. The host validates submitted actions and synchronizes tile draws, placements, meeple actions, and turn changes through RPCs.

## Project structure

```text
assets/                   Art, fonts, music, and sound effects
resources/                Tile definitions and deck resources
scenes/                   Menus, game board, tiles, meeples, and UI scenes
scripts/
  ai/                     Bot evaluation, search, and feature connectivity
  core/                   Game lifecycle, board, turns, menus, and UI
  feature_system/         Feature traversal and scoring
  meeple/                 Meeple placement and ownership
  network/                Sessions, discovery, and synchronized turns
  tile_system/            Tile data, rotation, and deck management
  ui/                     Score cards, hints, sounds, and overlays
tests/
  unit/                   Coordinate, rotation, and score-manager tests
  integration/            Placement, tile-data, and scoring-helper tests
addons/gut/               GUT testing framework
project.godot             Godot project configuration
```

## Tests

After completing the GUT installation and letting Godot import the project, run the suite from the GUT editor panel, or from the repository root:

```sh
godot --headless --path . -s addons/gut/gut_cmdln.gd -gdir=res://tests -ginclude_subdirs -gexit
```

Replace `godot` with the path to your Godot executable if it is not on your PATH.

The suite includes cases for occupied cells, disconnected placements, mismatched edges, rotation round trips, score accumulation, and tied ownership. These tests cover selected mechanics and helpers; they do not establish complete rule coverage or multiplayer correctness.

## Credits and license

Developed as a team course project. The implementation described above covers the shared codebase.

The repository includes an [MIT license](LICENSE), with the existing copyright notice credited to Satyar Salehian.D. GUT includes its own [license](addons/gut/LICENSE.md). Carcassonne is the board game that inspired this educational project.
