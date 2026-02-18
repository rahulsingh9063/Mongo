# MongoDB — Shell, Commands, BSON & ObjectId

### Complete Technical Reference — Interview Prep & Long-Term Revision

---

## 📚 What You'll Learn

This guide covers **everything about MongoDB's shell, commands, and how data is stored**. After reading this, you'll understand:

✅ How to use MongoDB shell (mongosh) to run commands  
✅ How to create, read, update, and delete databases and collections  
✅ What BSON is and why MongoDB uses it instead of regular JSON  
✅ What ObjectId is and how MongoDB generates unique IDs  
✅ How to manage the MongoDB server (start/stop)

**Best for:** Beginners to intermediate developers, interview preparation, quick reference

---

## 🎯 Quick Start Guide

**Never used MongoDB before?** Start here:

1. **What is MongoDB?** → A database that stores data as JSON-like documents (flexible structure)
2. **What is mongosh?** → A command-line tool to talk to MongoDB (like a terminal for your database)
3. **What is BSON?** → Binary format MongoDB uses internally (faster than text JSON)
4. **What is ObjectId?** → The unique ID MongoDB creates for every document automatically

**Already familiar?** Jump to any section using the table of contents below.

---

## 🛣️ Suggested Learning Path

**If you're new to MongoDB, read in this order:**

```
1. Start here → What is mongosh? (Section 2)
2. Then learn → How to start/stop MongoDB server (Section 4)
3. Next → Basic commands (Section 6 & 7)
4. Finally → Understand BSON and ObjectId (Section 9 & 10)
```

**If you're preparing for interviews:**

- Read Section 9 (BSON) and Section 10 (ObjectId) carefully
- Memorize the command reference tables (Section 11)
- Practice the revision checklist (Section 13)

**If you just need a quick command reference:**

- Jump straight to Section 11 (Complete Command Reference)

---

## Table of Contents

