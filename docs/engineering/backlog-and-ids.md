# Backlog and ID Convention

Status: Proposal.

## Files

Each repo has one `BACKLOG.md`, in its `docs/` folder. Do not keep a second overlapping task file, because two files drift.

```markdown
# BACKLOG.md

## Epic SS-01: Pack validation and signing
Goal: validate templates against a policy's contract, and sign packs.

### Now (max 3)
- [ ] `ST-0014` Validate function names against the contract

### Next
### Later / Ideas
### Blocked
### Done (recent)
```

## IDs

- **Epics** use the product prefix: `SS-01`, `SS-02`.
- **Repo prefixes:** `SH-` shiba-shared, `SK-` shiba-sdk, `ST-` shiba-tools, `SA-` shiba-app, `MP-` shiba-mps.
- **Stories** use the repo prefix: `ST-0014` (shiba-tools, alias `sht`), `SK-0003` (shiba-sdk).
- IDs are zero-padded, never renumbered and never reused. Before adding one, look at the highest number already in that file and take the next.
- A story names its epic in its heading or with a `[SS-01]` tag.
- The epic's definition lives in one repo (the product repo); other repos link to it by ID.

## Rules for agents

- Update `BACKLOG.md` in the same branch and PR as the work, never on `main`.
- "Now" is capped at 3. If nothing is in progress, say so there in one line instead of leaving it empty.
- Prune old Done items into `CHANGELOG.md`.
- Ideas that arrive at 3 AM go under "Later / Ideas", not into a release.
- Small, single-line edits reduce merge conflicts when agents run in parallel.

## Optional

If a `BACKLOG.md` gets long, split into `backlog/epics/SS-01-*.md` and keep `BACKLOG.md` as a short index.
