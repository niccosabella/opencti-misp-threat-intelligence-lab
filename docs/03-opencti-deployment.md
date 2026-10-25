# OpenCTI Deployment and Configuration

## 1. Overview

This document outlines the deployment, configuration, optimization, and validation of OpenCTI within a self-hosted Cyber Threat Intelligence (CTI) laboratory.

OpenCTI was deployed on a dedicated Ubuntu Server virtual machine hosted through Proxmox Virtual Environment. Docker Compose was used to manage the OpenCTI application, its supporting services, background workers, and intelligence connectors.

The deployment provides a centralized environment for collecting, enriching, correlating, and analyzing threat intelligence from external sources and an independently deployed MISP instance.

The objective was to establish an operational intelligence platform while maintaining resource efficiency, restricted administrative access, and flexibility for future expansion.

---

## 2. Deployment Objectives

The primary objectives were to:

- Deploy OpenCTI within a dedicated virtualized Linux environment.
- Configure Docker-based application services and dependencies.
- Establish persistent intelligence storage and indexing.
- Enable asynchronous intelligence processing.
- Implement secure administrative and API authentication.
- Restrict direct access to the application interface.
- Integrate external collection and enrichment services.
- Optimize application performance for available system resources.
- Validate successful intelligence ingestion and analysis workflows.

---

## 3. Infrastructure and Technology Stack

### Virtualized Environment

OpenCTI operates on a dedicated Ubuntu Server virtual machine within the Proxmox environment.

Separating OpenCTI from MISP allows each platform to be administered, monitored, and maintained independently.

OpenCTI was allocated additional memory to support its indexing, message processing, and intelligence analysis workloads.

### Supporting Services

| Component | Technology | Purpose |
|---|---|---|
| Virtualization | Proxmox VE | Virtual machine hosting |
| Operating system | Ubuntu Server | Container host |
| Containerization | Docker | Application deployment |
| Orchestration | Docker Compose | Service lifecycle management |
| CTI platform | OpenCTI | Intelligence analysis and knowledge management |
| Search and indexing | Elasticsearch | Intelligence indexing and retrieval |
| Cache | Redis | Supporting application operations |
| Messaging | RabbitMQ | Asynchronous task distribution |
| Object storage | MinIO-compatible storage | Files and stored objects |
| Background processing | OpenCTI Workers | Intelligence ingestion and processing |
| Integrations | OpenCTI Connectors | External intelligence collection and enrichment |

The supporting services run within the OpenCTI application environment and are managed through Docker Compose.

---

## 4. Operating System Preparation

Following Ubuntu Server installation, the operating system was updated and prepared for containerized application workloads.

Example:

```bash
sudo apt update
sudo apt upgrade -y
```

Docker Engine and the Docker Compose plugin were installed and verified.

```bash
sudo systemctl status docker
docker compose version
```

The system was also configured to accommodate Elasticsearch memory-mapping requirements.

### Elasticsearch Host Configuration

Elasticsearch requires an appropriate virtual memory mapping limit.

The laboratory configuration used:

```bash
sudo sysctl -w vm.max_map_count=1048576
```

The setting was made persistent through the host's system configuration.

Verification:

```bash
sysctl vm.max_map_count
```

This host-level configuration is important because Elasticsearch runs inside a container but still depends on Linux kernel resources provided by the underlying virtual machine.

---

## 5. OpenCTI Application Deployment

### 5.1 Docker Deployment

The OpenCTI Docker deployment was obtained from the official OpenCTI Docker repository.

Reference:

https://github.com/OpenCTI-Platform/docker

Example:

```bash
git clone https://github.com/OpenCTI-Platform/docker.git
cd docker
```

The repository provides Docker Compose definitions for the OpenCTI platform and supporting components.

### 5.2 Environment Configuration

OpenCTI requires environment-specific configuration for application authentication, service credentials, storage, messaging, and connector operation.

Sensitive values were maintained in environment configuration and were not included directly in published documentation.

A simplified example demonstrates the approach:

```dotenv
OPENCTI_ADMIN_EMAIL=admin@example.test
OPENCTI_ADMIN_PASSWORD=<SET_A_UNIQUE_SECRET>
OPENCTI_ADMIN_TOKEN=<SET_A_UNIQUE_TOKEN>
OPENCTI_ENCRYPTION_KEY=<SET_A_UNIQUE_ENCRYPTION_KEY>
```

