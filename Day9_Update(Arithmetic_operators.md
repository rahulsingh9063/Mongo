# MongoDB Update Operators — Arithmetic & Arrays

### Complete Guide to $inc, $mul, $min, $max, $push, $addToSet & More

---

## 📚 What You'll Learn

This guide covers **MongoDB update operators for numbers and arrays**:

✅ Arithmetic operators ($inc, $mul, $min, $max)  
✅ Array operators ($push, $addToSet)  
✅ Modifiers ($each, $position, $sort, $slice)  
✅ When to use which operator  
✅ Common mistakes and gotchas  
✅ Real-world project examples

**Best for:** Data updates, counters, array manipulation, interview preparation

---

## Table of Contents

1. [Arithmetic Operators Overview](#1-arithmetic-operators-overview)
2. [$inc — Increment/Decrement](#2-inc--incrementdecrement)
3. [$mul — Multiply](#3-mul--multiply)
4. [$max — Update Maximum](#4-max--update-maximum)
5. [$min — Update Minimum](#5-min--update-minimum)
6. [Array Operators Overview](#6-array-operators-overview)
7. [$push — Add to Array](#7-push--add-to-array)
8. [$push + $each — Multiple Elements](#8-push--each--multiple-elements)
9. [$push + $position — Specify Index](#9-push--position--specify-index)
10. [$push + $sort — Sort Array](#10-push--sort--sort-array)
11. [$push + $slice — Limit Array Size](#11-push--slice--limit-array-size)
12. [$addToSet — Unique Elements Only](#12-addtoset--unique-elements-only)
13. [Comparison: $push vs $addToSet](#13-comparison-push-vs-addtoset)
14. [Real-World Project Examples](#14-real-world-project-examples)
15. [Summary](#15-summary)
16. [Revision Checklist](#16-revision-checklist)

---

## 1. Arithmetic Operators Overview

### What Are Arithmetic Operators?

**Operators that perform math operations on number fields** (add, subtract, multiply, compare).

Think of them like:

- **Calculator buttons** — perform calculations
- **Excel formulas** — auto-update values
- **Counters** — increment/decrement numbers

---

### The 4 Main Arithmetic Operators

```
$inc  → Increment or Decrement by value
$mul  → Multiply by value
$max  → Update only if new value is GREATER
$min  → Update only if new value is SMALLER
```

---

### When to Use Each

| Use Case             | Operator        | Example                                 |
| -------------------- | --------------- | --------------------------------------- |
| **Increase counter** | $inc            | Views, likes, inventory                 |
| **Decrease counter** | $inc (negative) | Stock, credits                          |
| **Double value**     | $mul            | Apply discount multiplier               |
| **Set upper limit**  | $max            | Ensure value doesn't go below threshold |
| **Set lower limit**  | $min            | Ensure value doesn't exceed limit       |

---

## 2. $inc — Increment/Decrement

### What Does $inc Do?

**Increases or decreases a number field by a specified amount.**

Think of it like:

- **"+1" button** on a counter
- **Volume knob** (turn up or down)
- **Thermometer** (temperature goes up/down)

---

### Syntax

```javascript
{
  $inc: {
    fieldName: value;
  }
}

// Positive value = Increase
{
  $inc: {
    age: 5;
  }
} // Add 5

// Negative value = Decrease
{
  $inc: {
    age: -3;
  }
} // Subtract 3
```

---

### Basic Example — Increase

```javascript
// Before:
{ name: "Alice", age: 25 }

// Update:
db.students.updateOne(
    { name: "Alice" },
    { $inc: { age: 2 } }
);

// After:
{ name: "Alice", age: 27 }
//                   ↑ 25 + 2 = 27
```

---

### Basic Example — Decrease

```javascript
// Before:
{ name: "Alice", age: 30 }

// Update:
db.students.updateOne(
    { name: "Alice" },
    { $inc: { age: -5 } }
);

// After:
{ name: "Alice", age: 25 }
//                   ↑ 30 - 5 = 25
```

---

### Multiple Documents

```javascript
// Increase age by 2 for all students in Gurugram
db.students.updateMany(
    { city: "Gurugram" },
    { $inc: { age: 2 } }
);

// Before:
{ name: "Alice", city: "Gurugram", age: 25 }
{ name: "Bob", city: "Gurugram", age: 30 }
{ name: "Charlie", city: "Delhi", age: 28 }

// After:
{ name: "Alice", city: "Gurugram", age: 27 }  // +2
{ name: "Bob", city: "Gurugram", age: 32 }    // +2
{ name: "Charlie", city: "Delhi", age: 28 }   // Unchanged
```

---

### If Field Doesn't Exist — Creates It!

```javascript
// Before:
{ name: "Alice", age: 25 }
// No "salary" field

// Update:
db.students.updateOne(
    { name: "Alice" },
    { $inc: { salary: 5000 } }
);

// After:
{ name: "Alice", age: 25, salary: 5000 }
//                         ↑ Field created with value 5000
```

**Rule:** If field doesn't exist, $inc creates it and sets it to the increment value.

---

### ⚠️ Cannot Use null with $inc

```javascript
// ❌ WRONG — null not allowed
db.students.updateOne({ name: "Alice" }, { $inc: { age: null } });
// Error: Cannot increment with non-numeric argument

// ✅ CORRECT — Use number
db.students.updateOne({ name: "Alice" }, { $inc: { age: 1 } });
```

---

### Real-World Examples

#### Example 1: Page View Counter

```javascript
// Increment views each time page is visited
db.articles.updateOne({ slug: "intro-to-mongodb" }, { $inc: { views: 1 } });

// 1st visit: views = 1
// 2nd visit: views = 2
// 3rd visit: views = 3
```

---

#### Example 2: Product Inventory

```javascript
// Decrease stock when product is sold
db.products.updateOne({ sku: "LAPTOP-001" }, { $inc: { stock: -1 } });

// Before: { sku: "LAPTOP-001", stock: 50 }
// After:  { sku: "LAPTOP-001", stock: 49 }

// Increase stock when restocked
db.products.updateOne({ sku: "LAPTOP-001" }, { $inc: { stock: 20 } });

// After: { sku: "LAPTOP-001", stock: 69 }
```

---

#### Example 3: User Points/Credits

```javascript
// Award points for completing task
db.users.updateOne({ username: "alice" }, { $inc: { points: 100 } });

// Deduct points for purchase
db.users.updateOne({ username: "alice" }, { $inc: { points: -50 } });
```

---

### Visual: How $inc Works

```
Before:
{ name: "Alice", age: 25 }

Query: { $inc: { age: 3 } }

Step 1: Get current value → 25
Step 2: Add increment value → 25 + 3 = 28
Step 3: Update field → age = 28

After:
{ name: "Alice", age: 28 }
```

---

## 3. $mul — Multiply

### What Does $mul Do?

**Multiplies a number field by a specified value.**

Think of it like:

- **"×2" button** on calculator
- **Zoom in/out** (scale by factor)
- **Doubling/halving** values

---

### Syntax

```javascript
{
  $mul: {
    fieldName: multiplier;
  }
}
```

---

### Basic Example

```javascript
// Before:
{ name: "Alice", age: 10 }

// Update:
db.students.updateOne(
    { name: "Alice" },
    { $mul: { age: 2 } }
);

// After:
{ name: "Alice", age: 20 }
//                   ↑ 10 × 2 = 20
```

---

### Multiple Documents

```javascript
// Double age for all students
db.students.updateMany(
    {},
    { $mul: { age: 2 } }
);

// Before:
{ name: "Alice", age: 25 }
{ name: "Bob", age: 30 }

// After:
{ name: "Alice", age: 50 }  // 25 × 2
{ name: "Bob", age: 60 }    // 30 × 2
```

---

### If Field Doesn't Exist — Creates with 0!

```javascript
// Before:
{ name: "Alice", age: 25 }
// No "salary" field

// Update:
db.students.updateOne(
    { name: "Alice" },
    { $mul: { salary: 3 } }
);

// After:
{ name: "Alice", age: 25, salary: 0 }
//                         ↑ Field created with 0 (because 0 × 3 = 0)
```

**Important:** $mul on non-existent field = 0 (different from $inc!)

---

### Using Decimals

```javascript
// Apply 10% discount (multiply by 0.9)
db.products.updateMany({ category: "electronics" }, { $mul: { price: 0.9 } });

// Before: { name: "Laptop", price: 1000 }
// After:  { name: "Laptop", price: 900 }  // 1000 × 0.9
```

---

### Real-World Examples

#### Example 1: Apply Discount

```javascript
// 20% discount (multiply by 0.8)
db.products.updateMany({ onSale: true }, { $mul: { price: 0.8 } });
```

#### Example 2: Compound Interest

```javascript
// 5% interest per year (multiply by 1.05)
db.accounts.updateMany({ type: "savings" }, { $mul: { balance: 1.05 } });

// Before: { balance: 10000 }
// After:  { balance: 10500 }  // 10000 × 1.05
```

---

## 4. $max — Update Maximum

### What Does $max Do?

**Updates the field ONLY if the new value is GREATER than the current value.**

Think of it like:

- **"Raise the bar"** — only go higher, never lower
- **High score** — only update if you beat it
- **Maximum temperature** — only update if hotter

---

### Syntax

```javascript
{
  $max: {
    fieldName: newValue;
  }
}
```

---

### The Rule

```
If newValue > currentValue → UPDATE
If newValue ≤ currentValue → NO CHANGE
```

---

### Basic Example — Updates

```javascript
// Before:
{ name: "Alice", bonus: 10 }

// Update with HIGHER value:
db.students.updateOne(
    { name: "Alice" },
    { $max: { bonus: 20 } }
);

// After:
{ name: "Alice", bonus: 20 }
//                    ↑ Updated (20 > 10)
```

---

### Basic Example — No Update

```javascript
// Before:
{ name: "Alice", bonus: 100 }

// Update with LOWER value:
db.students.updateOne(
    { name: "Alice" },
    { $max: { bonus: 50 } }
);

// After:
{ name: "Alice", bonus: 100 }
//                    ↑ No change (50 < 100)
```

---

### Visual: $max Logic

```
Current value: 100

Try $max: 200
  200 > 100? ✅ Yes → Update to 200

Try $max: 100
  100 > 100? ❌ No (equal) → Keep 100

Try $max: 50
  50 > 100? ❌ No → Keep 100
```

---

### If Field Doesn't Exist — Creates It!

```javascript
// Before:
{ name: "Alice", age: 25 }
// No "bonus" field

// Update:
db.students.updateOne(
    { name: "Alice" },
    { $max: { bonus: 15 } }
);

// After:
{ name: "Alice", age: 25, bonus: 15 }
//                         ↑ Field created with value 15
```

---

### Real-World Examples

#### Example 1: High Score

```javascript
// User gets new score
let newScore = 850;

db.users.updateOne({ username: "alice" }, { $max: { highScore: newScore } });

// Scenario 1: Current highScore = 700
// → Updates to 850 (new high score!)

// Scenario 2: Current highScore = 1000
// → Stays 1000 (didn't beat it)
```

---

#### Example 2: Salary Cap

```javascript
// Ensure salary is at least minimum
db.employees.updateMany(
    { department: "Sales" },
    { $max: { salary: 50000 } }
);

// Before:
{ name: "Alice", salary: 40000 }  → After: 50000 (raised to min)
{ name: "Bob", salary: 60000 }    → After: 60000 (already above)
```

---

## 5. $min — Update Minimum

### What Does $min Do?

**Updates the field ONLY if the new value is SMALLER than the current value.**

Think of it like:

- **"Lower the bar"** — only go down, never up
- **Price floor** — set maximum price
- **Minimum temperature** — only update if colder

---

### Syntax

```javascript
{
  $min: {
    fieldName: newValue;
  }
}
```

---

### The Rule

```
If newValue < currentValue → UPDATE
If newValue ≥ currentValue → NO CHANGE
```

---

### Basic Example — Updates

```javascript
// Before:
{ name: "Alice", bonus: 100 }

// Update with LOWER value:
db.students.updateOne(
    { name: "Alice" },
    { $min: { bonus: 50 } }
);

// After:
{ name: "Alice", bonus: 50 }
//                    ↑ Updated (50 < 100)
```

---

### Basic Example — No Update

```javascript
// Before:
{ name: "Alice", bonus: 10 }

// Update with HIGHER value:
db.students.updateOne(
    { name: "Alice" },
    { $min: { bonus: 20 } }
);

// After:
{ name: "Alice", bonus: 10 }
//                    ↑ No change (20 > 10)
```

---

### Visual: $min Logic

```
Current value: 100

Try $min: 50
  50 < 100? ✅ Yes → Update to 50

Try $min: 100
  100 < 100? ❌ No (equal) → Keep 100

Try $min: 200
  200 < 100? ❌ No → Keep 100
```

---

### $max vs $min Comparison

```javascript
// Starting value: 100

$max with 200:  100 → 200  ✅ (raise to 200)
$max with 50:   100 → 100  ❌ (keep 100, don't go down)

$min with 200:  100 → 100  ❌ (keep 100, don't go up)
$min with 50:   100 → 50   ✅ (lower to 50)
```

---

### Real-World Examples

#### Example 1: Price Ceiling

```javascript
// Ensure price doesn't exceed maximum
db.products.updateMany(
    { category: "books" },
    { $min: { price: 50 } }
);

// Before:
{ name: "Book A", price: 60 }  → After: 50 (capped at max)
{ name: "Book B", price: 30 }  → After: 30 (already below)
```

---

#### Example 2: Shortest Delivery Time

```javascript
// Track fastest delivery
db.orders.updateOne({ orderId: 12345 }, { $min: { fastestDeliveryDays: 3 } });

// Current: 5 days → Updates to 3 days (new record!)
// Current: 2 days → Stays 2 days (already faster)
```

---

## 6. Array Operators Overview

### What Are Array Operators?

**Operators that modify array fields** (add, remove, sort elements).

Think of them like:

- **Shopping cart** — add/remove items
- **Playlist** — add songs, reorder
- **Tag list** — manage tags

---

### The Main Array Operators

```
$push      → Add element(s) to array
$addToSet  → Add only unique element(s)
$pull      → Remove element(s)
$pop       → Remove first/last element
```

---

### Array Modifiers

```
$each      → Add multiple elements (with $push/$addToSet)
$position  → Specify index for insertion (with $push)
$sort      → Sort array after update (with $push)
$slice     → Limit array size (with $push)
```

---

## 7. $push — Add to Array

### What Does $push Do?

**Adds an element to the END of an array.**

Think of it like:

- **Append to list** — add to bottom
- **Queue** — join the line at the back
- **Stack** — push onto pile

---

### Syntax

```javascript
{
  $push: {
    arrayField: value;
  }
}
```

---

### Basic Example

```javascript
// Before:
{
    _id: 1,
    name: "Sri",
    skills: ["mongodb", "sql", "nodejs"]
}

// Update:
db.arrayUpdate.updateOne(
    { _id: 1 },
    { $push: { skills: "express" } }
);

// After:
{
    _id: 1,
    name: "Sri",
    skills: ["mongodb", "sql", "nodejs", "express"]
    //                                    ↑ Added at end
}
```

---

### If Array Doesn't Exist — Creates It!

```javascript
// Before:
{ _id: 1, name: "Sri", skills: ["mongodb"] }
// No "hobbies" field

// Update:
db.arrayUpdate.updateOne(
    { _id: 1 },
    { $push: { hobbies: "gaming" } }
);

// After:
{
    _id: 1,
    name: "Sri",
    skills: ["mongodb"],
    hobbies: ["gaming"]  // ← Created as array with one element
}
```

---

### ⚠️ Pushing an Array Creates Nested Array

```javascript
// Before:
{ _id: 2, skills: ["postgres", "sql"] }

// Update:
db.arrayUpdate.updateOne(
    { _id: 2 },
    { $push: { skills: [".net", "laravel"] } }
);

// After:
{
    _id: 2,
    skills: ["postgres", "sql", [".net", "laravel"]]
    //                           ↑ Nested array!
}
```

**Problem:** You wanted to add two elements, but got a nested array instead!

**Solution:** Use `$push` with `$each` (next section)

---

### Visual: How $push Works

```
Before:
skills: ["mongodb", "sql", "nodejs"]
         ↑           ↑     ↑

$push: { skills: "express" }

After:
skills: ["mongodb", "sql", "nodejs", "express"]
         ↑           ↑     ↑         ↑ Added here
```

---

## 8. $push + $each — Multiple Elements

### What Does $each Do?

**Allows adding MULTIPLE elements to array** (instead of just one).

Think of it like:

- **Bulk add** — add many at once
- **Import list** — add entire list
- **Batch insert**

---

### Syntax

```javascript
{
  $push: {
    arrayField: {
      $each: [value1, value2, value3];
    }
  }
}
```

**Rule:** $each MUST be used with $push (can't use alone)

---

### Basic Example

```javascript
// Before:
{ _id: 3, skills: ["python", "sql"] }

// Update:
db.arrayUpdate.updateOne(
    { _id: 3 },
    { $push: { skills: { $each: [".net", "django", "ai/ml"] } } }
);

// After:
{
    _id: 3,
    skills: ["python", "sql", ".net", "django", "ai/ml"]
    //                        ↑      ↑        ↑ All 3 added
}
```

---

### vs Pushing Array (Wrong Way)

```javascript
// ❌ WRONG — Creates nested array
{
  $push: {
    skills: [".net", "laravel"];
  }
}

// Result:
skills: ["postgres", "sql", [".net", "laravel"]];
//                           ↑ Nested!

// ✅ CORRECT — Adds multiple elements
{
  $push: {
    skills: {
      $each: [".net", "laravel"];
    }
  }
}

// Result:
skills: ["postgres", "sql", ".net", "laravel"];
//                           ↑       ↑ Separate elements
```

---

## 9. $push + $position — Specify Index

### What Does $position Do?

**Specifies WHERE to insert elements** (at which index).

Think of it like:

- **Insert at line 5** — specific position
- **Cut in line** — insert at specific spot
- **Array splice** — insert at index

---

### Syntax

```javascript
{
    $push: {
        arrayField: {
            $each: [values],
            $position: index
        }
    }
}
```

**Rule:** $position REQUIRES $each (can't use alone)

---

### Basic Example

```javascript
// Before:
{ _id: 1, skills: ["mongodb", "sql", "nodejs"] }
//                  ↑ Index 0  ↑ 1    ↑ 2

// Insert "react" at index 1:
db.arrayUpdate.updateOne(
    { _id: 1 },
    {
        $push: {
            skills: {
                $each: ["react"],
                $position: 1
            }
        }
    }
);

// After:
{ _id: 1, skills: ["mongodb", "react", "sql", "nodejs"] }
//                  ↑ Index 0  ↑ 1     ↑ 2    ↑ 3
//                             ↑ Inserted here!
```

---

### Position 0 — Insert at Beginning

```javascript
// Insert at start of array
{
    $push: {
        skills: {
            $each: ["html", "css"],
            $position: 0
        }
    }
}

// Before: ["mongodb", "sql"]
// After:  ["html", "css", "mongodb", "sql"]
//          ↑      ↑ Inserted at beginning
```

---

### No Position Specified — Adds at End

```javascript
// Without $position, adds to end (default behavior)
db.arrayUpdate.updateOne(
  { _id: 4 },
  { $push: { skills: { $each: ["html"] } } },
);

// Before: ["python", "django"]
// After:  ["python", "django", "html"]
//                              ↑ Added at end
```

---

## 10. $push + $sort — Sort Array

### What Does $sort Do?

**Sorts the array after adding elements.**

Think of it like:

- **Auto-organize** — alphabetize or numerically order
- **Leaderboard** — sort by score
- **Chronological order** — sort by date

---

### Syntax

```javascript
{
    $push: {
        arrayField: {
            $each: [values],
            $sort: 1 or -1
        }
    }
}

// $sort: 1  → Ascending (A-Z, 0-9, oldest-newest)
// $sort: -1 → Descending (Z-A, 9-0, newest-oldest)
```

---

### Ascending Sort (A-Z)

```javascript
// Before:
{ _id: 1, skills: ["nodejs", "mongodb", "sql"] }

// Add and sort ascending:
db.arrayUpdate.updateOne(
    { _id: 1 },
    {
        $push: {
            skills: {
                $each: ["react"],
                $sort: 1  // Ascending
            }
        }
    }
);

// After (alphabetically sorted):
{ _id: 1, skills: ["mongodb", "nodejs", "react", "sql"] }
//                  ↑ m        ↑ n       ↑ r     ↑ s (alphabetical)
```

---

### Descending Sort (Z-A)

```javascript
// Before:
{ _id: 2, skills: ["postgres", "sql", "php"] }

// Add and sort descending:
db.arrayUpdate.updateOne(
    { _id: 2 },
    {
        $push: {
            skills: {
                $each: ["react"],
                $sort: -1  // Descending
            }
        }
    }
);

// After (reverse alphabetical):
{ _id: 2, skills: ["sql", "react", "postgres", "php"] }
//                  ↑ s    ↑ r     ↑ p         ↑ p (reverse)
```

---

### Sort Without Adding (Empty $each)

```javascript
// Just sort existing array, don't add anything
db.arrayUpdate.updateOne(
  { _id: 1 },
  {
    $push: {
      skills: {
        $each: [], // Empty array (add nothing)
        $sort: 1, // Just sort
      },
    },
  },
);
```

---

### Sorting Numbers

```javascript
// Before:
{ _id: 1, scores: [85, 92, 78, 95] }

// Sort ascending:
{
    $push: {
        scores: {
            $each: [88],
            $sort: 1
        }
    }
}

// After:
{ _id: 1, scores: [78, 85, 88, 92, 95] }
//                  ↑ Sorted low to high
```

---

## 11. $push + $slice — Limit Array Size

### What Does $slice Do?

**Keeps only N elements after update** (removes the rest).

Think of it like:

- **Top 10 list** — keep only top 10
- **Recent history** — keep last 5 items
- **Memory limit** — prevent array from growing too large

---

### Syntax

```javascript
{
    $push: {
        arrayField: {
            $each: [values],
            $slice: N
        }
    }
}

// $slice: 3   → Keep first 3 elements
// $slice: -3  → Keep last 3 elements
```

---

### Keep First N Elements

```javascript
// Before:
{ _id: 1, skills: ["mongodb", "sql", "nodejs"] }

// Add and keep only first 2:
db.arrayUpdate.updateOne(
    { _id: 1 },
    {
        $push: {
            skills: {
                $each: ["react", "express"],
                $slice: 2  // Keep only first 2
            }
        }
    }
);

// After adding: ["mongodb", "sql", "nodejs", "react", "express"]
// After slicing: ["mongodb", "sql"]
//                 ↑ First 2 kept, rest removed
```

---

### Keep Last N Elements

```javascript
// Before:
{ _id: 1, skills: ["mongodb", "sql", "nodejs"] }

// Add and keep only last 2:
db.arrayUpdate.updateOne(
    { _id: 1 },
    {
        $push: {
            skills: {
                $each: ["react"],
                $slice: -2  // Keep only last 2
            }
        }
    }
);

// After adding: ["mongodb", "sql", "nodejs", "react"]
// After slicing: ["nodejs", "react"]
//                 ↑ Last 2 kept
```

---

### Visual: $slice Behavior

```
Original array:
[A, B, C, D, E]

$slice: 3  (keep first 3)
Result: [A, B, C]
        ↑  ↑  ↑

$slice: -3 (keep last 3)
Result: [C, D, E]
        ↑  ↑  ↑

$slice: -1 (keep last 1)
Result: [E]
        ↑
```

---

### Real-World Example: Recent Activity Log

```javascript
// Keep only last 5 activities
db.users.updateOne(
  { userId: 123 },
  {
    $push: {
      recentActivity: {
        $each: [
          {
            action: "login",
            timestamp: new Date(),
          },
        ],
        $slice: -5, // Keep only last 5 activities
      },
    },
  },
);

// Prevents recentActivity array from growing forever
```

---

### Combining $sort and $slice

```javascript
// Add, sort, then keep top 3
db.leaderboard.updateOne(
  { gameId: 1 },
  {
    $push: {
      topScores: {
        $each: [{ player: "Alice", score: 950 }],
        $sort: { score: -1 }, // Sort by score descending
        $slice: 3, // Keep only top 3
      },
    },
  },
);
```

---

## 12. $addToSet — Unique Elements Only

### What Does $addToSet Do?

**Adds element to array ONLY if it doesn't already exist** (prevents duplicates).

Think of it like:

- **Set in mathematics** — no duplicates allowed
- **Email subscription** — can't subscribe twice
- **Unique tags** — each tag appears once

---

### Syntax

```javascript
// Single element:
{
  $addToSet: {
    arrayField: value;
  }
}

// Multiple elements:
{
  $addToSet: {
    arrayField: {
      $each: [v1, v2, v3];
    }
  }
}
```

---

### Basic Example — Adds Unique

```javascript
// Before:
{ _id: 1, skills: ["mongodb", "sql", "nodejs"] }

// Try to add "python" (doesn't exist):
db.arrayUpdate.updateOne(
    { _id: 1 },
    { $addToSet: { skills: "python" } }
);

// After:
{ _id: 1, skills: ["mongodb", "sql", "nodejs", "python"] }
//                                              ↑ Added (was unique)
```

---

### Basic Example — Ignores Duplicate

```javascript
// Before:
{ _id: 1, skills: ["mongodb", "sql", "nodejs"] }

// Try to add "sql" (already exists):
db.arrayUpdate.updateOne(
    { _id: 1 },
    { $addToSet: { skills: "sql" } }
);

// After:
{ _id: 1, skills: ["mongodb", "sql", "nodejs"] }
//                            ↑ Not added (duplicate)
```

---

### With $each — Multiple Elements

```javascript
// Before:
{ _id: 3, skills: ["python", "sql"] }

// Add multiple unique elements:
db.arrayUpdate.updateOne(
    { _id: 3 },
    {
        $addToSet: {
            skills: {
                $each: [".net", "django", "sql", "ai/ml"]
            }
        }
    }
);

// After:
{
    _id: 3,
    skills: ["python", "sql", ".net", "django", "ai/ml"]
    //                 ↑ Already existed (not duplicated)
    //                        ↑      ↑        ↑ Added (were unique)
}
```

---

### ⚠️ Cannot Use $position, $sort, or $slice with $addToSet

```javascript
// ❌ WRONG — $addToSet doesn't support these modifiers
db.arrayUpdate.updateOne(
  { _id: 1 },
  {
    $addToSet: {
      skills: {
        $each: ["react"],
        $position: 1, // ERROR!
      },
    },
  },
);
// Error: $addToSet doesn't support $position

// ✅ CORRECT — Use $push for these features
db.arrayUpdate.updateOne(
  { _id: 1 },
  {
    $push: {
      skills: {
        $each: ["react"],
        $position: 1, // Works with $push
      },
    },
  },
);
```

---

### Real-World Examples

#### Example 1: User Tags

```javascript
// Add tags to article (no duplicates)
db.articles.updateOne(
  { articleId: 123 },
  { $addToSet: { tags: "javascript" } },
);

// Can be called multiple times, won't create duplicates:
// Call 1: tags = ["javascript"]
// Call 2: tags = ["javascript"] (not added again)
// Call 3: tags = ["javascript"] (still not added)
```

---

#### Example 2: Following List

```javascript
// User follows another user
db.users.updateOne({ username: "alice" }, { $addToSet: { following: "bob" } });

// If Alice already follows Bob, nothing happens
// Prevents duplicate follows
```

---

## 13. Comparison: $push vs $addToSet

### Side-by-Side Comparison

| Feature                      | $push                        | $addToSet          |
| ---------------------------- | ---------------------------- | ------------------ |
| **Allows duplicates?**       | ✅ Yes                       | ❌ No              |
| **Adds multiple with $each** | ✅ Yes                       | ✅ Yes             |
| **Supports $position**       | ✅ Yes                       | ❌ No              |
| **Supports $sort**           | ✅ Yes                       | ❌ No              |
| **Supports $slice**          | ✅ Yes                       | ❌ No              |
| **Use case**                 | Order matters, duplicates OK | Unique values only |

---

### When to Use Which

**Use $push when:**

- Order matters (chronological, priority)
- Duplicates are allowed (log entries, history)
- Need sorting or slicing
- Need to insert at specific position

**Use $addToSet when:**

- Must be unique (tags, categories)
- Duplicates would be wrong (subscriptions, follows)
- Order doesn't matter

---

### Example Comparison

```javascript
// Document:
{ _id: 1, skills: ["mongodb", "sql"] }

// Using $push (allows duplicate):
db.arrayUpdate.updateOne(
    { _id: 1 },
    { $push: { skills: "sql" } }
);
// Result: ["mongodb", "sql", "sql"]  ← Duplicate added

// Using $addToSet (prevents duplicate):
db.arrayUpdate.updateOne(
    { _id: 1 },
    { $addToSet: { skills: "sql" } }
);
// Result: ["mongodb", "sql"]  ← No duplicate
```

---

## 14. Real-World Project Examples

### Project 1: Social Media App

```javascript
// User Activity Feed (chronological, duplicates OK)
db.users.updateOne(
  { userId: 123 },
  {
    $push: {
      activityFeed: {
        $each: [
          {
            type: "post",
            content: "Hello World",
            timestamp: new Date(),
          },
        ],
        $position: 0, // Add to beginning
        $slice: 50, // Keep only 50 most recent
      },
    },
  },
);

// User Tags/Interests (unique only)
db.users.updateOne(
  { userId: 123 },
  {
    $addToSet: {
      interests: {
        $each: ["coding", "music", "travel"],
      },
    },
  },
);

// Post Likes (unique, can't like twice)
db.posts.updateOne(
  { postId: 456 },
  {
    $addToSet: { likes: 123 }, // userId 123
    $inc: { likeCount: 1 },
  },
);
```

---

### Project 2: E-Commerce Platform

```javascript
// Product Views Counter
db.products.updateOne({ sku: "LAPTOP-001" }, { $inc: { views: 1 } });

// Shopping Cart (duplicates OK if buying multiple)
db.carts.updateOne(
  { userId: 123 },
  {
    $push: {
      items: {
        productId: 456,
        quantity: 2,
        price: 1200,
      },
    },
  },
);

// Wishlist (unique items only)
db.users.updateOne(
  { userId: 123 },
  { $addToSet: { wishlist: 456 } }, // productId
);

// Apply 20% discount
db.products.updateMany({ category: "electronics" }, { $mul: { price: 0.8 } });

// Restock inventory
db.products.updateOne({ sku: "LAPTOP-001" }, { $inc: { stock: 50 } });
```

---

### Project 3: Learning Platform

```javascript
// Course Progress (increment completed lessons)
db.enrollments.updateOne(
  { userId: 123, courseId: 789 },
  {
    $inc: { completedLessons: 1 },
    $push: {
      completionHistory: {
        $each: [
          {
            lesson: "Intro to MongoDB",
            completedAt: new Date(),
          },
        ],
        $slice: -20, // Keep last 20
      },
    },
  },
);

// User Skills/Certificates (unique)
db.users.updateOne(
  { userId: 123 },
  {
    $addToSet: {
      certificates: {
        $each: ["MongoDB Professional", "Node.js Developer"],
      },
    },
  },
);

// Course Reviews (keep top 5 highest rated)
db.courses.updateOne(
  { courseId: 789 },
  {
    $push: {
      topReviews: {
        $each: [
          {
            userId: 123,
            rating: 5,
            comment: "Great course!",
          },
        ],
        $sort: { rating: -1 }, // Sort by rating descending
        $slice: 5, // Keep top 5
      },
    },
  },
);
```

---

### Project 4: Gaming Leaderboard

```javascript
// Update player score
db.players.updateOne(
  { username: "alice" },
  {
    $max: { highScore: 9500 }, // Only update if higher
    $inc: { totalGames: 1 }, // Increment games played
    $push: {
      recentScores: {
        $each: [9500],
        $slice: -10, // Keep last 10 scores
      },
    },
  },
);

// Global leaderboard (keep top 100)
db.leaderboard.updateOne(
  { gameMode: "classic" },
  {
    $push: {
      rankings: {
        $each: [{ username: "alice", score: 9500 }],
        $sort: { score: -1 }, // Sort by score descending
        $slice: 100, // Keep top 100 only
      },
    },
  },
);
```

---

## 15. Summary — Key Takeaways

### 🔢 Arithmetic Operators

```
$inc   → Add/subtract value (counter)
$mul   → Multiply value (percentage)
$max   → Update if greater (high score)
$min   → Update if smaller (price cap)
```

---

### 📊 Array Operators

```
$push       → Add to end (allows duplicates)
$addToSet   → Add if unique (no duplicates)
```

---

### 🎛️ Array Modifiers

```
$each       → Add multiple elements
$position   → Specify index (with $push only)
$sort       → Sort array (with $push only)
$slice      → Limit size (with $push only)
```

---

### ⚠️ Important Rules

```
✅ $inc creates field with value if doesn't exist
✅ $mul creates field with 0 if doesn't exist
✅ $push creates array if doesn't exist
✅ $addToSet creates array if doesn't exist

❌ Cannot use null with $inc
❌ Cannot use $position/$sort/$slice with $addToSet
❌ Pushing array creates nested array (use $each)
```

---

## 16. Revision Checklist

### Arithmetic Operators

- [ ] Can you use $inc to increment?
- [ ] Can you use $inc to decrement?
- [ ] Do you know $inc creates field if missing?
- [ ] Can you use $mul to multiply?
- [ ] Do you know $mul creates with 0 if missing?
- [ ] Can you use $max correctly?
- [ ] Can you use $min correctly?
- [ ] Do you know when to use $max vs $min?

### Array Basics

- [ ] Can you use $push to add element?
- [ ] Do you know $push adds to END?
- [ ] Can you use $addToSet for unique elements?
- [ ] Do you know difference between $push and $addToSet?

### $push Modifiers

- [ ] Can you use $each to add multiple?
- [ ] Can you use $position to specify index?
- [ ] Can you use $sort to sort array?
- [ ] Can you use $slice to limit size?
- [ ] Can you combine $sort and $slice?

### Restrictions

- [ ] Do you know $addToSet doesn't support $position?
- [ ] Do you know $addToSet doesn't support $sort?
- [ ] Do you know $addToSet doesn't support $slice?
- [ ] Do you know not to push array without $each?

---

> **🎤 Interview Tip — "How would you implement a 'top 10 leaderboard' that always shows the 10 highest scores?"**
>
> **Answer like this:**
>
> _"I'd use $push with $each, $sort, and $slice together. Here's how:_
>
> ```javascript
> db.leaderboard.updateOne(
>   { gameMode: "classic" },
>   {
>     $push: {
>       rankings: {
>         $each: [{ player: "Alice", score: 9500 }],
>         $sort: { score: -1 }, // Sort by score descending
>         $slice: 10, // Keep only top 10
>       },
>     },
>   },
> );
> ```
>
> _This approach combines three modifiers: $each adds the new score, $sort orders all scores from highest to lowest, and $slice keeps only the top 10. The key is using $sort with -1 for descending order, so the highest scores come first, and then $slice removes everything after position 10._
>
> _This is more efficient than sorting the entire array in application code or using aggregation pipelines. It's atomic, so there's no risk of race conditions with concurrent updates. The limitation is that all three modifiers only work with $push, not $addToSet, which is fine here since we might have duplicate scores from different players."_
>
> **Why this works:** Shows understanding of combining modifiers, explains the order of operations, mentions atomicity, discusses performance, and identifies the limitation with $addToSet.