1. [MongoDB Architecture — Driver & WiredTiger](#1-mongodb-architecture--driver--wiredtiger)
2. [MongoDB Shell (mongosh)](#2-mongodb-shell-mongosh)
3. [GUI vs CLI](#3-gui-vs-cli)
4. [MongoDB Server Management](#4-mongodb-server-management)
5. [MongoDB Connection URLs](#5-mongodb-connection-urls)
6. [Database-Level Commands](#6-database-level-commands)
7. [Collection-Level Commands](#7-collection-level-commands)
8. [CRUD Operations on Documents](#8-crud-operations-on-documents)
9. [BSON — Binary JSON](#9-bson--binary-json)
10. [ObjectId — Deep Dive](#10-objectid--deep-dive)
11. [Complete Command Reference](#11-complete-command-reference)
12. [Summary](#12-summary)
13. [Revision Checklist](#13-revision-checklist)

---

## 1. MongoDB Architecture — Driver & WiredTiger

### The Complete Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Database Layer                             │
│                                                                 │
│  ┌──────────────────────┐        ┌─────────────────────────┐  │
│  │  MongoDB Driver      │        │     WiredTiger          │  │
│  │                      │        │                         │  │
│  │  interface/shell     │───────►│  RAM                    │  │
│  │  → mongo shell       │        │                         │  │
│  │                      │◄───────│  MongoDB Storage Engine │  │
│  │  use library         │        │                         │  │
│  │  find()              │        │  1) Converts JSON/JS    │  │
│  │                      │        │     object to BSON      │  │
│  └──────────────────────┘        │                         │  │
│                                   │  2) Allocates disk      │  │
│  db.gameNames.insertOne({...})   │     space for new data  │  │
│  db.gameNames.insertOne({...})   │                         │  │
│                                   │  3) Fetches data from   │  │
│                                   │     disk (BSON → JSON)  │  │
│                                   └────────┬────────────────┘  │
│                                            │                   │
│                                            ▼                   │
│                                   ┌─────────────────────────┐  │
│                                   │   Disk Storage          │  │
│                                   │                         │  │
│                                   │   data (BSON format)    │  │
│                                   │   newData (BSON)        │  │
│                                   │                         │  │
│                                   └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

### MongoDB Driver

**Simple explanation:** Think of the MongoDB driver as a **translator** between you (speaking JavaScript/commands) and the database (speaking BSON/binary).

Just like you need a translator to speak to someone who doesn't speak your language, you need a driver to "talk" to MongoDB.

**What it does:**

1. Takes your commands (like "find all books from 2020")
2. Translates them into something MongoDB understands
3. Sends them to the database
4. Gets the results back
5. Translates the results back into JavaScript objects you can use

**Real-world example:**

```javascript
// You type this (JavaScript):
db.books.find({ year: 2020 });

// The driver translates it to MongoDB's internal format
// MongoDB processes it
// The driver translates the results back to JavaScript
// You get: [ { name: "Book1", year: 2020 }, { name: "Book2", year: 2020 } ]
```

**Where you use it:**

- **In mongosh (shell)** — When you type commands directly
- **In your Node.js code** — When you use packages like `mongoose` or `mongodb`

---

### WiredTiger Storage Engine

**Simple explanation:** WiredTiger is like a **librarian** who manages the actual storage of books. When you ask for a book, the librarian:

1. Finds it on the shelf (disk)
2. Brings it to you in a format you can read (JSON)
3. When you return a book, the librarian stores it back efficiently (compressed BSON)

**What WiredTiger does (step by step):**

| Step                  | What happens                                                                                         | Why                                           |
| --------------------- | ---------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **1. Saving data**    | You give it a JavaScript object → WiredTiger converts it to BSON (compressed binary) → Saves to disk | Binary is smaller and faster to work with     |
| **2. Reading data**   | You ask for data → WiredTiger reads BSON from disk → Converts back to JavaScript object → You get it | You need JavaScript objects, not binary       |
| **3. Memory caching** | Frequently used data stays in RAM                                                                    | Reading from RAM is 100,000x faster than disk |
| **4. Compression**    | Data is squeezed to take less space on disk                                                          | A 100MB file might become 30MB                |

**Think of it this way:**

```
Your data journey:

You write:                    WiredTiger stores:           You read back:
{ name: "Alice" }    →        [binary: 01001010...]   →    { name: "Alice" }
(JavaScript object)           (BSON on disk)               (JavaScript object)
```

**Why this matters:**

- **Speed:** Binary format = faster processing
- **Size:** Compression = less disk space used
- **Features:** BSON supports things JSON can't (like dates and special IDs)

---

### Flow Summary

```
1. User writes: db.books.insertOne({ name: "NFS", year: 2001, rating: 4.73 })
   ↓
2. MongoDB Driver receives the JavaScript object
   ↓
3. WiredTiger converts it to BSON (Binary JSON)
   ↓
4. BSON is written to disk storage
   ↓
5. When reading, WiredTiger fetches BSON from disk
   ↓
6. WiredTiger converts BSON back to JSON
   ↓
7. User receives the data as a JavaScript object
```

---

## 2. MongoDB Shell (mongosh)

### What is mongosh?

**Simple explanation:** `mongosh` is like the **control panel** for your MongoDB database. Just like you use a terminal/command prompt to control your computer, you use mongosh to control MongoDB.

```
Think of it as:
Your Computer's Terminal  →  Controls your computer's files
MongoDB Shell (mongosh)   →  Controls your database
```

---

### How to Enter mongosh

**Step-by-step:**

```bash
1. Open your terminal/command prompt
2. Type: mongosh
3. Press Enter
4. You'll see:

Current Mongosh Log ID: ...
Connecting to: mongodb://127.0.0.1:27017
Using MongoDB: 7.0.0
Using Mongosh: 2.0.0

test>   ← You're in! This is your prompt
```

> 💡 **Remember:** The `test>` prompt means you're connected to the "test" database (MongoDB's default database).

---

### How to Exit mongosh

**Any of these work:**

```bash
# Method 1: Type exit
test> exit

# Method 2: Press Ctrl + C (twice)
^C
^C

# Method 3: Type quit
test> quit
```

---

### 📝 **Remember This:**

```
mongosh = Your database terminal
test>   = You're connected and ready to type commands
exit    = Leave mongosh
```

---

## 3. GUI vs CLI

| Feature                | GUI (Compass)                       | CLI (mongosh)                                |
| ---------------------- | ----------------------------------- | -------------------------------------------- |
| **Full name**          | Graphical User Interface            | Command Line Interface                       |
| **Commands required?** | ❌ No — click buttons, fill forms   | ✅ Yes — type commands                       |
| **Ease of use**        | Easier for beginners                | Harder initially, faster once learned        |
| **Functionality**      | Limited to what's built into the UI | Full access to all MongoDB features          |
| **Speed**              | Slower (lots of clicking)           | Faster (type commands directly)              |
| **Automation**         | Cannot script or automate           | Can write scripts, automate tasks            |
| **Use case**           | Quick exploration, visual browsing  | Production work, scripting, advanced queries |

```
GUI (Compass):
  Click "Databases" → Click "library" → Click "books" → Click "Insert Document" → Fill form → Click "Insert"

CLI (mongosh):
  db.books.insertOne({ name: "Book Title", year: 2023 })
  (Done in one line)
```

> **Real-world:** Developers use **Compass for exploration** and **mongosh for actual work** (scripting, automation, production queries).

---

## 4. MongoDB Server Management

### Starting and Stopping MongoDB Server

> **Important:** MongoDB must be **running as a service** for mongosh or Compass to connect to it.

```bash
# Check MongoDB server status (Windows)
sc query mongodb

# Output shows:
# STATE: 4 RUNNING   ← Server is running
# STATE: 1 STOPPED   ← Server is stopped
```

---

### Windows — Start/Stop Commands

```bash
# Open Command Prompt AS ADMINISTRATOR (right-click → Run as administrator)

# Stop MongoDB server
net stop mongodb

# Start MongoDB server
net start mongodb
```

---

### macOS / Linux — Start/Stop Commands

```bash
# Start MongoDB server
sudo systemctl start mongod
# or (older systems)
sudo service mongod start

# Stop MongoDB server
sudo systemctl stop mongod
# or
sudo service mongod stop

# Check status
sudo systemctl status mongod
```

---

## 5. MongoDB Connection URLs

### Structure of a MongoDB URL

```
mongodb://host:port
```

| Part         | Example                    | Meaning                     |
| ------------ | -------------------------- | --------------------------- |
| **Protocol** | `mongodb://`               | MongoDB connection protocol |
| **Host**     | `localhost` or `127.0.0.1` | Where MongoDB is running    |
| **Port**     | `27017`                    | Default MongoDB port        |

---

### localhost vs 127.0.0.1

```javascript
// mongosh (shell)
mongodb://127.0.0.1:27017
// Uses IP address directly

// MongoDB Compass (GUI)
mongodb://localhost:27017
// Uses domain name (resolves to 127.0.0.1)
```

**What's the difference?**

| Term        | Type            | Meaning                                                |
| ----------- | --------------- | ------------------------------------------------------ |
| `127.0.0.1` | **IP Address**  | The numeric address — always refers to "this computer" |
| `localhost` | **Domain Name** | A human-readable name that **resolves to** 127.0.0.1   |

> **Note:** They both mean the **same thing** — your local machine. `localhost` is just more human-friendly.

---

## 6. Database-Level Commands

### Before You Start

> 💡 **Default database:** When you open mongosh, you're automatically in the `test` database.
>
> ```bash
> test>  ← This means you're in the "test" database
> ```

---

### 6.1 Show All Databases

**What it does:** Lists all databases that actually exist (have data in them)

```javascript
show dbs
// or
show databases

// Example output:
admin    40.00 KiB  ← System database
config   12.00 KiB  ← System database
local    40.00 KiB  ← System database
library  8.00 KiB   ← Your database (if you created one)
```

> ⚠️ **Note:** An empty database won't appear here! It only shows up after you add data.

---

### 6.2 Create or Switch to a Database

**What it does:** Creates a new database OR switches to an existing one

```javascript
use database-name

// Example 1: Switch to "library" (if it exists)
use library
// Output: switched to db library

// Example 2: Create a new database called "school"
use school
// Output: switched to db school

// ⚠️ BUT... check this:
show dbs  // "school" is NOT listed yet!
```

**Why isn't "school" listed?** Because it's empty! MongoDB doesn't save empty databases to disk.

**To make it permanent:**

```javascript
use school
db.createCollection("students")  // Now create a collection
show dbs  // NOW "school" appears in the list!
```

> 💡 **Remember:** `use` = create OR switch (depending on if it exists)

---

### 6.3 Create a Collection

**What it does:** Creates a new collection (like a table in SQL) inside the current database

```javascript
db.createCollection("collection-name")

// Example:
db.createCollection("books")

// Output:
{ ok: 1 }  ← Success!
```

**Visual guide:**

```
Before:                After:
library (database)     library (database)
  (empty)                ├── books (collection) ✅
```

> 💡 **Tip:** After running commands in mongosh, refresh MongoDB Compass to see the changes visually!

---

### 6.4 Show All Collections

**What it does:** Lists all collections in the current database

```javascript
show collections
// or
show tables

// Example output:
books
authors
reviews
```

---

### 6.5 Delete a Collection

**What it does:** Permanently removes a collection and ALL its documents

```javascript
db.collectionName.drop()

// Example:
db.books.drop()

// Output:
true  ← Collection deleted!
```

> ⚠️ **Warning:** This deletes ALL data in that collection. There's no undo!

---

### 6.6 Rename a Collection

**What it does:** Changes a collection's name

```javascript
db.oldName.renameCollection("newName");

// Example:
db.books.renameCollection("novels");

// Output:
{
  ok: 1;
}

// Before: collection was called "books"
// After:  collection is called "novels"
```

---

### 6.7 Delete a Database

**What it does:** Permanently deletes the ENTIRE database and all its collections

```javascript
// Step 1: Switch to the database you want to delete
use library

// Step 2: Delete it
db.dropDatabase()

// Output:
{ ok: 1, dropped: 'library' }
```

> ⚠️ **DANGER:** This is permanent! The entire database and all its data will be gone forever.

---

### 6.8 Can You Rename a Database? ❌ NO

**Important:** MongoDB does NOT have a rename command for databases.

**If you need to rename a database, you must:**

1. Create a new database with the new name
2. Copy all collections to the new database (manually or with a script)
3. Delete the old database

```javascript
// ❌ This does NOT exist:
db.renameDatabase("newName"); // Not a real command!

// ✅ You must do this instead:
// 1. use newDatabaseName
// 2. Copy all collections manually
// 3. use oldDatabaseName
// 4. db.dropDatabase()
```

---

### 🎯 Common Mistakes to Avoid

| Mistake                                           | Problem                                | Solution                                          |
| ------------------------------------------------- | -------------------------------------- | ------------------------------------------------- |
| Create database but it doesn't show in `show dbs` | Empty databases aren't saved           | Add at least one collection or document           |
| Try to delete a collection from wrong database    | You're in the wrong database           | Use `use dbName` first, then `db.coll.drop()`     |
| Think `use` only creates databases                | It also switches between existing ones | `use` does both: create new OR switch to existing |
| Expect `db.renameDatabase()` to work              | This command doesn't exist             | Must manually copy to new DB and delete old one   |

---

## 7. Collection-Level Commands

### Summary Table

| Task                 | Command                          | Example                                 |
| -------------------- | -------------------------------- | --------------------------------------- |
| Create collection    | `db.createCollection("name")`    | `db.createCollection("users")`          |
| Show all collections | `show collections`               | `show tables`                           |
| Drop collection      | `db.collection.drop()`           | `db.users.drop()`                       |
| Rename collection    | `db.old.renameCollection("new")` | `db.users.renameCollection("accounts")` |

---

## 8. CRUD Operations on Documents

### 8.1 Inserting a Single Document

```javascript
db.collectionName.insertOne({ key: value, ... })

// Example:
db.gameNames.insertOne({
    name: "Call of Duty",
    year: 1996,
    rating: 4.73
})

// Output:
{
  acknowledged: true,
  insertedId: ObjectId("69959910aba5099b6c73518a")
}
```

```javascript
// Another example:
db.gameNames.insertOne({
    name: "NFS",
    year: 2001,
    rating: 4.73
})

// Output:
{
  acknowledged: true,
  insertedId: ObjectId("69959910aba5099b6c73518b")
}
```

> **Note:** MongoDB automatically adds an `_id` field with a unique ObjectId if you don't provide one.

---

### 8.2 Finding Documents

```javascript
// Find all documents
db.collectionName.find();

// Find with a filter
db.gameNames.find({ year: 2001 });

// Find one document
db.gameNames.findOne({ name: "NFS" });
```

---

## 9. BSON — Binary JSON

### What is BSON? (Simple Explanation)

**BSON = Binary JSON**

Think of it like this:

- **JSON** is like a handwritten letter → humans can read it easily
- **BSON** is like a ZIP file → computers can process it faster, but humans can't read it

**Real-world example:**

```javascript
// This is JSON (what you write):
{ "name": "Alice", "age": 30 }

// MongoDB converts it to BSON (what gets stored):
[01001010 11010010 ...] ← Binary code (not readable by humans)

// When you read it back, MongoDB converts it back to JSON:
{ "name": "Alice", "age": 30 }
```

**Why MongoDB does this:**

- Computers work with binary (0s and 1s) naturally — it's their native language
- Binary is faster to read/write than text
- Binary takes less space (like how a ZIP file is smaller)

---

### JSON vs BSON — The Key Differences

| Aspect                   | JSON                                                   | BSON                                          | Winner               |
| ------------------------ | ------------------------------------------------------ | --------------------------------------------- | -------------------- |
| **Can humans read it?**  | ✅ Yes — it's text                                     | ❌ No — it's binary                           | JSON (for humans)    |
| **Which is faster?**     | Slow — needs parsing                                   | Fast — direct binary                          | BSON (for computers) |
| **Which is smaller?**    | Bigger (text takes more space)                         | Smaller (compressed)                          | BSON                 |
| **Supported data types** | 6 types (string, number, boolean, null, array, object) | 15+ types (includes dates, special IDs, etc.) | BSON                 |

---

### Extra Data Types in BSON (Not in JSON)

MongoDB (BSON) can store things that regular JSON cannot:

```javascript
{
  // ✅ All these work in MongoDB:
  _id: ObjectId("507f1f77bcf86cd799439011"),  // Special unique ID
  name: "Alice",
  birthdate: ISODate("1994-01-15"),           // Actual date object
  profilePic: BinData(0, "..."),              // Binary data (for images)
  searchPattern: /abc/i,                      // Regular expressions
  bonus: undefined,                           // Undefined values

  // ❌ But in pure JSON, you'd have to convert these to strings:
  // "birthdate": "1994-01-15"  ← Just text, not a real date
}
```

**Why this matters:**

- You can store actual **dates** (not just text that looks like a date)
- You can store **images and files** directly
- You can use **undefined** (which JSON doesn't allow)

---

### The Magic Conversion (Automatic)

**You don't have to do anything!** MongoDB handles this automatically:

```
When you INSERT:
Your JS object → MongoDB converts to BSON → Saves to disk
{ name: "Alice" } → [binary] → 💾

When you READ:
Disk → MongoDB converts BSON to JS object → You get it back
💾 → [binary] → { name: "Alice" }
```

**You always work with JavaScript objects.** The BSON conversion happens behind the scenes.

---

## 10. ObjectId — Deep Dive

### What is ObjectId? (Simple Explanation)

**ObjectId is like a super-smart ID card** that MongoDB automatically creates for every document.

Think of it as:

- A **unique employee ID number** — no two people can have the same one
- It contains **hidden information** — like when the person was hired
- It's **automatically generated** — you don't have to think about it

```javascript
_id: ObjectId("69959910aba5099b6c73518a")
              └────────┬────────────────┘
                    24 characters
                  (looks random, but it's not!)
```

---

### Breaking It Down — The 3 Secret Parts

ObjectId looks random, but it actually has 3 pieces of information hidden inside:

```
ObjectId("69959910aba5099b6c73518a")
         └──┬───┘└────┬────┘└──┬──┘
         Part 1    Part 2   Part 3
        WHEN?     WHERE?    COUNT
```

| Part                  | Name                        | What it stores                                         | Example                                         |
| --------------------- | --------------------------- | ------------------------------------------------------ | ----------------------------------------------- |
| **Part 1** (8 chars)  | **Timestamp**               | WHEN was this document created?                        | `69959910` → Feb 15, 2025 at 10:30 AM           |
| **Part 2** (10 chars) | **PUI** (Process Unique ID) | WHERE was it created? (which computer + which process) | `aba5099b6c` → Computer #123, Process #456      |
| **Part 3** (6 chars)  | **Counter**                 | COUNT - which number document is this?                 | `73518a` → This is document #7,500,000 (approx) |

---

### Visual Example with Real Meaning

```javascript
_id: ObjectId("69959910aba5099b6c73518a")

Let's decode this:

"69959910"     → Created on: Feb 15, 2025 at 10:30:16 AM
"aba5099b6c"   → On computer: Server-A, Process: node-12345
"73518a"       → Document number: 7,500,000 (increments by 1 each time)

Translation: "This is the 7,500,000th document created by
             node-12345 on Server-A at 10:30 AM on Feb 15, 2025"
```

---

### Why ObjectId is Awesome

| Feature                    | Benefit                                       | Real-world example                                                       |
| -------------------------- | --------------------------------------------- | ------------------------------------------------------------------------ |
| **Globally unique**        | No duplicates, ever                           | Like a fingerprint — unique across the entire world                      |
| **Sortable by time**       | Older documents = older IDs                   | Sort by `_id` = sort by creation time (no need for separate timestamp)   |
| **No coordination needed** | Multiple servers can create IDs independently | 1000 servers can all create documents at the same time without conflicts |
| **Small size**             | Only 12 bytes (vs UUID's 16 bytes)            | Saves space when you have millions of documents                          |

---

### MongoDB Creates It Automatically

**You don't have to do anything!**

```javascript
// You insert this:
db.users.insertOne({ name: "Alice", age: 30 })

// MongoDB automatically adds _id:
{
  _id: ObjectId("69959910aba5099b6c73518a"),  ← Added automatically!
  name: "Alice",
  age: 30
}
```

---

### Can You Make Your Own `_id`? (Yes!)

Sometimes you want to use your own ID instead of MongoDB's:

```javascript
// Example 1: Using a number as _id
db.users.insertOne({ _id: 123, name: "Bob" });
// Result: { _id: 123, name: "Bob" }  ← Uses your ID

// Example 2: Using a string as _id
db.users.insertOne({ _id: "user-alice-2025", name: "Alice" });
// Result: { _id: "user-alice-2025", name: "Alice" }  ← Uses your ID

// Example 3: Using an email as _id (common pattern)
db.users.insertOne({ _id: "alice@example.com", name: "Alice" });
// Result: { _id: "alice@example.com", name: "Alice" }
```

**⚠️ Warning:** If you use your own `_id`, **you must ensure it's unique**. If you try to insert a duplicate, MongoDB will throw an error.

---

### ObjectId is Locked Forever (Immutable)

Once created, you **cannot change** the `_id` — it's permanent!

```javascript
// ❌ This will NOT work:
db.users.updateOne({ name: "Alice" }, { $set: { _id: "new-id-123" } });
// Error: Cannot update _id field!

// ✅ If you need a different _id, you must:
// 1. Create a NEW document with the new _id
// 2. Delete the old document
```

**Why is this?** Because `_id` is the **primary key** — it's how MongoDB finds your document. If you could change it, MongoDB would lose track of your data!

---

### Cool Trick: Extract the Creation Time

Since the timestamp is built into ObjectId, you can find out **when a document was created** without storing a separate date!

```javascript
// Get the creation time from any ObjectId:
const doc = db.users.findOne({ name: "Alice" });
const createdAt = doc._id.getTimestamp();

console.log(createdAt);
// Output: 2025-02-15T10:30:16.000Z

// This means you can sort by creation time without a "createdAt" field:
db.users.find().sort({ _id: 1 }); // Oldest first
db.users.find().sort({ _id: -1 }); // Newest first
```

---

### Quick Summary

| Question                     | Answer                                                          |
| ---------------------------- | --------------------------------------------------------------- |
| What is ObjectId?            | A 12-byte unique ID automatically created by MongoDB            |
| How long is it?              | 24 hexadecimal characters (0-9, a-f)                            |
| What's inside?               | Timestamp (when) + Machine/Process ID (where) + Counter (which) |
| Can I use my own?            | Yes, but you must ensure it's unique                            |
| Can I change it later?       | No — it's permanent (immutable)                                 |
| Can I get the creation time? | Yes — use `objectId.getTimestamp()`                             |

---

## 11. Complete Command Reference

### Database Commands

| Command             | Purpose                   | Example             |
| ------------------- | ------------------------- | ------------------- |
| `show dbs`          | List all databases        | `show dbs`          |
| `use dbName`        | Create/switch to database | `use library`       |
| `db.dropDatabase()` | Delete current database   | `db.dropDatabase()` |

### Collection Commands

| Command                           | Purpose              | Example                               |
| --------------------------------- | -------------------- | ------------------------------------- |
| `db.createCollection("name")`     | Create collection    | `db.createCollection("books")`        |
| `show collections`                | List all collections | `show collections`                    |
| `db.coll.drop()`                  | Delete collection    | `db.books.drop()`                     |
| `db.coll.renameCollection("new")` | Rename collection    | `db.books.renameCollection("novels")` |

### Document Commands (CRUD)

| Command                            | Purpose                | Example                                               |
| ---------------------------------- | ---------------------- | ----------------------------------------------------- |
| `db.coll.insertOne({})`            | Insert single document | `db.users.insertOne({name:"Alice"})`                  |
| `db.coll.find()`                   | Find all documents     | `db.users.find()`                                     |
| `db.coll.findOne({})`              | Find one document      | `db.users.findOne({name:"Alice"})`                    |
| `db.coll.updateOne({}, {$set:{}})` | Update single document | `db.users.updateOne({name:"Alice"}, {$set:{age:31}})` |
| `db.coll.deleteOne({})`            | Delete single document | `db.users.deleteOne({name:"Alice"})`                  |

---

## 12. Summary — Key Takeaways

### 💡 The Big Picture

**Remember these 5 core concepts:**

1. **MongoDB Driver** = The translator between you and the database
2. **WiredTiger** = The librarian who converts JSON ↔ BSON and manages storage
3. **mongosh** = Your command-line tool to control MongoDB
4. **BSON** = Binary format (faster, smaller, more types than JSON)
5. **ObjectId** = Unique 12-byte ID with hidden timestamp + machine info + counter

---

### 🎯 Most Important Commands (Memorize These)

```javascript
// Databases
show dbs                    // List all databases
use dbName                  // Create or switch to database
db.dropDatabase()           // Delete current database

// Collections
show collections            // List all collections
db.createCollection("name") // Create collection
db.coll.drop()             // Delete collection

// Documents (Basic CRUD)
db.coll.insertOne({...})   // Insert one document
db.coll.find()             // Find all documents
db.coll.findOne({...})     // Find one document
```

---

### ⚠️ Common Gotchas (Don't Forget!)

| Gotcha                                            | What Happens                 | Remember This                                       |
| ------------------------------------------------- | ---------------------------- | --------------------------------------------------- |
| Created a database but `show dbs` doesn't show it | Empty databases aren't saved | Add a collection or document first                  |
| Trying to change `_id` after insertion            | Error!                       | `_id` is immutable (locked forever)                 |
| Expecting to rename a database                    | No command exists            | Must copy to new DB and delete old one              |
| Using `localhost` vs `127.0.0.1`                  | Both work the same           | `localhost` = domain name, `127.0.0.1` = IP address |

---

### 🔑 Key Facts for Interviews

| Question                                | Quick Answer                     |
| --------------------------------------- | -------------------------------- |
| What port does MongoDB use?             | **27017**                        |
| What's the default database in mongosh? | **test**                         |
| How big is an ObjectId?                 | **12 bytes** (24 hex characters) |
| Can you rename a database?              | **No** — must copy and delete    |
| What format does MongoDB store on disk? | **BSON** (not JSON)              |
| Can you provide your own `_id`?         | **Yes** — but it must be unique  |

---

### 📚 Quick Reference

```
Connection URL:    mongodb://localhost:27017
Shell command:     mongosh
Exit shell:        exit (or Ctrl+C)
Start server:      net start mongodb (Windows)
Stop server:       net stop mongodb (Windows)
Check status:      sc query mongodb (Windows)
```

---

## 13. Revision Checklist

### Architecture

- [ ] Can you explain the MongoDB Driver and what it does?
- [ ] Can you explain WiredTiger and its role (JSON↔BSON conversion, disk I/O)?
- [ ] Can you trace the flow: User → Driver → WiredTiger → Disk?

### Shell & Server

- [ ] Can you enter and exit mongosh?
- [ ] Can you start and stop the MongoDB server (Windows: `net start/stop mongodb`)?
- [ ] Can you check MongoDB server status (`sc query mongodb`)?
- [ ] Do you know the default database when entering mongosh (`test`)?

### GUI vs CLI

- [ ] Can you list 3 differences between Compass (GUI) and mongosh (CLI)?
- [ ] Do you know when to use each (exploration vs production work)?

### Connection URLs

- [ ] Can you break down `mongodb://localhost:27017`?
- [ ] Do you know the difference between `localhost` and `127.0.0.1`?

### Database Commands

- [ ] Can you list all databases (`show dbs`)?
- [ ] Can you create or switch to a database (`use library`)?
- [ ] Do you know when a database is saved to disk (after collection creation or data insert)?
- [ ] Can you delete a database (`db.dropDatabase()`)?
- [ ] Do you know that renaming a database is NOT possible?

### Collection Commands

- [ ] Can you create a collection (`db.createCollection("books")`)?
- [ ] Can you show all collections (`show collections`)?
- [ ] Can you drop a collection (`db.books.drop()`)?
- [ ] Can you rename a collection (`db.books.renameCollection("novels")`)?

### Document Operations

- [ ] Can you insert a single document (`db.coll.insertOne({})`)?
- [ ] Can you find all documents (`db.coll.find()`)?
- [ ] Can you find one document (`db.coll.findOne({})`)?

### BSON

- [ ] Can you explain what BSON is (Binary JSON)?
- [ ] Can you list 3 differences between JSON and BSON?
- [ ] Can you name 3 extra datatypes in BSON (ObjectId, Date, Binary)?
- [ ] Do you know why MongoDB uses BSON (performance, rich types, efficiency)?

### ObjectId

- [ ] Can you explain what ObjectId is (12-byte unique identifier)?
- [ ] Can you break down the 3 parts of ObjectId (4B timestamp + 5B PUI + 3B counter)?
- [ ] Do you know that `_id` is auto-generated if not provided?
- [ ] Do you know that `_id` is immutable (cannot be changed)?
- [ ] Can you provide a custom `_id` when inserting?
- [ ] Can you extract the timestamp from an ObjectId (`id.getTimestamp()`)?

---

> **🎤 Interview Tip — How to Answer "What's the difference between JSON and BSON?"**
>
> **Don't just say:** "BSON is binary JSON"
>
> **Instead, say this:**
>
> _"JSON is a text-based data format that humans can read easily — it's used to send data between systems. BSON is MongoDB's internal binary format — it's what MongoDB actually stores on disk. The key differences are:_
>
> _1. **Format:** JSON is text, BSON is binary (0s and 1s)_  
> _2. **Speed:** BSON is faster because computers work with binary naturally_  
> _3. **Size:** BSON is smaller because it's compressed_  
> _4. **Types:** BSON supports extra types like ObjectId, Date, and Binary data that JSON doesn't have_
>
> _As a developer, you work with JavaScript objects — MongoDB automatically converts them to BSON when saving and back to JavaScript when reading. You never see the binary directly."_
>
> **Why this answer works:** It shows you understand BOTH the technical details AND the practical workflow. That's what interviewers want.
