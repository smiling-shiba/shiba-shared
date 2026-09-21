# Glossary

Status: Draft.

## Game

| Term | Meaning |
|---|---|
| **Land** | One of four territories on a player's half of the board. Holds one creature and belongs to a control/biome. |
| **Homeland** | A player's original four lands. |
| **Chain** | Lands are ordered 1 to 4. A player can only attack a land adjacent to territory they hold. If a link is cut, everything held behind the cut reverts. |
| **Siege clock** | X turns (start at 3) a player has to re-occupy a homeland land after all four are held by an opponent. Losing all lands is not instant death. |
| **Army (deck)** | The creature deck. Draw 1 per turn on the strategic layer, deploy onto lands. |
| **Spellbook** | The tactical deck of spells and abilities. Draw 5-7 when a land is attacked. |
| **Creature** | A card that occupies a land. Has no numeric stats, only one short boon or curse. |
| **Boon / Curse** | A creature's one rule: a boon helps its owner, a curse hinders the opponent. |
| **Might** | Considered and set aside. Creatures have no combat stats. Not a game term. |
| **Persona** | A deck identity: a global passive rule plus a card family (Necromancer, Fisherman...). The unit of expansion. |
| **Domain** | A persona's element projected onto the board (Fire, Darkness, Nature...). |
| **Synergy zone** | Where compatible domains meet and create a hybrid biome (Wildfire, Black Tide...). |
| **Battle zone** | The terrain an attacker declares a fight in. Effects apply to both sides. |
| **Prestige** | A persona-related resource that some cards drain or steal (Grifter, HIMBO Hunter). Exact rules undecided. |
| **Fusion / Fission** | Temporary, battle-scoped card combination / splitting. |
| **Secret objective** | A hidden per-persona alternate win (Necromancer: exactly 13 cards in graveyard at end of turn). |
| **Handler** | The Smiling Shiba: the in-world guide/mascot. See [future/guide-daemon.md](future/guide-daemon.md). |

## Modes

| Term | Meaning |
|---|---|
| **Local / Custom mode** | Player-owned server, no account, mods allowed, cheating is the host's own concern. |
| **Ladder / Official mode** | Official server owns cards, decks, RNG and rewards. Account required. No mods. |
| **Season** | A themed content package: persona, biome, campaign, ladder, progression. Ends; entitlements remain. |
| **Entitlement** | An account's right to use something (persona, cosmetic). Separate from season availability. |

## Rules tech

| Term | Meaning |
|---|---|
| **SDK** (`shiba-sdk`) | A library of helpers for *writing* a policy in a standard shape (`definePolicy`, `kind`, `hook`, `fn`, `t.*`). It holds no game rules and does not run anything. |
| **Runner** | Code that plays a policy: runs a template's steps by calling the policy's functions. Not built. Where it lives is open (O-14). |
| **Policy** | A game's rules as code (JS/TS), built to one bundle, versioned and signed. Written with the SDK. |
| **Policy function** | A named, trusted function in a policy that templates can call. |
| **Hook** | A named moment (for example a battle starting) that a template can attach steps to. Declared by the policy. |
| **Kind** | A type of template (for example `entity`), declared by the policy along with the fields it accepts. |
| **Template** | A YAML file describing one object of some kind. Holds data and calls to policy functions, never code. |
| **Asset** | Art, audio or other files that templates refer to. |
| **Pack** | A policy plus its templates and assets. What `shiba-app` loads. |
| **Contract** (`contract.json`) | Generated from a policy: its kinds, functions, hooks and argument schemas. `sht` reads this, never the policy source. |
| **Match pin** | A match records its runtime version and pack version and never changes them. |
| **Pack registry** | Official backend database of pack versions and their status (draft, active, deprecated, disabled, emergency_blocked). |
| **Feature flag** | An on/off switch for engineering gates and emergency toggles. Not a place for rules. |
| **Rules toggle** | A flag that disables a card, persona or queue. Short-lived. |
| **Sidecar** | Node process (Colyseus) that the Tauri desktop app launches as the local server. |
| **Authoritative** | The server decides what happened. Clients send intents, never results. |