The values above are placeholders and do not represent operational credentials.

The complete environment configuration also includes credentials and connection settings for dependent services.

### 5.3 Container Startup

Application services were deployed through Docker Compose.

Example:

```bash
sudo docker compose config -q
sudo docker compose up -d
```

The configuration was validated before deployment.

After startup, the operational state of the services was examined:

```bash
sudo docker compose ps
```

Application logs were used to identify startup errors and dependency failures:

```bash
sudo docker compose logs --tail 50 opencti
```

The deployment was considered functional only after OpenCTI and its required dependencies were operational.

---

## 6. Supporting Services

### 6.1 Elasticsearch

Elasticsearch provides indexing and search functionality for OpenCTI intelligence objects.

Because indexing can be memory-intensive, Elasticsearch was a primary consideration during capacity planning.

The deployment used a dedicated Java heap allocation and a single-node configuration appropriate for the laboratory.

A sanitized example of the memory setting is:

```dotenv
ELASTIC_MEMORY_SIZE=2G
```

This value reflects an application resource configuration rather than a required setting for every deployment.

### 6.2 Redis

Redis provides supporting application services used by the OpenCTI environment.

Its health and resource consumption were monitored through Docker.

### 6.3 RabbitMQ

RabbitMQ supports asynchronous communication and task processing.

Connectors send intelligence data through OpenCTI's processing workflow, allowing background workers to handle imported information.

This separation is particularly important when importing larger intelligence datasets.

### 6.4 Object Storage

An S3-compatible object storage service provides storage for applicable files and supporting objects.

Persistent Docker volumes were used to preserve application data across container restarts and recreation.

### 6.5 OpenCTI Workers

OpenCTI workers process background tasks associated with intelligence ingestion and platform operations.

The deployment included multiple workers to support asynchronous processing.

Worker operation was evaluated alongside RabbitMQ activity and connector logs to distinguish successful task submission from completed data ingestion.

---

## 7. Application Access and Security

### 7.1 Restricted Web Access

The OpenCTI application was not intentionally exposed as a generally accessible service.

Instead, access was restricted to the application host's loopback interface.

A sanitized example of the Docker port binding is:

```yaml
ports:
  - "127.0.0.1:8080:8080"
```

This limits direct access through the host's network interfaces.

### 7.2 Administrative Access

Administrative access was performed through an authenticated SSH connection and local port forwarding.

Example:

```bash
ssh -N -L 8080:127.0.0.1:8080 admin@opencti.example.test
```

The application could then be accessed through:

```text
http://localhost:8080
```

The SSH hostname and username are documentation placeholders.

Although the forwarded browser connection uses HTTP on localhost, communication between the management workstation and the Linux host is carried through the encrypted SSH session.

### 7.3 Authentication and Tokens

The deployment uses authentication for administrative and connector operations.

Where applicable, dedicated connector service accounts and API tokens were configured instead of sharing administrator credentials across integrations.

This approach supports better separation of access, simplifies credential rotation, and improves accountability.

Some connectors inherited administrator-token settings from the original deployment configuration. Moving remaining integrations to dedicated least-privilege identities is an ongoing security improvement.

### 7.4 Secrets Management

Sensitive values such as passwords, encryption keys, and connector API tokens were maintained in local environment files.

These files are excluded from the public documentation.

Docker Compose examples published in this repository reference environment variables rather than embedding secret values.

---

## 8. Resource Optimization

### 8.1 Resource Constraints

OpenCTI uses multiple supporting services and can become resource-intensive as intelligence collections grow.

The deployment was optimized for the available virtual machine resources instead of attempting to start every optional component.

### 8.2 Application Memory

The application runtime was configured with an explicit Node.js heap limit.

The configuration was reduced from the more generous default provided by the Docker deployment to better match laboratory resource availability.

A representative setting is:

```yaml
environment:
  - NODE_OPTIONS=--max-old-space-size=4096
```

This setting controls the maximum V8 heap size; it does not represent the total memory limit of the container.

### 8.3 Elasticsearch Memory

Elasticsearch memory allocation was adjusted to reduce resource pressure while supporting the laboratory's indexing workload.

The application's memory footprint was evaluated using both guest-level and container-level monitoring.

### 8.4 Optional Components

Optional platform services that were not required for the laboratory's immediate objectives were left disabled.

