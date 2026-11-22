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
    PLAN <-->|delegates system design, gets docs/systems specs| WORLDCHAT
    PLAN --> SEL[Select best plan, weight extras, split by lane]

    SEL --> IMP[Implementer: hub in every lane chat, owns shared asset spec]
    IMP <-->|TASK out, DONE and QUESTION back| CHATS

    IMP -->|small parallel coding jobs via delegate_task| KIDS[Coder children: coder profile route, up to max_concurrent_children]
    KIDS -->|results as final answers| IMP

    subgraph CHATS["Lane group chats: bots talk inside a lane, cross-lane goes through the Implementer"]
        direction LR
        CODECHAT["code-lane chat: coder instances, gameplay-mechanics"]
        ARTCHAT["art-lane chat: texture, model, animator, vfx, shader"]
        WORLDCHAT["world-lane chat: world-designer, crafting-manager, tech-system-expert, magic-system-expert, gameplay-mechanics"]
        AUDCHAT["audio-lane chat: sfx-designer, music-composer"]
        UICHAT["ui-lane chat: ui-ux-designer"]
        QACHAT["quality-lane chat: integrator, test-auditor, reviewer"]
    end

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

- A lane is a group chat. Bots in a lane talk to each other there; each bot keeps its own tools.
- Cross-lane questions are posted to the Implementer in the asker's own lane chat. The Implementer relays them and logs the relay.
- The Planner delegates design of the technology system, magic system, crafting, and world generation to the experts in #world-lane. It still reads only the crystallizer's final scope from upstream.
- Integration and Test/Auditor work on outputs routed to them by the Implementer in #quality-lane. The Reviewer stays the single final gate.
- Coder children cannot use chat, memory, or scheduling tools, so they return results as their final answer.
- The only backward path is the misaligned verdict, which restarts at the Crystallizer rather than the Dreamer.
- The Gamemaster reports progress between divining tasks so the user can stop or redirect.
