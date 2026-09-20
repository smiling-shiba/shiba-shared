# Smiling Shiba — Engineering Ground Rules

This document defines the day-to-day engineering rules for this repository.

It exists so humans and future AI agents can make changes without slowly turning the project into a haunted dependency swamp.

These rules are intentionally opinionated. If a rule needs to change, change it deliberately and document why.

---

## Non-negotiable checks

Before work is considered complete:

- **All tests must pass.**
- **The linter must pass.**
- **Type checking must pass.**
- **The relevant build must pass.**
- Do not leave known warnings, skipped tests, or broken checks behind and call the task complete.
- Never weaken, delete, or bypass a failing test merely to make CI green unless the behavior itself has intentionally changed.

If a test is flaky, fix the flakiness. Do not normalize rerunning CI until it happens to pass.

---

## Branch and Git discipline

### Never commit directly to `main`

**AI agents must never commit directly to `main`. EVER.**

Use a feature/fix branch and leave the work ready for review or merge.

The human maintainer is explicitly allowed to ignore this rule, commit directly to `main`, force-push something questionable, regret it later, and receive no lectures from the agents.

Agents do not get the same privilege.

### General Git rules

- Do not rewrite shared history unless explicitly requested.
- Do not force-push unless explicitly requested.
- Do not commit secrets, credentials, generated junk, editor state, or local machine configuration.
- Keep commits coherent enough that a future engineer can understand what changed and why.
- Do not mix unrelated cleanup into a focused feature unless necessary.

---

## README hygiene

`README.md` is maintained documentation, not:

- a graveyard,
- a scratch pad,
- a changelog,
- a dumping ground,
- or a pile of obsolete setup instructions.

When changes affect setup, architecture, commands, configuration, or contributor expectations, update the README if appropriate.

Remove stale instructions instead of endlessly appending corrections underneath them.

Detailed material should live in focused documents under `docs/` rather than making the README enormous.

---

## Environment configuration hygiene

Example environment files such as `.env.example` must be kept current.

Rules:

- Every required environment variable must appear in the example file.
- Example values must be safe placeholders.
- Never put real secrets in example files.
- Remove obsolete variables when the application no longer uses them.
- Group related variables and add brief comments when their purpose is not obvious.
- Startup should fail clearly when a required production variable is missing.
- Do not silently invent dangerous production defaults.

Local developer convenience defaults are fine when they are obviously safe.

---

## Dependency policy

### New packages require approval

If a package was not explicitly requested, **ask before adding it**.

Before proposing a dependency:

1. Check whether the standard library or an existing dependency already solves the problem.
2. Prefer a small implementation over adding a large framework for one helper function.
3. Explain what the package provides and why it is worth carrying.
4. Consider maintenance activity, license, bundle/runtime impact, security history, and platform support.

Do not add packages because they are fashionable.

Use the repository's existing package manager and lockfile. Do not switch package managers without explicit approval.

---

# Code conventions

## TypeScript

Use strict TypeScript.

- Avoid `any`.
- Prefer explicit domain types over loosely shaped objects.
- Validate data at system boundaries.
- Do not use `@ts-ignore` or equivalent escapes without a documented reason.
- Prefer boring readable code over clever abstractions.
- Keep functions and modules focused.
- Delete dead code instead of commenting it out.

Game-domain code should use domain language such as `AttackLand`, `PlaySpell`, `BattleState`, and `CreatureBoon` rather than UI terminology.

---

## Architecture boundaries

The rules established in `FOUNDATIONS.md` are architectural constraints.

In particular:

- Shared game rules remain headless.
- React and Phaser are presentation layers.
- Clients send commands/intents; authoritative hosts resolve truth.
- Platform-specific behavior stays behind adapters/capabilities.
- World exploration must not become a dependency of the battle core.
- Local/custom play may be modded; official ladder play remains server-authoritative.

Do not bypass these boundaries for convenience without an explicit architecture decision.

---

# Testing practices

Testing should protect behavior, not implementation trivia.

## Unit tests

Unit-test the shared game core heavily.

Priorities include:

- legal and illegal commands,
- battle state transitions,
- land ownership changes,
- boons and curses,
- spell effects,
- graveyard behavior,
- win conditions,
- replacement/reinforcement rules,
- deterministic random behavior.

Tests should read like game rules.

Prefer:

```text
given X
when Y
then Z
```

over tests coupled to private helper functions.

## Deterministic randomness

Game randomness must be injectable or seedable.

Tests must never depend on uncontrolled random outcomes.

The authoritative server owns meaningful gameplay randomness.

## Contract tests

Protocol messages and shared schemas need contract tests so the client and authoritative host cannot silently drift apart.

Breaking protocol/schema changes must be deliberate and versioned when required.

## Integration tests

Maintain a small number of integration tests proving that:

- a client command reaches the authoritative host,
- the host validates and resolves it,
- state/events are returned correctly,
- persistence works where relevant.

## End-to-end tests

Use E2E tests for critical user journeys, not every tiny interaction.

Examples:

- create/start a local game,
- enter a battle,
- play a spell,
- resolve a land battle,
- finish a match.

## Coverage

Do not chase a vanity coverage percentage.

Critical game rules should have strong behavioral coverage. Boilerplate does not need tests merely to increase a number.

## Test hygiene

- No permanently skipped tests without a documented reason.
- No tests dependent on execution order.
- No tests dependent on external production services.
- No sleeps used as synchronization when a deterministic signal is available.
- A bug fix should normally include a regression test.

---

# Production logging

Use **structured logging** in production.

The preferred production shape is newline-delimited structured records suitable for centralized collection later.

Each useful log entry should include context where applicable:

