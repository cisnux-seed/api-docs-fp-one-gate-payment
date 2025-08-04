### High Level Architecture (Production)
```mermaid
graph TB
%% External Users and Systems
subgraph "External Zone"
User[👤 End Users<br/>Mobile/Web]
BankIndonesia[🏛️ Bank Indonesia<br/>API Key Access]
OJK[⚖️ OJK Financial Authority<br/>API Key Access]

subgraph "External E-Wallet Services"
GopayExternal[💳 GoPay API<br/>api.gopay.com]
ShopeeExternal[🛒 ShopeePay API<br/>api.shopeepay.com]
end
end

%% Internet Gateway and CDN
subgraph "Edge Layer"
CloudFlare[☁️ CloudFlare CDN<br/>DDoS Protection]
DNS[🌐 DNS Management<br/>skynux.fun]
end

%% OpenShift Container Platform - Multi-Node
subgraph "OpenShift Container Platform - GCP Asia-Southeast2 (Multi-Node)"
direction TB

subgraph "Control Plane (3 Masters)"
Master1[🎛️ Master Node 1<br/>API Server, etcd]
Master2[🎛️ Master Node 2<br/>API Server, etcd]
Master3[🎛️ Master Node 3<br/>API Server, etcd]
end

subgraph "Worker Nodes (3 Workers)"
Worker1[💼 Worker Node 1<br/>Application Pods]
Worker2[💼 Worker Node 2<br/>Application Pods]
Worker3[💼 Worker Node 3<br/>Application Pods]
end

%% Ingress Layer
subgraph "Ingress & Gateway"
Router[🔀 OpenShift Router<br/>HAProxy Ingress]
Kong[🦍 Kong API Gateway<br/>Rate Limiting, Auth]
end

%% Application Services
subgraph "Microservices Layer"
AuthService[🔐 Authentication Service<br/>JWT, Refresh Tokens]
PaymentService[💰 Payment Service<br/>Balance, Topup]
TransactionService[📊 Transaction Service<br/>History, Reports]
end

%% Data Layer
subgraph "Data & Storage"
PostgresCluster[🐘 PostgreSQL Cluster<br/>Primary DB]
RedisCluster[⚡ Redis Cluster<br/>Session & Cache]
KafkaCluster[📨 Kafka Cluster<br/>Event Streaming]
end

%% Observability Stack
subgraph "Observability & Monitoring"
Prometheus[📊 Prometheus<br/>Metrics Collection]
Grafana[📈 Grafana<br/>Dashboards]
Loki[📝 Loki<br/>Log Aggregation]
Tempo[🔍 Tempo<br/>Distributed Tracing]
OtelCol[📡 OpenTelemetry<br/>Collector]
end

%% DevOps Tools
subgraph "DevOps & CI/CD"
Jenkins[🔧 Jenkins<br/>CI/CD Pipeline]
SonarQube[🔍 SonarQube<br/>Code Quality]
KafkaUI[📺 Kafka UI<br/>Stream Monitoring]
end
end

%% Connections
User --> CloudFlare
BankIndonesia --> CloudFlare
OJK --> CloudFlare
CloudFlare --> DNS
DNS --> Router
Router --> Kong

Kong --> AuthService
Kong --> PaymentService
Kong --> TransactionService

AuthService --> PostgresCluster
PaymentService --> PostgresCluster
PaymentService --> RedisCluster
PaymentService --> KafkaCluster
TransactionService --> PostgresCluster
TransactionService --> RedisCluster
TransactionService --> KafkaCluster

%% External service calls (outbound from cluster)
PaymentService --> GopayExternal
PaymentService --> ShopeeExternal

%% Observability connections
AuthService -.-> OtelCol
PaymentService -.-> OtelCol
TransactionService -.-> OtelCol
OtelCol --> Prometheus
OtelCol --> Loki
OtelCol --> Tempo
Prometheus --> Grafana
Loki --> Grafana
Tempo --> Grafana

%% DevOps connections
Jenkins -.-> SonarQube
KafkaCluster -.-> KafkaUI
```

