# Game Concept Notes

## Core Vision

A retro-styled, open-source card RPG / roguelite with an explorable
world, deck-building, persistent progression, multiplayer worlds, and
optional official commercial services.

Touch points include **Magic: The Gathering --- Shandalar**,
roguelikes/roguelites, **Diablo-style progression**, and the strongly
themed factions of **Smash Up**.

The game itself should remain open and moddable. The commercial
ecosystem is a separate concern.

## Technology Direction

Two implementation directions were explored:

-   **Ruby** --- especially attractive for expressive game rules, card
    DSLs, modding, and the sheer charm of building a game in Ruby.
-   **JavaScript / TypeScript** --- likely the more pragmatic option for
    a browser-first game because of its larger developer ecosystem, web
    deployment, libraries, and ability to package the same application
    for desktop stores such as Steam.

A TypeScript implementation could share common packages between client
and server for card definitions, world generation, networking types, and
game rules.

Browser/local play could potentially run the game server logic locally,
while multiplayer connects to an authoritative server over WebSockets.

Desktop releases could wrap the web game using something such as Tauri
or Electron.

## Open Source vs. Commercial Services

The guiding philosophy:

> **You can have the damn game code. The official commercial ecosystem
> is ours.**

The base game can be genuinely useful and complete as open-source
software:

-   game engine
-   card/rules engine
-   world generation
-   local saves/worlds
-   multiplayer server
-   modding
-   custom rules
-   custom cards
-   self-hosting

Separate proprietary/hosted components can provide:

-   official commerce
-   payments
-   entitlements
-   promotions
-   official accounts
-   official ladder
-   official seasonal events
-   platform integrations
-   premium cosmetics

The important boundary is therefore not **source code vs. binary**, but
**software vs. official service**.

Steam/GOG/platform builds could include a private commercial/platform
pack that is not part of the public repository.

## Worlds as Servers / Saves

A local game world is effectively a named server and a save file at the
same time.

Players could:

-   create any number of servers/worlds
-   name them
-   make them public, friends-only, or private
-   save and resume them
-   duplicate/export/import them
-   customize their rules
-   enable mods

A server owns the state for its particular run/world. Persistent account
services do **not** need to store the generated world itself.

## Procedural World Model

Each server generates its own world map.

Rather than storing or constantly transmitting an enormous map, the
world can largely be represented as:

**generation seed + configuration + mutations**

For example:

``` text
Seed: 918273645

Mutations:
- tile 441 -> encounter completed
- tile 120 -> chest looted
- tile 982 -> settlement built
- tile 333 -> boss defeated
```

Clients can deterministically generate the base world from the same
seed. The server distributes authoritative state changes/deltas.

Completed map objects or encounters might become grey/desaturated so
players can immediately see what has already been consumed.

World configuration could include parameters such as:

-   map size
-   biome distribution
-   difficulty
-   PvP rules
-   loot rates
-   settlement rules
-   special modifiers

These settings could be serialized into a compact shareable world code
alongside the RNG seed.

## Multiplayer Architecture

Each active world/run can have an authoritative game server.

Clients communicate through WebSockets and submit **intentions**, not
authoritative results.

For example:

``` text
Client: "I want to play card 482 on target 91."

Server:
- validates the card
- validates cost
- validates timing
- validates target
- resolves effects
- mutates state
- broadcasts the result
```

The same principle applies to movement, map interactions, rewards,
settlement upgrades, and other important state.

Friends can join an existing world unless it has been configured as
private.

## Settlements

Each player may claim a valid map tile and establish a personal
settlement.

Possible progression:

``` text
Camp
  -> Outpost
  -> Fort
  -> Town
  -> Citadel
```

Settlements could provide:

-   resource generation
-   healing
-   temporary buffs
-   crafting
-   card upgrades
-   merchants
-   daily quests
-   fast travel
-   storage
-   settlement cosmetics
-   other progression systems

Settlement location itself can become strategic because different
terrain could affect resource production or bonuses.

Moving a settlement can simply update its `(x, y)` coordinates after the
server validates that the destination tile is legal, reachable,
unoccupied, and appropriate for construction.

## Modding and Custom Servers

Modding should be embraced rather than fought.

Supported customization could include:

-   custom world generation
-   XP/drop multipliers
-   PvE/PvP rules
-   custom cards
-   custom personas
-   encounters
-   biomes
-   settlement rules
-   total conversions

Configuration can handle ordinary server customization, while a
supported mod API can handle deeper changes.

Arbitrary executable mods should be treated as trusted server-side code
rather than downloaded and executed blindly by clients.

## Ladder Mode

Official ladder play is deliberately different from open/custom play.

Custom/local:

-   mods allowed
-   custom rules
-   custom cards
-   custom worlds
-   self-hosting

Official ladder:

-   official ruleset
-   official card pool
-   no mods
-   authoritative official servers
-   authenticated players/builds
-   controlled seasonal content

Security should rely primarily on **server authority**, not trusting the
client to report that it is unmodified.

Modern cryptographic hashes/signatures should be used rather than MD5
for security-sensitive verification.

### Server-Owned Cards and Decks

In ladder mode there are no authoritative client-side decks or card
definitions.

The ladder service owns:

