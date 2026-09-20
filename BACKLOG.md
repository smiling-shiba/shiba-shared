# BACKLOG.md (shiba-shared)

Story prefix: `SH-`. This repo holds the **epic definitions** for the whole Smiling Shiba universe (`SS-NN`) plus cross-repo and documentation stories. Other repos keep their own `BACKLOG.md` and tag stories with an epic ID.

Convention: [docs/engineering/backlog-and-ids.md](docs/engineering/backlog-and-ids.md). Keep "Now" to 3 items or fewer.

## Epics

### SS-01: Rules contract & validation
Goal: rules logic lives in `shiba-core` as named handlers; `shiba-tools` (`sht`) validates YAML rulesets against the manifest it emits, and signs for official use.
Repos: `shiba-core` (SC-), `shiba-tools` (ST-).
Status: not started.

### SS-02: First playable vertical slice
Goal: prove the architecture end to end. Four lands per side, an `ATTACK_LAND` command, a two-creature battle with one boon/curse each, one Spellbook card, authoritative resolution, rendered in the client, and the same battle running headless in a core test.
Repos: `shiba-core` (SC-), `shiba-app` (SA-).
Status: not started.

### SS-03: Official services
Goal: accounts, matchmaking, seasons, entitlements and the rules registry.
Repos: `shiba-mps` (MP-).
Status: parked until SS-02 is proven. Decide O-12 (how MPS runs rules) first.

## Now

## Next

- [ ] `SH-0001` Resolve naming: engine vs core vs Rules Profile; handler (function) vs Handler (mascot). Update the glossary. (Open question O-10)
- [ ] `SH-0002` Decide how `shiba-core` is distributed to other repos: git tag, private registry or packed tarball.
- [ ] `SH-0003` Create one master `AGENTS.md` under `docs/engineering/` and copy it into each repo.
- [ ] `SH-0004` Review the remaining generated docs still in Downloads (`bootstrap-prompt`, `cardworld-stack-setup`) and file or archive them.

## Later / Ideas

- [ ] `SH-0005` Decide the licence split: code, base cards, base art, premium art. (O-07)
- [ ] `SH-0006` Set up a dev-only Flipt container plus OpenFeature adapter. (Needs package approval.)
- [ ] `SH-0007` Playtest the siege clock length. Start at 3 turns. (O-03)
- [ ] `SH-0008` Prototype the fan/carousel hand UI on one throwaway screen.

## Blocked

## Done (recent)

- [x] `SH-0000` Create cross-repo docs, decision log, glossary and repo map.
