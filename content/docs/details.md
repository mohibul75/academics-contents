---
title: "Details"
weight: 2
bookFlatSection: false
bookToc: true
bookHidden: false         
bookCollapseSection: false
bookComments: false
bookSearchExclude: false
---
# 📚 Distributed Systems Lab — Detailed Breakdown

This lab is structured as a step-by-step journey through the lifecycle of building a distributed system with microservices.

---
## 📂 Reference Project

You will find a reference project for this lab in the following repository: [Distributed Systems Project](https://github.com/mohibul75/reference-project-for-distributed-systems-lab).

---

## 🚀 Phase 1: Monolithic Smart Library System

We begin with a **monolithic design**, where all components of the system reside in a single codebase.

**You’ll Learn:**
- How to structure a monolithic application
- Why monoliths become difficult to scale and maintain
- Building core modules: Users, Catalog, Borrowing
- Separation of concerns: Controller → Service → Repository layers

---

## 🧩 Phase 2: Transition to Microservices

Next, we break down the monolith into independently deployable **microservices**.

**You’ll Learn:**
- How to decompose services by business capability
- RESTful API communication between services
- Managing independent data sources
- Principles of loose coupling and bounded context

---

## 🌐 Phase 3: Reverse Proxy with Nginx

We introduce **Nginx** to centralize and manage access to microservices.

**You’ll Learn:**
- How reverse proxies work
- Nginx configuration for routing and load balancing
- Handling HTTPS and static files with Nginx
- Benefits of centralized access control and logging

---

## 🐳 Phase 4: Containerization with Docker

Now we **containerize** each service to ensure consistency across environments.

**You’ll Learn:**
- Writing Dockerfiles for services
- Managing containers, images, and networks
- Creating isolated environments for each service

---

## ⚙️ Phase 5: Managing with Docker Compose 

With more services, orchestration becomes important. Enter **Docker Compose**.

**You’ll Learn:**
- Defining multi-service environments in `docker-compose.yml`
- Networking between services
- Simplifying development and testing

---

## 🚢 Phase 6: Orchestration with Docker Swarm (Optional)

For real-world scalability, we use **Docker Swarm** to orchestrate services across a cluster.

**You’ll Learn:**
- Setting up a Swarm cluster
- Deploying services in a distributed fashion
- Achieving high availability and self-healing systems

---

## 🧠 Group Presentation on System Design

In this collaborative exercise, teams will tackle real-world system design challenges that scale from hundreds to millions of users. Each team will:

1. **Select a Scalable Project**: Choose from industry-relevant scenarios like social media timelines, URL shorteners, or distributed search engines
2. **Create a Multi-Stage Scaling Plan**: Design architecture that evolves through three growth stages (1K → 100K → 1M+ users)
3. **Specify Technical Components**: Detail infrastructure, databases, caching strategies, API design, and monitoring solutions
4. **Present with Diagrams**: Deliver a comprehensive presentation with architecture diagrams, justifications for design choices, and analysis of potential failure points

Teams will be evaluated on their application of distributed systems principles, scalability considerations, and technical communication skills.

## 📚 Group Presentation on Industry Trendy Distributed Systems

Teams will deeply explore modern distributed technologies that power today's scalable applications. For this presentation:

1. **Research & Analyze**: Thoroughly investigate an assigned distributed system technology (e.g., Cassandra, Kafka, Redis Cluster, Etcd, Consul, CockroachDB, or MinIO)
2. **Document Key Aspects**:
   - **Architecture**: Core components and structural design
   - **Working Principles**: Algorithms and data flow mechanisms
   - **High Availability**: Failure handling and redundancy approaches
   - **Use Cases**: Optimal application scenarios and problem domains
   - **Integration**: How it fits within microservices ecosystems
3. **Deliver Engaging Presentation**: Share insights with peers and faculty, emphasizing practical applications

Presentations should reinforce understanding of consistency models, consensus protocols, data partitioning, and state management in distributed systems.

---

## 🎓 Final Outcome

By the end of this lab, you will have:

- Built and deployed a functional microservices application
- Practiced containerization and orchestration
- Understood the real-world workflow of backend and DevOps engineering
- Strengthened your system design thinking

Ready to build your first distributed system? Let’s dive in! 🚀
