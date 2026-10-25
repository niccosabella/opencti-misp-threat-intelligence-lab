# Network Security and Access Controls

## 1. Overview

This document describes the network security architecture, access restrictions, authentication practices, and security considerations implemented within the self-hosted Cyber Threat Intelligence (CTI) laboratory.

The environment hosts MISP and OpenCTI on dedicated Linux virtual machines managed through Proxmox Virtual Environment.

The network security design focuses on restricting administrative access, limiting unnecessary service exposure, and controlling communication between intelligence platforms.

All network examples are sanitized. Actual addresses, internal domains, usernames, credentials, and operational firewall rules are intentionally excluded.

---

## 2. Security Objectives

The primary security objectives were to:

- Restrict access to authorized management systems.
- Avoid unnecessary exposure of application services.
- Control network communication between CTI platforms.
- Implement encrypted administrative connections.
- Separate application authentication from connector authentication.
- Protect sensitive configuration values and API credentials.
- Understand how Docker networking affects host firewall policies.
- Validate security controls through configuration review and connectivity testing.

---

## 3. Network Architecture

### Logical Network Design

The laboratory operates within a privately managed network environment.

Dedicated Linux virtual machines host the MISP and OpenCTI application stacks.

Communication between the platforms is required for threat intelligence collection and synchronization, while administrative access is restricted to authorized systems.

The design separates several logical functions:

| Function | Purpose |
|---|---|
| Virtualization management | Administration of virtual infrastructure |
| MISP application access | Intelligence collection and management |
| OpenCTI application access | Intelligence analysis and reporting |
| API communications | Authenticated communication between platforms |
| External intelligence access | Retrieval of publicly available intelligence |
| Administrative access | Authorized system maintenance |

These functions are represented logically rather than as a complete operational network topology.

---

## 4. MISP Network Access

### HTTPS Application Access

MISP is accessed through an internal HTTPS endpoint.

A fictional documentation hostname is used throughout this repository:

```text
https://misp.example.test
```

The operational environment uses internal name resolution to provide consistent access to the application.

### Application Exposure

MISP requires browser access for authorized users and API connectivity for supported integrations.

Network controls were configured to limit access to the published HTTPS service.

### Docker Networking Considerations

Docker creates its own networking and packet-filtering rules when containers publish ports.

As a result, traditional host firewall rules may not operate as expected for container-forwarded traffic.

During deployment, the effective forwarding path was reviewed and access restrictions were implemented through a Docker-aware approach.

The `DOCKER-USER` chain was relevant to filtering forwarded Docker traffic before it reached application containers.

The exact firewall rules are excluded because they reference operational network boundaries.

---

## 5. OpenCTI Network Access

### Restricted Application Binding

The OpenCTI application was configured to bind its published management interface to the Linux host's loopback address.

A representative Docker configuration is:

```yaml
ports:
  - "127.0.0.1:8080:8080"
```

This prevents the published application port from listening directly on the host's external network interfaces.

### SSH Port Forwarding

Authorized administrators access the application using SSH local port forwarding.

A sanitized example is:

```bash
ssh -N -L 8080:127.0.0.1:8080 admin@opencti.example.test
```

Once the tunnel is established, the application is accessed through:

```text
http://localhost:8080
```

The SSH connection encrypts communication between the administrator's workstation and the OpenCTI host.

This approach avoids directly publishing the OpenCTI web interface across the management network.

### Administrative Security

SSH access should be restricted to trusted users and systems.

Additional hardening opportunities include:

- SSH key-based authentication.
- Disabling direct root login.
- Reviewing account permissions.
- Restricting permitted SSH clients.
- Monitoring authentication activity.

These measures are recommended security practices and are not all represented as verified controls in the laboratory.

---

## 6. DNS Configuration

Internal DNS was used to provide consistent application name resolution.

This helped avoid problems caused by accessing services through inconsistent hostnames or direct IP addresses.

Example:

```text
misp.example.test
```

Consistent name resolution was particularly important for MISP because its application base URL affected authentication and redirect behavior.

Internal DNS records were not published externally.

---

## 7. TLS and Certificate Management

### MISP HTTPS

MISP was configured to use HTTPS for application and API connectivity.

During initial deployment, the environment used a self-signed TLS certificate.

Although self-signed certificates can provide encrypted transport, clients must establish trust separately to authenticate the server's identity.

### Connector Certificate Verification

The MISP connector initially required a temporary exception to certificate verification.

The relevant setting was:

```yaml
- MISP_SSL_VERIFY=false
```

