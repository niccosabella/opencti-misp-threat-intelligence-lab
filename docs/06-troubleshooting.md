# CTI Lab Troubleshooting and Problem Resolution

## 1. Overview

This document presents technical challenges encountered during the deployment and operation of a self-hosted Cyber Threat Intelligence (CTI) laboratory built with Proxmox, Ubuntu Server, Docker, MISP, and OpenCTI.

The objective is to demonstrate the troubleshooting methodology used to identify, investigate, resolve, and validate infrastructure and application issues.

Rather than presenting a deployment in which every component worked immediately, this document highlights the practical engineering challenges involved in integrating multiple applications, services, and intelligence sources.

**All technical examples are sanitized.** Actual IP addresses, internal hostnames, credentials, API keys, authentication tokens, and identifying infrastructure details have been excluded.

---

## 2. Troubleshooting Methodology

A consistent approach was used when investigating issues throughout the deployment.

### Investigation Process

1. **Identify the problem:** Determine the affected component, observed behavior, and operational impact.
2. **Collect evidence:** Review application logs, container status, configuration, and relevant system information.
3. **Develop a hypothesis:** Identify potential root causes based on the available evidence.
4. **Apply a targeted change:** Modify the smallest practical part of the configuration.
5. **Validate the result:** Confirm that the original issue is resolved and the intended functionality works.
6. **Document the outcome:** Record the cause, corrective action, remaining limitations, and lessons learned.

This approach helped avoid unnecessary changes to otherwise functioning components.

### Common Diagnostic Commands

**Inspect running containers**

```bash
sudo docker ps
```

**Inspect all containers, including stopped services**

```bash
sudo docker ps -a
```

**Review container logs**

```bash
sudo docker logs <CONTAINER_NAME> --tail 50
```

**Check resource utilization**

```bash
sudo docker stats --no-stream
```

**Review available system memory**

```bash
free -h
```

**Validate Docker Compose configuration without printing expanded secrets**

```bash
sudo docker compose config -q
```

These commands provided a foundation for identifying issues across the containerized environment.

---

## 3. Case Study: MISP HTTPS Redirect and Authentication Issues

### Problem

Following the initial MISP deployment, accessing the application through HTTPS resulted in unexpected redirect and authentication behavior.

The issue interfered with normal administrative access and prevented the application from functioning consistently through its intended internal hostname.

### Investigation

The investigation focused on the relationship between:

- The hostname entered in the browser.
- The configured MISP application base URL.
- Docker environment variables.
- The application configuration.
- The HTTPS endpoint provided by the containerized web service.

The application settings were compared to identify inconsistencies between the internal access URL and the application's configured external address.

### Root Cause

The configured application URL was not consistently aligned with the hostname and HTTPS scheme used to access MISP.

This mismatch contributed to incorrect redirect behavior and authentication problems.

### Resolution

The intended internal HTTPS hostname was configured consistently across the relevant settings.

Sanitized example:

```dotenv
BASE_URL=https://misp.example.test
```

The corresponding internal and external base URL settings were aligned with the same application address.

Configuration permissions were also reviewed during troubleshooting.

### Verification

After the configuration changes:

- The application loaded through the intended HTTPS hostname.
- Browser authentication completed successfully.
- Redirect behavior was consistent.
- The MISP interface became accessible for intelligence collection and administration.

### Lessons Learned

Application URL configuration is a critical dependency for containerized web applications.

DNS resolution, reverse-proxy behavior, HTTPS configuration, and application settings must agree for authentication and navigation to operate correctly.

---

## 4. Case Study: MISP Configuration File Permissions

### Problem

During MISP troubleshooting, configuration changes did not initially behave as expected.

This required reviewing the permissions associated with the application's configuration files.

### Investigation

Linux file ownership and access permissions were examined to determine whether the application had the required access to its configuration.

The investigation considered the difference between the host filesystem and the service identities operating inside containers.

### Resolution

Configuration file ownership and permissions were adjusted to provide the necessary application access without granting unnecessarily broad permissions.

The application was then checked to verify that its configuration could be loaded.

### Verification

Subsequent testing confirmed that:

- The application could access its required configuration.
- The intended settings were applied.
- The MISP interface and authentication workflow functioned correctly.

### Lessons Learned

Containerization does not eliminate Linux filesystem permission requirements.

Files shared between hosts and containers must have permissions compatible with the users and processes accessing them.

Changing permissions should be targeted rather than relying on unrestricted access.

---

## 5. Case Study: Docker Firewall and Network Access Restrictions

### Problem

Initial network access restrictions for MISP did not behave as expected.

The application was published through Docker, which introduced additional packet-filtering behavior compared with an application listening directly on the Linux host.

### Investigation

The effective network path was reviewed, including:

- Docker-published application ports.
- Container forwarding rules.
- Host packet-filtering behavior.
- Rules governing access from authorized networks.
- The interaction between Docker-managed rules and administrator-defined firewall controls.

Particular attention was given to Docker's `DOCKER-USER` chain.

### Root Cause

Docker manages packet-filtering rules to support container networking and published ports.

Traffic destined for published container ports may traverse Docker forwarding chains in ways that differ from conventional host-level service filtering.

The original access-control approach did not fully account for this behavior.

### Resolution

The filtering policy was adjusted using Docker-aware firewall rules.

The revised approach restricted unwanted access to the MISP HTTPS service while allowing authorized management connectivity.

The firewall configuration was also made persistent.

Exact rules and operational network ranges are intentionally excluded from this documentation.

### Verification

Connectivity was tested from authorized access paths.

The resulting behavior demonstrated that intended clients could reach the application while traffic outside the permitted scope was restricted.

### Lessons Learned

Docker networking must be considered when implementing Linux firewall policies.

A published container port should not be assumed secure merely because conventional host firewall rules have been configured.

Network security controls must be validated against the actual packet path.

---

## 6. Case Study: MISP API Integration and TLS Certificate Trust

### Problem

The OpenCTI MISP connector required additional configuration to communicate with the HTTPS-enabled MISP instance.

The laboratory used a self-signed certificate, creating a certificate trust limitation for the connector.

### Investigation

The following areas were reviewed:

- MISP HTTPS connectivity.
- Connector endpoint configuration.
- API authentication.
- TLS certificate verification behavior.
- Connector logs.

### Root Cause

The MISP server certificate was not trusted by the connector's TLS validation configuration.

This was separate from whether the API endpoint was reachable or whether the supplied credentials were valid.

### Resolution

For initial laboratory testing, the connector was configured with certificate verification disabled.

```yaml
- MISP_SSL_VERIFY=false
```

This allowed the connector to establish communication with the existing HTTPS endpoint.

### Verification

The connector authenticated with MISP and successfully retrieved intelligence events.

Imported intelligence subsequently became visible within OpenCTI.

### Remaining Security Limitation

Disabling certificate verification means the connector does not validate the server's certificate identity.

Although the connection uses HTTPS, this configuration reduces protection against server impersonation and man-in-the-middle attacks.

The recommended improvement is to establish a trusted certificate configuration and re-enable verification.

### Lessons Learned

Encryption and authentication of a remote server are separate security properties.

A successful encrypted connection does not necessarily establish that the receiving system has verified the identity of the endpoint.

Security exceptions should be documented and revisited as part of infrastructure hardening.

---

## 7. Case Study: OpenCTI Runtime and Elasticsearch Configuration

### Problem

OpenCTI required careful resource management because its application stack depends on multiple supporting services.

Elasticsearch was particularly important due to its indexing responsibilities and host-level configuration requirements.

### Investigation

The deployment was evaluated for:

- Available virtual machine memory.
- Elasticsearch heap allocation.
- Node.js runtime memory settings.
- Linux virtual memory mapping limits.
- Supporting service requirements.
- Optional services that were not immediately required.

System and container-level diagnostics were used to evaluate the operating environment.

### Resolution

The Linux virtual memory mapping limit was configured to support Elasticsearch.

```bash
sudo sysctl -w vm.max_map_count=1048576
```

The setting was made persistent.

Elasticsearch memory allocation was configured for the laboratory workload.

```dotenv
ELASTIC_MEMORY_SIZE=2G
```

The OpenCTI application also used a defined Node.js heap limit.

```yaml
- NODE_OPTIONS=--max-old-space-size=4096
```

Optional services not required for the immediate CTI objectives were left disabled to conserve resources.

### Verification

The application stack became operational, including:

- OpenCTI.
- Elasticsearch.
- RabbitMQ.
- Redis.
- Object storage.
- Background workers.
- Intelligence connectors.

During subsequent monitoring, the virtual machine retained available memory without active swap usage.

Elasticsearch remained the largest memory consumer, as expected for the search and indexing workload.

### Lessons Learned

Resource-intensive platforms require planning at both the application and operating system levels.

Container memory statistics should be interpreted alongside overall Linux memory availability.

Resource optimization should also be revisited as intelligence collections, storage requirements, and connector activity increase.

---

## 8. Case Study: OpenCTI Administrative Access Recovery

### Problem

An administrative account issue temporarily prevented normal sign-in to OpenCTI.

This disrupted interactive access to the intelligence platform.

### Investigation

The availability of the OpenCTI application was reviewed separately from its authentication behavior.

The investigation focused on whether the issue prevented the application from operating or primarily affected administrative access.

### Resolution

The OpenCTI application container was restarted during the recovery process.

Sanitized example:

```bash
sudo docker restart <OPENCTI_CONTAINER>
```

