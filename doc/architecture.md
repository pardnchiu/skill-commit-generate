# commit-generate - Architecture

> Back to [README](../README.md)

## Overview

```mermaid
graph TB
    User[User invokes /commit-generate] --> Skill[SKILL.md instructions]
    Skill --> Input[Input layer]
    Skill --> Detect[Multi-Topic Detection]
    Skill --> Upgrade[Tag Upgrade Signals]
    Skill --> Tags[Classification Tags]
    Skill --> Format[Output Format]
    Input --> Git[git diff --cached]
    Git --> Output[Bilingual commit message]
    Detect --> Output
    Upgrade --> Output
    Tags --> Output
    Format --> Output
```

## Module: Input

Reads and validates the staged diff. Aborts when nothing is staged; never falls back to the working tree.

```mermaid
graph TB
    subgraph Input
        A[Run git diff --cached] --> B{Output empty?}
        B -->|Yes| C[Emit error message]
        B -->|No| D[Forward diff to next stage]
    end
    C --> Stop[Stop]
    D --> Next[Multi-Topic Detection]
```

## Module: Multi-Topic Detection

Decides whether a single diff mixes unrelated intents and, if so, emits a split recommendation.

```mermaid
graph TB
    subgraph MultiTopic[Multi-Topic Detection]
        A[Receive diff] --> B{Touches 2+ primary tags?}
        A --> C{Spans 2+ unrelated modules?}
        A --> D{Contains 3+ unrelated topics?}
        B -->|Any true| E[Mark as multi-topic]
        C -->|Any true| E
        D -->|Any true| E
        B -->|All false| F[Mark as single-topic]
        C -->|All false| F
        D -->|All false| F
    end
    E --> SplitWarn[Emit split recommendation]
    F --> Upgrade[Tag Upgrade Scan]
    SplitWarn --> Upgrade
```

## Module: Tag Upgrade Signals

Scans signals top-down; any hit forces an upgrade and blocks downgrades to `feat` or `update`.

```mermaid
graph TB
    subgraph Upgrade[Tag Upgrade Signals]
        A[Receive diff] --> B{Breaking signals hit?}
        B -->|Yes| C[Force tag = breaking]
        B -->|No| D{Security signals hit?}
        D -->|Yes| E[Force tag = security]
        D -->|No| F[Match intent via tag priority]
    end
    C --> Out[Emit tag]
    E --> Out
    F --> Out
```

## Module: Classification Tags

Final tag resolves through a fixed priority order.

```mermaid
graph LR
    subgraph Priority[Tag priority]
        BREAKING --> FEAT --> FIX --> SECURITY --> UPDATE --> REFACTOR --> PERF --> OTHERS
    end
    OTHERS --> Pool[add / remove / style / doc / test / chore]
```

## Module: Output Format

Fixed two-line format: English subject on line 1, Traditional Chinese body on line 2.

```mermaid
graph TB
    subgraph Output[Output Format]
        Tag[Resolved tag] --> Line1[tag: English imperative subject]
        Tag --> Line2[tag: Traditional Chinese verb-first description]
        Line1 --> Final[Plain-text output]
        Line2 --> Final
    end
```

## Data Flow

```mermaid
sequenceDiagram
    participant User
    participant Skill as commit-generate
    participant Git
    User->>Skill: /commit-generate
    Skill->>Git: git diff --cached
    Git-->>Skill: staged diff
    alt Diff empty
        Skill-->>User: Error: run git add first
    else
        Skill->>Skill: Multi-Topic Detection
        alt Multi-topic
            Skill-->>User: Split recommendation
        end
        Skill->>Skill: Tag Upgrade Scan
        Skill->>Skill: Match Classification Tag
        Skill->>Skill: Apply Output Format
        Skill-->>User: Bilingual commit message
    end
```

## State Machine

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> ReadDiff: Trigger
    ReadDiff --> Empty: diff empty
    Empty --> [*]: Emit error
    ReadDiff --> Detect: diff present
    Detect --> MultiTopic: multi-topic hit
    Detect --> SingleTopic: single-topic
    MultiTopic --> Upgrade: after split hint
    SingleTopic --> Upgrade
    Upgrade --> BreakingTag: breaking hit
    Upgrade --> SecurityTag: security hit
    Upgrade --> DefaultTag: none hit
    BreakingTag --> Emit
    SecurityTag --> Emit
    DefaultTag --> Emit
    Emit --> [*]
```