This reduced unnecessary resource consumption and allowed compute capacity to be reserved for intelligence ingestion and analysis.

### 8.5 Runtime Monitoring

Memory availability was monitored using:

```bash
free -h
```

Container-level resource utilization was monitored with:

```bash
sudo docker stats --no-stream
```

The checks showed healthy available memory and no active swap usage during the observed validation period.

The largest memory consumer was Elasticsearch, which was consistent with its indexing responsibilities.

---

## 9. Intelligence Connector Architecture

OpenCTI uses connectors to import external intelligence and enrich existing entities.

The laboratory supports both external intelligence collection and analyst-initiated enrichment.

### External Intelligence Collection

External import connectors retrieve intelligence from supported feeds, repositories, or platforms.

Examples include:

- MISP
- MITRE ATT&CK
- AlienVault OTX
- CISA Known Exploited Vulnerabilities

These integrations allow OpenCTI to import indicators, observables, vulnerabilities, reports, and adversary context.

### Intelligence Enrichment

Enrichment connectors provide additional context for existing intelligence objects.

Examples include:

- VirusTotal
- urlscan.io
- AbuseIPDB

Where appropriate, automatic enrichment was disabled to manage external API usage and reduce unnecessary processing.

Manual enrichment also permits an analyst to select specific observables for further investigation.

### Modular Connector Configuration

Additional integrations were maintained in separate Docker Compose configuration files.

This approach allowed connectors to be deployed and recreated independently of the core OpenCTI application.

Example:

```bash
sudo docker compose \
  -f docker-compose.yml \
  -f docker-compose.example-connector.yml \
  up -d --no-deps connector-example
```

This example illustrates the deployment pattern and is not intended to run without a corresponding connector configuration.

Separating optional connectors from the core stack simplified troubleshooting and reduced the risk of unintended changes to critical services.

---

## 10. Troubleshooting and Problem Resolution

### 10.1 Application Startup and Dependencies

**Problem**

The platform depended on multiple services being operational before OpenCTI could become healthy.

**Investigation**

Docker service status, dependency health checks, and application logs were reviewed.

**Resolution**

The required supporting services were configured and started through Docker Compose. Dependency health was verified before troubleshooting the application itself.

**Verification**

The OpenCTI interface became accessible and its background services operated successfully.

**Lesson Learned**

Container startup does not necessarily indicate application readiness. Health checks and dependency validation are essential.

---

### 10.2 Elasticsearch Resource Requirements

**Problem**

Elasticsearch required additional host-level configuration and careful memory planning.

**Investigation**

The Linux virtual memory mapping limit and available system memory were reviewed.

**Resolution**

The host's memory-mapping setting was adjusted and persisted. Elasticsearch memory allocation was configured to match the laboratory's resource constraints.

**Verification**

Elasticsearch remained operational, and OpenCTI successfully processed imported intelligence.

**Lesson Learned**

Containerized applications still rely on host kernel settings and physical resources. Proper sizing and host preparation are necessary for stable operation.

---

### 10.3 Administrative Access Recovery

**Problem**

An administrative account access issue temporarily prevented normal sign-in to OpenCTI.

**Investigation**

Application availability and authentication behavior were reviewed to determine whether the issue involved the platform or the account.

**Resolution**

The affected application service was restarted as part of the recovery process.

**Verification**

Administrative authentication was restored and platform access was confirmed.

**Lesson Learned**

Administrative identity management should be approached carefully. Recovery procedures and alternative authorized access methods are important for maintaining access to critical infrastructure.

---

### 10.4 Connector Initialization Failure

**Problem**

An intelligence connector encountered an exception during startup while attempting automatic service account handling.

The observed exception included:

```text
KeyError: 'name'
```

**Investigation**

The connector logs and service account creation settings were reviewed.

**Resolution**

Automatic service account creation was disabled for the affected connector, allowing it to use the configured OpenCTI authentication token.

**Verification**

The connector subsequently started, registered with OpenCTI, and imported intelligence objects.

**Lesson Learned**

Connector compatibility should be evaluated against the platform version and authentication settings. A valid Docker configuration does not guarantee successful connector initialization.

---

### 10.5 External Intelligence Import Timestamp Error

**Problem**

An external intelligence connector failed during its initial import because of a timestamp parsing error.

The error included:

```text
ValueError: unconverted data remains: Z
```

