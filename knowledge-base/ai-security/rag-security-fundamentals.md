# RAG Security Fundamentals

> Retrieval-Augmented Generation improves model grounding by adding external content at inference time. It also adds a data pipeline, authorization boundary, and indirect-instruction channel that must be secured independently of the model.

## Learning Goals

- Explain the complete RAG ingestion and query pipelines.
- Identify trust boundaries introduced by external knowledge.
- Distinguish corpus poisoning, retrieval manipulation, indirect prompt injection, and authorization failures.
- Assess data provenance, freshness, deletion, and tenant isolation.
- Design layered controls across ingestion, retrieval, context assembly, generation, and output.

## What RAG Does

A RAG application retrieves relevant content when a user asks a question and supplies that content to a language model.

RAG can improve freshness and grounding, but it does not make retrieved content true, authorized, current, or safe. The application—not the model—must establish those properties.

## Complete RAG Architecture

### Ingestion Pipeline

~~~text
Source → connector/parser → validation → chunking
→ metadata/provenance → embedding → vector/index storage
~~~

### Query Pipeline

~~~text
User + identity → query processing → embedding/search
→ authorization filter → reranking → context assembly
→ LLM → output validation/citations → user or tool
~~~

Core components include:

- source connectors;
- parsers and document processors;
- chunker;
- metadata and provenance store;
- embedding model;
- vector or hybrid search index;
- retriever;
- authorization filter;
- reranker;
- prompt/context assembler;
- LLM;
- output validator;
- citation layer;
- tools and downstream actions;
- monitoring and lifecycle controls.

A simplified four-component diagram is useful for learning, but it can hide the controls where real security decisions belong.

## Trust Boundaries

Every boundary should preserve identity, provenance, authorization, integrity, and lifecycle state:

1. external source to ingestion;
2. parser/chunker to indexed chunks;
3. user identity to retrieval filter;
4. vector store to retriever;
5. retrieved data to model context;
6. model output to user or tool;
7. document update/deletion to index synchronization.

Retrieved content is untrusted data, even when it came from an internal email, wiki, or shared drive.

## Main RAG Threat Classes

### 1. Corpus or Knowledge-Base Poisoning

An attacker or faulty process adds, modifies, or retains misleading content in the indexed corpus.

Examples:

- false policies;
- altered support instructions;
- duplicated attacker-controlled chunks;
- malicious metadata;
- outdated documents left active;
- poisoned embeddings or index entries.

“Data poisoning” is broad. In RAG, be precise about whether the attacker changed source documents, chunks, metadata, embeddings, ranking, or context.

### 2. Indirect Prompt Injection

A retrieved document contains instruction-like content that attempts to override system or developer intent.

The user may ask a normal question. The attack arrives through the retrieved source.

Potential impact depends on application capability:

- manipulated answer;
- hidden data disclosure;
- unsafe tool call;
- unauthorized message or transaction;
- persistent contamination through memory or re-ingestion.

### 3. Retrieval Manipulation

Content is crafted to rank highly for targeted queries.

Techniques may include:

- semantic similarity optimization;
- keyword or phrase stuffing;
- duplicate chunks;
- query-specific trigger language;
- manipulated metadata;
- poisoning a reranker;
- exploiting score thresholds or fallback behavior.

Semantic relevance is not evidence of correctness or trust.

### 4. Broken Retrieval Authorization

The retriever returns data the current user is not authorized to access.

Common causes:

- filtering after retrieval or generation;
- missing tenant metadata;
- stale ACL synchronization;
- shared caches;
- predictable document IDs;
- service accounts with global access;
- fallback search that ignores permissions;
- vector indexes not partitioned or filtered correctly.

Authorization must occur at retrieval time using trusted identity and access-control data. The model must never decide whether the user is allowed to see a chunk.

### 5. Provenance and Metadata Loss

Embeddings do not inherently erase authorship or approval status. Security is lost when the pipeline fails to retain or enforce metadata.

Preserve:

- source;
- owner;
- tenant;
- classification;
- ACL;
- approval status;
- version;
- effective and expiry dates;
- ingestion time;
- hash;
- deletion/tombstone state.

### 6. Stale and Conflicting Content

RAG failures do not require an attacker. Old policies, duplicate versions, incomplete deletion, or conflicting sources can produce confident but wrong answers.

Controls include source-of-truth ranking, effective dates, TTLs, conflict handling, re-indexing, deletion propagation, and freshness monitoring.

### 7. Vector Store and Embedding Risks

- unauthenticated or exposed vector database;
- cross-tenant collections;
- backup leakage;
- insecure management APIs;
- embedding inversion or sensitive-data inference;
- membership inference;
- poisoned or substituted embedding model;
- excessive metadata disclosure;
- denial of service through large ingestion or expensive queries.

