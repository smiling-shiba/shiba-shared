# Review: ChatGPT-generated docs

Status: Review, 2026-09-20. Reviewer: Claude, at the owner's request.

**Read in full:** `SHIBA_RULES_STUDIO.md`, `smiling-shiba-guide-system-addendum.md`, `FOUNDATIONS.md`, and the studio's `SRS-0001` files. **Only headings skimmed:** the engineering bootstrap prompt and `ENGINEERING_GROUND_RULES.md`. **Not read:** concept notes, stack research, setup guide.

## Where the docs over-scoped

- `SHIBA_RULES_STUDIO.md` is about 1,600 lines and 33 sections. It specifies Monaco with YAML schema support, generated forms, YAML/form dual editing that preserves comments, Storybook, Playwright, Docusaurus, typedoc, a Tauri host, file watching, profile hot-reload, a docs browser, a reference index, a CLI with a joke command, and three example projects (including a "tiny roguelike" meant to show the platform is general).
- Generalizing to a platform other people build games on is a second product. Smiling Shiba does not need it yet.
- "Simulate" appears only as a later item, not in v1. Testing rule behavior belongs in `shiba-core`'s own headless tests.

## What is good and worth keeping

- **Public contract, not bundle inspection.** The studio reads generated `manifest.json` and `api-schema.json`, never the JS. Keep this.
- **Diagnostics as a product:** file, line, column, code, suggestion ("Did you mean `on_card_drawn`?"). Keep.
- **Profile vs. Ruleset split:** capabilities (code) versus content (data). This matches the owner's model.
- **Vertical slice:** typo becomes a good diagnostic, fix it, build passes.
- **Guide addendum principles:** the guide never mutates authoritative state directly; mandatory system notices are separate from mascot dialogue; no LLM dependency for authoritative behavior; versioned hook API.

## Problems

| # | Problem | Where | Suggested fix |
|---|---|---|---|
| 1 | Stale architecture: Phoenix described as the authoritative official service | `FOUNDATIONS.md` | Superseded by D-07. Kept as `research/foundations-original.md`. |
| 2 | Chat voice pasted into a spec ("Absolutely. I'd actually call it...") | `FOUNDATIONS.md` | Rewrite as neutral prose when it becomes a real foundation doc. |
| 3 | Same docs in several places | Downloads, `shiba-tools/dev` (formerly `shiba-rules-studio`), elsewhere | One home per doc (see `INDEX.md`). |
| 4 | Inconsistent vocabulary: Profile, Ruleset, game-core, game-content, "Shiba Rules" | all | See [../GLOSSARY.md](../GLOSSARY.md). Naming still open (O-10). |
| 5 | "Handler" means both the mascot and a rules function | glossary | Rename one, probably the function. |
| 6 | Executable profile code loaded by the studio is arbitrary code execution | studio spec section 29 | Trusted local profiles for now; sign official bundles (D-14). |
| 7 | Empty `AGENTS.md` in the game repo while a 14 KB one exists in the studio repo | repos | One master `AGENTS.md` in `shiba-shared/docs/engineering`, copied into repos. |
| 8 | Joke asides inside requirements can be taken as requirements | studio spec | Trim or label jokes. |

## Proposed cuts (see `shiba-tools/docs/v1-scope.md`)

1. Emit the contract from `shiba-core`, then ship a `validate` CLI.
2. Use JSON Schema with VS Code's YAML support for autocomplete and squiggles before any Studio UI.
3. Defer Monaco, forms, Storybook, Docusaurus, Tauri host, watch and hot-reload, docs browser, search index and example games.
4. Build for Smiling Shiba only.
5. Park the guide daemon in `future/`.
