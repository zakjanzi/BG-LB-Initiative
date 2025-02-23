# BG-LB Initiative Digital Platform System Design



---

This document outlines two levels of system architecture for the tender application platform.

1. **Basic Architecture:**  
   A simple design suitable for the initial deployment/prototype. It is estimated to handle a moderate number of users.

2. **Production-Grade Architecture:**  
   An enhanced design that incorporates load balancing, resilient design, and fault tolerance to support high scalability and robustness.

Several factors play a role in deciding which design will address your needs, including the number of concurrent users, budget and available human-resources.

---

## 1. Basic Architecture Diagram (See image 1)

### Diagram


### How It Works

- **User Interaction:**  
  - A user accesses the platform using their web browser, logs in or registers, and views available tenders or calls via the Next.js frontend.
  - The frontend ensures fast load times and WCAG 2.1 accessibility while securely transmitting data to the backend.

- **Frontend to Backend Communication:**  
  - The Next.js frontend sends HTTPS requests to the API Gateway (implemented with FastAPI or Express, depending on the team's preference).
  - The security layer handles authentication and authorization using an Auth & Access module with OAuth2, 2FA, and role-based access control.

- **Tender Management Module:**  
  - **Role:** Receives tender submissions, writes submission data into the database, and retrieves AI-processed results (e.g., top three proposals).
  - **Data Flow:** Acts as the primary service interfacing with the database for tender-related data.

- **Queue System:**  
  - **Role:** Decouples the tender submission process from the heavy AI processing tasks by enqueuing jobs (using Celery or BullMQ).
  - **Data Flow:** Once a tender is submitted, a task containing the application data is sent to the queue system for AI processing.

- **AI Processing Pipeline:**  
  - **Load Balancer:** A simple load balancer directs queued tasks to available AI worker instances.
  - **AI Workers:** Multiple FastAPI worker instances pick up tasks, perform text preprocessing, scoring, ranking, and optionally compute vector embeddings.
  - **Results:** The processed results are written back into the database.

**A note regarding vector embeddings**: vector embeddings add value by enabling the AI processing pipeline to perform deep semantic analysis. 
They help ensure that the top-ranked proposals truly align with the tender's intent by capturing the contextual and nuanced meanings behind the text, rather than relying solely on basic keyword matching.

- **Database:**  
  - **Role:** Acts as the central store for tender submissions, user/security data, and AI processing results. A PostgreSQL database combined with Redis for caching is recommended.
  - **Integration:** Accessed by both the tender management module and AI workers for writing and reading data.

### Considerations

- **Simplicity:**  
  - This straightforward architecture is ideal for development or scenarios with lower user volume.
- **Security & Data Integrity:**  
  - The database is accessed exclusively via backend services (tender management and AI workers), protecting data from public internet exposure.
- **Limited Scalability:**  
  - Lacks advanced load balancing and fault tolerance, which may limit performance under high load.

---

## 2. Production-Grade Architecture Diagram (See image 2)

### Diagram


### How It Works

- **User Interaction:**  
  - A user accesses the platform via their web browser, logs in or registers, and views available tenders on the Next.js frontend.
  - The frontend maintains fast load times and WCAG 2.1 accessibility while securely sending data to the backend.

- **Frontend to Backend Communication:**  
  - The Next.js frontend sends HTTPS requests to the API Gateway (using FastAPI or Express).
  - A dedicated security layer (Auth & Access module) ensures authentication and authorization through OAuth2, 2FA, and role-based access control.

- **Global & Internal Load Balancing:**  
  - **Global Load Balancer:** Distributes incoming traffic from external clients (browsers, third-party systems) across multiple frontend server instances.
  - **Internal Load Balancer:** Routes traffic from the frontend to multiple API Gateway/backend instances, ensuring even distribution and high availability.

- **API Gateway / Backend:**  
  - **Role:** Acts as the unified entry point for all backend services, handling cross-cutting concerns like security, rate limiting, and logging.
  - **Integration:** Routes requests to the appropriate modules, such as the Auth & Access service and the Tender Management Module.

- **Tender Management Module:**  
  - **Role:** Handles tender-related operations (submission, validation, retrieval) and interfaces with the database.
  - **Data Flow:** Receives submissions via the API Gateway and sends tasks to the Queue System for AI processing.

- **Queue System with Scalable Workers:**  
  - **Role:** Manages and dispatches AI processing tasks asynchronously.
  - **Scalability:** Configured with auto-scaling worker instances (using Celery or BullMQ) to efficiently handle high loads.

- **AI Processing Module with Load Balancer:**  
  - **Load Balancer:** Distributes tasks among multiple AI worker instances.
  - **AI Workers:** Multiple FastAPI worker instances perform text preprocessing, scoring, ranking, and optionally compute vector embeddings (launched as container instances)
  - **Resilience:** Each worker is monitored via health checks and auto-scaling is managed to ensure fault tolerance.

- **Data Storage & Caching:**  
  - **Role:** A clustered PostgreSQL database stores all persistent data, including tender submissions and processed AI results.
  - **Caching:** A Redis cluster caches frequently accessed data, enhancing performance and reducing load on the primary database.

### Considerations

- **Scalability:**  
  - Global and internal load balancers enable horizontal scaling across frontend servers, backend services, and AI processing workers.
  - Auto-scaling and container orchestration (e.g., Kubernetes) support dynamic adjustment to traffic spikes.
  
- **Fault Tolerance & Resilience:**  
  - Redundant components and regular health checks ensure that failures are isolated and handled gracefully.
  - Clustered databases and caching layers provide data redundancy and faster recovery in case of a failure.
  
- **Security & Performance:**  
  - The API Gateway enforces strong security policies (authentication, authorization, rate limiting) before routing requests.
  - The distributed architecture minimizes single points of failure and handles high volumes of requests efficiently.

---
