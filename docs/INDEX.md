# Smiling Shiba: Docs Index (cross-repo)

Start here. This folder holds architecture, decisions and design that span more than one repo. Repo-specific docs live in that repo's own `docs/`.

Last reviewed: 2026-09-20.

## Repos

See [architecture/repos.md](architecture/repos.md) for the full map. Docs folders:

| Repo | Docs folder | Holds |
|---|---|---|
| `shiba-shared` | this folder | Cross-repo architecture, decisions, game design, engineering rules, future ideas |
| `shiba-sdk` | `shiba-sdk/docs/` | SDK and deterministic runtime for policies; the pack format, contract and signing |
| `shiba-tools` (CLI alias `sht`) | `shiba-tools/docs/` | CLI/validator tooling only |
| `shiba-app` | `shiba-app/docs/` | Local game: client, Tauri shell, local Colyseus server |
| `shiba-mps` | `shiba-mps/docs/` | Elixir multiplayer server: accounts, matchmaking, seasons, registry |

## Read in this order

1. [GLOSSARY.md](GLOSSARY.md): the vocabulary. Read before anything else.
2. [architecture/overview.md](architecture/overview.md): the one-page picture.
3. [architecture/repos.md](architecture/repos.md): what lives where.
4. [decisions/log.md](decisions/log.md): what is settled, leaning, open, or rejected.
5. [design/game-design.md](design/game-design.md): game rules and ideas, by theme.

## Everything else

| Doc | Status | Notes |
|---|---|---|
| [engineering/ground-rules.md](engineering/ground-rules.md) | Current | Copy of `ENGINEERING_GROUND_RULES.md`. Agents never commit to `main`. |
| [BACKLOG.md](BACKLOG.md) | Current | Cross-repo work queue and epic definitions. |
| [design/packs-and-modding.md](design/packs-and-modding.md) | Idea | Single-file packs, mod layering, trust per pack. To discuss. |
| [engineering/backlog-and-ids.md](engineering/backlog-and-ids.md) | Current | `BACKLOG.md` convention. Epics live in `shiba-shared/docs/BACKLOG.md`; stories use `SH-`, `SK-`, `ST-`, `SA-`, `MP-`. |
| [future/guide-daemon.md](future/guide-daemon.md) | Parked | Shiba Handler / tutorial system. Do not build yet. |

## Conventions

- Every doc starts with a `Status:` line: `Draft`, `Current`, `Proposal`, `Parked`, `Historical` or `Superseded`.
- One topic per file. Link to other docs by relative path within a repo, and by repo name across repos.
- `dev/` folders are local, gitignored scratch space and not part of the project docs.
- Update the relevant doc in the same PR as the change it describes.
