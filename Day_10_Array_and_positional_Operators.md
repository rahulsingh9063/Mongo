# MongoDB Array Removal & Positional Operators

### Complete Guide to $pop, $pull, $pullAll, $, $[], $[identifier]

---

## 📚 What You'll Learn

This guide covers **MongoDB array manipulation operators**:

✅ Array removal operators ($pop, $pull, $pullAll)  
✅ Positional operators ($, $[], $[identifier])  
✅ arrayFilters for conditional updates  
✅ Updating nested arrays  
✅ Real-world work history examples  
✅ Common patterns and use cases

**Best for:** Array manipulation, nested updates, complex queries, interview preparation

---

## Table of Contents

1. [Array Removal Operators Overview](#1-array-removal-operators-overview)
2. [$pop — Remove First or Last](#2-pop--remove-first-or-last)
3. [$pull — Remove by Condition](#3-pull--remove-by-condition)
4. [$pullAll — Remove Multiple Values](#4-pullall--remove-multiple-values)
5. [Positional Operators Overview](#5-positional-operators-overview)
6. [$ — Update First Match](#6---update-first-match)
7. [$[] — Update All Elements](#7---update-all-elements)
8. [$[identifier] — Update Filtered Elements](#8-identifier--update-filtered-elements)
9. [arrayFilters — Conditional Array Updates](#9-arrayfilters--conditional-array-updates)
10. [Complete Work History Examples](#10-complete-work-history-examples)
11. [Real-World Project Examples](#11-real-world-project-examples)
12. [Summary](#12-summary)
13. [Revision Checklist](#13-revision-checklist)

---

## 1. Array Removal Operators Overview

### The Three Removal Operators

```
$pop      → Remove first (-1) or last (1) element
$pull     → Remove all elements matching condition
$pullAll  → Remove all elements matching values
```

---

### When to Use Each

| Use Case                   | Operator | Example                      |
| -------------------------- | -------- | ---------------------------- |
| **Remove last item**       | $pop: 1  | Remove newest activity       |
| **Remove first item**      | $pop: -1 | Remove oldest activity       |
| **Remove by value**        | $pull    | Remove specific skill        |
| **Remove by condition**    | $pull    | Remove skills containing "a" |
| **Remove multiple values** | $pullAll | Remove ["css", "js"]         |

---

## 2. $pop — Remove First or Last

### What Does $pop Do?

**Removes ONE element from either the BEGINNING or END of an array.**

Think of it like:

- **Stack operations** — pop from top or bottom
- **Queue** — dequeue from front or back
- **Recent history** — remove oldest or newest

---

### Syntax

```javascript
// Remove LAST element (1)
{
  $pop: {
    arrayField: 1;
  }
}

// Remove FIRST element (-1)
{
  $pop: {
    arrayField: -1;
  }
}
```

**⚠️ Only accepts 1 or -1 — no other values!**

---

### Remove Last Element (1)

```javascript
// Before:
{
    _id: 1,
    skills: ["mongodb", "sql", "nodejs", "express"]
}

// Update:
db.arrayUpdate.updateOne(
    { _id: 1 },
    { $pop: { skills: 1 } }
);

// After:
{
    _id: 1,
    skills: ["mongodb", "sql", "nodejs"]
    //                         ↑ "express" removed
}
```

---

### Remove First Element (-1)

```javascript
// Before:
{
    _id: 1,
    skills: ["mongodb", "sql", "nodejs", "express"]
}

// Update:
db.arrayUpdate.updateOne(
    { _id: 1 },
    { $pop: { skills: -1 } }
);

// After:
{
    _id: 1,
    skills: ["sql", "nodejs", "express"]
    //      ↑ "mongodb" removed from beginning
}
```

---

### Visual: $pop Behavior

```
Original Array:
["mongodb", "sql", "nodejs", "express"]
 ↑ First                         ↑ Last
 (index 0)                       (index 3)

$pop: 1  (Remove LAST)
Result: ["mongodb", "sql", "nodejs"]
                             ↑ "express" removed

$pop: -1 (Remove FIRST)
Result: ["sql", "nodejs", "express"]
        ↑ "mongodb" removed
```

---

### Real-World Examples

#### Example 1: Recent Activity Log

```javascript
// Keep only last 5 activities, remove oldest when adding new
db.users.updateOne(
  { userId: 123 },
  {
    $push: {
      recentActivity: {
        $each: [{ action: "login", time: new Date() }],
        $slice: -5, // Keep last 5
      },
    },
  },
);

// Or manually remove oldest:
db.users.updateOne(
  { userId: 123 },
  { $pop: { recentActivity: -1 } }, // Remove oldest
);
```

---

#### Example 2: Undo/Redo Stack

```javascript
// Undo last action
db.editor.updateOne(
  { documentId: 456 },
  { $pop: { undoStack: 1 } }, // Remove last action
);
```

---

## 3. $pull — Remove by Condition

### What Does $pull Do?

**Removes ALL array elements that match a specified value or condition.**

Think of it like:

- **Filter opposite** — remove instead of keep
- **Array.filter()** but removes matches
- **Bulk delete** from array

---

### Syntax

```javascript
// Remove by exact value
{
  $pull: {
    arrayField: "value";
  }
}

// Remove by condition
{
  $pull: {
    arrayField: {
      condition;
    }
  }
}
```

---

### Remove by Exact Value

```javascript
// Before:
{
    _id: 4,
    skills: ["html", "css", "js", "react"]
}

// Remove "html":
db.arrayUpdate.updateOne(
    { _id: 4 },
    { $pull: { skills: "html" } }
);

// After:
{
    _id: 4,
    skills: ["css", "js", "react"]
    //      ↑ "html" removed
}
```

---

### Remove by Condition

```javascript
// Before:
{
    _id: 2,
    skills: ["react", "angular", "java", "laravel"]
}

// Remove all skills containing "a" (case-insensitive):
db.arrayUpdate.updateOne(
    { _id: 2 },
    { $pull: { skills: { $regex: /a/i } } }
);

// After:
{
    _id: 2,
    skills: []  // All removed! (react, angular, java, laravel all have "a")
}
```

---

### Remove Multiple Occurrences

```javascript
// Before:
{
    _id: 5,
    tags: ["javascript", "python", "javascript", "ruby", "javascript"]
}

// Remove ALL occurrences of "javascript":
db.arrayUpdate.updateOne(
    { _id: 5 },
    { $pull: { tags: "javascript" } }
);

// After:
{
    _id: 5,
    tags: ["python", "ruby"]
    //    ↑ All "javascript" removed
}
```

---

### Visual: $pull Behavior

```
Original Array:
["react", "angular", "java", "laravel"]

$pull with /a/i regex:
Check each element:
  "react"   → has "a" ✅ → REMOVE
  "angular" → has "a" ✅ → REMOVE
  "java"    → has "a" ✅ → REMOVE
  "laravel" → has "a" ✅ → REMOVE

Result: []  (All removed!)
```

---

### $pull with Conditions

```javascript
// Remove numbers greater than 50
db.collection.updateOne({ _id: 1 }, { $pull: { scores: { $gt: 50 } } });

// Before: { scores: [30, 60, 45, 70, 20] }
// After:  { scores: [30, 45, 20] }  (60 and 70 removed)
```

---

### Real-World Examples

#### Example 1: Remove Tag from Article

```javascript
// User removes tag "javascript" from article
db.articles.updateOne({ articleId: 123 }, { $pull: { tags: "javascript" } });
```

---

#### Example 2: Remove Expired Notifications

```javascript
// Remove all notifications older than 30 days
const thirtyDaysAgo = new Date();
thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

db.users.updateMany(
  {},
  {
    $pull: {
      notifications: {
        createdAt: { $lt: thirtyDaysAgo },
      },
    },
  },
);
```

---

#### Example 3: Unfollow User

```javascript
// User unfollows another user
db.users.updateOne(
  { userId: 123 },
  { $pull: { following: 456 } }, // Remove user 456 from following list
);
```

---

## 4. $pullAll — Remove Multiple Values

### What Does $pullAll Do?

**Removes ALL elements that match ANY of the specified values** (exact matches only).

Think of it like:

- **Bulk remove** specific values
- **$pull multiple times** in one operation
- **Blacklist** — remove all blacklisted items

---

### Syntax

```javascript
{
  $pullAll: {
    arrayField: [value1, value2, value3];
  }
}
```

---

### Basic Example

```javascript
// Before:
{
    _id: 4,
    skills: ["html", "css", "js", "react", "python"]
}

// Remove "css" AND "js":
db.arrayUpdate.updateOne(
    { _id: 4 },
    { $pullAll: { skills: ["css", "js"] } }
);

// After:
{
    _id: 4,
    skills: ["html", "react", "python"]
    //      ↑ "css" and "js" removed
}
```

---

### Multiple Values

```javascript
// Before:
{
    _id: 1,
    tags: ["javascript", "python", "java", "ruby", "go"]
}

// Remove multiple tags:
db.collection.updateOne(
    { _id: 1 },
    { $pullAll: { tags: ["python", "ruby", "go"] } }
);

// After:
{
    _id: 1,
    tags: ["javascript", "java"]
}
```

---

### Visual: $pullAll Behavior

```
Original Array:
["html", "css", "js", "react", "python"]

$pullAll: ["css", "js"]

Check each element:
  "html"   → Not in ["css", "js"] → KEEP
  "css"    → In ["css", "js"] ✅ → REMOVE
  "js"     → In ["css", "js"] ✅ → REMOVE
  "react"  → Not in ["css", "js"] → KEEP
  "python" → Not in ["css", "js"] → KEEP

Result: ["html", "react", "python"]
```

---

### $pull vs $pullAll

```javascript
// Same document:
{ _id: 1, tags: ["a", "b", "c", "d", "e"] }

// $pull (one value):
{ $pull: { tags: "b" } }
// Result: ["a", "c", "d", "e"]

// $pullAll (multiple values):
{ $pullAll: { tags: ["b", "d"] } }
// Result: ["a", "c", "e"]

// $pull with condition (flexible):
{ $pull: { tags: { $in: ["b", "d"] } } }
// Result: ["a", "c", "e"]  (same as $pullAll)
```

---

### Real-World Example: Remove Multiple Permissions

```javascript
// Remove multiple permissions from user role
db.users.updateOne(
  { userId: 123 },
  {
    $pullAll: {
      permissions: ["delete_user", "delete_post", "ban_user"],
    },
  },
);
```

---

## 5. Positional Operators Overview

### The Three Positional Operators

```
$              → Update FIRST matched element
$[]            → Update ALL elements
$[identifier]  → Update FILTERED elements (with arrayFilters)
```

---

### When to Use Each

| Use Case                | Operator      | Example                                 |
| ----------------------- | ------------- | --------------------------------------- |
| **Update first match**  | $             | Update first "html" to "xml"            |
| **Update all elements** | $[]           | Add bonus to all experiences            |
| **Update filtered**     | $[identifier] | Update only Google experiences < 1 year |

---

### Visual: Positional Operators

```
Array: [
    { company: "google", years: 2 },     ← Element 0
    { company: "yahoo", years: 1 },      ← Element 1
    { company: "google", years: 0.6 }    ← Element 2
]

$              → Updates element 0 (first match)
$[]            → Updates all 3 elements
$[identifier]  → Updates only elements matching filter
               (e.g., only google with years < 1)
```

---

## 6. $ — Update First Match

### What Does $ Do?

**Updates the FIRST array element that matches the query filter.**

Think of it like:

- **Find first, update that one**
- **Single target** update
- **Positional placeholder**

---

### Syntax

```javascript
// The $ represents the first matched element
{ $set: { "arrayField.$": newValue } }
```

**⚠️ Critical Rule:** The array element MUST be in the filter query!

---

### Basic Example

```javascript
// Before:
{
    _id: 3,
    skills: ["python", "sql", "django"]
}

// Update first occurrence of "sql" to "mongodb":
db.arrayUpdate.updateOne(
    { _id: 3, skills: "sql" },  // ← MUST include array element in filter
    { $set: { "skills.$": "mongodb" } }
);

// After:
{
    _id: 3,
    skills: ["python", "mongodb", "django"]
    //                 ↑ "sql" changed to "mongodb"
}
```

---

### How $ Works — Step by Step

```
Query: { _id: 3, skills: "sql" }

Step 1: Find document with _id: 3 ✅
Step 2: Find "sql" in skills array
        skills: ["python", "sql", "django"]
                          ↑ Found at index 1

Step 3: $ represents index 1
        "skills.$" = "skills.1"

Step 4: Update skills[1]
        skills[1] = "mongodb"

Result: ["python", "mongodb", "django"]
```

---

### Multiple Matches — Only First Updated

```javascript
// Before:
{
    _id: 1,
    skills: ["python", "java", "python", "ruby"]
}

// Update "python" to "javascript":
db.arrayUpdate.updateOne(
    { _id: 1, skills: "python" },
    { $set: { "skills.$": "javascript" } }
);

// After:
{
    _id: 1,
    skills: ["javascript", "java", "python", "ruby"]
    //      ↑ Only FIRST "python" updated
    //                            ↑ This "python" unchanged
}
```

---

### Nested Arrays

```javascript
// Before:
{
    _id: 1,
    exp: [
        { company: "google", years: 2 },
        { company: "yahoo", years: 1 },
        { company: "google", years: 0.6 }
    ]
}

// Update first Google experience:
db.workHistory.updateOne(
    { _id: 1, exp: { $elemMatch: { company: "google" } } },
    { $set: { "exp.$.isActive": true } }
);

// After:
{
    _id: 1,
    exp: [
        { company: "google", years: 2, isActive: true },  ← Updated!
        { company: "yahoo", years: 1 },
        { company: "google", years: 0.6 }                ← Not updated
    ]
}
```

---

### Real-World Example: Mark First Notification Read

```javascript
// User has unread notifications
db.users.updateOne(
  { userId: 123, notifications: { $elemMatch: { read: false } } },
  { $set: { "notifications.$.read": true } },
);

// Only marks FIRST unread notification as read
```

---

## 7. $[] — Update All Elements

### What Does $[] Do?

**Updates ALL elements in an array** (no filtering).

Think of it like:

- **Bulk update** entire array
- **Apply to everyone**
- **Broadcast update**

---

### Syntax

```javascript
// $[] with empty brackets = all elements
{ $set: { "arrayField.$[]": newValue } }

// Or update nested field in all elements:
{ $set: { "arrayField.$[].fieldName": value } }
```

---

### Basic Example

```javascript
// Before:
{
    _id: 2,
    exp: [
        { company: "wipro", years: 5 },
        { company: "hcl", years: 3 },
        { company: "google", years: 1 }
    ]
}

// Add incentive to ALL experiences:
db.workHistory.updateOne(
    { _id: 2 },
    { $set: { "exp.$[].incentive": "2000 given by wipro" } }
);

// After:
{
    _id: 2,
    exp: [
        { company: "wipro", years: 5, incentive: "2000 given by wipro" },
        { company: "hcl", years: 3, incentive: "2000 given by wipro" },
        { company: "google", years: 1, incentive: "2000 given by wipro" }
    ]
}
```

**All three elements updated!**

---

### Visual: $[] Updates All

```
Before:
exp: [
    { company: "wipro", years: 5 },   ← Element 0
    { company: "hcl", years: 3 },     ← Element 1
    { company: "google", years: 1 }   ← Element 2
]

$set: { "exp.$[].incentive": "2000" }

After:
exp: [
    { company: "wipro", years: 5, incentive: "2000" },   ← Updated
    { company: "hcl", years: 3, incentive: "2000" },     ← Updated
    { company: "google", years: 1, incentive: "2000" }   ← Updated
]

All elements get the same update!
```

---

### updateMany with $[]

```javascript
// Update ALL documents, ALL array elements
db.workHistory.updateMany(
  { exp: { $elemMatch: { company: "wipro" } } }, // Find docs with wipro
  { $set: { "exp.$[].incentive": "2000" } }, // Update all exp in those docs
);
```

---

### Real-World Example: Mark All Notifications Read

```javascript
// Mark ALL notifications as read for user
db.users.updateOne(
  { userId: 123 },
  { $set: { "notifications.$[].read": true } },
);
```

---

## 8. $[identifier] — Update Filtered Elements

### What Does $[identifier] Do?

**Updates ONLY array elements that match a filter condition** (specified in arrayFilters).

Think of it like:

- **Conditional bulk update**
- **Filter then update**
- **Selective update**

---

### Syntax

```javascript
db.collection.updateOne(
  { query },
  { $set: { "arrayField.$[identifier].field": value } },
  { arrayFilters: [{ "identifier.condition": value }] },
);
```

**Components:**

- `$[identifier]` — Custom variable name (e.g., `$[e]`, `$[item]`)
- `arrayFilters` — Condition for which elements to update

---

### Basic Example

```javascript
// Before:
{
    _id: 3,
    skills: ["python", "javascript", "java", "ruby"]
}

// Update only "python" to "ruby":
db.arrayUpdate.updateOne(
    { _id: 3 },
    { $set: { "skills.$[s]": "ruby" } },
    { arrayFilters: [{ s: "python" }] }
    //               ↑ "s" matches "python"
);

// After:
{
    _id: 3,
    skills: ["ruby", "javascript", "java", "ruby"]
    //      ↑ Only "python" changed
}
```

---

### Complex Example — Work History

```javascript
// Before:
{
    name: "varun",
    exp: [
        { company: "google", years: 2 },
        { company: "yahoo", years: 1 },
        { company: "google", years: 0.6 }
    ]
}

// Add wfh: true to Google experiences with < 1 year:
db.workHistory.updateOne(
    { name: "varun" },
    { $set: { "exp.$[e].wfh": true } },
    {
        arrayFilters: [
            { "e.company": "google" },
            { "e.years": { $lt: 1 } }
        ]
    }
);

// After:
{
    name: "varun",
    exp: [
        { company: "google", years: 2 },           ← Not updated (years >= 1)
        { company: "yahoo", years: 1 },            ← Not updated (not google)
        { company: "google", years: 0.6, wfh: true } ← Updated! (google AND < 1)
    ]
}
```

---

### Visual: $[identifier] Filtering

```
exp: [
    { company: "google", years: 2 },      ← Element 0
    { company: "yahoo", years: 1 },       ← Element 1
    { company: "google", years: 0.6 }     ← Element 2
]

arrayFilters: [
    { "e.company": "google" },
    { "e.years": { $lt: 1 } }
]

Check each element:
  Element 0: company="google" ✅, years=2 ❌ (not < 1) → SKIP
  Element 1: company="yahoo" ❌                        → SKIP
  Element 2: company="google" ✅, years=0.6 ✅        → UPDATE ✅

Result: Only element 2 updated
```

---

### Multiple Identifiers

```javascript
// Update nested arrays with multiple conditions
db.collection.updateMany(
  {},
  {
    $set: {
      "students.$[student].grades.$[grade].curve": 5,
    },
  },
  {
    arrayFilters: [
      { "student.year": 2024 }, // Filter students
      { "grade.score": { $lt: 70 } }, // Filter grades
    ],
  },
);
```

---

## 9. arrayFilters — Conditional Array Updates

### What are arrayFilters?

**Specify conditions for which array elements to update** when using `$[identifier]`.

Think of it like:

- **WHERE clause** for arrays
- **Filter before update**
- **Conditional targeting**

---

### Syntax

```javascript
{
  arrayFilters: [
    { "identifier.field": condition },
    { "identifier.field2": condition2 },
  ];
}
```

---

### Basic arrayFilter

```javascript
// Update only experiences where company = "TCS"
db.users.updateOne(
  { userId: 123 },
  { $set: { "experience.$[exp].isActive": true } },
  { arrayFilters: [{ "exp.company": "TCS" }] },
);
```

---

### Multiple Conditions in arrayFilters

```javascript
// Update Google experiences with less than 1 year AND mark as intern
db.workHistory.updateMany(
  {},
  { $set: { "exp.$[e].intern": true } },
  {
    arrayFilters: [
      {
        "e.company": "google",
        "e.years": { $lt: 1 },
      },
    ],
  },
);
```

---

### Complex arrayFilters

```javascript
// Update products in cart with price > 100 and quantity > 2
db.orders.updateOne(
  { orderId: 123 },
  { $set: { "items.$[item].discount": 10 } },
  {
    arrayFilters: [
      {
        "item.price": { $gt: 100 },
        "item.quantity": { $gt: 2 },
      },
    ],
  },
);
```

---

## 10. Complete Work History Examples

### Example 1: Add Intern Flag

**Problem:** Add `intern: true` to Google experiences < 1 year

```javascript
// Data:
{
    name: "varun",
    exp: [
        { company: "google", years: 2 },
        { company: "yahoo", years: 1 },
        { company: "google", years: 0.6 }  // ← Should be marked
    ]
}

// Solution:
db.workHistory.updateMany(
    { exp: { $elemMatch: { company: "google" } } },
    { $set: { "exp.$[e].intern": true } },
    {
        arrayFilters: [
            { "e.company": "google", "e.years": { $lt: 1 } }
        ]
    }
);

// Result:
{
    name: "varun",
    exp: [
        { company: "google", years: 2 },
        { company: "yahoo", years: 1 },
        { company: "google", years: 0.6, intern: true }  ← Marked!
    ]
}
```

---

### Example 2: Add WFH Flag

**Problem:** Add `wfh: true` to Google experiences < 1 year

```javascript
// Data (3 documents):
[
  {
    name: "varun",
    exp: [
      { company: "google", years: 2 },
      { company: "yahoo", years: 1 },
      { company: "google", years: 0.6 }, // ← Mark this
    ],
  },
  {
    name: "sri",
    exp: [
      { company: "wipro", years: 5 },
      { company: "hcl", years: 3 },
      { company: "google", years: 1 }, // ← NOT < 1
    ],
  },
  {
    name: "sirisha",
    exp: [
      { company: "oracle", years: 1 },
      { company: "google", years: 2 },
      { company: "wipro", years: 2 },
      { company: "google", years: 0.1 }, // ← Mark this
    ],
  },
];

// Solution:
db.workHistory.updateMany(
  { exp: { $elemMatch: { company: "google" } } },
  { $set: { "exp.$[e].wfh": true } },
  {
    arrayFilters: [{ "e.company": "google", "e.years": { $lt: 1 } }],
  },
);

// Result: Only varun's 0.6 years and sirisha's 0.1 years marked
```

---

### Example 3: Add Incentive to All

**Problem:** Add incentive to ALL experiences for employees who worked at Wipro

```javascript
// Data:
[
    {
        name: "sri",
        exp: [
            { company: "wipro", years: 5 },     // ← Has wipro
            { company: "hcl", years: 3 },
            { company: "google", years: 1 }
        ]
    },
    {
        name: "sirisha",
        exp: [
            { company: "oracle", years: 1 },
            { company: "google", years: 2 },
            { company: "wipro", years: 2 },     // ← Has wipro
            { company: "google", years: 0.1 }
        ]
    }
]

// Solution:
db.workHistory.updateMany(
    { exp: { $elemMatch: { company: "wipro" } } },  // Find docs with wipro
    { $set: { "exp.$[].incentive": "2000 given by wipro" } }  // Update ALL exp
);

// Result: ALL experiences in sri and sirisha get incentive
{
    name: "sri",
    exp: [
        { company: "wipro", years: 5, incentive: "2000 given by wipro" },
        { company: "hcl", years: 3, incentive: "2000 given by wipro" },
        { company: "google", years: 1, incentive: "2000 given by wipro" }
    ]
}
```

---

### Comparison: $, $[], $[identifier]

```javascript
// Same data for all examples:
{
    name: "varun",
    exp: [
        { company: "google", years: 2 },
        { company: "yahoo", years: 1 },
        { company: "google", years: 0.6 }
    ]
}

// Using $ (first match only):
db.workHistory.updateOne(
    { name: "varun", exp: { $elemMatch: { company: "google" } } },
    { $set: { "exp.$.updated": true } }
);
// Result: Only first google (2 years) updated

// Using $[] (all elements):
db.workHistory.updateOne(
    { name: "varun" },
    { $set: { "exp.$[].updated": true } }
);
// Result: All 3 elements updated

// Using $[identifier] (filtered):
db.workHistory.updateOne(
    { name: "varun" },
    { $set: { "exp.$[e].updated": true } },
    { arrayFilters: [{ "e.company": "google" }] }
);
// Result: Both google elements (2 years and 0.6 years) updated
```

---

## 11. Real-World Project Examples

### Project 1: Social Media App

```javascript
// Remove old notifications
db.users.updateMany(
  {},
  {
    $pull: {
      notifications: {
        createdAt: { $lt: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) },
      },
    },
  },
);

// Remove last activity (privacy)
db.users.updateOne({ userId: 123 }, { $pop: { recentActivity: 1 } });

// Remove multiple blocked users from followers
db.users.updateOne(
  { userId: 123 },
  { $pullAll: { followers: [456, 789, 101] } },
);

// Update all posts to set archived: true
db.users.updateOne({ userId: 123 }, { $set: { "posts.$[].archived": true } });

// Update only viral posts (likes > 1000) to featured: true
db.users.updateOne(
  { userId: 123 },
  { $set: { "posts.$[post].featured": true } },
  { arrayFilters: [{ "post.likes": { $gt: 1000 } }] },
);
```

---

### Project 2: E-Commerce Platform

```javascript
// Remove item from cart
db.carts.updateOne({ userId: 123 }, { $pull: { items: { productId: 456 } } });

// Remove multiple items
db.carts.updateOne(
  { userId: 123 },
  { $pullAll: { items: [{ productId: 456 }, { productId: 789 }] } },
);

// Remove oldest item from recent views
db.users.updateOne({ userId: 123 }, { $pop: { recentlyViewed: -1 } });

// Apply discount to all cart items
db.carts.updateOne({ userId: 123 }, { $set: { "items.$[].discount": 10 } });

// Apply 20% discount to expensive items only
db.carts.updateOne(
  { userId: 123 },
  { $set: { "items.$[item].discount": 20 } },
  { arrayFilters: [{ "item.price": { $gt: 500 } }] },
);
```

---

### Project 3: Task Management

```javascript
// Remove completed task
db.projects.updateOne(
  { projectId: 123 },
  { $pull: { tasks: { status: "completed" } } },
);

// Remove multiple deleted tasks
db.projects.updateOne(
  { projectId: 123 },
  { $pullAll: { taskIds: [1, 5, 8, 12] } },
);

// Update all tasks to set reviewed: false
db.projects.updateOne(
  { projectId: 123 },
  { $set: { "tasks.$[].reviewed": false } },
);

// Mark only high-priority tasks as urgent
db.projects.updateOne(
  { projectId: 123 },
  { $set: { "tasks.$[task].urgent": true } },
  { arrayFilters: [{ "task.priority": "high" }] },
);

// Update first incomplete task
db.projects.updateOne(
  { projectId: 123, tasks: { $elemMatch: { status: "pending" } } },
  { $set: { "tasks.$.status": "in_progress" } },
);
```

---

## 12. Summary — Key Takeaways

### 🗑️ Removal Operators

```
$pop      → Remove first (-1) or last (1)
$pull     → Remove all matching condition
$pullAll  → Remove all matching values
```

---

### 🎯 Positional Operators

```
$              → Update FIRST match (requires filter)
$[]            → Update ALL elements
$[identifier]  → Update FILTERED elements (with arrayFilters)
```

---

### 📋 When to Use Which

| Goal                | Operator      | Example             |
| ------------------- | ------------- | ------------------- |
| **Remove last**     | $pop: 1       | Remove newest       |
| **Remove first**    | $pop: -1      | Remove oldest       |
| **Remove matching** | $pull         | Remove "html"       |
| **Remove multiple** | $pullAll      | Remove ["a", "b"]   |
| **Update first**    | $             | Update first match  |
| **Update all**      | $[]           | Update everyone     |
| **Update filtered** | $[identifier] | Update if condition |

---

### ⚠️ Critical Rules

```
✅ $ requires array element in filter query
✅ $[] updates ALL elements
✅ $[identifier] requires arrayFilters
✅ $pop only accepts 1 or -1
✅ $pull can use conditions
✅ $pullAll uses exact values only
```

---

## 13. Revision Checklist

### Removal Operators

- [ ] Can you use $pop to remove first element?
- [ ] Can you use $pop to remove last element?
- [ ] Do you know $pop only accepts 1 or -1?
- [ ] Can you use $pull with exact value?
- [ ] Can you use $pull with condition?
- [ ] Can you use $pull with regex?
- [ ] Can you use $pullAll?
- [ ] Do you know difference between $pull and $pullAll?

### Positional Operators

- [ ] Can you use $ to update first match?
- [ ] Do you know $ requires filter?
- [ ] Can you use $[] to update all elements?
- [ ] Can you use $[identifier]?
- [ ] Do you know what arrayFilters does?
- [ ] Can you write arrayFilters conditions?

### Complex Updates

- [ ] Can you update nested array elements?
- [ ] Can you combine conditions in arrayFilters?
- [ ] Can you update first vs all vs filtered?
- [ ] Can you handle multiple array filters?

### Real-World

- [ ] Can you remove old notifications?
- [ ] Can you update cart items conditionally?
- [ ] Can you mark filtered tasks?
- [ ] Can you update work history?

---

> **🎤 Interview Tip — "Explain the difference between $, $[], and $[identifier] with an example"**
>
> **Answer like this:**
>
> _"These are MongoDB's positional operators for updating arrays, each serving different purposes:_
>
> _The $ operator updates only the FIRST element that matches the query filter. For example, if you have multiple 'google' experiences in an array, $ will only update the first one. Critically, the array element must be part of the query filter._
>
> _The $[] operator updates ALL elements in the array without any filtering. This is useful when you want to apply the same change to every element, like marking all experiences as reviewed._
>
> _The $[identifier] operator is the most flexible — it updates only elements that match specific conditions defined in arrayFilters. For example, you could update only Google experiences where years < 1._
>
> _Here's a concrete example:_
>
> ```javascript
> // Array: [
> //   { company: "google", years: 2 },
> //   { company: "yahoo", years: 1 },
> //   { company: "google", years: 0.6 }
> // ]
>
> // $ — Updates first google only
> { exp: { $elemMatch: { company: "google" } } },
> { $set: { "exp.$.updated": true } }
> // Result: Only first google (2 years) updated
>
> // $[] — Updates all three elements
> { $set: { "exp.$[].updated": true } }
> // Result: All elements updated
>
> // $[e] — Updates only google experiences < 1 year
> { $set: { "exp.$[e].intern": true } },
> { arrayFilters: [{ "e.company": "google", "e.years": { $lt: 1 } }] }
> // Result: Only google with 0.6 years updated
> ```
>
> _The choice depends on whether you need to target one element, all elements, or a filtered subset."_
>
> **Why this works:** Explains all three clearly, shows concrete differences with same dataset, provides working code, mentions critical requirements like filter query and arrayFilters, demonstrates understanding of use cases.
