# AI Threat Modelling

## Overview

AI systems inherit the risks of conventional applications and add new assets, trust boundaries, and failure modes. Training data can be poisoned, model behaviour can be copied, retrieval content can carry hidden instructions, and an agent can turn a text injection into a real action through its tools.

AI threat modelling is therefore not a replacement for traditional threat modelling. It extends it with three complementary lenses:

| Framework | Main Question | Purpose |
| --- | --- | --- |
| **STRIDE-AI** | What can go wrong? | Categorise threats systematically |
| **MITRE ATLAS** | How could an adversary do it? | Describe AI-specific tactics, techniques, and mitigations |
| **OWASP Top 10 for LLM Applications (2025)** | Where does the risk live? | Scope and prioritise risks in LLM architecture |

> **Version note:** This lesson uses the 2025 OWASP LLM Top 10 because that is the version used by the TryHackMe room. Frameworks change, so verify current names and technique IDs before using this material in a formal assessment.

This is a defender-focused methodology for evaluating and documenting threats, not exploiting them.

---

## Learning Objectives

By the end of this note, you should be able to:

1. Identify assets and attack surfaces unique to AI systems.
2. Trace risk across the AI data and model supply chain.
3. Apply STRIDE with AI-specific context.
4. Enrich findings with MITRE ATLAS techniques.
5. Map OWASP LLM risks to architectural components.
6. Produce a structured, prioritised threat assessment.

---

## Scenario: MegaCorp

MegaCorp operates three AI systems:

1. A customer-facing LLM chatbot connected to internal knowledge through a retrieval-augmented generation (RAG) pipeline.
2. An internal recommendation engine that processes sensitive customer data.
3. A fraud detection model that makes real-time transaction decisions and is retrained monthly.

These systems demonstrate three distinct threat surfaces: natural-language interaction and retrieval, sensitive-data processing and model intellectual property, and a continuously changing training pipeline.

---

## 1. What Makes AI Threat Modelling Different?

Traditional systems mainly execute explicit, deterministic code. AI systems learn behaviour from data and often produce probabilistic results. Their security depends not only on code and configuration, but also on data provenance, model state, prompts, retrieved context, embeddings, and connected tools.

### Important AI characteristics

| Characteristic | Security Effect |
| --- | --- |
| **Probabilistic output** | The same input can produce different results, complicating testing and incident reproduction. |
| **Limited explainability** | It may be difficult to determine why a model produced a decision. |
| **Learned behaviour** | Poisoned data can introduce faults that persist inside a trained model. |
| **Natural-language control** | Instructions and untrusted content can share the same context window. |
| **External context** | RAG, memory, plugins, and tools continuously add new trust boundaries. |
| **High inference cost** | Abuse can cause both service exhaustion and large financial loss. |

The key principle is that an AI system is not merely a traditional application with a model attached. The model lifecycle and the surrounding orchestration layer must both be included in scope.

---

## 2. AI Assets to Protect

| Asset | What It Is | Why It Matters |
| --- | --- | --- |
| **Training and fine-tuning data** | Data used to teach or adapt model behaviour | Poisoning, privacy leakage, or poor provenance can alter the deployed model. |
| **Labels and feedback data** | Human or automated judgments used during training | Manipulated labels teach incorrect associations and may introduce targeted blind spots. |
| **Model weights and parameters** | Numerical state representing what the model learned | Theft exposes valuable intellectual property; modification can implant backdoors. |
| **Model artifacts and registry entries** | Versioned models, metadata, and deployment packages | Registry compromise can replace an approved model with a malicious one. |
| **System and developer prompts** | Instructions governing model behaviour and constraints | Disclosure reveals business logic and guardrails; prompts must never contain secrets. |
| **Embeddings and vector indexes** | Numerical representations used for search and retrieval | Poisoning or unauthorised access changes retrieved context or exposes source information. |
| **Feature stores** | Preprocessed inputs supplied to ML models | Feature tampering changes what the model sees at decision time. |
| **Conversation state and memory** | Short- or long-term context retained by the application | Poisoning can persist false information or malicious instructions across sessions. |
| **Tools, plugins, and service accounts** | Capabilities available to an AI agent | A compromised model can act with every permission granted to these integrations. |
| **Prompts, outputs, and audit records** | Evidence needed to reproduce AI decisions | Weak logging creates repudiation, investigation, and compliance problems. |

