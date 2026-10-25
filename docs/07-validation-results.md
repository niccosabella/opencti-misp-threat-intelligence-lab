# CTI Lab Validation and Operational Results

## 1. Overview

This document summarizes the operational validation of a self-hosted Cyber Threat Intelligence (CTI) laboratory built using Proxmox Virtual Environment, Ubuntu Server, Docker, MISP, and OpenCTI.

Validation was performed throughout the deployment to confirm that the infrastructure, application services, intelligence connectors, and analysis workflows operated as intended.

The objective was not simply to establish that containers were running, but to verify that intelligence could be collected, processed, enriched, and analyzed across interconnected systems.

The findings represent observed results from the laboratory. They do not constitute a formal security audit, production readiness assessment, or comprehensive performance benchmark.

All infrastructure-specific identifiers and operational credentials have been excluded.

---

## 2. Validation Objectives

The validation process focused on the following objectives:

- Confirm that the underlying virtualized infrastructure was operational.
- Verify Docker container and application availability.
- Confirm successful access to MISP and OpenCTI.
- Validate authenticated communication between intelligence platforms.
- Verify external intelligence collection and ingestion.
- Confirm selected enrichment workflows.
- Evaluate basic system resource utilization.
- Identify limitations and incomplete verification activities.
- Establish a baseline for future operational testing.

---

## 3. Validation Methodology

Validation was conducted across several layers.

| Validation Layer | Objective | Evidence |
|---|---|---|
| Infrastructure | Verify Linux virtual machine operation | System resource checks |
| Containers | Confirm application processes are running | Docker container status |
| Applications | Verify functional platform access | Authenticated web interface |
| Integrations | Confirm connector registration | OpenCTI connector interface |
| Collection | Verify external intelligence retrieval | Connector execution logs |
| Processing | Confirm intelligence submission | Queue and processing messages |
| Data availability | Verify imported objects are accessible | OpenCTI reports and entities |
| Enrichment | Confirm external intelligence requests | Enrichment execution and observable history |
| Performance | Establish a resource baseline | Memory and CPU observations |

Successful validation required appropriate evidence for the specific layer being tested.

For example, connector registration confirmed platform communication but did not independently establish that all imported data had completed processing.

---

## 4. Infrastructure Validation

### Virtual Machine Operation

The MISP and OpenCTI workloads were deployed on dedicated Ubuntu Server virtual machines hosted through Proxmox.

The environments were independently accessible for administration and application maintenance.

### Operating System Validation

General system health was assessed using standard Linux commands.

```bash
free -h
df -h
uptime
```

These commands provided information about available memory, storage utilization, and system uptime.

### Result

The virtualized infrastructure successfully supported the CTI application environments and their supporting services.

**Status: Validated**

---

## 5. Docker and Application Validation

### Container Availability

Docker was used to deploy and manage the application stacks.

Running containers were inspected with:

```bash
sudo docker ps
```

Container resource consumption was reviewed using:

```bash
sudo docker stats --no-stream
```

### MISP Application

Validation included:

- Successful access through the configured internal HTTPS endpoint.
- Successful authenticated administrative access.
- Availability of MISP events and attributes.
- Successful ingestion of an external intelligence feed.

### OpenCTI Application

Validation included:

- Successful access through an SSH-forwarded management connection.
- Successful administrative authentication.
- Availability of reports, observables, indicators, and related intelligence entities.
- Successful connector registration.

### Result

Both intelligence platforms were operational during the validation period.

**Status: Validated**

---

## 6. MISP Intelligence Collection

### Collection Source

An OpenPhish URL feed was configured within MISP.

The feed was used to validate external intelligence retrieval and event creation.

### Observed Results

The import produced an intelligence event containing URL attributes.

The event and associated attributes were accessible through the MISP interface.

### Result

MISP demonstrated the ability to collect and manage external URL intelligence.

**Status: Validated**

### Analytical Limitation

Feed inclusion was treated as an investigation lead rather than independent proof that every imported URL remained malicious.

Indicator quality and freshness require additional assessment.

---

## 7. MISP-to-OpenCTI Integration

### Validation Objective

Confirm that intelligence collected within MISP could be transferred into OpenCTI.

### Observed Workflow

```text
OpenPhish Feed
      |
      v
     MISP
      |
      v
 MISP Connector
      |
      v
    OpenCTI
      |
      v
Reports, Indicators,
Observables, Relationships
```

### Evidence

The MISP connector processed the imported intelligence and reported successful batch completion.

The resulting OpenCTI data included:

- An intelligence report.
- URL observables.
- Indicators.
- Relationships between supported objects.
- Related intelligence metadata.

The imported report and associated observables were accessible through the OpenCTI interface.

### Result

