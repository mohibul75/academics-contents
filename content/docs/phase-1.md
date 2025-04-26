---
title: "Phase 1"
weight: 6
bookFlatSection: false
bookToc: true
bookHidden: false         
bookCollapseSection: false
bookComments: false
bookSearchExclude: false
---

# 📘 Smart Library System – Monolithic Architecture

## Overview

The **Smart Library System (Monolithic Version)** is a single, unified application that handles all core functionalities: managing users, books, and book loans. This system is ideal for simple deployments where all components are tightly coupled, sharing the same runtime and database.

---

## 🧩 Functional Modules

### 1. **User Management Module**
- Register a user (students/faculty).
- Update user profile.
- Retrieve user info.

### 2. **Book Management Module**
- Add/update/remove books.
- View book availability.
- Search books by title, author, or genre.

### 3. **Loan Management Module**
- Issue books to users.
- Return books.
- View active/past loans.

---

## 🛢️ Unified Database Schema

| Table         | Description                        |
|---------------|------------------------------------|
| `users`       | Stores user information.           |
| `books`       | Stores book catalog details.       |
| `loans`       | Tracks issued/returned books.      |

All modules interact with this **shared relational database**, typically PostgreSQL or MySQL.

---

## 🔄 Internal Communication

- All module calls happen via **function calls or internal classes**.
- Tight coupling between modules.
- No network-based interaction — all components reside in the same codebase and memory space.

---

## 🧪 API Documentation (REST Endpoints)

The Smart Library System exposes a comprehensive RESTful API that follows standard HTTP methods and status codes. This API allows external clients (web applications, mobile apps, CLI tools) to interact with the system.

### API Design Principles

- **RESTful Architecture**: Resources are accessed via standard HTTP methods (GET, POST, PUT, DELETE)
- **JSON Payloads**: All requests and responses use JSON format for data exchange
- **Consistent Naming**: Endpoints follow a consistent naming convention (/api/{resource})
- **Stateless Communication**: No client state is stored on the server between requests
- **Proper Status Codes**: HTTP status codes indicate success (2xx), client errors (4xx), or server errors (5xx)

The following endpoints demonstrate the core functionality of the monolithic library system. You are open to add more APIs to complete this system based on additional requirements or use cases.

### 🔹 User Endpoints

#### `POST /api/users`
Create/register a new user.
```json
{
  "name": "Alice Smith",
  "email": "alice@example.com",
  "role": "student"
}
```

#### `GET /api/users/{id}`
Fetch user profile by ID.

---

### 🔹 Book Endpoints

#### `POST /api/books`
Add a new book.
```json
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "copies": 3
}
```

#### `GET /api/books?search=clean`
Search for books by title, author, or keyword.

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
  "created_at": "2025-01-15T10:30:00Z",
  "updated_at": "2025-01-15T10:30:00Z"
}
```

#### `PUT /api/books/{id}`
Update book information.

**Request:**
```json
{
  "copies": 5,
  "available_copies": 3
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
  "available_copies": 3,
  "created_at": "2025-01-15T10:30:00Z",
  "updated_at": "2025-04-27T01:54:00Z"
}
```

#### `DELETE /api/books/{id}`
Remove a book from the catalog.

**Response:**
```
204 No Content
```

---

### 🔹 Loan Endpoints

#### `POST /api/loans`
Issue a book to a user.

**Request:**
```json
{
  "user_id": 1,
  "book_id": 42,
  "due_date": "2025-05-15T23:59:59Z"
}
```

**Response:**
```json
{
  "id": 1001,
  "user_id": 1,
  "book_id": 42,
  "issue_date": "2025-04-27T01:54:00Z",
  "due_date": "2025-05-15T23:59:59Z",
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

**Response:**
```json
{
  "id": 1001,
  "user_id": 1,
  "book_id": 42,
  "issue_date": "2025-04-27T01:54:00Z",
  "due_date": "2025-05-15T23:59:59Z",
  "return_date": "2025-04-30T14:22:00Z",
  "status": "RETURNED"
}
```

#### `GET /api/loans/{user_id}`
View loan history for a user.

**Response:**
```json
[
  {
    "id": 1001,
    "book": {
      "id": 42,
      "title": "Clean Code",
      "author": "Robert C. Martin"
    },
    "issue_date": "2025-04-27T01:54:00Z",
    "due_date": "2025-05-15T23:59:59Z",
    "return_date": "2025-04-30T14:22:00Z",
    "status": "RETURNED"
  },
  {
    "id": 1002,
    "book": {
      "id": 57,
      "title": "Design Patterns",
      "author": "Erich Gamma et al."
    },
    "issue_date": "2025-04-20T09:15:00Z",
    "due_date": "2025-05-04T23:59:59Z",
    "return_date": null,
    "status": "ACTIVE"
  }
]
```

#### `GET /api/loans/overdue`
List all overdue loans.

**Response:**
```json
[
  {
    "id": 985,
    "user": {
      "id": 15,
      "name": "John Smith",
      "email": "john@example.com"
    },
    "book": {
      "id": 23,
      "title": "The Pragmatic Programmer",
      "author": "Andrew Hunt, David Thomas"
    },
    "issue_date": "2025-03-15T10:30:00Z",
    "due_date": "2025-03-29T23:59:59Z",
    "days_overdue": 29
  }
]
```

#### `PUT /api/loans/{id}/extend`
Extend the due date for a loan.

**Request:**
```json
{
  "extension_days": 7
}
```

**Response:**
```json
{
  "id": 1002,
  "user_id": 1,
  "book_id": 57,
  "issue_date": "2025-04-20T09:15:00Z",
  "original_due_date": "2025-05-04T23:59:59Z",
  "extended_due_date": "2025-05-11T23:59:59Z",
  "status": "ACTIVE",
  "extensions_count": 1
}
```

### 🔹 Statistics Endpoints

#### `GET /api/stats/books/popular`
Get the most borrowed books.

**Response:**
```json
[
  {
    "book_id": 42,
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "borrow_count": 28
  },
  {
    "book_id": 57,
    "title": "Design Patterns",
    "author": "Erich Gamma et al.",
    "borrow_count": 23
  },
  {
    "book_id": 23,
    "title": "The Pragmatic Programmer",
    "author": "Andrew Hunt, David Thomas",
    "borrow_count": 19
  }
]
```

#### `GET /api/stats/users/active`
Get the most active users.

**Response:**
```json
[
  {
    "user_id": 15,
    "name": "John Smith",
    "books_borrowed": 12,
    "current_borrows": 3
  },
  {
    "user_id": 8,
    "name": "Emma Johnson",
    "books_borrowed": 10,
    "current_borrows": 1
  }
]
```

#### `GET /api/stats/overview`
Get system overview statistics.

**Response:**
```json
{
  "total_books": 1250,
  "total_users": 430,
  "books_available": 1089,
  "books_borrowed": 161,
  "overdue_loans": 12,
  "loans_today": 8,
  "returns_today": 5
}
```

---

## ⚠️ Limitations of Monolithic Design

- Hard to scale individual components independently.
- Tight coupling makes it difficult to change or test modules in isolation.
- Single point of failure: one bug can crash the entire app.
- Deployment of small changes requires redeploying the whole system.
- Technology stack is uniform across all modules.
- Development becomes slower as the codebase grows.
- Team coordination becomes challenging with larger teams.
