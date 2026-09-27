# TryHackMe — AI System Reconnaissance

> Clean room-specific study note. Reusable material lives in [AI System Reconnaissance](../../../knowledge-base/ai-security/ai-system-reconnaissance.md), the [methodology](../../../pentesting-methodology/reconnaissance/ai-system-reconnaissance-methodology.md), and the [cheat sheet](../../../cheatsheets/ai-system-reconnaissance.md).

## Purpose

AI threat modelling reasons about what could go wrong. AI reconnaissance establishes what AI/ML components actually exist, where they are reachable, what metadata they expose, and how they connect.

The Cyphira scenario is discovery-focused: find and map systems without breaking or modifying them.

## Learning Objectives

- Recognize production AI/ML components and their common protocols.
- Fingerprint AI services through multiple response signals.
- Enumerate authorized model, experiment, vector, and deployment metadata.
- Connect individual components into an attack-surface map.
- Apply a repeatable reconnaissance workflow.
- Recognize the defensive signatures of AI-focused reconnaissance.

## AI Infrastructure

A production AI environment may contain:

- model-serving endpoints;
- experiment trackers and registries;
- orchestration and distributed-compute platforms;
- vector databases;
- notebooks;
- artifact/object storage;
- model hubs, package sources, and container registries;
- metrics and debugging interfaces.

These systems create internal data flows that are as important as external exposure.

## Common Port Indicators

| Component | Common ports | Recon indicators |
|---|---:|---|
| Triton | 8000, 8001, 8002 | HTTP, gRPC, models, readiness, metrics |
| TensorFlow Serving | 8500, 8501 | gRPC and model HTTP API |
| TorchServe | 8080, 8081, 8082 | Inference, management, metrics |
| Ollama | 11434 | Model tags and runtime API |
| MLflow | 5000 | Experiments, runs, registry |
| Ray | 8265 | Dashboard and job API |
| Qdrant | 6333, 6334 | Collections and vector metadata |
| Weaviate | 8080, 50051 | REST/gRPC metadata and collections |
| Jupyter | 8888 | Kernels and contents API |
| MinIO | 9000, 9001 | S3-compatible storage |

Defaults vary. An open port is a lead, not proof of a product.

## Fingerprinting

Standard service detection may report only generic HTTP or gRPC. Confirm identity by combining:

1. response headers;
2. JSON or protobuf structure;
3. endpoint naming;
4. safe validation errors;
5. documentation or metrics routes;
6. TLS and proxy context.

Examples include model objects on /v1/models, Triton-style /v2 routes, MLflow API namespaces, vector collection metadata, or Jupyter kernel APIs.

A Uvicorn header identifies an ASGI server, not necessarily an AI application. Framework attribution needs more than one signal.

## gRPC

HTTP scanners may miss binary gRPC services. When authorized:

~~~bash
grpcurl -plaintext HOST:PORT list
grpcurl -plaintext HOST:PORT describe SERVICE
~~~

Reflection can expose complete service schemas. Treat that output as sensitive metadata.

## Enumeration

### MLflow and Registries

Potential metadata includes experiment names, runs, registered models, versions, lifecycle state, artifact URI hostnames, contributor IDs, and source-control tags. Use current API documentation because routes differ across versions.

### Inference Servers

Model configuration may disclose model names, versions, backend frameworks, input/output fields, tensor shapes, data types, and batching limits.

### Vector Databases

Collection and schema endpoints may reveal names, dimensions, distance metrics, properties, record counts, and vectorizer configuration. Collection names can reveal business context.

### Metrics and Debugging

Metrics, OpenAPI documents, GraphQL schemas, and verbose errors may disclose topology, model names, resource utilization, installed modules, or internal paths.

### Jupyter

Kernel and contents APIs can reveal activity and files. Notebook-cell access is higher-impact because cells may contain code, credentials, and proprietary data; obtain explicit authorization before reading them.

## Attack-Surface Mapping

Connect the discoveries:

~~~text
Notebook → tracking server → model registry → object storage
Application → inference/embedding service → vector database
Pipeline → external model/package source → registry → production
Metrics collector → model and orchestration services
~~~

Record protocols, authentication, trust boundaries, credentials, data types, owners, and evidence.

## Structured Methodology

1. **Passive discovery:** approved internet search, repositories, papers, job postings, registries.
2. **Active scanning:** AI-specific and gateway ports at safe rates.
3. **Fingerprinting:** multiple framework indicators.
4. **Metadata enumeration:** minimum necessary inventory data.
5. **Relationship and supply-chain mapping:** data flows and dependencies.
6. **Analysis and handoff:** asset map, observations, gaps, detection, and threat modelling.

## Detection Perspective

Defenders can monitor for:

- scans targeting recognizable AI-port combinations;
- sequential model or collection enumeration;
- raw tracking API access;
- metrics access outside monitoring networks;
- gRPC reflection from unusual sources;
- notebook APIs without expected sessions;
- repeated malformed model requests;
- bursts against model, docs, schema, and metrics endpoints.

Legitimate monitoring and automation can look similar, so detections need asset ownership and traffic baselines.

## Hardening Priorities

- Keep management, notebook, registry, and metrics interfaces off untrusted networks.
- Require authentication and least privilege.
- Segment development, training, registry, artifact, and production systems.
- Remove secrets from notebooks and repositories.
- Scope and rotate model-hub and storage credentials.
- Disable unnecessary reflection, debug output, and documentation routes.
- Log and alert on sensitive metadata APIs.
- Inventory third-party models, packages, images, and datasets.

## Framework Notes

The room connects activity to MITRE ATLAS, MITRE ATT&CK, NIST AI RMF, and NIST CSF. Technique catalogs evolve and the supplied room material contains conflicting ATLAS identifiers, so verify current IDs before using them in a professional report.

## Key Takeaway

A list of open ports is not an AI attack-surface map. The useful output is a verified inventory showing which services exist, what they expose, and how models, data, credentials, storage, and orchestration connect.