### Trust boundaries to mark

During diagramming, mark every place where data or control crosses between:

1. Users and the application.
2. The application and the model endpoint.
3. The model and the RAG or memory layer.
4. The model and external tools.
5. Data sources and the training pipeline.
6. Training infrastructure and the model registry.
7. The registry and production deployment.
8. The AI system and human decision-makers.

---

## 3. The AI Data and Model Supply Chain

AI systems inherit software supply-chain risk and add a separate data-and-model chain.

### Stage 1: Data collection

Data may come from internal databases, web scraping, users, purchased datasets, sensors, or third parties. Risks include untrusted contributors, unclear ownership, hidden sensitive data, and deliberate poisoning.

**Defensive focus:** source allowlists, provenance records, access control, licensing review, and anomaly detection.

### Stage 2: Cleaning and labelling

Data is filtered, transformed, and labelled by humans or automated systems. Compromised labels can quietly teach the wrong associations without making the dataset look obviously corrupted.

**Defensive focus:** reviewer separation, label-quality sampling, integrity checks, and traceable transformations.

### Stage 3: Training and fine-tuning

The model learns from prepared data. Poison that survives earlier controls can become embedded in the weights, and compromised dependencies or training infrastructure can alter the result.

**Defensive focus:** isolated training, reproducible pipelines, signed inputs, dependency controls, and controlled secrets.

### Stage 4: Validation, packaging, and registration

The model is evaluated, versioned, and stored. A backdoor may evade ordinary accuracy tests when its trigger is absent from the validation set. Registry compromise may replace an approved model.

**Defensive focus:** adversarial evaluation, model signing, artifact hashes, approval gates, lineage records, and immutable versioning.

### Stage 5: Deployment and inference

Production inputs, RAG documents, memory, plugins, and tools affect the model at runtime. This creates injection, disclosure, excessive-agency, and resource-consumption risks even when the underlying model is unchanged.

**Defensive focus:** input and output validation, least privilege, tool authorization, retrieval access controls, rate limits, monitoring, and human approval for high-impact actions.

### Why delayed effects matter

A conventional vulnerable package can often be replaced quickly. Training-data poisoning may remain hidden until the next retraining cycle and then persist in the new model. Recovery can require locating the contaminated records, rebuilding the dataset, retraining, validating, and redeploying the model.

**MegaCorp example:** crafted transactions slowly enter the fraud model's monthly training data. Over time, they shift the decision boundary so that a specific fraud pattern is approved. The compromise becomes visible only after financial loss occurs.

---

## 4. STRIDE Adapted for AI

STRIDE remains useful, but each category must be interpreted across data, model, orchestration, and human layers.

| Category | Security Property | AI Manifestation | MegaCorp Example |
| --- | --- | --- | --- |
| **Spoofing** | Authenticity | Data-source impersonation, fake model endpoints, adversarial identity inputs | Fabricated policy documents enter the chatbot's RAG source. |
| **Tampering** | Integrity | Data poisoning, model replacement, prompt injection, feature manipulation | Crafted transactions alter the fraud model after retraining. |
| **Repudiation** | Accountability | Missing prompt, context, model-version, tool-call, or decision logs | The team cannot reproduce why a transaction was approved. |
| **Information Disclosure** | Confidentiality | Training-data extraction, prompt leakage, embedding inversion, behavioural model extraction | A competitor builds a surrogate of the recommendation engine from API outputs. |
| **Denial of Service** | Availability and cost | GPU exhaustion, oversized context, expensive outputs, training disruption | Repeated long requests create a denial-of-wallet attack. |
| **Elevation of Privilege** | Authorisation | Guardrail bypass, excessive agency, unauthorised tool use | An injected chatbot abuses an overprivileged database tool. |

### S — Spoofing

Spoofing is no longer limited to user identity. An attacker may impersonate a trusted data source, model service, document, sensor, or identity signal.

Ask:

1. How does the system authenticate models, data sources, and retrieved documents?
2. Can an attacker publish content that the RAG system treats as authoritative?
3. Can a fake endpoint or model artifact replace a trusted service?

