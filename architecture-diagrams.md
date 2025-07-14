# CrewWork Architecture Diagrams

## Super High-Level Platform Overview

```mermaid
flowchart LR
    subgraph Users
        DEV[Developers]
        API[API Clients]
    end
    
    subgraph "CrewWork Platform"
        PLATFORM[AI-Powered Development Platform<br/>with Knowledge Graph Intelligence]
    end
    
    subgraph "External Services"
        GH[GitHub]
        LLM[LLM Providers<br/>Ollama/OpenAI/Anthropic]
    end
    
    DEV --> PLATFORM
    API --> PLATFORM
    PLATFORM <--> GH
    PLATFORM <--> LLM
```

## System Architecture Overview

```mermaid
flowchart TB
    subgraph "Client Layer"
        WEB[Web UI<br/>React + TypeScript]
        API_CLIENT[API Clients<br/>REST + WebSocket]
    end
    
    subgraph "API Gateway"
        NGINX[Nginx Reverse Proxy<br/>:80]
    end
    
    subgraph "Application Layer"
        FASTAPI[FastAPI Server<br/>:8000]
        WS[WebSocket Handler<br/>Enhanced Manager]
    end
    
    subgraph "Processing Layer"
        CELERY[Celery Workers<br/>Multi-Queue]
        BEAT[Celery Beat<br/>Scheduler]
        FLOWER[Flower Monitor<br/>:5555]
    end
    
    subgraph "Data Layer"
        PG[(PostgreSQL<br/>Primary Database)]
        REDIS[(Redis<br/>Cache & Queue)]
        NEO4J[(Neo4j<br/>Knowledge Graph)]
    end
    
    subgraph "AI Layer"
        OLLAMA[Ollama<br/>Local LLM<br/>:11434]
        LLM[LLM Manager<br/>Routing & Fallback]
    end
    
    WEB --> NGINX
    API_CLIENT --> NGINX
    NGINX --> FASTAPI
    NGINX --> WS
    FASTAPI --> CELERY
    CELERY --> REDIS
    BEAT --> REDIS
    FLOWER --> REDIS
    FASTAPI --> PG
    FASTAPI --> REDIS
    FASTAPI --> NEO4J
    FASTAPI --> LLM
    LLM --> OLLAMA
```

## Detailed Service Architecture

```mermaid
flowchart TB
    subgraph "Frontend Services"
        REACT[React Application<br/>Optimized Contexts]
        WS_CLIENT[WebSocket Client<br/>Channel Subscriptions]
        API_SERVICE[API Service Layer]
    end
    
    subgraph "API Layer"
        subgraph "Routers"
            AUTH_ROUTER[Auth Router]
            TASK_ROUTER[Tasks Router]
            PROJECT_ROUTER[Projects Router]
            GITHUB_ROUTER[GitHub Router]
            MONITOR_ROUTER[Monitoring Router]
            KG_ROUTER[Knowledge Graph Router]
            INTEL_ROUTER[Intelligence Router]
        end
        
        subgraph "Middleware"
            CSRF[CSRF Protection]
            RATE_LIMIT[Rate Limiter]
            AUTH_MW[Auth Middleware]
        end
    end
    
    subgraph "Core Services"
        TASK_ORCH[Task Orchestrator<br/>Direct Processing]
        CODE_GEN[Code Generation Service]
        GITHUB_SVC[GitHub Service]
        LLM_MGR[LLM Manager<br/>Model Routing]
        KG_SVC[Neo4j Knowledge Graph<br/>Modular Architecture]
        UNIFIED_SEC[Unified Security Service<br/>Multi-Scanner]
        CODE_REVIEW[Code Reviewer<br/>Quality Analysis]
    end
    
    subgraph "Analysis Services"
        ANALYZER_ORCH[Analyzer Orchestrator<br/>5-Phase Analysis]
        PY_ANALYZER[Python Analyzer<br/>Radon Metrics]
        JS_ANALYZER[JavaScript Analyzer<br/>React/TS Support]
        CONFIG_ANALYZER[Config Analyzer<br/>Tech Detection]
        TREE_SITTER[Tree-sitter Analyzer<br/>AST Parsing]
    end
    
    subgraph "Background Processing"
        subgraph "Task Queues"
            DEFAULT_Q[default]
            CODE_GEN_Q[code_generation]
            VALIDATION_Q[validation]
            DEPLOY_Q[deployment]
            KG_Q[knowledge_graph]
            PRIORITY_Q[priority]
        end
    end
    
    REACT --> API_SERVICE
    API_SERVICE --> AUTH_ROUTER
    API_SERVICE --> TASK_ROUTER
    WS_CLIENT --> WS
    
    TASK_ROUTER --> TASK_ORCH
    PROJECT_ROUTER --> KG_SVC
    KG_ROUTER --> KG_SVC
    INTEL_ROUTER --> ANALYZER_ORCH
    
    TASK_ORCH --> CODE_GEN
    TASK_ORCH --> UNIFIED_SEC
    CODE_GEN --> LLM_MGR
    TASK_ORCH --> DEFAULT_Q
    CODE_GEN --> CODE_GEN_Q
    
    ANALYZER_ORCH --> PY_ANALYZER
    ANALYZER_ORCH --> JS_ANALYZER
    ANALYZER_ORCH --> CONFIG_ANALYZER
    ANALYZER_ORCH --> TREE_SITTER
```

