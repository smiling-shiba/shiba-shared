# Smiling Shiba: Docs Index (cross-repo)

Start here. This folder holds architecture, decisions and design that span more than one repo. Repo-specific docs live in that repo's own `docs/`.

Last reviewed: 2026-09-20.

## Repos

See [architecture/repos.md](architecture/repos.md) for the full map. Docs folders:

| Repo | Docs folder | Holds |
|---|---|---|
| `shiba-shared` | this folder | Cross-repo architecture, decisions, game design, engineering rules, future ideas |
| `shiba-core` | `/Users/kevin/repo/shiba-core/docs` | Rules engine: handler functions, manifest, versioning, signing |
| `shiba-tools` (CLI alias `sht`) | `/Users/kevin/repo/shiba-tools/docs` | CLI/validator tooling only |
| `shiba-app` | `/Users/kevin/repo/shiba-app/docs` | Local game: client, Tauri shell, local Colyseus server |
| `shiba-mps` | `/Users/kevin/repo/shiba-mps/docs` | Elixir multiplayer server: accounts, matchmaking, seasons, registry |

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
| [engineering/backlog-and-ids.md](engineering/backlog-and-ids.md) | Current | `BACKLOG.md` convention. Epics live in `shiba-shared/BACKLOG.md`; stories use `SH-`, `SC-`, `ST-`, `SA-`, `MP-`. |
| [future/guide-daemon.md](future/guide-daemon.md) | Parked | Shiba Handler / tutorial system. Do not build yet. |
| [research/game-concept-notes.md](research/game-concept-notes.md) | Historical | Early concept summary. Superseded by `design/game-design.md` where they differ. |
| [research/typescript-game-stack-research.md](research/typescript-game-stack-research.md) | Historical | Stack survey with sources. Some conclusions changed, see the decisions log. |
| [research/foundations-original.md](research/foundations-original.md) | Partly stale | Original `FOUNDATIONS.md`. Its "Phoenix is authoritative" text was superseded, see decisions log D-07. |
| [research/shared-chat.md](research/shared-chat.md) | Raw source | Full 240-turn ChatGPT conversation. Design history, includes jokes. |
| [reviews/2026-09-20-chatgpt-docs-review.md](reviews/2026-09-20-chatgpt-docs-review.md) | Review | Findings on the generated docs and the cuts proposed. |

## Not yet organized (still in Downloads)

- `smiling-shiba-engineering-bootstrap-prompt.md` (about 1,600 lines). Only its headings were skimmed, not read in full. Review before copying in.
- `cardworld-stack-setup.md`. Predates the rename from "cardworld" and the React-versus-Next decisions.

## Conventions

- Every doc starts with a `Status:` line: `Draft`, `Current`, `Proposal`, `Parked`, `Historical` or `Superseded`.
- One topic per file. Link to other docs by relative path within a repo, and by repo name across repos.
- `dev/` folders are the owner's gitignored scratch pad. Agents do not write there unless asked.
- Update the relevant doc in the same PR as the change it describes.
