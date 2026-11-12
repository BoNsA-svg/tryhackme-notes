# AI System Reconnaissance

## Overview

AI system reconnaissance is the authorized discovery, identification, enumeration, and mapping of AI/ML infrastructure. It confirms what is actually deployed and exposed so that threat models and test plans are based on evidence rather than assumptions.

Traditional reconnaissance remains relevant, but AI deployments add unfamiliar services:

- inference and embedding endpoints;
- experiment trackers and model registries;
- vector databases;
- notebooks and training platforms;
- model artifact storage;
- orchestration and distributed-compute dashboards;
- metrics, tracing, and evaluation services;
- external model hubs and package dependencies.

> Use these techniques only against assets covered by written authorization. Metadata can contain credentials, personal data, proprietary model information, and internal storage locations.

## Reconnaissance vs Threat Modelling

| Activity | Question | Output |
|---|---|---|
| AI reconnaissance | What is deployed, reachable, and exposed? | Verified asset and data-flow inventory |
| AI threat modelling | What could go wrong and how should it be controlled? | Prioritized threats and mitigations |
| AI security testing | Can a specific weakness be safely validated? | Evidence-backed findings |

Reconnaissance feeds the other two activities.

## AI Infrastructure Stack

### Model Serving

Inference servers load models and expose prediction APIs. They may publish model names, versions, input schemas, backends, health status, and performance metrics.

Common examples include NVIDIA Triton, TensorFlow Serving, TorchServe, Ollama, vLLM, and custom FastAPI services.

### Experiment Tracking and Registries

Tracking systems record experiments, parameters, metrics, runs, artifacts, lineage, and model lifecycle state. Registries may expose:

- model inventories and versions;
- staging or production labels;
- run and artifact identifiers;
- internal object-storage paths;
- contributor identities;
- source-control references.

MLflow is a common example. Its API and behavior vary by version and deployment configuration.

### Vector Databases

Vector stores support semantic search and retrieval-augmented generation. Collection or schema metadata may reveal:

- collection names;
- vector dimensions and distance metrics;
- payload/property names;
- installed vectorizer modules;
- record counts;
- the business purpose of indexed data.

Examples include Qdrant, Weaviate, Milvus, and Chroma.

### Notebook and Orchestration Platforms

Jupyter, Kubeflow, Ray, and managed ML environments often connect to many parts of the AI stack. They may reveal notebook metadata, running kernels, jobs, pipeline definitions, service accounts, storage paths, and configuration.

### Artifact Storage and Observability

Object stores such as MinIO may hold weights, datasets, checkpoints, and artifacts. Prometheus-style metrics can reveal model names, load, latency, GPU utilization, and deployment topology.

## Port and Protocol Indicators

Ports are leads, not proof. Reverse proxies, containers, managed services, and custom configuration frequently change them.

| Component | Common/default ports | Protocol | Useful low-impact indicators |
|---|---:|---|---|
| NVIDIA Triton | 8000, 8001, 8002 | HTTP, gRPC, Prometheus | /v2/health/ready, /v2/models, /metrics |
| TensorFlow Serving | 8500, 8501 | gRPC, HTTP | /v1/models/<name> |
| TorchServe | 8080, 8081, 8082 | HTTP | /ping, /models, /metrics |
| Ollama | 11434 | HTTP | /api/tags, /api/show |
| vLLM/OpenAI-compatible service | Often 8000 | HTTP | /v1/models |
| MLflow | Often 5000 | HTTP | /api/2.0/mlflow/... |
| Kubeflow | Often 80/443 via ingress | HTTP | Pipeline and dashboard routes |
| Ray Dashboard | Often 8265 | HTTP | Dashboard and job routes |
| Qdrant | 6333, 6334 | HTTP, gRPC | /collections |
| Weaviate | 8080, 50051 | HTTP, gRPC | /v1/meta and schema/collection APIs |
| Milvus | Often 19530 | gRPC | gRPC service behavior |
| Jupyter | Often 8888 | HTTP/WebSocket | /api/kernels, /api/contents |
| MinIO | Often 9000, 9001 | S3-compatible HTTP | Service and bucket responses |
| Prometheus-style metrics | Service-specific | HTTP | /metrics |

