# RAG Fusion Strategy

This repository describes the workflow of RAG fusion strategy.

## Workflow Diagram

```mermaid
graph TD
    A[User Query] --> B[Retriever]
    B --> C{Fusion Strategy?}
    C -->|Early| D[Concatenate Documents]
    C -->|Late| E[Ranked Result Fusion]
    C -->|Adaptive| F[Weighted Attention]
    D --> G[Generator]
    E --> G
    F --> G
    G --> H[Final Answer]
