---
title: "Some Distributed Systems Exploration"
weight: 12
bookFlatSection: false
bookToc: true
bookHidden: false
bookCollapseSection: false
bookComments: false
bookSearchExclude: false
---

## 🌍 Phase 7: Distributed Systems Exploration

Distributed systems are the backbone of scalable and fault-tolerant applications. In this phase, we explore **6+ distributed technologies**, understanding **how they work**, **what problems they solve**, and **where they fit in microservices**.

---

### 📦 1. **Apache Cassandra** – Decentralized NoSQL DB

**Architecture Highlights**:
- Peer-to-peer ring-based system using consistent hashing
- Eventual consistency with tunable quorum reads/writes
- Replication across data centers
- Uses SSTables and Memtables for fast write-heavy workloads

**Use Case**: High-volume write system for analytics or logs.

---

### ⚡ 2. **Apache Kafka** – Distributed Event Streaming Platform

**Architecture Highlights**:
- Distributed commit log with partitioned topics
- Producer → Kafka Broker → Consumer
- Message durability via segment files on disk
- High-throughput stream processing with horizontal scaling

**Use Case**: Decoupled microservice communication, event sourcing, stream analytics.

---

### 🧠 3. **Redis Cluster** – In-Memory Key-Value Store (Distributed Mode)

**Architecture Highlights**:
- Hash-slot-based sharding (16,384 slots)
- Master-replica architecture with automatic failover (via Sentinel or Cluster)
- High-speed data access with optional persistence

**Use Case**: Caching for microservices, pub/sub for chat or real-time updates.

---

### 🧮 4. **Etcd** – Distributed Key-Value Store for Configuration & Coordination

**Architecture Highlights**:
- Strongly consistent store using **Raft consensus**
- Frequently used in **Kubernetes** for state/config storage
- Designed for leader election, distributed locking, and configuration management

**Use Case**: Service registry, feature flag storage, cluster coordination.

---

### 🔎 5. **Consul** – Service Discovery and Key/Value Configuration

**Architecture Highlights**:
- Gossip-based peer discovery
- Offers DNS + HTTP APIs for service registration and health checking
- KV store with ACLs for configuration sharing
- Integrates with Envoy for service mesh features

**Use Case**: Discover microservices dynamically, share config globally across services.

---

### 🌐 6. **CockroachDB** – Distributed SQL Database

**Architecture Highlights**:
- Follows **Google Spanner-inspired architecture**
- Distributed ACID transactions via Raft
- Multi-region, horizontally scalable SQL
- Strong consistency with PostgreSQL compatibility

**Use Case**: Global backend for services needing relational queries and strong consistency.

---

### 📘 7. **MinIO** – Distributed Object Storage (S3-Compatible)

**Architecture Highlights**:
- Distributed erasure-coded storage
- Horizontal scaling with node auto-discovery
- API-compatible with AWS S3
- High-performance object store for cloud-native workloads

**Use Case**: Store file uploads, logs, media in a microservices app without relying on AWS.

---

## 🔄 Summary Table

| System        | Type                    | Architecture Keywords              | Ideal Use Case                          |
|---------------|-------------------------|------------------------------------|------------------------------------------|
| Cassandra     | NoSQL DB                | Peer-to-peer, AP, Sharding         | High-speed writes, analytics             |
| Kafka         | Event Streaming         | Broker, Partitioning, Log          | Messaging, decoupling, events            |
| Redis Cluster | In-Memory KV Store      | Sharding, Replication              | Caching, real-time data                  |
| Etcd          | Config Store            | Raft, Leader Election              | Cluster state/config (e.g., Kubernetes)  |
| Consul        | Service Discovery/KV    | Gossip, DNS-based discovery        | Microservices service registry           |
| CockroachDB   | Distributed SQL DB      | Raft, ACID, SQL + Scale            | Transactional workloads, global apps     |
| MinIO         | Object Storage          | Erasure coding, S3 API             | Distributed storage for files/media      |

---

### 🧠 Concepts Reinforced:
- **Consistency models** (Strong, Eventual, Tunable)
- **Consensus protocols** (Raft, Gossip)
- **Data partitioning and replication**
- **Service coordination and discovery**
- **State management in stateless systems**
