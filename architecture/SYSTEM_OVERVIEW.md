# Talent Mesh System Architecture Overview

## Document Purpose

This document provides a high-level view of the Talent Mesh system architecture using C4 model diagrams and architectural principles.

---

## Architectural Vision

Talent Mesh is designed as a **cloud-native microservices platform** optimized for:

1. **Vibe Coding Compatibility** - Each service small enough for AI-assisted development
2. **Platform-Managed Infrastructure** - Hosted on OpenOva Platform
3. **Cost Efficiency** - Zero development cost via LLM Gateway
4. **Polyglot Flexibility** - TypeScript for platform, Rust for performance-critical services

---

## C4 Model Diagrams

### Level 1: System Context

Shows Talent Mesh and its interactions with external systems and users.

```mermaid
graph TB
    subgraph "Users"
        C[Candidate<br/>Takes assessments via browser]
        R[Recruiter<br/>Reviews results]
        A[Admin<br/>Manages system]
        H[Human Interviewer<br/>Joins hybrid sessions]
    end

    subgraph "Talent Mesh Platform"
        TM[Talent Mesh<br/>Hosted on OpenOva]
    end

    subgraph "External Systems"
        LI[LinkedIn<br/>OAuth Only]
        STUN[Google STUN<br/>NAT Discovery]
        EMAIL[Email Service<br/>SendGrid/SES]
        COL[Cost of Living API<br/>Geographic Data]
    end

    C -->|WebRTC P2P| TM
    H -->|WebRTC P2P| TM
    R -->|Reviews candidates| TM
    A -->|Configures platform| TM

    TM -->|OAuth authentication| LI
    TM -->|NAT traversal| STUN
    TM -->|Sends notifications| EMAIL
    TM -->|Salary adjustments| COL
```

**Context Description:**

| Actor/System | Description | Interaction |
|--------------|-------------|-------------|
| Candidate | Job seeker taking assessments | Assessment Portal UI (WebRTC) |
| Recruiter | Hiring professional reviewing results | Web app, APIs |
| Admin | Platform administrator | Admin dashboard |
| Human Interviewer | Joins hybrid AI+Human sessions | WebRTC P2P mesh |
| LinkedIn | Professional network | OAuth 2.0 (authentication only) |
| Google STUN | NAT discovery service | STUN protocol (free) |
| LLM Gateway | Large Language Model | OpenOva platform service |
| Email Service | Transactional email | SMTP/API |
| MinIO | Object storage | S3-compatible API (OpenOva service) |
| Cost of Living API | Geographic salary data | REST API (Numbeo/Expatistan) |

---

### Level 2: Container Diagram

Shows the high-level technology choices and containers within Talent Mesh.

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Assessment Portal<br/>Next.js + WebRTC]
        MOBILE[Mobile PWA<br/>Future]
    end

    subgraph "OpenOva Platform"
        subgraph "Edge Layer"
            ISTIO_GW[Istio Ingress Gateway<br/>Envoy-based]
            ISTIO[Istio Service Mesh<br/>mTLS]
            SIGNAL[Signaling Service<br/>Rust - WebRTC]
            TURN[STUNner TURN<br/>K8s-native]
        end

        subgraph "Platform Services"
            AUTH[Auth Service<br/>Node.js - LinkedIn OAuth]
            USER[User Service<br/>Node.js]
            ASSESS[Assessment Service<br/>Node.js]
            SCHED[Scheduling Service<br/>Node.js]
            NOTIFY[Notification Service<br/>Node.js]
            MATCH[Matching Service<br/>Python - Job Matching]
        end

        subgraph "AI Agent Pods"
            AIPOD[AI Agent Pod]
            STT[STT Service<br/>Rust - whisper.cpp]
            TTS[TTS Service<br/>Rust - Piper]
            LLM[LLM Gateway<br/>OpenOva Service]
        end

        subgraph "Intelligence Services"
            SCORE[Scoring Service<br/>Python]
            REPORT[Report Service<br/>Node.js]
        end

        subgraph "Data Stores"
            PG[(PostgreSQL<br/>via CNPG)]
            MONGO[(MongoDB)]
            DRAGONFLY[(Dragonfly<br/>Cache & Sessions)]
            MINIO[(MinIO<br/>S3-compatible)]
            REDPANDA[Redpanda<br/>Event Streaming]
        end
    end

    WEB --> ISTIO_GW --> ISTIO
    WEB <-->|WebRTC P2P| AIPOD
    WEB --> SIGNAL
    WEB -.->|Fallback ~20%| TURN
    ISTIO --> AUTH & USER & ASSESS & SCHED & NOTIFY & MATCH & REPORT

    AIPOD --> STT & TTS & LLM
    REDPANDA --> SCORE & REPORT & MATCH

    AUTH --> PG & DRAGONFLY
    USER --> PG & MONGO
    ASSESS --> MONGO
    SCHED --> PG & DRAGONFLY
    SCORE --> MONGO
    MATCH --> MONGO & PG
    REPORT --> PG & MINIO
    AIPOD --> MINIO
