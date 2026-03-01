# SQL vs NoSQL, Scaling, ACID/BASE/CAP & MongoDB

## Table of Contents

1. [SQL vs NoSQL — Deep Comparison](#1-sql-vs-nosql--deep-comparison)
2. [Data Integrity & Referential Integrity](#2-data-integrity--referential-integrity)
3. [Scaling — Vertical vs Horizontal](#3-scaling--vertical-vs-horizontal)
4. [Database Layer Architecture](#4-database-layer-architecture)
5. [ACID Transactions](#5-acid-transactions)
6. [BASE Transactions](#6-base-transactions)
7. [CAP Theorem](#7-cap-theorem)
8. [Microservices & Polyglot Persistence](#8-microservices--polyglot-persistence)
9. [MongoDB — Introduction](#9-mongodb--introduction)
10. [MongoDB Terminology vs SQL](#10-mongodb-terminology-vs-sql)
11. [MongoDB Connection URL](#11-mongodb-connection-url)
12. [MongoDB Compass](#12-mongodb-compass)
13. [Summary Tables](#13-summary-tables)
14. [Revision Checklist](#14-revision-checklist)

---

## 1. SQL vs NoSQL — Deep Comparison

### SQL (RDBMS — Relational Database Management System)

| Property             | Detail                                                             |
| -------------------- | ------------------------------------------------------------------ |
| **Schema**           | Fixed / Static — must define table structure before inserting data |
| **Data format**      | Tables — rows (records) and columns (fields)                       |
| **Scaling**          | Vertical scaling (upgrade the same machine)                        |
| **Best for**         | Structured data where relationships and data integrity matter      |
| **Transactions**     | ACID (Atomicity, Consistency, Isolation, Durability)               |
| **Rollback support** | ✅ Yes — can undo/rollback deleted or changed data                 |
| **Examples**         | MySQL, PostgreSQL, Oracle, SQLite                                  |

---

### NoSQL (Non-Relational Database)

| Property             | Detail                                                         |
| -------------------- | -------------------------------------------------------------- |
| **Schema**           | Dynamic / Flexible — no need to define schema upfront          |
| **Data format**      | Documents, Key-Value pairs, Graphs, Wide Columns               |
| **Scaling**          | Horizontal scaling (add more machines)                         |
| **Best for**         | Unstructured or semi-structured data, high-volume applications |
| **Transactions**     | BASE (Basically Available, Soft State, Eventually Consistent)  |
| **Rollback support** | ❌ No — generally does not support rollback                    |
| **Examples**         | MongoDB, Redis, Cassandra, HBase, Neo4j                        |

---

### Side-by-Side Comparison

```mermaid
graph LR
    subgraph SQL["SQL (RDBMS)"]
        s1["Fixed Schema"]
        s2["Tables: rows & cols"]
        s3["Vertical Scaling"]
        s4["ACID Transactions"]
        s5["Data Rollback ✅"]
        s6["MySQL, PostgreSQL"]
    end

    subgraph NoSQL["NoSQL"]
        n1["Dynamic Schema"]
        n2["Documents / KV / Graph"]
        n3["Horizontal Scaling"]
        n4["BASE Transactions"]
        n5["No Rollback ❌"]
        n6["MongoDB, Redis"]
    end
```

---

## 2. Data Integrity & Referential Integrity

### What is Data Integrity?

**Definition:** Data integrity means ensuring that data is **accurate, consistent, and valid** across the database — especially when data in one table **references** data in another table.

This is one of the key reasons to choose SQL over NoSQL when your data has relationships.

### Example

```
employees table:        departments table:
name  | id  | deptNo   id | dName       | location
------|-----|--------  ---|-------------|--------
"abc" | 123 | 20       10 | Engineering | Delhi
"xyz" | 456 | 80       20 | HR          | Mumbai
                        30 | Finance     | Pune
```

In this example, `deptNo 80` does not exist in the departments table. SQL databases with **foreign key constraints** would **reject** this insert — protecting data integrity.

NoSQL has no such enforcement by default — you could store `deptNo: 80` even if no matching department exists, which can lead to **orphaned/inconsistent data**.

```sql
-- SQL enforces referential integrity via foreign keys
CREATE TABLE employees (
    id     INT PRIMARY KEY,
    name   VARCHAR(100),
    deptNo INT,
    FOREIGN KEY (deptNo) REFERENCES departments(id)
    -- This prevents inserting deptNo = 80 if 80 doesn't exist in departments
);
```

---

## 3. Scaling — Vertical vs Horizontal

### What is Scaling?

**Definition:** Scaling refers to **altering the capacity or size** (RAM, storage, CPU) of a system to handle more load, more data, or more users.

```javascript
// Current system running low on resources
let myLaptop = {
  ram: 8, // GB
  storage: 512, // GB
};
// availableResource = 1 GB RAM left, 1 GB storage left
// System is struggling — what do we do?
```

There are 3 options:

---

### Option 1: Delete Existing Resources ❌

```
❌ NOT a valid solution — you can't just delete data to make space permanently
```

---

### Option 2: Vertical Scaling (Scale Up)

**Definition:** Add **more resources to the same existing machine** — upgrade its RAM, CPU, or storage.

```javascript
// Upgrade the same laptop
let myLaptopUpgraded = {
  ram: 16, // doubled RAM
  storage: 1024, // doubled storage
};
```

```mermaid
graph TD
    A["Server RAM: 8GB Storage: 512GB"] -->|"Vertical Scale (Upgrade)"| B["Same Server RAM: 16GB Storage: 1TB"]
```

**Pros:** Simple, no code changes needed, no distributed complexity
**Cons:** Has a physical limit (you can't upgrade a machine infinitely), expensive, single point of failure, **downtime** during upgrade

**Used by:** SQL databases (MySQL, PostgreSQL) primarily

---

### Option 3: Horizontal Scaling (Scale Out)

**Definition:** Add **new machines** alongside the existing ones, distributing the load across multiple servers.

```javascript
// Don't upgrade — add a new machine alongside
let existingServer = { ram: 8, storage: 512 };
let myNewLaptop = { ram: 8, storage: 512 }; // new machine added to the pool
```

```mermaid
graph TD
    A["Server 1 8GB RAM"] --- Data
    B["Server 2 8GB RAM"] --- Data
    C["Server 3 8GB RAM"] --- Data
    Data["Shared/Replicated Data Layer"]
```

**Pros:** No theoretical limit (keep adding machines), no downtime, highly available, cost-effective with commodity hardware
**Cons:** More complex to manage, requires distributed system knowledge

**Used by:** NoSQL databases (MongoDB, Cassandra, Redis) are designed for horizontal scaling

---

### Vertical vs Horizontal Scaling Summary

| Feature                     | Vertical (Scale Up)           | Horizontal (Scale Out)        |
| --------------------------- | ----------------------------- | ----------------------------- |
| **Method**                  | Upgrade existing machine      | Add new machines              |
| **Limit**                   | Physical hardware limit       | Virtually unlimited           |
| **Downtime**                | Often required                | Usually none                  |
| **Cost**                    | Expensive (high-end hardware) | Cheaper (commodity hardware)  |
| **Complexity**              | Simple                        | Complex (distributed systems) |
| **Used by**                 | SQL databases                 | NoSQL databases               |
| **Single point of failure** | ✅ Yes                        | ❌ No (distributed)           |

---

## 4. Database Layer Architecture

From the diagram: the **database layer** is a separate physical or logical tier that contains the database server(s). It runs on a computer/server with its own RAM and storage.

```
                    ┌─────────────────────────────────────────────┐
                    │              database layer                  │
                    │                                             │
  app server ──────►│  ┌──────────────────┐   ┌──────────────┐  │
                    │  │   Primary Server  │   │  Server 2    │  │
                    │  │  (ram, storage)   │   │  (ram,       │  │
                    │  │                   │   │   storage)   │  │
                    │  │  db server: 27017 │   │  (replica)   │  │
                    │  │                   │   │              │  │
                    │  │  ┌─────────────┐  │   │              │  │
                    │  │  │    data     │  │   │              │  │
                    │  │  │  d1  new    │  │   │              │  │
                    │  │  │      data   │  │   │              │  │
                    │  │  └─────────────┘  │   │              │  │
                    │  └──────────────────┘   └──────────────┘  │
                    │                                             │
                    │  write/update ──► Primary                  │
                    │  Secondary ──► Replicate (copy)            │
                    └─────────────────────────────────────────────┘
```

### Primary vs Secondary (Replica) Servers

| Server Type                    | Role                                                                                                               |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| **Primary Server**             | Handles all **write** and **update** operations. All new data goes here first. Port: `27017` (MongoDB default)     |
| **Secondary Server (Replica)** | Copies data from primary — used for **reads** and **failover**. If the primary fails, a secondary can be promoted. |

**Key rule:**

- `write / update` → always goes to the **Primary**
- `Secondary servers` → **replicate** the data from primary (eventual consistency)

This is the foundation of MongoDB's **Replica Set** — a group of MongoDB instances that maintain the same dataset, providing redundancy and high availability.

---

## 5. ACID Transactions

ACID is the set of properties that **SQL (relational) databases** guarantee for every transaction. It ensures data reliability even in case of errors, crashes, or concurrent access.

```mermaid
graph TD
    ACID --> A["Atomicity (All or Nothing)"]
    ACID --> C["Consistency (Valid State Always)"]
    ACID --> I["Isolation (Concurrent Transactions don't interfere)"]
    ACID --> D["Durability (Persisted after commit)"]
```

---

### A — Atomicity

**Definition:** A transaction is treated as a **single unit** — either **all operations succeed**, or **none of them do**. There are no checkpoints or partial commits.

**Real-world example — Bank Transfer:**

```
Transfer ₹1000 from Account A to Account B:
  Step 1: Deduct ₹1000 from Account A
  Step 2: Add ₹1000 to Account B

If Step 2 fails (network crash, etc.):
  Atomicity ROLLS BACK Step 1 as well
  Result: Neither account is changed (data integrity preserved)
```

```sql
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 1000 WHERE id = 'A';
    UPDATE accounts SET balance = balance + 1000 WHERE id = 'B';
COMMIT; -- only commits if BOTH succeed
-- If anything fails → automatic ROLLBACK
```

---

### C — Consistency

**Definition:** A transaction brings the database from one **valid state** to another valid state. It cannot violate defined rules (constraints, foreign keys, data types).

```
Before transaction: Total money = ₹10,000
After transaction:  Total money = ₹10,000 (unchanged — just moved between accounts)
Consistency ensures: Money was never "created" or "lost" during the transfer
```

---

### I — Isolation

**Definition:** Transactions executing **concurrently** do not interfere with each other. Each transaction sees a consistent snapshot of the data.

```
User A reads balance: ₹5000
User B simultaneously transfers ₹2000 out
User A should NOT see the partially-updated ₹3000 mid-transaction
→ Isolation ensures User A sees either the old state (₹5000) or new state (₹3000), never "in-between"
```

---

### D — Durability

**Definition:** Once a transaction is **committed**, it is **permanently saved** (persisted to disk) — even if the system crashes immediately after.

```
After COMMIT is executed:
  → Data is written to disk
  → Even a power failure won't lose the committed data
  → This is why bank transactions are "durable"
```

> **Use case for ACID:** Any financial application — bank transfers, payment systems, stock trading — where partial updates would be catastrophic.

---

## 6. BASE Transactions

BASE is the transaction model used by most **NoSQL databases**. It is a relaxed alternative to ACID, optimized for performance and availability at scale.

```mermaid
graph TD
    BASE --> BA["Basically Available (System always responds, even if data is stale)"]
    BASE --> SS["Soft State (Data can change without user input)"]
    BASE --> EC["Eventually Consistent (All nodes will be in sync eventually)"]
```

---

### BA — Basically Available

The system **always responds** to requests — even if the response contains stale or partial data. The system prioritizes availability over perfect consistency.

```
Instagram likes counter:
  You see "1,024 likes" — the real count might be 1,026
  But the app responds immediately rather than waiting for perfect sync
```

---

### S — Soft State

The state of the system **may change over time even without user input** — due to background replication processes syncing data across nodes.

```
Your Instagram post like count changes from 1,024 → 1,026
even though YOU didn't do anything — background replication updated it
```

---

### E — Eventually Consistent

All nodes in the distributed system will **eventually reach the same state** — it just might not be immediate.

```
You post a tweet → visible in US in 50ms, visible in India in 200ms
There's a brief window where different users see different states
Eventually → everyone sees the same tweet
```

> **Use case for BASE:** Social media feeds, product catalogs, analytics dashboards — where slightly stale data is acceptable and availability/speed matter more than perfect accuracy.

---

## 7. CAP Theorem

### What is CAP Theorem?

**Definition:** CAP Theorem states that in a **distributed database system**, you can only **guarantee 2 out of 3** properties simultaneously:

```mermaid
graph TD
    CAP["CAP Theorem (Pick 2 of 3)"]
    CAP --> C["C — Consistency Every read gets the most recent write"]
    CAP --> A["A — Availability Every request gets a response (no timeout)"]
    CAP --> P["P — Partition Tolerance System works even if network partitions occur (MUST always be chosen)"]
```

> **Critical rule:** In a real distributed system, **network partitions WILL happen** (servers lose connection). So **P (Partition Tolerance) is always required**. This means you actually choose between:
>
> - **CP** — Consistent + Partition Tolerant (sacrifice Availability)
> - **AP** — Available + Partition Tolerant (sacrifice Consistency)

---

### Choosing CP vs AP

| System Type          | CAP Choice                               | Why                                                 | Example            |
| -------------------- | ---------------------------------------- | --------------------------------------------------- | ------------------ |
| **Banking app**      | **CP** (Consistent + Partition Tolerant) | Cannot show wrong balance — consistency is critical | MySQL, PostgreSQL  |
| **Social media app** | **AP** (Available + Partition Tolerant)  | Better to show slightly stale posts than no posts   | MongoDB, Cassandra |

```
Bank App (CP):
  If two servers can't sync → system returns ERROR
  Reason: "I'd rather show no data than show wrong balance"

Instagram (AP):
  If two servers can't sync → system returns stale data
  Reason: "I'd rather show yesterday's like count than show nothing"
```

---

## 8. Microservices & Polyglot Persistence

### Microservices Architecture

**Definition:** Instead of building one large application (monolith), microservices break an application into **small, independent services** — each responsible for one specific function and communicating with others via APIs.

From the diagram: the crossed-out "fully connected mesh" shows that NOT every microservice talks directly to every other — they communicate in a **directed, controlled flow**.

```mermaid
graph LR
    A["Auth Service"] --> B["User Service"]
    B --> C["Order Service"]
    C --> D["Payment Service"]
```

Each service can use a **different technology, language, or database** — this is called **Polyglot Persistence**.

---

### Polyglot Persistence

**Definition:** Using **different types of databases** for different parts of the same application — each service picks the database best suited for its needs.

```
Netflix example:
  User profiles        → SQL (PostgreSQL) — relational, structured
  Video metadata       → Cassandra (NoSQL) — massive scale, wide column
  Search               → Elasticsearch
  Session/caching      → Redis (key-value)
  Recommendations      → Graph DB (Neo4j)
```

> **Real-world note:** Netflix famously uses CassandraDB for its massive-scale streaming data AND SQL for more structured data — this is **polyglot persistence** in action.

**Polyglot Applications:** Applications written in multiple programming languages (Node.js + Java + Python) — each microservice can use the language best suited for its task.

---

## 9. MongoDB — Introduction

**Definition:** MongoDB is a **document-based NoSQL database** that stores data in a **JSON-like format called BSON** (Binary JSON).

Key characteristics:

- Schema-less / flexible schema
- Documents are stored in **collections** (not tables)
- Each document is a set of **key-value pairs** (like a JavaScript object)
- Designed for horizontal scaling
- Port: **27017** (default)

```javascript
// MongoDB stores data like this — each document can have different fields
{ "_id": ObjectId("..."), "name": "Alice",  "age": 30, "email": "alice@gmail.com" }
{ "_id": ObjectId("..."), "name": "Bob",    "age": 25 }                             // no email field — ok!
{ "_id": ObjectId("..."), "name": "Charlie","age": 28, "salary": 50000, "city": "Mumbai" } // extra fields — ok!
```

---

## 10. MongoDB Terminology vs SQL

One of the most commonly tested comparisons in interviews:

| SQL Term         | MongoDB Equivalent | Description                                   |
| ---------------- | ------------------ | --------------------------------------------- |
| **Database**     | **Database**       | Top-level container — same name               |
| **Table**        | **Collection**     | Group of related records                      |
| **Column**       | **Field / Key**    | Individual data attribute                     |
| **Row / Record** | **Document `{}`**  | A single entry                                |
| **Primary Key**  | **`_id`**          | Unique identifier (auto-generated by MongoDB) |

```
SQL:
  database (myApp)
    └── table (users)
          ├── row: { id: 1, name: "Alice", age: 30 }
          └── row: { id: 2, name: "Bob",   age: 25 }

MongoDB:
  database (myApp)
    └── collection (users)
          ├── document: { _id: ObjectId("..."), name: "Alice", age: 30 }
          └── document: { _id: ObjectId("..."), name: "Bob",   age: 25 }
```

```mermaid
graph TD
    SQL["SQL"] --> sDB["Database"]
    sDB --> sT["Table"]
    sT --> sC["Columns (schema fixed)"]
    sT --> sR["Rows (records)"]

    Mongo["MongoDB"] --> mDB["Database"]
    mDB --> mC["Collection"]
    mC --> mF["Fields (flexible)"]
    mC --> mD["Documents {}"]
```

---

## 11. MongoDB Connection URL

When connecting to a MongoDB instance (local or cloud), you use a **connection URL** with a specific format:

```
mongodb://localhost:27017
```

Breaking it down:

| Part              | Value        | Meaning                                                                          |
| ----------------- | ------------ | -------------------------------------------------------------------------------- |
| **Protocol**      | `mongodb://` | Tells the driver to use the MongoDB protocol (like `https://` for web)           |
| **Domain / Host** | `localhost`  | The machine where MongoDB is running (`localhost` = your own computer)           |
| **Port**          | `27017`      | The port MongoDB listens on (default port — like `80` for HTTP, `443` for HTTPS) |

```javascript
// Connecting to local MongoDB (Mongoose example)
const mongoose = require("mongoose");

mongoose
  .connect("mongodb://localhost:27017/myDatabase")
  .then(() => console.log("Connected to MongoDB"))
  .catch((err) => console.error("Connection failed:", err));

// For MongoDB Atlas (cloud):
// mongodb+srv://username:password@cluster.mongodb.net/myDatabase
```

---

## 12. MongoDB Compass

**Definition:** MongoDB Compass is a **GUI (Graphical User Interface)** tool — an official desktop application provided by MongoDB — that allows you to interact with your MongoDB database **visually without writing any code**.

### What you can do in Compass

- View all databases and collections
- Browse, filter, and sort documents visually
- Insert, update, and delete documents via forms
- Run queries and aggregation pipelines visually
- Analyze schema and data distribution
- Monitor database performance

```
Without Compass: must write queries in terminal/code
  > db.users.find({ age: { $gt: 25 } })

With Compass: click a filter button, fill in a form — same result, no code needed
```

> **Analogy:** MongoDB Compass is like **phpMyAdmin or MySQL Workbench** — but for MongoDB. It's a visual dashboard for your database.

---

## 13. Summary Tables

### SQL vs NoSQL Quick Reference

|                | SQL                        | NoSQL                          |
| -------------- | -------------------------- | ------------------------------ |
| Schema         | Fixed                      | Dynamic                        |
| Structure      | Tables                     | Documents / KV / Graph / Wide  |
| Scaling        | Vertical                   | Horizontal                     |
| Transactions   | ACID                       | BASE                           |
| Rollback       | ✅                         | ❌                             |
| Data integrity | ✅ Strong                  | Manual                         |
| Best for       | Relational, financial data | Unstructured, large-scale data |

### ACID vs BASE

|             | ACID                                          | BASE                                                   |
| ----------- | --------------------------------------------- | ------------------------------------------------------ |
| Stands for  | Atomicity, Consistency, Isolation, Durability | Basically Available, Soft State, Eventually Consistent |
| Used by     | SQL (relational)                              | NoSQL                                                  |
| Priority    | Data correctness                              | Availability & performance                             |
| Example app | Banking                                       | Social media                                           |

### MongoDB vs SQL Terminology

| SQL         | MongoDB    |
| ----------- | ---------- |
| Database    | Database   |
| Table       | Collection |
| Row         | Document   |
| Column      | Field      |
| Primary Key | `_id`      |

---

## 14. Revision Checklist

### SQL vs NoSQL

- [ ] Can you list 4 differences between SQL and NoSQL?
- [ ] Can you explain what a fixed schema means vs a dynamic schema?
- [ ] Can you explain data integrity and give the employee/department example?
- [ ] Can you name 2 SQL and 4 NoSQL databases from memory?

### Scaling

- [ ] Can you define what scaling means?
- [ ] Can you explain vertical scaling with an example?
- [ ] Can you explain horizontal scaling with an example?
- [ ] Can you list pros and cons of each scaling type?
- [ ] Do you know which DB type uses which scaling (SQL=vertical, NoSQL=horizontal)?

### Database Layer & Replication

- [ ] Can you explain what the database layer is in 3-tier architecture?
- [ ] Can you explain primary vs secondary (replica) servers?
- [ ] Do you know that writes go to primary and secondaries replicate?

### ACID

- [ ] Can you expand ACID (Atomicity, Consistency, Isolation, Durability)?
- [ ] Can you explain Atomicity with the bank transfer example?
- [ ] Can you explain why Durability matters?
- [ ] Can you name a use case where ACID is required (bank app)?

### BASE

- [ ] Can you expand BASE (Basically Available, Soft State, Eventually Consistent)?
- [ ] Can you explain Soft State with an example (Instagram likes)?
- [ ] Can you explain Eventually Consistent with an example (tweet visibility)?
- [ ] Can you name a use case where BASE is acceptable (social media)?

### CAP Theorem

- [ ] Can you explain the 3 letters of CAP (Consistency, Availability, Partition Tolerance)?
- [ ] Do you know that Partition Tolerance is always required?
- [ ] Can you explain CP vs AP and give an example of each?

### MongoDB

- [ ] Can you define MongoDB in one sentence?
- [ ] Can you explain the MongoDB hierarchy: database → collection → document → key-value?
- [ ] Can you map SQL terms to MongoDB terms (table→collection, row→document, etc.)?
- [ ] Can you break down the MongoDB connection URL `mongodb://localhost:27017`?
- [ ] Can you explain what MongoDB Compass is and why it's useful?
- [ ] Do you know MongoDB's default port (27017)?

---

> **Interview tip:** When asked "When would you choose SQL over NoSQL?", say: _"I'd choose SQL when data has clear relationships and integrity constraints — like financial applications — where ACID transactions and referential integrity are non-negotiable. I'd choose NoSQL when the data is unstructured, schema changes frequently, or the system needs to scale horizontally to handle massive amounts of data — like a social media feed or recommendation engine."_