### High Level Architecture (Development)
```mermaid
graph TB
    %% External Users and Systems
    subgraph "External Zone"
        User[👤 End Users<br/>Mobile/Web]
        BankIndonesia[🏛️ Bank Indonesia<br/>API Key Access]
        OJK[⚖️ OJK Financial Authority<br/>API Key Access]
        
        subgraph "External E-Wallet Services"
            GopayExternal[💳 GoPay API<br/>api.gopay.com]
            ShopeeExternal[🛒 ShopeePay API<br/>api.shopeepay.com]
        end
    end

    %% Direct DNS (No CloudFlare)
    subgraph "DNS Layer"
        DNS[🌐 DNS Management<br/>skynux.fun<br/>Direct Resolution]
    end

    %% OpenShift Container Platform - Single Node
    subgraph "OpenShift Container Platform - GCP Asia-Southeast2 (Single Node)"
        direction TB
        
        subgraph "All-in-One Node"
            SingleNode[🎯 Single Node<br/>Master + Worker<br/>Control Plane + Apps]
        end
        
        %% Ingress Layer
        subgraph "Ingress & Gateway"
            Router[🔀 OpenShift Router<br/>HAProxy Ingress]
            Kong[🦍 Kong API Gateway<br/>Rate Limiting, Auth]
        end

        %% Application Services
        subgraph "Microservices Layer"
            AuthService[🔐 Authentication Service<br/>JWT, Refresh Tokens]
            PaymentService[💰 Payment Service<br/>Balance, Topup]
            TransactionService[📊 Transaction Service<br/>History, Reports]
        end

        %% Data Layer
        subgraph "Data & Storage"
            PostgresSingle[🐘 PostgreSQL Single<br/>Primary DB]
            RedisSingle[⚡ Redis Single<br/>Session & Cache]
            KafkaSingle[📨 Kafka Single<br/>Event Streaming]
        end

        %% Observability Stack
        subgraph "Observability & Monitoring"
            Prometheus[📊 Prometheus<br/>Metrics Collection]
            Grafana[📈 Grafana<br/>Dashboards]
            Loki[📝 Loki<br/>Log Aggregation]
            Tempo[🔍 Tempo<br/>Distributed Tracing]
            OtelCol[📡 OpenTelemetry<br/>Collector]
        end

        %% DevOps Tools
        subgraph "DevOps & CI/CD"
            Jenkins[🔧 Jenkins<br/>CI/CD Pipeline]
            SonarQube[🔍 SonarQube<br/>Code Quality]
            KafkaUI[📺 Kafka UI<br/>Stream Monitoring]
        end
    end

    %% Connections (Direct, No CloudFlare)
    User --> DNS
    BankIndonesia --> DNS
    OJK --> DNS
    DNS --> Router
    Router --> Kong
    
    Kong --> AuthService
    Kong --> PaymentService
    Kong --> TransactionService

    AuthService --> PostgresSingle
    PaymentService --> PostgresSingle
    PaymentService --> RedisSingle
    PaymentService --> KafkaSingle
    TransactionService --> PostgresSingle
    TransactionService --> RedisSingle
    TransactionService --> KafkaSingle

    %% External service calls (outbound from cluster)
    PaymentService --> GopayExternal
    PaymentService --> ShopeeExternal

    %% Observability connections
    AuthService -.-> OtelCol
    PaymentService -.-> OtelCol
    TransactionService -.-> OtelCol
    OtelCol --> Prometheus
    OtelCol --> Loki
    OtelCol --> Tempo
    Prometheus --> Grafana
    Loki --> Grafana
    Tempo --> Grafana

    %% DevOps connections
    Jenkins -.-> SonarQube
    KafkaSingle -.-> KafkaUI
```