**Investigation**

The connector's configured historical import timestamp was compared with the datetime format accepted by its parser.

**Root Cause**

The configured timestamp contained a trailing UTC timezone designator that the connector's date parser did not accept.

**Resolution**

The timestamp format was updated to match the connector's expected input format.

**Verification**

The connector successfully retrieved and processed intelligence after its configuration was corrected.

**Lesson Learned**

Connector-specific configuration schemas and datetime parsing expectations must be verified rather than assumed.

---

## 11. Deployment Validation

### 11.1 Platform Health

The Docker environment was checked for running and healthy application services.

Example:

```bash
sudo docker ps
```

### 11.2 Application Authentication

Authorized browser access was tested through the restricted management connection.

The administrator authenticated and confirmed access to the OpenCTI interface.

### 11.3 Connector Registration

The OpenCTI integration interface was used to verify that configured connectors registered with the platform.

Connector health and activity were also reviewed through application logs.

### 11.4 Intelligence Ingestion

Imported intelligence from supported external sources was located in the OpenCTI interface.

Validation included:

- Imported threat reports.
- Indicators and observables.
- ATT&CK-related intelligence.
- Vulnerability records entering the ingestion pipeline.
- Connector execution and processing logs.

The difference between data being submitted to the processing queue and data being fully available in OpenCTI was considered during verification.

### 11.5 Intelligence Enrichment

Manual enrichment was tested against selected observables.

The resulting operations demonstrated the platform's ability to request external intelligence and associate enrichment activity with existing objects.

Where enrichment returned limited or no reputation data, the results were assessed cautiously rather than treated as proof of benign activity.

### 11.6 Resource Monitoring

System-level and container-level resource statistics were reviewed after the integrations were deployed.

The system retained available memory and demonstrated low CPU utilization during the observed monitoring period.

These findings established a useful performance baseline but do not guarantee resource sufficiency under future workloads or larger imports.

---

## 12. Results

The OpenCTI deployment established a functional, containerized intelligence platform capable of supporting collection, enrichment, correlation, and reporting.

The environment demonstrated:

- Successful deployment of OpenCTI and required supporting services.
- Persistent intelligence storage and search functionality.
- Asynchronous background processing.
- Restricted administrative access through SSH forwarding.
- Integration with MISP and external intelligence sources.
- Manual enrichment of selected observables.
- Successful import and review of structured intelligence objects.
- Operational resource monitoring and troubleshooting.

The platform supports continued development of intelligence analysis workflows and further integration as laboratory requirements evolve.

---

## 13. Lessons Learned

**OpenCTI is an ecosystem rather than a single application.** Its effective operation depends on search infrastructure, object storage, messaging, background workers, and connector services.

**System health requires multiple layers of validation.** An active container is not the same as a functioning connector, and a completed connector task does not necessarily mean every object has finished indexing.

**Resource management is an ongoing responsibility.** Elasticsearch and intelligence ingestion workloads must be monitored as data volume increases.

**Modular deployment improves maintainability.** Separate Compose configurations help isolate optional integrations and simplify individual connector troubleshooting.

**Authentication choices influence operational security.** Dedicated connector identities and appropriate token permissions provide better separation than widespread use of administrator credentials.

**Intelligence quality matters as much as successful ingestion.** OpenCTI can organize and enrich data, but analysts must still evaluate source reliability, confidence, context, and uncertainty.

---

## 14. Future Improvements

Potential improvements include:

- Implementing more consistent least-privilege connector authentication.
- Improving centralized connector monitoring and error reporting.
- Evaluating backup and recovery procedures for persistent application data.
- Establishing repeatable procedures for platform updates.
- Monitoring Elasticsearch storage growth and retention requirements.
- Expanding intelligence collection and analysis workflows.
- Reviewing trusted TLS deployment and secure integration practices.
- Maintaining version-aware documentation for future platform upgrades.

---

## 15. References

- [OpenCTI Documentation](https://docs.opencti.io/)
- [Official OpenCTI Docker Repository](https://github.com/OpenCTI-Platform/docker)
- [OpenCTI Connectors](https://github.com/OpenCTI-Platform/connectors)
- [Docker Documentation](https://docs.docker.com/)
- [Elasticsearch Documentation](https://www.elastic.co/guide/)
- [Proxmox Virtual Environment](https://www.proxmox.com/)