## Task Processing Flow (Celery-Based)

```mermaid
sequenceDiagram
    participant User
    participant API as FastAPI
    participant TO as Task Orchestrator
    participant Celery
    participant LLM as LLM Manager
    participant KG as Knowledge Graph
    participant GitHub
    participant WS as WebSocket

    User->>API: Create Task Request
    API->>TO: Process Task
    TO->>KG: Get Project Context & FILE Nodes
    KG-->>TO: Context, Patterns & Code Structure
    TO->>Celery: Queue Task (specific queue)
    
    Note over Celery: Distributed Processing<br/>with Retry Logic
    
    Celery->>LLM: Enhance Task Description
    LLM-->>Celery: Enhanced Description
    Celery->>WS: Status: Processing
    
    Celery->>KG: Get Success Patterns
    KG-->>Celery: Previous Solutions
    
    Celery->>LLM: Generate Code with Context
    LLM-->>Celery: Generated Code
    
    Celery->>Celery: Validate Generated Code
    
    Celery->>GitHub: Create Branch
    GitHub-->>Celery: Branch Created
    
    Celery->>GitHub: Commit Code
    GitHub-->>Celery: Commit Success
    
    Celery->>KG: Update Knowledge<br/>Create FILE Nodes
    Celery->>WS: Status: Completed
    WS-->>User: Real-time Update
```

## Knowledge Graph Architecture

```mermaid
graph TD
    subgraph "Core Nodes"
        PROJECT[PROJECT Node<br/>languages, frameworks, technologies]
        USER[USER Node]
        TASK[TASK Node<br/>enhanced metadata]
        FILE[FILE Node<br/>code metrics, dependencies]
    end
    
    subgraph "Code Structure Nodes"
        MODULE[MODULE Node<br/>hierarchy info]
        CLASS[CLASS Node<br/>methods, complexity]
        FUNCTION[FUNCTION Node<br/>calls, side effects]
        API_ENDPOINT[API_ENDPOINT Node<br/>routes, methods]
    end
    
    subgraph "Analysis Nodes"
        TECH[TECHNOLOGY Node<br/>dynamic detection]
        ERROR[ERROR Node<br/>learning patterns]
        PATTERN[PATTERN Node<br/>success/failure]
        CONFIG[CONFIGURATION Node<br/>settings, scripts]
        VULN[VULNERABILITY Node<br/>security issues]
    end
    
    PROJECT -->|OWNS| PROJECT
    PROJECT -->|HAS_TASK| TASK
    PROJECT -->|HAS_FILE| FILE
    PROJECT -->|USES_TECHNOLOGY| TECH
    PROJECT -->|HAS_MODULE| MODULE
    
    USER -->|OWNS| PROJECT
    USER -->|CREATED| TASK
    
    TASK -->|GENERATES| FILE
    TASK -->|DEPENDS_ON| TASK
    TASK -->|LEARNS_FROM| ERROR
    TASK -->|APPLIES| PATTERN
    
    FILE -->|CONTAINS| MODULE
    FILE -->|HAS_CONFIG| CONFIG
    FILE -->|IMPORTS| FILE
    MODULE -->|CONTAINS| CLASS
    MODULE -->|CONTAINS| FUNCTION
    MODULE -->|CONTAINS| API_ENDPOINT
    
    CLASS -->|HAS_METHOD| FUNCTION
    FUNCTION -->|CALLS| FUNCTION
    
    PATTERN -->|DISCOVERED_IN| PROJECT
    ERROR -->|OCCURRED_IN| TASK
    VULN -->|FOUND_IN| FILE
```

