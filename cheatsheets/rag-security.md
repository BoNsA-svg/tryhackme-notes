# RAG Security — Quick Reference

## Architecture

~~~text
Ingestion:
Source → parse → validate → chunk → metadata/ACL → embed → index

Query:
Identity + query → retrieve → authorize → rerank
→ assemble context → LLM → validate/cite → user/tool
~~~

## Trust Rules

- Retrieved content is untrusted data.
- Similarity does not imply truth, authority, or permission.
- Preserve source, owner, tenant, ACL, version, dates, and hash.
- Unauthorized chunks must not enter model context.
- The model does not make access-control decisions.
- Tool authorization occurs outside the model.
- Embeddings are potentially sensitive derived data.

## Threat Matrix

| Threat | Example |
|---|---|
| Corpus poisoning | False or malicious document indexed |
| Indirect prompt injection | Retrieved text contains instructions |
| Retrieval manipulation | Crafted content ranks highly |
| Broken authorization | Cross-user or cross-tenant chunk returned |
| Stale content | Superseded policy remains active |
| Provenance loss | Chunk loses owner/approval metadata |
| Vector-store exposure | Unauthenticated collection/export |
| Context leakage | Hidden chunk or source disclosed |
| Unsafe agency | Retrieved content drives a tool call |

## Ingestion Checks

- Approved connector/publisher
- File/type/size validation
- Malware/content scanning
- ACL and tenant metadata preserved
- Version/hash/timestamps
- Approval and quarantine
- Deduplication/conflict handling
- Freshness and expiry
- Deletion/tombstone propagation

## Retrieval Checks

- User and tenant bound to every query
- ACL filter before chunks leave retrieval
- Missing metadata fails closed
- Score/source-quality threshold
- Reranker controlled
- Duplicate/source dominance limited
- Minimum necessary chunks
- Cache and fallback authorization tested

## Safe Test Corpus

- Public canary
- Role-restricted canary
- Tenant A/B canaries
- Expired document
- Superseded document
- Conflicting documents
- Controlled instruction-like chunk
- Duplicate/keyword-stuffed chunks
- Deleted document

## Context Controls

- Label chunks as untrusted evidence
- Keep privileged instructions separate
- Include provenance
- Bound chunk count and size
- Remove unnecessary secrets
- Do not allow chunks to define tools/policies
- Detect injection as defense in depth
- Fail safely when sources conflict or are insufficient

## Monitoring Fields

- User/tenant
- Query
- Chunk IDs/source/version
- Scores and reranker
- ACL decision
- Context hash
- Model/prompt version
- Citations
- Tool proposal/decision
- Output-validation result

## Framework Correction

Relevant OWASP 2025 categories:

- LLM01 Prompt Injection
- LLM02 Sensitive Information Disclosure
- LLM04 Data and Model Poisoning
- LLM06 Excessive Agency
- LLM08 Vector and Embedding Weaknesses

LLM07 is **System Prompt Leakage**, not “Insecure Model Monitoring.”

## Incident Response

~~~text
Disable source/connector → preserve evidence → quarantine documents
→ rebuild affected index → fix ACL/provenance → rotate credentials
→ validate deletion → regression test every tenant/role
~~~

## Key Warning

Output drift is not proof of poisoning. It may come from model, prompt, source, index, reranker, traffic, or configuration changes.
