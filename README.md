# Apache Kafka Tutorial - Course Agenda

## 📑 Contents
1. Introduction & Overview  
2. Kafka Terminology  
3. Kafka Architecture  
4. Deployment Strategies  
5. Kafka Installation & Setup  
6. Producers & Consumers  
7. Real-Time Project: Kafka Logging Pipeline  
8. Wrap-Up & Next Steps  

---

## 1. Introduction & Overview
- What is Apache Kafka?  
- Why Kafka? The problems it solves (scalability, high-throughput, fault tolerance, real-time streaming).  
- Real-world use cases: log aggregation, event streaming, metrics pipelines, messaging, analytics.  

---

## 2. Kafka Terminology
- **Topics** – logical categories for messages.  
- **Partitions** – how Kafka scales and ensures parallelism.  
- **Producers** – publish messages to topics.  
- **Consumers** – subscribe and read messages from topics.  
- **Brokers** – Kafka servers managing topics/partitions.  
- **Zookeeper / KRaft** – cluster coordination & metadata management.  
- **Consumer Groups & Offsets** – scaling consumers and tracking progress.  
- **Retention & Compaction Policies** – controlling log storage.  

---

## 3. Kafka Architecture
- Broker architecture and message flow.  
- Partitions and replication for HA & fault tolerance.  
- Producer internals: acknowledgements, batching, retries.  
- Consumer internals: polling, offset management, rebalancing.  
- Kafka storage layer and log structure.  
- Ecosystem overview: Kafka Connect, Kafka Streams, ksqlDB.  

---

## 4. Deployment Strategies
- On-premises vs. Cloud Kafka.  
- Self-managed vs. Managed (Confluent Cloud, AWS MSK, Aiven).  
- Single-node vs. multi-node deployments.  
- Scaling strategies: brokers, partitions, consumer groups.  
- HA and failover considerations.  

---

## 5. Kafka Installation & Setup
- Local single-node setup with binaries.  
- Multi-node cluster setup (ZooKeeper / KRaft).  
- Docker-based Kafka (docker-compose).  
- Kubernetes-based deployment.
- What is kafka UI and UI installation.
- How to Manage Kafka Deployment with ArgoCD approach. 
- Key configurations: heap sizing, disk setup, retention, security.  

---

## 6. Producers & Consumers
- Building a producer (Python) – sending messages.  
- Building a consumer – reading from topics.  
- Producer configs: acknowledgements, retries, idempotence.  
- Consumer group coordination and rebalancing.  
- Exactly-once semantics (EOS).  
- Performance tuning for throughput and latency.  

---

## 7. Real-Time Project Explination: Kafka Logging Pipeline
**Objective:** Implement a **real-time log processing system** using Kafka and ELK stack.  

- **Producers:** Applications send logs to Kafka topics.  
- **Consumer:** Logstash consumes logs from Kafka.  
- **Storage & Visualization:** Logstash forwards logs into Elasticsearch.  
- **Dashboard:** Use Kibana to explore and visualize real-time log data.  

This project demonstrates Kafka’s role in building a **centralized log aggregation system** for monitoring and analytics.  

---

## 8. Wrap-Up & Next Steps
- Kafka ecosystem recap.  
- Best practices for running Kafka in production.  
- Common pitfalls and troubleshooting tips.  
- Next steps:  
  - Schema Registry for data contracts.  
  - Kafka Streams & ksqlDB for stream processing.  
  - Security: TLS, SASL, RBAC.  
  - Monitoring Kafka clusters (Prometheus + Grafana).  

---