```

**Container Descriptions:**

| Container | Technology | Responsibility | Scale Strategy |
|-----------|------------|----------------|----------------|
| Assessment Portal | Next.js 14 + WebRTC | User interface, media capture | CDN, static assets |
| Istio Ingress Gateway | Envoy-based | HTTP/WebSocket routing, TLS, rate limiting | Platform-managed |
| Istio Service Mesh | Ambient Mode (ztunnel) | mTLS, traffic management | Sidecar-less |
| Signaling Service | Rust | WebRTC offer/answer exchange | Platform-managed |
| STUNner TURN | Go (K8s-native) | NAT traversal fallback (~20%) | Platform-managed |
| Auth Service | Node.js | LinkedIn OAuth, authorization | HPA |
| User Service | Node.js | User profiles, CV analysis | HPA |
| Assessment Service | Node.js | Assessment configuration | HPA |
| Scheduling Service | Node.js | Slot management, booking | HPA |
| Notification Service | Node.js | Email, webhooks | HPA |
| Matching Service | Python | Candidate-job matching, CoL data | HPA |
| AI Agent Pod | Container | WebRTC receiver, AI pipeline | Platform-managed |
| STT Service | Rust (whisper.cpp) | Speech-to-text | In AI Agent Pod |
| TTS Service | Rust (Piper) | Text-to-speech | In AI Agent Pod |
| LLM Gateway | Python | LLM session pool | OpenOva service |
| Scoring Service | Python | Assessment scoring | HPA |
| Report Service | Node.js | Report generation | HPA |

---

## Platform Architecture

### Overview

Talent Mesh runs on the **OpenOva Platform** with DNS-based failover:
- **Candidates** connect via WebRTC P2P directly to **AI Agent Pods**
- **Human Interviewers** can join sessions for hybrid AI+Human interviews (up to 5 participants)
- **All AI processing** (STT, TTS, LLM) runs in platform-managed pods
- **Infrastructure managed by OpenOva** - See [OpenOva Handbook](https://github.com/openova-io/handbook)

```mermaid
graph TB
    subgraph "Candidate & Interviewer Browsers"
        C1[Candidate 1<br/>Assessment Portal]
        C2[Candidate 2<br/>Assessment Portal]
        H1[Human Interviewer<br/>Join mid-session]
    end

    subgraph "OpenOva Platform"
        subgraph "Ingress"
            ISTIO_GW[Istio Ingress Gateway]
        end

        subgraph "Application Services"
            PLATFORM[Platform Services]
            AIPOD1[AI Agent Pod 1]
            AIPOD2[AI Agent Pod 2]
        end

        subgraph "Data Layer"
            PG[(PostgreSQL)]
            MONGO[(MongoDB)]
            MINIO[(MinIO)]
        end

        subgraph "DNS Failover"
            DNS[CoreDNS + Health Orchestrator]
        end
    end

    subgraph "External Services"
        STUN[Google STUN<br/>Free NAT Discovery]
    end

    C1 <-->|WebRTC P2P| AIPOD1
    C2 <-->|WebRTC P2P| AIPOD2
    H1 <-->|WebRTC P2P Mesh| AIPOD1
    C1 & C2 & H1 --> ISTIO_GW

    DNS --> ISTIO_GW
