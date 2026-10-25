# Threat Intelligence Connector Integration

## 1. Overview

This document describes the configuration, integration, troubleshooting, and validation of threat intelligence connectors within the self-hosted OpenCTI and MISP laboratory.

The objective was to establish a centralized intelligence workflow capable of collecting threat intelligence from external sources, enriching existing observables, and supporting analyst-driven investigations.

The environment uses modular Docker Compose configurations to manage integrations independently from the core OpenCTI services.

All examples are sanitized. Operational credentials, internal hostnames, private network addresses, and API tokens are excluded.

---

## 2. Integration Objectives

The connector deployment focused on the following objectives:

- Import external intelligence into a centralized analysis platform.
- Synchronize MISP intelligence with OpenCTI.
- Collect adversary tactics, techniques, and procedures (TTPs).
- Incorporate community threat intelligence and vulnerability reporting.
- Enrich observables through external reputation and analysis services.
- Manage API usage and ingestion frequency.
- Implement dedicated connector identities where supported.
- Validate successful data processing rather than relying solely on container status.
- Develop practical intelligence collection and correlation workflows.

---

## 3. Connector Architecture

OpenCTI supports different connector types depending on the purpose of an integration.

### External Import Connectors

External import connectors retrieve intelligence from external sources and transfer it into OpenCTI.

These integrations are typically scheduled and may maintain state to support incremental collection.

### Internal Enrichment Connectors

Internal enrichment connectors process existing OpenCTI objects and obtain additional information from external services.

Enrichment can provide reputation information, analysis results, categorization, or other context to support an investigation.

For this laboratory, enrichment is generally initiated manually to limit unnecessary requests and retain analyst control.

### Integration Overview

| Connector | Type | Intelligence Purpose |
|---|---|---|
| MISP | External Import | Feed-derived indicators, events, and observables |
| MITRE ATT&CK | External Import | Adversary behaviors, techniques, groups, and software |
| AlienVault OTX | External Import | Community intelligence pulses and threat reports |
| CISA KEV | External Import | Known exploited vulnerability intelligence |
| VirusTotal | Internal Enrichment | File, URL, domain, and IP reputation context |
| urlscan.io | Internal Enrichment | Website analysis and related infrastructure |
| AbuseIPDB | Internal Enrichment | IP address abuse reporting and reputation |

These integrations provide complementary intelligence rather than interchangeable judgments about malicious activity.

---

## 4. Docker Compose Integration Strategy

### Modular Connector Deployment

Connectors were configured using Docker Compose.

The laboratory uses separate Compose files for optional integrations, allowing connectors to be added, updated, restarted, or removed independently.

A generic connector definition is shown below.

```yaml
services:
  connector-example:
    image: example/connector-image:version
    restart: unless-stopped
    environment:
      - OPENCTI_URL=http://opencti:8080
      - OPENCTI_TOKEN=${EXAMPLE_OPENCTI_TOKEN}
      - CONNECTOR_ID=${EXAMPLE_CONNECTOR_ID}
      - CONNECTOR_TYPE=EXTERNAL_IMPORT
      - CONNECTOR_NAME=Example Connector
      - CONNECTOR_LOG_LEVEL=info
```

This is an architectural example, not a runnable connector configuration.

### Configuration Validation

Docker Compose configuration was validated before launching a connector.

```bash
sudo docker compose \
  -f docker-compose.yml \
  -f docker-compose.example.yml \
  config -q
```

The `config -q` option validates the Compose configuration without printing expanded environment variables.

### Independent Deployment

An individual connector could be started without intentionally recreating the core application.

```bash
sudo docker compose \
  -f docker-compose.yml \
  -f docker-compose.example.yml \
  up -d --no-deps connector-example
```

### Log Inspection

Connector activity was monitored through Docker logs.

```bash
sudo docker logs <CONNECTOR_CONTAINER> --tail 30
```

This approach helped isolate integration failures while preserving the availability of the broader CTI environment.

---

## 5. Authentication and API Security