-   legal card definitions
-   player collections
-   deck lists
-   deck legality
-   card versions / balance changes
-   match state
-   results
-   ratings
-   rewards

Clients may cache artwork/text for presentation, but the server's rules
and card definitions always win.

Editing a local card to deal `999999` damage therefore accomplishes
nothing on the official ladder.

## Personas and Deck Identity

Decks should have strong themes and identities similar in spirit to
**Smash Up factions**, but personas can also modify global game rules.

A persona isn't merely artwork or a starting deck. It establishes a
different philosophy for interacting with the card system.

### Necromancer

Core identity: **graveyard manipulation**.

Possible global mechanic:

> Once per turn, play/retrieve a qualifying card from your Graveyard.

The discard/graveyard effectively becomes another strategic resource.

Example achievements:

**Necro Grifter**\
Drain another player's Prestige to zero while playing the Necromancer.

**Dead Sea Scrolls**\
*"There's nothing dead here!"*\
Defeat another player as the Necromancer without using the Graveyard.

### The Fisherman

Originally conceived as a boss, but the mechanics were strong enough to
become a full persona.

Core identity: **card location and deck-order manipulation**.

Possible global mechanic:

> Once per turn, look at the top card of a draw pile and either leave it
> or move it to the bottom.

Potential thematic vocabulary:

-   **Fish** --- inspect cards from a zone
-   **Catch** --- take/use a selected card
-   **Release** --- return it
-   **Sink** --- place it on the bottom
-   **Bait** --- manipulate what an opponent will draw
-   **Trawl** --- inspect multiple cards

Example card concepts:

-   **Hook, Line & Sinker** --- inspect several cards and temporarily
    play one.
-   **Bottom Feeder** --- play the bottom card of a draw pile.
-   **Catch & Release** --- temporarily remove/manipulate a card from an
    opponent's hand.
-   **Piranha** --- punishes opponents for fishing it out of your deck.

The Fisherman can support multiple builds: control, theft, self-deck
manipulation, etc.

### HIMBO Hunter

Core identity revolves around **HIMBOs and Prestige thresholds**.

For example, high-Prestige HIMBOs could become *Hunted*, creating
bonuses for the Hunter or making them vulnerable to specialized cards.

A suitably stupid possible rule:

> If his Prestige exceeds his Intellect, draw a card.

## Cross-Persona Design

Most cards belong to strongly identifiable persona pools.

Occasional cross-persona cards can therefore feel special.

Example:

**Deadliest Catch** --- Fisherman / Necromancer\
Fish several cards from your Graveyard, catch one to play, and sink the
others.

Keeping cross-persona effects relatively uncommon preserves strong deck
identities and reduces balance chaos.

## Roguelite / Progression Structure

A run is temporary, while selected progression survives between runs.

During a run players may collect:

-   cards
-   upgrades
-   relics/power-ups
-   resources
-   settlement upgrades
-   temporary buffs
-   increasingly difficult encounters

Persistent progression might include:

-   unlocked personas
-   unlocked cards
-   cosmetics
-   achievements
-   starting options
-   account progression

Procedural scaling and recombination allow worlds to remain replayable
without requiring infinite handcrafted content.

## Commerce

Commerce should remain separate from the core game.

Potential products include:

-   cosmetics
-   card backs
-   board skins
-   settlement cosmetics
-   avatars
-   animations
-   progression boosts
-   temporary XP boosts
-   premium seasonal progression

Example:

``` text
XP BOOST
+100% XP
Duration: 1 hour
```

Competitive advantages should be treated carefully in multiplayer.
Cosmetics and progression acceleration are easier to separate from match
outcomes than direct combat power.

A specialized game-commerce provider could handle catalogs, payments,
promotions, and entitlements rather than making the game itself
responsible for payment processing.

## Seasons and DLC

Large content drops should happen on a sustainable cadence --- perhaps
**annual or bi-annual** --- rather than turning every spontaneous idea
into an immediate release obligation.

A persona becomes a natural unit around which an expansion can be
designed.

Example seasonal package:

``` text
Season: Something Fishy

New Persona:
- The Fisherman

New Content:
- Fisherman card pool
- Drowned Coast biome
- solo campaign
- world events
- bosses/encounters
- achievements
- cosmetics
- settlement decorations

Online:
- seasonal ladder
- seasonal progression tree
- free rewards
- premium rewards
```

The same development work can serve both solo and multiplayer players.

## End of Season

Season availability and ownership/entitlement are separate concepts.

When a season ends:

-   seasonal ladder closes
-   seasonal UI/buttons disappear
-   temporary seasonal services/events stop
-   the next season can begin later

Players retain things they legitimately earned or purchased, such as:

-   unlocked persona
-   unlocked cards
-   cosmetics
-   achievements/titles
-   settlement decorations
-   purchased premium unlocks

The end of a competitive season should not erase ownership.

## Development Philosophy

The project should not require continuous live-service churn.

Ideas can accumulate in a backlog and mature into cohesive expansions
rather than immediately becoming production commitments.

The base game should remain playable and useful between content
releases.

In short:

> **Build a game first. Operate optional official services around it.
> Let people hack on the game. Keep official commerce, ladder authority,
> and commercial infrastructure separate.**
