# MISP Deployment and Configuration

## 1. Overview

This document details the deployment, configuration, troubleshooting, and validation of the Malware Information Sharing Platform (MISP) within a self-hosted Cyber Threat Intelligence (CTI) laboratory.

MISP was deployed as a containerized application on a dedicated Ubuntu Server virtual machine hosted through Proxmox Virtual Environment.

The platform serves as an intelligence collection and management component, enabling the ingestion, organization, and sharing of threat intelligence before its integration into OpenCTI.

The deployment focused on establishing a functional intelligence pipeline while implementing appropriate access restrictions and separating sensitive configuration from application documentation.

---

## 2. Deployment Objectives

The primary objectives of the MISP deployment were to:

- Establish a self-hosted threat intelligence collection platform.
- Deploy MISP and its supporting services using Docker Compose.
- Configure secure administrative access to the application.
- Implement consistent application URL and HTTPS configuration.
- Configure external intelligence feeds.
- Establish authenticated API connectivity between MISP and OpenCTI.
- Restrict unnecessary network access to the platform.
- Validate successful collection and transfer of intelligence data.

---

## 3. Infrastructure and Technology Stack

MISP operates within a dedicated Ubuntu Server virtual machine.

| Component | Technology | Purpose |
|---|---|---|
| Virtualization | Proxmox VE | Infrastructure hosting |
| Operating system | Ubuntu Server | Application host |
| Containerization | Docker | Application isolation and deployment |
| Orchestration | Docker Compose | Container lifecycle management |
| Intelligence platform | MISP | Event and indicator management |
| Database | MariaDB | Persistent application data |
| Cache | Redis-compatible service | Application caching |
| Web services | Nginx | HTTP and HTTPS handling |
| Supporting services | MISP Modules | Enrichment and supporting functionality |
| Intelligence integration | MISP API | Data exchange with OpenCTI |

The MISP workload is managed independently from OpenCTI, allowing maintenance and troubleshooting without requiring the entire intelligence environment to be redeployed.

---

## 4. MISP Deployment

### 4.1 Preparing Ubuntu Server

Following operating system installation, the system was updated and prepared for containerized workloads.

Example commands:

```bash
sudo apt update
sudo apt upgrade -y
```

Docker Engine and the Docker Compose plugin were installed to support the deployment.

Container service availability was verified using:

```bash
sudo systemctl status docker
docker compose version
```

### 4.2 Obtaining the MISP Docker Deployment

The MISP Docker project provides the application and supporting service definitions necessary to deploy MISP through containers.

Reference repository:

https://github.com/MISP/misp-docker

Example:

```bash
git clone https://github.com/MISP/misp-docker.git
cd misp-docker
```

Deployment settings were managed through environment variables, with sensitive credentials stored outside published documentation.

### 4.3 Application Configuration

MISP requires environment-specific configuration for its application URL, administrative settings, database services, and supporting containers.

A sanitized example of the base URL configuration is shown below:

```dotenv
BASE_URL=https://misp.example.test
```

`misp.example.test` is a fictional documentation hostname and does not identify the operational laboratory.

The application URL was configured consistently across MISP and its container environment.

A mismatch between the configured URL and the address used to access the application can cause authentication redirects, session issues, and unexpected navigation behavior.

### 4.4 Starting the Containers

After validating the configuration, the MISP stack was deployed using Docker Compose.

Example:

```bash
sudo docker compose config -q
sudo docker compose up -d
```

The following command was used to inspect running containers:

```bash
sudo docker compose ps
```

Container logs were reviewed when troubleshooting startup and application errors:

```bash
sudo docker compose logs --tail 50
```

The deployment was considered operational only after the application interface, supporting services, and authenticated administrative access were verified.

---

## 5. HTTPS and Application Access

### 5.1 HTTPS Configuration

The MISP deployment uses HTTPS for browser access and API communication.

During the initial laboratory deployment, a self-signed certificate was used for HTTPS connectivity.

This provided encrypted transport but did not establish trust through a recognized certificate authority.

### 5.2 Internal Name Resolution

Internal DNS resolution was configured to provide a consistent hostname for accessing the application from authorized devices.

A documentation-only example is:

```text
misp.example.test
```

Using a consistent hostname simplified browser access and reduced confusion caused by switching between different application URLs.

### 5.3 Network Access Restrictions

Access to MISP was restricted to authorized internal networks and systems.

The deployment required consideration of both host-level firewall controls and Docker-managed networking.

Because Docker publishes container ports through its own networking rules, host firewall configuration alone may not provide the intended restrictions.

