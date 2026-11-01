<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=700&size=25&duration=2800&pause=900&color=38BDF8&center=true&vCenter=true&width=900&lines=ResearchPilot;AI-Powered+Document+Research+Assistant;Ask+better+questions.+Find+grounded+answers." alt="ResearchPilot animated title" />

<br/>

<p>
<strong>Turn document collections into searchable knowledge — with
answers linked back to their sources.</strong>
</p>
<p>
<img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python"/>
<img src="https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI"/>
<img src="https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB" alt="React"/>
<img src="https://img.shields.io/badge/RAG-Retrieval%20Augmented%20Generation-6D5AE6?style=for-the-badge" alt="RAG"/>
<img src="https://img.shields.io/badge/Status-In%20Development-CA8A04?style=for-the-badge" alt="In development"/>
</p>

<em>Ingest documents. Retrieve evidence. Generate traceable
answers.</em>

</div>

------------------------------------------------------------------------

## What is ResearchPilot?

**ResearchPilot** is an AI-powered document research assistant designed
to help users ask questions across PDF collections and receive answers
grounded in the source material.

Instead of treating a language model as the source of truth,
ResearchPilot is built around a **Retrieval-Augmented Generation (RAG)**
workflow: find relevant passages first, then provide those passages as
context for answer generation.

The goal is to make document question-answering more transparent by
showing **where an answer came from**, which passages were retrieved,
and when the available documents do not contain enough evidence.

> **Project status:** ResearchPilot is under active development. This
> README documents the intended architecture and engineering roadmap.
> Features and metrics will be marked as complete only after
> implementation and evaluation.

## The problem

Searching long PDFs manually is slow. Asking a general-purpose chatbot
about a document can also produce answers that sound convincing but are
not supported by the supplied material.

ResearchPilot aims to address this with:

- **Document-aware retrieval:** search by meaning, not only exact
  keyword matches.
- **Evidence-grounded generation:** generate responses using retrieved
  source passages.
- **Source traceability:** connect answer claims to document names and
  page references.
- **Answerability checks:** avoid presenting unsupported answers as
  facts.
- **Evaluation:** measure retrieval and citation quality against a
  curated dataset.

## Product workflow

``` mermaid
flowchart LR
    A["Upload PDF"] --> B["Extract text"]
    B --> C["Clean and split into chunks"]
    C --> D["Generate embeddings"]
    D --> E["Index vectors + metadata"]
    E --> F["Ask a question"]
    F --> G["Retrieve relevant passages"]
    G --> H["Build grounded prompt"]
    H --> I["Generate answer"]
    I --> J["Return answer + citations"]

    classDef input fill:#172554,stroke:#60A5FA,color:#EFF6FF
    classDef process fill:#164E63,stroke:#22D3EE,color:#ECFEFF
    classDef retrieval fill:#312E81,stroke:#A78BFA,color:#F5F3FF
    classDef output fill:#14532D,stroke:#4ADE80,color:#F0FDF4
    class A,F input
    class B,C,D,E process
    class G,H retrieval
    class I,J output
```

## System architecture

The application is planned as a React client communicating with a
FastAPI service. The backend coordinates document processing, retrieval,
and answer generation.

``` mermaid
flowchart TB
    U["User"] --> UI["React Web Application"]
    UI --> API["FastAPI REST API"]

    API --> DOC["Document Ingestion Service"]
    DOC --> PARSE["PDF Text Extraction"]
    PARSE --> CHUNK["Chunking + Metadata"]
    CHUNK --> EMB["Embedding Model"]
    EMB --> VS[("Vector Index")]

    API --> RET["Retrieval Service"]
    RET --> VS
    RET --> RERANK["Optional Reranking"]
    RERANK --> CTX["Evidence Context Builder"]
    CTX --> LLM["Language Model"]
    LLM --> VERIFY["Citation / Answer Validation"]
    VERIFY --> API
    API --> UI

    META[("Document Metadata Store")] --- API

    classDef client fill:#172554,stroke:#60A5FA,color:#EFF6FF
    classDef service fill:#164E63,stroke:#22D3EE,color:#ECFEFF
    classDef store fill:#312E81,stroke:#A78BFA,color:#F5F3FF
    classDef model fill:#713F12,stroke:#FBBF24,color:#FFFBEB
    class U,UI client
    class API,DOC,PARSE,CHUNK,RET,RERANK,CTX,VERIFY service
    class VS,META store
    class EMB,LLM model
```

### Main components

| Component               | Responsibility                                                                  |
|-------------------------|---------------------------------------------------------------------------------|
| **React client**        | PDF upload, document library, question interface, answer and citation display.  |
| **FastAPI service**     | Request validation, API orchestration, document and query workflows.            |
| **PDF extraction**      | Extract text while preserving page-level metadata where available.              |
| **Chunking**            | Split extracted text into retrievable passages with document and page metadata. |
| **Embedding model**     | Convert text passages and questions into vector representations.                |
| **Vector index**        | Store and retrieve semantically similar passages.                               |
| **Retrieval pipeline**  | Select relevant evidence, with optional reranking.                              |
| **Answer generation**   | Generate a response constrained by retrieved context.                           |
| **Citation validation** | Check that displayed references map to retrieved source passages.               |

## Retrieval-Augmented Generation

ResearchPilot follows a retrieve-then-generate approach.

``` mermaid
sequenceDiagram
    autonumber
    participant User
    participant API as FastAPI
    participant Index as Vector Index
    participant Model as Language Model
    participant UI as React UI

    User->>API: Ask a question
    API->>API: Embed the question
    API->>Index: Similarity search
    Index-->>API: Top-k passages + metadata
    API->>API: Build evidence context
    API->>Model: Question + retrieved context
    Model-->>API: Draft answer
    API->>API: Validate citation references
    API-->>UI: Answer + source citations
    UI-->>User: Display grounded response
```

