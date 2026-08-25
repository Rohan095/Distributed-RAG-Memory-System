# Distributed RAG & Memory System

A distributed Retrieval-Augmented Generation (RAG) system that combines **knowledge retrieval, semantic user memory, conversational history, asynchronous retrieval, and reranking** to generate context-aware and personalized responses.

The project uses multiple specialized storage and retrieval systems — **Qdrant, Weaviate, and Redis** — where each component handles a different type of context.

---

## Overview

Traditional RAG systems primarily retrieve information from a knowledge base. This project extends that architecture by introducing a **persistent semantic memory layer** that allows the system to retain useful information about users across conversations.

The system combines:

* **External knowledge** → Qdrant
* **Semantic user memory** → Weaviate
* **Conversation history** → Redis
* **Vector similarity search** → Initial retrieval
* **CrossEncoder reranking** → Improved relevance
* **Asynchronous retrieval** → Parallel retrieval from multiple sources
* **LLM generation** → Context-aware responses

The overall objective is to provide responses that are both **knowledge-grounded and personalized**.

---

## Architecture

```text
                         User Query
                             │
                             ▼
                  Conversation History
                         (Redis)
                             │
                             ▼
                  History-Aware Query
                       Rewriting
                             │
                             ▼
                ┌────────────────────────┐
                │   Parallel Retrieval   │
                └───────────┬────────────┘
                            │
                ┌───────────┴───────────┐
                ▼                       ▼
        ┌──────────────┐        ┌──────────────┐
        │   Weaviate   │        │    Qdrant    │
        │ User Memory  │        │  Knowledge   │
        └──────┬───────┘        └──────┬───────┘
               │                       │
               └───────────┬───────────┘
                           ▼
                  Retrieved Documents
                           │
                           ▼
                    CrossEncoder
                     Reranking
                           │
                           ▼
                     Top-K Context
                           │
                           ▼
                     LLM Generation
                           │
                           ▼
                    Final Response
                           │
                           ▼
                  Semantic Memory
                     Extraction
                           │
                           ▼
                       Weaviate
```

---

## Core Components

### Qdrant — Knowledge Retrieval

Qdrant is used as the vector database for the external knowledge base.

It handles:

* Document embeddings
* Semantic similarity search
* Knowledge retrieval
* Retrieval of relevant information for user queries

---

### Weaviate — Semantic Memory

Weaviate stores long-term semantic information extracted from conversations.

The system can identify memories such as:

* **Preferences**
* **Skills**
* **Interests**
* **Goals**
* **Projects**

For example:

```text
User prefers C++ for programming.
User is learning distributed systems.
User is interested in AI engineering.
```

These memories can later be retrieved and used to personalize responses.

---

### Redis — Conversational Memory

Redis stores short-term conversational history.

This enables the system to:

* Maintain conversation context
* Retrieve previous messages
* Rewrite context-dependent queries
* Support multi-turn conversations

---

## Retrieval Pipeline

The system uses a multi-stage retrieval pipeline.

### 1. Query Processing

The incoming query is combined with relevant conversation history.

If the query depends on previous messages, it can be transformed into a standalone query.

### 2. Parallel Retrieval

Memory and knowledge retrieval can run asynchronously.

```text
                 Query
                   │
          ┌────────┴────────┐
          ▼                 ▼
      Weaviate           Qdrant
      Memory             Knowledge
      Search             Search
          │                 │
          └────────┬────────┘
                   ▼
             Candidate Set
```

This avoids unnecessarily waiting for each retrieval operation sequentially.

### 3. Reranking

The retrieved candidates are passed through a CrossEncoder reranker.

```text
Vector Search
     ↓
Candidate Documents
     ↓
CrossEncoder
     ↓
Relevance Scores
     ↓
Top-K Documents
```

This provides an additional relevance-ranking stage before the context is sent to the LLM.

---

## Semantic Memory

One of the main features of the project is **semantic long-term memory**.

Instead of storing every conversation message as memory, the system extracts information that may remain useful across future conversations.

### Memory Extraction

The LLM identifies potentially useful information from conversations and categorizes it into structured memory.

```text
Conversation
     ↓
Memory Extraction
     ↓
Structured Memory
     ↓
Similarity Check
     ↓
Deduplication
     ↓
Weaviate
```

### Memory Deduplication

Before adding a new memory, the system checks existing memories using semantic similarity.

If a sufficiently similar memory already exists, the duplicate is not stored.