```

### WebRTC P2P Architecture (Up to 5 Participants)

```mermaid
graph TB
    subgraph "Full Mesh P2P - 5 Participants Max"
        C[Candidate]
        AI[AI Agent Pod]
        H1[Human Interviewer 1]
        H2[Human Interviewer 2]
        H3[Human Interviewer 3]

        C <-->|P2P| AI
        C <-->|P2P| H1
        C <-->|P2P| H2
        C <-->|P2P| H3
        AI <-->|P2P| H1
        AI <-->|P2P| H2
        AI <-->|P2P| H3
        H1 <-->|P2P| H2
        H1 <-->|P2P| H3
        H2 <-->|P2P| H3
    end

    subgraph "Signaling"
        SIG[Signaling Service<br/>Rust - WebSocket]
    end

    subgraph "NAT Traversal"
        STUN[Google/Twilio STUN<br/>Free]
        TURN[STUNner TURN<br/>Platform-managed ~20%]
    end

    C & AI & H1 & H2 & H3 --> SIG
    C & AI & H1 & H2 & H3 -.->|~80% direct| STUN
    C & AI & H1 & H2 & H3 -.->|~20% fallback| TURN
```

**Hybrid Interview Modes:**

| Mode | Participants | Use Case |
|------|-------------|----------|
| AI-Only | Candidate + AI Agent | Standard screening |
| Hybrid | Candidate + AI + 1-3 Humans | Final round interviews |
| Human-Only | Candidate + 1-4 Humans | Traditional interviews |

**Security Measures:**

1. **Cilium CNI + eBPF** - Network isolation (Platform-managed)
2. **Istio mTLS** - Service-to-service encryption (Ambient Mode)
3. **WebRTC DTLS-SRTP** - Encrypted media streams
4. **MinIO IAM** - S3-compatible access policies
5. **Audit Trail** - All interactions logged via Redpanda events

---

### Level 3: Component Diagram - AI Agent Pod

Detailed view of the AI Agent Pod running on the platform.

```mermaid
graph TB
    subgraph "Assessment Service"
        AS[Assessment Controller<br/>Session lifecycle]
        QM[Queue Manager<br/>Assessment queue]
        HM[Health Monitor<br/>Pod health checks]
        SC[Slot Calculator<br/>Cluster capacity]
    end

    subgraph "Signaling (Rust)"
        SIG[Signaling Server<br/>WebSocket - tokio]
        ICE[ICE Candidate Relay<br/>async-std]
    end

    subgraph "AI Agent Pod"
        subgraph "WebRTC Container"
            RTC[WebRTC Server<br/>Receive P2P stream]
            REC[Recording Manager<br/>Buffer + upload]
        end

        subgraph "AI Pipeline Container"
            AC[Audio Extractor<br/>From WebRTC stream]
            WC[STT Service<br/>Rust - whisper.cpp]
            CE[LLM Gateway<br/>Session pool]
            PC[TTS Service<br/>Rust - Piper]
            AB[Audio Injector<br/>To WebRTC stream]
        end
    end

    subgraph "Storage"
        MINIO[(MinIO - S3 Compatible)]
    end

    AS --> QM
    AS --> HM
    QM --> SIG
    HM --> RTC

    SIG --> ICE --> RTC
    RTC --> AC --> WC --> CE --> PC --> AB --> RTC
    RTC --> REC
    REC -->|Upload after session| MINIO