The effective network policy was therefore reviewed in the context of Docker's packet-filtering behavior.

The operational firewall configuration is intentionally excluded from this repository.

---

## 6. Threat Intelligence Feed Configuration

### 6.1 Intelligence Collection

MISP supports the collection and management of indicators through events, attributes, and external intelligence feeds.

An OpenPhish URL feed was used to validate the intelligence ingestion workflow.

### 6.2 Feed Import

The feed was configured through the MISP interface and used to populate an intelligence event containing URL attributes.

The imported intelligence was reviewed to confirm that the event and associated attributes were available for subsequent processing.

### 6.3 Intelligence Interpretation

Indicators obtained through external feeds should be evaluated according to their source, context, freshness, and available corroborating evidence.

The presence of a URL in a threat intelligence feed does not independently establish that the URL remains malicious at the time of investigation.

Feed-derived indicators were therefore treated as intelligence inputs requiring further analysis rather than automatic confirmation of malicious activity.

---

## 7. OpenCTI Integration

### 7.1 Integration Objective

The MISP connector was deployed to transfer intelligence from MISP into OpenCTI.

This integration allows intelligence collected through MISP to become available for additional enrichment, relationship analysis, and reporting within OpenCTI.

### 7.2 Authentication

Dedicated API credentials were configured for the integration.

The setup used separate authentication material for MISP and OpenCTI, allowing the integration to be managed independently from routine administrator access.

Credentials were stored in environment configuration rather than embedded directly into the connector definition.

### 7.3 Example Connector Configuration

The following is a sanitized configuration example illustrating the main integration settings.

```yaml
services:
  connector-misp:
    image: opencti/connector-misp:latest
    restart: unless-stopped
    environment:
      - OPENCTI_URL=http://opencti:8080
      - OPENCTI_TOKEN=${MISP_OPENCTI_TOKEN}
      - CONNECTOR_ID=${CONNECTOR_MISP_ID}
      - CONNECTOR_TYPE=EXTERNAL_IMPORT
      - CONNECTOR_NAME=MISP
      - CONNECTOR_SCOPE=misp
      - CONNECTOR_LOG_LEVEL=info
      - CONNECTOR_DURATION_PERIOD=PT5M
      - MISP_URL=https://misp.example.test
      - MISP_KEY=${MISP_API_KEY}
      - MISP_SSL_VERIFY=true
      - MISP_CREATE_REPORTS=true
      - MISP_CREATE_INDICATORS=true
      - MISP_CREATE_OBSERVABLES=true
      - MISP_IMPORT_ONLY_PUBLISHED=false
```

**Configuration note:** This example represents the intended certificate-validated configuration. During initial laboratory testing, certificate verification was temporarily disabled because the deployment used a self-signed certificate.

A trusted certificate or appropriately configured internal certificate authority is recommended before enabling certificate verification.

The example is not a complete deployment template and requires environment-specific integration settings.

### 7.4 Intelligence Processing

The connector was configured to import MISP intelligence into OpenCTI and create supported intelligence objects.

The resulting data included:

- MISP-derived reports.
- URL observables.
- Indicators.
- Relationships between imported objects.
- Associated intelligence metadata.

This provided a foundation for correlating feed-derived intelligence with additional enrichment sources.

---

## 8. Troubleshooting and Problem Resolution

The deployment required resolving several configuration and networking issues.

The following sections document significant problems encountered during implementation.

### 8.1 HTTPS Redirect and Login Issues

**Problem**

The MISP web interface exhibited unexpected redirect and authentication behavior.

**Investigation**

The configured application base URL was compared with the hostname used to access the platform.

The application settings and Docker environment configuration were reviewed for inconsistencies.

**Root Cause**

The application's configured URL was not consistently aligned with its external access address.

**Resolution**

The MISP base URL settings were aligned with the intended internal HTTPS hostname.

The container configuration and relevant application settings were updated, followed by application validation.

**Verification**

The browser successfully reached the expected application URL and authenticated sessions functioned correctly.

**Lesson Learned**

Application URL consistency is critical for reverse-proxied and containerized web applications, especially when HTTPS redirects and authenticated sessions are involved.

---

### 8.2 Application Configuration File Permissions

**Problem**

Application configuration changes did not behave as expected during troubleshooting.

**Investigation**

The ownership and permissions of the MISP configuration file were examined.

**Resolution**

File ownership and permissions were adjusted to permit the required application access while avoiding unnecessarily broad filesystem permissions.

**Verification**

The application successfully loaded its configuration and authentication behavior was rechecked.

**Lesson Learned**

Linux file permissions and service identities can affect containerized applications, particularly when configuration files are shared between host and container environments.