This helps prevent memory accumulation from repeatedly storing the same information.

---

## Personalized RAG

The final context can contain information from multiple sources:

```text
Conversation History
        +
User Semantic Memory
        +
External Knowledge
        ↓
     LLM Context
        ↓
  Personalized Response
```

This allows the system to answer questions using both **retrieved knowledge and user-specific context**.

---

## Technology Stack

| Component           | Technology                           |
| ------------------- | ------------------------------------ |
| Language            | Python                               |
| RAG Framework       | LangChain                            |
| LLM                 | Llama 3.3 70B                        |
| Memory Extraction   | Llama 3.1 8B                         |
| Knowledge Vector DB | Qdrant                               |
| Memory Vector DB    | Weaviate                             |
| Conversation Store  | Redis                                |
| Embeddings          | Hugging Face / Sentence Transformers |
| Reranker            | CrossEncoder                         |
| Async Processing    | Python `asyncio`                     |
| LLM Provider        | Groq                                 |

---

## Models

### Generation

```text
llama-3.3-70b-versatile
```

Used for conversational response generation and query processing.

### Memory Extraction

```text
llama-3.1-8b-instant
```

Used to extract structured semantic memories.

### Embeddings

Memory embeddings:

```text
sentence-transformers/all-MiniLM-L6-v2
```

Knowledge embeddings:

```text
BAAI/bge-small-en-v1.5
```

### Reranker

```text
cross-encoder/ms-marco-MiniLM-L-6-v2
```

---

## Project Evolution

### Phase 1 — Core RAG & Memory

The first phase establishes the distributed RAG architecture:

* Qdrant knowledge retrieval
* Weaviate semantic memory
* Redis conversational history
* History-aware query rewriting
* Semantic memory extraction
* Memory deduplication
* Personalized retrieval
* Context-aware generation

### Phase 2 — Retrieval Optimization

The second phase improves the retrieval pipeline with:

* Asynchronous retrieval
* Parallel memory and knowledge search
* CrossEncoder reranking
* Improved candidate selection
* Top-K relevance filtering

```text
Phase 1
RAG + Memory
     │
     ▼
Phase 2
Async Retrieval + Reranking
```

---

## Environment Variables

Create a `.env` file containing the required service credentials:

```env
WEAVIATE_URL=YOUR_WEAVIATE_URL
WEAVIATE_API_KEY=YOUR_WEAVIATE_API_KEY

QDRANT_URL=YOUR_QDRANT_URL
QDRANT_API_KEY=YOUR_QDRANT_API_KEY

GROQ_API_KEY=YOUR_GROQ_API_KEY

REDIS_URL=YOUR_REDIS_URL
```

**Never commit API keys or `.env` files to the repository.**

---

## Key Design Goals

The project focuses on four major goals:

### 1. Personalization

Retrieve relevant information about the user from semantic memory.

### 2. Retrieval Quality

Combine vector retrieval with CrossEncoder reranking to improve the relevance of retrieved context.

### 3. Retrieval Efficiency

Use asynchronous operations to retrieve memory and knowledge concurrently.

### 4. Persistent Context

Separate short-term conversation history from long-term semantic memory.

---

## High-Level Data Flow

```text
User
 │
 ▼
Query
 │
 ▼
Redis ──────────────► Conversation Context
 │
 ▼
Query Rewriting
 │
 ├──────────────► Weaviate ──► User Memory
 │
 └──────────────► Qdrant ─────► Knowledge
                    │
                    ▼
              Candidate Context
                    │
                    ▼
               CrossEncoder
                 Reranker
                    │
                    ▼
                Top Context
                    │
                    ▼
                 LLM
                    │
                    ▼
              Final Response
                    │
                    ▼
           Memory Extraction
                    │
                    ▼
                Weaviate
```

---

## Future Improvements

Potential directions for further development include:

* Streaming responses
* More advanced hybrid search
* Adaptive retrieval depth
* Memory importance scoring
* Memory expiration and lifecycle management
* Distributed caching
* Retrieval evaluation and benchmarking
* Observability and tracing
* Production API deployment
* Containerization and orchestration

---

## Project Status

**Current Status: Phase 2 — Distributed RAG + Semantic Memory + Async Retrieval + Reranking**

The project demonstrates a distributed architecture for building **personalized, memory-aware RAG applications** by combining multiple retrieval systems with asynchronous processing and relevance reranking.
