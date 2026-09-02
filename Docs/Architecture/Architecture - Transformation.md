# Architectural Transformation

## 1. Purpose

This document defines the architectural transformation of the existing OTS application from a **product-dependent architecture** to an **independent, modular, scalable, and maintainable application architecture**.

The objective is not simply to replace the existing product with newer technologies. The transformation focuses on identifying the limitations of the current architecture, defining the desired architectural characteristics, and selecting technologies and patterns based on business and technical requirements.

Every major architectural decision will be evaluated using the following questions:

1. **Why are we using it?**
2. **How are we using it?**
3. **What changes will happen because of it?**
4. **What alternatives were considered?**

---

# 2. Current Architecture

## 2.1 Overview

The current OTS application is tightly coupled with a proprietary product that provides several core application capabilities.

The application is deployed across five application servers behind a load balancer and uses a centralized Oracle database.

```text
                         PNB USERS
                             |
                             v
                    +----------------+
                    | Load Balancer  |
                    +-------+--------+
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
      Server 1          Server 2          Server 3
          |                 |                 |
          +-----------------+-----------------+
                            |
                      +-----+-----+
                      |           |
                      v           v
                  Server 4     Server 5
                      |           |
                      +-----+-----+
                            |
                            v
                   +------------------+
                   |   OTS Product    |
                   |                  |
                   | UI               |
                   | Field Tables     |
                   | Runtime          |
                   | Configuration    |
                   | Custom Java      |
                   | OTS JAR          |
                   +--------+---------+
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
         Oracle ORCL   PNBWebClient     OmniDocs
                            |
                            v
                   Core Banking System
```

---

# 3. Current Architectural Components

## 3.1 Load Balancer

The load balancer acts as the entry point for application traffic.

It distributes incoming requests across the five application servers.

```text
Users
  |
  v
Load Balancer
  |
  +----> Server 1
  +----> Server 2
  +----> Server 3
  +----> Server 4
  +----> Server 5
```

The load balancer provides traffic distribution and server availability management.

---

## 3.2 Product Layer

The existing product provides several application capabilities:

* User interface
* Field and form management
* Runtime execution
* Configuration
* Workflow-related capabilities
* Product-specific Java execution
* Document-related functionality

Custom application logic operates within this product environment.

This creates a strong dependency between the business application and the product runtime.

---

## 3.3 Java Layer

Custom Java functionality is implemented within the product environment.

The Java layer performs activities such as:

* Business validations
* Database interaction
* API invocation
* OTS processing
* Data transformation
* Application-specific processing

The Java components therefore depend on the product runtime and its execution model.

---

## 3.4 Configuration and Query Management

Database queries are maintained through configuration mechanisms provided by the product.

This allows Java functionality to retrieve and execute configured queries rather than maintaining all database interaction as part of an independently structured data-access layer.

---

## 3.5 Database

The system uses a centralized Oracle database.

```text
                    Oracle ORCL
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
      Tables           Views         Procedures
        |                |                |
        +----------------+----------------+
                         |
                         v
                 Application Data
```

The database also contains procedures responsible for processing and joining data from multiple tables.

---

# 4. Current Business Flow

## 4.1 Eligibility

The process starts from the dashboard.

```text
User
 |
 v
Dashboard
 |
 v
Eligible Account Menu
 |
 v
Eligible Account Report
 |
 v
Oracle Database
 |
 v
Eligibility Procedure
 |
 +-- Multiple table joins
 +-- Eligibility conditions
 +-- Business rules
 |
 v
Eligible Accounts
```

The eligibility report identifies accounts satisfying the required OTS eligibility conditions.

---

## 4.2 OTS Creation

When the user opens an eligible account, a Create option is available.

The Create action initiates an API call to retrieve the balance as on the NPA date from the Core Banking System.

The current integration uses the `PNBWebClient` JAR, which is consumed by the OTS JAR as a library.

```text
Eligible Account
       |
       v
Create Button
       |
       v
    OTS JAR
       |
       v
PNBWebClient JAR
       |
       v
Core Banking System
       |
       v
Balance as on NPA Date
       |
       v
OTS Application
```

---

