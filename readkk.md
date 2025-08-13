graph TD
    subgraph "Región Primaria (e.g., East US) - HA 99.99% con 3 AZs"
        subgraph "Availability Zone 1"
            AKS1[AKS Pod: MCAS Processing]
            ICE1[AKS Pod: ICE-XS HAR]
            CC1[AKS Pod: Control Center UI/App]
        end
        subgraph "Availability Zone 2"
            AKS2[AKS Pod: MCAS Replica]
            ICE2[AKS Pod: ICE-XS Replica]
            CC2[AKS Pod: Control Center Replica]
        end
        subgraph "Availability Zone 3"
            AKS3[AKS Pod: MCAS Replica]
            ICE3[AKS Pod: ICE-XS Replica]
            CC3[AKS Pod: Control Center Replica]
        end
        
        LB[Azure Load Balancer / Application Gateway] --> AKS1 & AKS2 & AKS3
        LB --> ICE1 & ICE2 & ICE3
        LB --> CC1 & CC2 & CC3
        
        RTDB[Azure PostgreSQL Flexible Server - RT DB (Sync Replica across AZs)] --> AKS1 & AKS2 & AKS3
        NRTDB[Azure PostgreSQL Flexible Server - NRT DB (Offload from RT)] --> AKS1 & AKS2 & AKS3
        CCDB[Azure PostgreSQL Flexible Server - Control Center DB] --> CC1 & CC2 & CC3
        
        HSM[Azure Dedicated HSM (Farm for Crypto/Signing)] --> ICE1 & ICE2 & ICE3
        MSG[Azure Service Bus (Messaging: Replaces IBM MQ)] --> AKS1 & AKS2 & AKS3
        
        MON[Azure Monitor + App Insights (Metrics, Alerts)] -.-> AKS1 & RTDB & HSM
        KV[Azure Key Vault (Secrets, TLS Certs)] -.-> All Components
        FW[Azure Firewall / NSGs (Security)] -.-> LB
    end
    
    subgraph "Región Secundaria (DR, e.g., West US) - Async Replication"
        AKS_DR[AKS Cluster DR (Passive/Failover)]
        RTDB_DR[PostgreSQL Replica (Async via BDR)]
        NRTDB_DR[NRT DB Replica]
        CCDB_DR[Control Center DB Replica]
        HSM_DR[HSM DR Farm]
        
        ASR[Azure Site Recovery (Orquestación de Failover)] --> AKS_DR & RTDB_DR
    end
    
    RTDB -->|Async Replication (BDR for >10ms latency)| RTDB_DR
    NRTDB -->|Async Replication| NRTDB_DR
    CCDB -->|Async Replication| CCDB_DR
    HSM -->|Replication| HSM_DR
    
    WAN[VNet Peering / ExpressRoute (Baja Latencia <10ms intra-región)] -.-> Región Primaria & Región Secundaria
    
    classDef primary fill:#a8ddf3,stroke:#333,stroke-width:2px;
    classDef dr fill:#f9d7a8,stroke:#333,stroke-width:2px;
    classDef db fill:#b3e6b3,stroke:#333;
    classDef security fill:#f4b3b3,stroke:#333;
    class AKS1,AKS2,AKS3,ICE1,ICE2,ICE3,CC1,CC2,CC3,AKS_DR primary;
    class RTDB,NRTDB,CCDB,RTDB_DR,NRTDB_DR,CCDB_DR db;
    class HSM,HSM_DR,MSG security;
    class LB,MON,KV,FW,ASR security;