### Why citations matter

A citation should be more than a decorative source link. Each reference
should identify the document and page (or another stable location) from
which the supporting passage was extracted.

The system should distinguish between:

- **Supported answer:** retrieved evidence adequately supports the
  response.
- **Partially supported answer:** some parts are supported, while others
  need qualification.
- **Insufficient evidence:** the documents do not provide enough
  information to answer confidently.

## Planned evaluation

A RAG system should be evaluated as a retrieval system and as an
answer-generation system. A fluent answer alone is not evidence of
correctness.

| Evaluation area   | Metric / method       | What it measures                                                        |
|-------------------|-----------------------|-------------------------------------------------------------------------|
| Retrieval quality | Recall@5              | Whether relevant passages appear in the top five retrieved results.     |
| Ranking quality   | MRR / nDCG@k          | How highly relevant passages are ranked.                                |
| Citation quality  | Citation correctness  | Whether citations point to passages that support the associated claims. |
| Answer quality    | Human-reviewed rubric | Factual support, completeness, and relevance.                           |
| Reliability       | Answerability tests   | Whether the system abstains when evidence is insufficient.              |
| Performance       | End-to-end latency    | Time from question submission to displayed answer.                      |

**Evaluation target:** build a curated question set with known
supporting passages, then report measured metrics with the dataset size,
retrieval configuration, and model details. No quality score is claimed
until the evaluation has been run.

## Planned repository structure

The following is a proposed structure. Actual module names may change as
implementation progresses.

``` text
ResearchPilot/
├── README.md
├── .gitignore
├── docs/
│   ├── architecture.md
│   ├── rag-pipeline.md
│   └── evaluation.md
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   │   ├── routes/
│   │   │   └── schemas/
│   │   ├── core/
│   │   │   └── config.py
│   │   ├── services/
│   │   │   ├── pdf_extractor.py
│   │   │   ├── chunking.py
│   │   │   ├── embeddings.py
│   │   │   ├── retrieval.py
│   │   │   ├── generation.py
│   │   │   └── citations.py
│   │   ├── storage/
│   │   └── models/
│   ├── tests/
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   └── types/
│   └── package.json
└── evaluation/
    ├── dataset/
    ├── run_retrieval_eval.py
    └── metrics.py
```

## Development roadmap

| Phase | Milestone                                           | Status  |
|-------|-----------------------------------------------------|---------|
| 01    | FastAPI application and project configuration       | Planned |
| 02    | PDF upload, extraction, and page metadata           | Planned |
| 03    | Text chunking and document metadata                 | Planned |
| 04    | Embedding generation and vector indexing            | Planned |
| 05    | Semantic retrieval and top-k passage selection      | Planned |
| 06    | Context-grounded answer generation                  | Planned |
| 07    | Citation mapping and insufficient-evidence handling | Planned |
| 08    | React document library and chat interface           | Planned |
| 09    | Retrieval evaluation and failure analysis           | Planned |
| 10    | Testing, deployment, and documentation              | Planned |

<details>
<summary>
<strong>Engineering checklist</strong>
</summary>

- [ ] Validate PDF type, size, and extraction errors
- [ ] Preserve document IDs, page numbers, and chunk metadata
- [ ] Define chunk size and overlap experimentally
- [ ] Choose and document an embedding model
- [ ] Persist vector index and metadata safely
- [ ] Implement top-k retrieval with configurable parameters
- [ ] Add optional reranking only after establishing a baseline
- [ ] Ensure citations map to actual source passages
- [ ] Test questions with no supporting evidence
- [ ] Create a fixed evaluation dataset and reproducible scripts
- [ ] Measure latency and retrieval quality
- [ ] Add automated tests for ingestion, retrieval, and API routes

</details>

## Technology stack

The planned stack is intentionally modular:

- **Backend:** Python, FastAPI, Pydantic
- **Frontend:** React, JavaScript / TypeScript
- **RAG:** document chunking, embeddings, vector similarity search,
  retrieval, grounded generation
- **Storage:** vector index and document metadata persistence
  (implementation choice to be documented)
- **Evaluation:** Recall@k, MRR / nDCG, citation review, latency
  measurement
- **Testing and tooling:** pytest, Git, API testing tools

Model providers and vector database choices should be selected based on
cost, local development needs, licensing, and reproducibility. Avoid
committing API keys or private documents.

## Reliability, privacy, and limitations

- Treat uploaded documents as untrusted input.
- Keep secrets in environment variables and exclude them from version
  control.
- Validate uploads and impose reasonable file and processing limits.
- Do not assume retrieved passages are correct simply because they are
  semantically similar.
- Make source references inspectable and preserve page metadata.
- Clearly communicate when the available evidence is insufficient.
- Document model limitations, evaluation scope, and known failure cases.

## References

- [Retrieval-Augmented Generation for Knowledge-Intensive NLP
  Tasks](https://arxiv.org/abs/2005.11401)
- [Sentence Transformers documentation](https://www.sbert.net/)
- [FastAPI documentation](https://fastapi.tiangolo.com/)
- [React documentation](https://react.dev/)

## Contributing and feedback

ResearchPilot is an individual learning and engineering project.
Feedback on retrieval quality, citation correctness, evaluation
methodology, and architecture is welcome.

When reporting an issue, include the document format, a minimal
reproducible example (without sharing private document contents),
expected behavior, and actual behavior.

------------------------------------------------------------------------

<div align="center">

**ResearchPilot — answers should be useful, traceable, and grounded in
evidence.**

<sub>Designed and developed by
<a href="https://github.com/Sarthaktanpure">Sarthak Tanpure</a></sub>

</div>