## LLM Integration Architecture

```mermaid
flowchart TB
    subgraph "LLM Manager Infrastructure"
        MGR[LLM Manager<br/>core/services/infrastructure/llm_manager.py]
        ROUTER[Provider Router<br/>YAML Configuration]
        MODEL_SELECT[Model Selection<br/>Task-Type Mapping]
        FALLBACK[Fallback Chain<br/>3-Level Cascade]
    end
    
    subgraph "Providers"
        subgraph "Primary - Local"
            OLLAMA[Ollama<br/>qwen2.5-coder:32b<br/>Auto-pull enabled]
        end
        
        subgraph "Fallback - Cloud"
            OPENAI[OpenAI API<br/>gpt-4o]
            ANTHROPIC[Anthropic API<br/>claude-3-5-sonnet]
        end
    end
    
    subgraph "Task Types"
        CODE_GEN[Code Generation<br/>→ qwen2.5-coder]
        ENHANCE[Task Enhancement<br/>→ qwen2.5-coder]
        ANALYSIS[Code Analysis<br/>→ qwen2.5-coder]
        REVIEW[Code Review<br/>→ qwen2.5-coder]
        SEC_ANALYSIS[Security Analysis<br/>→ qwen2.5-coder]
        SCRIPT_SELECT[Script Selection<br/>→ Container Config]
    end
    
    CODE_GEN --> MGR
    ENHANCE --> MGR
    ANALYSIS --> MGR
    REVIEW --> MGR
    SEC_ANALYSIS --> MGR
    SCRIPT_SELECT --> MGR
    
    MGR --> ROUTER
    ROUTER --> MODEL_SELECT
    MODEL_SELECT --> OLLAMA
    
    OLLAMA -.->|fail| FALLBACK
    FALLBACK --> OPENAI
    OPENAI -.->|fail| ANTHROPIC
```

## Security Architecture (Unified Security Service)

```mermaid
flowchart LR
    subgraph "Unified Security Service"
        USS[UnifiedSecurityService<br/>Orchestrator]
        SCANNER_MGR[Scanner Manager]
        VULN_MGR[Vulnerability Manager]
    end
    
    subgraph "Scanner Types"
        SAST[SAST Scanner<br/>CodeReviewer]
        DEP[Dependency Scanner<br/>safety/npm audit]
        CONTAINER[Container Scanner<br/>trivy]
        SECRET[Secret Scanner<br/>gitleaks]
        INFRA[Infrastructure Scanner<br/>checkov]
        CUSTOM[Custom Scanners<br/>semgrep/bandit]
    end
    
    subgraph "Analysis Flow"
        FILES[Source Files]
        DOCKER[Dockerfiles]
        CONFIGS[Config Files]
        DEPS[Dependencies]
    end
    
    subgraph "Output"
        VULNS[(Vulnerability<br/>Database)]
        KG[(Knowledge Graph<br/>VULNERABILITY Nodes)]
        REPORT[Security Report]
        TASKS[Auto-generated<br/>Fix Tasks]
    end
    
    FILES --> USS
    DOCKER --> USS
    CONFIGS --> USS
    DEPS --> USS
    
    USS --> SCANNER_MGR
    SCANNER_MGR --> SAST
    SCANNER_MGR --> DEP
    SCANNER_MGR --> CONTAINER
    SCANNER_MGR --> SECRET
    SCANNER_MGR --> INFRA
    
    SAST --> VULN_MGR
    DEP --> VULN_MGR
    CONTAINER --> VULN_MGR
    SECRET --> VULN_MGR
    INFRA --> VULN_MGR
    
    VULN_MGR --> VULNS
    VULN_MGR --> KG
    VULN_MGR --> REPORT
    VULN_MGR --> TASKS
```