### ERD
```mermaid
erDiagram
    users ||--o{ authentications : has
    users ||--|| accounts : has
    accounts ||--o{ historical_transactions : generates
    users ||--o{ historical_transactions : performs
    external_services ||--o{ api_keys : has
    api_keys ||--o{ api_access_logs : generates
    external_services ||--o{ api_access_logs : accesses

    users {
        serial id PK
        varchar username UK "not null unique"
        varchar email UK "not null unique"
        varchar phone UK "not null unique"
        varchar password "not null, hashed"
        timestamp created_at "default now()"
        timestamp updated_at "default now()"
    }

    authentications {
        varchar token PK "512 chars"
        serial user_id FK "references users(id)"
        timestamp created_at "default now()"
    }

    accounts {
        varchar id PK "UUID, default gen_random_uuid()"
        bigint user_id FK "not null unique, references users(id)"
        decimal balance "15,2 precision, default 0.00"
        varchar currency "3 chars, default IDR"
        account_status_enum account_status "ACTIVE,SUSPENDED,CLOSED, default ACTIVE"
        timestamp created_at "default now()"
        timestamp updated_at "default now()"
    }

    historical_transactions {
        varchar id PK "UUID, default gen_random_uuid()"
        bigint user_id FK "not null, references users(id)"
        varchar account_id FK "not null, references accounts(id)"
        varchar transaction_id UK "50 chars, unique"
        transaction_type_enum transaction_type "TOPUP,PAYMENT,REFUND,TRANSFER"
        transaction_status_enum transaction_status "PENDING,SUCCESS,FAILED,CANCELLED"
        decimal amount "15,2 precision, not null"
        decimal balance_before "15,2 precision, not null"
        decimal balance_after "15,2 precision, not null"
        varchar currency "3 chars, default IDR"
        text description "transaction description"
        varchar external_reference "255 chars, external system reference"
        payment_method_enum payment_method "GOPAY,SHOPEE_PAY"
        text metadata "additional transaction data"
        boolean is_accessible_external "default true"
        timestamp created_at "default now()"
        timestamp updated_at "default now()"
    }

    external_services {
        varchar id PK "UUID, default gen_random_uuid()"
        varchar company UK "255 chars, not null unique"
        varchar service_type "50 chars, default HISTORICAL_TRANSACTION"
        varchar contact_email "255 chars, not null"
        text description "service description"
        boolean is_active "default true"
        timestamp created_at "default now()"
        timestamp updated_at "default now()"
    }

    api_keys {
        varchar id PK "UUID, default gen_random_uuid()"
        varchar external_service_id FK "not null unique, references external_services(id)"
        varchar key_name "255 chars, not null"
        permissions_enum permissions "READ_TRANSACTIONS,READ_REPORTS,READ_TRANSACTIONS,READ_REPORTS"
        integer rate_limit_count "optional"
        rate_limit_unit_enum rate_limit_unit "second,minute,hour,day, default minute"
        timestamp expires_at "expiration date, optional"
        boolean is_active "default true"
        timestamp created_at "default now()"
        timestamp updated_at "default now()"
    }

    api_access_logs {
        varchar id PK "UUID, default gen_random_uuid()"
        varchar external_service_id FK "not null, references external_services(id)"
        varchar api_key_id FK "not null, references api_keys(id)"
        varchar endpoint "255 chars, accessed endpoint"
        http_method_enum http_method "GET,POST,PUT,DELETE,PATCH"
        text ip_address "client IP address"
        text user_agent "client user agent"
        integer response_status "HTTP status code"
        timestamp created_at "default now()"
    }
```
### Network Architecture & Security
```mermaid
graph TB
subgraph "GCP Project: fp-secure-api-gateway"
subgraph "VPC: ocp-one-gate-payment-network"
subgraph "Control Plane Subnet: 10.0.0.0/19"
Master1[🎛️ Master Node 1<br/>API Server, etcd, Controller]
Master2[🎛️ Master Node 2<br/>API Server, etcd, Controller]
Master3[🎛️ Master Node 3<br/>API Server, etcd, Controller]
end

subgraph "Worker Subnet: 10.0.32.0/19"
Worker1[💼 Worker Node 1<br/>Application Pods]
Worker2[💼 Worker Node 2<br/>Application Pods]
Worker3[💼 Worker Node 3<br/>Application Pods]
end

subgraph "Load Balancers"
ExtLB[🌐 External LB<br/>API: 6443]
IntLB[🔒 Internal LB<br/>API Internal: 6443]
end

subgraph "Network Services"
CloudRouter[🔀 Cloud Router<br/>NAT Gateway]
CloudNAT[🌐 Cloud NAT<br/>Outbound Internet]
end
end
end

subgraph "Security Zones"
Firewall[🔥 Firewall Rules<br/>- Internal: All protocols<br/>- SSH: Port 22<br/>- API: Port 6443<br/>- HTTP/S: 80,443]
end

Internet[🌍 Internet] --> ExtLB
ExtLB --> Master1
ExtLB --> Master2
ExtLB --> Master3
IntLB --> Master1
IntLB --> Master2
IntLB --> Master3

Master1 --> Worker1
Master2 --> Worker2
Master3 --> Worker3

Worker1 --> CloudRouter
Worker2 --> CloudRouter
Worker3 --> CloudRouter
CloudRouter --> CloudNAT
CloudNAT --> Internet
```