End-to-end transfer of feed-derived intelligence from MISP into OpenCTI was demonstrated.

**Status: Validated**

---

## 8. MITRE ATT&CK Integration

### Validation Objective

Confirm successful ingestion of structured adversary intelligence.

### Observed Results

The MITRE ATT&CK connector registered with OpenCTI following resolution of a startup configuration issue.

Imported intelligence became visible in the platform.

The imported objects included adversary-related entities, software, techniques, and mitigations.

### Result

MITRE ATT&CK intelligence ingestion was demonstrated.

**Status: Validated**

### Limitation

The validation confirmed the presence of imported intelligence objects but did not establish that every object in the source dataset was imported.

---

## 9. AlienVault OTX Integration

### Validation Objective

Confirm successful collection and import of subscribed community threat intelligence.

### Observed Results

Following correction of a timestamp parsing issue, the connector successfully retrieved recent OTX intelligence pulses.

The connector logs reported:

```text
Pulse importer completed (imported: 5, failed: 0, total: 5)
```

The imported intelligence reports subsequently appeared within OpenCTI.

### Result

The connector demonstrated successful source retrieval, intelligence processing, and data availability.

**Status: Validated**

### Limitation

These results describe the observed collection window and are not a permanent guarantee of future connector availability or data quality.

---

## 10. CISA KEV Integration

### Validation Objective

Confirm that the CISA Known Exploited Vulnerabilities catalog could be retrieved and processed.

### Observed Results

The connector successfully retrieved the KEV catalog and began submitting vulnerability entries into OpenCTI's processing queue.

Example sanitized progress output:

```text
Publishing CISA KEV entry <CVE-ID> (<ENTRY>/<TOTAL>)
CISA KEV sending bundle to queue
```

The available logs showed active processing without errors in the captured portion.

### Result

Source retrieval and queue submission were demonstrated.

**Status: Partially validated**

### Outstanding Verification

A final ingestion check should confirm that the import completed and that the expected vulnerability records are accessible in OpenCTI.

The initial evidence did not prove full catalog ingestion.

---

## 11. VirusTotal Enrichment

### Validation Objective

Verify that the VirusTotal connector could enrich an existing OpenCTI URL observable.

### Observed Results

A URL previously imported through MISP was selected for manual enrichment.

VirusTotal returned classifications from security vendors.

The enrichment operation updated the observable's metadata and was reflected in its history.

The observed results included:

| Classification | Results |
|---|---:|
| Malicious | 6 |
| Suspicious | 0 |
| Harmless | 52 |
| Undetected | 35 |

These results were associated with the investigated URL at the time of enrichment.

### Result

Manual VirusTotal URL enrichment was demonstrated.

**Status: Validated**

### Analytical Limitation

Vendor detections were not treated as conclusive proof of active malicious activity.

An OpenCTI numeric score was not interpreted as a mathematical probability of maliciousness.

---

## 12. urlscan.io Enrichment

### Validation Objective

Confirm that OpenCTI could initiate an enrichment operation through the urlscan.io connector.

### Test Method

A reserved public example URL was used to avoid submitting a sensitive internal application address.

The enrichment was initiated manually.

### Observed Results

The enrichment operation completed successfully according to the observed OpenCTI workflow.

### Result

Manual URL enrichment execution was demonstrated.

**Status: Operational test completed**

### Limitation

The captured validation did not include a detailed review of all returned website artifacts or scan output.

Therefore, the result confirms successful execution rather than complete validation of every enrichment capability.

---

## 13. AbuseIPDB Enrichment

### Validation Objective

Verify that an IP observable could be submitted to AbuseIPDB for reputation enrichment.

### Observed Results

An IPv4 observable was created based on a DNS resolution observed during a separate URL investigation.

The connector logs confirmed:

- Receipt of the enrichment request.
- Retrieval of the target observable.
- Submission of an intelligence bundle.
- Successful processing of the requested work.

### Independent Reputation Review

The public AbuseIPDB interface was also used to review the same IP.

The displayed results showed:

- Abuse confidence score: 0%.
- Total abuse reports: 0.
- No existing IP entry in the provider database at the time of review.

### Result

The connector's execution workflow was demonstrated, and a provider-side reputation result was independently reviewed.

**Status: Validated for the tested observable**

### Limitation

The OpenCTI history did not expose every underlying AbuseIPDB response field.

The absence of abuse reports was not interpreted as proof that the address was benign.

---

## 14. Resource Utilization

### Validation Objective

Determine whether the OpenCTI virtual machine retained sufficient resources after enabling intelligence integrations.

### Guest Memory

A Linux memory check produced the following observed results:

| Metric | Observed Value |
|---|---:|
| Total memory | Approximately 11 GiB |
| Used memory | Approximately 5.2 GiB |
| Available memory | Approximately 6.4 GiB |
| Swap used | 0 GiB |