## Codebase Analysis Architecture

```mermaid
flowchart TB
    subgraph "Analyzer Orchestrator"
        ORCH[AnalyzerOrchestrator<br/>5-Phase Process]
        PHASE1[Phase 1: Module Hierarchy]
        PHASE2[Phase 2: Technology Stack]
        PHASE3[Phase 3: File Nodes]
        PHASE4[Phase 4: Content Analysis]
        PHASE5[Phase 5: Relationships]
    end
    
    subgraph "Language Analyzers"
        PY[Python Analyzer<br/>Radon Metrics<br/>Business Logic]
        JS[JavaScript Analyzer<br/>React Components<br/>TypeScript Support]
        CSS[CSS/HTML Analyzer<br/>Styling & Structure]
        CONFIG[Config Analyzer<br/>Package Detection<br/>Script Analysis]
        TS[Tree-sitter Analyzer<br/>Universal AST]
    end
    
    subgraph "Extracted Metadata"
        IMPORTS[Import Graph]
        EXPORTS[Export Map]
        CLASSES[Class Hierarchy]
        FUNCS[Function Calls]
        DEPS[Dependencies]
        METRICS[Code Metrics<br/>Complexity/Quality]
        TECH_STACK[Technology Stack<br/>Dynamic Detection]
    end
    
    subgraph "Knowledge Graph Updates"
        FILE_NODES[FILE Nodes<br/>with Metadata]
        MODULE_NODES[MODULE Nodes<br/>with Hierarchy]
        TECH_NODES[TECHNOLOGY Nodes<br/>with Versions]
        REL_EDGES[Relationship Edges<br/>IMPORTS/CALLS/CONTAINS]
    end
    
    ORCH --> PHASE1
    PHASE1 --> PHASE2
    PHASE2 --> PHASE3
    PHASE3 --> PHASE4
    PHASE4 --> PHASE5
    
    PHASE4 --> PY
    PHASE4 --> JS
    PHASE4 --> CSS
    PHASE4 --> CONFIG
    PHASE4 --> TS
    
    PY --> IMPORTS
    JS --> EXPORTS
    PY --> CLASSES
    JS --> FUNCS
    CONFIG --> DEPS
    PY --> METRICS
    CONFIG --> TECH_STACK
    
    PHASE5 --> FILE_NODES
    PHASE5 --> MODULE_NODES
    PHASE5 --> TECH_NODES
    PHASE5 --> REL_EDGES
```

## Container Configuration Intelligence

```mermaid
sequenceDiagram
    participant User
    participant Preview as Preview Service
    participant ConfigSvc as Container Config Service
    participant KG as Knowledge Graph
    participant LLM as LLM Manager
    participant Docker

    User->>Preview: Start Preview (Custom)
    Preview->>ConfigSvc: Generate Configuration
    ConfigSvc->>KG: Get Project Data
    KG-->>ConfigSvc: FILE Nodes, Tech Stack, Config
    
    ConfigSvc->>KG: Get package.json Scripts
    KG-->>ConfigSvc: Script Commands
    
    alt Multiple Scripts Available
        ConfigSvc->>LLM: Select Best Script
        Note over LLM: Analyzes scripts for:<br/>- Dev server commands<br/>- Framework patterns<br/>- Environment suitability
        LLM-->>ConfigSvc: Recommended Script
    end
    
    ConfigSvc-->>Preview: Container Config<br/>with Smart Command
    
    Preview->>Docker: Create Container
    Docker-->>Preview: Container Running
    Preview-->>User: Preview URL
```

## WebSocket Event Architecture (Enhanced)

