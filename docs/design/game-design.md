# Smiling Shiba: Ideas & Decisions, by Theme

Status: Draft. Game design ideas by theme. Decided items are tracked in [../decisions/log.md](../decisions/log.md).

**Status key:** DECIDED = settled. LEANING = a direction not yet locked. OPEN = unresolved. IDEA = worth keeping.

---

## 1. How the project started

- Began as "is there a Ruby NES emulator?" (rnes, Optcarrot), then "can Ruby make games?" (Gosu, Ruby2D, DragonRuby), then a 90s-style 4X, then a card game.
- Settled on a **roguelite deckbuilder with an overworld** (Slay the Spire + Diablo loot logic + Shandalar).
- Early framing: run vs. account progression. A run is disposable, the profile persists (unlocks, cards, cosmetics).
- Endless scaling comes from recombination: enemy templates + level scaling + modifiers, and card "affixes" (upgrade/mutation), not handcrafted content.
- **Language pivot:** Ruby -> TypeScript/browser-first, because there are more contributors and more tooling. Ships to Steam through Tauri or Electron.

## 2. Product & business model

- **DECIDED, open-source game, commercial service.** "The game is free software. The official service is a commercial product." The distinction is software vs. service, not source vs. binary.
  - Public repo: client, card engine, local server, mods, protocol, base content.
  - Private: store, entitlements, accounts, ladder, promotions, platform adapters.
- **DECIDED, no account needed to play.** Clone it, play, hack it (Fireball = 9,999,999 is fine locally). An account is only for the official ecosystem: ladder, rewards, cosmetics, cross-device collection.
- **DECIDED, monetization stays cosmetic/progression-only.** XP boosters, card backs, board skins, avatars. Avoid pay-to-win power in multiplayer.
- Commerce backend: Shopify is a poor fit. Xsolla, PlayFab and Steam inventory are better matches. Server grants items, never the client.
- **Content tiers:** base content (free, in the repo) -> earned official (seasons/ladder, stored as entitlements) -> premium official (purchased cosmetics). Older seasonal content can be promoted into base later.
- **Licensing wrinkle to decide early:** code, base card definitions, base art and premium art can each carry a different license.
- **Cadence:** a persona is the unit of expansion. One or two themed seasons a year (persona + biome + campaign + ladder + progression track + achievements). Seasons are ephemeral, entitlements are permanent. 3 AM ideas go into `ideas.md`, not into a release.
- Ladder content ships as **signed content packs** (art, audio, biomes, text) downloaded on sign-in. Rules stay server-side. Packs aren't DRM, just integrity and caching.

## 3. Game design

### 3a. The core loop, latest version
This is the most refined design in the conversation and it supersedes the earlier "creature cards + hand" ideas.

- **Board:** each player's half has **four lands**. A creature placed on a land occupies it and converts it to your control and biome.
- **Two decks:**
  - **Army deck** (creatures): draw 1 per turn on the strategic layer, deploy to lands. If all four lands are full and you recruit, you discard one.
  - **Spellbook** (spells and abilities): when a land is attacked, the view zooms into the battle and you draw 5-7 spellbook cards.
- **No HP anywhere. The board is the health bar.** You win by taking the opponent's lands.
- **Chain rule:** lands form a supply chain (1 -> 2 -> 3 -> 4). You attack in sequence, and you can only attack a land adjacent to territory you hold. This stops blitzing.
  - Cutting the chain collapses everything held behind the cut, and those lands revert to their previous controller. So comebacks are dramatic (retake 3 and you get 1 and 2 back).
  - Keep a per-land control-history stack (matters with 3+ players).
- **Zero lands is not instant death:** you can't deploy troops but can still play spells and attack. The opponent must hold all four of your homeland lands for **X turns** (start playtesting at 3) or you can re-occupy one. This is the siege clock.
- **A creature dies -> the land stays yours,** the creature goes to the graveyard, and a replacement is drawn next turn. Attacking never leaves a land undefended. OPEN: whether leaving a land vulnerable is acceptable (currently leaning no).
- **Creatures have no numeric stats.** Instead each carries one short **boon** (helps you) or **curse** (hinders the opponent), for example Knight, Hexer, Mercenary, Berserker, Gravedigger. Combat starts on equal footing, and Spellbook play decides the battle. State per battle is tiny: two creature rules, two spellbook zones, whose turn, which land, temporary effects.
- Spellbook cards can reach back into the strategic layer: summon from the graveyard, pull a creature from the Army deck, retreat, sacrifice.
- **LEANING:** whether a win lets you keep campaigning within one strategic turn.
- **IDEA, multiple win conditions:** public route = conquer the homeland. Plus a **secret persona objective** (e.g. Necromancer: exactly 13 cards in the graveyard at end of turn). Leave the secrets partly undocumented so the community reverse-engineers them via clues (13 candles, 13 bones, card ID ending in 013).

