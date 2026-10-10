# Custodian AP - End-to-End Flow & Technology Stack

## System Flow Diagram

```mermaid
graph TD
    A["📄 Invoice Ingestion<br/>(PDF/OCR/API)"] -->|Parse & Validate| B["🔍 Data Governance<br/>(PII Redaction<br/>Presidio/Regex)"]
    
    B -->|Redacted Data| C["🤖 Risk & Fraud Scoring<br/>(LiteLLM<br/>OpenAI/Groq/HuggingFace)"]
    
    C -->|Risk Score| D["📋 Policy Engine<br/>(Thresholds<br/>Duplicate Check<br/>Vendor Denylist)"]
    
    D -->|Auto-Approved?| E{"Risk Decision"}
    
    E -->|Low Risk ✓| F["💳 Auto-Pay<br/>(Ledger Update<br/>Payment Rail)"]
    
    E -->|Medium Risk| G["👤 Human Review Queue<br/>(UI Dashboard<br/>Bulk Approve/Reject)"]
    
    E -->|High Risk ✗| H["🚫 Auto-Reject<br/>(Webhook/Notification)"]
    
    G -->|Reviewer Decision| I{Approved?}
    
    I -->|Yes| F
    I -->|No| H
    
    F --> J["📊 Persistence Layer<br/>(SQLite DB<br/>Audit Log)"]
    H --> J
    
    J --> K["🔒 Six Governance Layers<br/>Identity · Data · Model<br/>Policy · Agent Runtime<br/>Operations"]
    
    K -->|Observable & Auditable| L["📈 Observability Stack<br/>(MLflow · Langfuse<br/>Prometheus · Grafana)"]
    
    L -->|Metrics & Traces| M["💻 Console UI<br/>Dashboard · Queue<br/>Ledger · Governance<br/>Audit"]

    style A fill:#e1f5ff
    style C fill:#fff3e0
    style D fill:#f3e5f5
    style F fill:#c8e6c9
    style H fill:#ffcdd2
    style K fill:#fce4ec
    style L fill:#e0f2f1
    style M fill:#fff9c4
```

## Technology Stack by Layer

```mermaid
graph TB
    subgraph Frontend["🖥️ Frontend Layer"]
        UI1["React/Vite Dashboard<br/>#/dashboard · #/queue<br/>#/ledger · #/governance · #/audit"]
        UI2["Zero-Build Console<br/>(Single-file HTML)"]
    end
    
    subgraph API["🔌 API & Orchestration"]
        FAST["FastAPI<br/>(Python REST API)"]
        ORCH["Orchestrator<br/>(Agent Coordination)"]
    end
    
    subgraph Agents["🤖 Agent Pipeline"]
        INGEST["Ingest Agent<br/>(Validation)"]
        RISK["Risk Agent<br/>(Scoring)"]
        APPROVAL["Approval Agent<br/>(Routing)"]
        PAYMENT["Payment Agent<br/>(Auto-Pay)"]
    end
    
    subgraph LLM["🧠 LLM & ML"]
        LITELLM["LiteLLM Gateway<br/>(Model Routing)"]
        LLM1["OpenAI<br/>(GPT-4o-mini)"]
        LLM2["Groq<br/>(Llama-3.1)"]
        LLM3["HuggingFace"]
        LLM4["Anthropic"]
        MLFLOW["MLflow<br/>(Model Tracking)"]
    end
    
    subgraph Governance["🔐 Governance Layers"]
        IDENTITY["Identity<br/>(Keycloak · SPIRE)"]
        DATA["Data Privacy<br/>(Presidio/Regex)"]
        MODEL["Model Management<br/>(LiteLLM · MLflow)"]
        POLICY["Policy Engine<br/>(Thresholds)"]
        RUNTIME["Agent Runtime<br/>(Orchestrator)"]
        OPS["Operations<br/>(Observability)"]
    end
    
    subgraph Persistence["💾 Data & Persistence"]
        SQLITE["SQLite<br/>(Invoice Store<br/>Audit Log)"]
        LEDGER["Mock Ledger<br/>(Balance & Tx)"]
        SECRETS["Secrets<br/>(Infisical)"]
    end
    
    subgraph Observability["📊 Observability"]
        PROMETHEUS["Prometheus<br/>(Metrics)"]
        GRAFANA["Grafana<br/>(Dashboards)"]
        LANGFUSE["Langfuse<br/>(Traces)"]
    end
    
    subgraph Infrastructure["📦 Infrastructure"]
        DOCKER["Docker/Compose<br/>(Container Orchestration)"]
    end
    
    Frontend --> API
    API --> Agents
    API --> Persistence
    Agents --> LLM
    API --> Governance
    LLM --> MLFLOW
    LLM --> LITELLM
    LITELLM --> LLM1
    LITELLM --> LLM2
    LITELLM --> LLM3
    LITELLM --> LLM4
    Governance --> Observability
    Persistence --> Observability
    API --> Observability
    Everything["All Services"] -.-> DOCKER
    
    style Frontend fill:#bbdefb
    style API fill:#c8e6c9
    style Agents fill:#ffe0b2
    style LLM fill:#f8bbd0
    style Governance fill:#d1c4e9
    style Persistence fill:#b2dfdb
    style Observability fill:#ffecb3
    style Infrastructure fill:#d7ccc8
```

