# Decision Log

Status: Current as of 2026-09-21. Lightweight ADR log. When a decision becomes expensive to reverse, promote it to its own `decisions/NNNN-title.md`.

Status values: **Decided**, **Leaning**, **Open**, **Rejected**.

| ID | Decision | Status | Why / notes |
|---|---|---|---|
| D-01 | Server is authoritative in every mode; clients send intents | Decided | Local and ladder differ only in *who owns* the server. |
| D-02 | Open-source game, commercial service; no account needed to play | Decided | Software vs. service. Official store, ladder, accounts and premium content stay private. |
| D-03 | Monetization is cosmetic and progression only, never power | Decided | Avoids pay-to-win in multiplayer. |
| D-04 | Language/stack: TypeScript, Vite + React + Phaser 4, Tauri 2, Node 24 + Colyseus sidecar for local | Decided | Ruby was dropped for contributor pool and tooling. |
| D-05 | Client: Vite, not Next.js | Decided | Dynamic Next in Tauri needs a custom Node server (tech debt). |
| D-06 | Lint with Oxlint | Decided | Vite react-ts template default. |
| D-07 | Official match servers run the shared core in Node; Phoenix does accounts/matchmaking/etc. | Superseded | Replaced by D-30, then D-42. |
| D-08 | The SDK (`shiba-sdk`) is generic; a game's rules ship as a versioned policy plus templates | Decided | Fixes ship without client releases. |
| D-09 | A match pins the runtime version and the pack version for its lifetime | Decided | Reproducibility; no mid-match rule changes. |
| D-10 | Published packs are immutable; a change creates a new version | Decided | Rollback = reactivate an older version. |
| D-11 | Templates are authored in YAML, canonicalized to JSON and hashed | Decided | Human-friendly source, deterministic artifact. |
| D-12 | No code in templates; they call named functions declared by the policy | Decided | Policy functions are trusted code; templates stay data. See `shiba-sdk/docs/pack-format.md`. |
| D-13 | `sht` validates against the policy's contract, not by reading bundle source | Decided | Keeps the tooling independent of policy code, which is executable. |
| D-14 | The pack is signed as a whole (see D-38) | Decided | Provenance and integrity. Not required for local mode. |
| D-15 | Feature flags: Flipt + OpenFeature; local mode uses in-memory provider | Leaning | Used for dev gates and emergency toggles. |
| D-16 | Flags never hold rules; policies and templates do | Decided | Reproducibility trail. |
| D-17 | Logging: local `logs.sqlite`; official structured JSON to stdout | Leaning | Same `log(event, data)` adapter. |
| D-18 | Mobile is online-only, no local server, no executable mods | Decided | Store policy and product focus. |
| D-19 | Persona is the unit of expansion; seasons ship one or two times a year | Decided | Ideas go to `ideas.md`, not releases. |
| D-20 | Core game: four lands, chain attacks, no HP, creatures with one boon/curse, Army deck plus Spellbook | Decided | See `design/game-design.md` section 3a. |
| D-21 | `shiba-tools` is CLI-first (`sht`): validator library plus `validate`, `schemas`, `build-policy`, `pack`, `sign` and `verify` commands. No graphical tool | Decided | See `shiba-tools/docs/v1-scope.md`. |
| D-22 | Rule behavior is tested with the policy itself, not in the tooling | Decided | `sht` only validates names and arguments. |
| D-26 | No monorepo: separate repos with their own lockfiles and CI; `shiba-sdk` is consumed as a versioned package | Decided | See `architecture/repos.md`. |
| D-27 | npm (not pnpm) as the package manager in `shiba-tools`; use it as the default elsewhere | Decided | pnpm's workspace benefits do not apply without a monorepo. |
| D-28 | `shiba-app` holds the client, Tauri shell and local Colyseus server in one repo (two build targets, not a monorepo of packages) | Decided | The local game's client and server share one repo. |
| D-29 | `shiba-mps` is the Elixir/Phoenix multiplayer server, a separate private repo | Decided | Accounts, matchmaking, seasons, entitlements, registry. |
| D-30 | The built policy bundle (one JavaScript file) is shared by `shiba-app` and `shiba-mps`. `shiba-mps` loads the policy bundle (source maps optional) into the BEAM through an embedded JS runtime and is the source of truth for damage and outcomes whenever multiplayer is involved. The app uses the same bundle for previews and local play. | Superseded | Replaced by D-42: the ladder server no longer loads the shared policy bundle. Originally: one rules implementation, no Node fleet. |
| D-31 | Packs made in the app are signed and versioned. Custom games never send their pack to `shiba-mps`; the host's local Colyseus server is authoritative for them. Only official multiplayer runs on `shiba-mps`. Shareable server world files may come later, not now. | Decided | Custom games stay independent of the official service. |
| D-32 | The canonical policy artifact is a plain, minified, versioned, hashed JavaScript file, not QuickJS bytecode and not obfuscated. Each host may compile or cache it however it likes. | Decided | QuickBEAM warns serialized bytecode is locked to its exact QuickJS ABI. Obfuscation is not security. |
| D-33 | The runtime boundary is a tiny deterministic API (`applyCommand`, `getLegalActions`, `projectState`, `hashState`, ...) with injected RNG and no clock/network. See `shiba-sdk/docs/engine-api.md`. | Proposal | Not yet reviewed. Applies to the app's runner, which lives in `shiba-app` (D-42); `shiba-mps` is not bound by it. |
| D-34 | Current goal is the toolset, SDK and shell only, not the game. The **policy** is the rules as code (signed bundle), **templates** are YAML object definitions, **assets** are art/audio. `shiba-app` ships with empty policy, template and asset locations; a game's data lives in its own separate repo later. | Decided | The ladder server comes after, when building the game. |
| D-35 | Pack layout: `policies/` (versioned, one active), `templates/` (YAML), `assets/`. Templates may call policy functions by name with arguments and never contain code; `sht` validates function names, arguments and kind fields against the policy's generated `contract.json`. | Decided | Design doc: `shiba-sdk/docs/pack-format.md` (Proposal). |
| D-36 | Policies and packs use generated calver `YYYY.MM.DD.N`; the SDK (`shiba-sdk`) stays semver. Unknown template fields are rejected. One active policy per pack, found by name in `policies/`. Assets referenced by relative path. One generated `contract.json` per policy. Hot reload wanted later. | Decided | Detail in `shiba-sdk/docs/pack-format.md` section 10. Trusted-key folder and TypeBox for schemas are Leaning. |
| D-37 | `sht build-policy` bundles a policy with esbuild into one minified, platform-neutral bundle (a script since D-39, an ES module before), and reads its contract by loading the bundle in a separate Node process (`policy.contract(...)`). `shiba-tools` does not depend on `shiba-sdk`. | Decided | esbuild approved 2026-09-20. Host imports are rejected; a heuristic scan warns about non-deterministic APIs. See `shiba-tools/docs/build-policy.md`. |
| D-38 | A pack is locked and signed as a whole: `pack.lock.json` lists every file with its hash (YAML and JSON in canonical form, so comments and formatting do not matter), and `pack.sig.json` holds an ed25519 signature over the canonical lock, with the public key included. Trust is a separate step: `sht verify --trust <keys>`. Unsigned packs pass with a warning unless `--require-signature`. | Decided | Built in `shiba-tools` with Node's crypto, no new packages. Supersedes the policy-only signature idea in D-14. Key rotation and revocation are not built. See `shiba-tools/docs/signing.md`. |
| D-39 | The policy bundle is a plain script, not an ES module. Running it sets one global, `shibaPolicy`, and the policy is `shibaPolicy.default`. | Decided | The `MP-0001` spike found QuickBEAM can run an ES module but cannot return its `export default`. A script works in any host (V8 and QuickJS). Answers O-13. Built into `sht build-policy` on branch `feat/script-style-bundle`. |
| D-40 | `sht build-policy` rejects locale-dependent APIs in a policy (`localeCompare`, `toLocale...()`, `Intl`; error SH609). Last-digit floating-point differences between engines are accepted. | Decided | The server's engine has no locale support and results differ by host. A match runs on exactly one server (D-31), so tiny float differences between servers never matter. Player-facing text and translations belong in the app, not in the rules. Built on `shiba-tools` branch `feat/ban-locale-apis`. |
| D-41 | The policy runtime in `shiba-mps` holds only standard ECMAScript. Everything else is removed before the policy loads, and `Intl`, `localeCompare` and `toLocale...()` get quiet, plain stand-ins so libraries that touch them still load and give the same answer everywhere. | Decided | QuickBEAM's bare mode still exposes timers, a host bridge, DOM classes and crypto. Backs D-40 for library code the build scan cannot see. Halved memory per runtime. See `shiba-mps/docs/BACKLOG.md` (`MP-0001`, `MP-0012`). Applies only if a server or tool ever loads a JS policy (D-42). |
| D-42 | The public side is the JavaScript policy, the SDK, `sht`, the app and the local server: a game's rules as a policy, plus a default policy for players. The ladder is private: `shiba-mps` runs its own rules, written in Elixir when the server is developed further, and they may differ from the public policy. The two are not kept as mirrors, and the server does not load a JavaScript policy. | Decided | Supersedes D-30. Closes O-12 (no JS host needed in the server) and O-14 (no shared rules code; `shiba-core` stays unused; the app owns its runner). Ladder rules will change often, and a player who enters ladder mode enters a different world (D-02, D-31). The QuickBEAM spike stays as findings (`shiba-mps/docs/BACKLOG.md`, `MP-0001`): it worked, and D-39 and D-41 apply only if a JS policy is ever loaded on the server. Product risk to remember: players practising on the default policy may notice the ladder plays differently. |
| D-43 | License split, first pass: code (`shiba-sdk`, `shiba-tools`, `shiba-app`) is MIT. Base cards, base art and premium art stay all rights reserved (no license granted) for now, and can be opened up later. | Decided | Loosening later is easy; tightening after release is not, so start closed on content and art. Premium art was always going to be closed (D-03). |
| D-44 | `shiba-sdk` is distributed as `@smiling-shiba/sdk` on the public npm registry, published from GitHub Actions using npm's OIDC trusted publishing (no stored `NPM_TOKEN`). Ordinary semver, per D-36. | Decided | Simplest option now that the repo is public and MIT (D-43); a git-tag or tarball dependency would need consumers to build from source, or checked-in build output. Needs an npm Organization named `smiling-shiba` (owner: `kevin_asbury`) to reserve the scope; not yet created. Answers O-10 and `SH-0002`. |
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

| O-08 | Guest ladder play without an account | Later product decision. |
| O-09 | Logging library and event-tracking vendor | Both go behind small swappable interfaces (`SH-0014`). Secrets get the same treatment (`SH-0013`). |
| O-11 | Does the Shandalar-style overworld survive next to the four-lands game? | May become a campaign layer. |
