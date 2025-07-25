graph TD
    subgraph User Interaction Layer
        A[User Interface (Web/Mobile App)] -- User Query --> B(API Gateway/Load Balancer)
    end

    subgraph Application Backend (Microservices)
        B -- Requests --> C(Chatbot Service)
        C -- Observability/Monitoring --> L(LangSmith/Prometheus/Grafana)
        C -- Sends Events --> M(Message Queue - Kafka/RabbitMQ)
        M -- Consumed by --> L

        subgraph Chatbot Service (Python/FastAPI/Flask)
            C1(LangChain Application Logic)
            C2(LLM Provider API Key Management)
        end

        C1 -- LLM Calls --> D(LLM Provider - OpenAI/Gemini/Anthropic)
        C1 -- Embeddings Generation --> E(Embedding Model API - OpenAI/Cohere/Hugging Face)
        C1 -- Vector Search --> F(Vector Database - Pinecone/Chroma/Weaviate/Milvus)
        C1 -- SQL Queries --> G(SQL Database Service - PostgreSQL/MySQL)
        C1 -- API Calls --> H(External Public Database APIs/Custom APIs)
    end

    subgraph Data Ingestion & Knowledge Base Management (Offline/Batch)
        I(Data Sources)
        I -- Website Scraper --> I1[Website Content]
        I -- Code Documentation Parsers --> I2[Backend Code Docs (Markdown/PDF/JSON)]
        I -- Public Data Sources (Initial Sync) --> I3[Public Database Dump/APIs]

        I1 & I2 & I3 -- Document Loaders (LangChain) --> J(Preprocessing & Chunking Service)
        J -- Embeddings Generation --> K(Embedding Model API - same as E)
        K -- Indexing --> F
    end

    subgraph Ancillary Services
        N(Cache - Redis)
        C -- Caching --> N
        G -- Connects to --> G1[Actual Public Database (e.g., PostgreSQL for listings)]
        H -- Connects to --> H1[Actual External API Endpoints]
        L -- Stores Traces/Logs --> L1[Monitoring Dashboard/Log Aggregator]
        P(CI/CD Pipeline)
        P -- Deploys --> C
    end

    style A fill:#D0E0FF,stroke:#333,stroke-width:2px
    style B fill:#FFFACD,stroke:#333,stroke-width:2px
    style C fill:#D9FFD9,stroke:#333,stroke-width:2px
    style C1 fill:#F0FFF0,stroke:#333,stroke-width:1px
    style C2 fill:#F0FFF0,stroke:#333,stroke-width:1px
    style D fill:#FFDDC1,stroke:#333,stroke-width:2px
    style E fill:#FFDDC1,stroke:#333,stroke-width:2px
    style F fill:#ADD8E6,stroke:#333,stroke-width:2px
    style G fill:#FFC0CB,stroke:#333,stroke-width:2px
    style H fill:#E6E6FA,stroke:#333,stroke-width:2px
    style I fill:#F5F5DC,stroke:#333,stroke-width:2px
    style I1 fill:#F8F8FF,stroke:#333,stroke-width:1px
    style I2 fill:#F8F8FF,stroke:#333,stroke-width:1px
    style I3 fill:#F8F8FF,stroke:#333,stroke-width:1px
    style J fill:#FFE0B2,stroke:#333,stroke-width:2px
    style K fill:#FFDDC1,stroke:#333,stroke-width:2px
    style L fill:#FFEBCD,stroke:#333,stroke-width:2px
    style L1 fill:#F8F8FF,stroke:#333,stroke-width:1px
    style M fill:#ADD8E6,stroke:#333,stroke-width:2px
    style N fill:#F0F8FF,stroke:#333,stroke-width:2px
    style P fill:#DAF7A6,stroke:#333,stroke-width:2px
    style G1 fill:#F8F8FF,stroke:#333,stroke-width:1px
    style H1 fill:#F8F8FF,stroke:#333,stroke-width:1px