## Key Technologies Summary

| Category | Tools | Purpose |
|----------|-------|---------|
| **Frontend** | React/Vite, HTML/CSS/JavaScript | 5-section console UI with live data |
| **Backend** | FastAPI (Python) | REST API, request routing, orchestration |
| **AI/ML** | LiteLLM, MLflow | LLM gateway, multi-provider routing, model tracking |
| **Agents** | Custom Python modules | Invoice intake → risk scoring → approval → payment |
| **Governance** | Presidio, Keycloak, SPIRE | PII redaction, identity/auth, policy enforcement |
| **Database** | SQLite | Invoice persistence, audit trail (append-only) |
| **Observability** | Prometheus, Grafana, Langfuse | Metrics, dashboards, distributed tracing |
| **Secrets** | Infisical | Encrypted API key management |
| **Runtime** | Docker, Docker Compose | Container orchestration |
| **Notifications** | Webhooks, JSONL logs | Event notifications for rejections/alerts |

## Invoice Processing Journey

```
Invoice Input (PDF/OCR/API)
    ↓
[Data: PII Redaction] → Remove sensitive data
    ↓
[Model: LLM Scoring] → Risk/Fraud assessment (0-100)
    ↓
[Policy: Rule Engine] → Apply thresholds & hard limits
    ↓
Decision Tree:
├─ Risk < Auto-Approve Threshold → [Agent: Auto-Pay] → Ledger Update
├─ Risk Between Thresholds → [Human Review Queue] → Reviewer Decision
└─ Risk > Auto-Reject Threshold → [Agent: Auto-Reject] → Notification
    ↓
[Operations: Audit Log] → Append immutable decision trail
    ↓
[Observability: MLflow/Langfuse/Prometheus] → Track metrics & traces
    ↓
Dashboard Update → Real-time UI refresh (8s polling)
```

---

Interview Description: 

Custodian is a Python-based accounts payable automation platform designed to ingests invoices, extract and validate data, score them for fraud/risk, enforce policy rules, and route decisions for approval or automated payment. The platform combines a FastAPI backend with a Vite/React frontend, SQLite persistence, LiteLLM-based multi-provider LLM access, and MLflow tracking to create an end-to-end AP workflow with governance and auditability.

At a high level, the system follows an invoice lifecycle: OCR or structured invoice submission enters the pipeline, sensitive fields are redacted through privacy controls, risk scoring is performed using LLMs or heuristic fallback, and policy checks determine whether the invoice is auto-approved, sent for human review, or rejected. Approved invoices are posted to a ledger and payment flow, while all major actions are logged in an append-only audit trail. This makes the system suitable for finance operations where traceability, compliance, and explainability are critical.

The repository also emphasizes governance layers across identity, data protection, model access, policy enforcement, agent runtime, and operations. It includes observability through Prometheus, Grafana, and Langfuse, along with support for Docker-based deployment and multi-stack infrastructure. 

---

This project demonstrates strong architectural thinking: AI orchestration, workflow automation, secure enterprise patterns, and production-style observability in a domain-specific business workflow.