### 3b. Domains, biomes & the board as a second battlefield
- Personas project a **Domain** onto the table (Fire, Darkness, Nature, Water, Storm, Ice...). The board visibly changes as domains push against each other.
- Same-persona mirror matches get "saturation" modes (Inferno, Eclipse, Overgrowth, Feeding Frenzy).
- Compatible domains create **synergy zones** that spawn a hybrid biome (Fire+Nature = Wildfire, Water+Darkness = Black Tide, Ice+Fire = Steamfield).
- **Declared battle zones:** the attacker picks a zone whose terrain buffs them (e.g. Windscar: flyers +2, ground -1). Terrain applies **symmetrically** unless a card says otherwise. Cards manipulate it (Home Field Advantage, Scorched Earth, Tailwind, Ambush, Claim the High Ground).

### 3c. Personas
A persona = a global passive rule + a card family. Unlocking one changes how you think about the whole collection (Smash Up-like).

| Persona | Identity | Notes |
|---|---|---|
| **Necromancer** | Discard pile is a second hand | Play minions from the graveyard once per turn. Secret win at 13 graveyard cards. See character notes below. |
| **Fisherman** | Deck-order manipulation | Global "Gone Fishing" (look at top card, keep or sink). Keywords: Fish, Catch, Release, Sink, Bait, Trawl. Counterplay cards (Piranha, Unfishable, Anchored). |
| **HIMBO Hunter** | Prestige thresholds | Marks high-Prestige enemies ("Hunted") for bonuses and captures. |
| **Grifter** | Steals Prestige, redirects costs | |
| *Also floated* | | Corporate Witch (gold to spell power), Chaos Goblin, Archivist, Influencer, Cult Leader, Union Organizer, Tax Accountant (idea). |

- **Cross-persona cards** are special rewards (e.g. Deadliest Catch = Fisherman + Necromancer). Never full persona combination, which would be a balance nightmare.
- **Achievements** are persona-specific, reward cosmetics not power, and are jokes: Necro Grifter, Dead Sea Scrolls ("There's nothing dead here!"), Plenty of Fish in the Sea, Conscious Uncoupling (split 10 fused cards), Company of the Dead (trigger the 13-card win), Much Damage. Very Ouch.

### 3d. Necromancer character concept

- **Visual language:** osteomancy (divination by bones). Knucklebones, runes, bone charms and a casting cloth, with the number 13 recurring everywhere.
- **Tone:** not a conquering death-lord. An eccentric, mortality-anxious researcher in the spirit of 1970s storybook fantasy, who expands because "the sample size is inadequate." His minions are research staff, and his real weapon is scope creep. Tagline: *"A necromancer who buries you... in research."*
- **Look:** a relaxed, well-groomed older man in a skull-print resort shirt. The intent is confident, comfortable masculinity and an inclusive tone across the world, without treating identity as a checkbox.
- **Guardrail:** fantasy stays fantasy; no AI or technology tropes.
- **Kin:** a dry, bookkeeping-minded undead in the mold of Withers from Baldur's Gate 3.

### 3e. Fusion & fission
- Digital-only advantage: combine two cards into a new one with an animation (recipes for designed pairs plus inheritance of traits for generic ones, hybrid probably best). Fusion implies **fission** (splitting later, with clean/damaged/unstable/cursed outcomes).
- Inspired by Ball x Pit's Fission / Fusion / Evolution, applied as **battle-scoped mutations** that vanish after the fight. Example: Shiba + Laser Array -> lasers in four cardinal directions that singe cards in the path. Also Hot Dog (Shiba + Fireball), Undead Shiba.

