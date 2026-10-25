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

Example validation commands:

```bash
lsb_release -a
free -h
df -h
ip addr
```

All environment-specific IP addresses and identifying infrastructure details are intentionally excluded from this documentation.

---

## 5. Containerization Strategy

Docker and Docker Compose were selected to manage the application stack and supporting infrastructure services.

This deployment model provides:

- Reproducible application configurations.
- Independent container lifecycle management.
- Simplified service updates.
- Environment-based configuration.
- Isolation between application services.
- Centralized container logging and resource monitoring.

### Supporting Services

The OpenCTI environment uses several supporting services:

| Component | Function |
|---|---|
| Elasticsearch | Intelligence indexing and search |
| Redis | Caching and supporting application operations |
| RabbitMQ | Message queuing and asynchronous processing |
| MinIO | Object storage |
| OpenCTI Workers | Background intelligence processing |
| Intelligence Connectors | External collection and enrichment |

These services are managed through Docker Compose alongside the OpenCTI application.

---

## 6. Network Architecture and Security

The environment was designed with restricted administrative access and controlled communication between infrastructure components.

Security considerations included:

- Limiting access to administrative services.
- Restricting application exposure to authorized systems.
- Separating application workloads through virtual machines.
- Managing API authentication through dedicated credentials.
- Restricting unnecessary network access.
- Protecting sensitive configuration and environment files.

Specific IP addresses, internal domain names, and detailed network access rules are intentionally omitted from the public documentation.

The architecture diagrams represent logical communication flows rather than a complete operational network topology.

---

## 7. Infrastructure Validation

After deploying the virtual machines and application services, basic system health and resource utilization were checked.

### Memory Validation

```bash
free -h
```

This command was used to evaluate memory availability and swap utilization.

### Docker Container Health

```bash
sudo docker ps
```

This command provides a list of running containers and their operational status.

### Resource Utilization

```bash
sudo docker stats --no-stream
```

Container statistics were used to evaluate CPU and memory consumption across the application stack.

### System Services

```bash
sudo systemctl status docker
```

This command verifies that the Docker service is running.

These checks helped establish that the infrastructure was operating normally before enabling additional intelligence integrations.

---

## 8. Results

The infrastructure deployment established a functional virtualization environment capable of hosting interconnected CTI applications.

The completed environment supports:

- Independent MISP and OpenCTI workloads.
- Containerized application services.
- External intelligence collection and enrichment.
- Resource monitoring and operational troubleshooting.
- Expansion of the intelligence platform as requirements evolve.

---

## 9. Lessons Learned

The infrastructure deployment reinforced several important concepts:

- Resource planning is essential when deploying data-intensive intelligence platforms.
- Containerization simplifies application management but requires an understanding of underlying networking and dependencies.
- Separating major workloads improves operational flexibility.
- Infrastructure security should be considered during initial design rather than added after deployment.
- Monitoring application and system resources provides a baseline for future troubleshooting and capacity planning.

---

## 10. Next Steps

The following documentation provides a more detailed examination of the application deployments:

- MISP installation and configuration.
- OpenCTI installation and supporting services.
- Network security configuration.
- Intelligence connector integration.
- Troubleshooting and operational validation.

Sensitive environment-specific configuration details are excluded from this repository.