### T — Tampering

Tampering includes changes to training data, labels, model artifacts, embeddings, features, prompts, retrieved context, and runtime configuration.

Prompt injection may cross categories. It is **Tampering** when it changes the model's effective instructions and **Elevation of Privilege** when it causes unauthorised actions.

Ask:

1. Who can contribute to or modify each dataset?
2. Are models and datasets signed and versioned?
3. Is untrusted text clearly separated from trusted control instructions?

### R — Repudiation

AI decisions are difficult to defend when the system cannot reconstruct the complete execution context.

Useful audit evidence includes:

1. User and service identity.
2. Timestamp and request ID.
3. Model name, exact version, and configuration.
4. System/developer prompt version or hash.
5. Retrieved documents and their versions.
6. Tool calls, arguments, authorization decisions, and results.
7. Model output, policy decisions, and human approvals.

Logs must protect privacy and secrets. Do not solve accountability by recording sensitive prompts or credentials indiscriminately.

### I — Information Disclosure

AI systems can disclose conventional secrets plus learned or derived information. Black-box model extraction generally creates a behavioural surrogate; it does not normally recover the exact original weights.

Ask:

1. Can outputs reveal PII, confidential documents, or memorised data?
2. Does the API expose confidence scores, embeddings, or excessive metadata?
3. Are secrets embedded in system prompts?
4. Can repeated queries reproduce proprietary model behaviour?

### D — Denial of Service

Availability includes economic availability. The service may remain online while attackers exhaust tokens, GPU time, context capacity, or cloud budget.

Ask:

1. Are there limits on requests, tokens, output length, retrieval volume, and tool calls?
2. Can one user trigger unusually expensive workflows?
3. Are budgets and abnormal-cost alerts enforced per tenant?

### E — Elevation of Privilege

For agentic systems, privilege is the complete set of actions available through tools, plugins, credentials, and downstream APIs. A successful injection becomes far more dangerous when the agent has broad permissions.

Ask:

1. Which actions can the model request, and which can it execute directly?
2. Is authorization enforced by deterministic code outside the model?
3. Which actions require user confirmation or human approval?
4. Are tools scoped by tenant, resource, and operation?

### What STRIDE does not capture cleanly

Some issues span several categories or are not strictly adversarial:

1. Adversarial examples can involve spoofing, tampering, and privilege bypass.
2. Bias and unfair outcomes are safety, governance, and compliance risks even without an attacker.
3. Hallucination and emergent behaviour can cause harm without a conventional exploit.
4. Model drift may degrade security controls over time.

This is why STRIDE should be supplemented with AI-specific knowledge bases and risk lists.

---

## 5. MITRE ATLAS

MITRE ATLAS is a knowledge base of adversary tactics and techniques for AI-enabled systems. It plays a role similar to MITRE ATT&CK, but focuses on threats involving machine learning and generative AI.

| ATLAS Element | Question Answered |
| --- | --- |
| **Tactic** | Why is the adversary acting? |
| **Technique** | How is the objective achieved? |
| **Sub-technique** | Which specific variation is used? |
| **Mitigation** | What defensive measure reduces the risk? |
| **Case study** | What did a documented real-world or research attack look like? |

### Important techniques from this lesson

| Technique | Description | STRIDE Link |
| --- | --- | --- |
| **Data Poisoning (`AML.T0020`)** | Malicious data alters model behaviour during training. | Tampering |
| **Backdoor ML Model (`AML.T0018`)** | Hidden triggers cause malicious behaviour while ordinary inputs appear normal. | Tampering |
| **Evade ML Model (`AML.T0015`)** | Adversarial inputs cause a model to miss or misclassify content. | Spoofing / Tampering / EoP |
| **Extract ML Model (`AML.T0024`)** | Queries or access are used to reproduce or steal model capability. | Information Disclosure |
| **LLM Prompt Injection (`AML.T0051`)** | Direct or indirect instructions alter an LLM application's intended behaviour. | Tampering / EoP |

### STRIDE + ATLAS workflow

1. Use STRIDE on every component and trust boundary to identify what can go wrong.
2. Search ATLAS for techniques that describe how each threat could be performed.
3. Record relevant technique IDs, prerequisites, evidence, and mitigations.
4. Use ATLAS case studies to test whether the scenario is realistic.
5. Verify identifiers against the live ATLAS site before publishing a formal report.

