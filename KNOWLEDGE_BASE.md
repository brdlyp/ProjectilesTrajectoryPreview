# Projectiles Trajectory Preview (PTP) - Knowledge Base

## Project Overview

**Name:** Projectiles Trajectory Preview (PTP)  
**Type:** Minecraft Fabric Mod  
**Version:** 1.0.25  
**Minecraft Version:** 1.21.11  
**License:** MIT License  
**Author:** maDU59_  
**Homepage:** https://madu59.github.io/  
**Repository:** https://github.com/maDU59/ProjectilesTrajectoryPreview  
**Distribution:** [Modrinth](https://modrinth.com/mod/ptp) and [CurseForge](https://curseforge.com/minecraft/mc-mods/ptp)

### Purpose
This mod provides a visual preview of projectile trajectories in Minecraft, showing players where their arrows, snowballs, potions, and other throwable items will land before they launch them. It includes visual indicators for impact zones and customizable display options.

## Core Features

1. **Trajectory Visualization** - Shows the predicted path of projectiles before launch
2. **Target Highlighting** - Highlights the exact impact location
3. **Target Outlining** - Adds an outline to potential targets
4. **Customizable Display** - Extensive configuration options for colors, opacity, and styles
5. **Server-Client Handshake** - Anti-cheat mechanism to prevent unfair advantages in PVP

## Technical Architecture

### Technology Stack

- **Build System:** Gradle 
- **Mod Loader:** Fabric Loader 0.17.3
- **Java Version:** Java 21
- **Fabric API:** 0.139.4+1.21.11
- **Loom Version:** 1.13-SNAPSHOT
- **Optional Integration:** ModMenu 17.0.0-beta.2

### Project Structure

```
ProjectilesTrajectoryPreview/
├── src/
│   ├── main/                          # Server-side code
│   │   ├── java/fr/madu59/ptp/
│   │   │   ├── Ptp.java              # Main mod initializer
│   │   │   └── HandshakeNetworking.java  # Network protocol for mod detection
│   │   └── resources/
│   │       ├── fabric.mod.json       # Mod metadata
│   │       └── assets/ptp/
│   │           ├── icon.png
│   │           └── lang/             # Internationalization files
│   │               ├── en_us.json
│   │               ├── fr_fr.json
│   │               ├── de_de.json
│   │               ├── ja_jp.json
│   │               ├── ko_kr.json
│   │               ├── ru_ru.json
│   │               ├── zh_cn.json
│   │               └── zh_tw.json
│   └── client/                        # Client-side code
│       ├── java/fr/madu59/ptp/
│       │   ├── PtpClient.java        # Client mod initializer & main logic
│       │   ├── ProjectileInfo.java   # Projectile physics definitions
│       │   ├── PreviewImpact.java    # Impact calculation results
│       │   ├── RenderUtils.java      # Rendering utilities
│       │   └── config/
│       │       ├── Option.java       # Configuration option class
│       │       ├── SettingsManager.java  # Config management
│       │       └── configScreen/
│       │           ├── PtpConfigScreen.java     # Main config screen
│       │           ├── MyConfigListWidget.java  # Config UI widget
│       │           └── ModMenuApiImpl.java      # ModMenu integration
│       └── resources/
│           └── ptp.client.mixins.json  # Mixin configuration (currently empty)
├── .github/workflows/
│   └── build.yml                      # GitHub Actions CI/CD
├── build.gradle                       # Build configuration
├── gradle.properties                  # Project properties
├── settings.gradle                    # Gradle settings
└── README.md                          # Project documentation
```

## Core Components

### 1. Main Initialization (Ptp.java)

**Location:** `src/main/java/fr/madu59/ptp/Ptp.java`

**Responsibilities:**
- Mod initialization on both server and client
- Registers network packet handlers for client-server handshake
- Sets up payload types for custom networking

**Key Methods:**
- `onInitialize()` - Registers C2S and S2C packet types and handlers

### 2. Client Initialization (PtpClient.java)

**Location:** `src/client/java/fr/madu59/ptp/PtpClient.java`

**Responsibilities:**
- Client-side mod initialization
- Registers rendering hooks
- Manages keybindings
- Handles server mod detection via handshake
- Main rendering logic for trajectories

**Key Features:**
- **Server Detection:** Automatically detects if the mod is installed on the server
  - Always enabled in singleplayer
  - Sends handshake to multiplayer servers
  - Disables in multiplayer if server doesn't have the mod (anti-cheat)
  
- **Keybindings:**
  - `B` key (default) - Show item drop trajectory
  - Toggle key (unbound by default) - Toggle trajectory visibility

- **Rendering Hooks:**
  - `WorldRenderEvents.AFTER_ENTITIES` - Renders trajectories after entities
  - `ClientTickEvents.END_CLIENT_TICK` - Handles keybinding checks

**Key Methods:**
- `renderOverlay(WorldRenderContext)` - Main entry point for trajectory rendering
- `showProjectileTrajectory()` - Renders projectile trajectories
- `showItemTrajectory()` - Renders item drop trajectories  
- `calculateTrajectory()` - Simulates projectile physics
- `renderTrajectory()` - Renders the visual trajectory line
- `isEnabled()` - Checks if mod should be active

### 3. Projectile Physics (ProjectileInfo.java)

**Location:** `src/client/java/fr/madu59/ptp/ProjectileInfo.java`

**Purpose:** Defines physics parameters for all supported projectiles

**Supported Projectiles:**
1. **Bow & Arrows**
   - Gravity: 0.05
   - Drag: 0.99
   - Water Drag: 0.6
   - Velocity: 3.0 × pull strength
   - Offset: (0.2, -0.06, 0.2)
   - Order: Move → Drag → Gravity (MDG)

2. **Crossbow**
   - Gravity: 0.05
   - Drag: 0.99
   - Velocity: 3.15
   - Offset: (0, -0.06, 0.03)
   - Supports Multishot enchantment (±10° angle offset)
   - Firework rockets: No gravity, velocity 1.6

3. **Trident**
   - Gravity: 0.05
   - Drag: 0.99
   - Water Drag: 0.99
   - Velocity: 2.5
   - Offset: (0.2, 0.1, 0.2)
   - Doesn't work with Riptide enchantment

4. **Snowballs, Eggs, Ender Pearls**
   - Gravity: 0.03
   - Drag: 0.99
   - Water Drag: 0.8
   - Velocity: 1.5
   - Offset: (0.2, -0.06, 0.2)
   - Order: Gravity → Drag → Move (GDM)
   - Ender pearls bypass anti-cheat

5. **Wind Charge**
   - Gravity: 0
   - Drag: 0.95
   - Water Drag: 0.8
   - Velocity: 1.0
   - Offset: (0.2, -0.06, 0.2)

6. **Throwable Potions**
   - Gravity: 0.05
   - Drag: 0.99
   - Water Drag: 0.8
   - Velocity: 0.5
   - Pitch offset: -20°
   - Offset: (0.2, -0.06, 0.2)

7. **Experience Bottles**
   - Gravity: 0.07
   - Drag: 0.99
   - Water Drag: 0.8
   - Velocity: 0.7
   - Pitch offset: -20°
   - Bypasses anti-cheat

8. **Fishing Rod**
   - Gravity: 0.03
   - Drag: 0.92
   - Custom velocity calculation
   - Has water collision
   - Order: Gravity → Move → Drag (GMD)
   - Bypasses anti-cheat

9. **Item Drop**
   - Gravity: 0.04
   - Drag: 0.98
   - Water Drag: 0.98 × 0.99
   - Custom spawn position and velocity
   - Order: GMD

**Physics Simulation:**
- Maximum 200 iteration steps
- Three possible update orders:
  - MDG: Move → Drag → Gravity
  - GMD: Gravity → Move → Drag
  - GDM: Gravity → Drag → Move
- Handles underwater physics transitions
- Collision detection with blocks and entities
- Stops simulation at world min Y - 20

**Key Methods:**
- `getItemsInfo(ItemStack, Player, boolean)` - Returns physics info for held item
- `getDropTrajectory(Player)` - Returns physics info for dropped items
- `hasEnchantment(ItemStack, Enchantment)` - Checks for enchantments

### 4. Rendering System (RenderUtils.java)

**Location:** `src/client/java/fr/madu59/ptp/RenderUtils.java`

**Features:**
- Line rendering for trajectories (2.0f width)
- Filled box rendering for target highlights
- Wire box rendering for target outlines
- Camera-relative positioning
- Uses Minecraft's native rendering types

**Key Methods:**
- `renderFilledBox()` - Renders semi-transparent filled boxes
- `renderBox()` - Renders wireframe boxes
- `renderVector()` - Renders line segments
- `addChainedFilledBoxVertices()` - Internal vertex generation for filled boxes
- `renderLineBox()` - Internal vertex generation for wireframe boxes

### 5. Configuration System

#### SettingsManager.java

**Location:** `src/client/java/fr/madu59/ptp/config/SettingsManager.java`

**Features:**
- JSON-based configuration storage
- Dynamic option loading
- Color system with entity-based color coding
- Opacity system with pulsing effects

**Configuration Options:**

**Trajectory Settings:**
- `SHOW_TRAJECTORY` - Toggle trajectory visibility
  - ENABLED
  - TARGET_IS_ENTITY (only show if targeting an entity)
  - DISABLED
  
- `TRAJECTORY_COLOR` - Trajectory line color
  - DEPENDS_ON_TARGET (changes based on entity type)
  - RED, GREEN, BLUE, YELLOW, CYAN, MAGENTA, WHITE, PURPLE, BLACK
  
- `TRAJECTORY_OPACITY` - Line transparency
  - OPAQUE (255)
  - TRANSPARENT (100)
  - PULSING (animated 50-256)
  
- `TRAJECTORY_STYLE` - Line rendering style
  - SOLID (100% of segment)
  - DASHED (50% of segment)
  - DOTTED (15% of segment)

**Outline Settings:**
- `OUTLINE_TARGETS` - Toggle target outlines
- `OUTLINE_COLOR` - Outline color
- `OUTLINE_OPACITY` - Outline transparency

**Highlight Settings:**
- `HIGHLIGHT_TARGETS` - Toggle target highlighting
- `HIGHLIGHT_COLOR` - Highlight fill color
- `HIGHLIGHT_OPACITY` - Highlight transparency

**Other:**
- `ENABLE_OFFHAND` - Enable trajectory for offhand items

**Entity-Based Color Coding:**
- Player → BLUE
- Neutral Mob → YELLOW
- Ageable Mob (animals) → GREEN
- Monster → RED
- Other Mobs → PURPLE
- Living Entity → CYAN
- Other Entities → MAGENTA
- No entity (blocks) → WHITE

**Config File Location:** `.minecraft/config/ptp.json`

**Key Methods:**
- `loadSettings()` - Loads config from JSON
- `saveSettings()` - Saves config to JSON
- `setOptionValue()` - Updates option via command/UI
- `getARGBColorFromSetting()` - Converts settings to ARGB color
- `getColorFromSetting()` - Gets RGB color with entity context
- `convertColorToFloat()` - Converts RGB to float array for rendering

#### Option.java

**Location:** `src/client/java/fr/madu59/ptp/config/Option.java`

**Purpose:** Generic configuration option class with type safety

**Features:**
- Supports Boolean, Enum types
- Automatic value cycling
- Translation support via I18n
- Self-registration to SettingsManager

**Enums:**
- `State` - ENABLED, TARGET_IS_ENTITY, DISABLED
- `Opacity` - OPAQUE, TRANSPARENT, PULSING
- `Style` - SOLID, DASHED, DOTTED
- `Color` - RED, GREEN, BLUE, YELLOW, CYAN, MAGENTA, WHITE, PURPLE, BLACK, DEPENDS_ON_TARGET

### 6. Configuration UI

#### PtpConfigScreen.java

**Location:** `src/client/java/fr/madu59/ptp/config/configScreen/PtpConfigScreen.java`

**Features:**
- Accessible via ModMenu integration
- Accessible via `/ptpConfig` command
- Scrollable option list
- Organized into categories
- Auto-saves on close

**Categories:**
1. Trajectory Previsualization
2. Target Outlining
3. Target Highlighting

#### MyConfigListWidget.java

**Location:** `src/client/java/fr/madu59/ptp/config/configScreen/MyConfigListWidget.java`

**Purpose:** Custom scrollable widget for configuration options

**Entry Types:**
- `CategoryEntry` - Section headers
- `ButtonEntry` - Toggle buttons for enum/boolean options
- `SliderEntry` - Sliders for numeric values (not currently used)

**Features:**
- Custom scrollbar positioning
- Button click sound effects
- Live value updates
- Support for indented sub-options

#### ModMenuApiImpl.java

**Location:** `src/client/java/fr/madu59/ptp/config/configScreen/ModMenuApiImpl.java`

**Purpose:** Integration with ModMenu mod for config screen access

### 7. Networking (HandshakeNetworking.java)

**Location:** `src/main/java/fr/madu59/ptp/HandshakeNetworking.java`

**Purpose:** Custom packet protocol for mod detection

**Packet Types:**
- `HANDSHAKE_C2SPayload` - Client asks server if mod is installed
- `HANDSHAKE_S2CPayload` - Server confirms mod is installed

**Flow:**
1. Client joins multiplayer server
2. Client sends C2S handshake packet
3. If server has mod, it responds with S2C packet
4. Client enables/disables features based on response
5. In singleplayer, always enabled

**Anti-Cheat Purpose:** Prevents players from having an unfair advantage in PVP on vanilla servers

### 8. Impact Preview (PreviewImpact.java)

**Location:** `src/client/java/fr/madu59/ptp/PreviewImpact.java`

**Purpose:** Data class holding trajectory calculation results

**Fields:**
- `position` - Final impact position
- `impact` - Block hit result (if any)
- `entityImpact` - Entity hit (if any)
- `hasHit` - Whether trajectory hit something
- `trajectoryPoints` - List of Vec3 points forming the trajectory path

## Trajectory Calculation Algorithm

**Location:** `PtpClient.calculateTrajectory()`

**Process:**
1. Initialize position, velocity, and physics parameters
2. For each iteration (max 200):
   - Add current position to trajectory points
   - Apply physics in specified order (move/drag/gravity)
   - Check for entity collisions within AABB
   - Check for block collisions (with water handling)
   - Determine closest hit (entity vs block)
   - If hit detected, add final point and break
   - If below world minimum, stop simulation
   - Update position for next iteration
3. Return PreviewImpact with all trajectory data

**Entity Collision:**
- Excludes: spectators, dead entities, projectiles, items, XP orbs, ender dragon, local player
- Uses inflated bounding boxes (entity.getPickRadius())
- Ray-cast from previous to current position

**Block Collision:**
- Uses ClipContext with COLLIDER mode
- Supports water collision (fishing rod)
- Dynamic water physics when projectile enters water

**Water Physics Transition:**
- Projectiles without water collision change physics upon entering water
- Drag and gravity values switch to underwater variants
- Fishing rod always uses water collision mode

## Rendering Pipeline

**Hook:** `WorldRenderEvents.AFTER_ENTITIES`

**Process:**
1. Check if player exists
2. Get held item (main hand first, then offhand if enabled)
3. Get projectile info for item
4. Calculate trajectory
5. Render trajectory line
6. Render target highlight (if enabled)
7. Render target outline (if enabled)

**Trajectory Line Rendering:**
- Lerped hand-to-eye offset for smooth animation
- Style variations (solid/dashed/dotted)
- Crosshair marker at impact point (3-axis cross)
- Camera-relative coordinates
- 2.0f line width

**Target Rendering:**
- Highlight: Semi-transparent filled box
- Outline: Wireframe box
- Block targets: 1×1×1 box at block position
- Entity targets: Inflated bounding box

**Hand-to-Eye Delta:**
- Calculates offset from hand position to eye position
- Accounts for player rotation (yaw/pitch)
- Adjusts for main hand setting (left/right)
- Disabled in third-person camera mode
- Provides smooth visual connection from hand to trajectory start

## Configuration Methods

### 1. Via ModMenu
- Install ModMenu mod
- Click "Mods" button in main menu
- Find "Projectiles Trajectory Prediction"
- Click configure button

### 2. Via Command
```
/ptpConfig
```
Opens the configuration screen

### 3. Via Config File
Edit `.minecraft/config/ptp.json` manually

Example:
```json
{
  "SHOW_TRAJECTORY": "ENABLED",
  "TRAJECTORY_COLOR": "DEPENDS_ON_TARGET",
  "TRAJECTORY_OPACITY": "OPAQUE",
  "TRAJECTORY_STYLE": "SOLID",
  "OUTLINE_TARGETS": "ENABLED",
  "OUTLINE_COLOR": "DEPENDS_ON_TARGET",
  "OUTLINE_OPACITY": "OPAQUE",
  "HIGHLIGHT_TARGETS": "ENABLED",
  "HIGHLIGHT_COLOR": "DEPENDS_ON_TARGET",
  "HIGHLIGHT_OPACITY": "TRANSPARENT",
  "ENABLE_OFFHAND": "false"
}
```

## Internationalization

**Supported Languages:**
- English (en_us.json)
- French (fr_fr.json)
- German (de_de.json)
- Japanese (ja_jp.json)
- Korean (ko_kr.json)
- Russian (ru_ru.json)
- Chinese Simplified (zh_cn.json)
- Chinese Traditional (zh_tw.json)

**Translation Keys:**
- Config option names and descriptions
- Enum value labels
- Key binding names
- UI element labels

## Build & Development

### Building the Mod
```bash
./gradlew build
```

Output: `build/libs/ptp-1.0.25.jar`

### Development Setup
1. Clone repository
2. Import as Gradle project
3. Run `./gradlew genSources`
4. Use Fabric run configurations

### CI/CD
- GitHub Actions workflow: `.github/workflows/build.yml`
- Runs on: Ubuntu 24.04
- Java: Microsoft OpenJDK 21
- Triggers: Push and Pull Requests
- Artifacts: Uploaded to GitHub Actions

### Dependencies
- **Required:**
  - Fabric Loader ≥0.16.9
  - Minecraft ~1.21.11
  - Java ≥21
  - Fabric API (any version)

- **Optional:**
  - ModMenu (for config screen access)

## Known Limitations

1. **Random Offset:** Minecraft projectiles have random offsets, so actual impact may differ slightly from preview
2. **Modded Items:** Not yet supported - compatibility must be added manually
3. **Server Requirement:** Doesn't work on vanilla servers without the mod installed (anti-cheat design)
4. **Entity Types:** Some entity types (ender dragon, XP orbs, items) are excluded from targeting

## Anti-Cheat Design

The mod implements a server-client handshake system to prevent unfair advantages:

1. **Singleplayer:** Always enabled (no competitive advantage)
2. **Multiplayer with mod on server:** Enabled (server allows it)
3. **Multiplayer without mod:** Disabled (prevents PVP advantage)
4. **Bypass exceptions:** Some items (ender pearls, experience bottles, fishing rods, item drops) bypass this for quality-of-life features

## Future Enhancement Possibilities

Based on the codebase structure:

1. **Modded Projectile Support:** Add more projectile types via config
2. **Wind Effects:** Account for wind/weather
3. **More Visual Styles:** Additional trajectory styles beyond solid/dashed/dotted
4. **Accuracy Indicator:** Visual representation of random offset zones
5. **Distance Markers:** Show distance along trajectory
6. **Entity Prediction:** Account for moving target positions
7. **Bounce Mechanics:** Support for projectiles that bounce
8. **Custom Color Picker:** RGB color picker instead of preset colors

## Code Quality & Patterns

**Strengths:**
- Clean separation of client/server code
- Well-organized configuration system
- Type-safe option handling
- Comprehensive physics simulation
- Good internationalization support
- Proper use of Fabric API

**Architecture Patterns:**
- Singleton pattern for managers
- Data classes for immutable state
- Event-driven rendering
- Generic configuration options
- Factory methods for option creation

## File Locations Reference

**Configuration:**
- Config file: `.minecraft/config/ptp.json`

**Resources:**
- Translations: `src/main/resources/assets/ptp/lang/`
- Icon: `src/main/resources/assets/ptp/icon.png`

**Source Code:**
- Client: `src/client/java/fr/madu59/ptp/`
- Server: `src/main/java/fr/madu59/ptp/`
- Config UI: `src/client/java/fr/madu59/ptp/config/configScreen/`

## Distribution

**Platforms:**
- Modrinth: https://modrinth.com/mod/ptp
- CurseForge: https://curseforge.com/minecraft/mc-mods/ptp

**License:** MIT - Free to use in modpacks and modify

## Contact & Support

- **Author:** maDU59_
- **Homepage:** https://madu59.github.io/
- **GitHub:** https://github.com/maDU59/ProjectilesTrajectoryPreview
- **Issue Tracker:** GitHub Issues (for bug reports and feature requests)

---

*Knowledge Base Generated: February 3, 2026*  
*Based on version: 1.0.25*