### 3f. Framing, tone, mascot
- Name: **Smiling Shiba** (replaced "cardworld"). Your Shiba **Handler** sends you beyond the settled territories to find lost cards and personas. The dog is the smug tutorial nag (Navi/Clippy energy), and you're the "trainable human."
- Shiba reaction overlays (smug side-eye, judgmental counterspell, concerned Shiba on hacked 9999999 damage, playing-chess Shiba when the Fisherman steals your win-condition).

## 4. World, servers & saves

- **DECIDED, each run/world is its own server/session.** Public, friends or private. Local/custom "servers" are named saves (New / Load / Duplicate / Delete / Export / Import).
- Worlds are **seed + mutation log**, not shipped map blobs (fast joins, cheap reconnects, easy debugging). The server is authoritative. Clients send intents ("I want to do this"), never claims.
- **Settlements:** one tile per player. Upgrade tiers (Camp -> Outpost -> Fort -> Town -> Citadel) with resource, healing, crafting, quest and fast-travel benefits, and terrain bonuses. Moving one is a validated function and a strategic decision. Structures die with the run, while blueprints and cosmetics can persist.
- **Share codes** encode world settings + seed (compressed, base64url). Signing is only needed for locked official modes, and plain JWT is unnecessary.
- **Save format:** `server.yml` (human config) + `world.sqlite` (durable state) + `mods/`. Easy-to-edit saves are a feature in local mode, so no encryption theater.
- **Pause behavior:** solo pauses on menu. Co-op modes: no-pause default, all-players-paused, or host-controlled. Protects against the rage-mash menu problem.

## 5. Modes: local vs. ladder

| | Local / Custom | Official Ladder |
|---|---|---|
| Authority | Your Node/Colyseus server | Official server |
| Account | None | Required |
| Mods | Yes, freely | None |
| Cards/decks | Yours | Server-owned, versioned, legality-checked |
| Cheating | Your problem | Server ignores client claims |

- Ladder rules: server owns card definitions, collections, decklists, legality, RNG, rewards, rankings. Client only sends intents. Hash the build with SHA-256, not MD5, and never trust the client to attest itself clean.
- **DECIDED:** server is authoritative in both modes. The earlier "hash reconciliation" talk was only a CI parity idea, never a runtime one.
- Mobile: **online-only, no local server, no executable mods**, quick match + ranked + campaign, probably no overworld. Mobile happens after desktop and ladder are proven and a 24/7 backend exists. Design "mobile-ready" from day one.
- Mods: desktop gets full executable mods. Mobile gets **data-pack mods only** (App Store and Play policy on downloaded code).

## 6. Tech stack & architecture

**Current shape:**
- **Client:** Vite + React + TypeScript + Phaser 4. React for menus, HUD, deck builder, collection, shop, account. Phaser for the board, battle and effects.
- **Desktop shell:** Tauri 2, launching a **Node 24 + Colyseus sidecar** as the local server. Solo = "multiplayer with one person connected."
- **Persistence:** SQLite (local), YAML config.
- **Official multiplayer:** `shiba-mps`, Elixir/Phoenix (Channels, Presence, supervised processes, Postgres).
- **Linting:** Oxlint.

**Decisions and rejections along the way:**
- Rejected: forking React or Phaser. Use an adapter pattern or just let input be split (gameplay input to Phaser, menu input to React, with pause on menu in solo).
- Rejected: Next.js for the game client. Dynamic Next in Tauri is possible but a single custom Node server is tech debt. Vite + Colyseus sidecar is "the way".
- PhaserJSX is a promising canvas-native UI option but very new and GPL-3.0-only, so **prototype, don't architect around it**.
- Input: abstract everything into game actions (`CONFIRM`, `CANCEL`, `END_TURN`...). Track last-used device and swap glyphs. Nothing may depend on hover. Tap-tap always works (dragging optional). Existing packages worth studying: phaser3-merged-input (Phaser 3) and input-map (WIP).
- **Card hand UI:** fan/carousel of cards with the middle one magnified and glowing. Works for touch swipe and controller left/right.
- **Bluetooth:** use BLE via a Tauri native plugin, not Web Bluetooth (no iOS Safari support). Put the game rules behind a `GameTransport` abstraction (Colyseus | Bluetooth). Local Wi-Fi nearby play is easier and should come first.