### Dedicated Connector Identities

Where supported, integrations were assigned dedicated OpenCTI service accounts and authentication tokens.

This reduces dependence on shared administrator credentials and helps distinguish automated connector activity from interactive administrative actions.

### Credential Storage

External API keys and OpenCTI authentication tokens were stored through environment variables.

Example:

```dotenv
EXAMPLE_OPENCTI_TOKEN=<REPLACE_WITH_OPENCTI_TOKEN>
EXAMPLE_PROVIDER_API_KEY=<REPLACE_WITH_PROVIDER_API_KEY>
EXAMPLE_CONNECTOR_ID=<GENERATE_UNIQUE_UUID>
```

No operational values are included in this repository.

### Connector Permissions

Connector permissions should be restricted to the actions required by the integration.

Some initial configurations inherited administrator-level access from the base deployment. Migrating remaining connectors to dedicated least-privilege identities is an identified security improvement.

### Rate Limiting and Collection Scope

Connector schedules and enrichment behavior were configured with attention to:

- External API quotas.
- Initial historical collection volume.
- Processing overhead.
- Data quality and duplication.
- Available storage and memory.
- Restrictions on sensitive data submission.

---

## 6. MISP Connector

### Purpose

The MISP connector transfers intelligence collected within MISP into OpenCTI for further analysis and enrichment.

This provides a bridge between intelligence collection and centralized knowledge management.

### Configuration

The connector authenticates with the MISP API and uses a separate OpenCTI token for destination platform access.

Its configuration supports the import of MISP events, observables, indicators, and related information.

### Validation

An OpenPhish URL feed was imported into MISP and subsequently processed by the OpenCTI MISP connector.

The resulting OpenCTI data included reports, indicators, URL observables, and relationships.

Connector logs confirmed successful batch processing.

### Security Consideration

The MISP integration initially used a self-signed TLS certificate without connector-side certificate verification.

Establishing trusted certificate validation remains an identified hardening task.

---

## 7. MITRE ATT&CK Connector

### Purpose

The MITRE ATT&CK connector provides structured intelligence describing adversary behaviors, techniques, software, and associated entities.

This allows analysts to correlate indicators and observations with broader adversary tradecraft.

### Deployment

The existing OpenCTI Docker deployment included a MITRE ATT&CK connector definition.

The connector was enabled through Docker Compose and validated through its container logs and the OpenCTI interface.

### Troubleshooting: Service Account Initialization

**Problem**

The connector initially failed during startup.

The observed exception included:

```text
KeyError: 'name'
```

**Investigation**

The connector logs indicated that the exception occurred during automatic service account or token handling.

**Resolution**

Automatic service account creation was disabled, allowing the connector to use its configured OpenCTI authentication token.

**Verification**

The connector successfully registered with OpenCTI, and imported ATT&CK intelligence became visible in the platform.

**Lesson Learned**

Connector authentication features can introduce compatibility issues. Registration and successful data ingestion should be verified after configuration changes.

---

## 8. AlienVault OTX Connector

### Purpose

AlienVault Open Threat Exchange (OTX) provides community-contributed intelligence through pulses containing threat reports and related indicators.

The integration enables OpenCTI to ingest intelligence from subscribed OTX sources.

### Collection Configuration

The connector was configured to collect recent intelligence rather than importing an unrestricted historical dataset.

A scheduled collection interval was used to reduce unnecessary API activity.

Optional inference features that could create speculative malware or vulnerability associations were disabled.

### Troubleshooting: Timestamp Parsing

**Problem**

The initial collection failed because the connector rejected the configured start timestamp.

The observed error was:

```text
ValueError: unconverted data remains: Z
```

**Root Cause**

The configured timestamp ended with a UTC timezone designator that the connector's parser did not accept.

**Resolution**

The timestamp was changed to match the format expected by the installed connector version.

**Verification**

The subsequent connector execution successfully processed the available intelligence pulses without reported import failures.

**Lesson Learned**

