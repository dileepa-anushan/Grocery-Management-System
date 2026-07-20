<p align="center">
  <h1 align="center">🛒 Grocery Management System</h1>
  <p align="center">
    A Java EE (Servlet + JSP) web application for managing an online grocery store —
    products, inventory, orders, payments, reviews, and admin operations — with a custom
    flat-file data layer and DSA-driven sorting/queueing logic.
  </p>
</p>
<br />

## 📋 Table of Contents
- [Introduction](#-introduction)
- [Key Features](#-key-features)
- [Tech Stack](#-tech-stack)
- [System Architecture](#-system-architecture)
- [User Roles](#-user-roles)
- [Data Model](#-data-model)
- [Servlet / URL Structure](#-servlet--url-structure)
- [Technical Highlights](#-technical-highlights)
- [Prerequisites](#-prerequisites)
- [Project Setup](#-project-setup)
- [Running the Application](#-running-the-application)
- [Project Structure](#-project-structure)
- [Contributors](#-contributors)

---

## 🚀 Introduction
**Grocery Management System** is a full-stack Java EE web application that digitizes the
day-to-day operations of an online grocery shop. It covers the complete customer journey —
browsing products, managing a cart, checking out, paying, and reviewing purchases — alongside
a dedicated admin console for managing inventory, products, orders, users, and transactions.

Unlike typical database-backed apps, this project uses a **custom file-based persistence
layer**: all entities (users, products, cart items, orders, payments, reviews, transactions)
are stored and retrieved from plain `.txt` files under `WEB-INF/data/`, managed through a
DAO layer built on top of a shared `FileHandlerUtil`.

---

## ✨ Key Features
| Module | Description |
|---|---|
| **Product Catalog** | Browse, view, and manage grocery products with categories, pricing, stock, and images |
| **Shopping Cart** | Add/update/remove items, persisted per user |
| **Order Management** | Place orders, track status, view order history and details |
| **Payments** | Checkout flow with saved cards, payment confirmation, and error handling |
| **Reviews & Ratings** | Customers can create, edit, and view product reviews with aggregated stats |
| **Inventory Control** | Stock tracking with low-stock alerts and validation rules |
| **Admin Dashboard** | Centralized console for products, inventory, orders, users, reviews, and transactions |
| **Transactions & Refunds** | Transaction history with refund and confirmation workflows |
| **User Accounts** | Registration, login, and profile management with role-based access |
| **First-Time Setup** | Bootstraps a default admin account on first launch |

---

## 🛠️ Tech Stack
| Technology | Purpose |
|---|---|
| **Java 8** | Core application language |
| **Jakarta/Java Servlet API 4.0.1** | Request handling and controllers (`javax.servlet`) |
| **JSP + JSTL 1.2** | Server-rendered views |
| **Apache Tomcat** | Servlet container / application server |
| **Maven** (`war` packaging) | Build and dependency management |
| **org.json** | JSON serialization/parsing |
| **Custom flat-file storage** | Persistence layer (`.txt` files) instead of a database |
| **HTML / CSS / Vanilla JS** | Front-end styling (incl. a dark theme) and interactivity |

---

## 🏛️ System Architecture
```
┌───────────────────────────────────────────────────────────┐
│                        BROWSER                             │
│                  JSP Views + CSS + JS                      │
└───────────────────────────┬─────────────────────────────────┘
                            │ HTTP
                            ▼
┌───────────────────────────────────────────────────────────┐
│                   SERVLET LAYER (Controllers)               │
│  HomeServlet · UserServlet · ProductServlet · CartServlet   │
│  InventoryServlet · OrderServlet · PaymentServlet           │
│  ReviewServlet · TransactionServlet · AdminDashboardServlet │
│  FirstTimeSetupServlet                                      │
└───────────────────────────┬─────────────────────────────────┘
                            │
      ┌─────────────────────┼─────────────────────┐
      ▼                     ▼                     ▼
┌───────────┐      ┌────────────────┐      ┌────────────────┐
│  Services │      │   Util Layer   │      │  DTO / Models  │
│ OrderProcessing   │ SecurityUtil   │      │ User, Product, │
│ OrderSorting      │ ValidationUtil │      │ Cart, Order,   │
│ ProductSorting    │ FileHandlerUtil│      │ Payment, etc.  │
│ PaymentService    │                │      │                │
└─────┬─────┘      └────────────────┘      └────────────────┘
      │
      ▼
┌───────────────────────────────────────────────────────────┐
│                       DAO LAYER                             │
│ UserDAO · ProductDAO · CartDAO · OrderDAO · PaymentDAO       │
│ ReviewDAO · InventoryDAO · TransactionDAO                    │
└───────────────────────────┬─────────────────────────────────┘
                            ▼
               ┌────────────────────────────┐
               │  WEB-INF/data/*.txt files  │
               │  (flat-file "database")    │
               └────────────────────────────┘
```

---

## 👥 User Roles
The `User` model defines three roles via a Java enum:

| Role | Description |
|---|---|
| **CUSTOMER** | Default role — browse products, manage cart, place orders, pay, and leave reviews |
| **STAFF** | Elevated access for day-to-day store operations |
| **ADMIN** | Full access to the admin dashboard — products, inventory, orders, users, reviews, transactions |

A default `ADMIN` account is automatically created the first time the app runs, via
`FirstTimeSetupServlet` (`/setup`).

---

## 🗃️ Data Model
Data is modeled as plain Java objects and persisted as delimited rows in text files
(`WEB-INF/data/`), one file per entity:

| Model | Storage File | Description |
|---|---|---|
| `User` | `users.txt` | Account credentials (salted/hashed), role, registration date |
| `Product` | `products.txt` | Name, category, price, stock, description, image path |
| `Inventory` | `inventory.txt` | Stock levels linked to products |
| `Cart` | `cart.txt` | Per-user cart contents |
| `Order` | `orders.txt` | Order line items, status, totals |
| `Payment` | `payments.txt` | Payment records and saved card references |
| `Review` | `reviews.txt` | Product reviews and ratings |
| `Transaction` | `transactions.txt` | Payment/refund transaction ledger |
| `saved_cards.txt` | — | Stored (tokenized) card details for repeat checkout |

Passwords are never stored in plain text — `SecurityUtil` hashes them with **salted
SHA-256** before writing to `users.txt`.

---

## 🔌 Servlet / URL Structure
| Servlet | URL Pattern | Responsibility |
|---|---|---|
| `HomeServlet` | `/home` | Landing page |
| `UserServlet` | `/user/*` | Registration, login, profile |
| `ProductServlet` | `/product/*`, `/admin/products` | Product browsing and admin CRUD |
| `CartServlet` | `/cart`, `/cart/*` | Cart operations |
| `InventoryServlet` | `/inventory/*` | Stock management |
| `OrderServlet` | `/order/*` | Order placement and tracking |
| `PaymentServlet` | `/payment/*` | Checkout and payment processing |
| `ReviewServlet` | `/review/*` | Product review CRUD |
| `TransactionServlet` | `/transaction/*` | Transaction history, refunds |
| `AdminDashboardServlet` | `/admin/dashboard` | Admin overview |
| `FirstTimeSetupServlet` | `/setup` | Bootstraps the initial admin account |

---

## 🧠 Technical Highlights
A few notable implementation details worth calling out:

- **Merge sort from scratch** — `ProductSortingService` and `OrderSortingService` implement
  merge sort manually (rather than relying solely on `Collections.sort`) to sort products/orders
  by name, price, date, total, or status via pluggable `Comparator`s.
- **Thread-safe order queue** — `OrderQueue` wraps a `LinkedList` with a `ReentrantLock` to
  safely enqueue/dequeue orders, and `OrderProcessingService` drains it using an
  `ExecutorService` for asynchronous order processing.
- **Custom persistence layer** — `FileHandlerUtil` centralizes all file read/write/delete
  operations, so every DAO reads and writes rows to its corresponding `.txt` file rather than
  querying a database.
- **Validation utilities** — dedicated validators (`InventoryValidationUtil`,
  `OrderValidationUtil`, `ReviewValidationUtil`, `ValidationUtil`) enforce business rules
  before data is persisted.

---

## 📦 Prerequisites
- **JDK 8** (project is compiled with `source`/`target` 1.8)
- **Apache Maven** (project uses `maven-compiler-plugin` and `maven-war-plugin`)
- **Apache Tomcat 9+** (or any Servlet 4.0 / JSP 2.3-compatible container)
- An IDE with Tomcat integration is optional (project includes IntelliJ **SmartTomcat**
  configuration under `.idea/` for one-click local runs)

---

## ⚙️ Project Setup
### 1. Clone / Extract the Project
```bash
cd grocerymanagement
```

### 2. Build with Maven
```bash
mvn clean package
```
This produces `target/grocerymanagement.war`.

### 3. Deploy to Tomcat
Copy the generated WAR file into your Tomcat `webapps/` directory:
```bash
cp target/grocerymanagement.war $CATALINA_HOME/webapps/
```
Or run it directly from your IDE using its Tomcat/SmartTomcat integration.

---

## ▶️ Running the Application
1. Start Tomcat.
2. Visit `http://localhost:8080/grocerymanagement/home`.
3. On first launch, hit `http://localhost:8080/grocerymanagement/setup` once to create the
   default admin account (`admin` / `AdminPassword123!` — **change this immediately** after
   first login).
4. Log in as admin to manage products, inventory, and orders, or register a new account to
   shop as a customer.

---

## 📁 Project Structure
```
grocerymanagement/
├── pom.xml                          # Maven build config (war packaging, Java 8)
├── src/main/java/com/grocerymanagement/
│   ├── config/                      # FileInitializationUtil — bootstraps data files
│   ├── dao/                         # File-backed DAOs (User, Product, Cart, Order, ...)
│   ├── dto/                         # PaymentDetails
│   ├── model/                       # Domain models (User, Product, Order, Payment, ...)
│   ├── service/                     # OrderProcessingService, sorting services, PaymentService
│   ├── servlet/                     # HTTP controllers (Home, Cart, User, Product, ...)
│   └── util/                        # FileHandlerUtil, SecurityUtil, validation helpers
└── src/main/webapp/
    ├── WEB-INF/
    │   ├── data/                    # Flat-file "database" (*.txt)
    │   └── web.xml                  # Servlet/JSP config, error pages, session timeout
    ├── assets/                      # css / js / images (incl. dark-theme.css)
    ├── uploads/images/              # User/admin-uploaded product images
    ├── views/
    │   ├── admin/                   # Dashboard, product/inventory/order/user management
    │   ├── cart/, order/, payment/  # Shopping flow views
    │   ├── product/, review/        # Catalog and review views
    │   ├── user/                    # Login, register, profile
    │   ├── transaction/             # Payment, refund, confirmation
    │   ├── common/                  # Shared header/footer (site + admin)
    │   └── error/                   # 404 / 500 pages
    └── index.jsp                    # Entry point
```

---

## 👨‍💻 Contributors
- **Dileepa Anushan**

---
<p align="center">Built with Java, Servlets, and JSP 🛒</p>
