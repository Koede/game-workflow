# Gamemaster Team: Flowchart

Render in any Markdown viewer that supports Mermaid (GitHub, Obsidian, VS Code with a Mermaid extension).

Lanes are group chats that their bots share. Tools belong to the bots, not the lanes. Full SOULs, tool lists, and the roster are in `souls/README.md`.

## Main flow

```mermaid
flowchart TD
    USER([User game concept]) --> GM[Gamemaster]
    GM --> DEPTH{Dream depth?}

    DEPTH -->|Easy task| DAY[Daydreaming: 3 ideas, light research]
    DEPTH -->|Default| DRM[Dreaming: 10 ideas, deeper research]
    DEPTH -->|Requested, dreaming stalled, or Gamemaster judges the prompt complex enough| DIV[Divining: wide open fanout]

    DIV --> HOUR{Task at 1 hour cap?}
    HOUR -->|Yes, more needed| DIV2[Start new 1-hour research task]
    DIV2 --> HOUR
    HOUR -->|No, done| POOL

    DAY --> POOL[Idea pool: tagged original or inspired, extras marked, no pruning]
    DRM --> POOL

    POOL --> CRY[Crystallizer: score feasibility, cut unbuildable ideas]
    CRY --> SCOPE[Final agreed scope: core scope + extras]
    CRY -.->|cut list kept, not passed on| GM

    SCOPE -->|only this reaches the planner| PLAN[Planner: propose 2-3 candidate plans]
    PLAN <-->|delegates system design, gets docs/systems specs| WORLDL
    PLAN --> SEL[Select best plan, weight extras, split by lane]

    SEL --> IMP[Implementer: hub in every lane chat, owns shared asset spec]
    IMP <-->|TASK out, DONE and QUESTION back| CODEL
    IMP <--> ARTL
    IMP <--> WORLDL
    IMP <--> AUDL
    IMP <--> UIL
    IMP <--> QAL

    IMP -->|small parallel coding jobs via delegate_task| KIDS[Coder children: coder profile route, up to max_concurrent_children]
    KIDS -->|results as final answers| IMP

    subgraph CODEL["code-lane chat"]
        subgraph W0["Wave 0: publishes core interfaces first"]
            ENG["engine-coder"]
        end
        subgraph W1["Wave 1: work in parallel against the stubs"]
            CODER["coder (many generic workhorses)"]
            MECH["gameplay-mechanics"]
            RND["render-coder"]
            WGC["worldgen-coder"]
            PHY["physics-coder"]
            ENT["entity-ai-coder"]
            TCH["tech-coder"]
            MGC["magic-coder"]
            CFC["crafting-coder"]
            UIC["ui-coder"]
            AUC["audio-coder"]
        end
        W0 -->|docs/api stubs| W1
    end

    subgraph ARTL["art-lane chat"]
        TEX["texture-artist"]
        MOD["model-artist"]
        ANIM["animator"]
        VFXA["vfx-artist"]
        SHD["shader-artist"]
    end

    subgraph WORLDL["world-lane chat"]
        WDES["world-designer"]
        CRAFT["crafting-manager"]
        TECH["tech-system-expert"]
        MAGIC["magic-system-expert"]
    end

    subgraph AUDL["audio-lane chat"]
        SFX["sfx-designer"]
        MUS["music-composer"]
    end

    subgraph UIL["ui-lane chat"]
        UIX["ui-ux-designer"]
    end

    subgraph QAL["quality-lane chat"]
        INTEG["integrator"]
        TESTA["test-auditor"]
    end

    MECH -.->|also a member| WORLDL
    WGC -.->|also a member, talks to world-designer| WORLDL
    TCH -.->|also a member, talks to tech-system-expert| WORLDL
    MGC -.->|also a member, talks to magic-system-expert| WORLDL
    CFC -.->|also a member, talks to crafting-manager| WORLDL
    RND -.->|also a member, talks to shader and vfx artists| ARTL
    UIC -.->|also a member, talks to ui-ux-designer| UIL
    AUC -.->|also a member, talks to sound designers| AUDL
    REV -.->|also a member, final gate| QAL

    IMP --> MERGE[Merge outputs into one build]

    MERGE --> REV[Reviewer: code review + playtest]
    REV --> VERDICT{Matches dreamer intent?}

    VERDICT -->|Misaligned| NOTIFY[Notify planner and Gamemaster of misunderstanding]
    NOTIFY --> CRY

    VERDICT -->|Aligned| POL[Polisher: tune feel, optimize assets and performance]
    POL --> SAT{User satisfied?}

    SAT -->|No| GM
    SAT -->|Yes| DONE([Deliver final build + README])
```

## Notes

- A lane is a group chat. Each bot in a lane is shown as its own node. Bots in a lane talk to each other there; each bot keeps its own tools.
- The Implementer is a member of every lane chat, shown as one edge per lane. The Reviewer also sits in the quality-lane chat.
- The code lane has an engine-coder that publishes core interfaces first (Wave 0). Then the specialist coders and the generic coder workhorses work in parallel against those stubs (Wave 1).
- Specialist coders that implement another lane's spec also sit in that lane's chat (dotted edges), so they can ask the spec's author directly. Interface and asset-spec changes still go through the Implementer.
- Cross-lane questions are posted to the Implementer in the asker's own lane chat. The Implementer relays them and logs the relay.
- The Planner delegates design of the technology system, magic system, crafting, and world generation to the experts in #world-lane. It still reads only the crystallizer's final scope from upstream.
- Integration and Test/Auditor work on outputs routed to them by the Implementer in #quality-lane. The Reviewer stays the single final gate.
- Coder children cannot use chat, memory, or scheduling tools, so they return results as their final answer.
- The only backward path is the misaligned verdict, which restarts at the Crystallizer rather than the Dreamer.
- The Gamemaster reports progress between divining tasks so the user can stop or redirect.
