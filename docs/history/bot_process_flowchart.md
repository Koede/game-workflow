# Bot Process Flowchart

A general process flow for a bot, from trigger to response. Each diamond is a decision point. Render this file in any Markdown viewer that supports Mermaid (GitHub, Obsidian, VS Code with a Mermaid extension, and most wiki tools).

## Main flow

```mermaid
flowchart TD
    START([Trigger received]) --> SRC{Source type?}
    SRC -->|User message| INTAKE[Normalize input]
    SRC -->|Scheduled job| INTAKE
    SRC -->|Webhook or event| INTAKE

    INTAKE --> VALID{Input valid?}
    VALID -->|No| REJECT[Log and send error reply]
    REJECT --> END1([End])
    VALID -->|Yes| AUTH{Authorized?}
    AUTH -->|No| DENY[Refuse and log attempt]
    DENY --> END2([End])
    AUTH -->|Yes| LOAD[Load context: session, memory, config]

    LOAD --> PARSE[Parse intent and extract entities]
    PARSE --> CLARIFY{Enough information?}
    CLARIFY -->|No| ASK[Ask one clarifying question]
    ASK --> WAITUSER([Wait for reply])
    WAITUSER --> INTAKE
    CLARIFY -->|Yes| PLAN[Plan steps]

    PLAN --> TOOLNEEDED{Tool call needed?}
    TOOLNEEDED -->|No| COMPOSE[Compose response]
    TOOLNEEDED -->|Yes| PERM{Action allowed?}
    PERM -->|No, needs approval| APPROVE[Request user approval]
    APPROVE --> APPROVED{Approved?}
    APPROVED -->|No| COMPOSE
    APPROVED -->|Yes| EXEC
    PERM -->|Yes| EXEC[Execute tool]

    EXEC --> RESULT{Tool succeeded?}
    RESULT -->|Yes| MORE{More steps?}
    RESULT -->|No| RETRY{Retries left?}
    RETRY -->|Yes| EXEC
    RETRY -->|No| FALLBACK[Use fallback or report failure]
    FALLBACK --> COMPOSE
    MORE -->|Yes| PLAN
    MORE -->|No| COMPOSE

    COMPOSE --> CHECK{Output passes checks?}
    CHECK -->|No| PLAN
    CHECK -->|Yes| SEND[Send response]
    SEND --> SAVE[Write memory and logs]
    SAVE --> HANDOFF{Hand off to another agent?}
    HANDOFF -->|Yes| NEXT[Pass state to next agent]
    NEXT --> END3([End])
    HANDOFF -->|No| END4([End])
```

## Error and escalation path

```mermaid
flowchart LR
    ERR([Unhandled error]) --> CAUGHT{Caught by handler?}
    CAUGHT -->|Yes| LOGERR[Log with trace ID]
    CAUGHT -->|No| CRASH[Supervisor restarts bot]
    CRASH --> LOGERR
    LOGERR --> SEV{Severity high?}
    SEV -->|No| NOTIFYLOW[Note in next digest]
    SEV -->|Yes| NOTIFYHIGH[Alert owner]
    NOTIFYHIGH --> SAFE[Enter safe mode: read-only]
```

## Process stages

| Stage | Purpose | Typical output |
|---|---|---|
| Intake | Normalize and validate input | Clean request object |
| Authorization | Confirm the sender may ask for this | Allow or deny |
| Context load | Gather session, memory, and config | Working context |
| Intent parse | Decide what the user wants | Task and entities |
| Planning | Break the task into steps | Ordered step list |
| Execution | Run tools with retries and fallbacks | Tool results |
| Quality check | Verify output before sending | Pass or revise |
| Delivery | Send reply and persist state | Reply plus log entry |
| Handoff | Pass work to another bot if needed | Updated shared state |

## Notes

- Every run gets a trace ID so logs from each stage can be joined later.
- Actions that change external state (sending messages, deleting data, spending money) always pass through the approval branch.
- The quality check loops back to planning, so a bad draft gets revised rather than sent.
