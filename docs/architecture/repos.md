# Repository Map

Status: Proposal. Only create a repo when it has a real first commit's worth of work.

## Existing

| Repo (path) | Purpose | Visibility |
|---|---|---|
| `shiba-shared` (formerly `shiba-dev`) | Cross-repo docs and epics | Private or public, TBD |
| `shiba-core` | Headless rules engine: handler functions, manifest/API schema, ruleset compiler and validator | Public |
| `shiba-tools` (CLI alias `sht`; formerly `shiba-rules-studio`) | CLI tooling that validates and signs rulesets against the engine's manifest | Public or private, TBD |
| `shiba-app` | The local game: Vite + React + Phaser client, Tauri shell, and the Colyseus local/custom world server run as its sidecar. Consumes `shiba-core`. | Public |
| `shiba-mps` | Multiplayer server (Elixir/Phoenix): accounts, matchmaking, Channels/Presence, seasons, entitlements, rules registry. It loads the `shiba-core` bundle into the BEAM and is the source of truth for multiplayer outcomes (D-30). | Private |

## Planned (not yet created)

| Repo | Purpose | Visibility |
|---|---|---|
| Base rulesets (content) | YAML cards, personas, creatures. May live in `shiba-app` instead, so forks receive base content by merging upstream. | Public |
| Commercial pack | Store, Steam/GOG adapters, promotions | Private |
| Assets | Base art (open license) and premium art (proprietary). Large binaries want Git LFS or separate storage. | Split |
| Infra | Docker Compose, deploy config, Flipt and logging setup | Private |
| Website / portal | Marketing, patch notes, account portal | TBD |

## Dependency direction

```text
shiba-core  <-  shiba-tools           (consumes built manifest/bundle, not source)
shiba-core  <-  shiba-app            (client previews and local Colyseus server)
shiba-core  <-  shiba-mps            (bundle loaded into the BEAM, source maps optional, D-30)
```

`shiba-core` depends on nothing in the Shiba universe. It must not import React, Phaser, Tauri, Colyseus, a database or browser APIs.

## No monorepo (D-26)

Each repo stands alone with its own lockfile and CI. Consequences to plan for:

- **Sharing code = consuming a package.** `shiba-core` is versioned and installed by the others (a git tag, a private registry, or a packed tarball; pick when the first consumer exists). Nothing imports another repo's source by path.
- **Keep the shared surface small.** Protocol types and command/event schemas live inside `shiba-core` rather than in a fourth repo. Input abstractions stay in the client.
- **Version pinning is the coordination tool.** Matches already pin engine and ruleset versions, so cross-repo version discipline is part of the design.
- **Local development across repos** needs a linking step (`npm link`, or a `file:` dependency) while changing core and a consumer together. Document the exact steps in `shiba-app` when it has code.
- **Docs and IDs:** cross-repo docs stay in `shiba-shared`; each repo owns its `BACKLOG.md` and story prefix.

Revisit only if the linking and release overhead becomes the main source of pain.

## Decisions still needed

- Does base content live in the client repo or its own repo? Leaning: with the game, so forks merge it.
- Where do the ID prefixes point? Epics use the product prefix (`SS-01`); stories use the repo prefix (`ST-0014`). See [../engineering/backlog-and-ids.md](../engineering/backlog-and-ids.md).
- Split code, base-card and art licences early.
