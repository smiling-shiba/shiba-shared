# Architecture Overview

Status: Draft. Reflects decisions through 2026-09-20. See [../decisions/log.md](../decisions/log.md) for what is settled versus open.

## One-page picture

```text
shiba-core (SDK + deterministic runtime)
   |  used to write
   v
Pack = policy (code) + templates (YAML) + assets      lives in a game's own repo
   |
   +-- shiba-tools (`sht`): validate, pack, sign, verify
   |
   +-- shiba-app: client (Vite + React + Phaser) in a Tauri shell
   |       +-- local server (Node + Colyseus sidecar): authoritative for local games
   |
   +-- shiba-mps (Elixir/Phoenix): official multiplayer
           loads the signed policy into the BEAM; authoritative for official matches
           also: accounts, matchmaking, seasons, entitlements, pack registry
```

## Rules of the road

1. **Server is authoritative in every mode.** Clients send intents (`PLAY_SPELL`, `ATTACK_LAND`); the server validates and resolves. Local server, official server, Bluetooth host: same model.
2. **The runtime is shared and generic; a game's rules are a pack.** `shiba-core` provides the SDK and a deterministic runtime. A game's rules are a signed policy (code) plus YAML templates that call the policy's functions. Nothing in a template is code.
3. **Matches pin runtime and pack versions.** Hotfixes affect new matches only.
4. **No account to play.** Clone it, play, mod it. Accounts exist for the official ecosystem only.
5. **Open game, commercial service.** Code, base content and local server are public. Store, ladder, accounts, entitlements and premium art are private.
6. **Mobile is later, online-only.** No local server, no executable mods, probably no overworld. Design mobile-ready now (touch-sized UI, no hover-only info, abstracted input, headless rules).

## Modes

| | Local / Custom | Official Ladder |
|---|---|---|
| Authority | Local Node + Colyseus | Official match server |
| Account | None | Required |
| Mods | Yes (executable on desktop) | None |
| Cards/decks | Player's own | Server-owned, versioned, legality-checked |
| Saves | `server.yml` + `world.sqlite` | Postgres |
| Logs | `logs.sqlite` | Structured JSON to stdout |

## Client stack

Vite + React + TypeScript + Phaser 4, packaged with Tauri 2. Oxlint. React owns menus, HUD, deck builder, collection, shop. Phaser owns the board and battle. In solo play menus pause gameplay input; in co-op pausing is configurable.

## Cross-cutting

- **Input:** abstract to game actions (`CONFIRM`, `CANCEL`, `END_TURN`); track last-used device; nothing depends on hover.
- **Transport:** rules sit behind a `GameConnection`/`GameTransport` boundary so Colyseus, the official service, tests and a possible Bluetooth adapter are interchangeable.
- **Flags:** Flipt plus OpenFeature for engineering gates and emergency toggles. Local mode uses the in-memory provider. Flags never hold rules.
- **Logging:** adapter with two backends (see overview table).
- **Determinism:** injected RNG, no clocks or network in policy or runtime code.
- **Hidden information:** clients get a projected view of state, never the whole thing.

## Open architecture questions

See D-open entries in [../decisions/log.md](../decisions/log.md). The biggest: how `shiba-mps` hosts the JS runtime (O-12, decided direction D-30).