This permitted the connector to communicate with the laboratory's self-signed HTTPS endpoint.

However, disabling certificate verification weakens protection against impersonation and man-in-the-middle attacks.

### Recommended Improvement

A future improvement is to establish an internal certificate authority or deploy an appropriately trusted certificate.

The long-term objective is to enable:

```yaml
- MISP_SSL_VERIFY=true
```

This is a planned hardening improvement, not a control already verified as operational.

---

## 8. API Authentication and Service Accounts

### Separate Authentication Contexts

MISP and OpenCTI use API credentials to authenticate integration requests.

The deployment uses dedicated connector identities where configured, helping separate automated intelligence processing from interactive administration.

### Credential Protection

API keys, tokens, passwords, and encryption secrets were maintained outside published configuration examples.

Documentation references environment variables instead of embedding actual values.

Example:

```yaml
environment:
  - OPENCTI_TOKEN=${OPENCTI_CONNECTOR_TOKEN}
  - MISP_KEY=${MISP_API_KEY}
```

### Least-Privilege Considerations

Connector credentials should be restricted to the permissions required for their function.

The laboratory includes dedicated connector service accounts, although some configurations originally inherited administrator-level tokens.

Reviewing remaining connector privileges is an ongoing improvement.

---

## 9. Security Troubleshooting

### 9.1 MISP Application Redirect Issues

**Problem**

Authentication and navigation did not initially behave as expected when accessing MISP through HTTPS.

**Investigation**

Application base URL settings were compared with the internal DNS hostname and container environment configuration.

**Root Cause**

Inconsistent application URL configuration contributed to the redirect and authentication problems.

**Resolution**

Relevant base URL values were aligned with the intended HTTPS hostname.

**Verification**

Authenticated browser access operated successfully.

---

### 9.2 Docker Firewall Behavior

**Problem**

Initial access restrictions did not consistently match the intended network policy.

**Investigation**

Docker port publishing, forwarding behavior, and host packet-filtering rules were reviewed.

**Root Cause**

Container traffic traversed Docker-managed networking rules, which affected the enforcement path.

**Resolution**

The access policy was adjusted to account for Docker's forwarding behavior.

**Verification**

Authorized application access was confirmed while unwanted access paths were restricted.

---

### 9.3 TLS Certificate Trust

**Problem**

The OpenCTI MISP connector encountered a certificate trust limitation when communicating with MISP over HTTPS.

**Investigation**

TLS verification settings and the certificate trust model were reviewed.

**Resolution**

Certificate verification was temporarily disabled for the trusted laboratory connection.

**Verification**

The connector successfully authenticated and retrieved intelligence.

**Remaining Risk**

The server certificate is not validated by the connector while this exception remains enabled.

---

## 10. Security Validation

Validation focused on ensuring that application connectivity worked through intended access paths.

| Security Area | Validation |
|---|---|
| MISP HTTPS | Authorized browser connectivity |
| Internal DNS | Successful hostname resolution |
| OpenCTI access | Local port forwarding through SSH |
| API authentication | Successful connector registration and ingestion |
| Docker networking | Review of published ports and access restrictions |
| TLS | Encrypted MISP access, with known trust limitation |
| Secrets | No operational credentials included in documentation |

These results describe observed laboratory behavior rather than a comprehensive penetration test or formal security audit.

---

## 11. Results

The security configuration established controlled access to the CTI laboratory while preserving required communication between intelligence platforms.

The implementation demonstrates:

- Restricted OpenCTI management access.
- HTTPS-enabled MISP application connectivity.
- Internal DNS-based service access.
- Docker-aware firewall troubleshooting.
- Authenticated integration between intelligence platforms.
- Separation of interactive and automated access where configured.
- Awareness of remaining TLS and least-privilege improvements.

---

## 12. Future Security Improvements

Potential improvements include:

1. Establishing trusted TLS certificate management.
2. Reviewing and minimizing connector API permissions.
3. Expanding SSH key-based administrative authentication.
4. Developing repeatable firewall validation procedures.
5. Improving authentication and connector activity monitoring.
6. Establishing documented backup and recovery procedures.
7. Periodically reviewing exposed ports and service configurations.

---

## 13. References

- [Docker Packet Filtering and Firewalls](https://docs.docker.com/engine/network/packet-filtering-firewalls/)
- [OpenSSH Port Forwarding](https://man.openbsd.org/ssh)
- [OpenCTI Documentation](https://docs.opencti.io/)
- [MISP Project](https://www.misp-project.org/)
- [OWASP Transport Layer Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html)
