# CTI Lab Architecture

This directory contains sanitized architecture diagrams and documentation for the self-hosted OpenCTI and MISP threat intelligence environment.

## Logical Architecture

```mermaid
flowchart TB
    subgraph EXT["External Threat Intelligence"]
        FEEDS["Threat Intelligence Feeds"]
        MITRE["MITRE ATT&CK"]
        OTX["AlienVault OTX"]
        CISA["CISA KEV"]
        VT["VirusTotal"]
        URL["urlscan.io"]
        ABUSE["AbuseIPDB"]
    end

    subgraph PROXMOX["Proxmox Virtual Environment"]
        direction LR
        subgraph MISP_VM["MISP Virtual Machine"]
            MISP["MISP Platform"]
            MISP_DB["Database and Supporting Services"]
        end
        subgraph OPENCTI_VM["OpenCTI Virtual Machine"]
            OPENCTI["OpenCTI Platform"]
            STORAGE["Elasticsearch / MinIO"]
            QUEUE["RabbitMQ / Redis"]
            WORKERS["Workers and Connectors"]
        end
    end

    subgraph OUTPUT["Intelligence Analysis"]
        IOC["Indicators and Observables"]
        CORR["Intelligence Correlation"]
        REPORT["CTI Reports"]
    end

    FEEDS --> MISP
    MISP_DB --- MISP
    MISP -->|"MISP API Connector"| WORKERS

    MITRE --> WORKERS
    OTX --> WORKERS
    CISA --> WORKERS
    VT <-->|"Enrichment"| WORKERS
    URL <-->|"Enrichment"| WORKERS
    ABUSE <-->|"Enrichment"| WORKERS

    WORKERS <--> OPENCTI
    OPENCTI --- STORAGE
    OPENCTI --- QUEUE

    OPENCTI --> IOC
    OPENCTI --> CORR
    OPENCTI --> REPORT

    classDef platform fill:#173b5e,stroke:#57a0d3,color:#ffffff
    classDef source fill:#264653,stroke:#69b3b0,color:#ffffff
    classDef output fill:#274a39,stroke:#78b99a,color:#ffffff

    class MISP,OPENCTI platform
    class FEEDS,MITRE,OTX,CISA,VT,URL,ABUSE source
    class IOC,CORR,REPORT output
```
