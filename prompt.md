# Project Specification: Twitch Chat Isometric Sandbox Game

Create a complex, browser-based multiplayer sandbox game that integrates directly with Twitch chat. The application allows a streamer (host) to run a persistent game world where chat viewers automatically spawn as characters and control their actions via chat commands.

## Technical Stack & Architecture
- **Core**: Vanilla JavaScript using ES Modules. No build steps or bundlers.
- **Rendering**: `Three.js` (via CDN) for a 3D isometric/orthographic view.
- **Twitch**: `tmi.js` for connecting to Twitch IRC.
- **Storage**: `IndexedDB` (wrapped in a helper) for saving worlds, player data, and maps locally.
- **Audio**: Web Audio API for spatial sound effects.
- **Realtime**: Websockets (mocked or environment-specific) for a "Live View" second-screen experience.

## Core Gameplay Mechanics

### Player Interaction
- **Spawning**: Viewers spawn as players upon their first chat message.
- **Energy System**: Players have an "Energy" battery that fills when they chat and drains over time. Actions require energy. If energy hits zero, the player becomes idle.
- **Commands**:
  - `!chop`: Find the nearest tree and cut it down.
  - `!gather`: Find nearest forage (logs, bushes, flowers) and collect it.
  - `!follow <username>`: Pathfind to and follow another player.
  - `!me`: Focus the host's camera on the player.
  - `!link <code>`: Link a remote browser session to the player account.

### Automated Behaviors (State Machine)
- Characters function autonomously based on their active command.
- **Woodcutting**: Characters find a tree, move to it using A* pathfinding, chop it (showing a progress bar), and fell it. Trees turn into Logs and spawn Bushes nearby.
- **Gathering**: Characters locate ground items (Logs, Bushes, Flowers), move to them, and harvest them into their inventory.
- **Cooperation**: Multiple players can chop the same tree to speed up the process.

### Progression
- **Skills**: Track XP for Woodcutting and Gathering.
- **Inventory**: Store collected resources (Logs, Leaves, Flowers).
- **Persistence**: All player stats, positions, and inventories must be saved to IndexedDB periodically.

## Game World & Physics

### Map Generation
- **Grid System**: The world is a 2D tile grid (Grass, Tree, Logs, Bushes, Flowers, Dirt).
- **Terrain Height**: Use Simplex noise to generate a heightmap. The terrain should have smooth slopes and elevation changes.
- **Regeneration**: Tools to regenerate specific features (Trees, Flowers, Terrain) without wiping player data.

### Physics & Movement
- **Pathfinding**: A* algorithm handling obstacles and map boundaries.
- **Collision**:
  - **Hard Collisions**: Trees and map edges block movement.
  - **Slope Collision**: Players cannot walk up steep cliffs (height difference > 1 unit).
  - **Soft Collision**: Players gently push each other apart to avoid overlapping.

## 3D Rendering (Three.js)

### Camera
- Use an `OrthographicCamera` to achieve an isometric look.
- Implement orbit controls: Middle-mouse drag to rotate (yaw) and tilt (pitch). Scroll to zoom.
- Camera smoothing when focusing on targets.

### Visual Style
- **Terrain**: A 3D plane geometry displaced by the heightmap data. Apply a repeating grass texture that aligns with the grid.
- **Entities**:
  - **Trees**: "Billboard" style sprites (vertical planes) that stand upright.
  - **Ground Items**: Flat planes (Logs, Bushes) that lay flat and conform to the angle/slope of the terrain mesh beneath them to prevent clipping.
- **Players**: 3D Spheres with distinct colors.
- **UI Overlays**: Render HTML-canvas based sprites above players showing:
  - Username.
  - Energy bar (using block characters).
  - Circular progress bar for current action (Chopping/Gathering).
  - Skill icons when active.

## User Interface & Management

### Dashboard
- **World Select**: Menu to Create, Delete, and Rename save slots.
- **Import/Export**: Functionality to export world data to JSON and import it back.
- **Settings**: Comprehensive menu to tweak game rates (XP gain, action duration), visual settings (render distance), and terrain generation parameters (seed, scale).

### Asset Manager
- **Custom Assets**: UI to upload custom PNG images to replace default sprites (Tree, Ground, Icons).
- **AI Generation**: Integration with an AI image generation API (e.g., Flux Schnell) to generate new pixel-art assets on the fly based on user prompts.
- **Library**: A "wheel" or list of generated assets to drag-and-drop into active slots.

## Live View (Second Screen)
- A separate interface for players to view their stats on a mobile device.
- **Pairing**: Host generates a code; user types `!link <code>` in chat to authenticate.
- **Rendering**: The Live View renders a 3D view of the player's immediate surroundings using the same Three.js renderer logic but localized to a map chunk sent via Websockets.
- **Inventory**: Displays real-time inventory counts and XP.

## Code Structure Guidelines
- Use a modular architecture: separate Physics, Renderer, Pathfinding, and Behaviors into distinct files.
- Use an Event/Loop driven update cycle (Game Loop) handling `delta time`.
- Ensure robust error handling for image loading and AI generation failures.

