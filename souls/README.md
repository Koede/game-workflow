# Gamemaker Team: SOUL Files for Botmaker

23 bots for a Hermes client, one `SOUL.md` per folder, plus a shared `AGENTS.md` for the game project root. This replaces `soul_drafts.md`.

Project: a voxel sandbox game in the Minecraft style with a built-in technology system and a built-in magic system. All assets are original.

## How the pieces fit

- **Bots own tools.** Each bot is a Hermes profile with its own SOUL.md, model, and toolsets. The `tools:` block in each SOUL lists the toolsets it needs and which tool to use for which task. Lanes have no tools.
- **Lanes are group chats.** The bots in a lane share one chat. The implementer is in every lane chat.
- **Cross-lane traffic goes through the implementer.** A bot posts `QUESTION -> implementer` in its own lane chat; the implementer relays it and logs it in `docs/relay_log.md`. This keeps the shared asset spec in one place.
- **Quick parallel jobs are delegate children.** The dreamer's idea fanout and the implementer's coding batches use `delegate_task`. Children cannot use `memory`, `clarify`, `send_message`, or `cronjob`, so anything that needs a chat or persistent memory is a standing bot, not a child.
- **Technology and magic are delegation experts.** The planner asks `tech-system-expert` and `magic-system-expert` (plus `crafting-manager` and `world-designer`) in #world-lane to design their systems. Their `docs/systems/*.md` feed the plan. They do not write engine code.

## Flow

dream, crystallize, plan (planner consults #world-lane experts), implement (implementer dispatches to lanes), review, polish. A misaligned review restarts at crystallize. The planner reads only `docs/scope.md`.

## Lane chats

| Chat | Members |
|---|---|
| #phase-room | gamemaster, dreamer, crystallizer, planner, implementer, reviewer, polisher |
| #code-lane | implementer, coder (many instances), gameplay-mechanics |
| #art-lane | implementer, texture-artist, model-artist, animator, vfx-artist, shader-artist |
| #world-lane | implementer, planner, world-designer, crafting-manager, tech-system-expert, magic-system-expert, gameplay-mechanics |
| #audio-lane | implementer, sfx-designer, music-composer |
| #ui-lane | implementer, ui-ux-designer |
| #quality-lane | implementer, integrator, test-auditor, reviewer |

## Roster and tool matrix

Generated from the SOUL files.

| Bot | Chats | Toolsets |
|---|---|---|
| animator | #art-lane | terminal, code_execution, file, vision, browser, todo, messaging |
| coder | #code-lane | file, terminal, code_execution, web, todo, messaging |
| crafting-manager | #world-lane | file, code_execution, terminal, web, todo, messaging |
| crystallizer | #phase-room | file, web, code_execution, todo, messaging |
| dreamer | #phase-room | web, browser, vision, image_gen, delegation, file, terminal, todo, messaging |
| gamemaster | #phase-room | messaging, memory, session_search, todo, clarify, cronjob, file, terminal |
| gameplay-mechanics | #code-lane, #world-lane | file, terminal, code_execution, web, todo, messaging |
| implementer | #phase-room, #code-lane, #art-lane, #world-lane, #audio-lane, #ui-lane, #quality-lane | messaging, delegation, memory, file, terminal, todo |
| integrator | #quality-lane | terminal, file, code_execution, browser, vision, web, memory, todo, messaging |
| magic-system-expert | #world-lane | file, code_execution, terminal, web, todo, messaging |
| model-artist | #art-lane | terminal, code_execution, file, vision, image_gen, todo, messaging |
| music-composer | #audio-lane | terminal, code_execution, file, vision, web, todo, messaging |
| planner | #phase-room, #world-lane | web, file, terminal, code_execution, delegation, todo, messaging |
| polisher | #phase-room | file, terminal, code_execution, browser, vision, todo, messaging |
| reviewer | #phase-room, #quality-lane | file, terminal, code_execution, browser, vision, delegation, todo, messaging |
| sfx-designer | #audio-lane | terminal, code_execution, file, vision, web, todo, messaging |
| shader-artist | #art-lane | file, terminal, code_execution, browser, vision, web, todo, messaging |
| tech-system-expert | #world-lane | file, code_execution, terminal, web, todo, messaging |
| test-auditor | #quality-lane | terminal, code_execution, file, browser, vision, delegation, todo, messaging |
| texture-artist | #art-lane | image_gen, vision, terminal, file, web, todo, messaging |
| ui-ux-designer | #ui-lane | image_gen, vision, browser, file, terminal, code_execution, todo, messaging |
| vfx-artist | #art-lane | image_gen, code_execution, terminal, file, vision, browser, todo, messaging |
| world-designer | #world-lane | code_execution, terminal, file, vision, web, todo, messaging |

## Setup checklist for botmaker

1. Create one Hermes profile per folder, named exactly as the folder, and install that folder's `SOUL.md` as the profile's SOUL.md.
2. Enable only the toolsets listed in that SOUL's `toolsets:` line. No bot has `tts` enabled; `text_to_speech` is deliberately off for the audio bots.
3. Put `AGENTS.md` at the root of the game project.
4. Create the seven lane chats and add the members from the table above.
5. Enable the `advanced-delegation` plugin with `delegation.backend: profile_routes` and register one route, `coder`, pointing at the coder profile, so coding children run with the coder's persona and toolsets.
6. Set `delegation.max_concurrent_children` to the number of parallel workers you want. Default is 10, and larger batches error rather than truncate. Divining fanouts and large coding batches should be split into successive batches.
7. Install any skills each bot needs under that bot only.
8. In the client, check each profile's toolset list against its SOUL. The tool-to-toolset mapping here comes from the Hermes docs and was partly inferred, so confirm it before relying on it.

## Assumptions to confirm

- Lane chats are whatever shared channel your Hermes setup uses (your rooms system, or messaging-gateway channels). The Hermes docs I read do not describe built-in group rooms, so the SOULs refer to chats by name and use `send_message` to post.
- Cross-lane questions go through the implementer. If you would rather let lanes talk to each other directly, change the cross-lane rule in AGENTS.md and the implementer's relay duty.
- `#ui-lane` has two members. It is kept as its own chat so it can grow.
- `coder` is one SOUL used for many instances. Add instances to #code-lane as `coder-1`, `coder-2`, and so on, or rely on `delegate_task` children for short jobs.
- No built-in tool generates sound effects, music, or 3D models. Those bots synthesize and process in code with command-line tools, and verify by spectrogram or rendered preview since they cannot listen or look directly. Tools such as sox, fluidsynth, ffmpeg, and Blender must be installed where the bots run.
- Engine and toolkit are not fixed. The planner chooses and records it in `docs/stack.md`.