---

### 8.3 Docker Networking and Firewall Restrictions

**Problem**

Network access controls did not initially behave as expected when restricting HTTPS access to MISP.

**Investigation**

The host's packet-filtering rules were reviewed alongside Docker's automatically managed networking chains.

**Root Cause**

Docker's port-publishing behavior affected how inbound traffic was processed, requiring the access policy to account for Docker-managed forwarding rules.

**Resolution**

Access restrictions were implemented using an appropriate Docker-aware firewall approach.

The policy was designed to permit authorized management traffic while restricting unwanted access to the published application service.

**Verification**

Authorized clients could access MISP, while traffic outside the intended access scope was restricted.

**Lesson Learned**

Docker networking must be considered when designing host firewall policies. Published container ports cannot always be secured using the same assumptions as services listening directly on the host.

---

### 8.4 API Integration and Certificate Trust

**Problem**

Connecting OpenCTI to the HTTPS-enabled MISP instance required additional configuration because the certificate was not trusted by the connector.

**Investigation**

The connector's TLS validation behavior and MISP endpoint configuration were reviewed.

**Resolution**

Certificate verification was temporarily disabled for the initial trusted-lab integration while using the self-signed certificate.

**Verification**

The connector authenticated with MISP and successfully retrieved intelligence events.

**Security Consideration**

Disabling certificate verification weakens protection against server impersonation and man-in-the-middle attacks.

This remains a laboratory limitation to be addressed through trusted certificate management.

**Lesson Learned**

Encrypted transport and certificate trust are separate security requirements. Successful HTTPS connectivity does not, by itself, guarantee that a client has verified the identity of the remote endpoint.

---

## 9. Deployment Validation

### 9.1 Container Health

Docker commands were used to confirm that the expected MISP services were running.

```bash
sudo docker compose ps
```

Logs were examined when investigating service failures or abnormal behavior.

### 9.2 Application Access

The web interface was tested from an authorized management system.

Validation included:

- Successful DNS resolution.
- HTTPS connectivity.
- Access to the MISP login page.
- Successful administrator authentication.
- Availability of intelligence events.

### 9.3 Intelligence Import

The OpenPhish feed was used to confirm that MISP could ingest external URL intelligence.

The resulting MISP event and its attributes were verified within the platform.

### 9.4 OpenCTI Synchronization

After configuring the MISP connector, the imported event was located within OpenCTI.

Validation confirmed that MISP-derived intelligence had been transformed into supported OpenCTI objects, including observables, indicators, and relationships.

The connector logs reported successful batch processing.

This demonstrated that intelligence could move through the intended collection and analysis pipeline.

---

## 10. Results

The MISP deployment established a functional intelligence collection and management platform integrated with OpenCTI.

The completed implementation demonstrated:

- Successful containerized MISP deployment.
- Functional HTTPS-based application access.
- Internal DNS configuration.
- Restricted application access.
- External intelligence feed ingestion.
- Authenticated API integration with OpenCTI.
- Successful creation of intelligence objects from imported MISP data.
- Troubleshooting of application, certificate, permission, and container-networking issues.

---

## 11. Lessons Learned

Several important technical and operational lessons emerged from this deployment.

**Containerized applications still require infrastructure knowledge.** Successful deployment depends on understanding Linux permissions, Docker networking, DNS, application configuration, and authentication.

**Intelligence ingestion must be validated end to end.** Running containers and active connectors do not guarantee that intelligence has reached the destination platform.

**Security controls should be tested rather than assumed.** Access restrictions must be evaluated against the actual network path, including container forwarding behavior.

**TLS certificate verification matters.** Self-signed certificates may be practical during initial setup, but a trusted certificate strategy is preferable for sustained operation.

**Threat intelligence requires analytic judgment.** Imported feed indicators provide investigation leads, not automatic proof of compromise or malicious activity.

---

## 12. Future Improvements

Planned improvements include:

- Implementing trusted TLS certificate management.
- Expanding supported intelligence feed coverage.
- Improving connector monitoring and error visibility.
- Reviewing data retention and storage growth.
- Strengthening configuration management and documentation.
- Continuing validation of intelligence quality and source reliability.

---

## 13. References

- [MISP Project](https://www.misp-project.org/)
- [Official MISP Docker Repository](https://github.com/MISP/misp-docker)
- [OpenCTI Documentation](https://docs.opencti.io/)
- [OpenCTI MISP Connector](https://github.com/OpenCTI-Platform/connectors/tree/master/external-import/misp)
- [Docker Documentation](https://docs.docker.com/)
