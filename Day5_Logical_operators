# MongoDB Query Operators — Complete Guide

### Comparison, Logical, and Special Cases

---

## 📚 What You'll Learn

This guide covers **all MongoDB query operators** for filtering documents:

✅ All comparison operators ($eq, $ne, $gt, $gte, $lt, $lte, $in, $nin)  
✅ All logical operators ($and, $or, $nor, $not)  
✅ Syntax patterns for each operator  
✅ Multiple conditions — same field vs different fields  
✅ Common mistakes and how to avoid them  
✅ Real-world query examples

**Best for:** Writing MongoDB queries, filtering data, interview preparation

---

## Table of Contents

1. [What Are Operators?](#1-what-are-operators)
2. [MongoDB Operator Rule](#2-mongodb-operator-rule)
3. [Comparison Operators — Overview](#3-comparison-operators--overview)
4. [The Big 6 — Basic Comparison](#4-the-big-6--basic-comparison)
5. [$in and $nin — Multiple Values](#5-in-and-nin--multiple-values)
6. [Logical Operators — Overview](#6-logical-operators--overview)
7. [$and — All Conditions Must Match](#7-and--all-conditions-must-match)
8. [$or — Any Condition Can Match](#8-or--any-condition-can-match)
9. [$nor — None of the Conditions Match](#9-nor--none-of-the-conditions-match)
10. [$not — Invert Expression](#10-not--invert-expression)
11. [Multiple Conditions — Critical Rules](#11-multiple-conditions--critical-rules)
12. [Case 1: Same Field — Last One Wins](#12-case-1-same-field--last-one-wins)
13. [Case 2: Different Fields — Implicit AND](#13-case-2-different-fields--implicit-and)
14. [Complete Operator Reference](#14-complete-operator-reference)
15. [Summary](#15-summary)
16. [Revision Checklist](#16-revision-checklist)

---

## 1. What Are Operators?

### Simple Definition

**An operator is a reserved symbol that performs a specific task on operands.**

**In regular JavaScript:**

```javascript
let sum = 2 + 3;
//        ↑   ↑
//    operands (values)
//      ↓
//      + operator (performs addition)
```

**In MongoDB:**

```javascript
db.emp.find({ age: { $gt: 27 } });
//                   ↑    ↑
//               operator  operand
//        $gt means "greater than"
```

---

### Operands and Operators

| Term         | Definition                             | Example                   |
| ------------ | -------------------------------------- | ------------------------- |
| **Operand**  | The values being operated on           | `2`, `3`, `27`, `"smith"` |
| **Operator** | The symbol that performs the operation | `+`, `/`, `$gt`, `$eq`    |

```
Expression:  2 + 3
             ↓   ↓
         operand  operand
               ↓
           operator

MongoDB:  { age: { $gt: 27 } }
                   ↓    ↓
               operator  operand
```

---

## 2. MongoDB Operator Rule

### The $ Rule

**CRITICAL:** In MongoDB, **all operators start with the dollar sign `$`**

```javascript
// ✅ CORRECT — MongoDB operators
$eq   $ne   $gt   $gte   $lt   $lte
$in   $nin  $and  $or    $nor  $not

// ❌ WRONG — Missing the $
eq    ne    gt    gte    lt    lte
```

**Why the $ sign?**

- Distinguishes operators from field names
- Prevents conflicts (e.g., a field named "gt" vs the operator "$gt")

---

## 3. Comparison Operators — Overview

### What Are Comparison Operators?

**Comparison operators compare a field's value against a specified value.**

Think of them like:

- Is this person **older** than 18?
- Is this salary **less than** $50,000?
- Is this name **equal to** "Smith"?

---

### The 8 Comparison Operators

```
1. $eq   — equals to
2. $ne   — not equals to
3. $gt   — greater than
4. $gte  — greater than or equals to
5. $lt   — less than
6. $lte  — less than or equals to
7. $in   — matches any value in array
8. $nin  — matches none of the values in array
```

---

### Visual: Number Line Operators

```
      ┌───────────────────────────────────────┐
      │                                       │
  ←───┴───────────────────────────────────────┴───→
      0        10        20        30        40

$lt: 20   →  Values < 20 (less than)
             ◄──────────●

$lte: 20  →  Values ≤ 20 (less than or equal)
             ◄──────────●═

$gt: 20   →  Values > 20 (greater than)
                        ●──────────►

$gte: 20  →  Values ≥ 20 (greater than or equal)
                        ═●──────────►

$eq: 20   →  Values = 20 (exact match)
                        ●

$ne: 20   →  Values ≠ 20 (everything except 20)
             ◄──────────○──────────►
```

---

## 4. The Big 6 — Basic Comparison

### Syntax Pattern

```javascript
// General syntax for $eq, $ne, $gt, $gte, $lt, $lte:
{
  fieldName: {
    $operator: value;
  }
}
```

---

### 4.1 $eq — Equals To

**What it does:** Finds documents where field **exactly matches** the value.

```javascript
// Explicit use of $eq
db.emp.findOne({ empName: { $eq: "smith" } });

// Implicit use of $eq (same thing)
db.emp.findOne({ empName: "smith" });

// Both are equivalent!
```

**When implicit $eq is used:**

```javascript
{ name: "John" }        →  Implicit: { name: { $eq: "John" } }
{ age: 30 }             →  Implicit: { age: { $eq: 30 } }
{ status: "active" }    →  Implicit: { status: { $eq: "active" } }
```

> 💡 **Best practice:** Use implicit form (simpler and cleaner) unless you need to be explicit.

---

### 4.2 $ne — Not Equals To

**What it does:** Finds documents where field **does NOT match** the value.

```javascript
// Find all employees who are NOT clerks
db.emp.find({ job: { $ne: "clerk" } }, { job: 1 });

// Returns:
// { job: "manager" }
// { job: "analyst" }
// { job: "salesman" }
// (everything except "clerk")
```

---

### 4.3 $gt — Greater Than

**What it does:** Finds documents where field is **greater than** the value.

```javascript
// Find all employees with age > 27
db.emp.find({ age: { $gt: 27 } }, { empName: 1, age: 1, _id: 0 });

// Returns documents where age is 28, 29, 30, 31...
// Does NOT include age = 27 (strict greater than)
```

---

### 4.4 $gte — Greater Than or Equals To

**What it does:** Finds documents where field is **greater than or equal to** the value.

```javascript
// Find employees with bonus >= 1000
db.emp.find({ bonus: { $gte: 1000 } }, { bonus: 1 });

// Returns documents where bonus is 1000, 1001, 1002...
// INCLUDES bonus = 1000 (the equals part)
```

---

### 4.5 $lt — Less Than

**What it does:** Finds documents where field is **less than** the value.

```javascript
// Find employees with salary < 3000
db.emp.find({ sal: { $lt: 3000 } }, { sal: 1 });

// Returns documents where sal is 2999, 2500, 1000...
// Does NOT include sal = 3000
```

---

### 4.6 $lte — Less Than or Equals To

**What it does:** Finds documents where field is **less than or equal to** the value.

```javascript
// Find departments with budget <= 150000
db.dept.find({ budget: { $lte: 150000 } }, { budget: 1 });

// Returns departments where budget is 150000, 100000, 50000...
// INCLUDES budget = 150000
```

---

### Comparison Summary Table

| Operator | Meaning          | Example                 | Matches                  |
| -------- | ---------------- | ----------------------- | ------------------------ |
| `$eq`    | Equal to         | `{ age: { $eq: 30 } }`  | age = 30                 |
| `$ne`    | Not equal to     | `{ age: { $ne: 30 } }`  | age ≠ 30                 |
| `$gt`    | Greater than     | `{ age: { $gt: 30 } }`  | age > 30 (31, 32...)     |
| `$gte`   | Greater or equal | `{ age: { $gte: 30 } }` | age ≥ 30 (30, 31, 32...) |
| `$lt`    | Less than        | `{ age: { $lt: 30 } }`  | age < 30 (29, 28...)     |
| `$lte`   | Less or equal    | `{ age: { $lte: 30 } }` | age ≤ 30 (30, 29, 28...) |

---

## 5. $in and $nin — Multiple Values

### What Makes Them Special?

Unlike the Big 6 (which compare against **one value**), `$in` and `$nin` compare against **multiple values**.

---

### Syntax Pattern

```javascript
// Syntax for $in and $nin:
{ fieldName: { $operator: [value1, value2, value3, ...] } }
//                          ↑
//                      ARRAY of values
```

---

### 5.1 $in — Matches Any Value

**What it does:** Finds documents where field matches **ANY one** of the values in the array.

Think of it like **OR logic**: "Give me documents where field = v1 OR field = v2 OR field = v3"

```javascript
// Find employees with age 42, 45, OR 30
db.emp.find({ age: { $in: [42, 45, 30] } }, { age: 1, _id: 0 });

// Returns:
// { age: 42 }
// { age: 45 }
// { age: 30 }

// Equivalent to:
// { age: 42 } OR { age: 45 } OR { age: 30 }
```

---

### Real-World Example: $in

```javascript
// Find employees in cities Chicago, New York, OR Boston
db.emp.find({ city: { $in: ["Chicago", "New York", "Boston"] } });

// Returns all employees from any of these 3 cities
```

---

### 5.2 $nin — Matches None of the Values

**What it does:** Finds documents where field **does NOT match any** of the values in the array.

Think of it like **NOT (OR) logic**: "Give me documents where field ≠ v1 AND field ≠ v2 AND field ≠ v3"

```javascript
// Find employees who are NOT clerks or managers
db.emp.find(
  { job: { $nin: ["clerk", "manager"] } },
  { job: 1, empName: 1, _id: 0 },
);

// Returns:
// { job: "analyst", empName: "scott" }
// { job: "salesman", empName: "allen" }
// { job: "president", empName: "king" }

// EXCLUDES anyone with job = "clerk" or job = "manager"
```

---

### Visual: $in vs $nin

```
Collection:
{ age: 20 }
{ age: 30 }
{ age: 40 }
{ age: 50 }
{ age: 60 }

Query: { age: { $in: [30, 50] } }
Returns: ✅ { age: 30 }
         ❌ { age: 20 }
         ❌ { age: 40 }
         ✅ { age: 50 }
         ❌ { age: 60 }

Query: { age: { $nin: [30, 50] } }
Returns: ✅ { age: 20 }
         ❌ { age: 30 }
         ✅ { age: 40 }
         ❌ { age: 50 }
         ✅ { age: 60 }
```

---

## 6. Logical Operators — Overview

### What Are Logical Operators?

**Logical operators combine multiple conditions** — like AND, OR, NOT in programming.

Use them when you need to filter based on **more than one condition**.

---

### The 4 Logical Operators

```
1. $and  — ALL conditions must be true
2. $or   — ANY ONE condition must be true
3. $nor  — NONE of the conditions must be true
4. $not  — INVERT the expression (opposite)
```

---

### Syntax Pattern (for $and, $or, $nor)

```javascript
// General syntax:
{ $operator: [ {condition1}, {condition2}, {condition3}, ... ] }
//              ↑
//          ARRAY of condition objects
```

---

### Visual: Logical Operators

```
Conditions:
  C1: age > 25
  C2: deptNo = 20

$and: [C1, C2]  →  C1 AND C2 (both must be true)
      ◉◉ Required

$or:  [C1, C2]  →  C1 OR C2 (at least one true)
      ◉○ Either works
      ○◉ Either works

$nor: [C1, C2]  →  NOT (C1 OR C2) (both must be false)
      ○○ Neither can be true
```

---

## 7. $and — All Conditions Must Match

### What It Does

**Finds documents where ALL conditions are true.**

Think of it like: "I want employees who are **young AND rich**" → both must be true.

---

### Syntax

```javascript
{
  $and: [{ condition1 }, { condition2 }, { condition3 }];
}
```

---

### Example 1: Two Conditions

```javascript
// Find employees with bonus >= 1000 AND salary < 3000
db.emp.find(
  {
    $and: [
      { sal: { $lt: 3000 } }, // Condition 1
      { bonus: { $gte: 1000 } }, // Condition 2
    ],
  },
  { bonus: 1, sal: 1, _id: 0 },
);

// Returns ONLY documents where BOTH conditions are true:
// { bonus: 1000, sal: 2500 } ✅
// { bonus: 1200, sal: 2000 } ✅
// { bonus: 800, sal: 2500 }  ❌ (bonus too low)
// { bonus: 1500, sal: 3500 } ❌ (salary too high)
```

---

### Example 2: Three Conditions

```javascript
// Find employees with:
// - Salary >= 1000
// - Salary <= 2000
// - Name = "adams"
db.emp.find(
  {
    $and: [
      { sal: { $gte: 1000 } }, // C1
      { sal: { $lte: 2000 } }, // C2
      { empName: "adams" }, // C3
    ],
  },
  { sal: 1, empName: 1, _id: 0 },
);

// Returns ONLY if ALL three are true
```

---

### Truth Table: $and

```
C1   C2   Result
──   ──   ──────
✅   ✅   ✅ Match!
✅   ❌   ❌ No match
❌   ✅   ❌ No match
❌   ❌   ❌ No match

→ BOTH must be true
```

---

## 8. $or — Any Condition Can Match

### What It Does

**Finds documents where AT LEAST ONE condition is true.**

Think of it like: "I want employees who are **young OR rich**" → either one works.

---

### Syntax

```javascript
{
  $or: [{ condition1 }, { condition2 }];
}
```

---

### Example

```javascript
// Find employees with age >= 25 OR deptNo = 20
db.emp.find(
  {
    $or: [{ age: { $gte: 25 } }, { deptNo: 20 }],
  },
  { age: 1, deptNo: 1, _id: 0 },
);

// Returns if ANY condition is true:
// { age: 30, deptNo: 10 } ✅ (age condition true)
// { age: 20, deptNo: 20 } ✅ (deptNo condition true)
// { age: 30, deptNo: 20 } ✅ (BOTH conditions true)
// { age: 20, deptNo: 10 } ❌ (neither condition true)
```

---

### Truth Table: $or

```
C1   C2   Result
──   ──   ──────
✅   ✅   ✅ Match!
✅   ❌   ✅ Match!
❌   ✅   ✅ Match!
❌   ❌   ❌ No match

→ At least ONE must be true
```

---

## 9. $nor — None of the Conditions Match

### What It Does

**Finds documents where NONE of the conditions are true.**

Think of it like: "I want employees who are **NOT young AND NOT rich**" → both must be false.

---

### Syntax

```javascript
{
  $nor: [{ condition1 }, { condition2 }];
}
```

---

### Example

```javascript
// Find employees with age < 25 AND deptNo ≠ 20
// (Opposite of: age >= 25 OR deptNo = 20)
db.emp.find(
  {
    $nor: [{ age: { $gte: 25 } }, { deptNo: 20 }],
  },
  { age: 1, deptNo: 1, _id: 0 },
);

// Returns ONLY if BOTH conditions are FALSE:
// { age: 30, deptNo: 10 } ❌ (age condition true)
// { age: 20, deptNo: 20 } ❌ (deptNo condition true)
// { age: 30, deptNo: 20 } ❌ (both conditions true)
// { age: 20, deptNo: 10 } ✅ (both conditions false!)
```

---

### Truth Table: $nor

```
C1   C2   Result
──   ──   ──────
✅   ✅   ❌ No match
✅   ❌   ❌ No match
❌   ✅   ❌ No match
❌   ❌   ✅ Match!

→ BOTH must be false
```

---

### $or vs $nor — The Opposite

```
$or:  Match if ANY condition is TRUE
$nor: Match if ALL conditions are FALSE

They are OPPOSITES of each other.
```

---

## 10. $not — Invert Expression

### What It Does

**Inverts (reverses) the expression** — turns true into false and false into true.

---

### Syntax (Different from Others!)

```javascript
// Notice: $not wraps the ENTIRE expression
{
  fieldName: {
    $not: {
      expression;
    }
  }
}
```

---

### Example 1: $not with $eq

```javascript
// Find employees who are NOT clerks

// Method 1: Using $ne
db.emp.find({ job: { $ne: "clerk" } }, { job: 1 });

// Method 2: Using $not + $eq (same result)
db.emp.find({ job: { $not: { $eq: "clerk" } } }, { job: 1 });

// Both return the same thing!
```

---

### Example 2: $not with $gt

```javascript
// Find employees with age NOT greater than 28
// (i.e., age <= 28)
db.emp.find({ age: { $not: { $gt: 28 } } }, { age: 1, _id: 0 });

// Returns: age <= 28
// { age: 28 } ✅
// { age: 27 } ✅
// { age: 20 } ✅
// { age: 29 } ❌
// { age: 30 } ❌
```

---

### Example 3: $not with $gte

```javascript
// Find employees with age NOT >= 28
// (i.e., age < 28)
db.emp.find({ age: { $not: { $gte: 28 } } }, { age: 1, _id: 0 });

// Returns: age < 28
// { age: 27 } ✅
// { age: 20 } ✅
// { age: 28 } ❌
// { age: 29 } ❌
```

---

### Equivalence Table

| $not Expression              | Equivalent To      | Result        |
| ---------------------------- | ------------------ | ------------- |
| `{ $not: { $gt: 28 } }`      | `{ $lte: 28 }`     | age ≤ 28      |
| `{ $not: { $gte: 28 } }`     | `{ $lt: 28 }`      | age < 28      |
| `{ $not: { $lt: 28 } }`      | `{ $gte: 28 }`     | age ≥ 28      |
| `{ $not: { $lte: 28 } }`     | `{ $gt: 28 }`      | age > 28      |
| `{ $not: { $eq: "clerk" } }` | `{ $ne: "clerk" }` | job ≠ "clerk" |
| `{ $not: { $ne: "clerk" } }` | `{ $eq: "clerk" }` | job = "clerk" |

---

## 11. Multiple Conditions — Critical Rules

### The Two Cases

When you have **multiple conditions** in a query, MongoDB behaves differently depending on whether they apply to the **same field** or **different fields**.

```
Case 1: Same field    →  Last condition wins (WRONG!)
Case 2: Different fields  →  Implicit AND (CORRECT!)
```

---

## 12. Case 1: Same Field — Last One Wins

### ⚠️ The Problem

When you apply **multiple conditions to the SAME field without $and**, only the **last condition** is used.

---

### Example: Range Query (WRONG Way)

```javascript
// ❌ WRONG — Trying to find salary between 1000 and 2000
db.emp.find(
  {
    sal: { $gte: 1000 }, // First condition
    sal: { $lte: 2000 }, // Second condition OVERWRITES first!
  },
  { sal: 1, _id: 0 },
);

// What MongoDB sees:
// { sal: { $lte: 2000 } }  ← Only this one!
// The $gte: 1000 is IGNORED

// Returns: All salaries <= 2000 (including 0, 500, 800...)
// NOT what you wanted!
```

---

### Why This Happens

JavaScript objects **cannot have duplicate keys**. The second `sal:` overwrites the first.

```javascript
let obj = {
  sal: { $gte: 1000 },
  sal: { $lte: 2000 }, // This replaces the first one!
};

console.log(obj);
// Output: { sal: { $lte: 2000 } }
// The first one is gone!
```

---

### ✅ The Solution: Use $and

```javascript
// ✅ CORRECT — Use $and for multiple conditions on same field
db.emp.find(
  {
    $and: [
      { sal: { $gte: 1000 } }, // Condition 1
      { sal: { $lte: 2000 } }, // Condition 2
    ],
  },
  { sal: 1, _id: 0, empName: 1 },
);

// Returns: Salaries between 1000 and 2000 (inclusive)
// { sal: 1000 } ✅
// { sal: 1500 } ✅
// { sal: 2000 } ✅
// { sal: 500 }  ❌
// { sal: 2500 } ❌
```

---

### Another Example: Same Field Problem

```javascript
// ❌ WRONG — Trying to find job = "manager" OR job = "clerk"
db.emp.find(
  {
    job: "manager",
    job: "clerk", // Overwrites "manager"
  },
  { job: 1, _id: 0 },
);

// Only finds: job = "clerk"
// Does NOT find managers!

// ✅ CORRECT — Use $in instead
db.emp.find({ job: { $in: ["manager", "clerk"] } }, { job: 1, _id: 0 });
```

---

### Visual: Same Field Problem

```
Query: { sal: { $gte: 1000 }, sal: { $lte: 2000 } }

Step 1: Create object
  { sal: { $gte: 1000 } }

Step 2: Add second sal
  { sal: { $lte: 2000 } }  ← Overwrites!

Final object:
  { sal: { $lte: 2000 } }  ← First condition lost!

────────────────────────────────────

✅ Solution: Use $and
Query: { $and: [ {sal: {$gte: 1000}}, {sal: {$lte: 2000}} ] }

Now both conditions are preserved!
```

---

## 13. Case 2: Different Fields — Implicit AND

### ✅ Good News!

When you apply conditions to **different fields**, MongoDB treats it as **AND** automatically.

---

### Example 1: Different Fields

```javascript
// Find employees with age > 30 AND deptNo = 20
db.emp.find(
  {
    age: { $gt: 30 }, // Condition on 'age'
    deptNo: 20, // Condition on 'deptNo'
  },
  { age: 1, deptNo: 1, _id: 0 },
);

// MongoDB treats this as:
// { $and: [ { age: { $gt: 30 } }, { deptNo: 20 } ] }

// Returns ONLY documents where BOTH are true:
// { age: 35, deptNo: 20 } ✅
// { age: 32, deptNo: 20 } ✅
// { age: 35, deptNo: 10 } ❌ (wrong dept)
// { age: 25, deptNo: 20 } ❌ (age too low)
```

---

### Example 2: Explicit vs Implicit AND

```javascript
// Both queries are EQUIVALENT:

// Version 1: Implicit AND (simpler)
db.emp.find({ age: { $gt: 30 }, deptNo: 20 });

// Version 2: Explicit $and (more verbose)
db.emp.find({
  $and: [{ age: { $gt: 30 } }, { deptNo: 20 }],
});

// Use implicit version when possible (cleaner code)
```

---

### When to Use Explicit $and

**Use explicit `$and` when:**

1. Multiple conditions on the **same field**
2. You want to be very clear about the logic
3. Mixing with other logical operators

```javascript
// Example: Multiple conditions on same field + different field
db.emp.find({
  $and: [
    { sal: { $gte: 1000 } }, // Same field
    { sal: { $lte: 2000 } }, // Same field
    { empName: "adams" }, // Different field
  ],
});
```

---

### Visual: Different Fields

```
Query: { age: { $gt: 30 }, deptNo: 20 }

MongoDB interprets as:
{ $and: [ { age: { $gt: 30 } }, { deptNo: 20 } ] }

Different fields → Implicit AND (automatic!)

────────────────────────────────────

Document: { age: 35, deptNo: 20 }
  age > 30?   ✅ True
  deptNo = 20? ✅ True
  Result: ✅ Match!

Document: { age: 25, deptNo: 20 }
  age > 30?   ❌ False
  deptNo = 20? ✅ True
  Result: ❌ No match (both must be true)
```

---

## 14. Complete Operator Reference

### Comparison Operators

| Operator | Syntax                          | Example                       | Matches               |
| -------- | ------------------------------- | ----------------------------- | --------------------- |
| `$eq`    | `{ field: { $eq: value } }`     | `{ age: { $eq: 30 } }`        | age = 30              |
| `$ne`    | `{ field: { $ne: value } }`     | `{ age: { $ne: 30 } }`        | age ≠ 30              |
| `$gt`    | `{ field: { $gt: value } }`     | `{ age: { $gt: 30 } }`        | age > 30              |
| `$gte`   | `{ field: { $gte: value } }`    | `{ age: { $gte: 30 } }`       | age ≥ 30              |
| `$lt`    | `{ field: { $lt: value } }`     | `{ age: { $lt: 30 } }`        | age < 30              |
| `$lte`   | `{ field: { $lte: value } }`    | `{ age: { $lte: 30 } }`       | age ≤ 30              |
| `$in`    | `{ field: { $in: [v1, v2] } }`  | `{ age: { $in: [20, 30] } }`  | age = 20 OR age = 30  |
| `$nin`   | `{ field: { $nin: [v1, v2] } }` | `{ age: { $nin: [20, 30] } }` | age ≠ 20 AND age ≠ 30 |

---

### Logical Operators

| Operator | Syntax                        | Example                                    | Logic                        |
| -------- | ----------------------------- | ------------------------------------------ | ---------------------------- |
| `$and`   | `{ $and: [{c1}, {c2}] }`      | `{ $and: [{age: {$gt: 25}}, {dept: 10}] }` | c1 AND c2 (both true)        |
| `$or`    | `{ $or: [{c1}, {c2}] }`       | `{ $or: [{age: {$gt: 25}}, {dept: 10}] }`  | c1 OR c2 (at least one true) |
| `$nor`   | `{ $nor: [{c1}, {c2}] }`      | `{ $nor: [{age: {$gt: 25}}, {dept: 10}] }` | NOT (c1 OR c2) (both false)  |
| `$not`   | `{ field: { $not: {expr} } }` | `{ age: { $not: { $gt: 28 } } }`           | Invert expression (age ≤ 28) |

---

### Quick Patterns

```javascript
// 1. Simple equality
{ name: "John" }                          // Implicit $eq

// 2. Range query (MUST use $and for same field)
{ $and: [{age: {$gte: 20}}, {age: {$lte: 30}}] }

// 3. Multiple values (use $in)
{ city: { $in: ["NYC", "LA", "Chicago"] } }

// 4. Different fields (implicit AND)
{ age: { $gt: 25 }, dept: 10 }

// 5. Complex logic
{
    $and: [
        { $or: [{age: {$lt: 20}}, {age: {$gt: 60}}] },
        { status: "active" }
    ]
}
```

---

## 15. Summary — Key Takeaways

### 🎯 Operator Rules

| Rule             | Detail                                         |
| ---------------- | ---------------------------------------------- |
| **$ prefix**     | All MongoDB operators start with $             |
| **Comparison**   | 8 operators: eq, ne, gt, gte, lt, lte, in, nin |
| **Logical**      | 4 operators: and, or, nor, not                 |
| **Implicit $eq** | `{name: "John"}` = `{name: {$eq: "John"}}`     |

---

### ⚠️ Critical Cases

```
Same field, multiple conditions:
  ❌ { sal: {$gte: 1000}, sal: {$lte: 2000} }  (last wins)
  ✅ { $and: [{sal: {$gte: 1000}}, {sal: {$lte: 2000}}] }

Different fields, multiple conditions:
  ✅ { age: {$gt: 30}, dept: 10 }  (implicit AND)
```

---

### 📊 Truth Tables

```
$and:  Both must be true
$or:   At least one must be true
$nor:  Both must be false
$not:  Invert the result
```

---

### 🔄 Equivalences

```
$not + $gt   = $lte
$not + $gte  = $lt
$not + $lt   = $gte
$not + $lte  = $gt
$not + $eq   = $ne
$not + $ne   = $eq
```

---

## 16. Revision Checklist

### Operators Basics

- [ ] Do you know all operators start with $?
- [ ] Can you name all 8 comparison operators?
- [ ] Can you name all 4 logical operators?
- [ ] Do you understand implicit $eq?

### Comparison Operators

- [ ] Can you use $eq (equals)?
- [ ] Can you use $ne (not equals)?
- [ ] Can you use $gt (greater than)?
- [ ] Can you use $gte (greater or equal)?
- [ ] Can you use $lt (less than)?
- [ ] Can you use $lte (less or equal)?
- [ ] Can you use $in with multiple values?
- [ ] Can you use $nin to exclude values?

### Logical Operators

- [ ] Can you combine conditions with $and?
- [ ] Can you use $or for "any match" logic?
- [ ] Can you use $nor for "none match" logic?
- [ ] Can you invert expressions with $not?
- [ ] Do you know $not syntax is different?

### Critical Cases

- [ ] Do you know same field + multiple conditions needs $and?
- [ ] Can you explain why the last condition wins without $and?
- [ ] Do you know different fields = implicit AND?
- [ ] Can you write range queries correctly?

### Equivalences

- [ ] Do you know $not + $gt = $lte?
- [ ] Do you know $in vs $or difference?
- [ ] Do you know $nin = NOT + $in?
- [ ] Can you convert $not expressions to equivalents?

---

> **🎤 Interview Tip — "What's the difference between $and and multiple fields in a query?"**
>
> **Answer like this:**
>
> _"When you have multiple conditions on DIFFERENT fields, MongoDB implicitly treats them as AND — you don't need to explicitly use $and. For example, `{ age: 30, dept: 10 }` automatically means 'age = 30 AND dept = 10'._
>
> _However, when you have multiple conditions on the SAME field, you MUST use $and explicitly. Without it, JavaScript's object behavior causes the last condition to overwrite the previous ones. For example:_
>
> _`{ sal: {$gte: 1000}, sal: {$lte: 2000} }` — this is WRONG because the second `sal` overwrites the first, so you only get `sal <= 2000`._
>
> _The correct way is: `{ $and: [{sal: {$gte: 1000}}, {sal: {$lte: 2000}}] }` — now both conditions are preserved._
>
> _So the rule is: different fields = implicit AND (no $and needed), same field = explicit $and required."_
>
> **Why this works:** Shows you understand both the convenience feature (implicit AND) AND the critical gotcha (same field problem) with a clear example and explanation of WHY it happens.
