---
title: "Phase 1"
weight: 5
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

## 🧪 Example API Documentation (REST Endpoints)

Here’s how external clients (like CLI tools or a potential frontend) interact with the system.

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

---

### 🔹 Loan Endpoints

#### `POST /api/loans`
Issue a book to a user.
```json
{
  "user_id": 1,
  "book_id": 42
}
```

#### `POST /api/returns`
Return a borrowed book.
```json
{
  "loan_id": 1001
}
```

#### `GET /api/loans/{user_id}`
View loan history for a user.

---

## ⚠️ Limitations of Monolithic Design

- Hard to scale individual components independently.
- Tight coupling makes it difficult to change or test modules in isolation.
- Single point of failure: one bug can crash the entire app.
- Deployment of small changes requires redeploying the whole system.