### Verification

Administrative login was restored following the restart.

The platform interface was accessible again.

### Root-Cause Limitation

The observed recovery does not conclusively establish why the administrative access issue occurred.

The restart was an effective recovery action, but the available evidence does not prove that it addressed an underlying persistent root cause.

### Lessons Learned

Restoring service availability and identifying a root cause are different tasks.

Administrative account changes should be approached carefully, and critical applications benefit from documented recovery procedures.

---

## 9. Case Study: MITRE ATT&CK Connector Startup Failure

### Problem

The MITRE ATT&CK connector failed during initialization.

The container logs displayed an exception involving service-account token handling.

### Evidence

The relevant exception was:

```text
KeyError: 'name'
```

The traceback indicated that the error occurred while the connector attempted to process information related to automatic service account creation.

### Investigation

The connector's existing Docker Compose configuration was reviewed.

The configuration included automatic service account creation:

```yaml
- CONNECTOR_AUTO_CREATE_SERVICE_ACCOUNT=true
```

The troubleshooting process focused on the relationship between the connector version, authentication settings, and service account initialization behavior.

### Probable Root Cause

The connector encountered an incompatibility or unexpected response while processing service account token information.

The available traceback identified the failing operation but did not establish the exact underlying compatibility defect.

### Resolution

Automatic service account creation was disabled.

```yaml
- CONNECTOR_AUTO_CREATE_SERVICE_ACCOUNT=false
```

The connector continued using its configured OpenCTI authentication token.

The container was recreated to apply the updated configuration.

### Verification

Following the change:

- The connector started successfully.
- OpenCTI displayed the connector as active.
- MITRE ATT&CK intelligence became available within the platform.
- Imported objects included adversary-related intelligence and ATT&CK techniques.

### Lessons Learned

Successful configuration parsing does not guarantee application compatibility.

Connector startup failures should be investigated through tracebacks and configuration settings before attempting broad changes to the application environment.

Independent validation of imported intelligence is essential.

---

## 10. Case Study: AlienVault OTX Timestamp Parsing Failure

### Problem

The AlienVault OTX connector registered successfully but failed during its initial intelligence collection.

The failure occurred while parsing the configured historical import start timestamp.

### Evidence

The relevant error was:

```text
ValueError: unconverted data remains: Z
```

The configured timestamp used a trailing UTC designator.

Sanitized example:

```dotenv
ALIENVAULT_PULSE_START_TIMESTAMP=2026-10-24T00:00:00Z
```

### Investigation

The traceback was reviewed to identify which value the connector could not process.

The exception indicated that the parser expected a datetime string without the trailing timezone designator.

### Root Cause

The connector's installed datetime parser did not accept the supplied timestamp format.

### Resolution

The timestamp was changed to the format accepted by the connector.

```dotenv
ALIENVAULT_PULSE_START_TIMESTAMP=2026-10-24T00:00:00
```

The container was recreated with the updated configuration.

### Verification

The subsequent collection successfully:

- Retrieved subscribed intelligence pulses.
- Processed the available pulse records.
- Created intelligence bundles.
- Submitted those bundles to OpenCTI.
- Completed without reported import failures.

The imported threat reports were subsequently visible in OpenCTI.

### Lessons Learned

Configuration values must match the expectations of the installed software version.

Even widely used datetime conventions may not be accepted by every connector implementation.

Detailed application logs can reveal configuration defects that are not detected by Docker Compose validation.

---

## 11. Case Study: Distinguishing Connector Health from Intelligence Ingestion

### Problem

A connector can appear active and healthy without guaranteeing that intelligence has been fully imported into OpenCTI.

This distinction became relevant when enabling scheduled external import connectors.

### Investigation

Validation was divided into several stages:

1. Container startup.
2. Connector registration.
3. Authentication and API connectivity.
4. Source data retrieval.
5. Intelligence bundle creation.
6. Queue submission.
7. Background processing.
8. Visibility of imported entities in OpenCTI.

### Findings

The CISA KEV connector successfully retrieved a vulnerability catalog and began submitting entries to OpenCTI's processing pipeline.

However, the initial logs represented an import in progress rather than proof that every record had completed indexing.

Other connectors provided additional validation through imported reports or visible entities.

### Resolution

Connector validation procedures were adjusted to distinguish successful queue submission from successful data ingestion.

Application-level verification was performed when practical.

### Lessons Learned

A running container is evidence of process availability, not complete functional success.

Threat intelligence integrations should be validated end to end, particularly when asynchronous processing is involved.

---

## 12. Case Study: Interpreting Enrichment Results Correctly

### Problem

External enrichment can return incomplete, conflicting, or differently scoped information.

This creates a risk of drawing conclusions that are not supported by the available evidence.

### Investigation

A suspicious URL imported through MISP was enriched using VirusTotal.