### Database Architecture & Data Flow
```mermaid
---
config:
  layout: dagre
---
flowchart TB
 subgraph subGraph0["Application Layer"]
        AuthApp["🔐 Authentication Service"]
        PaymentApp["💰 Payment Service"]
        TransactionApp["📊 Transaction Service"]
  end
 subgraph subGraph1["Data Access Layer"]
        ConnPool["🏊 Connection Pool<br>HikariCP"]
  end
 subgraph subGraph2["PostgreSQL Primary"]
        PG1["🐘 PostgreSQL 17<br>Primary Instance<br>Read/Write"]
        PGData1[("📁 Data Volume<br>20Gi PVC")]
  end
 subgraph subGraph3["Redis Cache Cluster"]
        Redis1["⚡ Redis Master<br>Session Store"]
        RedisData[("💾 Cache Data<br>In-Memory")]
  end
 subgraph subGraph4["Message Queue"]
        Kafka1["📨 Kafka Broker<br>Event Streaming"]
        KafkaData[("📂 Kafka Logs<br>20Gi PVC")]
  end
 subgraph subGraph5["Database Cluster"]
        subGraph2
        subGraph3
        subGraph4
  end
 subgraph subGraph6["Database Schema"]
        Users["👤 users<br>- id, username, email<br>- phone, password"]
        Accounts["💳 accounts<br>- id, user_id, balance<br>- currency, status"]
        Transactions["📝 historical_transactions<br>- id, user_id, amount<br>- type, status, metadata"]
        ApiKeys["🔑 api_keys<br>- id, external_service_id<br>- permissions, rate_limits"]
        ExtServices["🏢 external_services<br>- id, company, service_type<br>- contact_email, is_active"]
        AccessLogs["📋 api_access_logs<br>- id, endpoint, method<br>- ip_address, response_status"]
  end
    AuthApp --> ConnPool & Redis1
    PaymentApp --> ConnPool & Redis1 & Kafka1
    TransactionApp --> ConnPool & Kafka1
    ConnPool --> PG1
    PG1 --> PGData1 & Users & Accounts & Transactions & ApiKeys & ExtServices & AccessLogs
    Redis1 --> RedisData
    Kafka1 --> KafkaData
```