```

**Component Descriptions:**

| Component | Location | Responsibility | Key Interfaces |
|-----------|----------|----------------|----------------|
| Assessment Controller | Platform | Session lifecycle, pod assignment | REST API, Redpanda |
| Queue Manager | Platform | Manage assessment queue, assignment | Dragonfly Streams |
| Health Monitor | Platform | Pod health via probes | Liveness/readiness |
| Slot Calculator | Platform | Calculate available slots from capacity | REST API |
| Signaling Server | Platform | WebRTC offer/answer exchange (Rust) | WebSocket |
| ICE Candidate Relay | Platform | Exchange ICE candidates for NAT traversal | WebSocket |
| WebRTC Server | AI Pod | Receive P2P media from candidate | WebRTC |
| Recording Manager | AI Pod | Buffer, upload to MinIO | S3-compatible API |
| Audio Extractor | AI Pod | Extract audio from WebRTC stream | Internal |
| STT Service | AI Pod | Rust wrapper around whisper.cpp | gRPC |
| LLM Gateway | AI Pod | Manage LLM sessions | Internal |
| TTS Service | AI Pod | Rust wrapper around Piper | gRPC |
| Audio Injector | AI Pod | Send TTS audio back to candidate | WebRTC |

---

### Level 3: Component Diagram - Platform Services

Detailed view of core platform services.

```mermaid
graph TB
    subgraph "Auth Service - LinkedIn OAuth Only"
        AC[Auth Controller<br/>REST endpoints]
        OA[LinkedIn OAuth Handler<br/>OAuth 2.0 flow]
        JM[JWT Manager<br/>Token issue/verify]
        SM[Session Manager<br/>Session store]
        RM[RBAC Manager<br/>Role permissions]
    end

    subgraph "User Service"
        UC[User Controller<br/>REST endpoints]
        PM[Profile Manager<br/>Profile CRUD]
        LI[LinkedIn Importer<br/>Profile import]
        CV[CV Analyzer<br/>CV parsing]
        VE[Verification Engine<br/>Skill verification]
    end

    subgraph "Assessment Service"
        ASC[Assessment Controller<br/>REST endpoints]
        TM[Template Manager<br/>Assessment templates]
        QM[Question Manager<br/>Question bank]
        CM[Config Manager<br/>Assessment config]
    end

    subgraph "Scheduling Service"
        SC[Scheduling Controller<br/>REST endpoints]
        SL[Slot Manager<br/>Slot availability]
        BK[Booking Manager<br/>Booking CRUD]
        REM[Reminder Manager<br/>Reminder scheduling]
    end

    subgraph "Matching Service"
        MC[Matching Controller<br/>REST endpoints]
        JM2[Job Matcher<br/>Candidate-Job scoring]
        COL[CoL Calculator<br/>Cost of living adjustment]
        RK[Ranking Engine<br/>Candidate ranking]
    end

    AC --> OA & JM & SM & RM
    UC --> PM & LI & CV & VE
    ASC --> TM & QM & CM
    SC --> SL & BK & REM
    MC --> JM2 & COL & RK
```

---

## Matching Service Architecture

The Matching Service provides intelligent candidate-job matching with cost-of-living adjustments.

```mermaid
graph TB
    subgraph "Inputs"
        CAND[Candidate Profile<br/>Skills, Experience, Location]
        JOB[Job Requirements<br/>Skills, Level, Salary Range]
        COL_DATA[Cost of Living Data<br/>Geographic indices]
    end

    subgraph "Matching Engine"
        SKILL[Skill Matcher<br/>Technical alignment]
        EXP[Experience Matcher<br/>Level alignment]
        SAL[Salary Calculator<br/>CoL-adjusted]
        SPIDER[Spider Map Scorer<br/>Multi-dimensional]
    end

    subgraph "Outputs"
        RANK[Ranked Candidates]
        SALARY[Adjusted Salary Range]
        FIT[Fit Score<br/>0-100]
    end

    CAND --> SKILL & EXP & SPIDER
    JOB --> SKILL & EXP & SAL
    COL_DATA --> SAL

    SKILL --> FIT
    EXP --> FIT
    SAL --> SALARY
    SPIDER --> RANK

    FIT --> RANK
```

**Matching Criteria:**

| Dimension | Weight | Data Source |
|-----------|--------|-------------|
| Technical Skills | 30% | Assessment spider map |
| Problem Solving | 20% | Assessment spider map |
| Communication | 15% | Assessment spider map |
| Experience Level | 15% | LinkedIn profile |
| Salary Fit | 10% | CoL-adjusted calculation |
| Availability | 10% | Scheduling data |

---

## Technology Stack Overview

### Language Distribution

```mermaid
pie title Codebase by Language
    "TypeScript/JavaScript" : 40
    "Rust" : 25
    "Python" : 25
    "SQL" : 8
    "Other" : 2
