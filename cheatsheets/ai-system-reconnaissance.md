# AI System Reconnaissance Cheat Sheet

> Authorized discovery and enumeration only. Treat model metadata, notebooks, vector schemas, and artifact paths as sensitive.

## Workflow

~~~text
Scope → Passive leads → Port/protocol discovery → Fingerprint
→ Enumerate metadata → Map relationships → Threat-model handoff
~~~

## Common Indicators

| Component | Common ports | Quick checks |
|---|---:|---|
| Triton | 8000/8001/8002 | /v2/health/ready, /v2/models, /metrics |
| TensorFlow Serving | 8500/8501 | /v1/models/<name> |
| TorchServe | 8080/8081/8082 | /ping, /models, /metrics |
| Ollama | 11434 | /api/tags |
| vLLM/OpenAI-compatible | Often 8000 | /v1/models |
| MLflow | Often 5000 | /api/2.0/mlflow/... |
| Ray | Often 8265 | Dashboard/job routes |
| Qdrant | 6333/6334 | /collections |
| Weaviate | 8080/50051 | /v1/meta, collection/schema APIs |
| Milvus | Often 19530 | gRPC behavior |
| Jupyter | Often 8888 | /api/kernels, /api/contents |
| MinIO | Often 9000/9001 | S3/service responses |

**Ports are clues, not identification. Confirm with multiple signals.**

## Targeted Discovery

~~~bash
nmap -sV --script=http-title,http-headers   -p 5000,6333,6334,8000-8002,8080-8082,8265,8500,8501,8888,9000,9001,11434,19530,50051   TARGET
~~~

Also check 80/443 and approved non-default ranges.

## Low-Impact HTTP Checks

~~~bash
curl -i http://HOST:PORT/
curl -i http://HOST:PORT/v1/models
curl -i http://HOST:PORT/v2/models
curl -i http://HOST:PORT/collections
curl -i http://HOST:PORT/openapi.json
curl -i http://HOST:PORT/metrics
~~~

## AI Endpoint Wordlist

Files:

- [ai_wordlist.txt](wordlists/ai_wordlist.txt) — HTTP paths, one per line
- [ai_grpc_services.txt](wordlists/ai_grpc_services.txt) — grpcurl service names

~~~bash
ffuf -w cheatsheets/wordlists/ai_wordlist.txt \
  -u http://HOST:PORTFUZZ \
  -mc all -fc 404

feroxbuster -u http://HOST:PORT \
  -w cheatsheets/wordlists/ai_wordlist.txt
~~~

Baseline a random path before filtering. Status 200 alone does not prove a route exists.

For model-specific paths, replace discovered names deliberately:

~~~bash
MODEL="discovered-model"
curl -i "http://HOST:PORT/v2/models/$MODEL/config"
curl -i "http://HOST:PORT/v1/models/$MODEL"
~~~

Do not place gRPC service names in the HTTP wordlist.

## gRPC

~~~bash
grpcurl -plaintext HOST:PORT list
grpcurl -plaintext HOST:PORT describe SERVICE
~~~

Use only when gRPC enumeration and reflection queries are authorized.

## Fingerprint Signals

- Headers: server, framework, request IDs, content type
- JSON: model objects, versions, tensor fields, collections
- Paths: predict, infer, embeddings, models, experiments, runs
- Errors: validation fields and framework namespaces
- Docs: OpenAPI/Swagger and GraphQL schema
- Metrics: model names, versions, latency, GPU/resource data
- TLS: certificates, gateway identity, automated-client patterns

Require at least two supporting signals.

## Enumerate

**Tracking/registry**
- Experiments and runs
- Registered models and versions
- Lifecycle/stage
- Artifact URI hostnames
- Contributor and Git metadata

**Inference**
- Readiness
- Model/version inventory
- Backend/platform
- Inputs, outputs, shapes, data types
- Batch/resource limits

**Vector store**
- Collections/classes
- Dimensions and distance metric
- Payload/property schema
- Record count
- Modules/vectorizer
- Tenant/access controls

**Notebook/orchestration**
- Authentication state
- Kernel/notebook/job inventory
- Pipeline/deployment metadata
- Storage and service-account relationships

## Map Connections

~~~text
Notebook → MLflow/registry → S3 or MinIO
Application → embedding service → vector store
CI/CD → model hub → registry → inference server
Prometheus → AI services
~~~

Record authentication, credential type, data flow, trust boundary, owner, and evidence.

## Stop Before

- Reading notebook cells without explicit permission
- Downloading weights, artifacts, embeddings, or datasets
- Using discovered credentials or tokens
- Submitting jobs or loading/unloading models
- Modifying collections or registries
- Querying third-party systems
- Triggering costly inference or instability

## Defender Indicators

- AI-port sequence scans
- Bursts to /v1/models or /v2/models
- Raw MLflow API access without normal UI behavior
- /metrics requests outside monitoring CIDRs
- gRPC reflection from unusual hosts
- Jupyter API access without expected sessions
- Repeated malformed tensor requests
- OpenAPI/schema enumeration from unknown sources

## Inventory Row

| Asset | Host:port | Component | Confidence | Auth | Exposure | Sensitive metadata | Owner | Evidence |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |

## Key Rule

Discovery proves what exists. Threat modelling decides what can go wrong. Exploitation requires separate authorization.

## Related

- [Full AI Recon Note](../knowledge-base/ai-security/ai-system-reconnaissance.md)
- [AI Recon Methodology](../pentesting-methodology/reconnaissance/ai-system-reconnaissance-methodology.md)
- [AI Threat Modelling Cheat Sheet](ai-threat-modelling.md)
