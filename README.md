# 📚 SlimPHP Book Borrowing API – Candidate Assignment

## 👋 Overview

You are building a secure REST API using **SlimPHP**, **MySQL**, and **OAuth2** for a book borrowing system.

The API should support:
- Book creation and borrow logging
- OAuth2-protected access
- Analytics powered by SQL **window functions (`PARTITION BY`)**
- Full-text search with summary statistics
- Middleware-based logging using **Monolog**
- Proper dependency injection using **PHP-DI**

This assignment evaluates your ability to:
- Build a modular, production-grade SlimPHP backend
- Use **`php-di/slim-bridge`** for dependency injection
- Model SQL data relationships with proper schema
- Implement secure token-based authentication
- Write SQL analytics using **window functions**

---

## 🧱 Part 1: Core API Endpoints

Implement the following authenticated REST endpoints:

| Method | Endpoint                  | Auth Required | Description                              |
|--------|---------------------------|---------------|------------------------------------------|
| POST   | `/oauth/token`            | ❌             | Issue an OAuth2 token via password grant |
| POST   | `/books`                  | ✅             | Add a new book                           |
| GET    | `/books`                  | ✅             | List all books                           |
| POST   | `/books/{bookId}/borrow`  | ✅             | Record a user borrowing a book           |
| GET    | `/books/{bookId}/borrows` | ✅             | List borrow logs for a given book        |

> All protected endpoints must use a Bearer token:  
> `Authorization: Bearer <access_token>`

---

## 🔐 Part 2: OAuth2 Authentication

Implement OAuth2 authentication using:

- [`league/oauth2-server`](https://oauth2.thephpleague.com/)

Requirements:
- Use **password grant** flow
- Store user credentials in the `users` table (hashed)
- Seed the database with at least **2 test users**

---

## 🗃️ Part 3: Database Schema

Use the following schema with **exact column names**:

```sql
CREATE TABLE users (
    userId INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    passwordHash VARCHAR(255) NOT NULL
);

CREATE TABLE books (
    bookId INT AUTO_INCREMENT PRIMARY KEY,
    bookTitle VARCHAR(255) NOT NULL,
    bookAuthor VARCHAR(255),
    bookPublishYear INT,
    FULLTEXT(bookTitle, bookAuthor)
);

CREATE TABLE borrowlog (
    borrowLogId INT AUTO_INCREMENT PRIMARY KEY,
    bookId INT NOT NULL,
    userId INT NOT NULL,
    borrowLogDateTime DATETIME NOT NULL,
    FOREIGN KEY (bookId) REFERENCES books(bookId),
    FOREIGN KEY (userId) REFERENCES users(userId)
);
```

- Enable **full-text search** on `bookTitle` and `bookAuthor`
- Use camelCase naming for all fields
- You may use migrations or raw SQL for setup

---

## 📊 Part 4: Analytics Endpoints (Using `PARTITION BY`)

Implement the following **authenticated** analytics endpoints using **SQL window functions**, specifically `ROW_NUMBER() OVER (PARTITION BY ...)`.

Do not use only `GROUP BY` or subqueries — the logic must involve proper window functions.

Return all results in clean JSON format.

### 1. `GET /analytics/latest-borrow-per-book`
**Purpose:** Return the most recent borrow log for each book.

**Expected Output per Record:**
- `bookId`
- `userId`
- `borrowLogDateTime`

### 2. `GET /analytics/borrow-rank-per-user`
**Purpose:** Return each borrow action ranked chronologically for each user-book pair.

**Expected Output per Record:**
- `borrowLogId`
- `userId`
- `bookId`
- `borrowLogDateTime`
- `borrowRank` (e.g., 1st, 2nd borrow, etc.)

### 3. `GET /analytics/book-summary`
**Purpose:** Return a summary of each book, including:
- Book info
- Total number of times the book has been borrowed
- Username of the most recent borrower (if any)

**Optional Query Parameter:**
- `?query=...` — If provided, filter books using full-text search on `bookTitle` and `bookAuthor`.

**Expected Output per Record:**
- `bookId`
- `bookTitle`
- `bookAuthor`
- `bookPublishYear`
- `borrowCount`
- `lastBorrowedBy` (may be `null` if never borrowed)

---

## 🧾 Part 5: Request Logging (Required)

Implement a **Slim middleware** that logs every incoming request to a file using [`monolog/monolog`](https://github.com/Seldaek/monolog).

**Logging requirements:**
- HTTP method (e.g., GET, POST)
- Request path (e.g., `/books/3/borrow`)
- Authenticated `userId` if available (or `null`)

**Log format:** Plain text or JSON — must be easily parsable  
**Log output file:** e.g., `logs/requests.log`

---

## 🧩 Part 6: Dependency Injection

Use **`php-di/slim-bridge`** for dependency injection in your Slim application.

**Requirements:**
- Register controllers, services, and loggers using the PHP-DI container
- Use constructor injection instead of `new` inside routes
- Use config files or factory definitions where appropriate

**Expected Structure:**
- `index.php` bootstraps Slim using `php-di/slim-bridge`
- Routes use services injected from the container
- Monolog logger is injected via DI

---

## 🚀 Setup Instructions

1. Clone or unzip the project  
2. Create a MySQL database and import the schema  
3. Create a `.env` file with DB and OAuth config  
4. Install dependencies via Composer:
   ```bash
   composer install
   ```
5. Run the SlimPHP app:
   ```bash
   php -S localhost:8080 -t public
   ```
6. Use Postman or `curl` to test all endpoints

---

## 📦 Deliverables

Please submit the following in your preferred format (GitHub repo or ZIP file):

- ✅ All source code  
- ✅ `README.md` with setup steps  
- ✅ `sql/schema.sql` or migration files  
- ✅ All required endpoints implemented  
- ✅ Middleware logging with Monolog  
- ✅ DI setup using `php-di/slim-bridge`  
- ✅ At least 2 seeded test users  
- ✅ Sample cURL or Postman test cases  

---

## 📝 Notes

- Use Composer for all dependency management  
- Use Slim’s routing + `php-di/slim-bridge` for DI  
- Follow SOLID principles where possible  
- All inputs must be validated and safe  
- Code must be compatible with **PHP 8.x**