### External Service Architecture
```mermaid
graph TB
    subgraph "One Gate Payment Platform"
        PaymentService[💰 Payment Service<br/>Core Business Logic]
        ExtServiceLayer[🔌 External Service Layer<br/>Adapters & Clients]
    end
    
    subgraph "API Security & Rate Limiting"
        APIGateway[🦍 Kong Gateway<br/>- Authentication<br/>- Rate Limiting<br/>- Request/Response Transform]
    end
    
    subgraph "External E-Wallet Providers"
        subgraph "GoPay Integration"
            GopayAuth[🔐 GoPay OAuth2<br/>client_credentials]
            GopayAPI[💳 GoPay API<br/>- Balance Check<br/>- Top Up Requests]
        end
        
        subgraph "ShopeePay Integration"
            ShopeeAuth[🔐 ShopeePay JWT<br/>Signature + Timestamp]
            ShopeeAPI[🛒 ShopeePay API<br/>- Account Balance<br/>- Wallet Top Up]
        end
        

    end
    
    subgraph "Government API Access (Inbound)"
        BankIndonesia[🏛️ Bank Indonesia<br/>Transaction Monitoring<br/>API Key: a1b2c3d4...<br/>READ_TRANSACTIONS]
        OJK[⚖️ OJK Financial Authority<br/>Compliance Reporting<br/>API Key: f9e8d7c6...<br/>READ_TRANSACTIONS,READ_REPORTS]
    end
    
    subgraph "API Rate Limits & Security"
        subgraph "Consumer: bank-indonesia"
            BiLimits[📊 Rate Limits<br/>- 100/minute<br/>- 1000/hour<br/>- 10000/day]
        end
        
        subgraph "Consumer: ojk"
            OjkLimits[📊 Rate Limits<br/>- 200/minute<br/>- 2000/hour<br/>- 20000/day]
        end
    end
    
    PaymentService --> ExtServiceLayer
    ExtServiceLayer --> APIGateway
    
    ExtServiceLayer --> GopayAuth
    GopayAuth --> GopayAPI
    
    ExtServiceLayer --> ShopeeAuth
    ShopeeAuth --> ShopeeAPI
    
    %% Government agencies access via API Gateway (inbound)
    BankIndonesia --> APIGateway
    OJK --> APIGateway
    APIGateway --> TransactionService
    
    BankIndonesia -.-> BiLimits
    OJK -.-> OjkLimits
```

### DevOps CI/CD Pipeline
```mermaid
graph LR
    subgraph "Source Control"
        Git[📚 Git Repository<br/>Source Code]
        Webhook[🔔 Git Webhook<br/>Push Events]
    end
    
    subgraph "CI/CD Pipeline - Jenkins"
        subgraph "Build Stage"
            Checkout[📥 Checkout Code]
            Test[🧪 Unit Tests<br/>Integration Tests]
            SonarScan[🔍 SonarQube Scan<br/>Code Quality]
        end
        
        subgraph "Build & Package"
            DockerBuild[🐳 Docker Build<br/>Multi-stage Build]
            ImageScan[🔒 Security Scan<br/>Vulnerability Check]
            Registry[📦 Image Registry<br/>Harbor/Quay]
        end
        
        subgraph "Deploy Stage"
            Deploy[🚀 Deploy to OpenShift<br/>Rolling Update]
            Verify[✅ Health Check<br/>Smoke Tests]
        end
    end
    
    subgraph "Environment Progression"
        Dev[🧪 Development<br/>one-gate-payment NS]
        Staging[🎭 Staging<br/>staging NS]
        Prod[🎯 Production<br/>production NS]
    end
    
    subgraph "Monitoring & Alerts"
        Monitor[📊 Prometheus Alerts<br/>Deployment Status]
        Notify[📧 Notifications<br/>Slack/Email]
    end
    
    Git --> Webhook
    Webhook --> Checkout
    Checkout --> Test
    Test --> SonarScan
    SonarScan --> DockerBuild
    DockerBuild --> ImageScan
    ImageScan --> Registry
    Registry --> Deploy
    Deploy --> Verify
    
    Deploy --> Dev
    Dev --> Staging
    Staging --> Prod
    
    Verify --> Monitor
    Monitor --> Notify
```

