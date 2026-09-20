# Smiling Shiba: Ideas & Decisions, by Theme

Status: Draft. Distilled from a brainstorming chat; decided items are tracked in [../decisions/log.md](../decisions/log.md).

Distilled from the shared ChatGPT conversation "Branch: Ruby NES emulators" (240 turns). The full raw transcript is in `../research/shared-chat.md`. Turn numbers are given as (T#) so you can jump to the source.

**Status key:** DECIDED = you settled on it. LEANING = a direction you liked but haven't locked. OPEN = raised, unresolved. IDEA = brainstorm worth keeping.

---

## 1. How the project started (T1-T14)

- Began as "is there a Ruby NES emulator?" (rnes, Optcarrot), then "can Ruby make games?" (Gosu, Ruby2D, DragonRuby), then a 90s-style 4X, then a card game.
- Settled on a **roguelite deckbuilder with an overworld** (Slay the Spire + Diablo loot logic + Shandalar).
- Early framing: run vs. account progression. A run is disposable, the profile persists (unlocks, cards, cosmetics).
- Endless scaling comes from recombination: enemy templates + level scaling + modifiers, and card "affixes" (upgrade/mutation), not handcrafted content.
- **Language pivot (T29-T32):** Ruby -> TypeScript/browser-first, because there are more contributors and more tooling. Ships to Steam through Tauri or Electron.

## 2. Product & business model

- **DECIDED, open-source game, commercial service (T27-T30, T67-T68).** "The game is free software. The official service is a commercial product." The distinction is software vs. service, not source vs. binary.
  - Public repo: client, card engine, local server, mods, protocol, base content.
  - Private: store, entitlements, accounts, ladder, promotions, platform adapters.
- **DECIDED, no account needed to play.** Clone it, play, hack it (Fireball = 9,999,999 is fine locally). An account is only for the official ecosystem: ladder, rewards, cosmetics, cross-device collection.
- **DECIDED, monetization stays cosmetic/progression-only.** XP boosters, card backs, board skins, avatars. Avoid pay-to-win power in multiplayer (T14).
- Commerce backend: Shopify is a poor fit. Xsolla, PlayFab and Steam inventory are better matches (T14). Server grants items, never the client.
- **Content tiers (T68):** base content (free, in the repo) -> earned official (seasons/ladder, stored as entitlements) -> premium official (purchased cosmetics). Older seasonal content can be promoted into base later.
- **Licensing wrinkle to decide early (T68):** code, base card definitions, base art and premium art can each carry a different license.
- **Cadence (T51-T54):** a persona is the unit of expansion. One or two themed seasons a year (persona + biome + campaign + ladder + progression track + achievements). Seasons are ephemeral, entitlements are permanent. 3 AM ideas go into `ideas.md`, not into a release.
- Ladder content ships as **signed content packs** (art, audio, biomes, text) downloaded on sign-in. Rules stay server-side (T70). Packs aren't DRM, just integrity and caching.

## 3. Game design

### 3a. The core loop, latest version (T145-T208)
This is the most refined design in the conversation and it supersedes the earlier "creature cards + hand" ideas.

- **Board:** each player's half has **four lands**. A creature placed on a land occupies it and converts it to your control and biome.
- **Two decks:**
  - **Army deck** (creatures): draw 1 per turn on the strategic layer, deploy to lands. If all four lands are full and you recruit, you discard one.
  - **Spellbook** (spells and abilities): when a land is attacked, the view zooms into the battle and you draw 5-7 spellbook cards.
- **No HP anywhere. The board is the health bar.** You win by taking the opponent's lands (T195-T196).
- **Chain rule (T146, T194):** lands form a supply chain (1 -> 2 -> 3 -> 4). You attack in sequence, and you can only attack a land adjacent to territory you hold. This stops blitzing.
  - Cutting the chain collapses everything held behind the cut, and those lands revert to their previous controller. So comebacks are dramatic (retake 3 and you get 1 and 2 back).
  - Keep a per-land control-history stack (matters with 3+ players).
- **Zero lands is not instant death (T145-T146):** you can't deploy troops but can still play spells and attack. The opponent must hold all four of your homeland lands for **X turns** (start playtesting at 3) or you can re-occupy one. This is the siege clock.
- **A creature dies -> the land stays yours,** the creature goes to the graveyard, and a replacement is drawn next turn (T192). Attacking never leaves a land undefended. OPEN: "not sure I like leaving a land vulnerable" was your caution.
- **Creatures have no numeric stats (T200-T206).** Instead each carries one short **boon** (helps you) or **curse** (hinders the opponent), for example Knight, Hexer, Mercenary, Berserker, Gravedigger. Combat starts on equal footing, and Spellbook play decides the battle. State per battle is tiny: two creature rules, two spellbook zones, whose turn, which land, temporary effects.
- Spellbook cards can reach back into the strategic layer: summon from the graveyard, pull a creature from the Army deck, retreat, sacrifice.
- **LEANING:** whether a win lets you keep campaigning within one strategic turn.
- **IDEA, multiple win conditions (T147-T156):** public route = conquer the homeland. Plus a **secret persona objective** (e.g. Necromancer: exactly 13 cards in the graveyard at end of turn). Leave the secrets partly undocumented so the community reverse-engineers them via clues (13 candles, 13 bones, card ID ending in 013).

### 3b. Domains, biomes & the board as a second battlefield (T137-T144)
- Personas project a **Domain** onto the table (Fire, Darkness, Nature, Water, Storm, Ice...). The board visibly changes as domains push against each other.
- Same-persona mirror matches get "saturation" modes (Inferno, Eclipse, Overgrowth, Feeding Frenzy).
- Compatible domains create **synergy zones** that spawn a hybrid biome (Fire+Nature = Wildfire, Water+Darkness = Black Tide, Ice+Fire = Steamfield).
- **Declared battle zones:** the attacker picks a zone whose terrain buffs them (e.g. Windscar: flyers +2, ground -1). Terrain applies **symmetrically** unless a card says otherwise. Cards manipulate it (Home Field Advantage, Scorched Earth, Tailwind, Ambush, Claim the High Ground).

### 3c. Personas (T35-T52, T157-T180)
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

### 3d. Necromancer character notes (T157-T180)
- Visual language: osteomancy. Knucklebones, runes, bone charms, a casting cloth, 13 everywhere.
- Personality: **not** an evil death-lord. A 1970s Hobbit-era eccentric (Rackham-style grotesque), a mortality-anxious researcher who expands because "the sample size is inadequate." Tagline: *"A necromancer who buries you... in research."* Minions are research staff. His weapon is scope creep.
- Look: modern silver-fox dad, great hair, Hawaiian-style shirts with skulls and coconut-shell drinks. **Not coded gay**, more relaxed and confident masculinity. You want a broader take on masculinity and general LGBTQIA+ texture in the world's tone, not a checkbox.
- **Guardrail you set:** no AI swarms, fantasy stays fantasy (T165).
- Cousin of Withers (BG3).

### 3e. Fusion & fission (T109-T112, T135-T136)
- Digital-only advantage: combine two cards into a new one with an animation (recipes for designed pairs plus inheritance of traits for generic ones, hybrid probably best). Fusion implies **fission** (splitting later, with clean/damaged/unstable/cursed outcomes).
- Inspired by Ball x Pit's Fission / Fusion / Evolution, applied as **battle-scoped mutations** that vanish after the fight. Example: Shiba + Laser Array -> lasers in four cardinal directions that singe cards in the path. Also Hot Dog (Shiba + Fireball), Undead Shiba.

### 3f. Framing, tone, mascot (T123-T134)
- Name: **Smiling Shiba** (replaced "cardworld"). Your Shiba **Handler** sends you beyond the settled territories to find lost cards and personas. The dog is the smug tutorial nag (Navi/Clippy energy), and you're the "trainable human."
- Shiba reaction overlays (smug side-eye, judgmental counterspell, concerned Shiba on hacked 9999999 damage, playing-chess Shiba when the Fisherman steals your win-condition).

## 4. World, servers & saves (T13-T26)

- **DECIDED, each run/world is its own server/session.** Public, friends or private. Local/custom "servers" are named saves (New / Load / Duplicate / Delete / Export / Import).
- Worlds are **seed + mutation log**, not shipped map blobs (fast joins, cheap reconnects, easy debugging). The server is authoritative. Clients send intents ("I want to do this"), never claims.
- **Settlements:** one tile per player. Upgrade tiers (Camp -> Outpost -> Fort -> Town -> Citadel) with resource, healing, crafting, quest and fast-travel benefits, and terrain bonuses. Moving one is a validated function and a strategic decision. Structures die with the run, while blueprints and cosmetics can persist.
- **Share codes** encode world settings + seed (compressed, base64url). Signing is only needed for locked official modes, and plain JWT is unnecessary.
- **Save format (T115-T118):** `server.yml` (human config) + `world.sqlite` (durable state) + `mods/`. Easy-to-edit saves are a feature in local mode, so no encryption theater.
- **Pause behavior (T103-T106):** solo pauses on menu. Co-op modes: no-pause default, all-players-paused, or host-controlled. Protects against the rage-mash menu problem.

## 5. Modes: local vs. ladder (T21-T24, T63-T68)

| | Local / Custom | Official Ladder |
|---|---|---|
| Authority | Your Node/Colyseus server | Official server |
| Account | None | Required |
| Mods | Yes, freely | None |
| Cards/decks | Yours | Server-owned, versioned, legality-checked |
| Cheating | Your problem | Server ignores client claims |

- Ladder rules: server owns card definitions, collections, decklists, legality, RNG, rewards, rankings. Client only sends intents. Hash the build with SHA-256, not MD5, and never trust the client to attest itself clean.
- **DECIDED:** server is authoritative in both modes. The earlier "hash reconciliation" talk was only a CI parity idea, never a runtime one (T65-T66).
- Mobile (T213-T218): **online-only, no local server, no executable mods**, quick match + ranked + campaign, probably no overworld. Mobile happens after desktop and ladder are proven and a 24/7 backend exists. Design "mobile-ready" from day one.
- Mods: desktop gets full executable mods. Mobile gets **data-pack mods only** (App Store and Play policy on downloaded code).

## 6. Tech stack & architecture

**Current shape (T97, T212):**
- **Client:** Vite + React + TypeScript + Phaser 4. React for menus, HUD, deck builder, collection, shop, account. Phaser for the board, battle and effects.
- **Desktop shell:** Tauri 2, launching a **Node 24 + Colyseus sidecar** as the local server. Solo = "multiplayer with one person connected."
- **Persistence:** SQLite (local), YAML config.
- **Official ladder:** Phoenix/Elixir originally (Channels, Presence, supervised world processes, Postgres).
- **Linting:** Oxlint (T122).

**Decisions and rejections along the way:**
- Rejected: forking React or Phaser. Use an adapter pattern (T99-T102) or just let input be split (T103-T104, gameplay input to Phaser, menu input to React, with pause on menu in solo).
- Rejected: Next.js for the game client. Dynamic Next in Tauri is possible but a single custom Node server is tech debt. Vite + Colyseus sidecar is "the way" (T83-T98).
- PhaserJSX is a promising canvas-native UI option but very new and GPL-3.0-only, so **prototype, don't architect around it** (T78-T82).
- Input: abstract everything into game actions (`CONFIRM`, `CANCEL`, `END_TURN`...). Track last-used device and swap glyphs. Nothing may depend on hover. Tap-tap always works (dragging optional). Existing packages worth studying: phaser3-merged-input (Phaser 3) and input-map (WIP) (T72-T74).
- **Card hand UI (T113-T114):** fan/carousel of cards with the middle one magnified and glowing. Works for touch swipe and controller left/right.
- **Bluetooth (T209-T210):** use BLE via a Tauri native plugin, not Web Bluetooth (no iOS Safari support). Put the game rules behind a `GameTransport` abstraction (Colyseus | Bluetooth). Local Wi-Fi nearby play is easier and should come first.

**The biggest late change (T225-T232):** a **shared headless rules library**, `@smiling-shiba/game-core`, used by the desktop client (previews), mobile client, local server, official match servers, bots and tests. That revises the earlier Phoenix-as-authority idea. Recommended shape (Option A): **Phoenix handles accounts/matchmaking/seasons/entitlements/social, and Node game instances run game-core** for authoritative matches, so rules exist once. WASM is not needed on day one. Design game-core as pure functions with explicit state in/out, injected RNG and no platform imports so it could move to WASM later. (You clarified that TS/JS -> WASM was the thought, not Rust. The AI noted plain TS doesn't compile straight to WASM, AssemblyScript or similar would be needed.)

## 7. Rules-as-data & the Rules Lab (T229-T238)

- **Engine code is shared and boring. Rules/balance are live, versioned data.** No rules API call per move.
- Card effects use a **primitive vocabulary** (draw, discard, damage, defend, summon, resurrect, steal, claim_land, cancel, choose, inspect, move, copy, modify, compare, return_to_deck...). Weird cards can call **named handlers** in the engine.
- **A match pins engine version + ruleset version** for its whole life, so rules never change mid-match. New matches get the hotfix (e.g. Fireball 9999 -> 5).
- **Rules registry** (official backend DB): version, min engine version, status (draft / active / deprecated / disabled / emergency_blocked), manifest URL, checksum, notes. Payloads live in object storage or a CDN. Published rulesets are **immutable**.
- **Emergency toggles** are a separate small layer (disable a card, persona or queue instantly).
- **Rules Lab** (your own editing tool): draft -> schema validation -> simulate -> diff vs. active -> publish -> set ACTIVE pointer. Start as a local React tool in the repo, later an authenticated admin app.
- **Rules DSL (you proposed, AWS-IAM style):** authored in **YAML**, validated, compiled to canonical **JSON IR**, hashed, versioned. **No arbitrary JavaScript in official rulesets**: programmable but sandboxed by design. (You already have a `shiba-rules-studio` repo and `SHIBA_RULES_STUDIO.md` in Downloads, presumably from this thread.)

## 8. Engineering ground rules (T223-T224, T239)

Your standing rules:
- All tests and the linter must pass.
- README kept hygienic (not forgotten, not a graveyard, not a dumping ground).
- Env example files maintained.
- New packages need approval unless requested.
- **Agents never commit to `main`**. You alone may be reckless.

Additional decisions the AI baked in: structured logs to stdout/stderr (no logging library chosen yet), analytics through a separate adapter, containers for backend/dev infra only (not the Tauri client), behavior-focused tests with deterministic RNG and no coverage-number target, ADRs only for expensive-to-reverse decisions.

Documents ChatGPT generated in the thread (each was a download link, not captured here): Game Concept Notes (T56), TypeScript/Node Game Stack Research (~46 KB, 45 sources) (T62), Cardworld Stack Setup Guide (T108), **FOUNDATIONS.md** (T220), **ENGINEERING_GROUND_RULES.md** (T224), generic **AGENTS.md** (T240).

## 9. Open questions

1. Attacking leaves the land vulnerable? (You leaned no; T189-T192.)
2. Does a successful attack let you keep campaigning in the same strategic turn? (T194)
3. Siege clock length X (start at 3).
4. How territory collapse resolves with 3+ players (history-stack rule).
5. React vs. canvas-native UI (PhaserJSX). Prototype a throwaway screen (menu -> deck select -> 7-card hand -> inspect -> settings) on mouse, touch, keyboard and controller.
6. Official backend split: Phoenix orchestration + Node match servers (leaning) vs. Elixir-only rules.
7. License mix for code / base cards / base art / premium art.
8. Guest ladder play (no account, nowhere to attach rewards).
9. Production logging library and event-tracking vendor.
10. Whether the overworld/exploration survives on desktop next to the new four-lands game (the conversation moved to the land-conquest core, so the Shandalar overworld may become a campaign layer).

## 9b. Flags & logging: post-thread research (Sept 2026, not from the ChatGPT thread)

- **Feature flags: Flipt + OpenFeature.** Flipt is a single container with a UI. The OpenFeature Web and Node SDKs have Flipt providers (`@openfeature/flipt-web-provider`, `@openfeature/flipt-provider`). Game code calls a thin `flags.isEnabled()` adapter and never imports OpenFeature directly. Local/no-account mode uses OpenFeature's in-memory provider fed from `server.yml`, so it never needs a Flipt server.
- **Flags vs. rulesets:** rulesets (immutable, versioned, hashed) define what cards *do*. Flags are only on/off switches for engineering gates and emergency rule toggles. Flags must never become the rules store. Rule-changing flags are evaluated server-side only. Consider prefixing `eng.` and `rules.`.
- **OPEN:** should a match snapshot flag values at start (stable) or re-read them (instant kill switch)? Leaning: snapshot, plus a separate emergency-kill flag that overrides.
- **Logging:** local mode writes to a separate `logs.sqlite` (JSON `data` column, WAL mode, retention pruning). Official ladder writes structured JSON to stdout, optionally into OpenObserve. Same `log(event, data)` adapter for both.
- **Considered and dropped (curiosity satisfied):** layering a localStorage/devtools override provider via OpenFeature's Multi-Provider. Judged more machinery than needed. Also skipped: PostHog (heavy to self-host), GrowthBook, Unleash (enterprise-oriented).

## 10. Fun stuff worth keeping

Achievement names, the Ozzy-Warlock-at-the-Dead-Sea bit, Chase's Laboratory behind the bookshelf, "Legend of Ruby: A Link to the Gemfile", Safari defeats the Necromancer (WONTFIX), Nancy vs. LaShirl and the AI kittens, and the Piranha card ("SURPRISE").
