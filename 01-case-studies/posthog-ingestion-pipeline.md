# How PostHog Ingests and Processes Events – A TPM Case Study

## 🔍 Overview

PostHog is an open source product analytics platform that helps teams capture, visualize, and act on user behavior data. It competes with tools like Mixpanel, Segment, and Amplitude.

A core part of PostHog’s product experience is its ability to ingest and process millions of events from client-side and server-side SDKs with low latency and high reliability.

This case study reverse-engineers PostHog’s ingestion pipeline — breaking down how it works, the technical tradeoffs involved, and the product implications of system design choices. It is written from the perspective of a Technical Product Manager responsible for scaling and evolving this part of the stack.

## ⚙️ Ingestion Architecture Breakdown

**1. Entry Point:**  
- Clients send events to the `/capture` endpoint via PostHog SDKs for web and mobile apps.  
- Events are JSON payloads sent over HTTP.

**2. API Gateway / Load Balancer:**  
- Handles routing incoming requests and enforces rate limiting to protect backend systems.  
- Tradeoff: Balancing latency vs. throttling abusive clients.

**3. Queue System:**  
- Uses Kafka to queue events asynchronously.  
- Kafka offers high throughput and durability but adds operational complexity.

**4. Processing Layer:**  
- Worker services consume from Kafka, validate, enrich, and transform events.  
- Data is then written to ClickHouse, a columnar database optimized for analytics.

**5. Storage:**  
- ClickHouse enables fast analytical queries on large event datasets.  
- Tradeoffs include fast reads but more complex write paths and infrastructure needs.

## 🧠 Key Technical Tradeoffs

| Area         | Decision          | Tradeoff                                                                 |
|--------------|-------------------|--------------------------------------------------------------------------|
| Queue System | Kafka             | High throughput and durability vs. operational complexity and maintenance overhead. |
| Storage      | ClickHouse        | Fast analytical reads vs. more complex write paths and infrastructure costs.      |
| API Endpoint | REST `/capture`   | Simplicity and broad compatibility vs. less efficient than binary protocols like gRPC. |
| Processing   | Batch workers     | Easier to implement and maintain vs. potential increased event processing latency. |


