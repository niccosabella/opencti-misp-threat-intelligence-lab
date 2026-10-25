# CTI Lab Evidence

This directory contains sanitized screenshots documenting the deployment, configuration, and validation of the self-hosted Cyber Threat Intelligence (CTI) laboratory.

The evidence demonstrates the operation of the underlying infrastructure, intelligence platforms, external integrations, and analysis workflows.

All screenshots are reviewed before publication to exclude credentials, internal IP addresses, private hostnames, authentication tokens, and other sensitive information.

---

## Evidence Index

| Evidence | Description |
|---|---|
| [Infrastructure Overview](#infrastructure-evidence) | Proxmox virtualization and host resource monitoring |
| [MISP Dashboard](#misp-platform-evidence) | Operational threat intelligence collection platform |
| MISP Intelligence Feed | External intelligence collection and event ingestion — planned |
| OpenCTI Dashboard | Centralized intelligence analysis platform — planned |
| Connector Integrations | External intelligence collection and enrichment services — planned |
| Intelligence Ingestion | Imported reports, indicators, and observables — planned |
| Enrichment Results | Manual enrichment and resulting intelligence — planned |
| Resource Monitoring | Application and container resource utilization — planned |

---

## Infrastructure Evidence

### Proxmox Virtual Environment

![Sanitized Proxmox infrastructure overview](01-proxmox-infrastructure.png)

**Figure 1 — Proxmox Infrastructure Overview**

The Proxmox management dashboard demonstrates the operational status of the virtualized CTI environment, including active virtual machines and host resource utilization.

The dashboard provides visibility into CPU, memory, and storage consumption, supporting infrastructure monitoring and resource planning.

Sensitive infrastructure identifiers have been permanently redacted.

---

## MISP Platform Evidence

### MISP Threat Intelligence Dashboard

![MISP threat intelligence dashboard](02-misp-dashboard.png)

**Figure 2 — MISP Threat Intelligence Dashboard**

The MISP dashboard demonstrates the operational intelligence collection platform, including imported attributes, event statistics, and intelligence reporting capabilities.

The screenshot provides evidence that MISP is accessible and managing threat intelligence data within the laboratory environment.

The dashboard also displays an intelligence report associated with the OpenPhish investigation, demonstrating the platform's ability to organize collected intelligence for subsequent analysis.

---

## Evidence Collection Methodology

Screenshots are selected to demonstrate specific technical capabilities or validation outcomes.

Each image should provide evidence supporting a documented implementation step, operational result, or intelligence workflow.

Evidence is organized according to the following principles:

1. **Relevance:** Each screenshot demonstrates a meaningful technical capability or result.
2. **Authenticity:** Screenshots originate from the deployed laboratory environment.
3. **Clarity:** Images are cropped appropriately to emphasize relevant information.
4. **Security:** Sensitive information is removed before publication.
5. **Traceability:** Screenshots are associated with the appropriate deployment or validation documentation.

The purpose of this evidence collection is to demonstrate technical implementation and validation rather than simply display application interfaces.

---

## Security and Privacy

All screenshots are reviewed before being committed to the repository.

Information requiring removal or sanitization includes:

- Internal IP addresses and private network ranges.
- Internal DNS names and management endpoints.
- Authentication credentials and API keys.
- Session tokens and authorization headers.
- Personal or administrative account information.
- Public IP addresses associated with private infrastructure.
- Sensitive configuration values.
- Infrastructure identifiers that are unnecessary to demonstrate the technical outcome.

Redactions are applied permanently to exported images rather than through reversible visual overlays.

Original screenshots containing operational information are not included in the repository.

Screenshots are reviewed again after export to verify that sensitive details are not visible.

---

## Supporting Documentation

For additional technical information about the laboratory, refer to:

- [Infrastructure Deployment](../docs/01-infrastructure-deployment.md)
- [MISP Deployment and Configuration](../docs/02-misp-deployment.md)
- [OpenCTI Deployment and Configuration](../docs/03-opencti-deployment.md)
- [Network Security and Access Controls](../docs/04-network-security.md)
- [Threat Intelligence Connector Integration](../docs/05-intelligence-connectors.md)
- [Troubleshooting and Problem Resolution](../docs/06-troubleshooting.md)
- [Validation and Operational Results](../docs/07-validation-results.md)

Additional evidence will be incorporated as the laboratory develops and new integration or analysis workflows are validated.
