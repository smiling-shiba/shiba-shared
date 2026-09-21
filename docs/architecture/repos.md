# Repository Map

Status: Proposal. Only create a repo when it has a real first commit's worth of work.

## Existing

| Repo (path) | Purpose | Visibility |
|---|---|---|
| `shiba-shared` (formerly `shiba-dev`) | Cross-repo docs and epics | Private or public, TBD |
| `shiba-sdk` | Helpers for writing a policy in a standard shape (the SDK). Generates a policy's contract. Ships a toy policy as its example and test fixture. Holds no game rules. | Public |
| `shiba-tools` (CLI alias `sht`) | CLI that validates templates against a policy's contract, and builds, packs, signs and verifies packs | Public or private, TBD |
| `shiba-app` | The local game: Vite + React + Phaser client, Tauri shell, and the Colyseus local/custom world server run as its sidecar. Consumes `shiba-sdk`. | Public |
| `shiba-mps` | Multiplayer server (Elixir/Phoenix): accounts, matchmaking, Channels/Presence, seasons, entitlements, rules registry. It runs its own rules, in Elixir, and is the source of truth for multiplayer outcomes. It does not load a policy bundle (D-42). | Private |

## Planned (not yet created)

| Repo | Purpose | Visibility |
|---|---|---|
| Game data (policy, templates, assets) | One game's rules and content, in its own repo per game. May include base content so forks can merge it. | Public or private |
| Commercial pack | Store, Steam/GOG adapters, promotions | Private |
| Assets | Base art (open license) and premium art (proprietary). Large binaries want Git LFS or separate storage. | Split |
| Infra | Docker Compose, deploy config, Flipt and logging setup | Private |
| Website / portal | Marketing, patch notes, account portal | TBD |

## Dependency direction

```text
shiba-sdk  <-  a game's policy        (imports it to describe the policy)
policy contract  <-  shiba-tools       (reads the generated contract, never policy source)
built policy bundle  <-  shiba-app   (loaded from a pack, for local play and previews)
```

`shiba-sdk` depends on nothing in the Shiba universe. It must not import React, Phaser, Tauri, Colyseus, a database or browser APIs.

## No monorepo (D-26)

Each repo stands alone with its own lockfile and CI. Consequences to plan for:

- **Sharing code = consuming a package or a built file.** `shiba-sdk` is versioned and installed by whoever writes a policy (a git tag, a private registry, or a packed tarball; pick when the first consumer exists). The app and the Elixir server load the built policy file. Nothing imports another repo's source by path.
- **Keep the shared surface small.** There is no shared rules code; the runner lives in `shiba-app` (D-42). Input abstractions stay in the client.
- **Version pinning is the coordination tool.** Matches already pin runtime and pack versions, so cross-repo version discipline is part of the design.
- **Local development across repos** needs a linking step (`npm link`, or a `file:` dependency) while changing core and a consumer together. Document the exact steps in `shiba-app` when it has code.
- **Docs and IDs:** cross-repo docs stay in `shiba-shared`; each repo owns its `docs/BACKLOG.md` and story prefix.

Revisit only if the linking and release overhead becomes the main source of pain.

## Decisions still needed

- Does base content live in the client repo or its own repo? Leaning: with the game, so forks merge it.
- Where do the ID prefixes point? Epics use the product prefix (`SS-01`); stories use the repo prefix (`ST-0014`). See [../engineering/backlog-and-ids.md](../engineering/backlog-and-ids.md).
- Split code, base-card and art licences early.