# 5. Current Architectural Characteristics

The current architecture can be summarized as follows:

| Area                 | Current Architecture                 |
| -------------------- | ------------------------------------ |
| Application          | Product-dependent                    |
| UI                   | Product-provided                     |
| Runtime              | Product runtime                      |
| Application servers  | 5                                    |
| Traffic management   | Load balancer                        |
| Database             | Oracle ORCL                          |
| Query management     | Product/configuration-based          |
| Business logic       | Java + Product + Database            |
| Eligibility          | Database procedures                  |
| External integration | JAR-based                            |
| CBS integration      | PNBWebClient JAR                     |
| Document management  | Product/OmniDocs                     |
| Deployment           | Product/application-server dependent |

---

# 6. Architectural Transformation Drivers

The transformation is primarily driven by the requirement to **remove dependency on the existing product**.

The major objectives are:

* Product independence
* Better separation of concerns
* Clear ownership of business logic
* Independent UI and backend
* Better API architecture
* Improved maintainability
* Improved testability
* Improved scalability
* Better observability
* Improved security
* Easier deployment
* Reduced vendor lock-in
* Greater flexibility for future development

---

# 7. Transformation Strategy

The transformation will be performed capability-by-capability rather than treating the application as one large replacement.

```text
Current Product Application
            |
            v
Identify Business Capabilities
            |
            v
Define Application Boundaries
            |
            v
Define APIs and Contracts
            |
            v
Build Independent Backend
            |
            v
Build Independent Frontend
            |
            v
Build Integration Layer
            |
            v
Build Document Services
            |
            v
Migrate Business Logic
            |
            v
Reduce Product Dependency
            |
            v
Independent OTS Application
```

---

# 8. Target Architecture

The target architecture removes the product as the central application runtime.

```text
                         PNB USERS
                             |
                             v
                    +----------------+
                    | Load Balancer  |
                    +-------+--------+
                            |
                            v
                    +----------------+
                    |  API Gateway   |
                    +-------+--------+
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
        +-----------+ +-----------+ +-----------+
        |    OTS    | | Customer  | | Reporting |
        |  Module   | |  Module   | |  Module   |
        +-----+-----+ +-----------+ +-----------+
              |
              v
        +-------------+
        |   Service   |
        |    Layer    |
        +------+------+
               |
       +-------+--------+----------------+
       |                |                |
       v                v                v
 Data Access       Integration       Document
    Layer              Layer           Service
       |                |                |
       v                v                v
 Oracle ORCL          CBS            OmniDocs
```

The exact decomposition into modules or services will be determined through further architectural analysis.

---

# 9. Transformation Area 1 — Product UI to Independent Frontend

## Why?

The current UI is controlled by the product.

This creates dependency on product-specific screen, field and form mechanisms.

The target architecture requires the application to own its presentation layer.

## How?

The frontend will become an independently developed web application.

```text
Browser
   |
   v
Independent Frontend
   |
   | REST/HTTP
   v
Backend APIs
```

## What changes?

### Current

```text
Product
  |
  +-- UI
  +-- Fields
  +-- Forms
```

### Target

```text
Frontend
  |
  +-- Screens
  +-- Forms
  +-- Components
  +-- Client-side validation
  +-- API communication
```

The frontend becomes independent of the product runtime.

---

# 10. Transformation Area 2 — Product Java to Independent Backend

## Why?

The current Java implementation operates within the product runtime.

The target architecture requires business functionality to be owned and executed by the application itself.

## How?

A structured backend architecture will be introduced.

```text
REST API
   |
   v
Controller
   |
   v
Service
   |
   v
Business Logic
   |
   v
Repository
   |
   v
Oracle
```

A Java enterprise framework such as Spring Boot may be considered for this layer.

The final technology selection will be based on project requirements and organizational standards.

## What changes?

```text
Current:

Product Runtime
      |
      v
Custom Java
      |
      v
Database


Target:

REST API
   |
Controller
   |
Service
   |
Repository
   |
Oracle
```

Business logic becomes independent from the product.

---

# 11. Transformation Area 3 — Business Logic Ownership

## Why?

Business rules are currently distributed across multiple layers, including Java code, product configuration and database procedures.