**MegaCorp example:** STRIDE identifies Tampering in the fraud model's training pipeline. ATLAS enriches the finding with Data Poisoning, turning a broad risk into a concrete scenario with prerequisites and defensive measures such as provenance tracking, anomaly detection, and drift monitoring.

---

## 6. OWASP Top 10 for LLM Applications (2025)

OWASP provides an application-focused view of major LLM risks. Component mapping makes the list actionable.

| ID | Risk | Common Locations |
| --- | --- | --- |
| **LLM01** | Prompt Injection | User input, model endpoint, RAG content, memory, tool output |
| **LLM02** | Sensitive Information Disclosure | Training data, prompts, retrieved context, model output |
| **LLM03** | Supply Chain | Base models, datasets, dependencies, plugins, registries |
| **LLM04** | Data and Model Poisoning | Training pipeline, feedback loop, feature store, registry |
| **LLM05** | Improper Output Handling | Frontend, APIs, shells, databases, downstream consumers |
| **LLM06** | Excessive Agency | Agent orchestrator, tools, service accounts, APIs |
| **LLM07** | System Prompt Leakage | Prompt configuration and inference interface |
| **LLM08** | Vector and Embedding Weaknesses | Vector store, embedding service, RAG pipeline |
| **LLM09** | Misinformation | Model output, stale knowledge, user-facing decisions |
| **LLM10** | Unbounded Consumption | Model endpoint, API gateway, tools, training jobs |

### Read the mapping in both directions

**Risk to component:** If assessing prompt injection, inspect every path that supplies text or content to the model, not only the chat box.

**Component to risk:** If adding a vector database, assess indirect prompt injection, vector and embedding weaknesses, access control, data provenance, and stale or misleading content.

### Component risk profiles

#### LLM inference endpoint

Primary concerns include prompt injection, sensitive information disclosure, improper output handling, excessive agency, prompt leakage, misinformation, and unbounded consumption.

#### Vector database and RAG pipeline

Primary concerns include malicious or unauthorised documents, retrieval manipulation, embedding weaknesses, cross-tenant access, stale sources, and indirect prompt injection.

#### Training pipeline

Primary concerns include sensitive-data ingestion, dataset provenance, poisoned data or labels, compromised base models, vulnerable dependencies, and artifact integrity.

#### Agent tools and orchestration

Primary concerns include excessive permissions, weak authorization, unsafe output-to-action chains, cross-tool escalation, secret exposure, and missing human approval.

---

## 7. Complete AI Threat-Modelling Workflow

### Step 1: Define scope and impact

Document the system's business purpose, users, deployment environment, consequential decisions, regulatory obligations, and unacceptable outcomes.

### Step 2: Draw the architecture and data flows

Include users, gateways, model endpoints, prompts, RAG sources, vector stores, memory, tools, feature stores, training infrastructure, registries, logs, and human reviewers. Mark trust boundaries.

### Step 3: Inventory assets

Record each asset's owner, sensitivity, source, location, consumers, retention, allowed modifications, and recovery path.

### Step 4: Identify threats with STRIDE-AI

Walk every component, data flow, and trust boundary through all six categories. Avoid forcing each threat into only one category.

### Step 5: Enrich with ATLAS

Map broad findings to relevant adversary techniques, prerequisites, mitigations, and case studies. Verify current IDs.

### Step 6: Map OWASP risks to components

Use the OWASP list to find coverage gaps and identify which components concentrate the most LLM risk.

### Step 7: Build abuse cases

Write short attacker stories:

> An attacker who controls **[entry point]** manipulates **[asset or component]** using **[technique]**, causing **[security impact]**.

Include attacker access, preconditions, trust-boundary crossing, affected asset, expected evidence, and business impact.

### Step 8: Score and prioritise

Consider:

1. Likelihood and required access.
2. Data sensitivity and financial impact.
3. Blast radius and persistence.
4. Detectability and reproducibility.
5. Existing controls and residual risk.
6. Human safety, legal, and compliance impact.

AI-specific persistence matters: poisoning that survives retraining may deserve higher priority than a visible, easily reversible failure.