### Security Architecture
```mermaid
graph TB
    subgraph "External Security"
        CloudFlare[☁️ CloudFlare<br/>- DDoS Protection<br/>- WAF Rules<br/>- SSL/TLS Termination]
        DNS[🌐 DNS Security<br/>- DNSSEC<br/>- DNS Filtering]
    end
    
    subgraph "Network Security"
        subgraph "OpenShift Security"
            RBAC[🔐 RBAC<br/>Role-Based Access]
            SCC[🛡️ Security Context Constraints<br/>Pod Security]
            NetworkPolicy[🔒 Network Policies<br/>Pod-to-Pod Communication]
            Secrets[🔑 OpenShift Secrets<br/>Encrypted at Rest]
        end
        
        subgraph "API Gateway Security"
            KongAuth[🦍 Kong Authentication<br/>- JWT Validation<br/>- API Key Management<br/>- Rate Limiting]
            CORS[🌐 CORS Policy<br/>Cross-Origin Protection]
            RequestTransform[🔄 Request Transform<br/>Header Sanitization]
        end
    end
    
    subgraph "Application Security"
        subgraph "Authentication Layer"
            JWT[🎫 JWT Tokens<br/>- Access Token 15min<br/>- Refresh Token 7 days]
            PassHash[🔒 Password Hashing<br/>Argon2ID]
            SessionMgmt[🗂️ Session Management<br/>Redis Store]
        end
        
        subgraph "Data Security"
            Encryption[🔐 Database Encryption<br/>- Data at Rest<br/>- Connection Encryption]
            PII[🛡️ PII Protection<br/>- Data Masking<br/>- Access Logging]
            Backup[💾 Secure Backups<br/>Encrypted Storage]
        end
    end
    
    subgraph "Compliance & Auditing"
        APILogs[📋 API Access Logs<br/>- IP Tracking<br/>- Request/Response<br/>- Rate Limit Events]
        AuditTrail[📊 Audit Trail<br/>- User Actions<br/>- Data Changes<br/>- System Events]
        Compliance[⚖️ Regulatory Compliance<br/>- Bank Indonesia<br/>- OJK Requirements]
    end
    
    CloudFlare --> DNS
    DNS --> RBAC
    RBAC --> SCC
    SCC --> NetworkPolicy
    NetworkPolicy --> KongAuth
    KongAuth --> CORS
    CORS --> RequestTransform
    RequestTransform --> JWT
    JWT --> PassHash
    PassHash --> SessionMgmt
    SessionMgmt --> Encryption
    Encryption --> PII
    PII --> Backup
    Backup --> APILogs
    APILogs --> AuditTrail
    AuditTrail --> Compliance
```