Even when documentation describes a standard date format, connector-specific parser behavior must be validated against the actual installed version.

### Intelligence Validation

The imported OTX reports became visible in OpenCTI.

This confirmed successful ingestion of community intelligence into the central analysis platform.

---

## 9. CISA Known Exploited Vulnerabilities Connector

### Purpose

CISA's Known Exploited Vulnerabilities catalog provides intelligence about vulnerabilities known to have been exploited in the wild.

This information helps analysts distinguish theoretical vulnerability exposure from documented exploitation activity.

### Deployment

The connector was configured to retrieve CISA's publicly available KEV catalog.

A scheduled collection interval was used for periodic updates.

### Validation

Initial connector logs confirmed that the catalog was retrieved and vulnerability entries were being submitted to OpenCTI's processing pipeline.

This verified that collection and queue submission were functioning.

**Validation limitation:** The captured evidence did not establish that every catalog entry had completed indexing. Full import completion should be checked separately.

### Intelligence Application

KEV information can support:

- Vulnerability intelligence reporting.
- Exploitation trend analysis.
- Threat exposure prioritization.
- Correlation with software and infrastructure context.

A vulnerability's presence in KEV does not establish that a particular environment is affected or compromised.

---

## 10. VirusTotal Connector

### Purpose

VirusTotal provides reputation and analysis information for supported observables, including URLs, domains, IP addresses, and file hashes.

### Configuration

The connector was configured for manual enrichment.

This allows analysts to select which observables require enrichment rather than automatically processing every imported item.

Manual operation also helps manage API usage under account-specific limits.

### Validation

A URL imported through the MISP intelligence pipeline was manually enriched.

VirusTotal returned security-vendor classifications that were reflected in OpenCTI.

The enrichment operation also updated the observable's metadata.

### Analytic Considerations

Vendor detection totals should be assessed in context.

A limited number of malicious detections may justify additional investigation, but it does not independently prove active exploitation, credential theft, or infrastructure ownership.

The associated OpenCTI score should not automatically be interpreted as a mathematical probability of maliciousness.

---

## 11. urlscan.io Connector

### Purpose

urlscan.io supports website-related intelligence collection and enrichment.

Depending on connector capabilities and available scan results, it can provide information relevant to web infrastructure, website behavior, and phishing investigations.

### Configuration

The integration was configured for manual enrichment.

The laboratory also used an unlisted scan visibility setting.

Unlisted scans are not equivalent to private scans and may remain accessible to anyone possessing the associated scan link.

Sensitive, confidential, or internal URLs should not be submitted without appropriate authorization and a suitable privacy configuration.

### Validation

A reserved public example URL was used to test the enrichment workflow.

The connector completed the requested enrichment operation.

This confirmed operational integration without submitting an internal laboratory address.

### Analytic Considerations

Website scan results should be considered alongside independent intelligence sources.

Observed web content or infrastructure does not, by itself, establish malicious intent or threat actor attribution.

---

## 12. AbuseIPDB Connector

### Purpose

AbuseIPDB provides reputation context for public IP addresses, including community abuse reports and associated reporting statistics.

### Configuration

The connector was deployed as an internal enrichment integration with automatic enrichment disabled.

### Validation

An IP observable identified through a DNS investigation was submitted for enrichment.

The connector retrieved the observable, processed the request, and sent its resulting data bundle to OpenCTI.

Connector logs reported successful processing without application errors.

### Independent Reputation Review

The same IP was reviewed through the AbuseIPDB web interface.

The displayed results showed no existing abuse reports and a 0% abuse confidence score at the time of review.

These findings were consistent with the absence of a positive abuse score in the investigated OpenCTI observable.

However, the OpenCTI enrichment history did not independently expose all underlying API response fields.

### Analytic Considerations

A lack of reported abuse does not establish that an IP address is safe.

An IP can be associated with a suspicious website without appearing in an IP reputation database.

URL reputation, IP reputation, and evidence of malicious behavior represent different forms of intelligence and should not be treated as equivalent.

---