**The biggest late change:** a **shared headless rules library** (`shiba-core`), used by the app (previews and local play) and by `shiba-mps`, which loads the same bundle into the BEAM and is the source of truth for official multiplayer (decision D-30). Rules therefore exist once. Design it as pure functions with explicit state in and out, injected RNG and no platform imports.

## 7. Rules as data, and the Rules Lab

Terminology and format follow decisions D-34 to D-36 and `shiba-core/docs/pack-format.md`.

- **The runtime is shared and generic; a game's rules are a versioned policy plus templates.** No rules API call per move.
- Card effects are built from a **primitive vocabulary** (draw, discard, damage, defend, summon, resurrect, steal, claim_land, cancel, choose, inspect, move, copy, modify, compare, return_to_deck...). One-off cards can call named policy functions.
- **A match pins the runtime version and the pack version** for its whole life, so rules never change mid-match. New matches get the hotfix (for example Fireball 9999 -> 5).
- **Pack registry** (official backend database): version, minimum runtime version, status (draft / active / deprecated / disabled / emergency_blocked), storage location, checksum, notes. Payloads live in object storage or a CDN. Published packs are **immutable**.
- **Emergency toggles** are a separate small layer (disable a card, persona or queue instantly).
- **Rules Lab** (internal admin tool, later): draft -> validate -> diff vs. active -> publish -> set ACTIVE pointer. Start small; it becomes an authenticated admin app once the official service exists.
- **Templates** (YAML, AWS-IAM style): validated against the policy's contract, canonicalized to JSON, hashed and versioned. **No code in templates**: they only call functions the policy declares.

## 8. Engineering ground rules

Standing rules:
- All tests and the linter must pass.
- README kept hygienic (not forgotten, not a graveyard, not a dumping ground).
- Env example files maintained.
- New packages need approval unless requested.
- **Agents never commit to `main`.**

Additional decisions: structured logs to stdout/stderr (no logging library chosen yet), analytics through a separate adapter, containers for backend/dev infra only (not the Tauri client), behavior-focused tests with deterministic RNG and no coverage-number target, ADRs only for expensive-to-reverse decisions.


## 9. Open questions

1. Attacking leaves the land vulnerable? (Leaning no.)
2. Does a successful attack let you keep campaigning in the same strategic turn?
3. Siege clock length X (start at 3).
4. How territory collapse resolves with 3+ players (history-stack rule).
5. React vs. canvas-native UI (PhaserJSX). Prototype a throwaway screen (menu -> deck select -> 7-card hand -> inspect -> settings) on mouse, touch, keyboard and controller.
6. Official backend split: Phoenix orchestration + Node match servers (leaning) vs. Elixir-only rules.
7. License mix for code / base cards / base art / premium art.
8. Guest ladder play (no account, nowhere to attach rewards).
9. Production logging library and event-tracking vendor.
10. Whether the overworld/exploration survives on desktop next to the new four-lands game (the conversation moved to the land-conquest core, so the Shandalar overworld may become a campaign layer).

## 10. Flags and logging (research, Sept 2026)

- **Feature flags: Flipt + OpenFeature.** Flipt is a single container with a UI. The OpenFeature Web and Node SDKs have Flipt providers (`@openfeature/flipt-web-provider`, `@openfeature/flipt-provider`). Game code calls a thin `flags.isEnabled()` adapter and never imports OpenFeature directly. Local/no-account mode uses OpenFeature's in-memory provider fed from `server.yml`, so it never needs a Flipt server.
- **Flags vs. packs:** a pack's policy and templates (immutable, versioned, hashed) define what cards *do*. Flags are only on/off switches for engineering gates and emergency rule toggles. Flags must never become the rules store. Rule-changing flags are evaluated server-side only. Consider prefixing `eng.` and `rules.`.
- **OPEN:** should a match snapshot flag values at start (stable) or re-read them (instant kill switch)? Leaning: snapshot, plus a separate emergency-kill flag that overrides.
- **Logging:** local mode writes to a separate `logs.sqlite` (JSON `data` column, WAL mode, retention pruning). Official ladder writes structured JSON to stdout, optionally into OpenObserve. Same `log(event, data)` adapter for both.
- **Considered and dropped (curiosity satisfied):** layering a localStorage/devtools override provider via OpenFeature's Multi-Provider. Judged more machinery than needed. Also skipped: PostHog (heavy to self-host), GrowthBook, Unleash (enterprise-oriented).
