# Smiling Shiba - Guide / Handler System Addendum

This document is a **future implementation note** for the Smiling Shiba project.

It records the intended architecture for the game's tutorial assistant, quest-giver, handler, narrator, occasional mischief-maker, and mascot-driven intervention system.

This system does **not** need to be implemented during the initial rules-engine bootstrap.

The purpose of this document is to preserve the design intent so that, when the project eventually reaches this feature, the implementation fits the rest of the architecture instead of becoming a pile of UI conditionals and hardcoded Shiba jokes.

---

# 1. Core idea

The in-world guide is not the same thing as the machinery that powers guide behavior.

Separate:

```text
SHIBA DAEMON
    =
background event observer / scheduler / intervention coordinator

GUIDE PROFILE
    =
identity / name / portrait / voice / tone / presentation

GUIDE CONTENT
    =
dialogue / tutorials / quests / reactions / jokes / mischief definitions

GUIDE HOOK API
    =
documented game moments and contextual data that guide content may respond to
```

The default experience may present this machinery as **Smiling Shiba**.

Internally, the code can continue to call the subsystem:

```text
Shiba Daemon
```

even if a player replaces the visible guide with something else.

The name is intentionally appropriate in both senses:

- Shiba is a mischievous in-world entity that is always lurking around the player's experience.
- A daemon is a background process that waits for events and performs work when conditions are met.

---

# 2. Shiba's fictional role

The default Smiling Shiba guide should feel like a combination of:

- handler,
- recruiter,
- boss,
- chief,
- quest giver,
- narrator,
- tutorial assistant,
- plot nudger,
- corporate performance-review menace,
- and chaos gremlin.

The character should be:

- ambitious,
- mischievous,
- snarky,
- self-amused,
- occasionally helpful,
- occasionally suspiciously helpful,
- and willing to get his giggles at the player's expense.

He is not necessarily malicious.

He should feel like someone who sincerely believes that inconvenience builds character.

Example tone:

```text
"Excellent work surviving that ambush."

"I've scheduled another one."
```

Or:

```text
PERFORMANCE REVIEW COMPLETE

Meets Expectations
```

He may communicate:

- tutorials,
- quest offers,
- system introductions,
- world events,
- challenge prompts,
- seasonal notices,
- story beats,
- contextual hints,
- and harmless commentary.

He may also occasionally deliver mischief when the active ruleset explicitly allows it.

---

# 3. Do not hardcode the guide identity into the game

The tutorial and handler system must not assume that the visible guide is always Smiling Shiba.

The default installation may ship with:

```yaml
id: smiling_shiba

identity:
  name: Shiba
  title: The Handler
  portrait: shiba_smug
  voice_style: snarky_corporate
```

But a user or mod should be able to supply another guide profile:

```yaml
id: neutral_guide

identity:
  name: Guide
  title: Assistant
  portrait: neutral_orb
  voice_style: concise

behavior:
  tutorials: true
  quests: true
  commentary: false
  mischief: off
```

The underlying daemon remains the same.

Only the presentation/content layer changes.

This keeps:

- tutorial logic reusable,
- accessibility options practical,
- community customization straightforward,
- and the mascot from becoming an architectural dependency.

---

# 4. Guide behavior is driven by documented hooks

Do **not** expose arbitrary internal game code to ordinary guide packs.

Instead, expose a stable, documented hook contract.

Examples:

```text
on_game_start
on_profile_created
on_first_match
on_match_started
on_turn_started
on_turn_ended
on_card_played
on_card_drawn
on_invalid_action
on_targeting_failed
on_player_died
on_boss_encountered
on_boss_defeated
on_quest_offered
on_quest_completed
on_card_unlocked
on_persona_unlocked
on_area_entered
on_world_event_started
on_session_idle
on_session_end
```

These names are illustrative.

The final hook vocabulary should be deliberately designed, documented, versioned, and stable.

Guide content subscribes to hooks.

It does not inspect private engine state directly.

---

# 5. Hooks expose context, not internals

A hook should expose a small documented context object.

Example:

```ts
type PlayerDiedHookContext = {
  playerId: string;
  personaId: string;
  cause: "combat" | "hazard" | "concede";
  totalDeaths: number;
  deathsThisSession: number;
  matchId?: string;
};
```

The guide package receives information that is intentionally part of the public guide API.

