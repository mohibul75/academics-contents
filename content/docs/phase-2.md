---
title: "Phase 2"
weight: 7
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

### 2. **Book Service**
Manages book inventory, search, and updates to availability.

- 🚪 REST Base Path: `/api/books`
- 📦 Owns a **book database**.

### 3. **Loan Service**
Issues and returns books by communicating with both **User Service** and **Book Service**.

- 🚪 REST Base Path: `/api/loans`
- 📦 Owns a **loan database**.

---

## 🛢️ Databases (One per service)

| Service       | Database         | Tables                    |
|---------------|------------------|---------------------------|
| User Service  | `user_db`        | `users`                   |
| Book Service  | `book_db`        | `books`                   |
| Loan Service  | `loan_db`        | `loans`                   |

---

## 🔗 Inter-Service Communication

### Communication Patterns

In this microservices architecture, services communicate with each other through **synchronous HTTP/REST APIs**. Here's how the communication flows:

1. **Direct Service-to-Service Communication**:
   - The **Loan Service** directly calls APIs exposed by the **User Service** and **Book Service**.
   - For example, when issuing a book, the Loan Service:
     - Calls `GET /api/users/{id}` to verify the user exists
     - Calls `GET /api/books/{id}` to check book availability
     - Calls `PATCH /api/books/{id}/availability` to update book availability

### Implementation Details

- **Service URLs**: In development, services might use predefined URLs (e.g., `http://user-service:8081`, `http://book-service:8082`).
- **HTTP Clients**: Services use HTTP clients to make API calls.
- **Circuit Breakers**: Implement circuit breakers to handle failures gracefully.
- **Timeout Handling**: Set appropriate timeouts for inter-service calls to prevent cascading failures.

### Example: Loan Creation Flow

When a client sends a request to create a loan:

1. **Client** sends `POST /api/loans` to the **Loan Service**
2. **Loan Service** performs:
   ```
   GET /api/users/{user_id} → User Service
   ↓
   GET /api/books/{book_id} → Book Service
   ↓
   PATCH /api/books/{book_id}/availability → Book Service
   ↓
   INSERT into loan_db.loans
   ↓
   Return response to Client
   ```

### Error Handling

- If the **User Service** is unavailable, the **Loan Service** returns a 503 Service Unavailable response.
- If the **Book Service** is unavailable, the **Loan Service** returns a 503 Service Unavailable response.
- If the user doesn't exist, the **Loan Service** returns a 404 Not Found response.
- If the book doesn't exist or has no available copies, the **Loan Service** returns a 400 Bad Request response.

No shared database. Each service is **data-isolated** for decoupling and autonomy.

---

## 🧪 API Documentation (Microservices)

The Smart Library System's microservices architecture exposes RESTful APIs for each service. These APIs follow standard HTTP methods and status codes, allowing clients and other services to interact with the system.

### API Design Principles

- **RESTful Architecture**: Resources are accessed via standard HTTP methods (GET, POST, PUT, DELETE)
- **JSON Payloads**: All requests and responses use JSON format for data exchange
- **Service Isolation**: Each service has its own API namespace and database
- **Stateless Communication**: No client state is stored on the server between requests
- **Proper Status Codes**: HTTP status codes indicate success (2xx), client errors (4xx), or server errors (5xx)

The following endpoints demonstrate the core functionality of each microservice. You are open to add more APIs to complete this system based on additional requirements or use cases.

### 🔹 User Service Endpoints

#### `POST /api/users`
Create/register a new user.

**Request:**
```json
{
  "name": "Alice Smith",
  "email": "alice@example.com",
  "role": "student"
}
```

**Response:**
```json
{
  "id": 1,
  "name": "Alice Smith",
  "email": "alice@example.com",
  "role": "student",
  "created_at": "2025-05-03T13:30:00Z"
}
```

#### `GET /api/users/{id}`
Fetch user profile by ID.

**Response:**
```json
{
  "id": 1,
  "name": "Alice Smith",
  "email": "alice@example.com",
  "role": "student",
  "created_at": "2025-05-03T13:30:00Z",
  "updated_at": "2025-05-03T13:30:00Z"
}
```

#### `PUT /api/users/{id}`
Update user information.

**Request:**
```json
{
  "name": "Alice Johnson",
  "email": "alice.johnson@example.com"
}
```

**Response:**
```json
{
  "id": 1,
  "name": "Alice Johnson",
  "email": "alice.johnson@example.com",
  "role": "student",
  "created_at": "2025-05-03T13:30:00Z",
  "updated_at": "2025-05-03T14:15:00Z"
}
```

---

### 🔹 Book Service Endpoints

#### `POST /api/books`
Add a new book.

**Request:**
```json
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "copies": 3
}
```

**Response:**
```json
{
  "id": 42,
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "copies": 3,
  "available_copies": 3,
  "created_at": "2025-05-03T13:45:00Z"
}
```

#### `GET /api/books?search=clean`
Search for books by title, author, or keyword.

