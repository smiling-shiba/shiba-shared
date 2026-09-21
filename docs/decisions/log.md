# Decision Log

Status: Current as of 2026-09-20. Lightweight ADR log. When a decision becomes expensive to reverse, promote it to its own `decisions/NNNN-title.md`.

Status values: **Decided**, **Leaning**, **Open**, **Rejected**.

| ID | Decision | Status | Why / notes |
|---|---|---|---|
| D-01 | Server is authoritative in every mode; clients send intents | Decided | Local and ladder differ only in *who owns* the server. |
| D-02 | Open-source game, commercial service; no account needed to play | Decided | Software vs. service. Official store, ladder, accounts and premium content stay private. |
| D-03 | Monetization is cosmetic and progression only, never power | Decided | Avoids pay-to-win in multiplayer. |
| D-04 | Language/stack: TypeScript, Vite + React + Phaser 4, Tauri 2, Node 24 + Colyseus sidecar for local | Decided | Ruby was dropped for contributor pool and tooling. |
| D-05 | Client: Vite, not Next.js | Decided | Dynamic Next in Tauri needs a custom Node server (tech debt). |
| D-06 | Lint with Oxlint | Decided | Vite react-ts template default. |
| D-07 | Official match servers run the shared core in Node; Phoenix does accounts/matchmaking/etc. | Superseded | Replaced by D-30. |
| D-08 | The runtime (`shiba-core`) is shared and generic; a game's rules ship as a versioned policy plus templates | Decided | Fixes ship without client releases. |
| D-09 | A match pins the runtime version and the pack version for its lifetime | Decided | Reproducibility; no mid-match rule changes. |
| D-10 | Published packs are immutable; a change creates a new version | Decided | Rollback = reactivate an older version. |
| D-11 | Templates are authored in YAML, canonicalized to JSON and hashed | Decided | Human-friendly source, deterministic artifact. |
| D-12 | No code in templates; they call named functions declared by the policy | Decided | Policy functions are trusted code; templates stay data. See `shiba-core/docs/pack-format.md`. |
| D-13 | `sht` validates against the policy's contract, not by reading bundle source | Decided | Keeps the tooling independent of policy code, which is executable. |
| D-14 | The policy (and, for the official ladder, the whole pack) is signed | Leaning | Provenance and integrity. Not required for local mode. |
| D-15 | Feature flags: Flipt + OpenFeature; local mode uses in-memory provider | Leaning | Used for dev gates and emergency toggles. |
| D-16 | Flags never hold rules; policies and templates do | Decided | Reproducibility trail. |
| D-17 | Logging: local `logs.sqlite`; official structured JSON to stdout | Leaning | Same `log(event, data)` adapter. |
| D-18 | Mobile is online-only, no local server, no executable mods | Decided | Store policy and product focus. |
| D-19 | Persona is the unit of expansion; seasons ship one or two times a year | Decided | Ideas go to `ideas.md`, not releases. |
| D-20 | Core game: four lands, chain attacks, no HP, creatures with one boon/curse, Army deck plus Spellbook | Decided | See `design/game-design.md` section 3a. |
| D-21 | `shiba-tools` is CLI-first (`sht`): validator library plus `validate`, `schemas`, `build-policy`, `pack`, `sign` and `verify` commands. No graphical tool | Decided | See `shiba-tools/docs/v1-scope.md`. |
| D-22 | Simulation lives in `shiba-core`'s tests, not in the tooling | Decided | `sht` only validates names and arguments. |
| D-26 | No monorepo: separate repos with their own lockfiles and CI; `shiba-core` is consumed as a versioned package | Decided | See `architecture/repos.md`. |
| D-27 | npm (not pnpm) as the package manager in `shiba-tools`; use it as the default elsewhere | Decided | pnpm's workspace benefits do not apply without a monorepo. |
| D-28 | `shiba-app` holds the client, Tauri shell and local Colyseus server in one repo (two build targets, not a monorepo of packages) | Decided | The local game's client and server share one repo. |
| D-29 | `shiba-mps` is the Elixir/Phoenix multiplayer server, a separate private repo | Decided | Accounts, matchmaking, seasons, entitlements, registry. |
| D-30 | `shiba-core` is a JavaScript library shared by `shiba-app` and `shiba-mps`. `shiba-mps` loads the policy bundle (source maps optional) into the BEAM through an embedded JS runtime and is the source of truth for damage and outcomes whenever multiplayer is involved. The app uses the same bundle for previews and local play. | Decided | Resolves O-12 direction. One rules implementation, no Node fleet. |
| D-31 | Packs made in the app are signed and versioned. Custom games never send their pack to `shiba-mps`; the host's local Colyseus server is authoritative for them. Only official multiplayer runs on `shiba-mps`. Shareable server world files may come later, not now. | Decided | Custom games stay independent of the official service. |
| D-32 | The canonical policy artifact is a plain, minified, versioned, hashed JavaScript file, not QuickJS bytecode and not obfuscated. Each host may compile or cache it however it likes. | Decided | QuickBEAM warns serialized bytecode is locked to its exact QuickJS ABI. Obfuscation is not security. |
| D-33 | The runtime boundary is a tiny deterministic API (`applyCommand`, `getLegalActions`, `projectState`, `hashState`, ...) with injected RNG and no clock/network. See `shiba-core/docs/engine-api.md`. | Proposal | Not yet reviewed line by line. |
| D-34 | Current goal is the toolset, SDK and shell only, not the game. The **policy** is the rules as code (signed bundle), **templates** are YAML object definitions, **assets** are art/audio. `shiba-app` ships with empty policy, template and asset locations; a game's data lives in its own separate repo later. | Decided | The ladder server comes after, when building the game. |
| D-35 | Pack layout: `policies/` (versioned, one active), `templates/` (YAML), `assets/`. Templates may call policy functions by name with arguments and never contain code; `sht` validates function names, arguments and kind fields against the policy's generated `contract.json`. | Decided | Design doc: `shiba-core/docs/pack-format.md` (Proposal). |
| D-36 | Policies and packs use generated calver `YYYY.MM.DD.N`; `shiba-core` runtime stays semver. Unknown template fields are rejected. One active policy per pack, found by name in `policies/`. Assets referenced by relative path. One generated `contract.json` per policy. Hot reload wanted later. | Decided | Detail in `shiba-core/docs/pack-format.md` section 10. Trusted-key folder and TypeBox for schemas are Leaning. |
| D-37 | `sht build-policy` bundles a policy with esbuild into one minified, platform-neutral ES module, and reads its contract by loading the bundle in a separate Node process (`policy.contract(...)`). `shiba-tools` does not depend on `shiba-core`. | Decided | esbuild approved 2026-09-20. Host imports are rejected; a heuristic scan warns about non-deterministic APIs. See `shiba-tools/docs/build-policy.md`. |
| D-23 | Layered flag overrides via OpenFeature Multi-Provider | Rejected | More machinery than a card game needs. |
| D-24 | Forking React or Phaser for input | Rejected | Use adapters or split input by mode. |
| D-25 | PostHog / GrowthBook / Unleash for self-hosting | Rejected | Too heavy or enterprise-oriented. |