```

### Technology Rationale

| Layer | Choice | Rationale |
|-------|--------|-----------|
| Frontend | Next.js 14 + WebRTC | SSR, React ecosystem, native media APIs |
| API Gateway | Istio/Envoy | Service mesh, mTLS, observability |
| Platform Services | Node.js | TypeScript, async I/O, npm ecosystem |
| Signaling | **Rust (tokio)** | Performance-critical, low latency WebSocket |
| NAT Traversal | **STUNner** | K8s-native TURN, platform-managed |
| STT Service | **Rust (whisper.cpp)** | x86_64 optimized, low latency |
| TTS Service | **Rust (Piper)** | x86_64 optimized, low latency |
| LLM Gateway | Python | Session pooling (OpenOva service) |
| Orchestration | **Kubernetes** | Platform-managed |
| Networking | **Cilium CNI** | eBPF-based networking |
| Primary DB | PostgreSQL | ACID, CNPG operator |
| Document DB | MongoDB | Flexible schemas |
| Cache | Dragonfly | Redis-compatible, multi-threaded |
| Events | Redpanda | Kafka-compatible, event replay |
| Objects | **MinIO** | S3-compatible, platform-managed |

### Rust vs Python Decision Matrix

| Criterion | Signaling | STT | TTS | LLM Gateway |
|-----------|-----------|-----|-----|-------------|
| Latency Critical | Yes | Yes | Yes | No |
| CPU Intensive | No | Yes | Yes | No |
| x86_64 Optimization | Yes | Yes | Yes | No |
| Complex AI Libraries | No | No | No | Yes |
| **Choice** | **Rust** | **Rust** | **Rust** | **Python** |

---

## Architectural Principles

### 1. Vibe Coding Compatibility

Each microservice is designed to fit within AI assistant context windows:

| Constraint | Limit | Rationale |
|------------|-------|-----------|
| Lines of code | < 50k | Claude ~200k token context |
| Files per service | < 500 | Cognitive load |
| Dependencies | < 50 | Stability, security |
| Cyclomatic complexity | < 10/function | Understandability |

### 2. Domain-Driven Design

Services align with bounded contexts:

```mermaid
graph TB
    subgraph "Identity Context"
        AUTH[Auth<br/>LinkedIn OAuth]
        USER[User]
    end

    subgraph "Assessment Context"
        ASSESS[Assessment]
        SCORE[Scoring]
        AIPOD[AI Agent Pods]
    end

    subgraph "Infrastructure Context"
        SCHED[Scheduling]
        SIGNAL[Signaling<br/>Rust]
        TURN[STUNner TURN]
    end

    subgraph "Analytics Context"
        REPORT[Reports]
        MATCH[Matching<br/>+ CoL Data]
    end
```

### 3. Event-Driven Architecture

Services communicate via events for loose coupling:

```mermaid
sequenceDiagram
    participant A as Assessment
    participant R as Redpanda
    participant S as Scoring
    participant M as Matching
    participant RP as Report

    A->>R: assessment.completed
    R->>S: assessment.completed
    S->>R: assessment.scored
    R->>M: assessment.scored
    M->>R: candidate.matched
    R->>RP: candidate.matched
    RP->>R: report.generated