### Step 9: Select controls

Use layered controls:

| Layer | Example Controls |
| --- | --- |
| **Data** | Provenance, validation, access control, deduplication, anomaly detection |
| **Model** | Adversarial evaluation, signing, versioning, safety testing, drift monitoring |
| **Application** | Input/output validation, secure prompt design, deterministic policy checks |
| **Retrieval** | Source allowlists, document ACLs, tenant isolation, freshness and integrity checks |
| **Tools** | Least privilege, scoped credentials, argument validation, approval gates |
| **Infrastructure** | Authentication, network isolation, secrets management, patching, rate limits |
| **Operations** | Logging, cost alerts, incident playbooks, rollback, periodic reassessment |

### Step 10: Document and revisit

Threat models are living documents. Reassess when the model, dataset, prompt, retrieval source, toolset, permissions, deployment architecture, or business purpose changes.

---

## 8. Structured Finding Template

Use this format for each identified threat:

```markdown
### Finding: [Short title]

- Component / trust boundary:
- Asset at risk:
- Threat actor and required access:
- STRIDE category:
- MITRE ATLAS technique:
- OWASP LLM risk:
- Attack path:
- Security and business impact:
- Existing controls:
- Evidence or assumptions:
- Likelihood:
- Impact:
- Priority:
- Recommended mitigations:
- Residual risk:
- Owner and target date:
```

### Example finding

```markdown
### Finding: Indirect prompt injection through the support knowledge base

- Component / trust boundary: Document ingestion -> vector store -> chatbot
- Asset at risk: Customer data and order-management tools
- Threat actor and required access: User able to submit content that enters the knowledge base
- STRIDE category: Tampering; Elevation of Privilege
- MITRE ATLAS technique: LLM Prompt Injection (verify current technique ID)
- OWASP LLM risk: LLM01 Prompt Injection; LLM06 Excessive Agency
- Attack path: A malicious document is indexed, retrieved for a later user's query, and instructs the model to invoke an overprivileged tool.
- Impact: Unauthorised data access or actions performed through the chatbot
- Recommended mitigations: Restrict ingestion, preserve source identity, scan and isolate untrusted content, enforce tool authorization outside the model, require approval for sensitive actions, and monitor retrieval-to-tool chains.
```

---

## 9. Common Mistakes

1. Treating the system prompt as a secret boundary or access-control mechanism.
2. Assessing only the model endpoint and ignoring data, RAG, memory, tools, and registries.
3. Trusting model output as safe code, HTML, SQL, shell input, or tool arguments.
4. Giving an agent broad permissions because the prompt tells it to behave safely.
5. Logging too little to reproduce decisions—or logging secrets and personal data without controls.
6. Testing only ordinary accuracy and missing adversarial triggers or harmful edge cases.
7. Using STRIDE, ATLAS, or OWASP as isolated checklists instead of connected layers.
8. Copying framework counts or technique IDs without verifying the current official source.

---

## Key Takeaways

1. AI threat modelling must cover data, models, prompts, embeddings, memory, tools, people, and infrastructure.
2. STRIDE identifies the broad threat category; ATLAS describes adversary techniques; OWASP maps major LLM risks to application components.
3. The highest-risk areas are often trust boundaries: data ingestion, RAG retrieval, registry-to-deployment, and model-to-tool execution.
4. Model instructions are not authorization. Enforce permissions with deterministic controls outside the model.
5. Record enough execution context to investigate decisions while protecting sensitive data in logs.
6. Revisit the assessment whenever the system learns, retrieves new data, gains a tool, or receives broader permissions.

---

## References

- [TryHackMe: Threat Modelling](https://tryhackme.com/room/threatmodelling)
- [TryHackMe: AI/ML Security Threats](https://tryhackme.com/room/aimlsecuritythreats)
- [MITRE ATLAS](https://atlas.mitre.org/)
- [MITRE ATLAS Matrix](https://atlas.mitre.org/matrices/ATLAS)
- [MITRE ATLAS Case Studies](https://atlas.mitre.org/studies)
- [OWASP GenAI Security Project](https://genai.owasp.org/)
- [Related: LLM Pentesting](llm-pentesting.md)
