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

## 🚀 Product Implications

- The ingestion pipeline design enables PostHog to provide near real-time dashboards and user behavior heatmaps, which are critical for product teams to make timely decisions.
- Batch processing trades off a bit of latency for easier maintenance and reliability, but high-frequency clients might experience slight delays in event availability.
- Rate limiting at the API gateway protects backend stability but can impact user experience if limits are too aggressive.
- Scaling Kafka and ClickHouse to handle 10x current traffic requires careful capacity planning, monitoring, and potential partitioning strategies to maintain performance.

## 🔧 Improvement Proposal (Optional)

If I were the TPM owning this ingestion system, I’d consider:

- Introducing schema validation at the API layer to reject malformed events early and reduce processing errors.
- Implementing adaptive rate limiting that considers client usage patterns to better accommodate burst traffic without sacrificing stability.
- Exploring streaming processing (e.g., Apache Flink) to reduce latency compared to batch workers.
- Adding more granular monitoring and alerting on queue lag and processing times to catch issues before they impact users.

---

## 📌 PM Takeaways

- Architectural trade-offs in infra systems directly impact product performance and user experience.
- TPMs must deeply understand these trade-offs to prioritize roadmap and guide engineering efforts.
- Building scalable, reliable pipelines requires close coordination across teams and constant iteration.
- Thinking in terms of throughput, latency, and system resiliency is critical for TPM success.

---

## 📚 Sources

- [PostHog Documentation](https://posthog.com/docs)
- [PostHog GitHub Repo](https://github.com/posthog/posthog)
- [PostHog Blog on Event Ingestion](https://posthog.com/blog/event-ingestion-pipeline)
