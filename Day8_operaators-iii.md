# MongoDB Advanced Queries & Updates — The Ultimate Guide

### $regex, $expr, Update Operators, Upsert & Array Filtering

---

## 📚 What You'll Learn

This guide covers **advanced MongoDB query and update techniques**:

✅ $regex — Pattern matching with regular expressions  
✅ All regex patterns (^, $, ., *, \w, {n})  
✅ $expr — Comparing fields within documents  
✅ Update operators ($set, $unset, $rename, $setOnInsert, $currentDate)  
✅ upsert — Insert if not found  
✅ $elemMatch with complex conditions  
✅ Real-world examples and use cases

**Best for:** Advanced MongoDB queries, data updates, interview preparation

---

## Table of Contents

1. [$regex — Pattern Matching Basics](#1-regex--pattern-matching-basics)
2. [Regex Patterns — The Complete Guide](#2-regex-patterns--the-complete-guide)
3. [$expr — Comparing Document Fields](#3-expr--comparing-document-fields)
4. [Update Operators Overview](#4-update-operators-overview)
5. [$set — Update or Add Fields](#5-set--update-or-add-fields)
6. [$rename — Change Field Names](#6-rename--change-field-names)
7. [$unset — Delete Fields](#7-unset--delete-fields)
8. [upsert — Insert If Not Found](#8-upsert--insert-if-not-found)
9. [$setOnInsert — Set Only When Inserting](#9-setoninsert--set-only-when-inserting)
10. [$currentDate — Add Current Timestamp](#10-currentdate--add-current-timestamp)
11. [$elemMatch — Complex Array Queries](#11-elemmatch--complex-array-queries)
12. [Real-World Project Examples](#12-real-world-project-examples)
13. [Complete Reference](#13-complete-reference)
14. [Summary](#14-summary)
15. [Revision Checklist](#15-revision-checklist)

---

## 1. $regex — Pattern Matching Basics

### What is $regex?

**$regex allows you to search for patterns in string fields** using regular expressions.

Think of it like:

- **"Find all" in Word** — but with superpowers
- **Ctrl+F** on steroids
- **Advanced search** with wildcards

---

### Why Use $regex?

**Without $regex (limited):**

```javascript
// Only exact matches
db.emp.find({ empName: "alice" }); // Finds only "alice"
// Doesn't find: "Alice", "ALICE", "alice123", "alicesmith"
```

**With $regex (powerful):**

```javascript
// Pattern matching
db.emp.find({ empName: { $regex: /alice/i } });
// Finds: "alice", "Alice", "ALICE", "Alice123", "alicesmith"
```

---

### Basic Syntax (2 Ways)

```javascript
// Method 1: Regex literal (with slashes)
{
  fieldName: {
    $regex: /pattern/;
  }
}

// Method 2: String pattern
{
  fieldName: {
    $regex: "pattern";
  }
}
```

**Both work the same!** Most people use Method 1.

---

### Simple Example

```javascript
// Find employees with "a" in their name
db.emp.find(
    { empName: { $regex: /a/ } },
    { empName: 1, _id: 0 }
);

// Matches:
{ empName: "alice" }    ✅ (has "a")
{ empName: "mark" }     ✅ (has "a")
{ empName: "anna" }     ✅ (has "a")
{ empName: "bob" }      ❌ (no "a")
{ empName: "charlie" }  ✅ (has "a")
```

---

### ⚠️ Important: Case Sensitive by Default

```javascript
// Case-sensitive (default)
db.emp.find({ empName: { $regex: /alice/ } });
// Matches: "alice", "alicesmith" ✅
// Doesn't match: "Alice", "ALICE" ❌

// Case-insensitive (add "i" flag)
db.emp.find({ empName: { $regex: /alice/i } });
// Matches: "alice", "Alice", "ALICE", "ALiCe" ✅
```

**The `i` flag = ignore case**

---

## 2. Regex Patterns — The Complete Guide

### Pattern 1: Anywhere Match (Basic)

**Pattern:** `/letters/`

**What it does:** Finds the letters anywhere in the string, in order.

```javascript
// Find "ar" anywhere in name
db.emp.find({ empName: { $regex: /ar/ } });

// Matches:
"mark"      ✅ (has "ar")
"martin"    ✅ (has "ar")
"arthur"    ✅ (has "ar")
"charlie"   ✅ (has "ar")
"bob"       ❌ (no "ar")
"anna"      ❌ (no "ar" - has "a" and "n" but not together)
```

**Order matters!** `/ar/` ≠ `/ra/`

---

### Pattern 2: Starts With (^)

**Pattern:** `/^letter/`

**What it does:** Matches only if string STARTS with the pattern.

```javascript
// Find names starting with "a"
db.emp.find({ empName: { $regex: /^a/ } });

// Matches:
"alice"     ✅ (starts with "a")
"anna"      ✅ (starts with "a")
"arthur"    ✅ (starts with "a")
"mark"      ❌ (doesn't start with "a")
"barack"    ❌ (doesn't start with "a")
```

**Symbol:** `^` (Shift + 6) means "start of string"

---

### Pattern 3: Ends With ($)

**Pattern:** `/letter$/`

**What it does:** Matches only if string ENDS with the pattern.

```javascript
// Find names ending with "n"
db.emp.find({ empName: { $regex: /n$/ } });

// Matches:
"john"      ✅ (ends with "n")
"martin"    ✅ (ends with "n")
"susan"     ✅ (ends with "n")
"alice"     ❌ (ends with "e")
"anna"      ❌ (ends with "a")
```

**Symbol:** `$` (Shift + 4) means "end of string"

---

### Pattern 4: Dot (.) — Skip One Character

**Pattern:** `/.pattern/` or `/pattern./`

**What it does:** `.` matches ANY single character (wildcard).

```javascript
// Find "n" as second-to-last character
db.emp.find({ empName: { $regex: /n.$/ } });
//                              ↑ ↑
//                              n (any char)

// Matches:
"anna"      ✅ ("n" + "a" at end)
"kent"      ✅ ("n" + "t" at end)
"john"      ❌ ("n" is last, no char after)
"martin"    ❌ ("i" + "n" at end, not "n" + something)
```

**Another example:**

```javascript
// Find names where 3rd letter is "l"
db.emp.find({ empName: { $regex: /^..l/ } });
//                              ↑↑↑
//                            (any)(any)l

// Matches:
"alice"     ✅ (a-l-i = 3rd is "i", wait... let me recheck)
// Actually: ^..l means: start, any, any, then "l" as 3rd char
"allison"   ✅ (a-l-l = 3rd is "l")
"allen"     ✅ (a-l-l = 3rd is "l")
"bob"       ❌ (only 3 chars, no "l" as 3rd)
```

---

### Pattern 5: Exact Length

**Pattern:** `/^\w{n}$/` or `/^.{n}$/`

**What it does:** Matches strings with EXACTLY n characters.

```javascript
// Find names with exactly 4 letters
db.emp.find({ empName: { $regex: /^....$/ } });
//                              ↑↑↑↑
//                            4 dots = 4 chars

// Or using \w (word character):
db.emp.find({ empName: { $regex: /^\w{4}$/ } });
//                              ↑    ↑
//                          start {4} end

// Matches:
"john"      ✅ (4 letters)
"anna"      ✅ (4 letters)
"mark"      ✅ (4 letters)
"alice"     ❌ (5 letters)
"bob"       ❌ (3 letters)
```

**Symbols:**

- `\w` = word character (letter, digit, underscore)
- `{n}` = exactly n times
- `^` and `$` = start and end (ensures EXACT length)

---

### Pattern 6: Starts and Ends With

**Pattern:** `/^start.*end$/`

**What it does:** Starts with X, ends with Y, anything in between.

```javascript
// Find names starting with "a" and ending with "s"
db.emp.find({ empName: { $regex: /^a.*s$/ } });
//                              ↑ ↑↑↑
//                            start a (any chars) s end

// Matches:
"adams"     ✅ (a...s)
"alias"     ✅ (a...s)
"amos"      ✅ (a...s)
"alice"     ❌ (starts with "a" but ends with "e")
"thomas"    ❌ (ends with "s" but starts with "t")
```

**Symbol:** `*` means "0 or more of anything"

---

### Visual: All Patterns

```
Text: "alice"

/a/       ✅ "alice"    (has "a" anywhere)
/^a/      ✅ "alice"    (starts with "a")
/e$/      ✅ "alice"    (ends with "e")
/n$/      ❌ "alice"    (doesn't end with "n")
/^..i/    ✅ "alice"    (3rd char is "i": a-l-i)
/^\w{5}$/ ✅ "alice"    (exactly 5 letters)
/^a.*e$/  ✅ "alice"    (starts "a", ends "e")
```

---

### Common Regex Patterns Reference

| Pattern     | Meaning              | Example                 | Matches           |
| ----------- | -------------------- | ----------------------- | ----------------- |
| `/a/`       | Contains "a"         | `{ $regex: /a/ }`       | "mark", "anna"    |
| `/^a/`      | Starts with "a"      | `{ $regex: /^a/ }`      | "alice", "anna"   |
| `/n$/`      | Ends with "n"        | `{ $regex: /n$/ }`      | "john", "susan"   |
| `/^.../`    | First 3 chars        | `{ $regex: /^ali/ }`    | "alice", "alison" |
| `/...$/ `   | Last 3 chars         | `{ $regex: /ice$/ }`    | "alice", "price"  |
| `/^\w{4}$/` | Exactly 4 chars      | `{ $regex: /^\w{4}$/ }` | "john", "anna"    |
| `/^a.*n$/`  | Starts "a", ends "n" | `{ $regex: /^a.*n$/ }`  | "adrian", "allan" |
| `/[aeiou]/` | Contains vowel       | `{ $regex: /[aeiou]/ }` | "alice", "bob"    |
| `/^[sk]/`   | Starts with s or k   | `{ $regex: /^[sk]/ }`   | "smith", "king"   |

---

### Case-Insensitive Flag

```javascript
// Without "i" — case sensitive
db.emp.find({ empName: { $regex: /alice/ } });
// Matches: "alice" ✅, "Alice" ❌

// With "i" — case insensitive
db.emp.find({ empName: { $regex: /alice/i } });
// Matches: "alice" ✅, "Alice" ✅, "ALICE" ✅, "ALiCe" ✅
```

---

## 3. $expr — Comparing Document Fields

### What is $expr?

**$expr allows you to compare TWO FIELDS within the same document.**

Think of it like:

- **"Find people taller than their age"** (height > age)
- **"Products where discount > originalPrice"**
- **"Employees where commission > bonus"**

---

### Why Do We Need $expr?

**Without $expr (doesn't work):**

```javascript
// ❌ This doesn't work
db.emp.find({ comm: { $gt: bonus } });
// Error: "bonus" is not defined (JavaScript variable)

// MongoDB thinks you're looking for:
db.emp.find({ comm: { $gt: undefined } });
```

**With $expr (works!):**

```javascript
// ✅ This works
db.emp.find({ $expr: { $gt: ["$comm", "$bonus"] } });
// MongoDB compares comm field to bonus field within each document
```

---

### Syntax

```javascript
{
  $expr: {
    $comparisonOperator: ["$field1", "$field2"];
  }
}
```

**CRITICAL:** Field names must:

1. Be in quotes: `"$comm"`
2. Start with $: `"$field"`not just`"field"`

---

### Basic Example

```javascript
// Document:
{
    empName: "Alice",
    comm: 500,
    bonus: 300
}

// Find employees where comm > bonus
db.emp.find(
    { $expr: { $gt: ["$comm", "$bonus"] } },
    { empName: 1, comm: 1, bonus: 1, _id: 0 }
);

// Returns:
{ empName: "Alice", comm: 500, bonus: 300 }  ✅ (500 > 300)
```

---

### How $expr Works — Step by Step

```
Document: { empName: "Alice", comm: 500, bonus: 300 }

Query: { $expr: { $gt: ["$comm", "$bonus"] } }

Step 1: Get value of "$comm" → 500
Step 2: Get value of "$bonus" → 300
Step 3: Compare: 500 > 300?
Step 4: Yes! → Include this document

Result: ✅ Match!
```

---

### Testing with Constants

```javascript
// Compare constant values (for testing)
db.emp.find({ $expr: { $gt: [20, 10] } });
// 20 > 10? ✅ Yes → Returns ALL documents (always true)

db.emp.find({ $expr: { $gt: [10, 20] } });
// 10 > 20? ❌ No → Returns NO documents (always false)
```

**This shows $expr works, but using constants is pointless!**

---

### Real Example — Age vs Department

```javascript
// Find employees where age < deptNo
// (Unusual but demonstrates the concept)

db.emp.find(
  { $expr: { $lt: ["$age", "$deptNo"] } },
  { empName: 1, age: 1, deptNo: 1, _id: 0 },
);

// Document: { empName: "Bob", age: 18, deptNo: 20 }
// 18 < 20? ✅ Match!

// Document: { empName: "Alice", age: 35, deptNo: 10 }
// 35 < 10? ❌ No match
```

---

### All Comparison Operators with $expr

| Operator | Meaning          | Example                                       |
| -------- | ---------------- | --------------------------------------------- |
| `$gt`    | Greater than     | `{ $expr: { $gt: ["$field1", "$field2"] } }`  |
| `$gte`   | Greater or equal | `{ $expr: { $gte: ["$field1", "$field2"] } }` |
| `$lt`    | Less than        | `{ $expr: { $lt: ["$field1", "$field2"] } }`  |
| `$lte`   | Less or equal    | `{ $expr: { $lte: ["$field1", "$field2"] } }` |
| `$eq`    | Equal            | `{ $expr: { $eq: ["$field1", "$field2"] } }`  |
| `$ne`    | Not equal        | `{ $expr: { $ne: ["$field1", "$field2"] } }`  |

---

### Visual: $expr Comparison

```
Without $expr (comparing to constant):
──────────────────────────────────────
{ comm: { $gt: 300 } }

Document: { comm: 500, bonus: 200 }
Compare: 500 > 300? ✅

With $expr (comparing to field):
──────────────────────────────────────
{ $expr: { $gt: ["$comm", "$bonus"] } }

Document: { comm: 500, bonus: 200 }
Step 1: Get $comm = 500
Step 2: Get $bonus = 200
Compare: 500 > 200? ✅
```

---

### Real-World Example: E-Commerce

```javascript
// Products collection:
{
    name: "Laptop",
    originalPrice: 1500,
    discountPrice: 1200,
    stock: 50,
    reserved: 45
}

// Find products where discount > 20% of original
db.products.find({
    $expr: {
        $gt: [
            "$originalPrice",
            { $multiply: ["$discountPrice", 1.2] }
        ]
    }
});

// Find products where reserved stock > 90% of total stock
db.products.find({
    $expr: {
        $gt: [
            "$reserved",
            { $multiply: ["$stock", 0.9] }
        ]
    }
});
```

---

## 4. Update Operators Overview

### What Are Update Operators?

**Special operators that modify documents** in different ways.

Think of them like:

- **Tools in a toolbox** — each for a specific job
- **Edit menu** in Word (copy, paste, delete, rename)
- **Modification commands**

---

### The 5 Main Update Operators

```
$set            → Update or add fields
$rename         → Change field names
$unset          → Delete fields
$setOnInsert    → Set only when inserting new document
$currentDate    → Set to current date/time
```

---

### ⚠️ Critical Rule: Cannot Edit \_id

```javascript
// ❌ WRONG — Can't change _id value
db.users.updateOne(
  { _id: ObjectId("123") },
  { $set: { _id: ObjectId("456") } },
);
// Error: "Performing an update on the path '_id' would modify the immutable field '_id'"

// ❌ WRONG — Can't rename _id field
db.users.updateOne({ _id: ObjectId("123") }, { $rename: { _id: "userId" } });
// Error!

// ✅ CORRECT — Can change other fields
db.users.updateOne({ _id: ObjectId("123") }, { $set: { name: "Alice" } });
```

**\_id is IMMUTABLE = Cannot be changed after document creation!**

---

## 5. $set — Update or Add Fields

### What Does $set Do?

**Updates existing fields OR adds new fields if they don't exist.**

Think of it like:

- **"Save As"** in Word (overwrites or creates)
- **Upsert for fields** (update or insert)
- **Most commonly used update operator**

---

### Syntax

```javascript
{
    $set: {
        field1: newValue1,
        field2: newValue2,
        field3: newValue3
    }
}
```

---

### Example 1: Updating Existing Field

```javascript
// Before:
{ dept: 10, loc: "NEW YORK", budget: 50000 }

// Update:
db.dept.updateOne(
    { dept: 10 },
    { $set: { budget: 78723 } }
);

// After:
{ dept: 10, loc: "NEW YORK", budget: 78723 }
//                              ↑ Changed!
```

---

### Example 2: Adding New Field

```javascript
// Before:
{ dept: 10, loc: "NEW YORK", budget: 78723 }

// Update (floor doesn't exist):
db.dept.updateOne(
    { dept: 10 },
    { $set: { floor: 5 } }
);

// After:
{ dept: 10, loc: "NEW YORK", budget: 78723, floor: 5 }
//                                           ↑ Added!
```

---

### Example 3: Multiple Fields at Once

```javascript
// Before:
{ dept: 10, loc: "NEW YORK" }

// Update multiple fields:
db.dept.updateOne(
    { dept: 10 },
    {
        $set: {
            loc: "BOSTON",           // Update existing
            budget: 100000,          // Add new
            floor: 3,                // Add new
            manager: "Alice Smith"   // Add new
        }
    }
);

// After:
{
    dept: 10,
    loc: "BOSTON",              // ✅ Updated
    budget: 100000,             // ✅ Added
    floor: 3,                   // ✅ Added
    manager: "Alice Smith"      // ✅ Added
}
```

---

### Return Value

```javascript
let response = db.dept.updateOne(
    { dept: 10 },
    { $set: { budget: 78723 } }
);

console.log(response);

// Output:
{
    acknowledged: true,     // MongoDB received command
    matchedCount: 1,        // 1 document matched filter
    modifiedCount: 1        // 1 document was changed
}
```

**If no changes:**

```javascript
// Same value update (no actual change)
{
    acknowledged: true,
    matchedCount: 1,        // Found the document
    modifiedCount: 0        // But nothing changed
}
```

---

### Real-World Example: User Profile Update

```javascript
// User clicks "Save Profile"

db.users.updateOne(
  { email: "alice@example.com" },
  {
    $set: {
      name: "Alice Johnson", // Update
      age: 30, // Update
      city: "San Francisco", // Add new
      lastModified: new Date(), // Add timestamp
    },
  },
);
```

---

## 6. $rename — Change Field Names

### What Does $rename Do?

**Changes the name of a field** (key), keeping the value.

Think of it like:

- **"Rename file"** in Windows
- **Changing variable name** in refactoring
- **Fixing typos** in field names

---

### Syntax

```javascript
{
  $rename: {
    oldFieldName: "newFieldName";
  }
}
```

---

### Basic Example

```javascript
// Before:
{ "user-name": "Rajesh Kumar", email: "rajesh@example.com" }

// Rename "email" to "user-email":
db.users.updateOne(
    { "user-name": "Rajesh Kumar" },
    { $rename: { email: "user-email" } }
);

// After:
{ "user-name": "Rajesh Kumar", "user-email": "rajesh@example.com" }
//                              ↑ Field name changed!
```

---

### Multiple Renames

```javascript
// Before:
{ firstName: "Alice", lastName: "Smith", eMail: "alice@example.com" }

// Rename multiple fields:
db.users.updateOne(
    { firstName: "Alice" },
    {
        $rename: {
            firstName: "first_name",   // Camel case → snake_case
            lastName: "last_name",
            eMail: "email"             // Fix typo
        }
    }
);

// After:
{ first_name: "Alice", last_name: "Smith", email: "alice@example.com" }
```

---

### ⚠️ If Field Doesn't Exist

```javascript
// Document: { name: "Alice" }

// Try to rename non-existent field:
db.users.updateOne({ name: "Alice" }, { $rename: { age: "userAge" } });

// Result: No error, no change
// { name: "Alice" }  (age field didn't exist, so nothing happens)
```

---

### Real-World Example: Schema Migration

```javascript
// Old schema had inconsistent naming
// Fix all user documents:

db.users.updateMany(
  {}, // All documents
  {
    $rename: {
      "user-name": "username", // Kebab → camel
      "user-email": "email", // Simplify
      "user-phone": "phone", // Simplify
      is_active: "isActive", // Snake → camel
    },
  },
);

// This updates ALL users to new consistent naming scheme
```

---

## 7. $unset — Delete Fields

### What Does $unset Do?

**Removes fields (key-value pairs) from documents.**

Think of it like:

- **"Delete column"** in Excel
- **Removing properties** from object
- **Cleaning up** unwanted data

---

### Syntax

```javascript
{
    $unset: {
        field1: "",    // Value doesn't matter (use "")
        field2: "",
        field3: ""
    }
}
```

**Note:** The value (e.g., `""`) doesn't matter — MongoDB ignores it and just removes the field.

---

### Basic Example

```javascript
// Before:
{ "user-name": "Rajesh Kumar", email: "rajesh@example.com", age: 30 }

// Remove "age" field:
db.users.updateOne(
    { "user-name": "Rajesh Kumar" },
    { $unset: { age: "" } }
);

// After:
{ "user-name": "Rajesh Kumar", email: "rajesh@example.com" }
//                                                        ↑ age removed!
```

---

### Multiple Fields

```javascript
// Before:
{
    name: "Alice",
    age: 30,
    tempPassword: "abc123",
    resetToken: "xyz789",
    lastLogin: ISODate("2024-01-15")
}

// Remove temporary/sensitive fields:
db.users.updateOne(
    { name: "Alice" },
    {
        $unset: {
            tempPassword: "",    // Remove
            resetToken: ""       // Remove
        }
    }
);

// After:
{
    name: "Alice",
    age: 30,
    lastLogin: ISODate("2024-01-15")
}
```

---

### ⚠️ If Field Doesn't Exist

```javascript
// Document: { name: "Alice" }

// Try to remove non-existent field:
db.users.updateOne({ name: "Alice" }, { $unset: { age: "" } });

// Result: No error, no change
// { name: "Alice" }  (age didn't exist, so nothing happens)
```

---

### Real-World Example: GDPR Data Cleanup

```javascript
// User requests data deletion (GDPR compliance)
// Remove all personal identifiable information:

db.users.updateOne(
    { userId: 12345 },
    {
        $unset: {
            email: "",
            phone: "",
            address: "",
            ssn: "",
            creditCard: ""
        }
    }
);

// After: User exists but all PII removed
{
    userId: 12345,
    accountCreated: ISODate("2020-01-01"),
    accountStatus: "deleted"
}
```

---

## 8. upsert — Insert If Not Found

### What is upsert?

**upsert = Update + Insert**

- If document **exists** → UPDATE it
- If document **doesn't exist** → INSERT it

Think of it like:

- **"Save or Create"**
- **"Update or Add"**
- **"Ensure this document exists"**

---

### Syntax

```javascript
db.collection.updateOne(
  { filter },
  { $set: { updates } },
  { upsert: true }, // ← This enables upsert
);
```

**Default:** `upsert: false` (no auto-insert)

---

### Scenario 1: Document Exists

```javascript
// Document exists:
{ email: "alice@example.com", name: "Alice", age: 28 }

// Update with upsert: true
db.users.updateOne(
    { email: "alice@example.com" },
    { $set: { age: 30 } },
    { upsert: true }
);

// Result: UPDATES existing document
{ email: "alice@example.com", name: "Alice", age: 30 }
//                                            ↑ Updated!

// Return value:
{
    acknowledged: true,
    matchedCount: 1,       // Found existing
    modifiedCount: 1,      // Updated it
    upsertedCount: 0       // No insert
}
```

---

### Scenario 2: Document Doesn't Exist (upsert: true)

```javascript
// Document doesn't exist

// Update with upsert: true
db.users.updateOne(
    { email: "bob@example.com" },
    { $set: { name: "Bob", age: 25 } },
    { upsert: true }
);

// Result: INSERTS new document
{ email: "bob@example.com", name: "Bob", age: 25, _id: ObjectId("...") }
//        ↑ From filter          ↑ From $set      ↑ Auto-generated

// Return value:
{
    acknowledged: true,
    matchedCount: 0,           // Didn't find existing
    modifiedCount: 0,          // No update
    upsertedCount: 1,          // Inserted new!
    upsertedId: ObjectId("...")
}
```

---

### Scenario 3: Document Doesn't Exist (upsert: false)

```javascript
// Document doesn't exist

// Update with upsert: false (or omitted)
db.users.updateOne(
    { email: "charlie@example.com" },
    { $set: { name: "Charlie", age: 35 } },
    { upsert: false }    // Or omit this line
);

// Result: NOTHING happens (no insert, no update)

// Return value:
{
    acknowledged: true,
    matchedCount: 0,       // Didn't find
    modifiedCount: 0       // Didn't update
    // No upsertedCount because upsert was false
}
```

---

### Visual Decision Tree

```
Query: { email: "user@example.com" }

             Document exists?
                   ↓
          ┌────────┴────────┐
         YES               NO
          ↓                 ↓
    UPDATE document    upsert: true?
                            ↓
                     ┌──────┴──────┐
                    YES            NO
                     ↓              ↓
               INSERT new    Do nothing
               document
```

---

### Complete Example with All Fields

```javascript
// Trying to update user that doesn't exist
db.students.updateOne(
    {
        email: "new@example.com",    // Filter (user not found)
        age: 20,
        isActive: true
    },
    {
        $set: {
            phone: "123-456-7890",
            enrolled: true
        }
    },
    { upsert: true }
);

// Result: New document created with:
{
    _id: ObjectId("..."),        // Auto-generated
    email: "new@example.com",    // From filter
    age: 20,                     // From filter
    isActive: true,              // From filter
    phone: "123-456-7890",       // From $set
    enrolled: true               // From $set
}
```

**Key insight:** Filter fields + $set fields = Complete new document

---

### Real-World Example: Page View Counter

```javascript
// Track page views
// If page doesn't exist, create it with count = 1
// If page exists, increment count

db.pageViews.updateOne(
  { url: "/about" },
  {
    $inc: { views: 1 }, // Increment views
    $set: { lastViewed: new Date() },
  },
  { upsert: true },
);

// First visit: Creates { url: "/about", views: 1, lastViewed: ... }
// Second visit: Updates { url: "/about", views: 2, lastViewed: ... }
// Third visit: Updates { url: "/about", views: 3, lastViewed: ... }
```

---

## 9. $setOnInsert — Set Only When Inserting

### What Does $setOnInsert Do?

**Sets fields ONLY when a NEW document is inserted** (during upsert).

Think of it like:

- **"Default values for new records"**
- **"Initialize only once"**
- **"Set creation metadata"**

---

### When It Runs

```
Document exists?
  → $set runs, $setOnInsert is IGNORED

Document doesn't exist (upsert creates new)?
  → $set runs, $setOnInsert ALSO runs
```

---

### Syntax

```javascript
{
    $set: { alwaysUpdate },
    $setOnInsert: { onlyOnInsert }
}
```

---

### Example 1: User Fee Receipt

```javascript
// Old student exists in database
db.students.updateOne(
    { username: "Priya Gupta" },
    {
        $set: { feeReceipt: 16000 },        // Always update
        $setOnInsert: { id: 8 }             // Only set if inserting
    },
    { upsert: true }
);

// Result: Student exists, so only feeReceipt updated
{
    username: "Priya Gupta",
    feeReceipt: 16000,      // ✅ Updated (was 15000 → now 16000)
    id: 7                   // ❌ NOT changed (still 7, not 8)
}

// Return:
{
    matchedCount: 1,
    modifiedCount: 1,
    upsertedCount: 0        // No insert, so $setOnInsert ignored
}
```

---

### Example 2: New Student

```javascript
// New student doesn't exist
db.students.updateOne(
    { username: "Amit Kumar" },  // Not found
    {
        $set: { feeReceipt: 18000 },
        $setOnInsert: { id: 9, enrollDate: new Date() }
    },
    { upsert: true }
);

// Result: New student created with BOTH $set and $setOnInsert
{
    _id: ObjectId("..."),
    username: "Amit Kumar",      // From filter
    feeReceipt: 18000,           // From $set
    id: 9,                       // From $setOnInsert ✅
    enrollDate: ISODate("...")   // From $setOnInsert ✅
}

// Return:
{
    matchedCount: 0,
    modifiedCount: 0,
    upsertedCount: 1            // Insert happened, so $setOnInsert ran
}
```

---

### Comparison: $set vs $setOnInsert

```javascript
// Scenario 1: Document exists

db.users.updateOne(
    { email: "alice@example.com" },  // Found
    {
        $set: { lastLogin: new Date() },
        $setOnInsert: { createdAt: new Date() }
    },
    { upsert: true }
);

// Result:
{
    email: "alice@example.com",
    lastLogin: ISODate("2024-02-21"),    // ✅ Updated (always)
    createdAt: ISODate("2024-01-01")     // ❌ Not changed (was already set)
}
```

```javascript
// Scenario 2: Document doesn't exist

db.users.updateOne(
    { email: "bob@example.com" },  // Not found
    {
        $set: { lastLogin: new Date() },
        $setOnInsert: { createdAt: new Date() }
    },
    { upsert: true }
);

// Result:
{
    email: "bob@example.com",
    lastLogin: ISODate("2024-02-21"),    // ✅ Set
    createdAt: ISODate("2024-02-21")     // ✅ Set (first time!)
}
```

---

### ⚠️ Cannot Change \_id with $setOnInsert

```javascript
// ❌ WRONG — Can't set _id
db.students.updateOne(
  { _id: ObjectId("6971e7d8fcb2b0e3f7735119") },
  {
    $set: { subjects: "PCM" },
    $setOnInsert: { _id: "12345" }, // ERROR!
  },
  { upsert: true },
);

// Error: "_id" is immutable
```

---

### Real-World Example: User Registration

```javascript
// User logs in for first time (OAuth)
// If new: Set creation date and default preferences
// If existing: Just update last login

db.users.updateOne(
  { oauthId: "google-123456" },
  {
    $set: {
      lastLogin: new Date(),
      loginCount: { $inc: 1 },
    },
    $setOnInsert: {
      createdAt: new Date(),
      preferences: {
        theme: "light",
        notifications: true,
      },
      role: "user",
    },
  },
  { upsert: true },
);

// First login: ALL fields set
// Subsequent logins: Only lastLogin and loginCount updated
```

---

## 10. $currentDate — Add Current Timestamp

### What Does $currentDate Do?

**Sets a field to the current date/time** (right now).

Think of it like:

- **Auto timestamp**
- **"Last modified" tracker**
- **Audit trail**

---

### Syntax

```javascript
{
  $currentDate: {
    fieldName: true; // Sets to current date
  }
}
```

---

### Basic Example

```javascript
// Before:
{ empName: "smith", hireDate: ISODate("1981-12-01") }

// Update with current date:
db.emp.updateOne(
    { empName: "smith" },
    { $currentDate: { lastIncrement: true } }
);

// After:
{
    empName: "smith",
    hireDate: ISODate("1981-12-01"),
    lastIncrement: ISODate("2024-02-21T10:30:00Z")  // ✅ Added!
}
```

---

### vs new Date()

```javascript
// Method 1: $currentDate (MongoDB sets date)
db.emp.updateMany(
  { hireDate: { $lt: ISODate("1982-01-01") } },
  { $currentDate: { lastIncrement: true } },
);

// Method 2: new Date() (Node.js sets date)
db.emp.updateMany(
  { hireDate: { $lt: ISODate("1982-01-01") } },
  { $set: { lastIncrement: new Date() } },
);

// Both work the same!
// Difference: $currentDate runs on MongoDB server
//             new Date() runs on your Node.js server
```

**Use $currentDate for accuracy** (MongoDB's clock, not yours).

---

### Real-World Example: Audit Trail

```javascript
// Update user profile and track when
db.users.updateOne(
    { userId: 123 },
    {
        $set: {
            email: "newemail@example.com",
            phone: "555-1234"
        },
        $currentDate: {
            lastModified: true,
            emailChanged: true
        }
    }
);

// Result:
{
    userId: 123,
    email: "newemail@example.com",
    phone: "555-1234",
    lastModified: ISODate("2024-02-21T10:30:00Z"),
    emailChanged: ISODate("2024-02-21T10:30:00Z")
}
```

---

## 11. $elemMatch — Complex Array Queries

### Review: Basic $elemMatch

**From previous guide:** $elemMatch finds documents where at least ONE array element satisfies ALL conditions.

---

### Your Complex Example

```javascript
// Survey collection:
{
    _id: 1,
    results: [
        { product: "abc", score: 10 },
        { product: "xyz", score: 5 }
    ]
}

// Find docs where product = "def" AND score > 7
db.survey.find({
    results: {
        $elemMatch: {
            product: "def",
            score: { $gt: 7 }
        }
    }
});
```

---

### With $and Inside $elemMatch

```javascript
// Explicit $and (redundant but valid)
db.survey.find({
  results: {
    $elemMatch: {
      $and: [{ product: "def" }, { score: { $gt: 7 } }],
    },
  },
});

// Same as:
db.survey.find({
  results: {
    $elemMatch: {
      product: "def",
      score: { $gt: 7 },
    },
  },
});
```

**Both are equivalent!** $and is implicit when you list multiple conditions.

---

### ❌ Wrong Syntax

```javascript
// ❌ WRONG — $elemMatch and score at same level
db.survey.find({
  results: {
    $elemMatch: { product: "def" },
    score: { $gt: 7 }, // This is OUTSIDE $elemMatch!
  },
});

// This doesn't work because:
// 1. $elemMatch checks array elements
// 2. score is at the wrong level
```

**The `score` condition must be INSIDE $elemMatch!**

---

### Complete Working Example

```javascript
// Data:
[
  {
    _id: 1,
    results: [
      { product: "abc", score: 10 },
      { product: "xyz", score: 5 },
    ],
  },
  {
    _id: 4,
    results: [
      { product: "abc", score: 7 },
      { product: "def", score: 8 }, // ✅ Matches!
    ],
  },
];

// Query: Find product="def" with score > 7
db.survey.find({
  results: {
    $elemMatch: {
      product: "def",
      score: { $gt: 7 },
    },
  },
});

// Returns: Document with _id: 4
// Because results array has ONE element that matches BOTH conditions
```

---

## 12. Real-World Project Examples

### Project 1: E-Commerce — Search & Updates

```javascript
// ===== SEARCH =====

// Find products whose name contains "laptop"
db.products.find({ name: { $regex: /laptop/i } });

// Find products starting with "mac"
db.products.find({ name: { $regex: /^mac/i } });

// ===== PRICE COMPARISON =====

// Find products where discount price > 20% savings
db.products.find({
  $expr: {
    $gt: [
      { $subtract: ["$originalPrice", "$salePrice"] },
      { $multiply: ["$originalPrice", 0.2] },
    ],
  },
});

// ===== UPDATES =====

// Update stock
db.products.updateOne(
  { sku: "LAPTOP-001" },
  {
    $set: { stock: 45 },
    $currentDate: { lastUpdated: true },
  },
);

// Add new product or update if exists
db.products.updateOne(
  { sku: "LAPTOP-002" },
  {
    $set: {
      name: "MacBook Pro",
      price: 2499,
      stock: 10,
    },
    $setOnInsert: {
      createdAt: new Date(),
      reviews: [],
    },
  },
  { upsert: true },
);
```

---

### Project 2: HR System — Employee Management

```javascript
// ===== SEARCH =====

// Find employees with "manager" in title
db.employees.find({ jobTitle: { $regex: /manager/i } });

// Find employees starting with S or K
db.employees.find({ empName: { $regex: /^[sk]/i } });

// ===== COMPARISONS =====

// Find employees where commission > bonus
db.employees.find({
  $expr: { $gt: ["$commission", "$bonus"] },
});

// ===== UPDATES =====

// Give increment to employees hired before 1982
db.employees.updateMany(
  { hireDate: { $lt: ISODate("1982-01-01") } },
  {
    $set: { salaryIncrement: 5 },
    $currentDate: { lastIncrementDate: true },
  },
);

// Rename field across all documents
db.employees.updateMany({}, { $rename: { "emp-name": "employeeName" } });
```

---

### Project 3: Content Platform — Articles & Reviews

```javascript
// ===== SEARCH =====

// Find articles with exactly 5-letter titles
db.articles.find({ title: { $regex: /^\w{5}$/ } });

// Find articles with "react" anywhere in title
db.articles.find({ title: { $regex: /react/i } });

// ===== ARRAY QUERIES =====

// Find articles with highly-rated reviews
db.articles.find({
  reviews: {
    $elemMatch: {
      rating: { $gte: 4 },
      verified: true,
    },
  },
});

// ===== UPDATES =====

// Track article views
db.articles.updateOne(
  { slug: "intro-to-mongodb" },
  {
    $inc: { views: 1 },
    $currentDate: { lastViewed: true },
  },
  { upsert: true },
);

// Remove draft fields when publishing
db.articles.updateOne(
  { _id: ObjectId("...") },
  {
    $set: { status: "published" },
    $unset: { draftContent: "", editorNotes: "" },
    $currentDate: { publishedAt: true },
  },
);
```

---

## 13. Complete Reference

### $regex Patterns

| Pattern     | Matches                                 | Example                |
| ----------- | --------------------------------------- | ---------------------- |
| `/text/`    | Contains "text"                         | "context", "textbook"  |
| `/^text/`   | Starts with "text"                      | "textbook", "text123"  |
| `/text$/`   | Ends with "text"                        | "contest", "hypertext" |
| `/^.ext/`   | 2nd char is "e", 3rd is "x", 4th is "t" | "text", "next"         |
| `/^\w{4}$/` | Exactly 4 word characters               | "text", "1234"         |
| `/^a.*z$/`  | Starts "a", ends "z"                    | "arizona", "amaze"     |
| `//i`       | Case insensitive                        | "Text", "TEXT", "text" |

---

### $expr Operators

| Operator | Example                                       |
| -------- | --------------------------------------------- |
| `$gt`    | `{ $expr: { $gt: ["$field1", "$field2"] } }`  |
| `$gte`   | `{ $expr: { $gte: ["$field1", "$field2"] } }` |
| `$lt`    | `{ $expr: { $lt: ["$field1", "$field2"] } }`  |
| `$lte`   | `{ $expr: { $lte: ["$field1", "$field2"] } }` |
| `$eq`    | `{ $expr: { $eq: ["$field1", "$field2"] } }`  |
| `$ne`    | `{ $expr: { $ne: ["$field1", "$field2"] } }`  |

---

### Update Operators

| Operator       | Purpose            | Syntax                               |
| -------------- | ------------------ | ------------------------------------ |
| `$set`         | Update/add fields  | `{ $set: { field: value } }`         |
| `$rename`      | Change field name  | `{ $rename: { old: "new" } }`        |
| `$unset`       | Remove fields      | `{ $unset: { field: "" } }`          |
| `$setOnInsert` | Set only on insert | `{ $setOnInsert: { field: value } }` |
| `$currentDate` | Set to now         | `{ $currentDate: { field: true } }`  |

---

## 14. Summary — Key Takeaways

### 🔍 $regex

```
Anywhere:      /text/
Starts with:   /^text/
Ends with:     /text$/
Exact length:  /^\w{4}$/
Case-ignore:   /text/i
```

---

### 🔄 $expr

```javascript
// Compare two fields
{
  $expr: {
    $gt: ["$field1", "$field2"];
  }
}

// MUST use "$field" syntax (with $ and quotes)
```

---

### ✏️ Update Operators

```
$set          → Update or add
$rename       → Change name
$unset        → Remove
$setOnInsert  → Only on insert
$currentDate  → Timestamp
```

---

### 🔀 upsert

```
Document exists?
  YES → Update
  NO  → Insert (if upsert: true)
```

---

## 15. Revision Checklist

### $regex

- [ ] Can you find text containing a pattern?
- [ ] Can you find text starting with a letter?
- [ ] Can you find text ending with a letter?
- [ ] Can you use dot (.) to skip characters?
- [ ] Can you find exact length strings?
- [ ] Can you use case-insensitive search?

### $expr

- [ ] Can you compare two fields?
- [ ] Do you know to use "$field" syntax?
- [ ] Can you use all comparison operators?
- [ ] Do you know when $expr is needed?

### $set

- [ ] Can you update existing fields?
- [ ] Can you add new fields?
- [ ] Can you update multiple fields at once?

### $rename

- [ ] Can you change field names?
- [ ] Can you rename multiple fields?

### $unset

- [ ] Can you remove fields?
- [ ] Do you know value doesn't matter?

### upsert

- [ ] Can you explain upsert behavior?
- [ ] Do you know the two scenarios?
- [ ] Can you use upsert: true?

### $setOnInsert

- [ ] Do you know when it runs?
- [ ] Can you combine with $set?

### $currentDate

- [ ] Can you add current timestamp?
- [ ] Do you know vs new Date()?

---

> **🎤 Interview Tip — "How would you implement a page view counter that creates a new record if the page doesn't exist?"**
>
> **Answer like this:**
>
> _"I'd use MongoDB's upsert feature combined with the $inc operator for the counter. Here's how:_
>
> ```javascript
> db.pageViews.updateOne(
>   { url: "/about" }, // Find this page
>   {
>     $inc: { views: 1 }, // Increment views by 1
>     $currentDate: { lastViewed: true }, // Update timestamp
>   },
>   { upsert: true }, // Create if doesn't exist
> );
> ```
>
> _This handles both cases automatically. If the document exists, it increments the view count and updates the timestamp. If it doesn't exist, MongoDB creates a new document with the URL from the filter, sets views to 1 (via $inc), and sets lastViewed to the current date._
>
> _The beauty of upsert is that it's atomic — it prevents race conditions where two simultaneous requests might both try to create the same document. MongoDB guarantees exactly one document will be created, even under concurrent access."_
>
> **Why this works:** Shows understanding of upsert, $inc, $currentDate, and bonus points for mentioning atomicity and race conditions.
