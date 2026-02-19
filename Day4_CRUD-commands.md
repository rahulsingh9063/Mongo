# MongoDB CRUD Operations — Complete Guide

## 📚 What You'll Learn

This guide covers **all MongoDB CRUD (Create, Read, Update, Delete) operations** in detail:

✅ Inserting multiple documents with insertMany()  
✅ Finding documents with findOne() and find()  
✅ Understanding projections (showing/hiding fields)  
✅ Deleting documents safely  
✅ Updating documents (single and multiple)  
✅ Capped collections (auto-delete old data)  
✅ Bulk operations (multiple queries in one go)  
✅ Importing data from JSON files

**Best for:** MongoDB beginners, CRUD mastery, interview preparation

---

## Table of Contents

1. [insertMany() — Insert Multiple Documents](#1-insertmany--insert-multiple-documents)
2. [insertMany() Options — ordered & writeConcern](#2-insertmany-options--ordered--writeconcern)
3. [findOne() — Find Single Document](#3-findone--find-single-document)
4. [Projection — Show/Hide Fields](#4-projection--showhide-fields)
5. [find() — Find Multiple Documents](#5-find--find-multiple-documents)
6. [Cursors — What They Are](#6-cursors--what-they-are)
7. [deleteOne() — Delete Single Document](#7-deleteone--delete-single-document)
8. [deleteMany() — Delete Multiple Documents](#8-deletemany--delete-multiple-documents)
9. [updateOne() — Update Single Document](#9-updateone--update-single-document)
10. [updateMany() — Update Multiple Documents](#10-updatemany--update-multiple-documents)
11. [Capped Collections — Auto-Delete Old Data](#11-capped-collections--auto-delete-old-data)
12. [bulkWrite() — Batch Operations](#12-bulkwrite--batch-operations)
13. [mongoimport — Import JSON Data](#13-mongoimport--import-json-data)
14. [Complete CRUD Summary](#14-complete-crud-summary)
15. [Revision Checklist](#15-revision-checklist)

---

## 1. insertMany() — Insert Multiple Documents

### What It Does

**Simple explanation:** `insertMany()` lets you insert **multiple documents at once** instead of calling `insertOne()` repeatedly.

Think of it like:

- **insertOne()** = Adding one student to a class roster (slow if you have 50 students)
- **insertMany()** = Adding all 50 students at once (fast!)

---

### Syntax

```javascript
db.collectionName.insertMany(
    [ document1, document2, document3, ... ],  // Array of documents
    { options }                                // Optional settings
)
```

---

### Basic Example

```javascript
db.students.insertMany([
    { name: "ashwin", age: 12, _id: "1" },
    { name: "sri", age: 11, _id: "2" },
    { name: "varun", age: 23, _id: "3" }
]);

// Output:
{
    acknowledged: true,
    insertedIds: {
        '0': '1',
        '1': '2',
        '2': '3'
    }
}
```

**What happened:**

1. MongoDB received 3 documents
2. All 3 were inserted successfully
3. MongoDB confirms with `acknowledged: true`
4. Returns the IDs of all inserted documents

---

### Visual Flow

```mermaid
graph LR
    A["Your Code:\ninsertMany([doc1, doc2, doc3])"] -->|Send to MongoDB| B["MongoDB Database"]
    B -->|Process| C["Insert doc1"]
    C -->|Next| D["Insert doc2"]
    D -->|Next| E["Insert doc3"]
    E -->|Success| F["Return:\n{acknowledged: true,\ninsertedIds: [1,2,3]}"]
```

---

### Why Use insertMany() Instead of Multiple insertOne()?

| Aspect            | insertOne() × 3      | insertMany()        |
| ----------------- | -------------------- | ------------------- |
| **Network calls** | 3 separate calls     | 1 single call       |
| **Speed**         | Slow (3 round trips) | Fast (1 round trip) |
| **Code**          | Repetitive           | Clean               |
| **Atomicity**     | Each independent     | Batch operation     |

**Bottom line:** Always use `insertMany()` when inserting multiple documents.

---

## 2. insertMany() Options — ordered & writeConcern

### The Options Object

```javascript
db.collectionName.insertMany(
    [ documents ],
    {
        ordered: true/false,        // Stop on error? (default: true)
        writeConcern: { ... }       // Acknowledgment settings
    }
)
```

---

### Option 1: `ordered` (Stop on Error?)

**What it controls:** What happens if one document fails to insert?

```
ordered: true  (default) → Stop immediately on error
ordered: false           → Skip errors, continue inserting
```

---

#### Example: Duplicate Key Error

```javascript
// Try to insert 3 documents, but document 2 has a duplicate _id
db.students.insertMany(
  [
    { name: "sirisha", age: 11, _id: "4" }, // ✅ Inserts successfully
    { name: "ajay", age: 10, _id: "2" }, // ❌ ERROR! _id "2" already exists
    { name: "ajay", age: 10, _id: "5" }, // ❓ What happens to this one?
  ],
  { ordered: true }, // Default behavior
);

// Result with ordered: true
// ✅ Document 1 inserted (_id: "4")
// ❌ Document 2 fails (duplicate _id: "2")
// ⛔ Document 3 SKIPPED (operation stopped)
```

---

#### Changing to `ordered: false`

```javascript
db.students.insertMany(
  [
    { name: "sirisha", age: 11, _id: "4" }, // ✅ Inserts successfully
    { name: "ajay", age: 10, _id: "2" }, // ❌ ERROR! Duplicate _id
    { name: "ajay", age: 10, _id: "5" }, // ✅ Inserts successfully (skips error)
  ],
  { ordered: false }, // Continue despite errors
);

// Result with ordered: false
// ✅ Document 1 inserted (_id: "4")
// ❌ Document 2 fails (duplicate _id: "2") — but we continue
// ✅ Document 3 inserted (_id: "5")
```

---

#### Visual Comparison

```
ordered: true (default)
─────────────────────────────────
Insert doc1  ✅
Insert doc2  ❌  [STOP! Don't continue]
Insert doc3  ⛔  [Never attempted]


ordered: false
─────────────────────────────────
Insert doc1  ✅
Insert doc2  ❌  [Skip error, continue]
Insert doc3  ✅
```

---

#### When to Use Each

| Scenario                   | Use              | Reason                                                   |
| -------------------------- | ---------------- | -------------------------------------------------------- |
| **Importing user data**    | `ordered: false` | Some users might have duplicate emails — insert the rest |
| **Financial transactions** | `ordered: true`  | If one fails, stop everything (data integrity)           |
| **Migration scripts**      | `ordered: false` | Skip bad records, import what you can                    |
| **Critical operations**    | `ordered: true`  | All or nothing approach                                  |

---

### Option 2: `writeConcern` (Acknowledgment Settings)

**What it controls:** How MongoDB confirms that data was written successfully.

```javascript
writeConcern: {
    w: 0/1/"majority",    // Acknowledgment level
    j: true/false,        // Write to journal?
    wtimeout: 5000        // Max wait time (milliseconds)
}
```

---

#### `w` — Acknowledgment Level

```
w: 0  →  "Fire and forget" (no confirmation, fastest but dangerous)
w: 1  →  "Confirm primary wrote it" (default, safe)
w: "majority"  →  "Wait for majority of replicas to confirm" (safest)
```

**Simple explanation:**

| Value           | What MongoDB Does                    | Speed      | Safety       |
| --------------- | ------------------------------------ | ---------- | ------------ |
| `w: 0`          | "I sent it, don't care if it worked" | ⚡ Fastest | ❌ Not safe  |
| `w: 1`          | "Confirm primary server wrote it"    | ✅ Normal  | ✅ Safe      |
| `w: "majority"` | "Wait for most replicas to confirm"  | 🐌 Slower  | ✅ Very safe |

**Real-world example:**

```javascript
// Social media "like" button — speed matters more than perfect safety
db.likes.insertMany(
  [{ userId: 123, postId: 456 }],
  { writeConcern: { w: 0 } }, // Don't wait for confirmation
);

// Bank transaction — safety is CRITICAL
db.transactions.insertMany(
  [{ from: "A", to: "B", amount: 1000 }],
  { writeConcern: { w: "majority" } }, // Wait for majority confirmation
);
```

---

#### `j` — Journal (Write-Ahead Log)

**What it is:** MongoDB keeps a **journal** — a special log file that records every write operation before actually writing to the database files.

Think of it like:

- You write in your **diary** (journal) before organizing the info into your **notebook** (database)
- If something crashes, you can replay the diary to recover

```
j: true  → Write to journal first (slower but safer)
j: false → Skip journal (faster but less durable)
```

**When to use `j: true`:**

- Financial systems
- Medical records
- Any data you CANNOT afford to lose

---

#### `wtimeout` — Maximum Wait Time

**What it does:** If MongoDB can't confirm the write within this time, it times out.

```javascript
db.students.insertMany([{ name: "test" }], {
  writeConcern: {
    w: "majority",
    wtimeout: 5000, // Wait max 5 seconds for majority confirmation
  },
});

// If majority of replicas don't confirm within 5 seconds → timeout error
```

---

### Complete Example with All Options

```javascript
db.students.insertMany(
  [
    { name: "sirisha", age: 11, _id: "4" },
    { name: "ajay", age: 10, _id: "2" }, // Duplicate — will fail
    { name: "ajay", age: 10, _id: "5" },
  ],
  {
    ordered: false, // Continue despite errors
    writeConcern: {
      w: 1, // Wait for primary confirmation
      j: true, // Write to journal
      wtimeout: 5000, // Max 5 seconds wait
    },
  },
);

// Result:
// ✅ Documents 1 and 3 inserted
// ❌ Document 2 failed (but didn't stop the others)
// ✅ Primary server confirmed write
// ✅ Write recorded in journal
```

---

## 3. findOne() — Find Single Document

### What It Does

**Simple explanation:** `findOne()` finds and returns **one document** that matches your filter. If multiple documents match, it returns the **first one**.

Think of it like:

- You're in a library looking for a book
- You give the librarian the book title
- They give you the **first copy** they find (even if there are 10 copies)

---

### Syntax

```javascript
db.collectionName.findOne(
  { filter }, // Which document to find
  { projection }, // Which fields to show/hide (optional)
);
```

---

### Example 1: No Filter (Get First Document)

```javascript
// All three do the same thing — return the FIRST document
db.students.findOne();
db.students.findOne({});
db.students.findOne({}, {});

// Returns:
{
    _id: "1",
    name: "ashwin",
    age: 12
}
```

> 💡 **Note:** "First" means the first document in the collection's internal storage order (not necessarily the first inserted).

---

### Example 2: Filter by Field

```javascript
// Find student named "sirisha"
db.students.findOne({ name: "sirisha" });

// Returns:
{
    _id: "4",
    name: "sirisha",
    age: 11
}
```

> ⚠️ **MongoDB is case-sensitive!**
>
> ```javascript
> db.students.findOne({ name: "Sirisha" }); // Won't find "sirisha"
> db.students.findOne({ name: "SIRISHA" }); // Won't find "sirisha"
> ```

---

### Example 3: Multiple Matches (Returns First)

```javascript
// Multiple students have age 10
db.students.findOne({ age: 10 });

// Returns ONLY THE FIRST match:
{
    _id: "2",
    name: "sri",
    age: 10
}

// Even if there are 5 students with age 10, you only get one back
```

---

### What If No Match?

```javascript
db.students.findOne({ name: "nonexistent" });

// Returns:
null;
```

---

## 4. Projection — Show/Hide Fields

### What Is Projection?

**Simple explanation:** Projection lets you control **which fields** appear in the result. You can show or hide specific fields.

Think of it like:

- You have a student record with 10 fields
- You only want to see `name` and `age` (not email, phone, address, etc.)
- Projection = choosing which columns to display

---

### Projection Syntax

```javascript
db.collectionName.findOne(
  { filter },
  {
    fieldName1: 1, // Show this field
    fieldName2: 0, // Hide this field
    _id: 0, // Hide _id (optional)
  },
);
```

---

### The Rules

| Value | Meaning                | Example                   |
| ----- | ---------------------- | ------------------------- |
| `1`   | **Include** this field | `{ name: 1 }` → Show name |
| `0`   | **Exclude** this field | `{ age: 0 }` → Hide age   |

**Special rule:** `_id` is **always included by default** (unless you explicitly set `_id: 0`)

---

### Example 1: Show Only Specific Fields

```javascript
// Show only name and age (hide everything else)
db.students.findOne(
    { age: 10 },
    { name: 1, age: 1 }
);

// Returns:
{
    _id: "2",      // ← Still included (default behavior)
    name: "sri",
    age: 10
}
```

---

### Example 2: Hide \_id

```javascript
// Show only name, hide _id
db.students.findOne({ age: 10 }, { name: 1, _id: 0 });

// Returns:
{
  name: "sri"; // ← Clean! No _id
}
```

---

### Example 3: Hide Specific Fields

```javascript
// Hide age, show everything else
db.students.findOne(
    { name: "ashwin" },
    { age: 0 }
);

// Returns:
{
    _id: "1",
    name: "ashwin"
    // age is hidden
}
```

---

### ⚠️ Cannot Mix Include and Exclude (Except \_id)

```javascript
// ❌ WRONG — Cannot mix 1 and 0 (except for _id)
db.students.findOne(
  {},
  { name: 1, age: 0 }, // ERROR!
);

// ✅ CORRECT — Either include OR exclude (except _id)
db.students.findOne({}, { name: 1, age: 1, _id: 0 }); // Include name, age; exclude _id
db.students.findOne({}, { age: 0 }); // Exclude age, show rest
```

---

### Visual Example

```
Original Document:
{
    _id: "1",
    name: "ashwin",
    age: 12,
    email: "ash@example.com",
    phone: "1234567890"
}

Projection: { name: 1, _id: 0 }
↓
Result:
{
    name: "ashwin"
}
```

---

## 5. find() — Find Multiple Documents

### What It Does

**Simple explanation:** `find()` returns **all documents** that match your filter (not just one).

Think of it like:

- `findOne()` = "Give me **one** book by this author"
- `find()` = "Give me **all** books by this author"

---

### Syntax

```javascript
db.collectionName.find(
  { filter }, // Which documents to find
  { projection }, // Which fields to show/hide
);
```

---

### Example 1: Find All Documents

```javascript
// No filter = return ALL documents
db.students.find();
db.students.find({});

// Returns:
[
  { _id: "1", name: "ashwin", age: 12 },
  { _id: "2", name: "sri", age: 11 },
  { _id: "3", name: "varun", age: 23 },
  { _id: "4", name: "sirisha", age: 11 },
];
```

---

### Example 2: Find with Filter

```javascript
// Find all employees in department 10
db.emp.find({ deptNo: 10 });

// Returns all matching documents:
[
  { _id: 1, empName: "Alice", deptNo: 10 },
  { _id: 5, empName: "Bob", deptNo: 10 },
  { _id: 9, empName: "Charlie", deptNo: 10 },
];
```

---

### Example 3: Find with Projection

```javascript
// Show only employee names (hide _id)
db.emp.find(
  {}, // No filter — all employees
  { empName: 1, _id: 0 }, // Show name only
);

// Returns:
[{ empName: "Alice" }, { empName: "Bob" }, { empName: "Charlie" }];
```

---

### Example 4: Filter + Projection

```javascript
// Find employees in dept 10, show only name and dept
db.emp.find({ deptNo: 10 }, { empName: 1, deptNo: 1, _id: 0 });

// Returns:
[
  { empName: "Alice", deptNo: 10 },
  { empName: "Bob", deptNo: 10 },
];
```

---

### findOne() vs find()

|                 | findOne()             | find()                     |
| --------------- | --------------------- | -------------------------- |
| **Returns**     | One document (object) | Multiple documents (array) |
| **Return type** | `{...}` or `null`     | `[{...}, {...}]` or `[]`   |
| **Use when**    | You want exactly one  | You want all matches       |

```javascript
// findOne() returns an OBJECT
let student = db.students.findOne({ age: 10 });
console.log(student.name); // "sri"

// find() returns an ARRAY
let students = db.students.find({ age: 10 });
console.log(students[0].name); // "sri"
```

---

## 6. Cursors — What They Are

### What Is a Cursor?

**Simple explanation:** When you use `find()`, MongoDB doesn't immediately return all documents. Instead, it returns a **cursor** — a pointer that lets you iterate through results one at a time.

Think of it like:

- `find()` doesn't give you all books at once
- It gives you a **library cart** that brings you books one by one as you ask for them
- This saves memory (you don't need to hold 1 million books in your hands)

---

### How Cursors Work

```mermaid
graph LR
    A["db.students.find()"] -->|Returns| B["Cursor\n(pointer)"]
    B -->|"Request next doc"| C["Document 1"]
    B -->|"Request next doc"| D["Document 2"]
    B -->|"Request next doc"| E["Document 3"]
    B -->|"..."| F["Document N"]
```

---

### Using Cursors in mongosh

```javascript
// Get a cursor
let cursor = db.students.find();

// Iterate through results
cursor.forEach((doc) => {
  print(doc.name);
});

// Or convert to array (loads all into memory)
let allStudents = db.students.find().toArray();
```

---

### Cursor Methods

```javascript
let cursor = db.students.find();

// Get next document
cursor.next();

// Check if more documents exist
cursor.hasNext();

// Limit results
db.students.find().limit(5);

// Skip documents
db.students.find().skip(10);

// Sort results
db.students.find().sort({ age: 1 }); // 1 = ascending, -1 = descending
```

---

### Why Cursors Matter

**Without cursors:**

```javascript
// ❌ BAD — Load 1 million documents into memory at once
let allDocs = db.bigCollection.find(); // BOOM! Out of memory
```

**With cursors:**

```javascript
// ✅ GOOD — Process 1 million documents one at a time
db.bigCollection.find().forEach((doc) => {
  processDocument(doc); // Only one document in memory at a time
});
```

---

## 7. deleteOne() — Delete Single Document

### What It Does

**Simple explanation:** `deleteOne()` deletes **the first document** that matches your filter.

---

### Syntax

```javascript
db.collectionName.deleteOne({ filter });
```

---

### Example 1: Delete Specific Document

```javascript
// Delete student with _id "5"
db.students.deleteOne({ _id: "5" });

// Returns:
{
    acknowledged: true,
    deletedCount: 1
}
```

---

### Example 2: No Filter (Dangerous!)

```javascript
// ❌ ERROR — You must provide a filter
db.emp.deleteOne();
// Error: deleteOne() requires a filter

// ⚠️ DANGEROUS — Deletes the first document
db.emp.deleteOne({});
// Deletes the first employee (whoever that is)
```

> ⚠️ **Warning:** `deleteOne({})` with an empty filter deletes the first document. Always be specific!

---

### Example 3: Multiple Matches (Deletes First)

```javascript
// Multiple students have age 10
db.students.deleteOne({ age: 10 });

// Deletes ONLY THE FIRST match
// Returns:
{
    acknowledged: true,
    deletedCount: 1
}
```

---

### What If No Match?

```javascript
db.students.deleteOne({ name: "nonexistent" });

// Returns:
{
    acknowledged: true,
    deletedCount: 0  // ← Nothing deleted
}
```

---

## 8. deleteMany() — Delete Multiple Documents

### What It Does

**Simple explanation:** `deleteMany()` deletes **all documents** that match your filter.

---

### Syntax

```javascript
db.collectionName.deleteMany({ filter });
```

---

### Example 1: Delete All Matching Documents

```javascript
// Delete all students with age 10
db.students.deleteMany({ age: 10 });

// Returns:
{
    acknowledged: true,
    deletedCount: 3  // Deleted 3 students
}
```

---

### Example 2: Delete All Documents (VERY DANGEROUS!)

```javascript
// ⚠️⚠️⚠️ DANGER — Deletes EVERYTHING
db.students.deleteMany({});

// Returns:
{
    acknowledged: true,
    deletedCount: 50  // All documents deleted!
}
```

> ⚠️ **CRITICAL WARNING:** `deleteMany({})` with an empty filter deletes **ALL documents** in the collection. This is permanent and cannot be undone!

---

### deleteOne() vs deleteMany()

|                  | deleteOne()                | deleteMany()          |
| ---------------- | -------------------------- | --------------------- |
| **Deletes**      | First match only           | All matches           |
| **Empty filter** | Deletes first document     | Deletes ALL documents |
| **Safety**       | Safer                      | More dangerous        |
| **Use when**     | Removing one specific item | Cleaning up old data  |

---

## 9. updateOne() — Update Single Document

### What It Does

**Simple explanation:** `updateOne()` modifies **the first document** that matches your filter.

---

### Syntax

```javascript
db.collectionName.updateOne(
    { filter },      // Which document to update
    { $set: {...} }, // What to change
    { options }      // Optional settings
)
```

---

## 10. updateMany() — Update Multiple Documents

### What It Does

**Simple explanation:** `updateMany()` modifies **all documents** that match your filter.

---

### Syntax

```javascript
db.collectionName.updateMany(
    { filter },      // Which documents to update
    { $set: {...} }, // What to change
    { options }      // Optional settings
)
```

---

### updateOne() vs updateMany()

|                  | updateOne()            | updateMany()          |
| ---------------- | ---------------------- | --------------------- |
| **Updates**      | First match only       | All matches           |
| **Empty filter** | Updates first document | Updates ALL documents |
| **Safety**       | Safer                  | More dangerous        |
| **Use when**     | Changing one record    | Bulk updates          |

---

## 11. Capped Collections — Auto-Delete Old Data

### What Is a Capped Collection?

**Simple explanation:** A capped collection is like a **circular buffer** — when it gets full, MongoDB automatically deletes the **oldest documents** to make room for new ones.

Think of it like:

- A **DVR** that records TV shows
- When the hard drive is full, it deletes the oldest recordings
- You always have the most recent data

---

### Why Use Capped Collections?

| Use Case            | Why It Works                         |
| ------------------- | ------------------------------------ |
| **Log files**       | Keep only last 1000 log entries      |
| **Chat messages**   | Keep only last 100 messages per room |
| **Recent activity** | Keep only last hour of user actions  |
| **Live feeds**      | Keep only latest tweets/posts        |

**Key benefit:** You don't have to manually delete old data — MongoDB does it automatically!

---

### Creating a Capped Collection

```javascript
db.createCollection("capColl", {
  capped: true, // Enable capping
  size: 1024 * 1024 * 1024, // Max size: 1GB
  max: 3, // Max documents: 3
});
```

**Two limits work together:**

1. **Size limit** (in bytes) — Total storage space
2. **Document limit** — Total number of documents

**Whichever limit is hit first triggers deletion.**

---

### How It Works (Visual)

```
Capped collection with max: 3

Insert doc1:  [doc1]
Insert doc2:  [doc1, doc2]
Insert doc3:  [doc1, doc2, doc3]  ← Full!

Insert doc4:  [doc2, doc3, doc4]  ← doc1 auto-deleted!
Insert doc5:  [doc3, doc4, doc5]  ← doc2 auto-deleted!
```

---

### Example

```javascript
// Create capped collection with max 3 documents
db.createCollection("capColl", {
  capped: true,
  size: 1024 * 1024 * 1024,
  max: 3,
});

// Insert 2 documents
db.capColl.insertMany([{ id: 1 }, { id: 2 }]);

// Current state: [doc1, doc2]

// Insert 2 more (exceeds limit)
db.capColl.insertMany([{ id: 3 }, { id: 4 }]);

// Current state: [doc2, doc3, doc4]
// doc1 was auto-deleted!

// Find all
db.capColl.find();
// Returns: [{ id: 2 }, { id: 3 }, { id: 4 }]
```

---

### Capped Collection Properties

| Property            | Behavior                                            |
| ------------------- | --------------------------------------------------- |
| **Insertion order** | Always preserved (oldest deleted first)             |
| **No deletion**     | You CANNOT manually delete documents                |
| **No updates**      | Updates cannot increase document size               |
| **Fast inserts**    | Extremely fast (no index updates needed for oldest) |
| **Query order**     | Natural order = insertion order                     |

---

### Real-World Example: Log System

```javascript
// Keep only last 1000 log entries
db.createCollection("logs", {
  capped: true,
  size: 10 * 1024 * 1024, // 10MB
  max: 1000, // 1000 entries
});

// Application logs here continuously
db.logs.insertOne({
  timestamp: new Date(),
  level: "info",
  message: "User logged in",
});

// Old logs automatically deleted when limit reached
// No cleanup scripts needed!
```

---

## 12. bulkWrite() — Batch Operations

### What Is bulkWrite()?

**Simple explanation:** `bulkWrite()` lets you combine **multiple different operations** (insert, update, delete) into **one single database call**.

Think of it like:

- Instead of making 10 separate trips to the store
- You make **one trip** and buy everything at once
- Faster and more efficient

---

### Why Use bulkWrite()?

| Benefit         | Explanation                                        |
| --------------- | -------------------------------------------------- |
| **Speed**       | One network call instead of many                   |
| **Atomicity**   | All operations succeed or fail together (optional) |
| **Flexibility** | Mix insert, update, delete in one call             |
| **Efficiency**  | Less overhead, better performance                  |

---

### Syntax

```javascript
db.collectionName.bulkWrite([
    { insertOne: { document: {...} } },
    { updateOne: { filter: {...}, update: {...} } },
    { deleteOne: { filter: {...} } },
    // ... more operations
], { options });
```

---

### Example: Mixed Operations

```javascript
db.students.bulkWrite([
  // Insert a new student
  {
    insertOne: {
      document: { name: "new student", age: 20 },
    },
  },

  // Update a student's age
  {
    updateOne: {
      filter: { name: "ashwin" },
      update: { $set: { age: 14 } },
    },
  },

  // Delete a student
  {
    deleteOne: {
      filter: { name: "old student" },
    },
  },
]);

// All 3 operations done in ONE database call!
```

---

### Available Operations

```javascript
bulkWrite([
    // 1. Insert
    { insertOne: { document: {...} } },

    // 2. Update one
    { updateOne: { filter: {...}, update: {...} } },

    // 3. Update many
    { updateMany: { filter: {...}, update: {...} } },

    // 4. Replace one (replace entire document)
    { replaceOne: { filter: {...}, replacement: {...} } },

    // 5. Delete one
    { deleteOne: { filter: {...} } },

    // 6. Delete many
    { deleteMany: { filter: {...} } }
]);
```

---

### Options: ordered

```javascript
db.students.bulkWrite(
  [
    { insertOne: { document: { name: "test1" } } },
    { insertOne: { document: { _id: "duplicate" } } }, // Will fail
    { insertOne: { document: { name: "test2" } } },
  ],
  {
    ordered: false, // Continue on error
  },
);

// With ordered: true  → stops at error
// With ordered: false → continues despite errors
```

---

### Performance Comparison

```
Without bulkWrite (10 operations):
  insert() → network call
  update() → network call
  delete() → network call
  ... (7 more network calls)
  Total: 10 network round trips

With bulkWrite (10 operations):
  bulkWrite([...all 10 operations...])
  Total: 1 network round trip

Speed improvement: ~10x faster
```

---

## 13. mongoimport — Import JSON Data

### What Is mongoimport?

**Simple explanation:** `mongoimport` is a **command-line tool** that lets you import data from JSON, CSV, or TSV files directly into MongoDB.

Think of it like:

- You have a spreadsheet with 10,000 rows
- Instead of manually inserting each row, you import the whole file at once
- Like copy-pasting but automated

---

### Installation

**Download MongoDB Database Tools:**

```
https://fastdl.mongodb.org/tools/db/mongodb-database-tools-windows-x86_64-100.14.1.msi
```

After installation, `mongoimport` command becomes available in your terminal.

---

### Basic Syntax

```bash
mongoimport "path/to/file.json" -d databaseName -c collectionName --jsonArray
```

**Parameters:**

- `"path/to/file.json"` — Path to your JSON file
- `-d databaseName` — Which database to import into
- `-c collectionName` — Which collection to import into
- `--jsonArray` — Treat file as array of documents (not one doc per line)

---

### Example: Import Students

**Step 1: Create a JSON file (students.json)**

```json
[
  { "name": "Alice", "age": 20, "grade": "A" },
  { "name": "Bob", "age": 22, "grade": "B" },
  { "name": "Charlie", "age": 21, "grade": "A" }
]
```

**Step 2: Import via command line**

```bash
mongoimport "C:/data/students.json" -d school -c students --jsonArray
```

**Result:**

```
2024-02-15T10:30:00.000+0000    connected to: mongodb://localhost/
2024-02-15T10:30:00.100+0000    3 document(s) imported successfully
```

**Step 3: Verify in mongosh**

```javascript
use school
db.students.find()

// Returns:
[
    { _id: ObjectId("..."), name: "Alice", age: 20, grade: "A" },
    { _id: ObjectId("..."), name: "Bob", age: 22, grade: "B" },
    { _id: ObjectId("..."), name: "Charlie", age: 21, grade: "A" }
]
```

---

### Common Options

```bash
# Import with custom _id field
mongoimport "file.json" -d myDB -c myColl --jsonArray --mode upsert

# Import CSV file
mongoimport "file.csv" -d myDB -c myColl --type csv --headerline

# Drop collection before import (clean slate)
mongoimport "file.json" -d myDB -c myColl --jsonArray --drop

# Import to remote MongoDB
mongoimport "file.json" --uri "mongodb://username:password@host:port/database" -c myColl
```

---

### When to Use mongoimport

| Use Case             | Why It Works                          |
| -------------------- | ------------------------------------- |
| **Migrating data**   | Move data from old system to MongoDB  |
| **Seeding database** | Pre-populate database for testing     |
| **Bulk import**      | Load 100,000 records from spreadsheet |
| **Data recovery**    | Restore from JSON backup              |
| **Development**      | Load sample data quickly              |

---

## 14. Complete CRUD Summary

### The 4 CRUD Operations

```
C = Create  → insertOne(), insertMany()
R = Read    → findOne(), find()
U = Update  → updateOne(), updateMany()
D = Delete  → deleteOne(), deleteMany()
```

---

### Quick Reference Table

| Operation       | Method                         | Returns            | Use When                  |
| --------------- | ------------------------------ | ------------------ | ------------------------- |
| **Insert one**  | `insertOne({doc})`             | `insertedId`       | Adding one record         |
| **Insert many** | `insertMany([docs])`           | `insertedIds`      | Adding multiple records   |
| **Find one**    | `findOne({filter})`            | Document or `null` | Finding one specific item |
| **Find many**   | `find({filter})`               | Cursor (array)     | Finding all matches       |
| **Update one**  | `updateOne({filter}, {$set})`  | Modified count     | Changing one record       |
| **Update many** | `updateMany({filter}, {$set})` | Modified count     | Changing multiple records |
| **Delete one**  | `deleteOne({filter})`          | Deleted count      | Removing one record       |
| **Delete many** | `deleteMany({filter})`         | Deleted count      | Removing multiple records |

---

### Common Patterns

```javascript
// 1. Create
db.users.insertOne({ name: "Alice", email: "alice@example.com" });

// 2. Read
let user = db.users.findOne({ email: "alice@example.com" });

// 3. Update
db.users.updateOne({ email: "alice@example.com" }, { $set: { age: 30 } });

// 4. Delete
db.users.deleteOne({ email: "alice@example.com" });
```

---

### Safety Checklist

| ⚠️ Danger Zone                  | Safe Practice                              |
| ------------------------------- | ------------------------------------------ |
| `deleteMany({})` — deletes ALL  | Always use a filter                        |
| `updateMany({})` — updates ALL  | Always use a filter                        |
| No backup before bulk delete    | Always backup first                        |
| `ordered: true` on large import | Use `ordered: false` to continue on errors |

---

## 15. Revision Checklist

### insertMany()

- [ ] Can you insert multiple documents at once?
- [ ] Do you know what `ordered: true/false` does?
- [ ] Can you explain `writeConcern` options (w, j, wtimeout)?
- [ ] Do you know when to use `ordered: false`?

### findOne() & find()

- [ ] Can you find a single document with a filter?
- [ ] Can you find all matching documents?
- [ ] Do you know the difference between findOne() and find()?
- [ ] Can you explain what happens with an empty filter?

### Projection

- [ ] Can you show only specific fields?
- [ ] Can you hide specific fields?
- [ ] Do you know \_id is included by default?
- [ ] Can you hide \_id explicitly?
- [ ] Do you know you cannot mix include and exclude (except \_id)?

### Cursors

- [ ] Can you explain what a cursor is?
- [ ] Do you know why cursors save memory?
- [ ] Can you use cursor methods (limit, skip, sort)?

### Delete Operations

- [ ] Can you delete one document?
- [ ] Can you delete multiple documents?
- [ ] Do you know `deleteMany({})` deletes ALL documents?
- [ ] Can you check how many documents were deleted?

### Update Operations

- [ ] Can you update one document with `$set`?
- [ ] Can you update multiple documents?
- [ ] Can you increment a number with `$inc`?
- [ ] Can you remove a field with `$unset`?
- [ ] Do you know the difference between updateOne() and updateMany()?

### Capped Collections

- [ ] Can you explain what a capped collection is?
- [ ] Do you know the two limits (size and max)?
- [ ] Can you name a use case (logs, chat messages)?
- [ ] Do you know old documents are auto-deleted?

### bulkWrite()

- [ ] Can you combine multiple operations in one call?
- [ ] Can you mix insert, update, and delete?
- [ ] Do you know why bulkWrite() is faster?
- [ ] Can you use `ordered: false` to continue on errors?

### mongoimport

- [ ] Can you import a JSON file into MongoDB?
- [ ] Do you know the basic syntax?
- [ ] Can you use `--jsonArray` flag?
- [ ] Do you know when to use mongoimport?

---

> **🎤 Interview Tip — "What's the difference between updateOne and updateMany?"**
>
> **Answer like this:**
>
> _"Both updateOne and updateMany modify documents that match a filter, but updateOne stops after modifying the first match, while updateMany continues and updates all matching documents._
>
> _For example, if I want to give a 10% raise to one specific employee, I'd use updateOne with their employee ID. But if I want to give a 5% raise to everyone in the Sales department, I'd use updateMany with a department filter._
>
> _The key difference is scale — updateOne affects one document maximum, updateMany can affect thousands. For safety, I always check how many documents match my filter before using updateMany, especially with an empty filter which would update everything."_
>
> **Why this works:** Shows you understand both the technical difference AND the practical use cases, plus mentions safety considerations.
