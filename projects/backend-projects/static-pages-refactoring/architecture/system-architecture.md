# 🏗️ **System Architecture Overview - Static Pages Refactoring**

## 📋 **Overview**
This document provides a comprehensive visual overview of the system architecture for the Static Pages Refactoring project, showing the complete flow from content management to static page delivery.

---

## 🔄 **Complete System Flow**

```mermaid
graph TD
    A[Umbraco CMS] -->|Publish/Unpublish| B[Delivery API]
    B -->|URL/GUID Message| C[Service Bus Queue]
    C -->|Message Processing| D[Azure Function]
    D -->|Content Orchestration| E[ContentOrchestrationService<br/>105 lines]
    E -->|Content Processing| F[ContentProcessingService<br/>184 lines]
    E -->|API Calls| G[DeliveryApiService<br/>99 lines]
    E -->|HTML Generation| H[StaticPageGenerationService<br/>69 lines]
    H -->|Generate HTML| I[Node.js Service]
    I -->|Static HTML| J[Akamai CDN]
    J -->|Serve Content| K[End Users]
    
    L[Error Handling] -->|Retry Logic| C
    L -->|Dead Letter Queue| M[Manual Recovery]
    
    style A fill:#e1f5fe
    style D fill:#f3e5f5
    style E fill:#e8f5e8
    style F fill:#e8f5e8
    style G fill:#e8f5e8
    style H fill:#e8f5e8
    style I fill:#fff3e0
    style J fill:#fce4ec
```

---

## 🎯 **Service Architecture Detail**

```mermaid
graph LR
    subgraph "Azure Function (457 lines total)"
        A[ContentOrchestrationService<br/>105 lines<br/>23%]
        B[ContentProcessingService<br/>184 lines<br/>40%]
        C[StaticPageGenerationService<br/>69 lines<br/>15%]
        D[DeliveryApiService<br/>99 lines<br/>22%]
    end
    
    subgraph "External Services"
        E[Umbraco Delivery API]
        F[Node.js HTML Service]
        G[Akamai CDN]
        H[Service Bus]
        I[Environment Variables]
    end
    
    A -->|Orchestrates| B
    A -->|Calls| C
    A -->|Uses| D
    D -->|API Calls| E
    C -->|HTML Generation| F
    C -->|Upload| G
    A -->|Receives Messages| H
    A -->|Configuration| I
    
    style A fill:#e8f5e8
    style B fill:#e8f5e8
    style C fill:#e8f5e8
    style D fill:#e8f5e8
```

---

## 🔧 **Before vs After Architecture**

### **Before: Monolithic Function**
```mermaid
graph TD
    A[Service Bus Message] --> B[Monolithic Function<br/>342 lines]
    B -->|All Logic| C[Content Validation]
    B -->|All Logic| D[URL Processing]
    B -->|All Logic| E[HTML Generation]
    B -->|All Logic| F[Akamai Upload]
    B -->|All Logic| G[Error Handling]
    B -->|All Logic| H[Retry Logic]
    B -->|All Logic| I[Logging]
    B -->|All Logic| J[Service Bus]
    B -->|All Logic| K[Environment Config]
    
    style B fill:#ffebee
    style C fill:#ffebee
    style D fill:#ffebee
    style E fill:#ffebee
    style F fill:#ffebee
    style G fill:#ffebee
    style H fill:#ffebee
    style I fill:#ffebee
    style J fill:#ffebee
    style K fill:#ffebee
```

### **After: Microservices Architecture**
```mermaid
graph TD
    A[Service Bus Message] --> B[ContentOrchestrationService<br/>105 lines<br/>23%]
    B -->|Delegates| C[ContentProcessingService<br/>184 lines<br/>40%]
    B -->|Delegates| D[StaticPageGenerationService<br/>69 lines<br/>15%]
    B -->|Uses| E[DeliveryApiService<br/>99 lines<br/>22%]
    
    C -->|Business Logic| F[Content Validation]
    C -->|Bulk Processing| G[Service Bus Integration]
    C -->|Language Support| H[Multi-language Processing]
    D -->|HTML Operations| I[HTML Generation]
    D -->|CDN Operations| J[Akamai Upload]
    E -->|API Operations| K[Delivery API Calls]
    E -->|Ancestor Loading| L[Lazy Loading]
    
    style B fill:#e8f5e8
    style C fill:#e8f5e8
    style D fill:#e8f5e8
    style E fill:#e8f5e8
    style F fill:#f1f8e9
    style G fill:#f1f8e9
    style H fill:#f1f8e9
    style I fill:#f1f8e9
    style J fill:#f1f8e9
    style K fill:#f1f8e9
    style L fill:#f1f8e9
```