**Response:**
```json
{
  "books": [
    {
      "id": 42,
      "title": "Clean Code",
      "author": "Robert C. Martin",
      "isbn": "9780132350884",
      "copies": 3,
      "available_copies": 2
    },
    {
      "id": 43,
      "title": "Clean Architecture",
      "author": "Robert C. Martin",
      "isbn": "9780134494166",
      "copies": 2,
      "available_copies": 2
    }
  ],
  "total": 2,
  "page": 1,
  "per_page": 10
}
```

#### `GET /api/books/{id}`
Retrieve detailed information about a specific book.

**Response:**
```json
{
  "id": 42,
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "copies": 3,
  "available_copies": 2,
  "created_at": "2025-05-03T13:45:00Z",
  "updated_at": "2025-05-03T14:30:00Z"
}
```

#### `PUT /api/books/{id}`
Update book information.

**Request:**
```json
{
  "copies": 5
}
```

**Response:**
```json
{
  "id": 42,
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "copies": 5,
  "available_copies": 4,
  "created_at": "2025-05-03T13:45:00Z",
  "updated_at": "2025-05-03T15:10:00Z"
}
```

#### `PATCH /api/books/{id}/availability`
Update a book's available copies (used internally by Loan Service during issue/return).

**Request:**
```json
{
  "available_copies": 4,
  "operation": "increment"
}
```

**Response:**
```json
{
  "id": 42,
  "available_copies": 4,
  "updated_at": "2025-05-03T15:20:00Z"
}
```

#### `DELETE /api/books/{id}`
Remove a book from the catalog.

**Response:**
```
204 No Content
```

---

### 🔹 Loan Service Endpoints

#### `POST /api/loans`
Issue a book to a user.

**Request:**
```json
{
  "user_id": 1,
  "book_id": 42,
  "due_date": "2025-06-03T23:59:59Z"
}
```

**Process:**
- Validate `user_id` via `User Service`.
- Validate `book_id` and availability via `Book Service`.
- If all checks pass, reduce the book's available copy count.

**Response:**
```json
{
  "id": 1001,
  "user_id": 1,
  "book_id": 42,
  "issue_date": "2025-05-03T15:30:00Z",
  "due_date": "2025-06-03T23:59:59Z",
  "status": "ACTIVE"
}
```

#### `POST /api/returns`
Return a borrowed book.

**Request:**
```json
{
  "loan_id": 1001
}
```

**Process:**
- Update loan status.
- Increment book availability in **Book Service**.

**Response:**
```json
{
  "id": 1001,
  "user_id": 1,
  "book_id": 42,
  "issue_date": "2025-05-03T15:30:00Z",
  "due_date": "2025-06-03T23:59:59Z",
  "return_date": "2025-05-15T10:45:00Z",
  "status": "RETURNED"
}
```

#### `GET /api/loans/user/{user_id}`
Get a user's loan history (active and returned books).

**Response:**
```json
{
  "loans": [
    {
      "id": 1001,
      "book": {
        "id": 42,
        "title": "Clean Code",
        "author": "Robert C. Martin"
      },
      "issue_date": "2025-05-03T15:30:00Z",
      "due_date": "2025-06-03T23:59:59Z",
      "return_date": "2025-05-15T10:45:00Z",
      "status": "RETURNED"
    },
    {
      "id": 1002,
      "book": {
        "id": 43,
        "title": "Clean Architecture",
        "author": "Robert C. Martin"
      },
      "issue_date": "2025-05-10T09:15:00Z",
      "due_date": "2025-06-10T23:59:59Z",
      "status": "ACTIVE"
    }
  ],
  "total": 2
}
```

#### `GET /api/loans/{id}`
Get details of a specific loan.

**Response:**
```json
{
  "id": 1001,
  "user": {
    "id": 1,
    "name": "Alice Johnson",
    "email": "alice.johnson@example.com"
  },
  "book": {
    "id": 42,
    "title": "Clean Code",
    "author": "Robert C. Martin"
  },
  "issue_date": "2025-05-03T15:30:00Z",
  "due_date": "2025-06-03T23:59:59Z",
  "return_date": "2025-05-15T10:45:00Z",
  "status": "RETURNED"
}
```

---

## ⚙️ Deployment Strategy

Each microservice:
- Runs in its **own container or VM**.
- Has its **own codebase**, tests, deployment pipeline.
- Can be updated, scaled, or restarted **independently**.

---

## ✅ Advantages of Microservices

- **Independent Development**: Teams can work on different services simultaneously.
- **Technology Diversity**: Each service can use the most appropriate tech stack.
- **Fault Isolation**: One service failing doesn't crash the whole application.
- **Scalability**: Services can be scaled independently based on demand.
- **Focused Responsibility**: Each service has a clear, bounded context.

---

## ⚠️ Trade-offs

- **Increased Operational Complexity**: Managing multiple services requires more sophisticated DevOps.
- **Network Latency**: Inter-service communication adds overhead compared to function calls.
- **Data Consistency Challenges**: Maintaining consistency across service boundaries is harder.
- **Distributed Debugging**: Tracing issues across services requires specialized tooling.
- **Service Discovery**: Services need to locate and communicate with each other reliably.
