# Packs, single-file packaging and modding

Status: Idea, to discuss. Nothing here is decided or built beyond "Where we are".

## Where we are

- A **pack** is a folder: `pack.yml`, `policies/`, `templates/`, `assets/`.
- `sht pack` writes `pack.lock.json` (a hash of every file), `sht sign` writes `pack.sig.json`, and `sht verify` checks both. See `shiba-tools/docs/signing.md` and decision D-38.
- One signature covers the **whole pack**, not just the policy code, because template values (costs, numbers) change outcomes as much as code does.

## Gaps worth thinking about

1. **A single-file pack** (for example `.shibapack`): an archive holding the folder that can be shared, downloaded and dropped into the app. Open points: how the lock and signature travel with it, and making the archive byte-for-byte reproducible.
2. **Layering for mods:** a base pack plus mod packs on top. Open points: load order, and what happens when two packs define the same kind and id (replace, merge, or error).
3. **What a mod may carry:** templates and assets only, or its own policy code too? Executable code is allowed on desktop local mode but not on mobile (D-18).
4. **Dependencies and compatibility:** a mod saying "I need base pack 2026.09 or newer." Today a pack only declares an SDK version range.
5. **Trust per pack:** local games may run unsigned packs with a warning; the official ladder accepts only the official key. The app needs a folder of trusted keys and should show a player who signed a pack.
6. **Checking the combined result:** `sht validate` checks one pack. A stack of packs needs to be validated together.
7. **Updates and versions:** packs already use generated calver; what does updating a mod look like?

## Related

- Pack loader in the app (`SA-0013`), key rotation and revocation and the app-side verify (`ST-0017`, `ST-0018`).
- The runner lives in `shiba-app` (D-42).
- Custom games never send their pack to `shiba-mps` (D-31).

## Questions for later

- Do mods carry their own code, or only templates and assets?
- One active pack at a time, or a stack of packs?
- Is a single-file format needed before the app exists, or only once mods do?
