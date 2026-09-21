# BACKLOG.md (shiba-shared)

Story prefix: `SH-`. This repo holds the **epic definitions** for the whole Smiling Shiba universe (`SS-NN`) plus cross-repo and documentation stories. Other repos keep their own `docs/BACKLOG.md` and tag stories with an epic ID.

Convention: [engineering/backlog-and-ids.md](engineering/backlog-and-ids.md). Keep "Now" to 3 items or fewer.

## Epics

### SS-01: Rules contract & validation
Goal: a game's rules live in a policy (code) written with the `shiba-sdk` SDK; `shiba-tools` (`sht`) validates YAML templates against the policy's contract and signs packs for official use.
Repos: `shiba-sdk` (SK-), `shiba-tools` (ST-).
Status: in progress and the current focus. The toolset is built and tested: `sht validate`, `schemas`, `build-policy`, `pack`, `keygen`, `sign` and `verify`, with `shiba-sdk` generating contracts. Remaining: the pack loader in the app, and the open question of where shared rules code and the runner live (O-14).

### SS-02: First playable vertical slice (after the toolset is stable)
Goal: prove the architecture end to end. Four lands per side, an `ATTACK_LAND` command, a two-creature battle with one boon/curse each, one Spellbook card, authoritative resolution, rendered in the client, and the same battle running headless in a core test.
Repos: `shiba-sdk` (SK-), `shiba-app` (SA-).
Status: not started.

### SS-03: Official services
Goal: accounts, matchmaking, seasons, entitlements and the rules registry.
Repos: `shiba-mps` (MP-).
Status: parked until SS-02 is proven. Decide O-12 (how MPS runs rules) first.

## Now

Three decisions are waiting on the owner; nothing else is in progress.

- [ ] `SH-0001` Decide where the shared rules code and the runner live (open question O-14). *(waiting on you)*
- [ ] `SH-0002` Decide how `shiba-sdk` is distributed to other repos: git tag, private registry or packed tarball. *(waiting on you)*
- [ ] `SH-0009` Talk through packs as single files and mod layering; decide what to build. Notes: [design/packs-and-modding.md](design/packs-and-modding.md). *(waiting on you)*

## Next

- [ ] `SH-0003` Create one master `AGENTS.md` under `docs/engineering/` and copy it into each repo.

## Later / Ideas

- [ ] `SH-0005` Decide the licence split: code, base cards, base art, premium art. (O-07)
- [ ] `SH-0006` Set up a dev-only Flipt container plus OpenFeature adapter. (Needs package approval.)
- [ ] `SH-0007` Playtest the siege clock length. Start at 3 turns. (O-03)
- [ ] `SH-0008` Prototype the fan/carousel hand UI on one throwaway screen.
- [ ] `SH-0009` Parked idea: translate a policy from JavaScript to Elixir before the server starts (a Mix task that skips when the policy version is unchanged), to remove the embedded JS engine as a speed limit. Not planned yet. In multiplayer the server's rules are the truth (D-30, D-31), so a different implementation on the server would be allowed. The real costs are that JavaScript is hard to copy exactly, so a translation bug means the ladder plays a different game from the one the designer tested locally, and that a translator must be built, maintained and its output trusted or signed. Measured so far: about 6 ms per round of rules over 4,000 objects, 3 to 4 times slower than V8, one thread per match (`shiba-mps` `MP-0001`). Try first: write rules that touch less state, or swap the engine for a Node worker pool (the O-12 fallback). Revisit only if a real game's rules make QuickBEAM the bottleneck.

## Blocked

## Done (recent)

- [x] `SH-0000` Create cross-repo docs, decision log, glossary and repo map.
