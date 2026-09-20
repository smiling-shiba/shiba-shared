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
| D-08 | Rules are versioned data; engine code is shared and boring | Decided | Fixes ship without client releases. |
| D-09 | A match pins engine version and ruleset version for its lifetime | Decided | Reproducibility; no mid-match rule changes. |
| D-10 | Published rulesets are immutable; editing creates a draft | Decided | Rollback = reactivate an older version. |
| D-11 | Rulesets authored in YAML, compiled to canonical JSON, hashed | Decided | Human-friendly source, deterministic artifact. |
| D-12 | No arbitrary JS in rulesets; YAML references named handlers in the engine bundle | Decided | Handlers are trusted code; rulesets stay data. See `shiba-core/docs/handlers-and-manifest.md`. |
| D-13 | `sht` validates against the engine's manifest, not by reading bundle source | Decided | The original studio draft already said this. |
| D-14 | Ruleset (and its bundle hash) is signed for official ladder use | Leaning | Provenance and integrity. Not required for local mode. |
| D-15 | Feature flags: Flipt + OpenFeature; local mode uses in-memory provider | Leaning | Used for dev gates and emergency toggles. |
| D-16 | Flags never hold rules; rulesets do | Decided | Reproducibility trail. |
| D-17 | Logging: local `logs.sqlite`; official structured JSON to stdout | Leaning | Same `log(event, data)` adapter. |
| D-18 | Mobile is online-only, no local server, no executable mods | Decided | Store policy and product focus. |
| D-19 | Persona is the unit of expansion; seasons ship one or two times a year | Decided | Ideas go to `ideas.md`, not releases. |
| D-20 | Core game: four lands, chain attacks, no HP, creatures with one boon/curse, Army deck plus Spellbook | Decided | See `design/game-design.md` section 3a. |
| D-21 | `shiba-tools` is CLI-first (`sht`): emit contract, validator library, `validate`/`build`/`sign` commands, VS Code schema support. No studio UI | Decided | See `shiba-tools/docs/v1-scope.md`. |
| D-22 | Simulation lives in `shiba-core`'s tests, not in the tooling | Decided | `sht` only validates names and arguments. |
| D-26 | No monorepo: separate repos with their own lockfiles and CI; `shiba-core` is consumed as a versioned package | Decided | Owner preference. See `architecture/repos.md`. Supersedes earlier monorepo layouts in `research/foundations-original.md`. |
| D-27 | npm (not pnpm) as the package manager in `shiba-tools`; use it as the default elsewhere | Decided | pnpm's workspace benefits do not apply without a monorepo. |
| D-28 | `shiba-app` holds the client, Tauri shell and local Colyseus server in one repo (two build targets, not a monorepo of packages) | Decided | Owner decision 2026-09-20. Supersedes the earlier `shiba-client` / `shiba-server` split. |
| D-29 | `shiba-mps` is the Elixir/Phoenix multiplayer server, a separate private repo | Decided | Accounts, matchmaking, seasons, entitlements, registry. |
| D-30 | `shiba-core` is a JavaScript library shared by `shiba-app` and `shiba-mps`. `shiba-mps` loads the rules bundle (source maps optional) into the BEAM through an embedded JS runtime and is the source of truth for damage and outcomes whenever multiplayer is involved. The app uses the same bundle for previews and local play. | Decided | Owner decision 2026-09-20. Resolves O-12 direction. One rules implementation, no Node fleet. |
| D-31 | Rules made in the app are signed and versioned. Custom games never send rules to `shiba-mps`; the host's local Colyseus server is authoritative for them. Only official multiplayer runs on `shiba-mps`. Shareable server world files may come later, not now. | Decided | Owner decision 2026-09-20. |
| D-32 | The canonical rules artifact is a plain, minified, versioned, hashed JavaScript file, not QuickJS bytecode and not obfuscated. Each host may compile or cache it however it likes. | Decided | QuickBEAM warns serialized bytecode is locked to its exact QuickJS ABI. Obfuscation is not security. |
| D-33 | Engine boundary is a tiny deterministic API (`applyCommand`, `getLegalActions`, `projectState`, `hashState`, ...) with injected RNG and no clock/network. See `shiba-core/docs/engine-api.md`. | Proposal | From the owner's ChatGPT discussion; not yet reviewed line by line. |
| D-23 | Layered flag overrides via OpenFeature Multi-Provider | Rejected | More machinery than a card game needs. |
| D-24 | Forking React or Phaser for input | Rejected | Use adapters or split input by mode. |
| D-25 | PostHog / GrowthBook / Unleash for self-hosting | Rejected | Too heavy or enterprise-oriented. |

## Open questions

| ID | Question | Notes |
|---|---|---|
| O-01 | Does an attack leave the land vulnerable? | Leaning no (the land stays, creature dies, replacement drawn). |
| O-02 | Does one win let you keep campaigning in the same strategic turn? | |
| O-03 | Siege clock length X | Start playtests at 3. |
| O-04 | Territory collapse with 3+ players | History-stack rule undecided. |
| O-05 | React alone, or canvas-native UI (PhaserJSX)? | PhaserJSX is GPL-3.0 and very new. Prototype one throwaway screen before deciding. |
| O-06 | Flag snapshot per match versus live re-read | Leaning: snapshot plus an emergency-kill flag that overrides. |
| O-07 | Licensing split: code, base cards, base art, premium art | Decide early. |
| O-08 | Guest ladder play without an account | Later product decision. |
| O-09 | Logging library and event-tracking vendor | |
| O-10 | Naming: engine vs core vs Rules Profile; Handler (mascot) vs handler (function) | See [../GLOSSARY.md](../GLOSSARY.md). |
| O-12 | Confirm QuickBEAM as the JS host in `shiba-mps` (leading candidate) versus a Node worker pool or WASM. Test engine parity, limits and latency in a spike. | Direction set by D-30. See `shiba-mps/docs/scope.md`. |
| O-11 | Does the Shandalar-style overworld survive next to the four-lands game? | May become a campaign layer. |
