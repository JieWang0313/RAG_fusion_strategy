%%{init: {
  "theme": "base",
  "themeVariables": {
    "fontFamily": "Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, sans-serif",
    "primaryColor": "#E3F2FD",
    "primaryTextColor": "#0D47A1",
    "primaryBorderColor": "#90CAF9",
    "lineColor": "#B0BEC5"
  }
}}%%
flowchart TB
  %% ========== Shared Retrieval ==========
  Q[用户查询 / Query]:::q
  R[检索器<br/>BM25 + 向量检索<br/>RRF 融合 Top-K]:::retriever
  K[Top-K 文档]:::docs
  Q --> R --> K

  %% ========== Early Fusion ==========
  subgraph E[Early Fusion｜早期融合]
    direction TB
    C[拼接：Query + Top-K 上下文]:::early
    ENC[一次编码/理解<br/>Encoder/LLM]:::early
    DEC[解码生成答案]:::gen
    C --> ENC --> DEC
  end

  %% ========== Late Fusion ==========
  subgraph L[Late Fusion｜晚期融合]
    direction TB
    LOOP((对每个文档<br/>独立调用 LLM)):::late
    CANDS[候选答案 1…K]:::late
    AGG[结果聚合<br/>• 平均对数概率 (avg_log_prob)<br/>• 交叉编码器打分 (cross-encoder)<br/>• 多数投票 (majority)]:::agg
    LOOP --> CANDS --> AGG
    AGG --> ANS2[最终答案]:::gen
  end

  %% ========== Adaptive Fusion ==========
  subgraph A[Adaptive Fusion｜自适应融合]
    direction TB
    GATE[自适应门/权重<br/>(基于 cross-attention 或自评判器)]:::adaptive
    MIX[动态混合：<br/>“模型内知识” ⟷ “检索知识”]:::adaptive
    OUT[生成答案]:::gen
    GATE --> MIX --> OUT
  end

  %% ========== Wiring ==========
  K --> E
  K --> L
  K --> A
  Q -. 可选自评判/self-referee .-> GATE

  %% ========== Legend ==========
  subgraph Legend[图例 Legend]
    direction LR
    q1[输入/查询]:::q
    r1[检索/候选]:::retriever
    d1[文档/上下文]:::docs
    e1[融合与编码]:::early
    l1[独立生成/聚合]:::late
    a1[自适应控制]:::adaptive
    g1[最终生成]:::gen
  end

  %% ========== Styles ==========
  classDef q fill:#D1E9FF,stroke:#1E40AF,color:#0B1B3B,stroke-width:1.2px;
  classDef retriever fill:#E0F7FA,stroke:#00838F,color:#004D40,stroke-width:1.2px;
  classDef docs fill:#E8EAF6,stroke:#3949AB,color:#1A237E,stroke-width:1.2px;

  classDef early fill:#E3F2FD,stroke:#1976D2,color:#0D47A1,stroke-width:1.2px;
  classDef late fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20,stroke-width:1.2px;
  classDef agg fill:#C8E6C9,stroke:#1B5E20,color:#1B5E20,stroke-width:1.2px;
  classDef adaptive fill:#FFF3E0,stroke:#EF6C00,color:#E65100,stroke-width:1.2px;
  classDef gen fill:#F3E5F5,stroke:#6A1B9A,color:#4A148C,stroke-width:1.2px;

  style E fill:#F0F9FF,stroke:#93C5FD,stroke-width:1.5px,corner-radius:4px
  style L fill:#F1F8E9,stroke:#A5D6A7,stroke-width:1.5px,corner-radius:4px
  style A fill:#FFF8E1,stroke:#FFCC80,stroke-width:1.5px,corner-radius:4px
  style Legend fill:#F9FAFB,stroke:#CBD5E1,stroke-width:1px,corner-radius:4px