Embeddings should be treated as potentially sensitive derived data.

### 8. Context and Output Leakage

The application may reveal:

- hidden chunks;
- document titles or IDs;
- neighboring content;
- system instructions;
- retrieval scores;
- source URLs;
- data from another user;
- secrets included in indexed documents.

Citations improve traceability but may themselves expose restricted metadata or links.

## Why the Model Cannot Solve This Alone

A model can use provenance only if the application supplies it. It cannot independently prove a document is authentic, current, or authorized.

Role labels, delimiters, and warnings can reduce instruction confusion, but they do not enforce access control. The system must constrain what reaches the context and what actions model output may trigger.

## Retrieval Abuse: Active and Passive

- **Passive:** poisoned, stale, or malicious content already exists and waits for a normal query.
- **Active:** an attacker crafts or uploads content to rank for targeted queries.

The attacker may need ingestion access, control of an indexed external source, or influence over content synchronized from another service. Threat modeling should state that prerequisite.

## Detection

No single signal proves poisoning or injection.

Monitor:

- newly ingested sources and ownership changes;
- unusual duplicate or near-duplicate chunks;
- abnormal retrieval concentration;
- sudden source dominance;
- rank/score distribution changes;
- retrieval of expired or deleted documents;
- cross-tenant authorization failures;
- instruction-like content in retrieved chunks;
- unexpected tool calls after retrieval;
- answer/citation disagreement;
- behavior changes on a fixed evaluation suite.

Output drift is nonspecific. It may result from model, prompt, retrieval, data, or traffic changes—not necessarily poisoning.

## Defensive Principles

### Ingestion

- approve source connectors;
- authenticate publishers;
- scan files and content;
- preserve metadata and ACLs;
- validate type, size, and structure;
- deduplicate and version;
- separate trusted and untrusted corpora;
- quarantine new or changed sources;
- review high-impact knowledge.

### Retrieval

- bind every query to user and tenant identity;
- enforce ACL filters before returning chunks;
- set score and source-quality thresholds;
- use trusted reranking signals;
- limit duplicates and single-source dominance;
- retrieve the minimum necessary content;
- fail closed when authorization metadata is missing.

### Context Assembly

- label content and provenance;
- treat chunks as quoted data, not instructions;
- isolate sources;
- limit length and number of chunks;
- remove unnecessary secrets;
- separate tool instructions from retrieved text;
- preserve citations;
- apply injection detection as defense in depth.

### Generation and Actions

- require grounded answers when appropriate;
- validate citations against retrieved chunks;
- refuse when evidence is insufficient or conflicting;
- enforce tool authorization outside the model;
- require approval for consequential actions;
- validate output before downstream use.

### Lifecycle

- propagate updates and deletions;
- support emergency source revocation;
- expire stale content;
- rebuild affected embeddings;
- retain audit logs;
- run regular authorization and poisoning tests.

## Case-Study Accuracy

EchoLeak, CVE-2025-32711, was publicly disclosed in 2025—not 2026. It demonstrated a zero-click indirect prompt-injection chain against Microsoft 365 Copilot using crafted email content and data-exfiltration mechanisms. It should be described as a disclosed vulnerability and proof of impact, not evidence that arbitrary organizations were confirmed breached.

Plugin and browsing research also demonstrates the general indirect-injection pattern, but each case should be tied to its original disclosure before exact impact claims are repeated.

## Framework Mapping

### OWASP Top 10 for LLM Applications 2025

Relevant categories include:

- **LLM01: Prompt Injection**
- **LLM04: Data and Model Poisoning**
- **LLM08: Vector and Embedding Weaknesses**
- **LLM06: Excessive Agency**, when retrieved content drives unsafe actions
- **LLM02: Sensitive Information Disclosure**, when restricted data is exposed

Important correction: **LLM07:2025 is System Prompt Leakage**, not “Insecure Model Monitoring.”

### NIST AI RMF

- **Govern:** ownership, policy, and accountability;
- **Map:** sources, dependencies, users, and impacts;
- **Measure:** retrieval quality, authorization, drift, and adversarial evaluation;
- **Manage:** mitigation, monitoring, response, and lifecycle control.

### EU AI Act

Articles 9 and 10 address risk-management and data/data-governance obligations for relevant high-risk AI systems. Applicability depends on the system's legal classification and role; they should not be presented as universal RAG-specific controls.

## Related Notes

- [RAG Security Assessment Methodology](../../pentesting-methodology/ai-security/rag-security-assessment.md)
- [RAG Security Cheat Sheet](../../cheatsheets/rag-security.md)
- [Prompt Injection](prompt-injection.md)
- [Prompt Defence](prompt-defence.md)
- [AI Threat Modelling](ai-threat-modelling.md)