This makes ownership and maintenance more difficult.

## How?

Business rules will be analyzed and classified.

```text
Business Rules
      |
      +---- Application Business Logic
      |
      +---- Data Processing
      |
      +---- Database Optimization
```

Application-level business rules will be implemented in the service layer.

Database operations that are more efficiently executed using SQL may remain in the database.

## What changes?

### Current

```text
Business Logic
   |
   +-- Product
   +-- Java
   +-- Oracle
```

### Target

```text
Business Logic
      |
      v
Application Service Layer

Database
      |
      v
Persistence + Data Processing
```

---

# 12. Transformation Area 4 — JAR Integration to Integration Layer

## Why?

The current OTS implementation directly depends on the `PNBWebClient` JAR for Core Banking integration.

This creates coupling between OTS business logic and a specific integration mechanism.

## How?

An integration layer will isolate external-system communication.

```text
OTS Service
    |
    v
CBS Integration Interface
    |
    v
CBS Adapter / Client
    |
    v
Core Banking System
```

## What changes?

### Current

```text
OTS
 |
 v
OTS JAR
 |
 v
PNBWebClient JAR
 |
 v
CBS
```

### Target

```text
OTS
 |
 v
Integration Layer
 |
 v
CBS
```

The OTS business layer will not need to know the implementation details of the external communication mechanism.

---

# 13. Transformation Area 5 — Document Generation

## Why?

Document generation is currently closely associated with the product and OmniDocs.

The target system should treat document generation as an explicit application capability.

## How?

A dedicated document service can manage:

* Template management
* Data population
* Document generation
* PDF conversion
* Document metadata
* OmniDocs storage

```text
OTS Backend
     |
     v
Document Service
     |
     +-- Template
     +-- Data Population
     +-- PDF Generation
     +-- Metadata
     |
     v
OmniDocs
```

## What changes?

Document processing becomes independent of the product runtime.

---

# 14. Transformation Area 6 — Load Balancer

## Why?

The load balancer is not being removed merely because the architecture is being modernized.

It still provides:

* Traffic distribution
* High availability
* Health checks
* Horizontal scaling

## How?

```text
Users
  |
  v
Load Balancer
  |
  +---- Backend Instance 1
  +---- Backend Instance 2
  +---- Backend Instance 3
  +---- Backend Instance N
```

## What changes?

The load balancer remains part of the infrastructure, but the application instances behind it change.

```text
Current:

Load Balancer
      |
      v
Product Servers


Target:

Load Balancer
      |
      v
Independent Application Instances
```

The transformation therefore changes the **application behind the load balancer**, not necessarily the load balancer itself.

---

# 15. Transformation Area 7 — Database Access

## Why?

Oracle currently acts as both the persistence layer and a location for some business/data processing logic.

The objective is to establish a cleaner boundary between application logic and data access.

## How?

```text
Controller
    |
    v
Service
    |
    v
Repository / DAO
    |
    v
Oracle
```

## What changes?

The application gains explicit ownership of database access.

Existing procedures will be evaluated individually rather than being automatically migrated.

The goal is not:

> "Move everything from Oracle to Java."

The goal is:

> "Place each responsibility in the layer where it is most appropriate."

---

# 16. Transformation Area 8 — API Architecture

## Why?

The current architecture relies heavily on product-specific integration mechanisms.

The target system should expose well-defined APIs between application components.

## How?

```text
Frontend
    |
    v
API Layer
    |
    +---- OTS APIs
    +---- Customer APIs
    +---- Reporting APIs
    +---- Document APIs
```

APIs will establish clear contracts between frontend, backend and external systems.

## What changes?

Instead of direct product-dependent communication:

```text
UI → Product → Java
```

the target becomes:

```text
Frontend → API → Backend
```

---

# 17. Transformation Area 9 — Security

## Why?

Removing the product means that security capabilities previously provided or controlled by the product must be explicitly addressed.

## How?

Security will be designed across:

```text
Authentication
      |
Authorization
      |
API Security
      |
Data Security
      |
Audit Logging
```

## What changes?

Security becomes an explicit responsibility of the new application architecture instead of being primarily inherited from the product.

