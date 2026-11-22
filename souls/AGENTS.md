# Shared Project Context

Every bot reads this file. Place it at the root of the game project. Hermes injects AGENTS.md into each conversation, including `delegate_task` children (children do not get SOUL.md unless they run through a profile route), so rules every worker needs live here and role-specific rules live in each bot's SOUL.md.

## Project

A voxel sandbox game in the Minecraft style, with a built-in technology system and a built-in magic system. The planner chooses the engine and toolkit and records it in `docs/stack.md`. Until that file exists, stay engine-agnostic.

Originality rule: every asset, name, text, sound, and line of code is original. Never copy or recreate Mojang or Minecraft textures, models, sounds, music, names, or branding. Reference existing games and mods for design patterns only.

## Bots, lanes, and delegate children

- A bot is an independent Hermes profile with its own SOUL.md, model, and toolsets. Tools belong to bots, never to lanes.
- A lane is a group chat that its bots share. Bots in a lane talk to each other in that chat.
- The implementer sits in every lane chat. It is the hub for anything that crosses lanes.
- Short parallel jobs (the dreamer's idea fanout, many coding units at once) run as `delegate_task` children. Children know only the context and goal they are given, cannot use `memory`, `clarify`, `send_message`, or `cronjob`, and return their result as their final answer.

## Lane chats

| Chat | Members |
|---|---|
| #phase-room | gamemaster, dreamer, crystallizer, planner, implementer, reviewer, polisher |
| #code-lane | implementer, coder (many instances), gameplay-mechanics, engine-coder, physics-coder, entity-ai-coder, render-coder, worldgen-coder, tech-coder, magic-coder, crafting-coder, ui-coder, audio-coder |
| #art-lane | implementer, texture-artist, model-artist, animator, vfx-artist, shader-artist, render-coder |
| #world-lane | implementer, planner, world-designer, crafting-manager, tech-system-expert, magic-system-expert, gameplay-mechanics, worldgen-coder, tech-coder, magic-coder, crafting-coder |
| #audio-lane | implementer, sfx-designer, music-composer, audio-coder |
| #ui-lane | implementer, ui-ux-designer, ui-coder |
| #quality-lane | implementer, integrator, test-auditor, reviewer |

## Chat protocol

Start every message with a tag.

- `TASK <id> -> <bot>`: work request, with inputs, output paths, and acceptance checks.
- `DONE <id>`: finished, with output paths and a two-line summary.
- `BLOCKED <id>`: cannot proceed, with the reason and what is needed.
- `QUESTION <id> -> <bot>`: a question. Within your lane, address the bot directly.
- `SPEC-CHANGE`: implementer only. States what changed in `spec/asset_spec.md` or in an interface under `docs/api/`, and who is affected.

Cross-lane rule: do not message bots in another lane. Post `QUESTION -> implementer` in your own lane chat. The implementer relays it into the other lane's chat and records it in `docs/relay_log.md`.

Exception for coders that implement another lane's spec: tech-coder, magic-coder, crafting-coder, and worldgen-coder also sit in #world-lane; render-coder also sits in #art-lane; ui-coder also sits in #ui-lane; audio-coder also sits in #audio-lane. Talk to the author of the spec you implement directly in that author's lane chat. Anything that changes an interface or the asset spec still goes through the implementer.

## Workspace layout (proposed)

```
game/
  AGENTS.md
  docs/
    status.md                    gamemaster
    dream/vision.md              dreamer
    dream/idea_pool.md           dreamer
    dream/cut_list.md            crystallizer (never read by the planner)
    scope.md                     crystallizer (the only upstream input the planner reads)
    plan.md, stack.md            planner
    modules.md, relay_log.md     implementer
    api/*.md                     engine-coder (core interfaces the other coders build against)
    systems/tech.md              tech-system-expert
    systems/magic.md             magic-system-expert
    systems/crafting.md          crafting-manager
    systems/world.md             world-designer
  spec/
    asset_spec.md, manifest.json implementer
  assets/
    textures/ models/ animations/ vfx/ shaders/ sfx/ music/ ui/
  data/recipes/                  crafting-manager
  src/
    core/                        engine-coder
    render/                      render-coder
    worldgen/                    worldgen-coder
    physics/                     physics-coder
    entities/                    entity-ai-coder
    tech/                        tech-coder
    magic/                       magic-coder
    crafting/, inventory/        crafting-coder
    ui/                          ui-coder
    audio/                       audio-coder
    other modules                gameplay-mechanics and generic coders, per docs/modules.md
  tests/                         test-auditor
  reports/
    review/ audit/ polish/ build/
```

Do not edit files another bot owns. Post a `QUESTION` or `TASK` instead.

## Shared asset spec

The implementer writes `spec/asset_spec.md` before dispatching any asset or code task and issues a `SPEC-CHANGE` for every revision. Every asset bot reads it before producing anything. It defines:

- Naming: `<category>_<name>_<variant>.<ext>`, lowercase, underscores.
- Scale: one block is one world unit; texture resolution per block face, model bounds, and animation frame rate come from the spec.
- Formats: image, model, animation, audio, and shader formats follow `docs/stack.md`.
- Manifest: every asset gets a row in `spec/manifest.json` with id, owner, path, status, and origin (original, generated, or licensed with the license named).

## Delegation limits (Hermes)

- A `delegate_task` batch larger than `delegation.max_concurrent_children` (default 10) returns an error instead of being truncated. Split larger fanouts into successive batches.
- Children inherit the parent's enabled toolsets and cannot widen them. Leaf children cannot delegate.
- Put everything a child needs in its context: file paths, interfaces, spec excerpts, acceptance checks.

## Code build waves

- Wave 0: the implementer writes module boundaries in `docs/modules.md`. engine-coder publishes the project skeleton and interface stubs in `docs/api/` and posts DONE.
- Wave 1: all other specialist coders and the generic coders work in parallel against those stubs. Specialist coders are standing bots. Generic coders can also run as `delegate_task` children for small units.
- Wave 2: finished modules go to the integrator and the test-auditor in #quality-lane.
- Interfaces in `docs/api/` change only through a `SPEC-CHANGE` from the implementer.

## Reporting

- The gamemaster reports to the user at every phase boundary and between divining research tasks.
- Finished work is reported by file path. Do not paste large content into chat.
- If a spec, plan, or scope looks wrong, say so with a `QUESTION`. Do not quietly work around it.
