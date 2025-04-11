---
title: "Phase 3"
weight: 7
bookFlatSection: false
bookToc: true
bookHidden: false
bookCollapseSection: false
bookComments: false
bookSearchExclude: false
---
## 🌐 Phase 3: Reverse Proxy with Nginx

This phase introduces **Nginx** as a **reverse proxy**, acting as the single entry point to route traffic to backend microservices.

### 🎯 You’ll Learn:
- What is a reverse proxy and why it's important
- Routing and load balancing strategies
- How to serve APIs and static content behind Nginx
- Configuring HTTPS (SSL termination)

### 🛠 Topics Covered:
- Installing and configuring Nginx on Linux
- `nginx.conf` structure and virtual hosts
- Path-based routing:
  ```
  /api/users → User Service  
  /api/books → Book Service  
  /api/loans → Loan Service  
  ```
- Static file delivery for frontend (optional)
- Logging requests centrally
- Handling 404s and upstream errors




