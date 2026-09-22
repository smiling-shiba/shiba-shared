# BACKLOG.md (shiba-shared)

Story prefix: `SH-`. This repo holds the **epic definitions** for the whole Smiling Shiba universe (`SS-NN`) plus cross-repo and documentation stories. Other repos keep their own `docs/BACKLOG.md` and tag stories with an epic ID.

Convention: [engineering/backlog-and-ids.md](engineering/backlog-and-ids.md). Keep "Now" to 3 items or fewer.

## Epics

### SS-01: Rules contract & validation
Goal: a game's rules live in a policy (code) written with the `shiba-sdk` SDK; `shiba-tools` (`sht`) validates YAML templates against the policy's contract and signs packs for official use.
Repos: `shiba-sdk` (SK-), `shiba-tools` (ST-).
Status: in progress and the current focus. The toolset is built and tested: `sht validate`, `schemas`, `build-policy`, `pack`, `keygen`, `sign` and `verify`, with `shiba-sdk` generating contracts. Remaining: the pack loader in the app. The runner lives in `shiba-app`; there is no shared rules code (D-42).

### SS-02: First playable vertical slice (after the toolset is stable)
Goal: prove the architecture end to end. Four lands per side, an `ATTACK_LAND` command, a two-creature battle with one boon/curse each, one Spellbook card, authoritative resolution, rendered in the client, and the same battle running headless in a core test.
Repos: `shiba-sdk` (SK-), `shiba-app` (SA-).
Status: not started.

### SS-03: Official services
Goal: accounts, matchmaking, seasons, entitlements and the rules registry.
Repos: `shiba-mps` (MP-).
Status: started. The Phoenix API scaffold exists (`MP-0002`, `MP-0003`); the rest waits until SS-02 is proven. The server's rules are its own, written in Elixir later (D-42). The QuickBEAM spike is finished and kept as findings.

## Now

Two decisions are waiting on the owner; nothing else is in progress.

- [ ] `SH-0002` Decide how `shiba-sdk` is distributed to other repos: git tag, private registry or packed tarball. *(waiting on you)*
- [ ] `SH-0009` Talk through packs as single files and mod layering; decide what to build. Notes: [design/packs-and-modding.md](design/packs-and-modding.md). *(waiting on you)*

## Next

- [ ] `SH-0003` Create one master `AGENTS.md` under `docs/engineering/` and copy it into each repo.
- [ ] `SH-0010` Connect the repos to GitHub, under the `smiling-shiba` org. Public: `shiba-shared`, `shiba-sdk`, `shiba-tools`, `shiba-app`. Private: `shiba-mps` (D-02, D-29). Each gets `main` branch protection: require a pull request before merging (no direct pushes, `enforce_admins` on so this applies to the owner too), no formal required-approval count. Decided 2026-09-21: since every push goes through the owner's own GitHub account, GitHub will not let a required approval be satisfied (you cannot approve your own PR), so the merge click itself is the approval, not a separate review step.

## Later / Ideas

- [ ] `SH-0012` Talk through giving Claude visibility into your GitHub: what it may read, what it may change, and how access is granted. Not now. Do after `SH-0010`.
- [ ] `SH-0013` Secrets behind a small interface. Start with GitHub Secrets for CI and plain environment variables at run time, but write code against one small "get a secret" interface so the provider can be swapped later (Vault, a cloud manager). No cloud secret manager for now. Note: GitHub Secrets cannot be read back once saved, and only reach a workflow as environment variables. The real signing key lives in CI only (D-38). Decide the interface before the first real secret is added.
- [ ] `SH-0014` Event tracking and logs behind small interfaces, plug and play, so vendors can be swapped without touching game code. Builds on the `log(event, data)` adapter (D-17) and on open question O-09 (which library and vendor). Flags already sit behind OpenFeature (D-15). Decide before the first vendor is added.
- [ ] `SH-0006` Set up a dev-only Flipt container plus OpenFeature adapter. (Needs package approval.)
- [ ] `SH-0007` Playtest the siege clock length. Start at 3 turns. (O-03)
- [ ] `SH-0008` Prototype the fan/carousel hand UI on one throwaway screen.

## Blocked

- [ ] `SH-0011` GitHub Actions for each repo. Blocked on `SH-0010`. `shiba-tools` and `shiba-sdk`: lint, typecheck and tests on Node 24. `shiba-mps`: `mix test` with a Postgres service, and later the Docker image (`MP-0013`). Signing runs here only, with the key in GitHub Secrets (D-38, `SH-0013`).

## Done (recent)

- [x] `SH-0005` Decide the licence split (O-07). Answer (D-43): code is MIT; base cards, base art and premium art stay closed for now.

- [x] `SH-0001` Decide where the shared rules code and the runner live (O-14). Answer (D-42): no shared rules code. The app owns its runner, and the ladder server has its own rules in Elixir.

- [x] `SH-0000` Create cross-repo docs, decision log, glossary and repo map.