The associated IP address was separately investigated using AbuseIPDB.

The findings included:

- Multiple security-vendor detections for the URL.
- A DNS resolution identifying a public IPv4 address.
- No existing AbuseIPDB reports for that IP at the time of review.

### Assessment

These results evaluated different objects and different aspects of potential malicious activity.

VirusTotal provided URL reputation information, while AbuseIPDB provided IP abuse reporting information.

The absence of IP abuse reports did not invalidate the URL detections.

Likewise, URL detections did not establish that the hosting IP was dedicated to malicious activity.

### Resolution

The resulting CTI report distinguished:

- Confirmed observations.
- Source-provided classifications.
- Analytical judgments.
- Intelligence gaps.
- Recommendations requiring further validation.

### Lessons Learned

Successful enrichment is not equivalent to confirmed malicious activity.

Analysts must understand what each intelligence source measures and avoid turning reputation scores into unsupported attribution or compromise claims.

---

## 13. Operational Troubleshooting Reference

| Symptom | Initial Investigation |
|---|---|
| Container repeatedly restarts | Review container logs and configuration |
| Web interface unavailable | Check container health, port bindings, and service dependencies |
| Authentication redirects fail | Compare application URL, DNS, and HTTPS settings |
| Connector cannot authenticate | Verify endpoint configuration and token permissions |
| HTTPS connection fails | Review certificate trust and endpoint accessibility |
| Connector active but no intelligence | Inspect collection logs, queue processing, and imported entities |
| Unexpected memory usage | Compare `free -h` with Docker container statistics |
| Firewall policy ineffective | Review Docker forwarding and published-port behavior |
| Imported data appears incomplete | Check ingestion status, workers, and processing queues |
| Enrichment lacks reputation findings | Distinguish successful execution from available provider evidence |

---

## 14. Security Considerations

Troubleshooting artifacts can contain sensitive information even when they appear harmless.

Potentially sensitive content includes:

- Internal IP addresses and subnets.
- Private DNS records.
- Management usernames.
- API keys and access tokens.
- Authentication headers.
- Container environment variables.
- Public IP addresses identifying a private network.
- Internal URLs and administration endpoints.
- Raw configuration files containing credentials.

For that reason, this repository uses sanitized configuration examples and selected error messages rather than unreviewed operational logs.

Commands that expand environment variables or display complete service configurations should not be copied into public documentation without reviewing their output.

Screenshots should be permanently redacted before publication, including removal of sensitive metadata where applicable.

---

## 15. Overall Results

The troubleshooting process contributed to a stable, functional CTI laboratory with integrated intelligence collection and enrichment capabilities.

The work demonstrated practical experience with:

- Linux administration.
- Docker and Docker Compose.
- Application configuration.
- Container networking and firewall behavior.
- DNS and HTTPS troubleshooting.
- Certificate trust.
- API authentication.
- Connector compatibility.
- Resource optimization.
- Asynchronous intelligence ingestion.
- Technical validation.
- Evidence-based intelligence analysis.

The environment continues to evolve, and future changes will require additional testing and documentation.

---

## 16. Key Takeaways

**1. Investigate before changing configurations.**

Application logs, configuration files, and operational context should guide troubleshooting rather than trial-and-error changes.

**2. Make targeted changes.**

Changing one relevant setting at a time makes it easier to determine whether a modification resolved the problem.

**3. Validate the actual objective.**

A restarted container or successful API call should not be confused with a completed intelligence workflow.

**4. Document remaining uncertainty.**

If a workaround restores functionality without proving the original root cause, the limitation should be explicitly recorded.

**5. Treat security exceptions as technical debt.**

Temporary decisions such as disabling TLS certificate verification should be tracked for future remediation.

**6. Preserve analytical discipline.**

Threat intelligence platforms provide structured data and enrichment capabilities, but the analyst remains responsible for evaluating evidence and communicating uncertainty.

---

## 17. Future Improvements

Planned improvements include:

- Implementing trusted certificate verification for MISP integrations.
- Refining connector service account permissions.
- Establishing repeatable application backup and recovery procedures.
- Improving connector health monitoring.
- Reviewing indexing and storage capacity.
- Maintaining version-aware troubleshooting documentation.
- Developing a formal operational validation checklist.
- Expanding intelligence quality and correlation testing.

---

## 18. References

- [OpenCTI Documentation](https://docs.opencti.io/)
- [OpenCTI Connectors](https://github.com/OpenCTI-Platform/connectors)
- [MISP Documentation](https://www.misp-project.org/documentation/)
- [Docker Networking and Firewalls](https://docs.docker.com/engine/network/packet-filtering-firewalls/)
- [Elasticsearch Documentation](https://www.elastic.co/guide/)
- [MITRE ATT&CK](https://attack.mitre.org/)
