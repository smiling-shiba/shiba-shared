# Research: A Modern TypeScript / Node.js Game Stack
## Browser-first 2D card RPG / roguelite with local worlds, multiplayer, moddability, Steam packaging, and optional Phoenix Channels

**Research date:** September 17, 2026

---

## Executive summary

For the game concept we have been sketching — a retro/pixel-art card RPG with an explorable Shandalar-like world, procedural servers/worlds, persona-specific decks, settlements, local/custom servers, official ladders, seasons, and a hard separation between the open-source game and commercial services — a modern web-first stack is very plausible.

The strongest **all-TypeScript baseline** I found is:

- **TypeScript** everywhere practical
- **React** for menus, cards, collection/deck building, shops, settings, dialogs, and accessibility-heavy UI
- **Phaser 4** for the world map, sprites, animation, particles, camera work, map rendering, and game-scene input
- **Vite** for browser development/builds
- **Node.js 24 LTS** for production Node services
- **Colyseus** if the authoritative game server is kept in Node/TypeScript
- **pnpm workspaces + Turborepo** for the monorepo
- **Zod** at serialization/network/mod/save boundaries
- **Zustand** for lightweight client UI state; optionally **XState v5** for high-level workflows that genuinely benefit from a state machine
- **SQLite** for local/custom world saves
- **PostgreSQL** for official persistent account/ladder/entitlement data
- **Dexie / IndexedDB** for browser-local caches and browser-only saves
- **Vitest** for unit/property-style tests and **Playwright** for browser integration/end-to-end tests
- **Tauri 2** as the first desktop-packaging candidate for Steam/GOG/itch builds

There is also a very serious server alternative:

> **TypeScript client + Elixir/Phoenix authoritative server**

Phoenix Channels maps *extremely* well to the “a world is a server/session with a group of connected players” model. Phoenix 1.8 Channels provides long-lived connections, topic multiplexing, cluster-aware PubSub, an official JavaScript client, Presence, and BEAM process isolation. A `DynamicSupervisor`/Registry-based design can map naturally to “one supervised world process per active world.”

The tradeoff is important: **Node/Colyseus wins at shared TypeScript code and lower conceptual overhead; Phoenix wins at process-oriented realtime architecture, fault isolation, supervision, presence, and clustering.**

I would not casually run both Node and Phoenix as co-equal authoritative game servers. Pick one authoritative runtime once the game rules stabilize. If Phoenix becomes the choice, keep cards/personas/world-generation definitions in a neutral declarative format so the client does not need to duplicate authoritative rules.

---

# 1. Where I started

You specifically suggested the “Awesome” repositories, and that was useful.

## Awesome JavaScript

The long-running `sorrycc/awesome-javascript` list still has useful category-level signal. Its game-engine section points toward the web-game ecosystem (including Phaser and PixiJS), and its realtime section points toward Socket.IO and `ws`.

Source:

