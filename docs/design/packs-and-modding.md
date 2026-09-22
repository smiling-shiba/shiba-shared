# Packs, single-file packaging and modding

Status: Decided on the shape (D-45), 2026-09-22. Implementation details below are next steps, not decided.

## The shape

- **A pack** = one policy + its base templates/assets. Unchanged from today.
- **A mod** = templates and assets only, never code. A pack-shaped folder (or later, archive) without `policies/`.
- One active policy at a time. Many mods may layer templates/assets on top of it.
- A ruleset that needs mechanics the active policy's functions and hooks do not expose is not a mod — it's a different pack, built and shared on its own with the public SDK.

This closes most of the open questions the earlier version of this doc raised (see D-45 for why): a mod can never do more than a template already could, so it needs no new trust or execution model, and D-38's whole-pack lock/sign/verify already applies unchanged.

## Where we are

- `sht pack` writes `pack.lock.json` (a hash of every file), `sht sign` writes `pack.sig.json`, and `sht verify` checks both. See `shiba-tools/docs/signing.md` and decision D-38.
- One signature covers the **whole pack**, not just the policy code, because template values (costs, numbers) change outcomes as much as code does. The same applies to a mod.

## Next steps (not decided, not built)

1. **Load order and collisions.** An ordered list of active mods (host-configured), later wins on a same-kind-same-id collision — the boring, proven default (Minecraft, Factorio). Merge semantics are more powerful and much harder to debug; skip for v1. Worth an error (not a silent pick) when two mods at the *same* position both introduce a new id that doesn't exist in the base — that's a real authoring conflict, not an intentional override.
2. **Validating a stack.** `sht validate` already checks one pack's templates against its policy's contract. Extend it to walk mods in load order against the same contract, flagging real id collisions as it goes. Not a new paradigm, an incremental one.
3. **Single-file format** (for example `.shibapack`): a mod or pack is already a pack-shaped folder, so this is close to "zip the folder" once it matters. Only needed once the app has a place to drop one into — i.e., once `SA-0013` (the app's pack loader) exists. Don't build ahead of it.
4. **Trust per mod.** A mod carries the same real risk as any unsigned pack today (bad or unexpected game state, never code execution) — D-18's existing stance (local mode runs unsigned with a warning) extends cleanly. The app should still show who signed what, once it has a UI to show it in.
5. **Dependencies and compatibility.** A mod saying "I need base pack 2026.09 or newer." Today a pack only declares an SDK version range (D-36); a mod would need to declare a target pack (and maybe version range) too. Revisit once real mods exist to test the idea against.
6. **Updates and versions.** Packs and mods both use generated calver already (D-36). What "update a mod" looks like in the app's UI is a product question for later, not a format question.

## Related

- Pack loader in the app (`SA-0013`), key rotation and revocation and the app-side verify (`ST-0017`, `ST-0018`).
- The runner lives in `shiba-app` (D-42).
- Custom games never send their pack to `shiba-mps` (D-31).