---

## 📊 **Performance Flow**

```mermaid
sequenceDiagram
    participant U as Umbraco CMS
    participant SB as Service Bus
    participant AF as Azure Function
    participant CO as ContentOrchestration
    participant CP as ContentProcessing
    participant SG as StaticPageGeneration
    participant N as Node.js Service
    participant A as Akamai CDN
    
    U->>SB: Publish Content (URL)
    SB->>AF: Process Message
    AF->>CO: Orchestrate Processing
    
    CO->>CP: Process Content Type
    CP-->>CO: Processing Result
    
    CO->>SG: Generate Static Page
    SG->>N: Request HTML Generation
    N-->>SG: Return HTML
    SG->>A: Upload to CDN
    A-->>SG: Upload Confirmation
    SG-->>CO: Generation Complete
    
    CO-->>AF: Processing Complete
    AF-->>SB: Message Processed
```

---

## 🛡️ **Error Handling Flow**

```mermaid
graph TD
    A[Service Bus Message] --> B[Azure Function]
    B -->|Try| C[Process Content]
    C -->|Success| D[Complete Processing]
    C -->|Transient Error| E[Retry Logic]
    E -->|Retry Success| D
    E -->|Max Retries| F[Dead Letter Queue]
    C -->|Permanent Error| F
    F -->|Manual Recovery| G[Admin Intervention]
    G -->|Fix & Retry| A
    
    H[Monitoring] -->|Alert| I[Error Notification]
    F -->|Trigger| I
    
    style C fill:#e8f5e8
    style E fill:#fff3e0
    style F fill:#ffebee
    style G fill:#f3e5f5
```

---

## 📈 **Scalability Architecture**

```mermaid
graph TB
    subgraph "Load Balancer"
        LB[Azure Load Balancer]
    end
    
    subgraph "Azure Functions (Auto-scaling)"
        AF1[Function Instance 1]
        AF2[Function Instance 2]
        AF3[Function Instance N]
    end
    
    subgraph "Service Bus (Partitioned)"
        SB1[Queue Partition 1]
        SB2[Queue Partition 2]
        SB3[Queue Partition N]
    end
    
    subgraph "External Services"
        API[Umbraco API]
        NODE[Node.js Service]
        CDN[Akamai CDN]
    end
    
    LB --> AF1
    LB --> AF2
    LB --> AF3
    
    AF1 --> SB1
    AF2 --> SB2
    AF3 --> SB3
    
    AF1 --> API
    AF2 --> API
    AF3 --> API
    
    AF1 --> NODE
    AF2 --> NODE
    AF3 --> NODE
    
    AF1 --> CDN
    AF2 --> CDN
    AF3 --> CDN
```

---



## 🎯 **Key Architecture Benefits**

### **1. Separation of Concerns**
- **ContentOrchestrationService**: Workflow coordination (105 lines, 23%)
- **ContentProcessingService**: Business logic (184 lines, 40%)
- **StaticPageGenerationService**: HTML operations (69 lines, 15%)
- **DeliveryApiService**: External API communication (99 lines, 22%)

### **2. Scalability**
- **Independent Scaling**: Each service scales independently
- **Load Distribution**: Workload distributed across instances
- **Resource Optimization**: Resources allocated based on need

### **3. Reliability**
- **Fault Isolation**: Service failures don't cascade
- **Error Handling**: Comprehensive retry and recovery
- **Monitoring**: Real-time health monitoring

### **4. Maintainability**
- **Clear Boundaries**: Well-defined service responsibilities
- **Easy Testing**: Independent service testing
- **Code Reusability**: Services can be reused

---

*This system architecture demonstrates advanced microservices design principles and provides a solid foundation for scalable, maintainable, and reliable static page generation.* 🏗️