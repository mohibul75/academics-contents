---
title: "Phase 2"
weight: 6
bookFlatSection: false
bookToc: true
bookHidden: false
bookCollapseSection: false
bookComments: false
bookSearchExclude: false
---

# 🧩 Smart Library System – Microservices Architecture

## Overview

In the **microservices version** of the Smart Library System, the application is divided into three independent services — each responsible for a specific domain: **User**, **Book**, and **Loan**. Every service has its **own database** and communicates with others via **HTTP APIs** (no queues or Kafka involved in this version).

---

## 🧱 Services Overview

### 1. **User Service**
Handles registration, profile management, and user-related queries.

- 🚪 REST Base Path: `/api/users`
- 📦 Owns a **user database**.

#### 🔹 API Endpoints

##### `POST /api/users`
Register a new user.
```json
{
  "name": "Alice",
  "email": "alice@example.com",
  "role": "student"
}
```

##### `GET /api/users/{id}`
Fetch user details by ID.

---

### 2. **Book Service**
Manages book inventory, search, and updates to availability.

- 🚪 REST Base Path: `/api/books`
- 📦 Owns a **book database**.

#### 🔹 API Endpoints

##### `POST /api/books`
Add a new book.
```json
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "copies": 3
}
```

##### `GET /api/books?search=code`
Search books by title, author, or keyword.

##### `PATCH /api/books/{id}`
Update a book's available copies (used by Loan Service during issue/return).
```json
{
  "copies": 2
}
```

---

### 3. **Loan Service**
Issues and returns books by communicating with both **User Service** and **Book Service**.

- 🚪 REST Base Path: `/api/loans`
- 📦 Owns a **loan database**.

#### 🔹 API Endpoints

##### `POST /api/loans`
Create a loan.
```json
{
  "user_id": 1,
  "book_id": 42
}
```

**Process:**
- Validate `user_id` via `User Service`.
- Validate `book_id` and availability via `Book Service`.
- If all checks pass, reduce the book's available copy count.

---

##### `POST /api/returns`
Return a borrowed book.
```json
{
  "loan_id": 1001
}
```

**Process:**
- Update loan status.
- Increment book availability in **Book Service**.

---

##### `GET /api/loans/user/{user_id}`
Get a user's loan history (active and returned books).

---

## 🔗 Inter-Service Communication

- **Loan Service** makes HTTP calls to:
  - `User Service`: to validate user identity.
  - `Book Service`: to check and update book inventory.

No shared database. Each service is **data-isolated** for decoupling and autonomy.

---

## 🛢️ Databases (One per service)

| Service       | Database         | Tables                    |
|---------------|------------------|---------------------------|
| User Service  | `user_db`        | `users`                   |
| Book Service  | `book_db`        | `books`                   |
| Loan Service  | `loan_db`        | `loans`                   |

---

## ⚙️ Deployment Strategy

Each microservice:
- Runs in its **own container or VM**.
- Has its **own codebase**, tests, deployment pipeline.
- Can be updated, scaled, or restarted **independently**.

---

## ✅ Advantages of Microservices

- Independent development and deployment.
- Fault isolation: one service failing doesn’t crash the whole app.
- Easier scaling: Book Service can be scaled independently if traffic spikes.

---

## ⚠️ Trade-offs

- Increased operational complexity.
- Requires robust **service discovery**, **monitoring**, and **API versioning**.
- Debugging across services can be harder without centralized logs.