```mermaid
flowchart LR
    subgraph "Event Sources"
        TASK_SVC[Task Service]
        GITHUB_SVC[GitHub Service]
        SEC_SVC[Security Service]
        KG_SVC[Knowledge Graph]
        DEPLOY_SVC[Deployment Service]
        SYSTEM[System Monitor]
    end
    
    subgraph "Enhanced WebSocket Manager"
        WS_MGR[WebSocket Manager<br/>Channel-based]
        CONN_POOL[Connection Pool<br/>User Tracking]
        MSG_FILTER[Message Filter<br/>Type-based]
        DEBOUNCE[Event Debouncer<br/>Performance]
    end
    
    subgraph "Event Channels"
        subgraph "Task Channel"
            TC[task_created]
            TU[task_updated]
            TP[task_progress]
            TCO[task_completed]
            TF[task_failed]
        end
        
        subgraph "System Channel"
            SH[system_health]
            SS[service_status]
            SA[security_alert]
            KGU[knowledge_graph_update]
        end
        
        subgraph "Deployment Channel"
            DS[deployment_started]
            DP[deployment_progress]
            DC[deployment_completed]
        end
    end
    
    subgraph "Clients"
        WEB1[Web Client 1<br/>Subscribed: tasks]
        WEB2[Web Client 2<br/>Subscribed: all]
        ADMIN[Admin Dashboard<br/>Subscribed: system]
    end
    
    TASK_SVC --> WS_MGR
    GITHUB_SVC --> WS_MGR
    SEC_SVC --> WS_MGR
    KG_SVC --> WS_MGR
    
    WS_MGR --> MSG_FILTER
    MSG_FILTER --> DEBOUNCE
    DEBOUNCE --> CONN_POOL
    
    CONN_POOL --> WEB1
    CONN_POOL --> WEB2
    CONN_POOL --> ADMIN
```

## Performance Architecture

```mermaid
flowchart TB
    subgraph "Frontend Optimizations"
        CONTEXTS[18 Optimized Contexts<br/>useMemoizedContextValue]
        CALLBACKS[Stable Callbacks<br/>useStableCallback]
        REFS[State Refs<br/>No Re-renders]
        DEBOUNCE[WebSocket Debouncing]
    end
    
    subgraph "Backend Optimizations"
        CACHE[Redis Caching<br/>Knowledge Graph Queries]
        BATCH[Batch Operations<br/>Bulk Node/Edge Creation]
        POOL[Connection Pooling<br/>PG & Neo4j]
        INDEXES[Database Indexes<br/>Optimized Queries]
    end
    
    subgraph "Processing Optimizations"
        QUEUES[Multi-Queue Celery<br/>Priority Routing]
        PARALLEL[Parallel Analysis<br/>Concurrent File Processing]
        RETRY[Smart Retry Logic<br/>Exponential Backoff]
    end
    
    CONTEXTS --> CALLBACKS
    CALLBACKS --> REFS
    REFS --> DEBOUNCE
    
    CACHE --> BATCH
    BATCH --> POOL
    POOL --> INDEXES
    
    QUEUES --> PARALLEL
    PARALLEL --> RETRY
```

## Deployment Architecture

```mermaid
flowchart TB
    subgraph "Development Environment"
        LOCAL[Local Docker Compose<br/>All Services]
        DEV_DB[(Development DBs)]
        OLLAMA_LOCAL[Local Ollama]
    end
    
    subgraph "Docker Services"
        subgraph "Core"
            CORE[crewwork-core]
            FRONTEND[crewwork-frontend]
            NGINX[crewwork-nginx]
        end
        
        subgraph "Workers"
            WORKER[celery-worker<br/>+ repo volume]
            BEAT[celery-beat]
            FLOWER[flower]
        end
        
        subgraph "Data"
            PG[postgres]
            REDIS[redis]
            NEO4J[neo4j]
        end
    end
    
    subgraph "Volumes"
        CODE_VOL[Source Code]
        REPO_VOL[Repositories]
        DATA_VOL[Database Data]
        SECRETS[Secrets]
    end
    
    LOCAL --> CORE
    CORE --> CODE_VOL
    WORKER --> REPO_VOL
    PG --> DATA_VOL
    CORE --> SECRETS
```