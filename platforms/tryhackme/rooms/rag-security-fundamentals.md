# TryHackMe Room Notes — RAG Security Fundamentals

## Room Purpose

This room explains how Retrieval-Augmented Generation introduces inference-time trust dependencies. External content can change answers without retraining the model, making ingestion, retrieval authorization, provenance, context assembly, and lifecycle management security-critical.

## Core RAG Flow

~~~text
User query → query embedding/search → authorization
→ retrieval/reranking → context assembly → LLM response
~~~

The complete system also includes source connectors, parsers, chunking, metadata, caches, citations, tools, monitoring, and deletion workflows.

## Core Components

- source connectors and parsers;
- chunking pipeline;
- embedding model;
- vector or hybrid index;
- retriever and reranker;
- authorization filter;
- context assembler;
- language model;
- output validator and citations;
- tools and lifecycle controls.

## Important Corrections

- Embeddings do not automatically remove authorship or approval status; pipelines lose security when they discard or ignore metadata.
- A model can see provenance if the application supplies it, but cannot independently verify authenticity.
- Retrieved content is not necessarily treated as authoritative; behavior depends on prompt construction and training. It must still be treated as untrusted.
- Retrieval is not universally the highest-risk component. Risk depends on data, identities, tools, and architecture.
- Output drift is a weak, nonspecific signal—not proof of poisoning.

## Main RAG Risks

### Corpus Poisoning

Malicious, false, stale, or unauthorized content enters the indexed knowledge base.

### Retrieval Manipulation

Content is crafted to rank highly using semantic targeting, keyword stuffing, duplicates, metadata abuse, or reranker weaknesses.

### Indirect Prompt Injection

Retrieved content contains instructions that attempt to override application intent or cause disclosure and actions.

### Broken Retrieval Authorization

The system retrieves content outside the active user's role or tenant. Authorization must be enforced before chunks reach the model context.

### Provenance and Lifecycle Failure

Chunks lose source, owner, ACL, version, approval, freshness, or deletion state.

### Vector and Embedding Weaknesses

The vector store may be exposed, poorly isolated, over-permissioned, or contain sensitive derived data.

## Active vs Passive Abuse

- **Passive:** poisoned or stale content waits for a normal query.
- **Active:** an attacker crafts or uploads content intended to rank for a target query.

State the attacker's prerequisite: ingestion access, control of a synchronized source, or influence over indexed content.

## Defense in Depth

### Ingestion

- approved sources;
- publisher authentication;
- file/content scanning;
- ACL and tenant metadata;
- versioning and hashes;
- quarantine and approval;
- freshness and deletion controls.

### Retrieval

- identity-bound queries;
- pre-context ACL filtering;
- score and source-quality thresholds;
- duplicate limits;
- reranker controls;
- fail-closed handling for missing metadata.

### Context

- separate privileged instructions;
- label retrieved chunks as untrusted evidence;
- preserve provenance;
- minimize chunks and secrets;
- prevent chunks from defining tools or authorization.

### Output and Actions

- validate citations;
- distinguish facts from inference;
- refuse when evidence is insufficient;
- authorize tools outside the model;
- require approval for consequential actions.

## Case-Study Correction

EchoLeak (CVE-2025-32711) was disclosed in 2025, not 2026. It demonstrated a zero-click indirect prompt-injection chain affecting Microsoft 365 Copilot through crafted email content. It should not be described as proof that organizations were actually breached unless a source confirms that outcome.

## Framework Mapping

Relevant OWASP Top 10 for LLM Applications 2025 categories include:

- LLM01 Prompt Injection;
- LLM02 Sensitive Information Disclosure;
- LLM04 Data and Model Poisoning;
- LLM06 Excessive Agency;
- LLM08 Vector and Embedding Weaknesses.

Critical correction: LLM07:2025 is **System Prompt Leakage**, not “Insecure Model Monitoring.”

NIST AI RMF supports governance, mapping, measurement, and management of RAG dependencies and risk.

EU AI Act Articles 9 and 10 may apply to relevant high-risk AI systems. Their applicability depends on legal classification; they are not universal RAG requirements.

## Safe Practical Assessment

Use synthetic:

- role and tenant canaries;
- expired and superseded documents;
- conflicting sources;
- controlled injection text;
- duplicate/ranking-manipulation chunks;
- deleted records;
- mock tools.

Capture retrieved chunk IDs, scores, sources, ACL decisions, citations, and tool outcomes. Never test cross-user exposure using real confidential data.

## Related Permanent Notes

- [RAG Security Fundamentals](../../../knowledge-base/ai-security/rag-security-fundamentals.md)
- [RAG Security Assessment Methodology](../../../pentesting-methodology/ai-security/rag-security-assessment.md)
- [RAG Security Cheat Sheet](../../../cheatsheets/rag-security.md)
- [Prompt Injection](../../../knowledge-base/ai-security/prompt-injection.md)
- [Prompt Defence](../../../knowledge-base/ai-security/prompt-defence.md)

## Room Summary

RAG security is not merely prompt filtering. It requires trusted ingestion, identity-aware retrieval, provenance, lifecycle control, safe context assembly, independent authorization, and continuous validation of what was retrieved and why.
