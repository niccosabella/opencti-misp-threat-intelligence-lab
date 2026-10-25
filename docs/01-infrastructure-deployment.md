# Infrastructure Deployment

## 1. Overview

This document outlines the infrastructure deployment process for a self-hosted Cyber Threat Intelligence (CTI) laboratory using Proxmox Virtual Environment and Ubuntu Server.

The environment was designed to provide dedicated compute resources for threat intelligence platforms while maintaining logical separation between application workloads.

The infrastructure supports containerized deployments of MISP and OpenCTI, allowing the environment to be maintained, expanded, and troubleshot independently.

---

## 2. Infrastructure Design

### Virtualization Platform

Proxmox VE was selected as the virtualization platform because it provides:

- Centralized management of Linux virtual machines.
- Flexible CPU, memory, and storage allocation.
- Virtual networking capabilities.
- Snapshot and backup functionality.
- Support for expanding the laboratory without requiring additional physical servers.

### Virtual Machine Design

Dedicated Ubuntu Server virtual machines were provisioned to separate the MISP and OpenCTI workloads.

| Virtual Machine | Operating System | Primary Function |
|---|---|---|
| MISP VM | Ubuntu Server 24.04 LTS | Threat intelligence collection, event management, and sharing |
| OpenCTI VM | Ubuntu Server 24.04 LTS | Threat intelligence correlation, enrichment, and analysis |

Separating these workloads allows independent configuration, resource management, troubleshooting, and maintenance.

---

## 3. Virtual Machine Provisioning

The virtual machines were deployed through the Proxmox web management interface.

The provisioning process included:

1. Uploading the Ubuntu Server installation image to Proxmox.
2. Creating dedicated virtual machines for the CTI workloads.
3. Configuring CPU, memory, virtual disk, and network interfaces.
4. Installing Ubuntu Server.
5. Configuring administrative access.
6. Applying operating system updates.
7. Installing Docker and Docker Compose.
8. Verifying network connectivity and service availability.

### Resource Planning

Resources were allocated based on application requirements and available host capacity.

MISP primarily supports collection and management of threat intelligence data, while OpenCTI requires additional resources for intelligence storage, indexing, messaging, and processing.

The virtualized architecture allows resources to be adjusted as intelligence volume and connector activity increase.

---

## 4. Operating System Preparation

Following Ubuntu Server installation, the operating systems were updated to establish a consistent baseline.

Example:

```bash
sudo apt update
sudo apt upgrade -y
```

Administrative access was configured for authorized management of the environment.

Basic validation included checking the operating system version, available storage, memory, and network connectivity.

Example validation