The distinction between free memory and available memory was considered when evaluating system health.

Linux uses available memory for filesystem caching and can reclaim much of that memory when required.

### Container Resource Consumption

Container-level monitoring showed that:

- Elasticsearch was the largest memory consumer.
- The OpenCTI application had a comparatively smaller memory footprint.
- Individual intelligence connectors had relatively low memory consumption.
- CPU utilization was low during the captured observation.

### Result

The platform retained memory headroom during the observed monitoring period.

**Status: Healthy at time of measurement**

### Limitation

The measurements represent a point-in-time snapshot, not a sustained load test.

Resource requirements may change as intelligence datasets, connectors, and background processing activity increase.

---

## 15. Structured Intelligence Analysis

### Validation Objective

Confirm that imported and enriched intelligence could support an analyst-driven investigation.

### Investigation Workflow

A suspicious URL collected through MISP was reviewed within OpenCTI.

The investigation involved:

1. Reviewing the original feed-derived observable.
2. Performing VirusTotal enrichment.
3. Recording the resulting security-vendor classifications.
4. Conducting DNS resolution.
5. Reviewing the associated IP through AbuseIPDB.
6. Evaluating differences between URL and IP reputation.
7. Documenting findings, uncertainty, and additional intelligence requirements.
8. Creating a structured OpenCTI intelligence report.

### Observed Results

The report included:

- An executive summary.
- Technical findings.
- Source-based observations.
- An analytical assessment.
- Intelligence gaps.
- Recommended investigative actions.
- Associated observables and an indicator.

### Result

The CTI environment successfully supported an end-to-end analyst-driven intelligence assessment.

**Status: Validated**

### Limitation

The report represented a preliminary assessment rather than confirmation of an active phishing campaign or attribution to a specific threat actor.

---

## 16. Validation Summary

| Component | Validation Outcome |
|---|---|
| Proxmox-hosted Linux infrastructure | Validated |
| Docker application services | Validated |
| MISP application access | Validated |
| OpenCTI application access | Validated |
| MISP intelligence collection | Validated |
| MISP-to-OpenCTI transfer | Validated |
| MITRE ATT&CK ingestion | Validated for observed objects |
| AlienVault OTX ingestion | Validated |
| CISA KEV collection | Partially validated |
| VirusTotal enrichment | Validated |
| urlscan.io enrichment | Operational test completed |
| AbuseIPDB enrichment | Validated for tested observable |
| Resource availability | Healthy during observation |
| Structured CTI reporting | Validated |

The results demonstrate a functioning intelligence environment while preserving visibility into areas requiring additional verification.

---

## 17. Outstanding Validation Tasks

The following activities would strengthen future operational confidence:

- Confirm completion of the CISA KEV catalog import.
- Review detailed urlscan.io enrichment artifacts.
- Establish repeatable connector health checks.
- Verify application backup and recovery procedures.
- Test restoration from known-good backups.
- Review TLS certificate trust and remove temporary verification exceptions.
- Evaluate connector authentication permissions.
- Monitor storage growth and indexing performance.
- Perform periodic validation after platform updates.

---

## 18. Lessons Learned

**Functional validation requires multiple forms of evidence.**

Container status, connector logs, queue activity, and application-visible intelligence provide different levels of assurance.

**Successful enrichment does not establish maliciousness.**

Reputation and scan results require interpretation in the context of the specific observable being investigated.

**Performance measurements require context.**

A healthy memory snapshot does not guarantee that a system will behave the same way during large historical imports or future data growth.

**Limitations should be documented openly.**

Identifying incomplete verification activities makes the assessment more credible and provides a practical roadmap for further testing.

**End-to-end workflows provide the strongest evidence of value.**

The most meaningful validation was demonstrating that intelligence could move from collection into enrichment, analysis, and structured reporting.

---

## 19. Conclusion

The CTI laboratory successfully demonstrated the collection, processing, enrichment, and analysis of external threat intelligence using a self-hosted MISP and OpenCTI environment.

The deployment provided practical experience with virtualization, containerization, application security, connector integration, intelligence modeling, and technical troubleshooting.

The results establish a functional foundation for continued threat intelligence investigations and infrastructure improvements.

While the environment is operational, validation remains an ongoing process as new intelligence sources and platform capabilities are introduced.

---

## 20. References

- [OpenCTI Documentation](https://docs.opencti.io/)
- [MISP Project](https://www.misp-project.org/)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [AlienVault OTX](https://otx.alienvault.com/)
- [CISA Known Exploited Vulnerabilities](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [VirusTotal](https://www.virustotal.com/)
- [urlscan.io](https://urlscan.io/)
- [AbuseIPDB](https://www.abuseipdb.com/)
