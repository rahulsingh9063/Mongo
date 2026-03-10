# MongoDB Cursors, Sorting, Pagination & Data Modeling

### Complete Guide to find(), Cursors, Query Chaining & Schema Design

---

## 📚 What You'll Learn

This guide covers **MongoDB advanced query operations and data modeling**:

✅ find() vs findOne() — Return types  
✅ Cursors explained  
✅ Cursor methods (count, forEach, pretty, sort, limit, skip)  
✅ Sorting (ascending/descending)  
✅ Pagination with limit() and skip()  
✅ Query chaining  
✅ Data modeling fundamentals  
✅ Embedded vs Reference documents  
✅ Schema validation

**Best for:** MongoDB queries, pagination, data structure design, interview preparation

---

## Table of Contents

1. [find() vs findOne()](#1-find-vs-findone)
2. [What is a Cursor?](#2-what-is-a-cursor)
3. [Cursor Batching](#3-cursor-batching)
4. [Cursor Methods](#4-cursor-methods)
5. [count() — Count Documents](#5-count--count-documents)
6. [forEach() — Iterate Documents](#6-foreach--iterate-documents)
7. [sort() — Sort Results](#7-sort--sort-results)
8. [limit() — Restrict Results](#8-limit--restrict-results)
9. [skip() — Skip Documents](#9-skip--skip-documents)
10. [Query Chaining](#10-query-chaining)
11. [Pagination Patterns](#11-pagination-patterns)
12. [Data Modeling Basics](#12-data-modeling-basics)
13. [Embedded Documents](#13-embedded-documents)
14. [Schema Validation](#14-schema-validation)
15. [Summary](#15-summary)
16. [Revision Checklist](#16-revision-checklist)

---

## 1. find() vs findOne()

### Return Type Difference

```javascript
// find() returns CURSOR (not actual documents)
db.emp.find({ age: { $gt: 700 } });
// Returns: Cursor object

// findOne() returns DOCUMENT (actual object)
db.emp.findOne({ age: { $gt: 700 } });
// Returns: { _id: ..., name: "Alice", age: 750 }
```

---

### Key Differences

| Method      | Returns                           | Use For                        |
| ----------- | --------------------------------- | ------------------------------ |
| `find()`    | **Cursor** (pointer to documents) | Multiple documents, pagination |
| `findOne()` | **Document** (single object)      | Single document only           |

---

### find() Return Type

```javascript
// find() returns cursor
let result = db.emp.find({ age: { $gt: 700 } });

console.log(result);
// Output: Cursor object (not documents)

// To see documents, iterate cursor:
result.forEach((doc) => {
  console.log(doc);
});
```

---

### findOne() Return Type

```javascript
// findOne() returns single document
let result = db.emp.findOne({ age: { $gt: 700 } });

console.log(result);
// Output: { _id: ObjectId("..."), name: "Alice", age: 750 }

// It's already a document, not a cursor
console.log(result.name); // "Alice"
console.log(result.age); // 750
```

---

### Visual: Return Types

```
find() Flow:
────────────────────────────────────────
Query: db.emp.find({ age: { $gt: 700 } })
   ↓
Returns: Cursor { ... }
   ↓
Cursor points to documents in database
   ↓
Access documents via cursor methods


findOne() Flow:
────────────────────────────────────────
Query: db.emp.findOne({ age: { $gt: 700 } })
   ↓
Returns: { _id: ..., name: "Alice", age: 750 }
   ↓
Directly usable document object
```

---

### When to Use Which

```javascript
// Use find() when:
// - Getting multiple documents
// - Need pagination
// - Need sorting/filtering
db.emp.find({ department: "Sales" });

// Use findOne() when:
// - Getting single document by ID
// - Checking if document exists
// - Need just one document
db.emp.findOne({ _id: ObjectId("...") });
```

---

## 2. What is a Cursor?

### Definition

**A cursor is a pointer to the result set of a query. It does NOT contain the actual documents, but points to them in the database.**

---

### Why Cursors?

```
Problem without cursors:
────────────────────────────────────────
Query returns 1 million documents
All loaded into memory at once
Application crashes (out of memory)


Solution with cursors:
────────────────────────────────────────
Query returns cursor (pointer)
Documents loaded in batches (e.g., 20 at a time)
Memory efficient
Application stays responsive
```

---

### Cursor Object Structure

```javascript
// Cursor is an object with methods
let cursor = db.emp.find({ age: { $gt: 700 } });

// Cursor object (conceptual structure):
{
    // Data
    query: { age: { $gt: 700 } },
    currentPosition: 0,
    batchSize: 20,

    // Methods
    count: function() { },
    pretty: function() { },
    forEach: function() { },
    sort: function() { },
    skip: function() { },
    limit: function() { },
    toArray: function() { }
}
```

---

### Cursor vs Documents

```javascript
// Cursor (lightweight pointer)
let cursor = db.emp.find({ age: { $gt: 700 } });
// Memory usage: ~1 KB (just the cursor object)

// Actual documents (heavy data)
let documents = cursor.toArray();
// Memory usage: depends on document size × count
// Example: 1000 documents × 1KB each = 1 MB
```

---

### Visual: Cursor Concept

```
Database:
┌─────────────────────────────────────────┐
│  Documents (stored on disk)             │
│  [Doc1] [Doc2] [Doc3] ... [Doc1000000] │
└─────────────────────────────────────────┘
         ↑
         │ Cursor points here
         │
┌─────────────────────────────────────────┐
│  Cursor Object (in memory)              │
│  - Query: { age: { $gt: 700 } }        │
│  - Current position: 0                  │
│  - Batch size: 20                       │
└─────────────────────────────────────────┘
```

---

## 3. Cursor Batching

### Default Batch Size

**In MongoDB shell (mongosh), the default batch size is 20 documents.**

```javascript
db.emp.find({ age: { $gt: 700 } });
// Displays first 20 documents automatically
```

---

### How Batching Works

```
Query: db.emp.find() → 1000 matching documents

Step 1: First 20 documents displayed
Step 2: Type "it" (iterate) → Next 20 documents
Step 3: Type "it" again → Next 20 documents
...
Continue until all documents shown
```

---

### Using "it" Command

```javascript
// First query
db.emp.find({ age: { $gt: 700 } });
// Shows documents 1-20

// Type: it
// Shows documents 21-40

// Type: it
// Shows documents 41-60

// And so on...
```

---

### Visual: Batching Process

```
Total: 100 documents

Batch 1 (default):
[Doc1] [Doc2] ... [Doc20] ← Displayed immediately
                            Type "it" for next batch

Batch 2:
[Doc21] [Doc22] ... [Doc40] ← Displayed after "it"
                             Type "it" for next batch

Batch 3:
[Doc41] [Doc42] ... [Doc60] ← Displayed after "it"

...continues...
```

---

### Why Batching?

```
Benefits:
────────────────────────────────────────
✅ Memory efficient
✅ Faster initial response
✅ Better user experience
✅ Prevents memory overflow
✅ Network bandwidth optimization
```

---

## 4. Cursor Methods

### Available Cursor Methods

```javascript
let cursor = db.emp.find({ age: { $gt: 700 } });

// Available methods:
cursor.count(); // Count matching documents
cursor.pretty(); // Format output nicely
cursor.forEach(fn); // Iterate each document
cursor.sort({ age: 1 }); // Sort results
cursor.limit(10); // Limit to 10 documents
cursor.skip(5); // Skip first 5 documents
cursor.toArray(); // Convert to array
```

---

### Method Categories

```
Query Execution:
- forEach()    → Iterate documents
- toArray()    → Get all as array

Query Modification:
- sort()       → Sort results
- limit()      → Restrict count
- skip()       → Skip documents

Information:
- count()      → Count documents
- pretty()     → Format output
```

---

## 5. count() — Count Documents

### What Does count() Do?

**Returns the number of documents matching the query.**

---

### Basic Usage

```javascript
// Count all employees
db.emp.find().count();
// Returns: 150

// Count employees with age > 700
db.emp.find({ age: { $gt: 700 } }).count();
// Returns: 25

// Alternative syntax (deprecated but still works)
db.emp.count({ age: { $gt: 700 } });
// Returns: 25
```

---

### Examples

```javascript
// Example 1: Count documents in collection
db.emp.find().count();
// Returns: 1000

// Example 2: Count with filter
db.emp.find({ department: "Sales" }).count();
// Returns: 150

// Example 3: Count with multiple conditions
db.emp
  .find({
    age: { $gt: 30 },
    department: "Sales",
  })
  .count();
// Returns: 75
```

---

### Your Example

```javascript
// Data
db.emp.insertMany([
  { count: 15 },
  { count: 16 },
  { count: 17 },
  { count: 18 },
  { count: 19 },
  { count: 19 },
  { count: 20 },
]);

// Count all documents
db.emp.find().count();
// Returns: 7

// Count documents where count > 17
db.emp.find({ count: { $gt: 17 } }).count();
// Returns: 4  (18, 19, 19, 20)
```

---

### count() Return Type

```javascript
let result = db.emp.find({ age: { $gt: 700 } }).count();

console.log(result);
// Output: 25 (number, not cursor or object)

console.log(typeof result);
// Output: "number"
```

---

## 6. forEach() — Iterate Documents

### What Does forEach() Do?

**Executes a function for each document in the cursor.**

---

### Syntax

```javascript
db.collection.find().forEach(function (doc) {
  // Code to execute for each document
});

// Arrow function syntax
db.collection.find().forEach((doc) => {
  // Code to execute for each document
});
```

---

### Basic Example

```javascript
// Print each document
db.emp.find().forEach((doc) => {
  print(doc.name);
});

// Output:
// Alice
// Bob
// Charlie
// ...
```

---

### Your Example

```javascript
db.collection_name.find().forEach((doc) => {
  print(doc.title);
  print(doc.year);
});

// Assuming documents:
// { title: "Movie 1", year: 2020 }
// { title: "Movie 2", year: 2021 }

// Output:
// Movie 1
// 2020
// Movie 2
// 2021
```

---

### Multiple Fields Example

```javascript
db.emp.find({ department: "Sales" }).forEach((doc) => {
  print("Name: " + doc.name);
  print("Age: " + doc.age);
  print("Salary: " + doc.salary);
  print("---");
});

// Output:
// Name: Alice
// Age: 30
// Salary: 50000
// ---
// Name: Bob
// Age: 35
// Salary: 60000
// ---
```

---

### Processing Documents

```javascript
// Calculate total salary
let totalSalary = 0;

db.emp.find({ department: "Sales" }).forEach((doc) => {
  totalSalary += doc.salary;
});

print("Total Salary: " + totalSalary);
```

---

### Conditional Processing

```javascript
db.emp.find().forEach((doc) => {
  if (doc.age > 30) {
    print(doc.name + " is senior");
  } else {
    print(doc.name + " is junior");
  }
});

// Output:
// Alice is junior
// Bob is senior
// Charlie is senior
```

---

## 7. sort() — Sort Results

### What Does sort() Do?

**Sorts documents based on field values.**

---

### Syntax

```javascript
db.collection.find().sort({ fieldName: 1 or -1 })

// 1  = Ascending (A-Z, 0-9, oldest-newest)
// -1 = Descending (Z-A, 9-0, newest-oldest)
```

---

### Ascending Sort (1)

```javascript
// Sort by age (low to high)
db.emp.find().sort({ age: 1 });

// Before: [30, 25, 35, 28]
// After:  [25, 28, 30, 35]
```

---

### Descending Sort (-1)

```javascript
// Sort by salary (high to low)
db.emp.find().sort({ salary: -1 });

// Before: [50000, 70000, 60000, 80000]
// After:  [80000, 70000, 60000, 50000]
```

---

### Multiple Field Sort

```javascript
// Sort by salary descending, then name descending
db.emp.find().sort({ salary: -1, empName: -1 });

// Logic:
// 1. Sort by salary (high to low)
// 2. If salaries are equal, sort by name (Z-A)

// Example data:
// Before:
{ name: "Alice", salary: 70000 }
{ name: "Bob", salary: 70000 }
{ name: "Charlie", salary: 80000 }

// After:
{ name: "Charlie", salary: 80000 }  // Highest salary first
{ name: "Bob", salary: 70000 }      // Same salary, B before A
{ name: "Alice", salary: 70000 }
```

---

### Sort Examples

```javascript
// Sort by name (A-Z)
db.emp.find().sort({ name: 1 });

// Sort by age (oldest first)
db.emp.find().sort({ age: -1 });

// Sort by department, then salary
db.emp.find().sort({ department: 1, salary: -1 });

// Sort with projection
db.emp.find({}, { name: 1, age: 1 }).sort({ age: -1 });
```

---

### Visual: Sorting

```
Original documents:
{ name: "Charlie", age: 35 }
{ name: "Alice", age: 25 }
{ name: "Bob", age: 30 }

sort({ age: 1 }) — Ascending:
{ name: "Alice", age: 25 }    ← Youngest
{ name: "Bob", age: 30 }
{ name: "Charlie", age: 35 }  ← Oldest

sort({ age: -1 }) — Descending:
{ name: "Charlie", age: 35 }  ← Oldest
{ name: "Bob", age: 30 }
{ name: "Alice", age: 25 }    ← Youngest
```

---

## 8. limit() — Restrict Results

### What Does limit() Do?

**Restricts the output to a specified number of documents.**

---

### Syntax

```javascript
db.collection.find().limit(number);

// number = positive integer
```

---

### Basic Example

```javascript
// Get only 3 documents
db.emp.find().limit(3);

// Returns first 3 documents:
{ _id: 1, name: "Alice" }
{ _id: 2, name: "Bob" }
{ _id: 3, name: "Charlie" }
// Stops here (total 3)
```

---

### limit() with Filter

```javascript
// Get 5 employees from Sales department
db.emp.find({ department: "Sales" }).limit(5);

// Returns:
// First 5 documents where department = "Sales"
```

---

### Your Example

```javascript
// Get only 3 documents
db.emp.find({}, { salary: 1, _id: 0, empName: 1 }).limit(3);

// Returns first 3 employees with salary and name only:
{ empName: "Alice", salary: 50000 }
{ empName: "Bob", salary: 60000 }
{ empName: "Charlie", salary: 70000 }
```

---

### Use Cases

```javascript
// Use Case 1: Top 10 products
db.products.find().sort({ sales: -1 }).limit(10);

// Use Case 2: Latest 5 posts
db.posts.find().sort({ createdAt: -1 }).limit(5);

// Use Case 3: Preview results
db.users.find().limit(20); // Show first 20 users
```

---

## 9. skip() — Skip Documents

### What Does skip() Do?

**Skips a specified number of documents from the beginning.**

---

### Syntax

```javascript
db.collection.find().skip(number);

// number = positive integer
```

---

### Basic Example

```javascript
// Skip first 2 documents
db.emp.find().skip(2);

// Original order:
{ _id: 1, name: "Alice" }   ← Skip
{ _id: 2, name: "Bob" }     ← Skip
{ _id: 3, name: "Charlie" } ← Start here
{ _id: 4, name: "David" }
{ _id: 5, name: "Eve" }

// Returns:
{ _id: 3, name: "Charlie" }
{ _id: 4, name: "David" }
{ _id: 5, name: "Eve" }
```

---

### skip() with limit()

```javascript
// Skip first 5, then get next 10
db.emp.find().skip(5).limit(10);

// Documents: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]
//             ↑___Skip___↑  ↑_______Return these_______↑
// Returns documents 6-15
```

---

### Visual: skip() and limit()

```
Total documents: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

skip(3):
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
 ↑___↑   ↑___Return from here___↑
 Skip

skip(3).limit(4):
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
 ↑___↑   ↑__↑
 Skip    Return only 4

Result: [4, 5, 6, 7]
```

---

## 10. Query Chaining

### What is Query Chaining?

**Combining multiple cursor methods in sequence.**

---

### Chaining Syntax

```javascript
db.collection.find().sort({ field: -1 }).skip(10).limit(5);

// Methods execute in this order:
// 1. find()  → Get matching documents
// 2. sort()  → Sort results
// 3. skip()  → Skip first 10
// 4. limit() → Get next 5
```

---

### Execution Order

**Important:** Methods execute in this order regardless of how you write them:

```
1. sort()    → Always first
2. skip()    → Always second
3. limit()   → Always third
```

---

### Example: Different Order, Same Result

```javascript
// Write order: sort → skip → limit
db.emp.find().sort({ age: -1 }).skip(5).limit(3);

// Write order: limit → skip → sort (different!)
db.emp.find().limit(3).skip(5).sort({ age: -1 });

// Execution order is ALWAYS: sort → skip → limit
// Both queries produce the same result!
```

---

### Your Example 1: Top 3 Highest Salaries

```javascript
db.emp.find(
    {},  // No filter (all documents)
    { salary: 1, _id: 0, empName: 1 }  // Projection
)
.sort({ salary: -1, empName: -1 })  // Sort by salary desc
.limit(3);  // Get top 3

// Process:
// 1. Get all employees
// 2. Project: salary and empName only
// 3. Sort: salary high to low
// 4. Limit: top 3

// Result:
{ empName: "Alice", salary: 90000 }
{ empName: "Bob", salary: 85000 }
{ empName: "Charlie", salary: 80000 }
```

---

### Your Example 2: Second Highest Salary

```javascript
db.emp.find(
    {},
    { salary: 1, _id: 0, empName: 1 }
)
.sort({ salary: -1, empName: -1 })
.skip(1)  // Skip highest
.limit(1); // Get next one

// Process:
// 1. Sort by salary (high to low)
// 2. Skip first (highest salary)
// 3. Get next one (second highest)

// Salaries: [90000, 85000, 80000, 75000]
//           ↑ Skip  ↑ Return

// Result:
{ empName: "Bob", salary: 85000 }
```

---

### Complex Chaining Example

```javascript
// Get 5 products from page 3 (pagination)
db.products
  .find({ category: "Electronics" })
  .sort({ price: -1, rating: -1 })
  .skip(10) // Skip first 2 pages (5 × 2 = 10)
  .limit(5); // Get page 3 (5 items)

// Process:
// 1. Filter: category = "Electronics"
// 2. Sort: price desc, then rating desc
// 3. Skip: 10 (pages 1 and 2)
// 4. Limit: 5 (page 3)
```

---

## 11. Pagination Patterns

### What is Pagination?

**Breaking results into pages (like Google search results).**

---

### Pagination Formula

```
skip = (pageNumber - 1) × pageSize
limit = pageSize

Example:
Page 1: skip(0).limit(10)   → Items 1-10
Page 2: skip(10).limit(10)  → Items 11-20
Page 3: skip(20).limit(10)  → Items 21-30
```

---

### Pagination Function

```javascript
function getPage(pageNumber, pageSize) {
  let skip = (pageNumber - 1) * pageSize;

  return db.products.find().sort({ createdAt: -1 }).skip(skip).limit(pageSize);
}

// Usage:
getPage(1, 10); // Page 1: items 1-10
getPage(2, 10); // Page 2: items 11-20
getPage(3, 10); // Page 3: items 21-30
```

---

### Pagination Examples

```javascript
// Page 1 (First 10 items)
db.products.find().skip(0).limit(10);

// Page 2 (Items 11-20)
db.products.find().skip(10).limit(10);

// Page 3 (Items 21-30)
db.products.find().skip(20).limit(10);

// Page 4 (Items 31-40)
db.products.find().skip(30).limit(10);
```

---

### Visual: Pagination

```
Total: 100 documents
Page size: 10

Page 1: skip(0).limit(10)
[Doc1] [Doc2] ... [Doc10]

Page 2: skip(10).limit(10)
[Doc11] [Doc12] ... [Doc20]

Page 3: skip(20).limit(10)
[Doc21] [Doc22] ... [Doc30]

...

Page 10: skip(90).limit(10)
[Doc91] [Doc92] ... [Doc100]
```

---

### Pagination with Total Count

```javascript
// Get page data + total count
let pageNumber = 2;
let pageSize = 10;
let skip = (pageNumber - 1) * pageSize;

// Get documents for page
let documents = db.products.find()
    .sort({ price: -1 })
    .skip(skip)
    .limit(pageSize)
    .toArray();

// Get total count
let totalCount = db.products.find().count();

// Calculate total pages
let totalPages = Math.ceil(totalCount / pageSize);

// Result:
{
    page: 2,
    pageSize: 10,
    totalCount: 95,
    totalPages: 10,
    data: documents
}
```

---

## 12. Data Modeling Basics

### What is Data Modeling?

**Data modeling defines:**

1. **How data is stored** (structure)
2. **Relationships between data** (embedded vs reference)

---

### Two Key Aspects

```
1. Data Structure:
   - What fields to include
   - Field types (string, number, date)
   - Nesting level

2. Data Relationships:
   - How documents relate to each other
   - Embedded (nested) documents
   - Referenced (separate) documents
```

---

### Types of Relationships

```
1-to-1 (One to One)
- User → Profile
- Person → Passport

1-to-N (One to Many)
- User → Orders
- Author → Books

N-to-N (Many to Many)
- Students ← → Courses
- Products ← → Categories
```

---

### Embedded vs Referenced

```
EMBEDDED (Nested):
────────────────────────────────────────
Store related data INSIDE the document

{
    _id: 1,
    name: "Alice",
    address: {              ← Embedded
        city: "New York",
        state: "NY"
    }
}


REFERENCED (Separate):
────────────────────────────────────────
Store related data in SEPARATE collection

User collection:
{ _id: 1, name: "Alice", addressId: 101 }

Address collection:
{ _id: 101, city: "New York", state: "NY" }
```

---

### When to Use Embedded

```
Use embedded documents when:
✅ Data is tightly coupled
✅ Small data size
✅ Read data together
✅ One-to-few relationship
✅ Data doesn't change often

Example: User profile with address
```

---

### When to Use Referenced

```
Use referenced documents when:
✅ Data is loosely coupled
✅ Large data size
✅ Data shared across documents
✅ One-to-many relationship
✅ Data changes frequently

Example: Users and Orders (separate collections)
```

---

## 13. Embedded Documents

### What are Embedded Documents?

**Storing documents inside other documents (nesting).**

---

### Your Example 1: User with Address

```javascript
let user = {
  name: "Alice",
  age: 30,
  address: {
    // ← Embedded document
    city: "New York",
    state: "NY",
  },
  dname: "accounting",
};

// address is embedded inside user
// No separate address collection needed
```

---

### Accessing Embedded Fields

```javascript
// Insert user with embedded address
db.users.insertOne({
    name: "Alice",
    age: 30,
    address: {
        city: "New York",
        state: "NY",
        zip: "10001"
    }
});

// Query by embedded field (use dot notation)
db.users.find({ "address.city": "New York" });

// Projection of embedded field
db.users.find({}, { "address.city": 1, name: 1 });

// Result:
{ name: "Alice", address: { city: "New York" } }
```

---

### Your Example 2: Contact Info

```javascript
let c1 = {
  email: "alice@example.com",
  phone: "999999999",
};

// This could be embedded in user:
let user = {
  name: "Alice",
  contact: {
    // ← Embedded
    email: "alice@example.com",
    phone: "999999999",
  },
};
```

---

### Your Example 3: Department

```javascript
let dept = {
  name: "accounting",
  loc: "Building A",
  floor: 8,
  head: "new head",
};

// Can be embedded in employee:
let employee = {
  name: "Alice",
  department: {
    // ← Embedded
    name: "accounting",
    loc: "Building A",
    floor: 8,
    head: "new head",
  },
};

// Or referenced:
let employee = {
  name: "Alice",
  departmentId: ObjectId("..."), // ← Reference
};
```

---

### Nested Arrays Example

```javascript
// User with multiple addresses
let user = {
  name: "Alice",
  addresses: [
    // ← Array of embedded documents
    {
      type: "home",
      city: "New York",
      state: "NY",
    },
    {
      type: "work",
      city: "Boston",
      state: "MA",
    },
  ],
};

// Query addresses
db.users.find({ "addresses.type": "home" });

// Query addresses array with elemMatch
db.users.find({
  addresses: {
    $elemMatch: {
      type: "home",
      city: "New York",
    },
  },
});
```

---

### Benefits of Embedding

```
Advantages:
────────────────────────────────────────
✅ Single query to get all data
✅ Better performance (no joins)
✅ Atomic updates (all or nothing)
✅ Data locality (related data together)


Disadvantages:
────────────────────────────────────────
❌ Document size limit (16 MB)
❌ Data duplication if shared
❌ Harder to query nested data
❌ Updates can be complex
```

---

## 14. Schema Validation

### What is Schema Validation?

**Enforcing rules on document structure to ensure data consistency.**

---

### MongoDB Schema

**MongoDB is schemaless by default**, meaning:

- No fixed structure required
- Can insert any document shape
- Fields can vary between documents

---

### Why Validate Schema?

```
Without validation:
────────────────────────────────────────
{ name: "Alice", age: 30 }
{ naam: "Bob", years: 25 }  ← Different fields!
{ name: 123, age: "thirty" }  ← Wrong types!

Data is inconsistent and messy


With validation:
────────────────────────────────────────
{ name: "Alice", age: 30 }
{ name: "Bob", age: 25 }
// Rejected: { naam: "Bob" }  ← Missing required field
// Rejected: { name: 123 }    ← Wrong type

Data is consistent and clean
```

---

### Your Example: createCollection with Validator

```javascript
db.createCollection("c2", {
  validator: {
    $jsonSchema: {
      type: "object",
      required: ["name", "hasInsurance"],
      properties: {
        name: {
          type: "string",
        },
        hasInsurance: {
          type: "string",
        },
      },
    },
  },
});
```

---

### Breaking Down the Validation

```javascript
validator: {
    $jsonSchema: {
        // Document must be an object
        type: "object",

        // These fields are REQUIRED
        required: ["name", "hasInsurance"],

        // Field rules
        properties: {
            name: {
                type: "string"  // name must be string
            },
            hasInsurance: {
                type: "string"  // hasInsurance must be string
            }
        }
    }
}
```

---

### Testing the Validation

```javascript
// ✅ Valid document (all required fields, correct types)
db.c2.insertOne({
  name: "Alice",
  hasInsurance: "yes",
});
// Success!

// ❌ Invalid: missing required field
db.c2.insertOne({
  name: "Bob",
  // Missing hasInsurance
});
// Error: Document failed validation

// ❌ Invalid: wrong type
db.c2.insertOne({
  name: 123, // Should be string
  hasInsurance: "yes",
});
// Error: Document failed validation

// ❌ Invalid: missing both required fields
db.c2.insertOne({
  age: 30,
});
// Error: Document failed validation
```

---

### Common Validation Types

```javascript
// String type
{
  type: "string";
}

// Number type
{
  type: "number";
}

// Integer type
{
  type: "integer";
}

// Boolean type
{
  type: "boolean";
}

// Array type
{
  type: "array";
}

// Object type
{
  type: "object";
}

// Date type
{
  bsonType: "date";
}
```

---

### Advanced Validation Example

```javascript
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      type: "object",
      required: ["name", "email", "age"],
      properties: {
        name: {
          type: "string",
          minLength: 3,
          maxLength: 50,
        },
        email: {
          type: "string",
          pattern: "^.+@.+$", // Basic email pattern
        },
        age: {
          type: "integer",
          minimum: 18,
          maximum: 100,
        },
        status: {
          enum: ["active", "inactive", "pending"],
        },
        address: {
          type: "object",
          properties: {
            city: { type: "string" },
            zip: { type: "string" },
          },
        },
      },
    },
  },
});
```

---

### Validation Level

```javascript
// Strict validation (default)
{
    validator: { /* rules */ },
    validationLevel: "strict"  // Validates all inserts and updates
}

// Moderate validation
{
    validator: { /* rules */ },
    validationLevel: "moderate"  // Only validates documents that match rules
}
```

---

### Validation Action

```javascript
// Error action (default)
{
    validator: { /* rules */ },
    validationAction: "error"  // Rejects invalid documents
}

// Warn action
{
    validator: { /* rules */ },
    validationAction: "warn"  // Logs warning but allows invalid documents
}
```

---

## 15. Summary

### find() vs findOne()

```
find()     → Returns cursor (multiple documents)
findOne()  → Returns document (single object)
```

---

### Cursors

```
Cursor = Pointer to documents (not actual data)
Benefits: Memory efficient, batched loading
Default batch: 20 documents
```

---

### Cursor Methods

```
count()       → Count documents
forEach(fn)   → Iterate each document
sort({ f: 1}) → Sort ascending
sort({ f: -1})→ Sort descending
limit(n)      → Get first n documents
skip(n)       → Skip first n documents
```

---

### Query Chaining

```
Execution order (always):
1. sort()
2. skip()
3. limit()

Example:
db.collection.find()
    .sort({ age: -1 })
    .skip(10)
    .limit(5);
```

---

### Pagination

```
Formula:
skip = (page - 1) × pageSize
limit = pageSize

Page 1: skip(0).limit(10)
Page 2: skip(10).limit(10)
Page 3: skip(20).limit(10)
```

---

### Data Modeling

```
Embedded:   Data inside document (nested)
Referenced: Data in separate collection (linked)

Use embedded: Small, tightly coupled data
Use referenced: Large, shared data
```

---

### Schema Validation

```javascript
db.createCollection("name", {
  validator: {
    $jsonSchema: {
      required: ["field1", "field2"],
      properties: {
        field1: { type: "string" },
      },
    },
  },
});
```

---

## 16. Revision Checklist

### find() and Cursors

- [ ] Do you know difference between find() and findOne()?
- [ ] Can you explain what a cursor is?
- [ ] Do you understand cursor batching?
- [ ] Do you know default batch size (20)?
- [ ] Can you use "it" command?

### Cursor Methods

- [ ] Can you use count()?
- [ ] Can you use forEach()?
- [ ] Can you iterate documents with forEach()?
- [ ] Can you process documents in forEach()?

### Sorting

- [ ] Can you use sort() with 1 (ascending)?
- [ ] Can you use sort() with -1 (descending)?
- [ ] Can you sort by multiple fields?
- [ ] Do you know sort execution order?

### Pagination

- [ ] Can you use limit()?
- [ ] Can you use skip()?
- [ ] Can you combine skip() and limit()?
- [ ] Can you calculate pagination formula?
- [ ] Can you implement page 1, 2, 3?

### Query Chaining

- [ ] Can you chain sort(), skip(), limit()?
- [ ] Do you know execution order (sort → skip → limit)?
- [ ] Can you get top N results?
- [ ] Can you get second highest value?

### Data Modeling

- [ ] Do you understand embedded documents?
- [ ] Do you understand referenced documents?
- [ ] Do you know when to use each?
- [ ] Can you access embedded fields with dot notation?

### Schema Validation

- [ ] Can you create collection with validator?
- [ ] Can you specify required fields?
- [ ] Can you specify field types?
- [ ] Do you know common validation types?

---

> **💡 Interview Tip — "How would you implement pagination for a product listing?"**
>
> **Answer like this:**
>
> _"I would implement pagination using MongoDB's skip() and limit() methods with a formula:_
>
> ```javascript
> function getProductPage(pageNumber, pageSize) {
>   // Calculate skip amount
>   let skip = (pageNumber - 1) * pageSize;
>
>   // Get paginated results
>   let products = db.products
>     .find()
>     .sort({ createdAt: -1 }) // Newest first
>     .skip(skip)
>     .limit(pageSize);
>
>   // Get total count for UI
>   let totalCount = db.products.find().count();
>
>   return {
>     page: pageNumber,
>     pageSize: pageSize,
>     totalPages: Math.ceil(totalCount / pageSize),
>     data: products,
>   };
> }
> ```
>
> _For page 1, skip is 0, so we get items 1-10. For page 2, skip is 10, giving us items 11-20, and so on._
>
> _The sort() executes first, then skip(), then limit(), regardless of the order I write them in the query. This ensures consistent pagination._
>
> _I would also include totalCount so the UI can display total pages and navigation buttons."_
>
> **Why this works:** Shows understanding of pagination formula, query chaining, execution order, and practical implementation with total count for UI.