---

# 18. Transformation Area 10 — Observability

## Why?

The new application will require independent operational visibility.

## How?

The target architecture should support:

```text
Application
    |
    +---- Structured Logs
    |
    +---- Metrics
    |
    +---- Request IDs
    |
    +---- Distributed Tracing
    |
    v
Monitoring Platform
```

## What changes?

Issues can be traced across:

```text
Frontend
   |
   v
API
   |
   v
Service
   |
   v
Database
   |
   v
CBS
```

This improves production troubleshooting and operational support.

---

# 19. Technology Selection Framework

No technology should be introduced solely because it is considered modern.

Every major technology decision must answer:

## WHY?

What problem does the technology solve?

## HOW?

How will it fit into the architecture?

## WHAT CHANGES?

What impact will its introduction have on:

* Code
* Infrastructure
* Deployment
* Performance
* Security
* Operations
* Development process

## ALTERNATIVES

What other solutions could solve the same problem?

## DECISION

Why was the selected solution preferred?

---

# 20. Example Technology Decision

### Spring Boot

**Why?**

Potential candidate for building an independent Java backend with support for REST APIs, dependency injection, validation, security and database integration.

**How?**

```text
Frontend
   |
   v
Spring Boot REST API
   |
   v
Service Layer
   |
   v
Repository
   |
   v
Oracle
```

**What changes?**

* Product runtime dependency is removed.
* Business logic moves into an independently deployable backend.
* APIs become explicit.
* Unit and integration testing become easier.
* Backend deployment becomes independent.

**Alternatives:**

* Jakarta EE
* Quarkus
* Micronaut
* Plain Java

**Decision:**

To be finalized after evaluating project requirements, organizational standards, existing infrastructure and operational constraints.

---

# 21. Target Architectural Principles

The target architecture will follow these principles:

### 1. Product Independence

Business functionality should not depend on the proprietary product.

### 2. Separation of Concerns

Frontend, API, business logic, data access and integrations should have clearly defined responsibilities.

### 3. Loose Coupling

Components should communicate through well-defined interfaces.

### 4. High Cohesion

Each module should focus on a clearly defined business responsibility.

### 5. API-First Integration

External communication should use defined contracts and integration boundaries.

### 6. Independent Deployment

Components should be structured to support independent deployment where justified.

### 7. Security by Design

Security must be considered throughout the architecture.

### 8. Observability by Design

Logging, metrics and tracing should be incorporated from the beginning.

### 9. Technology Justification

Every major technology must have a documented architectural reason for its introduction.

---

# 22. Transformation Roadmap

```text
                    CURRENT
                       |
                       v
             Product-Coupled OTS
                       |
                       v
              Capability Analysis
                       |
                       v
             Define Architecture
                       |
                       v
          Independent Backend APIs
                       |
                       v
             Independent Frontend
                       |
                       v
             Integration Layer
                       |
                       v
              Document Service
                       |
                       v
             Security & Audit
                       |
                       v
               Observability
                       |
                       v
              Modern Deployment
                       |
                       v
                  TARGET
                       |
                       v
            Independent OTS Platform
```

---

# 23. Expected Outcome

The architectural transformation will move the OTS application from:

```text
Product-Centric Architecture
```

to:

```text
Business-Capability-Centric Architecture
```

### Current

```text
             Product
                |
      +---------+---------+
      |         |         |
     UI       Java      Config
                |
                v
             Oracle
                |
         External Systems
```

### Target

```text
              Independent OTS
                     |
       +-------------+-------------+
       |             |             |
   Frontend       Backend      Integration
                     |             |
                     |             +---- CBS
                     |             +---- OmniDocs
                     |
              +------+------+
              |             |
          Business       Data
           Logic         Access
              |             |
              +------+------+
                     |
                   Oracle
```

The transformation therefore establishes an independent application foundation while preserving existing enterprise systems such as Oracle, Core Banking and OmniDocs where they continue to satisfy business and technical requirements.

The target architecture should remain **modular first**, with service decomposition and microservices introduced only when independently deployable business capabilities, scaling requirements, availability requirements or organizational boundaries justify them.