It must **not** receive arbitrary engine objects such as:

```text
game._privateState
server.room.internalWhatever
player.secretDeckOrder
databaseHandle
UI component references
```

This protects:

- encapsulation,
- hidden information,
- mod compatibility,
- engine refactors,
- mobile portability,
- and community stability.

---

# 6. Guide packs respond declaratively

Ordinary guide customization should be possible without JavaScript.

Example:

```yaml
guide_api: "1.0"

hooks:
  on_player_died:
    first_time:
      message: first_death
      portrait: smug

  on_invalid_action:
    after: 3
    message: targeting_hint

  on_boss_defeated:
    message_pool:
      - boss_win_01
      - boss_win_02
      - boss_win_03
```

Dialogue content:

```yaml
dialogue:
  first_death:
    - "Excellent. You've discovered mortality."
    - "That seemed avoidable."
    - "Let's call that exploratory testing."

  targeting_hint:
    - "That target remains illegal, despite your persistence."
```

The daemon evaluates the hook, conditions, cooldowns, and eligibility rules and emits an intervention.

---

# 7. Supported guide interventions

The guide system should have a constrained vocabulary of safe outputs.

Possible intervention types:

```text
show_message
show_toast
show_tip
highlight_control
open_help_panel
offer_quest
advance_tutorial
play_emote
play_animation
set_guide_mood
show_notification
request_rules_action
```

These are presentation or request operations.

They are **not direct state mutation APIs**.

If a guide wants to create an actual game effect, it must request a legal action from the authoritative rules engine.

Example:

```yaml
do:
  - request_rules_action:
      action: grant_modifier
      modifier: slippery_deck
      duration: one_match
```

The rules engine validates whether that action:

- exists,
- is legal,
- is permitted by the active ruleset,
- is permitted in the current mode,
- and can execute at the current timing point.

The guide daemon itself never bypasses rules authority.

---

# 8. Tutorials should react to player behavior

Avoid building the tutorial as one giant linear script.

Prefer contextual learning goals.

Example concept:

```yaml
concept: deck_inspection

learned_when:
  any:
    - action_used: inspect_deck
    - quest_completed: fisherman_intro
```

Possible intervention:

```yaml
hook: on_turn_ended

when:
  all:
    - concept_not_learned: deck_inspection
    - opportunities_missed: 3

do:
  - show_tip: deck_inspection_hint
```

If a player already discovered the mechanic independently, silently mark the concept learned and skip the tutorial.

This enables:

- experienced players to move faster,
- beginners to receive help when needed,
- tutorials to feel responsive,
- fewer forced modal interruptions.

---

# 9. Quests use the same event vocabulary

The Shiba Daemon may act as a quest giver, but quest progress should be based on normal semantic game events.

Example:

```yaml
id: suspiciously_specific_request

offered_by: smiling_shiba

objectives:
  - type: win_with_persona
    persona: fisherman
    count: 2

  - type: inspect_cards
    count: 10

rewards:
  xp: 800
  unlock:
    cosmetic: fisherman_blue_tie
```

The quest system should consume events such as:

```text
MATCH_WON
CARD_INSPECTED
PERSONA_USED
BOSS_DEFEATED
CARD_PLAYED
QUEST_COMPLETED
```

Do not couple quest progress to UI components.

---

# 10. Separate mandatory system notices from guide personality

Turning off Shiba must not remove essential application information.

Keep these separate:

```text
SYSTEM NOTICES
    mandatory technical/game-state messages

TUTORIAL SYSTEM
    optional instructional guidance

SHIBA DAEMON
    eligibility / scheduling / intervention coordinator

GUIDE PROFILE
    who appears and how that intervention is presented
```

Examples of mandatory system notices:

```text
Connection lost.
You must choose a legal target.
Save failed.
This match uses ruleset version 2027.03.2.
```

Those messages must remain available even when the guide is disabled.

---

# 11. Player customization and opt-out controls

The guide system should support independent preferences.

Example:

```text
Tutorials
- Contextual
- Full
- Off

Quest Prompts
- On
- Off

Commentary
- Frequent
- Normal
- Sparse
- Off

Mischief
- Normal
- Mild
- Off

Story Dialogue
- On
- Off
```

Do not make "disable tutorials" equivalent to "remove all guide content."

A player might want:

- Shiba quests but no tutorials,
- tutorials but no snark,
- story dialogue but no mischief,
- or a silent guide with icon-only hints.