```

### 4. CAP Theorem Application

| System | CAP Choice | Justification |
|--------|------------|---------------|
| PostgreSQL | CP | Consistency for auth, booking |
| MongoDB | AP | Availability for profiles, results |
| Dragonfly | AP | Speed for cache, sessions (Redis-compatible) |
| Redpanda | AP | Event streaming (Kafka-compatible) |

### 5. Twelve-Factor App Compliance

| Factor | Implementation |
|--------|----------------|
| Codebase | Git monorepo with service folders |
| Dependencies | package.json / requirements.txt |
| Config | Environment variables |
| Backing services | Attached resources via URLs |
| Build, release, run | CI/CD pipeline stages |
| Processes | Stateless, share-nothing |
| Port binding | Self-contained HTTP servers |
| Concurrency | Process model, horizontal scaling |
| Disposability | Fast startup, graceful shutdown |
| Dev/prod parity | Docker Compose ~ Kubernetes |
| Logs | Stdout, collected by platform |
| Admin processes | Separate CLI tools |

---

## Cross-Cutting Concerns

### Security Architecture

```mermaid
graph TB
    subgraph "External"
        U[User/Candidate]
        H[Human Interviewer]
        E[External API]
    end

    subgraph "Edge - OpenOva Platform"
        ISTIO_GW[Istio Ingress Gateway<br/>Ambient Mode]
        ISTIO[Istio Service Mesh<br/>ztunnel-based]
    end

    subgraph "Platform Services"
        S1[Platform Services]
        S2[AI Agent Pods]
        SIG[Signaling Service<br/>Rust]
    end

    U --> ISTIO_GW --> ISTIO
    H --> ISTIO_GW
    ISTIO -->|JWT - LinkedIn OAuth| S1
    S1 -->|mTLS| S2
    E -->|API Key| ISTIO_GW
    U -->|WebRTC DTLS-SRTP| S2
    H -->|WebRTC DTLS-SRTP| S2
    SIG -->|mTLS| S1
```

### Authentication Flow - LinkedIn Only

```mermaid
sequenceDiagram
    participant U as User
    participant WEB as Web App
    participant AUTH as Auth Service
    participant LI as LinkedIn

    U->>WEB: Click "Sign in with LinkedIn"
    WEB->>LI: OAuth 2.0 Authorization Request
    LI->>U: LinkedIn Login Page
    U->>LI: Enter Credentials
    LI->>WEB: Authorization Code
    WEB->>AUTH: Exchange Code
    AUTH->>LI: Token Request
    LI->>AUTH: Access Token
    AUTH->>LI: Get Profile
    LI->>AUTH: Profile Data
    AUTH->>WEB: JWT Token
    WEB->>U: Logged In
```

### Observability Stack

```mermaid
graph LR
    subgraph "Application Services"
        PS[Platform Services]
        AIPOD[AI Agent Pods]
        SIG[Signaling Service]
    end

    subgraph "Collection"
        L[Logs<br/>Fluent Bit]
        M[Metrics<br/>Prometheus]
        T[Traces<br/>OpenTelemetry]
    end

    subgraph "Storage"
        LOKI[Loki]
        PROM[Prometheus]
        JAE[Jaeger]
    end

    subgraph "Visualization"
        GR[Grafana<br/>Unified Dashboard]
    end

    PS & AIPOD & SIG --> L --> LOKI --> GR
    PS & AIPOD & SIG --> M --> PROM --> GR
    PS & AIPOD & SIG --> T --> JAE --> GR
```

---

## Deployment Architecture

### Development Environment

```mermaid
graph TB
    subgraph "Local Development"
        DC[Docker Compose]
        DC --> PG[PostgreSQL]
        DC --> MONGO[MongoDB]
        DC --> REDIS[Redis + pub/sub]
        DC --> AI[AI Agent Container<br/>STT + TTS + LLM]
    end

    subgraph "LLM Gateway"
        CLI[LLM Gateway<br/>Zero Cost Development]
        POOL[Session Pool<br/>Avoid Cold Start]
    end

    DC --> CLI
    CLI --> POOL
```

### Production Environment

```mermaid
graph TB
    subgraph "OpenOva Platform"
        subgraph "Control Plane"
            K8S[Kubernetes Control Plane]
        end

        subgraph "Worker Nodes"
            W1[Worker 1]
            W2[Worker 2]
            W3[Worker 3]
        end

        subgraph "Platform Services"
            ISTIO_GW[Istio Gateway]
            PLATFORM[TalentMesh Services]
            AIPOD[AI Agent Pods]
        end

        subgraph "Data Services"
            PG[(PostgreSQL)]
            MONGO[(MongoDB)]
            MINIO[(MinIO)]
        end

        subgraph "DNS Failover"
            DNS[CoreDNS + Health Orchestrator]
        end
    end

    subgraph "External Services"
        STUN[Google/Twilio STUN]
    end

    subgraph "Users"
        C1[Candidate 1]
        C2[Candidate 2]
        H1[Human Interviewer]
    end

    DNS --> ISTIO_GW
    C1 <-->|WebRTC P2P| AIPOD
    C2 <-->|WebRTC P2P| AIPOD
    H1 <-->|WebRTC P2P| AIPOD
    C1 & C2 & H1 --> DNS
    C1 & C2 & H1 -.->|NAT Discovery| STUN