```text
timestamp
level
service
environment
request/session/match id
player/account id only when appropriate
event/action
message
error metadata
```

Rules:

- Production services log to stdout/stderr rather than managing their own rotating log files.
- Development may use human-friendly formatting.
- Use normal severity levels: debug, info, warn, error.
- Avoid noisy logs inside hot game loops.
- Never log passwords, auth tokens, secrets, private messages, payment data, or complete sensitive payloads.
- Avoid logging complete deck/hand contents merely for convenience.
- Errors should preserve enough structured context to diagnose the failure.
- Do not swallow exceptions.

**No logging library is mandated yet.** Choose one only when needed, under the dependency approval rule.

---

# Product analytics / event tracking

Product analytics is separate from operational logging.

Create a small analytics/event interface rather than scattering vendor SDK calls throughout game code.

Example conceptual events:

```text
match_started
match_completed
battle_started
land_claimed
campaign_encounter_completed
deck_saved
season_reward_claimed
```

Rules:

- Event names and payloads should be stable and documented.
- Events should be schema-versioned when changes would break downstream consumers.
- Do not place analytics logic inside the shared rules engine.
- Local/self-hosted/custom play should not silently send gameplay analytics to official services.
- Collect the minimum useful data.
- Avoid sensitive personal information.
- A future analytics vendor must remain replaceable behind an adapter.

Gameplay telemetry used for balancing should be designed deliberately rather than logging everything and hoping it becomes useful later.

---

# Containers

Use containers where they solve a deployment or integration problem.

## Use Docker for

- official backend services,
- databases and infrastructure dependencies in development,
- reproducible integration environments,
- CI services where useful,
- eventual production service images.

## Do not use Docker for

- the desktop game client,
- the Tauri application,
- the local desktop game server merely because it is a server,
- anything that becomes harder to develop by being unnecessarily containerized.

For service containers:

- use pinned base versions,
- prefer multi-stage builds,
- run as a non-root user where practical,
- include health checks where meaningful,
- keep images small,
- keep secrets outside images,
- make local orchestration possible with a small Compose file when multiple services actually exist.

Do not build a Kubernetes theme park before there is traffic.

---

# Persistence and schema changes

Database and save-data changes must be explicit.

- Use migrations for persistent databases.
- Do not edit an already-applied production migration; add a new migration.
- Backward compatibility for local saves should be considered before changing persisted schemas.
- Save formats, content formats, and network protocols should carry versions where migration may become necessary.
- Migration failures must fail clearly rather than silently corrupting state.

SQLite remains appropriate for local/custom world state unless requirements prove otherwise.

---

# Error handling

Errors should be useful to both players and engineers.

- User-facing messages should be understandable and not expose internals.
- Logs should preserve technical detail.
- Expected domain failures should be represented explicitly where practical.
- Unexpected failures should surface rather than being silently ignored.
- Network disconnects and reconnects are expected conditions, not exceptional mysteries.

---

# Security basics

- Never trust client-provided game state in authoritative modes.
- Validate commands on the authoritative host.
- Validate untrusted content and configuration at boundaries.
- Never store secrets in source control.
- Keep authentication/authorization concerns out of the shared game rules.
- Do not execute downloaded arbitrary code on mobile.
- Desktop mods are explicitly less trusted and must not be confused with official ladder content.

Security-sensitive shortcuts require explicit review.

---

# Repository hygiene

Leave the repository cleaner than you found it, but stay within task scope.

- No unexplained generated files.
- No abandoned experimental files.
- No giant commented-out blocks.
- No mystery TODOs.

A TODO should explain the missing work and why it remains.

Prefer:

```text
TODO: reconnect state restoration after Colyseus room resume
```

over:

```text
TODO: fix this
```

---

# Architecture decisions

Use a short ADR (`docs/adr/`) for decisions that are expensive to reverse or likely to confuse future engineers.

Examples:

- choosing the official backend architecture,
- changing the authoritative rules boundary,
- replacing persistence technology,
- adopting a new mod execution model,
- introducing a major third-party platform dependency.

Do not create an ADR for every trivial coding choice.

---

# CI expectations

Once CI exists, the protected branch should require the relevant checks before merge:

```text
lint
typecheck
unit tests
integration tests
build
```

Add E2E checks when they become stable enough to be useful.

Agents must treat CI failures as unfinished work.

---

# Performance

Measure before optimizing.

Avoid premature architecture built around imagined scale.

However:

- keep authoritative game state compact,
- avoid unnecessary client/server chatter,
- avoid rendering work unrelated to visible state,
- keep mobile constraints in mind,
- do not knowingly introduce obvious N² behavior into frequently executed game logic.

---

# Things still requiring an explicit future decision

These are intentionally not locked down yet:

- exact production logging library,
- exact analytics/event vendor,
- crash-reporting vendor,
- hosting provider,
- production database for official services,
- metrics/tracing stack,
- CI provider and deployment pipeline,
- code-formatting tool if not already chosen,
- release/versioning strategy,
- package publishing strategy,
- backup/restore policy for official services,
- retention periods for logs and analytics.

When one of these becomes necessary, choose it deliberately rather than allowing the first implementation to become policy by accident.

---

# The short version

Future agent:

1. Do not commit to `main`.
2. Make the tests, linter, type checker, and build pass.
3. Keep docs and environment examples accurate.
4. Ask before adding dependencies.
5. Keep game rules headless and authoritative.
6. Test rules heavily and randomness deterministically.
7. Use structured production logs and separate product analytics.
8. Containerize backend infrastructure, not everything that moves.
9. Do not leak secrets or trust clients.
10. Do not over-engineer speculative future systems.

The human maintainer may violate rule #1.

You may not.