NVIDIA documents Triton’s common HTTP, gRPC, and metrics ports as 8000, 8001, and 8002. Qdrant and Weaviate also document REST/gRPC interfaces, but deployed ports and authentication must always be confirmed against the live environment.

## Fingerprinting

Use multiple signals because any single indicator can be altered by a proxy or custom wrapper.

### Headers

Look for:

- server/framework headers;
- request IDs;
- content types;
- gRPC status headers;
- metrics-specific headers;
- proxy and gateway fingerprints.

A header such as uvicorn suggests a Python ASGI service but does not by itself prove the application is an ML service.

### Response Structure

Inspect:

- model objects and identifiers;
- model version/status arrays;
- tensor names, dimensions, and data types;
- collection metadata;
- experiment, run, and artifact fields;
- health and readiness objects.

### Endpoint Vocabulary

AI APIs often use terms such as:

- predict, infer, generate, embeddings, score;
- models, versions, config, metadata;
- experiments, runs, artifacts;
- collections, schema, vector;
- kernels, pipelines, deployments;
- metrics, health, ready.

### Error Messages

Within scope, a harmless malformed request can reveal framework-specific validation fields or namespaces. Do not use error generation that risks expensive inference, service instability, or sensitive-data exposure. Record verbose stack traces as information disclosure and stop once identification is established.

### gRPC

If gRPC is in scope, grpcurl can confirm connectivity. Reflection, when enabled, may reveal service and message schemas:

~~~bash
grpcurl -plaintext HOST:PORT list
grpcurl -plaintext HOST:PORT describe SERVICE
~~~

Reflection output is sensitive API metadata. Enumerate only authorized services and preserve evidence.

## AI Endpoint Wordlist

A reusable, one-path-per-line HTTP wordlist is stored at:

- [ai_wordlist.txt](../../cheatsheets/wordlists/ai_wordlist.txt)
- [AI gRPC service names](../../cheatsheets/wordlists/ai_grpc_services.txt)

Use the leading-slash wordlist by placing FUZZ directly after the authority:

~~~bash
ffuf -w ../../cheatsheets/wordlists/ai_wordlist.txt \
  -u http://HOST:PORTFUZZ \
  -mc all -fc 404
~~~

With feroxbuster:

~~~bash
feroxbuster -u http://HOST:PORT \
  -w ../../cheatsheets/wordlists/ai_wordlist.txt
~~~

Establish a baseline first. Some applications return the same 200 response for every unknown path, so status code alone is not proof that an endpoint exists. Compare response length, words, headers, redirects, and body structure.

### Dynamic Model and Collection Routes

Literal placeholders such as <model> and <name> do not belong in a discovery wordlist. First discover an authorized model or collection name, then substitute it deliberately:

~~~bash
MODEL="discovered-model"
COLLECTION="discovered-collection"

curl -i "http://HOST:PORT/v2/models/$MODEL/config"
curl -i "http://HOST:PORT/v2/models/$MODEL/infer"
curl -i "http://HOST:PORT/v1/models/$MODEL"
curl -i "http://HOST:PORT/collections/$COLLECTION"
~~~

Inference routes may execute costly workloads or process sensitive data. A route check or safe metadata request is different from submitting inference. Follow the Rules of Engagement.

### Sensitive Conventional Paths

The wordlist includes paths such as /.env and /.git/config because AI services can inherit ordinary web deployment mistakes. If one returns sensitive content, capture the minimum necessary evidence, stop further retrieval, protect the evidence, and notify the client.

### gRPC Reference

The gRPC service-name file is for follow-up with grpcurl, not HTTP fuzzing:

~~~bash
grpcurl -plaintext HOST:PORT list
grpcurl -plaintext HOST:PORT describe inference.GRPCInferenceService
grpcurl -plaintext HOST:PORT describe tensorflow.serving.PredictionService
~~~

Reflection and schema enumeration must be explicitly authorized.

## Enumeration Targets

### MLflow

Depending on version and authorization, useful objects include:

- experiments;
- runs, parameters, metrics, and tags;
- registered models and model versions;
- artifact references and lineage.

Use the current MLflow REST API documentation rather than assuming older list/search routes behave identically.

### Inference Servers

Collect only necessary metadata:

- server and model identity;
- model versions and readiness;
- backend/platform;
- input/output names, shapes, and data types;
- batch limits;
- metrics exposure.

Avoid submitting real customer data or triggering expensive model jobs.

### Vector Stores

Document:

- collection/class names;
- vector dimensions and distance functions;
- payload/property schemas;
- point/object counts;
- vectorizer and module settings;
- authentication and tenant boundaries.

Do not retrieve vector contents or source documents unless the Rules of Engagement explicitly authorize data access.

### Notebooks

Notebook APIs and cells can expose secrets and proprietary code. First establish whether authentication is enforced. Reading notebook content is a higher-impact step and requires explicit authorization and data-handling controls.

### Metrics and Debug Interfaces

Check whether /metrics, /docs, /openapi.json, GraphQL introspection, debug modes, and verbose errors are reachable from untrusted networks. Treat exposed documentation and telemetry as intelligence, not automatically as a vulnerability; risk depends on sensitivity and access controls.

## From Findings to an Attack-Surface Map

Do not stop at a port list. Connect components through observed evidence:

~~~text
Notebook → experiment tracker → model registry → artifact store
Application → embedding service → vector database → source documents
CI/CD → model hub → registry → inference server
Metrics collector → model servers and orchestration platform
~~~

For every connection, record:

- source and destination;
- protocol and authentication;
- credential type;
- data or model artifacts transferred;
- trust boundary crossed;
- exposure and owner;
- evidence and confidence.

This reveals concentration points such as notebooks, registries, and service accounts that can bridge multiple systems.

## Supply-Chain Reconnaissance

Within authorization, identify:

- public model and dataset sources;
- model-hub organizations and repositories;
- container images and registries;
- package manifests and internal package names;
- CI/CD references;
- artifact buckets;
- tokens or credentials exposed in approved repositories.

Never attempt to register a package name, modify a model, use a discovered token, or access third-party resources without explicit written authorization.

## Defensive View

Suspicious patterns include:

- scans concentrated on AI-specific port groups;
- rapid model-listing or collection-listing requests;
- MLflow API calls without the normal UI request sequence;
- /metrics access from outside monitoring networks;
- gRPC reflection queries from unusual sources;
- Jupyter API access without expected sessions;
- bursts against /v1/models, /v2/models, /openapi.json, and /api/contents;
- verbose-error probes and repeated malformed tensor requests.

Baselining legitimate automation is essential because monitoring systems and ML clients may produce similar traffic.

## High-Value Controls

- Inventory AI/ML services and assign owners.
- Bind management interfaces to trusted networks.
- Require strong authentication and authorization.
- Segment notebooks, registries, stores, and inference systems.
- Restrict metrics and debug endpoints.
- Disable unnecessary gRPC reflection and documentation routes.
- Use scoped, short-lived service credentials.
- Remove secrets from notebooks and repositories.
- Apply least privilege to model hubs, object storage, and service accounts.
- Log model, registry, notebook, and vector-store API access.
- Review external model, dataset, package, and image provenance.

## Framework Mapping

AI reconnaissance can be described with:

- MITRE ATT&CK Reconnaissance and Network Service Scanning concepts;
- MITRE ATLAS reconnaissance and ML artifact/system-information discovery techniques;
- NIST AI RMF Map activities for identifying components, interactions, and third-party resources;
- NIST CSF Asset Management and Risk Assessment outcomes.

Framework names and technique IDs change. Verify them on the authoritative framework site before formal reporting.

## Related Notes

- [AI Threat Modelling](ai-threat-modelling.md)
- [LLM Pentesting](llm-pentesting.md)
- [AI System Reconnaissance Methodology](../../pentesting-methodology/reconnaissance/ai-system-reconnaissance-methodology.md)
- [AI System Reconnaissance Cheat Sheet](../../cheatsheets/ai-system-reconnaissance.md)
