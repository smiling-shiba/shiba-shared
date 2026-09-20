# Smiling Shiba — Engineering Foundations

You are an expert software engineer joining a brand-new repository for **Smiling Shiba**.

Your job is **not** to build the whole game, invent features, or over-engineer future systems. Your job is to establish the foundations so the desktop game can be built now without creating architectural traps for a possible mobile client years later.

Assume the basic toolchain and dependencies can be installed without hand-holding.

## Product direction

Desktop ships first.

The desktop game is expected to support:

- React for menus, HUD, deck building, collection, settings, and other DOM-heavy UI.
- Phaser for the board, battles, animation, effects, and other game-canvas work.
- Tauri as the desktop application shell.
- A local authoritative Node/Colyseus server for local/custom games.
- Local persistence such as SQLite, with human-readable configuration where useful.
- Mods and self-hosted/custom play on desktop.
- An eventual official online service, likely Phoenix/Elixir, for accounts, matchmaking, seasons, entitlements, and authoritative ladder play.

Mobile may be years away.

When mobile eventually exists, assume:

- It is **online-only**.
- It connects to the official authoritative service.
- It does **not** run the desktop Node/Colyseus sidecar.
- It does **not** need the full desktop world-exploration experience.
- It may focus on quick matches, ladder, campaign/challenges, deck building, collections, and account progression.
- It should reuse the same core battle/card/persona rules and content definitions wherever practical.

The architecture must therefore make the game **mobile-ready from day one without making mobile a current implementation target**.

---

## Core architectural principle

Absolutely. I’d actually call it **mobile-ready from day one**, even if desktop ships first.

The trick is: don’t build the desktop game in a way that assumes a mouse, giant screen, local server, filesystem, or mods are always present.

A good shape would be:

```text
                    SHARED CORE
        ┌─────────────────────────────┐
        │ rules / cards / personas    │
        │ battle state machine        │
        │ protocol / commands         │
        │ save/account models         │
        │ content definitions         │
        └──────────────┬──────────────┘
                       │
             ┌─────────┴─────────┐
             │                   │
       DESKTOP CLIENT        MOBILE CLIENT
       React + Phaser        React + Phaser
       Tauri                Tauri/native shell
       mouse/gamepad        touch
       local + online       online only
             │                   │
             └─────────┬─────────┘
                       │
                transport layer
              WebSocket / local
                       │
              authoritative server
```

The things I’d bake in **now**:

- **Battle rules are headless.** They should not know or care whether the UI is desktop, mobile, or a toaster.
- **UI layouts are responsive.** No critical information that only works at 1920×1080.
- **Touch-sized controls from the beginning.** Even desktop buttons can be comfortably tappable.
- **No hover-required interactions.** Hover can decorate; never make it necessary.
- **Input is abstracted.** `select`, `confirm`, `cancel`, `nextCard` — not hardcoded `mousemove`, `keydown`, etc.
- **Server commands are platform-neutral.** `PLAY_CARD`, `ATTACK_LAND`, `PASS`, not UI-specific events.
- **Content is declarative.** Cards/personas/boons/curses are data, so desktop and mobile use the exact same definitions.
- **Platform capabilities are optional.** Mods, local servers, filesystem access, Bluetooth, etc. live behind capability interfaces.
- **Campaign/world exploration is a feature module**, not something the battle system depends on.

That last one matters a lot.

```text
CORE GAME
├── Battles
├── Army / deck building
├── Personas
├── Account progression
└── Campaign rules

OPTIONAL EXPERIENCES
├── Desktop world exploration
├── Local/custom servers
├── Mods
├── Official ladder
└── Mobile quick play
```

Then years from now, mobile isn’t:

> “Oh God, how do we port this?”

It’s:

> “Cool, use the existing battle/deck/account core and just build the mobile navigation shell.”

And frankly, the design decisions you’ve been making already help enormously: **four lands, two creatures visible in battle, no HP bookkeeping, small hands, short creature rules**. That is *extremely* mobile-friendly without feeling like we designed a mobile game first. 😈

---

# What to establish now

## 1. Enforce dependency direction

The shared game core must be the lowest-level application code.

It may contain:

- domain types
- commands
- battle state and transitions
- card/persona/creature definitions
- validation
- deterministic rule resolution where possible
- content schemas
- protocol/message schemas

It must **not** import:

- React
- Phaser
- Tauri
- Colyseus
- browser APIs
- filesystem APIs
- SQLite
- platform-specific code

UI, networking, storage, and platform integrations depend on the core. The core does not depend on them.

## 2. Make the authoritative game API command-based

Clients should express intent, not mutate game state.

