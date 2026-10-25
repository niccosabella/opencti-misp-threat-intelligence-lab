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

Detailed technical documentation is organized into the following areas:

- **Infrastructure Deployment:** Virtual machine provisioning, Linux configuration, and resource allocation.
- **MISP Deployment:** Container configuration, threat intelligence feeds, API access, and platform validation.
- **OpenCTI Deployment:** Application services, container orchestration, and platform configuration.
- **Network Security:** Access restrictions, network segmentation, and secure service communication.
- **Intelligence Integrations:** Connector configuration, authentication, and enrichment workflows.
- **Troubleshooting:** Technical problems encountered during deployment, root-cause analysis, and implemented resolutions.
- **Validation and Results:** Evidence demonstrating successful intelligence collection, processing, and enrichment.

Supporting documentation and screenshots will be added as the project develops.

---

## Security and Privacy

This repository contains sanitized documentation of a private cybersecurity laboratory.

Internal network addresses, management endpoints, authentication credentials, API tokens, and identifying infrastructure information are excluded or replaced with documentation examples.

Configuration examples are provided for educational purposes and should not be interpreted as exact copies of the operational environment.

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