```

### AI Agent Pod Architecture

```mermaid
graph TB
    subgraph "AI Agent Pod - Deployment"
        subgraph "Pod Resources"
            REQ1[~3 GB RAM]
            REQ2[1-2 vCPU]
            REQ3[GPU: None required]
        end

        subgraph "Containers"
            RTC[WebRTC Container<br/>Node.js]
            STT[STT Container<br/>Rust + whisper.cpp]
            TTS[TTS Container<br/>Rust + Piper]
            LLM[LLM Container<br/>Session Pool]
        end
    end

    subgraph "Platform Management"
        HPA[Horizontal Pod Autoscaler<br/>Based on queue depth]
        SVC[K8s Service<br/>ClusterIP]
        PDB[Pod Disruption Budget<br/>minAvailable: 2]
    end

    REQ1 & REQ2 & REQ3 --> RTC & STT & TTS & LLM
    HPA --> RTC
    SVC --> RTC
    PDB --> RTC
```

---

## LLM Gateway Architecture

Zero development cost through session pooling via OpenOva LLM Gateway:

```mermaid
graph TB
    subgraph "LLM Gateway"
        POOL[Session Pool Manager<br/>2+ warm sessions]
        S1[Session 1<br/>Warm]
        S2[Session 2<br/>Warm]
        SN[Session N<br/>Warm]
    end

    subgraph "Request Flow"
        REQ[Incoming Request]
        ACQ[Acquire Session]
        QUERY[Send Prompt]
        REL[Release Session]
        RESP[Response]
    end

    REQ --> ACQ --> POOL
    POOL --> S1 & S2 & SN
    S1 --> QUERY --> RESP --> REL --> POOL

    subgraph "Cold Start Avoidance"
        NOTE[13s cold start<br/>avoided via pooling]
    end
```

**Configuration:**

```python
# Environment-based backend switching
LLM_BACKEND=cli   # Development: Zero cost
LLM_BACKEND=api   # Production: Claude API

# Session pool configuration
POOL_SIZE=2                    # Warm sessions
SESSION_TIMEOUT=300            # 5 min idle timeout
MAX_REQUESTS_PER_SESSION=100   # Refresh threshold
```

For full LLM Gateway specification, see [OpenOva LLM Gateway](https://github.com/openova-io/handbook/blob/main/services/LLM_GATEWAY.md).

---

## Quality Attributes

| Attribute | Target | Measurement |
|-----------|--------|-------------|
| Availability | 99.5% | Uptime monitoring (Grafana) |
| Latency (API) | < 100ms P95 | Prometheus metrics |
| Latency (AI Response) | < 2.5s P95 | Custom OpenTelemetry metrics |
| Throughput | 6-8 concurrent assessments (MVP) | Pod metrics |
| Scalability | Platform-managed | Request via platform team |
| Maintainability | < 50k LOC/service | Static analysis |
| Security | OWASP Top 10 compliant | Security scanning |

---

## Platform Documentation

For infrastructure, deployment, and platform services documentation, see:

| Resource | Description |
|----------|-------------|
| [OpenOva Handbook](https://github.com/openova-io/handbook) | Platform documentation |
| [LLM Gateway](https://github.com/openova-io/handbook/blob/main/services/LLM_GATEWAY.md) | LLM service specification |
| [DNS Failover](https://github.com/openova-io/handbook/blob/main/technical/DNS_FAILOVER_SPEC.md) | High availability |
| [Deployment Architecture](https://github.com/openova-io/handbook/blob/main/architecture/DEPLOYMENT_ARCHITECTURE.md) | K8s deployment |

---

*Document Version: 5.0*
*Last Updated: 2026-01-12*
*Owner: Architecture Team*