### Deployment & Scaling Strategy
```mermaid
graph TB
    subgraph "Horizontal Pod Autoscaler (HPA)"
        HPAAuth[📈 Auth Service HPA<br/>CPU: 70% threshold<br/>Min: 2, Max: 10 pods]
        HPAPayment[📈 Payment Service HPA<br/>CPU: 70% threshold<br/>Min: 3, Max: 15 pods]
        HPATransaction[📈 Transaction Service HPA<br/>CPU: 70% threshold<br/>Min: 2, Max: 8 pods]
    end
    
    subgraph "Resource Management"
        subgraph "Authentication Service"
            AuthPods[🔐 Auth Pods<br/>Requests: 200m CPU, 256Mi<br/>Limits: 500m CPU, 512Mi]
        end
        
        subgraph "Payment Service"  
            PaymentPods[💰 Payment Pods<br/>Requests: 300m CPU, 512Mi<br/>Limits: 1000m CPU, 1Gi]
        end
        
        subgraph "Transaction Service"
            TransactionPods[📊 Transaction Pods<br/>Requests: 200m CPU, 256Mi<br/>Limits: 500m CPU, 512Mi]
        end
    end
    
    subgraph "Database Scaling"
        PostgresHA[🐘 PostgreSQL<br/>Single Instance<br/>Memory: 2Gi, CPU: 1000m<br/>Storage: 20Gi]
        RedisCluster[⚡ Redis Cluster<br/>Memory: 512Mi<br/>Maxmemory Policy: allkeys-lru]
        KafkaPartitions[📨 Kafka Scaling<br/>Partitions: 3 per topic<br/>Replication: 1<br/>Storage: 20Gi]
    end
    
    subgraph "Load Balancing"
        ServiceMesh[🕸️ Service Discovery<br/>OpenShift Services<br/>Round-robin Load Balancing]
        Ingress[🚪 Ingress Controllers<br/>HAProxy Router<br/>SSL Termination]
    end
    
    HPAAuth --> AuthPods
    HPAPayment --> PaymentPods
    HPATransaction --> TransactionPods
    
    AuthPods --> ServiceMesh
    PaymentPods --> ServiceMesh
    TransactionPods --> ServiceMesh
    
    ServiceMesh --> Ingress
    
    AuthPods -.-> RedisCluster
    PaymentPods -.-> PostgresHA
    PaymentPods -.-> RedisCluster
    PaymentPods -.-> KafkaPartitions
    TransactionPods -.-> PostgresHA
    TransactionPods -.-> KafkaPartitions
```

### Monitoring & Observability Stack
```mermaid
graph TB
    subgraph "Application Instrumentation"
        AuthMetrics[🔐 Auth Service<br/>OpenTelemetry SDK]
        PaymentMetrics[💰 Payment Service<br/>OpenTelemetry SDK]
        TransactionMetrics[📊 Transaction Service<br/>OpenTelemetry SDK]
    end
    
    subgraph "OpenTelemetry Collector"
        OtelReceiver[📡 OTLP Receiver<br/>gRPC: 4317, HTTP: 4318]
        OtelProcessor[⚙️ Batch Processor<br/>- Batch Size: 100<br/>- Timeout: 10s]
        OtelExporter[📤 Exporters<br/>- Prometheus<br/>- Loki<br/>- Tempo]
    end
    
    subgraph "Observability Backends"
        subgraph "Metrics"
            Prometheus[📊 Prometheus<br/>- Metrics Storage<br/>- Alerting Rules<br/>- Remote Write: Enabled]
            Grafana[📈 Grafana Dashboards<br/>- System Metrics<br/>- Business KPIs<br/>- Alerts Visualization]
        end
        
        subgraph "Logging"
            Loki[📝 Loki<br/>- Log Aggregation<br/>- Label-based Indexing<br/>- Log Retention: 168h]
            LogDashboard[📋 Log Dashboard<br/>- Error Tracking<br/>- Performance Logs<br/>- Audit Trails]
        end
        
        subgraph "Tracing"
            Tempo[🔍 Tempo<br/>- Distributed Tracing<br/>- Trace Storage<br/>- Service Dependencies]
            TraceDashboard[🗺️ Trace Dashboard<br/>- Request Flow<br/>- Latency Analysis<br/>- Error Attribution]
        end
    end
    
    subgraph "Alert Management"
        AlertManager[🚨 AlertManager<br/>- Alert Routing<br/>- Notification Channels<br/>- Alert Grouping]
        Notifications[📧 Notifications<br/>- Slack<br/>- Email<br/>- PagerDuty]
    end
    
    AuthMetrics --> OtelReceiver
    PaymentMetrics --> OtelReceiver
    TransactionMetrics --> OtelReceiver
    
    OtelReceiver --> OtelProcessor
    OtelProcessor --> OtelExporter
    
    OtelExporter --> Prometheus
    OtelExporter --> Loki
    OtelExporter --> Tempo
    
    Prometheus --> Grafana
    Prometheus --> AlertManager
    Loki --> LogDashboard
    Tempo --> TraceDashboard
    
    AlertManager --> Notifications
    Grafana --> Notifications
```