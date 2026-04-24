# commit-generate - 架構

> 返回 [README](./README.zh.md)

## Overview

```mermaid
graph TB
    User[使用者呼叫 /commit-generate] --> Skill[SKILL.md 指令集]
    Skill --> Input[Input 層]
    Skill --> Detect[Multi-Topic Detection]
    Skill --> Upgrade[Tag Upgrade Signals]
    Skill --> Tags[Classification Tags]
    Skill --> Format[Output Format]
    Input --> Git[git diff --cached]
    Git --> Output[雙語 commit message]
    Detect --> Output
    Upgrade --> Output
    Tags --> Output
    Format --> Output
```

## Module: Input

負責取得並驗證 staged diff。無 staged 時直接終止，不 fallback 至工作區。

```mermaid
graph TB
    subgraph Input
        A[執行 git diff --cached] --> B{輸出是否為空}
        B -->|是| C[輸出錯誤訊息]
        B -->|否| D[傳遞 diff 至後續階段]
    end
    C --> Stop[停止]
    D --> Next[Multi-Topic Detection]
```

## Module: Multi-Topic Detection

判斷單一 diff 是否混合多個不相關意圖，若命中則要求警示拆分。

```mermaid
graph TB
    subgraph MultiTopic[Multi-Topic Detection]
        A[接收 diff] --> B{觸及 2+ primary tag?}
        A --> C{橫跨 2+ 無關模組?}
        A --> D{包含 3+ 無關主題?}
        B -->|任一為真| E[標記為跨主題]
        C -->|任一為真| E
        D -->|任一為真| E
        B -->|皆否| F[標記為單一主題]
        C -->|皆否| F
        D -->|皆否| F
    end
    E --> SplitWarn[輸出拆分建議]
    F --> Upgrade[Tag Upgrade Scan]
    SplitWarn --> Upgrade
```

## Module: Tag Upgrade Signals

由上而下掃描訊號，命中即強制升級 Tag，禁止降級為 feat / update。

```mermaid
graph TB
    subgraph Upgrade[Tag Upgrade Signals]
        A[接收 diff] --> B{命中 Breaking Signals?}
        B -->|是| C[強制 tag = breaking]
        B -->|否| D{命中 Security Signals?}
        D -->|是| E[強制 tag = security]
        D -->|否| F[依 Tag 優先序匹配意圖]
    end
    C --> Out[輸出 Tag]
    E --> Out
    F --> Out
```

## Module: Classification Tags

最終 Tag 選擇依固定優先序決議。

```mermaid
graph LR
    subgraph Priority[Tag 優先序]
        BREAKING --> FEAT --> FIX --> SECURITY --> UPDATE --> REFACTOR --> PERF --> OTHERS
    end
    OTHERS --> Pool[add / remove / style / doc / test / chore]
```

## Module: Output Format

固定雙行格式，第一行英文 subject、第二行繁體中文 body。

```mermaid
graph TB
    subgraph Output[Output Format]
        Tag[決議 Tag] --> Line1[tag: English imperative subject]
        Tag --> Line2[tag: 繁體中文動詞開頭描述]
        Line1 --> Final[純文字輸出]
        Line2 --> Final
    end
```

## Data Flow

```mermaid
sequenceDiagram
    participant User as 使用者
    participant Skill as commit-generate
    participant Git as Git
    User->>Skill: /commit-generate
    Skill->>Git: git diff --cached
    Git-->>Skill: staged diff
    alt Diff 為空
        Skill-->>User: 錯誤：請先 git add
    else
        Skill->>Skill: Multi-Topic Detection
        alt 跨主題
            Skill-->>User: 警示拆分建議
        end
        Skill->>Skill: Tag Upgrade Scan
        Skill->>Skill: 匹配 Classification Tag
        Skill->>Skill: 套用 Output Format
        Skill-->>User: 雙語 commit message
    end
```

## State Machine

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> ReadDiff: 觸發
    ReadDiff --> Empty: diff 為空
    Empty --> [*]: 輸出錯誤
    ReadDiff --> Detect: diff 存在
    Detect --> MultiTopic: 跨主題命中
    Detect --> SingleTopic: 單一主題
    MultiTopic --> Upgrade: 輸出拆分建議後繼續
    SingleTopic --> Upgrade
    Upgrade --> BreakingTag: Breaking 命中
    Upgrade --> SecurityTag: Security 命中
    Upgrade --> DefaultTag: 皆未命中
    BreakingTag --> Emit
    SecurityTag --> Emit
    DefaultTag --> Emit
    Emit --> [*]
```