## Open questions

| ID | Question | Notes |
|---|---|---|
| O-01 | Does an attack leave the land vulnerable? | Leaning no (the land stays, creature dies, replacement drawn). |
| O-02 | Does one win let a player keep campaigning in the same strategic turn? | |
| O-03 | Siege clock length X | Start playtests at 3. |
| O-04 | Territory collapse with 3+ players | History-stack rule undecided. |
| O-05 | React alone, or canvas-native UI (PhaserJSX)? | PhaserJSX is GPL-3.0 and very new. Prototype one throwaway screen before deciding. |
| O-06 | Flag snapshot per match versus live re-read | Leaning: snapshot plus an emergency-kill flag that overrides. |
| O-07 | Licensing split: code, base cards, base art, premium art | Decide early. |
| O-08 | Guest ladder play without an account | Later product decision. |
| O-09 | Logging library and event-tracking vendor | |
| O-10 | Final naming for the shared runtime/SDK (`shiba-core`) and its published package name | See [../GLOSSARY.md](../GLOSSARY.md). |
| O-12 | Confirm QuickBEAM as the JS host in `shiba-mps` (leading candidate) versus a Node worker pool or WASM. Test engine parity, limits and latency in a spike. | Direction set by D-30. See `shiba-mps/docs/scope.md`. |
| O-13 | Does the embedded runtime in `shiba-mps` load the policy bundle in its current form (ES module with a default export)? | Check in the `MP-0001` spike; the build format may need a script/IIFE variant. |
| O-11 | Does the Shandalar-style overworld survive next to the four-lands game? | May become a campaign layer. |
