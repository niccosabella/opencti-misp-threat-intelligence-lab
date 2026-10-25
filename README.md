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

The environment is hosted on Proxmox VE, with dedicated Linux virtual machines supporting MISP and OpenCTI.

MISP serves as a threat intelligence collection and management platform, while OpenCTI provides centralized intelligence analysis, enrichment, and relationship modeling.

External intelligence sources are integrated through dedicated connectors, allowing threat intelligence data to be imported and analyzed within OpenCTI.

Docker Compose is used to manage application services, their dependencies, and connector deployments.

**Architecture diagram:** To be added following the infrastructure documentation and security review.

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