Examples:

```text
DRAW_ARMY_CARD
DEPLOY_CREATURE
ATTACK_LAND
PLAY_SPELL
PASS
CONCEDE
```

The authoritative game host validates the command, applies the rules, and emits the resulting state/events.

The same command model should be usable by:

- the local Colyseus server
- the eventual official ladder server
- tests
- bots/simulations
- a future mobile client

Do not make React components or Phaser scenes the source of game truth.

## 3. Separate transport from rules

Define a small client-facing transport boundary early.

Conceptually:

```ts
interface GameConnection {
  send(command: GameCommand): void;
  subscribe(listener: (update: GameUpdate) => void): () => void;
  close(): void;
}
```

The exact API can evolve.

The important rule is that gameplay UI should not care whether it is connected through:

- localhost Colyseus
- remote official WebSocket service
- some future nearby/Bluetooth adapter
- a test harness

## 4. Treat platform features as capabilities

Do not scatter checks such as `if desktop` throughout game logic.

Represent optional platform features behind boundaries such as:

```text
StorageCapability
ModCapability
LocalHostCapability
FileSystemCapability
NearbyPlayCapability
AccountCapability
```

Desktop can provide capabilities that mobile never provides.

The game should degrade by capability, not by forks of the core rules.

## 5. Keep content declarative

Cards, creatures, personas, boons, curses, and similar content should primarily be data interpreted by the shared rules engine.

A creature should be able to say, conceptually:

```text
"When battle begins, draw one extra card."
```

without requiring UI-specific code.

This is important for:

- desktop mods
- server authority
- mobile reuse
- testing
- future content delivery

Do not build arbitrary executable mod support into the shared core.

## 6. Keep game state small and explicit

The current battle direction intentionally avoids unnecessary bookkeeping.

Strategic state centers on land ownership and deployed creatures.

Combat centers on:

- attacking land
- defending land
- attacker creature rule
- defender creature rule
- Spellbook hands/decks/discards
- active temporary battle effects
- turn/action state

There is currently **no creature HP system** and no requirement for persistent combat-stat counters.

Do not introduce one unless game design explicitly asks for it.

## 7. Build responsive interaction primitives now

The desktop client should still be pleasant with:

- touch-sized targets
- narrow layouts
- keyboard
- mouse
- controller

Do not rely on hover for required information or actions.

Create semantic input actions such as:

```text
select
confirm
cancel
next
previous
openMenu
```

Then map keyboard, controller, pointer, and future touch controls to those actions.

## 8. Keep world exploration outside the battle core

The desktop procedural/world-exploration layer may become substantial.

It must remain a consumer of the game core, not a prerequisite for using it.

A future mobile client should be able to launch:

```text
Quick Match → Battle
Campaign Encounter → Battle
Ranked Match → Battle
```

without loading or emulating the desktop exploration world.

---

# Suggested repository boundaries

Use whatever concrete workspace layout best fits the project, but preserve these conceptual boundaries:

```text
/apps
  /desktop-client
  /local-server

/packages
  /game-core
  /game-content
  /protocol
  /client-ui
  /game-renderer

/platform
  /tauri

/services
  /official-server     # future; do not build prematurely
```

Exact names are not sacred.

The dependency boundaries are.

---

# First foundation milestone

Before building substantial game content, make one thin vertical slice prove the architecture:

1. Start the desktop client.
2. Connect it to the local authoritative server.
3. Create a tiny game state with four lands per side.
4. Issue an `ATTACK_LAND` command from the client.
5. Enter a minimal battle containing two creatures with one boon/curse each.
6. Play a simple Spellbook card through a command.
7. Let the authoritative rules resolve the battle.
8. Send the resulting state back to the client.
9. Render the result in React/Phaser.
10. Prove the headless core can run the same battle in a unit test without React, Phaser, Tauri, or Colyseus.

If that works cleanly, the foundation is doing its job.

---

# Avoid for now

Do **not** spend foundation time on:

- production Phoenix infrastructure
- mobile builds
- Bluetooth
- commerce
- season systems
- elaborate mod tooling
- procedural-world complexity
- generalized plugin frameworks
- speculative abstractions with no current caller

Leave clean seams for them.

Do not build them.

---

# Definition of success

A future engineer or AI agent should be able to open this repository and immediately understand:

> The game rules live in a headless shared core. Clients issue platform-neutral commands. An authoritative host resolves them. React/Phaser presents the game. Desktop-specific capabilities are optional adapters. World exploration is an experience layered around the core game, not part of the core game itself. Mobile can therefore arrive later as a thinner online-only client without requiring a rewrite.

That is the foundation to protect.
