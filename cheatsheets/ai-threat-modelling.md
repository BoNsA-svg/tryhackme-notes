# AI Threat Modelling Cheat Sheet

> Fast reference for scoping, analysing, and documenting threats to AI, ML, RAG, and agentic systems.

## Framework Stack

| Framework | Ask | Output |
| --- | --- | --- |
| **STRIDE-AI** | What can go wrong? | Threat category |
| **MITRE ATLAS** | How could it be done? | Technique, prerequisites, mitigation |
| **OWASP LLM Top 10 (2025)** | Where does it live? | Component-focused risk coverage |

## Assets

- Training, fine-tuning, feedback, and validation data
- Labels and data provenance records
- Model weights, artifacts, and registry entries
- System/developer prompts and configuration
- Embeddings, vector indexes, and source documents
- Feature stores and real-time inputs
- Conversation state and long-term memory
- Tools, plugins, service accounts, and API credentials
- Prompts, outputs, decisions, and audit logs

## Architecture Checklist

Map:

1. Users and identities
2. API gateway and application backend
3. Model endpoint and model version
4. System/developer prompts
5. RAG ingestion, vector store, and retrieval
6. Memory and conversation state
7. Tools, plugins, and downstream services
8. Training and evaluation pipelines
9. Model registry and deployment path
10. Monitoring, logs, and human approvals

Mark every trust boundary and data flow.

## AI Supply Chain

| Stage | Main Risks | Key Controls |
| --- | --- | --- |
| **Collection** | Poisoned, sensitive, or unlicensed data | Provenance, source allowlists, access control |
| **Cleaning / labelling** | Manipulated labels, hidden outliers | Review separation, sampling, integrity checks |
| **Training** | Embedded poison, compromised dependencies | Isolation, signed inputs, reproducible pipelines |
| **Validation / registry** | Backdoors, model swap, weak lineage | Adversarial tests, signing, hashes, approval gates |
| **Inference** | Injection, disclosure, agency, cost abuse | Validation, least privilege, rate limits, monitoring |

## STRIDE-AI

| Category | AI Examples | Ask |
| --- | --- | --- |
| **S — Spoofing** | Fake data source, model endpoint, identity input | How is this source or model authenticated? |
| **T — Tampering** | Data poisoning, model replacement, prompt injection | Who can modify data, prompts, models, or context? |
| **R — Repudiation** | Missing model/context/tool audit trail | Can this exact decision be reconstructed? |
| **I — Information Disclosure** | Training-data leakage, prompt leakage, model extraction | What sensitive information can outputs or metadata reveal? |
| **D — Denial of Service** | GPU exhaustion, denial of wallet, oversized context | What limits bound compute, tokens, tools, and cost? |
| **E — Elevation of Privilege** | Guardrail bypass, excessive agency, tool abuse | Can model influence become an unauthorised action? |

## Important MITRE ATLAS Techniques

| Technique | ID* | STRIDE |
| --- | --- | --- |
| Data Poisoning | `AML.T0020` | Tampering |
| Backdoor ML Model | `AML.T0018` | Tampering |
| Evade ML Model | `AML.T0015` | Spoofing / Tampering / EoP |
| Extract ML Model | `AML.T0024` | Information Disclosure |
| LLM Prompt Injection | `AML.T0051` | Tampering / EoP |

\*Verify current technique names and IDs on [MITRE ATLAS](https://atlas.mitre.org/) before formal reporting.

## OWASP LLM Top 10 (2025)

| ID | Risk | Primary Components |
| --- | --- | --- |
| **LLM01** | Prompt Injection | Input, RAG, memory, tool output |
| **LLM02** | Sensitive Information Disclosure | Data, prompts, retrieval, output |
| **LLM03** | Supply Chain | Models, datasets, dependencies, plugins |
| **LLM04** | Data and Model Poisoning | Training, feedback, registry |
| **LLM05** | Improper Output Handling | Frontend, APIs, downstream interpreters |
| **LLM06** | Excessive Agency | Agents, tools, service accounts |
| **LLM07** | System Prompt Leakage | Prompt configuration, model endpoint |
| **LLM08** | Vector and Embedding Weaknesses | Vector store, embeddings, RAG |
| **LLM09** | Misinformation | Output and user decision points |
| **LLM10** | Unbounded Consumption | Endpoint, gateway, tools, training jobs |

## Fast Assessment Workflow

1. Define purpose, users, sensitive decisions, and unacceptable outcomes.
2. Draw components, data flows, and trust boundaries.
3. Inventory AI-specific assets and owners.
4. Apply every STRIDE category to every component and boundary.
5. Enrich findings with ATLAS techniques and mitigations.
6. Map OWASP risks to each LLM component.
7. Write concrete abuse cases and prerequisites.
8. Score likelihood, impact, persistence, blast radius, and detectability.
9. Assign layered controls, owners, and deadlines.
10. Reassess after any model, data, prompt, tool, permission, or architecture change.

## Abuse-Case Formula

> An attacker who controls **[entry point]** manipulates **[asset/component]** using **[technique]**, causing **[technical and business impact]**.

Record:

- Attacker access and prerequisites
- Entry point and crossed trust boundary
- Affected asset and component
- STRIDE category
- ATLAS technique
- OWASP LLM risk
- Evidence or assumptions
- Likelihood and impact
- Controls and residual risk
- Owner and target date

## High-Value Controls

| Area | Controls |
| --- | --- |
| **Data** | Provenance, validation, anomaly detection, restricted contributors |
| **Models** | Signing, versioning, adversarial evaluation, drift monitoring |
| **RAG** | Source ACLs, tenant isolation, content integrity, freshness checks |
| **Prompts** | No secrets, version control, treat prompt leakage as possible |
| **Output** | Encode, validate, and constrain before downstream use |
| **Tools** | Least privilege, scoped credentials, deterministic authorization |
| **Actions** | Human approval for sensitive or irreversible operations |
| **Resources** | Rate, token, cost, retrieval, and tool-call limits |
| **Operations** | Full execution trace, alerts, rollback, incident playbooks |

## Non-Negotiable Rules

1. A system prompt is not an access-control boundary.
2. Model output is untrusted input to every downstream system.
3. Tool authorization must be enforced outside the model.
4. RAG content and tool output may contain hostile instructions.
5. Never store secrets in prompts.
6. Log enough to reproduce decisions, but protect sensitive log data.
7. Reassess whenever the model learns, retrieves, remembers, or gains a tool.

## Finding Template

```markdown
### [Finding title]

- Component / trust boundary:
- Asset:
- Threat actor / access:
- STRIDE:
- MITRE ATLAS:
- OWASP LLM:
- Attack path:
- Impact:
- Existing controls:
- Priority:
- Mitigations:
- Residual risk:
- Owner / due date:
```

## Related Notes

- [Full AI Threat Modelling Note](../knowledge-base/ai-security/ai-threat-modelling.md)
- [LLM Pentesting](../knowledge-base/ai-security/llm-pentesting.md)
- [MITRE ATLAS](https://atlas.mitre.org/)
- [OWASP GenAI Security Project](https://genai.owasp.org/)
