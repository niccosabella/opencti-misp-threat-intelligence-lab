# OpenCTI & MISP Threat Intelligence Lab
 
### Self-Hosted Cyber Threat Intelligence Infrastructure | Proxmox • Docker • Linux • STIX 2.1

## Project Overview

This project documents the design, deployment, configuration, and operation of a self-hosted Cyber Threat Intelligence (CTI) laboratory built using OpenCTI, MISP, Proxmox Virtual Environment, and Docker.

The environment was developed to explore how threat intelligence platforms collect, normalize, enrich, correlate, and operationalize intelligence from external sources. It provides a practical environment for investigating indicators of compromise (IOCs), analyzing adversary behavior, and developing structured intelligence assessments.

Rather than relying solely on publicly available threat intelligence platforms, this project focuses on building and maintaining the underlying infrastructure while demonstrating how intelligence moves between interconnected systems.

The lab also serves as an environment for developing threat intelligence analysis, infrastructure administration, and technical troubleshooting skills.

---

## Project Objectives

The primary objectives of this project are to:

- Design and deploy a virtualized, self-hosted threat intelligence environment.
- Implement MISP for threat intelligence collection, event management, and indicator sharing.
- Deploy OpenCTI as a centralized platform for intelligence analysis and knowledge management.
- Integrate external threat intelligence sources and enrichment services.
- Explore STIX-based intelligence modeling and relationships between indicators, observables, and threat actors.
- Implement appropriate network segmentation, authentication, and access controls.
- Troubleshoot infrastructure, networking, API connectivity, and connector integration issues.
- Validate intelligence ingestion and enrichment through hands-on investigations.
- Document the deployment process in a reproducible, security-conscious format.

---

## Platforms and Technologies

| Technology | Purpose |
|---|---|
| Proxmox VE | Virtualization and infrastructure management |
| Ubuntu Server | Linux operating system for CTI workloads |
| Docker & Docker Compose | Container deployment and orchestration |
| MISP | Threat intelligence collection, management, and sharing |
| OpenCTI | Intelligence correlation, enrichment, visualization, and analysis |
| Elasticsearch | Search and storage for OpenCTI intelligence data |
| Redis | Caching and supporting services |
| RabbitMQ | Asynchronous message processing |
| MinIO | Object storage |
| MITRE ATT&CK | Adversary tactics, techniques, and procedure intelligence |
| AlienVault OTX | Community-driven threat intelligence |
| CISA KEV | Known exploited vulnerability intelligence |
| VirusTotal | Indicator reputation and enrichment |
| urlscan.io | Website and URL analysis |
| AbuseIPDB | IP address reputation and abuse reporting |

---

## Lab Architecture

The environment is hosted on Proxmox Virtual Environment, with dedicated Linux virtual machines running containerized MISP and OpenCTI services.

MISP manages threat intelligence collection and sharing, while OpenCTI centralizes intelligence ingestion, enrichment, correlation, and analysis.

```mermaid
flowchart LR
    A["Threat Intelligence Feeds"] --> B["MISP"]
    B -->|"API Integration"| C["OpenCTI"]
    D["External Intelligence Sources"] -->|"Connectors"| C
    C --> E["Intelligence Analysis & Reporting"]

    classDef platform fill:#173b5e,stroke:#57a0d3,color:#ffffff
    classDef source fill:#264653,stroke:#69b3b0,color:#ffffff
    classDef output fill:#274a39,stroke:#78b99a,color:#ffffff

    class B,C platform
    class A,D source
    class E output
```

For a detailed breakdown of the infrastructure, supporting services, and intelligence integrations, see the [Technical Architecture Documentation](architecture/README.md).

---

## Project Documentation

The laboratory's implementation, configuration, troubleshooting, and validation procedures are documented in the following technical guides.

| Documentation | Description |
|---|---|
| [Infrastructure Deployment](docs/01-infrastructure-deployment.md) | Proxmox virtualization, Ubuntu Server provisioning, resource planning, and Docker preparation. |
| [MISP Deployment and Configuration](docs/02-misp-deployment.md) | MISP installation, HTTPS configuration, intelligence feeds, API integration, and troubleshooting. |
| [OpenCTI Deployment and Configuration](docs/03-opencti-deployment.md) | OpenCTI deployment, supporting services, container orchestration, and resource optimization. |
| [Network Security and Access Controls](docs/04-network-security.md) | Restricted management access, internal DNS, Docker firewall behavior, TLS, and authentication. |
| [Threat Intelligence Connector Integration](docs/05-intelligence-connectors.md) | External intelligence collection, enrichment workflows, connector configuration, and API integration. |
| [Troubleshooting and Problem Resolution](docs/06-troubleshooting.md) | Technical challenges, investigations, root-cause analysis, corrective actions, and verification. |
| [Validation and Operational Results](docs/07-validation-results.md) | Evidence of platform availability, successful intelligence ingestion, enrichment, and resource monitoring. |

### Technical Architecture

For detailed information about the laboratory's components, supporting services, and intelligence data flows, see the [Architecture Documentation](architecture/README.md).

### Documentation Approach

The documentation emphasizes reproducible deployment practices, technical decision-making, security considerations, and evidence-based validation.

Configuration examples are sanitized, and the repository is maintained as an evolving technical reference as the laboratory develops.

---

## Laboratory Evidence

The deployment is supported by sanitized screenshots demonstrating infrastructure operation, platform availability, intelligence collection, and connector integration.

| Evidence | Description |
|---|---|
| [Proxmox Infrastructure](screenshots/README.md#01-proxmox-infrastructure) | Virtualization environment and host resource monitoring |
| [MISP Dashboard](screenshots/README.md#02-misp-threat-intelligence-dashboard) | Operational intelligence collection platform |
| [OpenPhish Feed Import](screenshots/README.md#03-misp-openphish-feed-ingestion) | Collection and organization of URL-based threat intelligence |
| [OpenCTI Integrations](screenshots/README.md#04-opencti-intelligence-integrations) | External intelligence collection and enrichment connectors |

See the [Complete Evidence Gallery](screenshots/README.md) for screenshots, captions, and validation context.

---

## Security and Privacy

This repository contains sanitized documentation of a private cybersecurity laboratory.

Internal network addresses, management endpoints, authentication credentials, API tokens, and identifying infrastructure information are excluded or replaced with documentation examples.

Configuration files contain placeholders rather than operational secrets, and published screenshots are reviewed to remove confidential information.

The architecture diagrams represent logical components and intelligence data flows rather than a complete operational network topology.

Security limitations discovered during implementation, including temporary certificate trust exceptions, are documented alongside proposed improvements.

Sensitive values are never intentionally committed to the repository.

---

## Project Status

**Active Development and Continuous Improvement**

The laboratory is operational and continues to evolve through additional intelligence sources, infrastructure improvements, analysis workflows, and hands-on investigations.

The documentation will be updated to reflect meaningful architectural changes, technical findings, and lessons learned.

---

## References

- [OpenCTI Documentation](https://docs.opencti.io/)
- [MISP Project](https://www.misp-project.org/)
- [Proxmox Virtual Environment](https://www.proxmox.com/)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [STIX 2.1 Specification](https://docs.oasis-open.org/cti/stix/v2.1/stix-v2.1.html)