## 13. End-to-End Intelligence Workflow

The configured integrations support a repeatable intelligence investigation process.

### Collection

External threat intelligence is collected through MISP and supported OpenCTI import connectors.

### Normalization

Imported data is represented within OpenCTI using supported intelligence object types and relationships.

### Enrichment

Selected observables are enriched through external providers.

### Correlation

Analysts compare results from different intelligence sources, identifying corroboration, inconsistencies, and intelligence gaps.

### Assessment

Available evidence is evaluated to develop an analytic judgment.

### Reporting

Investigation findings are documented in structured CTI reports, including supporting observables, indicators, confidence assessments, and recommended next steps.

### Example Workflow

```text
External Intelligence Feed
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
   Observable Review
          |
          v
   Manual Enrichment
          |
          v
 Intelligence Correlation
          |
          v
     CTI Assessment
          |
          v
    Intelligence Report
```

---

## 14. Connector Validation Methodology

A connector was not considered fully validated solely because its Docker container was running.

Validation was performed across several levels.

| Validation Level | Evidence |
|---|---|
| Container startup | Docker container running |
| Platform registration | Connector visible in OpenCTI |
| API connectivity | Successful authentication and provider communication |
| Task execution | Connector logs show a completed operation |
| Queue submission | Intelligence bundle submitted for processing |
| Data ingestion | Imported entities visible within OpenCTI |
| Analytic usefulness | Intelligence contains relevant and interpretable findings |

These stages distinguish infrastructure availability from successful delivery of usable intelligence.

---

## 15. Resource Monitoring

The addition of intelligence connectors introduces additional CPU, memory, network, and storage activity.

System-level monitoring was performed using:

```bash
free -h
```

Container resource consumption was reviewed using:

```bash
sudo docker stats --no-stream
```

The observed laboratory environment retained available memory and exhibited relatively low CPU usage during the monitoring period.

Elasticsearch remained the largest memory consumer, consistent with its indexing responsibilities.

These measurements represent a point-in-time operational baseline rather than a permanent performance guarantee.

Resource requirements may increase as intelligence volume, historical collection, and connector activity grow.

---

## 16. Lessons Learned

### Successful Startup Is Not Successful Ingestion

A running connector may still encounter authentication errors, malformed configuration, or data processing failures.

### Intelligence Sources Provide Different Perspectives

URL classifications, IP abuse reports, vulnerability records, and ATT&CK mappings answer different intelligence questions.

### External Feeds Require Source Evaluation

The presence of an indicator in a feed should initiate investigation, not automatically establish malicious activity.

### Manual Enrichment Supports Controlled Analysis

Manual enrichment helps manage API quotas, reduce unnecessary external submissions, and maintain analyst oversight.

### Configuration Compatibility Matters

Connector behavior can vary across software versions. Configuration validation and log review are necessary during deployment and updates.

### Modular Configuration Improves Maintainability

Independent connector configurations simplify deployment, troubleshooting, and future expansion.

---

## 17. Future Improvements

Future development opportunities include:

- Expanding trusted intelligence sources.
- Refining ingestion schedules and historical collection windows.
- Implementing more consistent least-privilege service accounts.
- Improving API quota and connector failure monitoring.
- Evaluating data quality, duplication, and confidence handling.
- Establishing trusted TLS verification across integrations.
- Monitoring storage growth and Elasticsearch performance.
- Developing automated intelligence quality checks.
- Expanding structured reporting and analytic workflows.

---

## 18. References

- [OpenCTI Connector Documentation](https://docs.opencti.io/latest/deployment/connectors/)
- [Official OpenCTI Connector Repository](https://github.com/OpenCTI-Platform/connectors)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [AlienVault OTX](https://otx.alienvault.com/)
- [CISA Known Exploited Vulnerabilities](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [VirusTotal](https://www.virustotal.com/)
- [urlscan.io](https://urlscan.io/)
- [AbuseIPDB](https://www.abuseipdb.com/)
- [MISP Project](https://www.misp-project.org/)