Accessibility and player control should be intentional, not an afterthought.

---

# 12. The annoyance budget

Guide systems become unbearable when they interrupt too often.

The daemon should maintain an intervention budget.

Conceptually:

```yaml
interruption_budget:
  critical:
    unlimited_when_required: true

  major:
    max_per_match: 1

  minor:
    max_per_match: 3

  ambient:
    max_per_minute: 2

minimum_gap_seconds: 60
```

Possible intervention categories:

```text
CRITICAL
    required to proceed or avoid severe confusion

MAJOR
    quest, important tutorial, major story/system introduction

MINOR
    context hint, reaction, joke

AMBIENT
    small toast, portrait reaction, emote, icon, bark
```

The daemon should consider:

- priority,
- cooldown,
- session limits,
- recent interruptions,
- player preference,
- tutorial state,
- and eligibility.

The mascot should feel present.

He should not feel like Clippy with admin privileges.

---

# 13. Eligibility before selection

When a hook fires, many interventions may theoretically match.

Evaluate them in stages.

Conceptually:

```text
hook fires
   |
   v
find candidate interventions
   |
   v
evaluate conditions
   |
   v
remove disabled content
   |
   v
remove cooldown violations
   |
   v
apply player preferences
   |
   v
apply interruption budget
   |
   v
priority / weighting
   |
   v
choose intervention
```

Example definition:

```yaml
id: shiba_bad_targeting

hook: on_invalid_action

conditions:
  all:
    - invalid_action_type: target
    - repeated_count_at_least: 3

priority: medium

cooldown:
  session: once

presentation:
  type: toast
  portrait: shiba_concerned
  message: targeting_hint
```

---

# 14. Determinism and replay

Where guide behavior affects the deterministic game state, it must participate in the same reproducibility guarantees as the rules engine.

If guide behavior is purely cosmetic, strict replay identity may not always be necessary.

Separate:

```text
AUTHORITATIVE INTERVENTIONS
    gameplay effects / quests / rewards / rule-impacting actions

PRESENTATION INTERVENTIONS
    jokes / animations / flavor / portrait reactions
```

Authoritative interventions must be:

- ruleset-defined,
- deterministic where required,
- replayable,
- validated,
- and versioned.

If weighted/random selection affects authoritative state, use deterministic seeded RNG.

Never use `Math.random()` for authoritative guide behavior.

---

# 15. Mischief categories

Mischief should be intentionally categorized.

## Cosmetic mischief

Usually safe:

- strange dialogue,
- fake corporate memo,
- portrait changes,
- UI flourish,
- confetti,
- Shiba wearing blue on Pink Wednesday,
- harmless temporary label substitutions,
- smug animation,
- suspiciously timed notification.

## Gameplay mischief

Requires explicit ruleset support.

Examples:

- temporary modifier,
- surprise encounter,
- challenge rule,
- unusual reward,
- short-lived world condition.

Gameplay mischief must specify allowed modes.

Example:

```yaml
mischief:
  allowed_modes:
    - solo
    - custom

  prohibited_modes:
    - ranked
```

Official competitive play may still allow Shiba to talk trash.

It should not randomly give one player competitive advantage unless that behavior is formally part of the official ruleset.

---

# 16. Guide memory is explicit game data

The guide may need long-lived contextual memory, but it should be ordinary structured game data.

Example:

```json
{
  "guide": {
    "profile": "smiling_shiba",

    "seen": [
      "first_death",
      "first_boss",
      "deck_builder_intro"
    ],

    "cooldowns": {
      "bad_targeting": 4
    },

    "tutorials": {
      "deck_inspection": "learned",
      "graveyard": "unseen"
    }
  }
}
```

Potential optional personality-state dimensions:

```text
familiarity
respect
annoyance
chaos
trust
```

These should remain lightweight and purposeful.

Do not build a simulated consciousness because Shiba made a funny face.

---

# 17. Dialogue variation

Repeated dialogue should use pools and context tags.

Example:

```yaml
dialogue:
  player_died:
    tags:
      - smug
      - faux_supportive

    variants:
      - "That seemed avoidable."
      - "Fascinating approach."
      - "I have updated your performance review."
      - "Let's call that exploratory testing."
```

Variation selection may consider:

- recent lines,
- event context,
- guide mood,
- player history,
- cooldowns,
- persona,
- area,
- season,
- or ruleset.

Avoid repetition that makes the character annoying.

---

# 18. Guide packs are content packages

A guide pack may contain:

```text
guide-pack/
  manifest.yml
  profile.yml
  hooks.yml
  dialogue.yml
  tutorials/
  quests/
  portraits/
  animations/
  audio/
```

Example manifest:

```yaml
id: smiling_shiba
version: 1.3.0
guide_api: "^1.0"

profile: profile.yml
hooks: hooks.yml
dialogue: dialogue.yml
```

A community guide pack should not need to fork:

- rules-engine,
- desktop client,
- server,
- or official ruleset.

It consumes documented hooks.

---

# 19. Version the guide hook API

Community guide content may depend on hook names and context fields.

Therefore the guide hook contract must be versioned.

Example:

```yaml
guide_api: "1.2"
```

Avoid casually renaming:

```text
on_player_died
```

to:

```text
on_character_failure_event
```

after community packs exist.

If hooks must evolve:

- add new hooks,
- add optional context,
- deprecate old names,
- document migration,
- maintain compatibility where reasonable.

Stable seams matter more than cute naming changes.

---

# 20. Ordinary customization consumes hooks

This is the expected community path.

A creator may customize:

- identity,
- portrait,
- dialogue,
- tutorial responses,
- quest presentation,
- reaction behavior,
- frequency,
- allowed mischief,
- and hook mappings.

They do not need game source access for ordinary guide customization.

The contract is:

```text
game exposes hooks
guide pack consumes hooks
```

---

# 21. Advanced modding may add hooks

Because Smiling Shiba is open/moddable, advanced users may decide the existing guide API is insufficient.

They may modify their own build to expose a new hook such as:

```text
on_fisherman_fished_his_own_card_for_the_third_time
```

That is acceptable.

The distinction is:

```text
ORDINARY CUSTOMIZATION
    consume documented hooks

ADVANCED MODDING
    add new hooks or daemon behavior

ENGINE HACKING
    source is available; good luck, you beautiful lunatic
```

Do not make ordinary users fork the engine just to change the mascot.

---

# 22. Shiba Daemon should not be an LLM dependency

The guide system must function without generative AI.

Authoritative behavior should come from:

- hooks,
- conditions,
- structured dialogue,
- deterministic rules,
- quests,
- explicit state,
- and versioned content.

A future optional language model integration might assist with:

- flavor dialogue,
- recap generation,
- low-stakes banter,
- prose variation,
- or accessibility summaries.

It must not decide authoritative rewards, legal actions, match outcomes, or game rules.

Principle:

> AI may decorate. The engine decides.

---

# 23. Suggested package boundary

Future structure may resemble:

```text
packages/
  shiba-daemon/
    hooks/
    scheduler/
    eligibility/
    cooldowns/
    intervention-budget/
    memory/
    types/

  guide-schema/
    profile/
    hooks/
    dialogue/
    tutorials/
    quests/

  guide-runtime/
    content-loader/
    intervention-resolution/

apps/
  tools/
    guide-editor/
    hook-inspector/
    dialogue-preview/
```

Exact names are not sacred.

The separation is.

---

# 24. Tooling integration

Eventually, the tooling (`sht`) could gain guide-pack validation and authoring support.

Possible screen:

```text
GUIDE / SHIBA DAEMON

Guide Profile:
[ Smiling Shiba            v ]

Hook:
[ on_player_died           v ]

Conditions:
[x] first death
[x] not previously shown

Priority:
[ High                     v ]

Cooldown:
[ Once per profile         v ]

Intervention:
[ Message                  v ]

Portrait:
[ shiba_smug               v ]

Message:
+--------------------------------------+
| Excellent. You've discovered        |
| mortality.                           |
+--------------------------------------+

[ TEST HOOK ] [ PREVIEW ] [ VALIDATE ]
```

It should be possible to:

- simulate a hook,
- inspect its context schema,
- preview eligible responses,
- preview dialogue,
- test cooldowns,
- and validate guide packs.

The Guide API documentation should be generated or linked from the same hook registry that drives validation.

---

# 25. Suggested hook registration model

Hooks themselves should be documented definitions, not magic string constants spread through the codebase.

Conceptually:

```ts
defineGuideHook({
  id: "on_player_died",

  title: "Player Died",

  description:
    "Fires after an authoritative player-death event has resolved.",

  contextSchema: PlayerDiedHookContext,

  authoritative: true
});
```

The hook registry can then power:

- validation,
- documentation,
- Monaco autocomplete,
- editor help,
- compatibility checks,
- test fixtures,
- and community reference documentation.

This follows the same philosophy as the rules engine:

> If the system exposes a programmable concept, the system should also be able to document that concept.

---

# 26. Security and hidden information

Guide hooks must respect state projection and secrecy.

Never expose hidden information merely because a guide script might want to comment on it.

A client-side guide may only see the player's legal projection.

An authoritative server-side guide may see more state only when the active ruleset explicitly requires it.

Example:

A player-facing hook must not reveal:

```text
opponent hand identities
deck order
hidden traps
secret server selections
future RNG outcomes
```

unless those values are already legitimately revealed.

Guide flavor must never become an accidental side-channel.

---

# 27. Persistence boundaries

Persist only guide state that actually needs to survive.

Possible scopes:

```text
PER INTERVENTION
PER MATCH
PER SESSION
PER WORLD
PER PROFILE
PER ACCOUNT
```

Examples:

```text
"Have I shown this targeting hint?"
    -> profile/account

"Did I already make this joke this match?"
    -> match

"Is this quest currently active?"
    -> world/account depending on mode

"How many minor interruptions have occurred recently?"
    -> session/runtime
```

Do not throw every guide detail into permanent saves.

---

# 28. Official vs custom guide behavior

Local/custom play:

```text
player controls guide pack
player may add hooks in custom builds
player may enable weird gameplay mischief
player may replace Shiba entirely
```

Official online play:

```text
server controls authoritative guide actions
client may choose allowed presentation/profile alternatives
gameplay-affecting interventions come from official ruleset
competitive hidden information remains protected
```

This allows visual/personal customization without compromising server authority.

---

# 29. Implementation timing

Do not implement this subsystem during initial rules-engine foundation work beyond any tiny seams that are cheap and obvious.

Useful early preparation:

- semantic event vocabulary,
- clean event bus or event log,
- documented state transitions,
- stable quest/event concepts,
- ruleset versioning,
- state projection,
- schema infrastructure,
- content package loading.

These foundations make Shiba Daemon straightforward later.

Avoid premature work such as:

- complete guide UI,
- giant dialogue databases,
- elaborate relationship simulation,
- generative AI integration,
- full quest authoring tools,
- voice systems,
- or hundreds of hook definitions.

Document now.

Build when the core game can actually produce meaningful events.

---

# 30. Architectural rules

When this feature is implemented, reject changes that:

- hardcode Smiling Shiba into core game rules,
- allow guide packs arbitrary access to engine internals,
- allow guide code to mutate authoritative state directly,
- bypass rules validation for gameplay mischief,
- expose hidden information through hooks,
- make mandatory system notices dependent on the guide being enabled,
- require JavaScript for ordinary guide customization,
- create undocumented hook strings scattered through the code,
- introduce non-versioned hook contracts,
- use nondeterministic randomness for authoritative guide actions,
- or force players to endure optional commentary/tutorials they disabled.

---

# 31. Definition of success

A future implementation should make all of the following true:

> The Shiba Daemon is an event-driven background coordinator, not a hardcoded mascot controller.

> The visible guide identity is replaceable content.

> Tutorial behavior, quest behavior, dialogue, and reactions can be customized through documented hooks.

> Ordinary customizers do not need access to engine internals.

> Advanced open-source users may extend the hook surface in their own builds.

> Essential system messages remain functional even when the guide is disabled.

> Shiba can be funny, intrusive, ambitious, smug, helpful, and chaotic without becoming an architectural dependency.

> Gameplay-affecting mischief always goes through the authoritative rules engine.

> The hook API is documented and versioned.

> The guide system respects player preferences, annoyance limits, determinism, state projection, and hidden information.

> The default guide can remain gloriously Smiling Shiba while the underlying system stays generic enough for users to make it their own.

---

# Final note

Smiling Shiba should feel like he is always somewhere nearby:

watching,
judging,
recruiting,
assigning opportunities,
occasionally helping,
occasionally creating the problem he is helping with,
and smiling like none of this could possibly be his fault.

The architecture should make that personality **content**.

The daemon should make it **possible**.
