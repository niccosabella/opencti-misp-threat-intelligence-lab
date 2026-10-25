# CTI Lab Evidence

This directory contains sanitized screenshots documenting the deployment, configuration, integration, and validation of the self-hosted Cyber Threat Intelligence (CTI) laboratory.

The evidence demonstrates the operation of the underlying infrastructure, intelligence platforms, external integrations, and analysis workflows.

All screenshots are reviewed before publication to exclude sensitive infrastructure details, authentication information, and other confidential data.

---

## Evidence Index

| Evidence | Description |
|---|---|
| [Infrastructure Overview](#infrastructure-evidence) | Proxmox virtualization and host resource monitoring |
| [MISP Dashboard](#misp-platform-evidence) | Operational threat intelligence collection platform |
| [OpenPhish Feed Import](#openphish-intelligence-collection) | Imported threat intelligence event and URL attributes |
| [OpenCTI Integrations](#opencti-integration-evidence) | External intelligence collection and enrichment connectors |
| Intelligence Ingestion | Imported OpenCTI reports, indicators, and observables — planned |
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

The dashboard also displays an intelligence report associated with an OpenPhish investigation, demonstrating the platform's ability to organize collected intelligence for subsequent analysis.

---

## OpenPhish Intelligence Collection

### Imported MISP Event

![Sanitized OpenPhish intelligence event](03-misp-openphish-feed.png)

**Figure 3 — OpenPhish Threat Intelligence Feed Import**

The MISP event view demonstrates the successful ingestion of URL-based threat intelligence from the OpenPhish feed.

Imported URLs are organized as event attributes, allowing the intelligence to be reviewed, managed, and transferred to OpenCTI through an authenticated connector.

The screenshot demonstrates the collection and organization of threat intelligence. It does not independently establish that every imported URL remains malicious.

Sensitive account information, internal identifiers, and individual URL values have been redacted.

---

## OpenCTI Integration Evidence

### Intelligence Collection and Enrichment Connectors

![OpenCTI connector integrations](04-opencti-connectors.png)

**Figure 4 — OpenCTI Threat Intelligence Integrations**

The OpenCTI integrations interface demonstrates the deployment and registration of external intelligence collection and internal enrichment connectors.

The environment incorporates external intelligence sources such as MISP, MITRE ATT&CK, AlienVault OTX, and CISA Known Exploited Vulnerabilities, alongside enrichment capabilities provided by VirusTotal, urlscan.io, and AbuseIPDB.

The integrations support complementary intelligence workflows, including threat intelligence collection, observable enrichment, vulnerability intelligence, and adversary behavior analysis.

Active connector status demonstrates registration and connectivity with OpenCTI at the time of capture. Successful intelligence ingestion and enrichment are verified separately through application records and execution results.

---

## Evidence Collection Methodology

Screenshots are selected to demonstrate specific technical capabilities, implementation decisions, or operational validation results.

Each image should support a documented implementation step, system capability, or intelligence workflow.

Evidence is organized according to the following principles:

1. **Relevance:** Each screenshot demonstrates a meaningful technical capability or operational result.
2. **Authenticity:** Screenshots originate from the deployed laboratory environment.
3. **Clarity:** Images are cropped appropriately to emphasize relevant information.
4. **Security:** Confidential information is removed before committing images.
5. **Traceability:** Screenshots are associated with the appropriate deployment, integration, or validation documentation.
6. **Accuracy:** Captions distinguish demonstrated functionality from assumptions or incomplete validation.

The purpose of this evidence collection is to demonstrate technical implementation and verification rather than simply display application interfaces.

---

## Security and Privacy

All screenshots are reviewed and sanitized before being committed to the repository.

Information requiring removal or sanitization includes:

- Internal IP addresses and private network ranges.
- Internal DNS names and management endpoints.
- Authentication credentials and API keys.
- Session tokens and authorization headers.
- Personal or administrative account information.
- Public IP addresses associated with private infrastructure.
- Sensitive configuration values and operational network details.
- Infrastructure identifiers unnecessary to demonstrate technical functionality.

Redactions are applied permanently to exported images rather than through reversible overlays, blur effects, or transparent shapes.

Original screenshots containing operational information are not included in the repository.

Images are reviewed after export to verify that sensitive details are no longer visible.

Screenshots are intended to document system functionality without exposing the underlying operational environment.

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
- [Technical Architecture](../architecture/README.md)

Additional evidence will be incorporated as the laboratory develops and new integration or analysis workflows are validated.