- [Awesome JavaScript — sorrycc/awesome-javascript](https://github.com/sorrycc/awesome-javascript)

GitHub also maintains a JavaScript Game Engines collection that prominently includes PixiJS, Phaser, and melonJS:

- [GitHub Collection: JavaScript Game Engines](https://github.com/collections/javascript-game-engines)

## Awesome TypeScript

Interesting 2026 development: the well-known `awesome-typescript` repository was **archived on February 11, 2026**. The maintainer's explanation is basically that TypeScript became so pervasive that a generic TypeScript list stopped being useful curation.

That is actually a useful signal for this research: instead of asking “which game package supports TypeScript?”, the more useful question is now “which active game/runtime packages have good TypeScript ergonomics?”

Source:

- [Awesome TypeScript — dzharii/awesome-typescript](https://github.com/dzharii/awesome-typescript)

## Game-specific curated lists

Two lists were more useful for game-specific discovery:

- [MagicTools — ellisonleao/magictools](https://github.com/ellisonleao/magictools)
- [Awesome Gamedev — Calinou/awesome-gamedev](https://github.com/Calinou/awesome-gamedev)

MagicTools is particularly broad and surfaced engines such as Excalibur.js as well as the usual larger projects.

---

# 2. Recommended high-level architecture

I would treat this as a **web application that happens to be a game**, rather than trying to force every UI element through a canvas engine.

```text
                         ┌─────────────────────────────┐
                         │       Browser / Tauri        │
                         │                              │
                         │  React / CSS                 │
                         │  - cards                     │
                         │  - deck builder              │
                         │  - shop                      │
                         │  - collection                │
                         │  - dialogs / settings        │
                         │  - season UI                 │
                         │                              │
                         │  Phaser 4                    │
                         │  - overworld                 │
                         │  - sprites                   │
                         │  - tilemaps                  │
                         │  - animation / VFX           │
                         │  - camera / input            │
                         └──────────────┬───────────────┘
                                        │
                         WebSocket / Channels / HTTPS
                                        │
                  ┌─────────────────────┴─────────────────────┐
                  │                                           │
        Option A: Node/Colyseus                    Option B: Phoenix/BEAM
        authoritative worlds                      authoritative worlds
                  │                                           │
                  └─────────────────────┬─────────────────────┘
                                        │
                           Persistent official data
                                PostgreSQL
```

For local/custom worlds:

```text
Local world
├── seed + generation config
├── world-generation version
├── mutation/state records
├── settlement state
├── players / inventories
├── custom server settings
├── mod manifest
└── save metadata
```

The persistent commercial/account service does **not** need to own the entire generated map.

---

# 3. Rendering/game-engine candidates

## Candidate A — Phaser 4

### Current status

Phaser 4 is no longer hypothetical. Phaser lists:

- 4.0.0 — April 10, 2026
- 4.1.0 — April 30, 2026
- 4.2.0 — June 19, 2026
- 4.2.1 — July 9, 2026

Sources:

- [Phaser 4 releases](https://phaser.io/download/phaser4)
- [Phaser project templates](https://docs.phaser.io/phaser/getting-started/project-templates)
- [Phaser installation](https://docs.phaser.io/phaser/getting-started/installation)

Phaser provides official project templates for **React + Vite + TypeScript** and also plain Vite + TypeScript.

Phaser 4 also rebuilt its WebGL renderer around a new render-node architecture.

- [Phaser 4 renderer overview](https://phaser.io/news/2026/04/phaser-4-renderer-faster-cleaner-and-built-for-modern-games)

### Why it fits this game

Phaser is the best fit if we want a **game framework** rather than just a renderer.

It already gives us the kinds of systems the overworld will eventually want:

- scenes
- cameras
- input
- sprites
- animation
- tweening
- asset loading
- tilemaps
- particles
- collision/physics when useful

For this project, I would deliberately **not** put every card/menu into Phaser. Let Phaser own the “world canvas” and let normal HTML/CSS/React own information-heavy UI.

That yields a pleasant split:

```text
React/CSS                 Phaser
----------------------    ------------------------
deck builder              overworld map
card inspection           player sprite
collection                NPCs / encounters
shop                      settlement art
settings                  animated map objects
season progression        particles / transitions
dialogs                   camera movement
```

This also preserves web accessibility and normal CSS layout for text-heavy card-game screens.

### Risk

Phaser 4 is a very recent major release. I would pin the selected 4.x version and upgrade intentionally rather than automatically floating to every new release.

---

## Candidate B — PixiJS 8

PixiJS is a **2D rendering engine**, not as much of a batteries-included game framework.

Current PixiJS 8 documentation describes:

- scene graph / containers
- assets
- ticker
- WebGL/WebGL2
- optional WebGPU
- filters
- text
- sprites and graphics

The current renderer guide explicitly recommends **WebGL for production** while describing WebGPU as feature-complete but still subject to browser inconsistencies.

Sources:

- [PixiJS 8 introduction](https://pixijs.com/8.x/guides/getting-started/intro)
- [PixiJS renderer guide](https://pixijs.com/8.x/guides/components/renderers)
- [PixiJS architecture](https://pixijs.com/8.x/guides/concepts/architecture)

### Why choose Pixi instead

Pixi becomes attractive if we decide:

> “I want complete control over the game architecture and mostly need an extremely good 2D renderer.”

For this project that would mean building more of our own:

- scene management
- tile/world abstractions
- game object conventions
- collision helpers
- higher-level animation conventions
- editor/import workflow

That is not necessarily bad, but it is extra engine work that does not make the card/world design better.

### Assessment

**Excellent renderer; probably more low-level than this project needs for v1.**

---

## Candidate C — Excalibur.js

Excalibur is worth taking seriously because it is **written in TypeScript from the ground up** and specifically targets 2D web games.

Its stated philosophy is a flexible, batteries-included 2D engine.

Sources:

- [Excalibur documentation](https://excaliburjs.com/docs/)
- [Excalibur GitHub repository](https://github.com/excaliburjs/Excalibur)

### Advantage

It is probably the philosophically cleanest “TypeScript-first game engine” of the candidates.

### Main caution

As of this research, Excalibur still labels itself **pre-1.0**, and the project warns that breaking changes can occur between releases.

That does not make it bad. It just makes it less conservative than Phaser for a project we might keep alive for years.

### Assessment

**Very interesting contender for a prototype. I would evaluate it, but not automatically pick it over Phaser 4.**

---

## Candidate D — melonJS

melonJS surprised me during this research.

The current engine advertises:

- WebGPU, WebGL2, and Canvas fallback
- Tiled integration
- tilemaps
- physics
- cameras
- audio
- particles
- shaders
- ES modules
- TypeScript declarations
- Vite/TypeScript project scaffolding
- no external runtime dependencies

Source:

- [melonJS](https://melonjs.org/)
- [melonJS GitHub](https://github.com/melonjs/melonJS)

This is much more modern than the “old HTML5 game engine” impression its age can give.

### Assessment

**Credible alternative, particularly if its Tiled/tilemap workflow feels better in practice.**

---

# 4. Engine recommendation for this specific project

For a first real implementation, I would start with:

> **React + Phaser 4 + Vite + TypeScript**

Not because Phaser wins every benchmark, but because it minimizes how much “engine” we need to invent while still keeping our actual game logic outside Phaser.

A critical architectural rule:

> **Phaser should render and interact with the world. It should not become the domain model.**

For example:

```text
packages/world-model
    knows:
    - Tile
    - Settlement
    - Encounter
    - WorldMutation

apps/web/phaser
    knows:
    - how a Tile looks
    - how a Settlement animates
    - how to click an Encounter
```

The `Tile` should not *be* a Phaser sprite.

That separation makes:

- server authority easier
- headless tests easier
- modding easier
- a future renderer replacement possible
- Phoenix migration possible
- browser and desktop builds less coupled to the engine

---

# 5. React + CSS is genuinely useful here

This game has an unusually large amount of **information UI**:

- cards
- deck lists
- persona pages
- collections
- card filters
- shops
- quest lists
- progression tracks
- achievement lists
- season rewards
- settings
- friend/server browsers
- tooltips and rules explanations

Those are better served by ordinary web layout than by manually positioning canvas text.

So, somewhat hilariously after the CSS-only-game joke, I actually *would* use a lot of CSS.

A battle screen might be structurally:

```text
<div class="game-shell">
  <PhaserCanvas />
  <BattleHUD />
  <Hand />
  <InspectCardDialog />
</div>
```

The world is canvas. The hand can be DOM.

That also makes card animation easier than it may sound: CSS transforms, transitions, container queries, and Web Animations can handle a surprising amount before we need custom canvas animation.

---

# 6. Client state

There are three kinds of state and they should not be conflated.

## Authoritative game state

Examples:

- whose turn it is
- card zones
- player resources
- encounter state
- world mutations
- settlement ownership
- rewards

**Source of truth: server** in multiplayer/ladder.

The client keeps a synchronized representation.

## UI/view state

Examples:

- which card is hovered
- which modal is open
- selected deck filter
- camera preference
- current settings tab

For this, **Zustand** is a reasonable lightweight choice and has TypeScript guidance.

Sources:

- [Zustand docs](https://zustand.docs.pmnd.rs/)
- [Zustand TypeScript guide](https://github.com/pmndrs/zustand/blob/main/docs/learn/guides/beginner-typescript.md)

## Workflow state

Some flows may eventually become complex enough to justify **XState v5**:

```text
boot
 -> authenticating
 -> menu
 -> joiningWorld
 -> world
 -> enteringBattle
 -> battle
 -> rewards
 -> world
```

XState is built around state machines/statecharts and actor-model orchestration.

Source:

- [XState / Stately docs](https://stately.ai/docs)

I would **not** start by modeling every card effect as an XState machine. Use it only where explicit workflow transitions improve clarity.

---

# 7. Monorepo/build tooling

A modern layout could be:

```text
game/
├── apps/
│   ├── web/                   # React + Phaser + Vite
│   ├── desktop/               # Tauri shell/config
│   └── world-server/          # Node/Colyseus option
│
├── packages/
│   ├── rules/                 # pure deterministic game rules
│   ├── content/               # personas/cards/biomes definitions
│   ├── worldgen/              # seed -> base world
│   ├── protocol/              # network contracts
│   ├── save-format/           # snapshots/mutations/versioning
│   ├── mod-sdk/               # supported mod interfaces
│   ├── ui/                    # reusable React UI
│   └── test-fixtures/
│
├── services/
│   └── ladder-phoenix/        # only if Phoenix becomes server runtime
│
├── pnpm-workspace.yaml
└── turbo.json
```

## pnpm

pnpm has first-class workspaces, the workspace protocol, a shared lockfile, package filtering, and content-addressed dependency storage.

Source:

- [pnpm](https://pnpm.io/)

## Turborepo

Turborepo fits well when multiple apps/packages need coordinated:

- build
- test
- typecheck
- lint
- asset generation

I would use it for task dependency/caching, but keep each package independently understandable.

Source:

- [Turborepo](https://turborepo.com/)

## Node version

As of September 17, 2026:

- Node 26 is Current.
- **Node 24 “Krypton” is LTS.**
- Node’s own guidance says production applications should use Active LTS or Maintenance LTS releases.

So I would target:

> **Node.js 24 LTS**

rather than chasing Node 26 merely because it is newer.

Source:

- [Node.js release status](https://nodejs.org/en/about/previous-releases)

---

# 8. Vite

Vite is the obvious frontend build/dev-server choice here, and it is already part of the official Phaser templates and Tauri documentation.

Tauri’s current Vite guide is written for **Vite 8**.

Sources:

- [Phaser project templates](https://docs.phaser.io/phaser/getting-started/project-templates)
- [Tauri + Vite guide](https://tauri.app/start/frontend/vite/)

One important build detail:

> Keep a dedicated TypeScript type-check task.

Fast web bundlers commonly transpile TypeScript without treating the type checker as the compilation gate.

A CI pipeline should explicitly run something like:

```text
pnpm typecheck
pnpm test
pnpm build
pnpm e2e
```

---

# 9. Runtime validation: Zod

Static TypeScript types disappear at runtime.

This game has many boundaries where data is untrusted or versioned:

- network messages
- imported server saves
- mod manifests
- card/content packs
- world configuration
- share codes
- commerce entitlements
- persisted browser data

Zod 4 is stable and works in modern browsers and Node.

Source:

- [Zod](https://zod.dev/)

I would use Zod schemas for *boundary data*, not for every internal object.

Example conceptually:

```ts
const WorldConfig = z.object({
  formatVersion: z.number().int(),
  worldgenVersion: z.string(),
  seed: z.string(),
  mapSize: z.enum(["small", "medium", "large"]),
  pvp: z.boolean(),
  lootMultiplier: z.number().positive()
})
```

---

# 10. Deterministic world generation

This system matters more than the renderer.

The world should be generated from:

```text
worldgen algorithm version
        +
seed
        +
server configuration
        =
immutable base world
```

Then store only state that diverges from that base:

```text
Base world
  +
mutation snapshot / log
  =
current world
```

For example:

```text
seed: "FISHERMAN-77348"
worldgen: "3.1"
config:
  map_size: large
  biome_profile: coastal

mutations:
  tile 3002 => encounter_completed
  tile 4041 => settlement(player_18)
  tile 4410 => boss_defeated
```

## Noise

`simplex-noise` remains a compact TypeScript/JavaScript option and supports injecting a seeded PRNG such as `alea`.

Sources:

- [simplex-noise.js](https://github.com/jwagner/simplex-noise.js/)
- [simplex-noise on npm](https://www.npmjs.com/package/simplex-noise)

## Important design rule

**Never treat the seed alone as sufficient forever.**

Store a `worldgen_version` with it.

If the algorithm changes between releases, this:

```text
seed = 123
```

may generate a different map next year.

This:

```text
worldgen = 2
seed = 123
```

gives us a migration/compatibility contract.

Also avoid letting random game code casually call `Math.random()`. Pass explicit seeded RNG objects into procedural systems.

---

# 11. Save format

For a local/custom server, I like:

```text
worlds/
  goblin-county/
    world.sqlite
    server.json
    mods/
```

or even a single SQLite file if all metadata can be represented cleanly there.

Possible tables:

```text
world_metadata
players
settlements
tile_mutations
encounters
inventories
decks
quests
server_settings
mod_state
snapshots
```

That gives the “server == save” idea a concrete form.

## SQLite in Node 24

Node now has a built-in `node:sqlite` module, but in the Node 24.21 documentation it is still marked **release candidate stability (1.2)**.

So I would *watch it*, but I would not make “must use Node's built-in SQLite API” an architectural requirement yet.

Source:

- [Node 24 `node:sqlite`](https://nodejs.org/download/release/latest-v24.x/docs/api/sqlite.html)

## Kysely

Kysely is a mature TypeScript SQL query builder supporting SQLite and PostgreSQL.

Source:

- [Kysely](https://www.kysely.dev/)
- [Kysely GitHub](https://github.com/kysely-org/kysely)

That gives us one pleasant query layer if the Node backend needs both local SQLite and official PostgreSQL.

---

# 12. Browser-local persistence

For browser-only local data, IndexedDB is the standard persistence mechanism.

Dexie 4 provides a much friendlier TypeScript API over IndexedDB.

Source:

- [Dexie TypeScript docs](https://dexie.org/docs/Typescript)
- [Dexie API](https://dexie.org/docs/API-Reference)

Good uses:

- cached art/content manifests
- client settings
- remembered servers
- offline local practice saves
- draft decks
- downloaded campaign metadata

I would not use browser IndexedDB as the official multiplayer authority.

---

# 13. The Node multiplayer path: Colyseus

If we stay all TypeScript, Colyseus is the standout package from this research.

It is not just “a WebSocket wrapper.” Its abstractions line up eerily well with this game.

Sources:

- [Colyseus documentation](https://docs.colyseus.io/)
- [Colyseus Rooms](https://docs.colyseus.io/room)
- [Colyseus state synchronization](https://docs.colyseus.io/state)
- [Colyseus matchmaking](https://docs.colyseus.io/matchmaker)

## Room ≈ world/run

A Colyseus `Room` encapsulates:

- connected clients
- state
- game logic
- lifecycle
- visibility/access
- reconnection

That is almost exactly our model of:

> “Goblin County is a world/server/save that several players can join.”

## Authoritative state

Colyseus explicitly uses a server-authoritative state model:

- server mutates state
- clients request actions
- clients receive synchronized state

This matches the ladder security philosophy we already chose.

## Binary state patches

Colyseus tracks property changes and sends binary delta patches rather than retransmitting the whole room state continuously.

That is useful, although for this game I would still avoid putting the entire procedurally generated map into live synchronized room state.

Use:

```text
join:
  seed
  generation config
  mutation snapshot

live:
  mutation events
  players
  currently relevant encounter state
```

## Matchmaking/lobbies

Colyseus supplies:

- `joinOrCreate`
- room IDs
- visibility flags
- live lobby listings
- queue rooms
- standalone matchmaker options

Again, unusually close to the design we had already invented.

## Card-game precedent

Colyseus's current learning examples include an authoritative turn-based UNO card-game demo.

Source:

- [Colyseus learning examples](https://docs.colyseus.io/learn)

## Extremely relevant community template

In June 2026, Phaser highlighted a community-built **TypeScript online-game template** combining:

- Phaser
- React
- Colyseus
- Electron
- Turborepo
- Vite

That does not mean we should clone it blindly, but it is useful evidence that this exact family of technologies is already being composed into a coherent monorepo.

Source:

- [Phaser: TypeScript Online Game Template](https://www.phaser.io/news/2026/06/typescript-online-game-template-phaser-colyseus-react-and-electron-in-one-monorepo)

---

# 14. The Elixir/Phoenix multiplayer path

Your interruption was correct: a lot of the things that make Colyseus attractive in Node are simply **native architectural territory for Elixir/Phoenix**.

Phoenix deserves to be treated as a first-class option, not an appendix.

## Phoenix Channels

Phoenix 1.8.14 documentation describes Channels as long-lived realtime connections built around topics. Clients can join multiple channels over one connection.

Phoenix also uses PubSub underneath Channels so broadcasts can cross nodes in a cluster.

Source:

- [Phoenix 1.8 Channels](https://phoenix.hexdocs.pm/channels.html)

Phoenix ships an **official JavaScript Channels client**.

- [Phoenix JavaScript client](https://phoenix.hexdocs.pm/js/)

The JavaScript client also automatically attempts channel rejoin using backoff after connection trouble.

## Presence

Phoenix Presence can replicate presence information across a cluster and provides join/leave diffs to clients.

Source:

- [Phoenix Presence guide](https://phoenix.hexdocs.pm/presence.html)

That maps naturally to:

- world player list
- friends-online list
- lobby occupancy
- reconnect visibility
- “who is currently at this table/world?”

Presence should remain ephemeral presence metadata, not become the world database.

## PubSub / clustering

Phoenix PubSub's default adapter uses Distributed Elixir between nodes.

Source:

- [Phoenix.PubSub](https://phoenix-pubsub.hexdocs.pm/Phoenix.PubSub.html)

This is attractive for an official world cluster because a client does not need to care which physical Phoenix node owns another player's connection.

## One world as a supervised process

The Elixir process model maps beautifully to the game's conceptual model:

```text
WorldSupervisor
├── World("goblin-county")
├── World("dead-sea-scrolls")
├── World("fishermans-folly")
└── ...
```

`DynamicSupervisor` exists specifically for starting supervised children dynamically.

Sources:

- [Elixir DynamicSupervisor](https://elixir.hexdocs.pm/dynamic-supervisor.html)
- [Elixir Supervisor](https://elixir.hexdocs.pm/1.19.2/Supervisor.html)

A possible model:

```text
Phoenix Socket
  │
  ├── Channel topic: "world:abc123"
  │
  └── sends player intent
           │
           ▼
      World process
      ├── authoritative state
      ├── players
      ├── encounter coordination
      └── mutation buffer
           │
           ▼
        PubSub
           │
           ▼
      connected Channels
```

This is a remarkably natural fit for the “each run is its own server” idea.

---

# 15. Colyseus vs Phoenix Channels

| Concern | Node + Colyseus | Elixir + Phoenix |
|---|---|---|
| Client language | TypeScript | TypeScript/JS client, Elixir server |
| Shared client/server TS types | Excellent | Protocol schemas required |
| Room/world abstraction | Built in | Natural to model with supervised processes |
| Matchmaking | Built in | Build it from Phoenix/OTP/application logic |
| State delta sync | Built in binary schema patches | Explicit protocol; you decide what to push |
| Reconnection | Built-in game framework support | Channel auto-rejoin + app-level world/session recovery |
| Presence | Framework infrastructure | Phoenix Presence is excellent |
| Cluster messaging | Supported scaling infrastructure | Native Phoenix PubSub / BEAM clustering |
| Fault isolation | Node process/room model | OTP supervision is a major strength |
| Developer pool | Larger JS/TS pool | Smaller Elixir pool |
| Rule code reuse with client | Very easy | Do not duplicate authoritative rules |
| Self-hosting for JS developers | Very approachable | Requires BEAM release/container knowledge |
| Browser-only local mode | Easy to share TS logic | Server runtime cannot simply execute in browser |
| “one process per active world” mental model | Simulated by framework objects/processes | Extremely idiomatic BEAM architecture |

## The real decision

The question is not:

> “Can Phoenix do WebSockets?”

Obviously yes.

The real decision is:

> **How much do we value a single TypeScript runtime versus BEAM's server architecture?**

For a twitch action game, Colyseus's integrated prediction/netcode machinery may be a substantial advantage.

For *this* game — card battles, map mutation, settlements, world events, turn/state-heavy mechanics — the networking cadence is much less demanding. Phoenix becomes especially compelling because our problem is dominated by **many isolated stateful sessions**, not 120 Hz physics.

---

# 16. Do not duplicate the authoritative rule engine

If Phoenix wins, avoid this:

```text
TypeScript rules implementation
+
Elixir rules implementation
```

with both independently determining whether a card can be played.

That eventually produces horrifying parity bugs.

Instead:

```text
cards/personas/content
     ↓
neutral declarative definitions
     ↓
authoritative server interprets rules
     ↓
client renders the results
```

For example:

```json
{
  "id": "fisherman.bottom_feeder",
  "cost": 3,
  "target": "draw_pile",
  "effects": [
    {
      "op": "play_bottom_card",
      "owner": "target_player"
    }
  ]
}
```

The client needs enough semantics to:

- display the card
- show legal-target hints provided/calculated from safe data
- preview text
- animate server results

It does **not** need to be authoritative.

This also makes:

- mods
- DLC persona packs
- server-managed ladder cards
- validation
- content tooling

much easier.

Some exceptional mechanics will still need code, but a small rule-operation vocabulary can cover a surprising amount.

---

# 17. Which server approach I would prototype first

Because the original research target was “modern TypeScript + Node.js + game packages,” I would still make the **first vertical slice all TypeScript**:

```text
React + Phaser
      │
      ▼
   Colyseus
      │
      ▼
 SQLite
```

Why:

1. fastest shared iteration
2. one language while game rules are still changing violently
3. Colyseus already solves the exact networking shape
4. easier open-source contributor onboarding
5. easy local server distribution

But I would deliberately keep these boundaries clean:

```text
protocol/
rules/
worldgen/
persistence/
```

Then, before building the official ladder infrastructure, evaluate Phoenix again.

If we discover we want:

- huge numbers of long-lived worlds
- strong process isolation
- supervision semantics
- easy clustered presence
- BEAM-native operational behavior

then **Phoenix is not a rewrite of the game client**. It is a replacement of the authoritative server behind a well-defined protocol.

And if the developer maintaining the server already enjoys Elixir, I would take Phoenix even more seriously. Developer affinity matters for a long-lived indie service.

---

# 18. Desktop / Steam packaging

## Tauri 2

Tauri is frontend-agnostic and accepts a normal HTML/CSS/JavaScript frontend. Its documentation recommends Vite for SPA-style JavaScript/TypeScript frontends.

Sources:

- [What is Tauri?](https://tauri.app/start/)
- [Tauri frontend configuration](https://v2.tauri.app/start/frontend/)
- [Tauri + Vite](https://tauri.app/start/frontend/vite/)

That lets the same game frontend become:

```text
Browser
  └── React + Phaser

Desktop
  └── Tauri
       └── same React + Phaser build
```

Tauri uses the operating system webview rather than bundling an entire Chromium engine, which is attractive for a 2D game that does not inherently need Electron.

## Electron

Electron remains a reasonable fallback, particularly because its Node integration and ecosystem make certain desktop integrations straightforward.

Notably, the June 2026 Phaser/Colyseus community monorepo uses Electron rather than Tauri.

I would prototype **Tauri first**, then switch only if Steamworks/native integration proves materially more awkward than Electron.

## Steam integration

Steam does not care that the rendered game began life as TypeScript in a browser. The shipped artifact is a desktop application.

The native wrapper layer is also a clean place for:

- Steam identity
- achievements
- overlay hooks
- cloud saves
- platform entitlements

That platform adapter should remain separate from the open-source game-domain packages.

---

# 19. The open-source / private-commercial boundary

The repository can be genuinely playable:

```text
PUBLIC
├── client
├── world server
├── rules engine
├── personas/cards shipped with base game
├── world generation
├── local saves
├── mod API
└── protocol
```

while official services remain separate:

```text
PRIVATE / HOSTED
├── commerce adapter
├── payment-provider integration
├── promotion administration
├── official account service
├── entitlement service
├── official ladder operations
└── commercial analytics
```

The public client can depend on a tiny abstraction such as:

```ts
interface CommerceProvider {
  getCatalog(): Promise<Catalog>
  beginPurchase(sku: string): Promise<PurchaseResult>
  getEntitlements(): Promise<Entitlement[]>
}
```

The open-source implementation can return no paid catalog.

The official Steam/web build injects the official provider.

This is much cleaner than attempting to hide game logic.

---

# 20. Mod architecture

I would make mods **data-first**.

### Level 1 — server configuration

```yaml
world:
  size: large
  pvp: false

economy:
  xp_multiplier: 2
  drop_multiplier: 1.5
```

### Level 2 — content packs

```text
mods/fishermans-revenge/
├── mod.json
├── cards/
├── personas/
├── encounters/
├── biomes/
└── assets/
```

### Level 3 — trusted code extensions

Only local/self-hosted servers explicitly opted into code mods should execute arbitrary TypeScript/JavaScript extensions.

Official ladder:

```text
mods = disabled
content manifest = official signed manifest
authoritative card definitions = ladder server
```

That preserves the “have the damn game code” philosophy without letting a random multiplayer server silently send executable code to clients.

---

# 21. Protocol design

Do not start by inventing an exotic binary protocol.

For this game's message rates, start boring:

```json
{
  "type": "play_card",
  "request_id": "01...",
  "card_instance_id": "c_901",
  "target_id": "enemy_12"
}
```

and:

```json
{
  "type": "card_resolved",
  "request_id": "01...",
  "events": [...]
}
```

Validate at the boundary.

If profiling later shows payload size is a real cost:

- Colyseus already provides binary state patches for its synchronized schema.
- Phoenix supports custom socket serializers if we later want a binary payload format.

The world itself should usually be:

```text
seed + worldgen version + config + mutation state
```

rather than streamed as a giant binary grid.

---

# 22. Testing strategy

This game is an excellent candidate for aggressive headless testing because so much of it is deterministic state transformation.

## Vitest

Use Vitest for:

- card rules
- persona globals
- combat resolution
- seeded world generation
- map validation
- settlement movement
- deck legality
- save migrations
- protocol schemas

Source:

- [Vitest](https://vitest.dev/guide/)

Example conceptual test:

```ts
it("Dead Sea Scrolls remains achievable", () => {
  const state = createNecromancerMatch()

  // ...play a legal game without touching graveyard...

  expect(state.achievements).toContain("dead_sea_scrolls")
})
```

## Golden world-generation tests

For every worldgen version, keep fixtures:

```text
seed "abc" + config X
  => map hash Y
```

That catches accidental seed incompatibility instantly.

## Playwright

Use Playwright for:

- deck builder flows
- browser joins a world
- reconnect behavior
- browser refresh restores session
- shop/catalog presentation
- major battle UI paths
- browser compatibility

Playwright supports Chromium, Firefox, and WebKit.

Source:

- [Playwright](https://playwright.dev/docs/intro)

---

# 23. A practical package shortlist

This is intentionally smaller than an “install all the things” starter kit.

## Core client

```text
react
react-dom
phaser
zustand
zod
```

Optional when complexity earns it:

```text
xstate
```

## Build/workspace

```text
typescript
vite
pnpm
turbo
```

## Node authoritative server option

```text
colyseus
@colyseus/sdk
```

Database/query layer as needed:

```text
kysely
<selected SQLite driver>
pg
```

## World generation

```text
simplex-noise
alea
```

Do not make the engine own world generation.

## Browser persistence

```text
dexie
```

## Tests

```text
vitest
@playwright/test
```

## Desktop

```text
@tauri-apps/cli
@tauri-apps/api
```

plus native/platform plugins only when needed.

## Phoenix option

Server-side:

```text
Phoenix 1.8
Phoenix Channels
Phoenix.Presence
Phoenix.PubSub
Ecto / Ecto SQL
PostgreSQL
```

Client:

```text
phoenix
```

---

# 24. Proposed repository shape

## All-TypeScript / Node version

```text
cardworld/
├── apps/
│   ├── web/
│   │   ├── src/
│   │   │   ├── game/              # Phaser integration
│   │   │   ├── screens/           # React screens
│   │   │   ├── components/
│   │   │   └── stores/
│   │   └── vite.config.ts
│   │
│   ├── desktop/
│   │   └── src-tauri/
│   │
│   └── world-server/
│       ├── rooms/
│       ├── persistence/
│       └── app.config.ts
│
├── packages/
│   ├── content/
│   │   ├── cards/
│   │   ├── personas/
│   │   ├── encounters/
│   │   └── biomes/
│   │
│   ├── rules/
│   ├── protocol/
│   ├── worldgen/
│   ├── save-format/
│   ├── mod-sdk/
│   └── shared-test-fixtures/
│
├── pnpm-workspace.yaml
└── turbo.json
```

## Phoenix-authoritative version

```text
cardworld/
├── apps/
│   ├── web/
│   └── desktop/
│
├── packages/
│   ├── content-schema/
│   ├── protocol/
│   ├── worldgen-client/
│   └── ui/
│
├── server/
│   └── cardworld_phoenix/
│       ├── lib/
│       │   ├── worlds/
│       │   ├── battles/
│       │   ├── matchmaking/
│       │   └── cardworld_web/channels/
│       └── priv/
│
└── content/
    ├── cards/
    ├── personas/
    ├── biomes/
    └── encounters/
```

With Phoenix, I would move even harder toward neutral content definitions so the TS client does not try to share executable rule code.

---

# 25. Suggested first vertical slice

Before settlements, seasons, shops, commerce, ladder signing, or seventeen HIMBO rarities:

### World

- seed generates one small deterministic map
- player moves on it
- encounter tiles exist
- completed encounter turns gray
- save/reload works

### Battle

- one persona
- ~12 cards
- hand / draw / discard
- one simple enemy
- one global persona effect
- server validates every action

### Multiplayer

- create named world
- private/public setting
- second browser joins
- both players see movement/state mutations
- disconnect/reconnect works

### Persistence

- server closes
- world starts again from SQLite
- same seed
- same mutations
- same settlement/player positions

### Desktop

- wrap the existing web client
- verify local save paths
- verify gamepad/mouse behavior
- no Steamworks yet

That slice answers almost every architectural question without prematurely building a live service.

---

# 26. Things I would explicitly postpone

These are attractive but unnecessary for the first architecture proof:

- WebGPU-specific rendering
- custom binary network protocol
- Kubernetes
- microservices
- Redis unless scaling requires it
- Steamworks integration
- payment integration
- official ladder cryptographic attestation
- user-generated executable client mods
- complicated ECS
- a custom game editor
- a separate asset CDN
- cross-region world migration
- event sourcing every microscopic action forever

The game has a lot of potential complexity already. The stack should remove complexity rather than become another game system.

---

# 27. Bottom line

A very credible 2026 implementation is:

```text
TypeScript
├── React + CSS          information-heavy game UI
├── Phaser 4             world rendering / animation
├── Vite                 dev + web builds
├── Zustand              local UI state
├── Zod                  runtime boundaries
├── deterministic rules/worldgen packages
└── pnpm + Turborepo     monorepo

Node 24 LTS
├── Colyseus             authoritative worlds (if all-TS)
├── SQLite               local/custom world persistence
├── PostgreSQL           central official persistence
└── Kysely               typed SQL

Desktop
└── Tauri 2              web build -> native desktop shell

Testing
├── Vitest
└── Playwright
```

And the alternative I would keep on the architecture board in large friendly letters is:

```text
TypeScript + React + Phaser client
             │
             ▼
     Phoenix 1.8 Channels
             │
       supervised worlds
             │
        Ecto/Postgres
```

**Node/Colyseus is the easiest path to one-language development. Phoenix is arguably the more elegant server model for this exact “many stateful worlds” concept.**

The important decision is not needed on day one.

The client, content format, protocol, rules model, and deterministic world generator can be designed so that the server boundary stays clean. Build the game loop first; then let actual load, operational needs, and developer preference determine whether the authoritative runtime remains Node or moves to the BEAM.

---

# Sources / reading list

## Curated discovery lists

1. Awesome JavaScript  
   https://github.com/sorrycc/awesome-javascript

2. Awesome TypeScript (archived Feb. 11, 2026; useful historical index)  
   https://github.com/dzharii/awesome-typescript

3. MagicTools  
   https://github.com/ellisonleao/magictools

4. Awesome Gamedev  
   https://github.com/Calinou/awesome-gamedev

5. GitHub JavaScript Game Engines collection  
   https://github.com/collections/javascript-game-engines

## Game engines / rendering

6. Phaser 4 releases  
   https://phaser.io/download/phaser4

7. Phaser project templates  
   https://docs.phaser.io/phaser/getting-started/project-templates

8. Phaser installation  
   https://docs.phaser.io/phaser/getting-started/installation

9. Phaser 4 renderer article  
   https://phaser.io/news/2026/04/phaser-4-renderer-faster-cleaner-and-built-for-modern-games

10. PixiJS 8 introduction  
    https://pixijs.com/8.x/guides/getting-started/intro

11. PixiJS renderers  
    https://pixijs.com/8.x/guides/components/renderers

12. PixiJS architecture  
    https://pixijs.com/8.x/guides/concepts/architecture

13. Excalibur.js docs  
    https://excaliburjs.com/docs/

14. Excalibur.js GitHub  
    https://github.com/excaliburjs/Excalibur

15. melonJS  
    https://melonjs.org/

16. melonJS GitHub  
    https://github.com/melonjs/melonJS

## Multiplayer / Node

17. Colyseus main documentation  
    https://docs.colyseus.io/

18. Colyseus Rooms  
    https://docs.colyseus.io/room

19. Colyseus state synchronization  
    https://docs.colyseus.io/state

20. Colyseus matchmaking  
    https://docs.colyseus.io/matchmaker

21. Colyseus learning examples  
    https://docs.colyseus.io/learn

22. Phaser article: Phaser + Colyseus + React + Electron TypeScript monorepo  
    https://www.phaser.io/news/2026/06/typescript-online-game-template-phaser-colyseus-react-and-electron-in-one-monorepo

## Elixir / Phoenix

23. Phoenix 1.8 Channels  
    https://phoenix.hexdocs.pm/channels.html

24. Phoenix JavaScript client  
    https://phoenix.hexdocs.pm/js/

25. Phoenix Presence  
    https://phoenix.hexdocs.pm/presence.html

26. Phoenix.PubSub  
    https://phoenix-pubsub.hexdocs.pm/Phoenix.PubSub.html

27. Elixir DynamicSupervisor  
    https://elixir.hexdocs.pm/dynamic-supervisor.html

28. Elixir Supervisor  
    https://elixir.hexdocs.pm/1.19.2/Supervisor.html

29. Ecto PostgreSQL adapter  
    https://hexdocs.pm/ecto_sql/3.13.3/Ecto.Adapters.Postgres.html

## Toolchain

30. Node.js release schedule/status  
    https://nodejs.org/en/about/previous-releases

31. pnpm  
    https://pnpm.io/

32. Turborepo  
    https://turborepo.com/

33. Zod  
    https://zod.dev/

34. Zustand  
    https://zustand.docs.pmnd.rs/

35. XState / Stately  
    https://stately.ai/docs

## Persistence / world generation

36. Node 24 SQLite documentation  
    https://nodejs.org/download/release/latest-v24.x/docs/api/sqlite.html

37. Kysely  
    https://www.kysely.dev/

38. Kysely GitHub  
    https://github.com/kysely-org/kysely

39. Dexie TypeScript documentation  
    https://dexie.org/docs/Typescript

40. simplex-noise.js  
    https://github.com/jwagner/simplex-noise.js/

## Desktop packaging

41. Tauri  
    https://tauri.app/start/

42. Tauri frontend configuration  
    https://v2.tauri.app/start/frontend/

43. Tauri + Vite  
    https://tauri.app/start/frontend/vite/

## Testing

44. Vitest  
    https://vitest.dev/guide/

45. Playwright  
    https://playwright.dev/docs/intro

---

## Short version

If I were creating the repo today, I would scaffold around:

```text
pnpm + Turborepo
TypeScript strict mode
React + Phaser 4 + Vite
Zod
Zustand
Vitest + Playwright

Node 24 LTS + Colyseus for the first server prototype
SQLite local worlds
PostgreSQL official persistence
Tauri 2 desktop wrapper
```

…and I would preserve a clean enough server protocol that **Phoenix Channels remains a genuine option before the official ladder becomes permanent infrastructure**.
