# MongoDB Advanced Queries — The Complete Playground

### Nested Properties, Dates, Arrays & Element Operators

---

## 📚 What You'll Learn

This guide covers **advanced MongoDB query techniques** with real-world examples:

✅ Accessing nested object properties (performance.rating, address.city)  
✅ Querying dates with ISODate  
✅ Searching by ObjectId (\_id queries)  
✅ Filtering arrays (skills, tags, cart items)  
✅ Array operators: $all, $elemMatch, $size  
✅ Element operators: $exists, $type  
✅ Real project examples (e-commerce, HR system, social media)

**Best for:** MongoDB mastery, complex queries, interview preparation, real-world projects

---

## Table of Contents

1. [Nested Properties — Dot Notation](#1-nested-properties--dot-notation)
2. [Date Queries — ISODate Format](#2-date-queries--isodate-format)
3. [ObjectId Queries — Finding by \_id](#3-objectid-queries--finding-by-_id)
4. [Array Queries — Basic Filtering](#4-array-queries--basic-filtering)
5. [$all Operator — Match All Values](#5-all-operator--match-all-values)
6. [$in vs $all — The Difference](#6-in-vs-all--the-difference)
7. [$size Operator — Array Length](#7-size-operator--array-length)
8. [$elemMatch — Complex Array Queries](#8-elemmatch--complex-array-queries)
9. [$exists Operator — Field Presence](#9-exists-operator--field-presence)
10. [$type Operator — Data Type Checking](#10-type-operator--data-type-checking)
11. [Real-World Project Examples](#11-real-world-project-examples)
12. [Complete Operator Reference](#12-complete-operator-reference)
13. [Summary](#13-summary)
14. [Revision Checklist](#14-revision-checklist)

---

## 1. Nested Properties — Dot Notation

### The Problem

**Documents often have nested objects:**

```javascript
{
  _id: 1,
  empName: "Alice",
  performance: {
    rating: 4.5,
    lastReviewDate: "2024-01-15"
  }
}
```

**Question:** How do you query `rating` inside `performance`?

---

### The Solution — Dot Notation

**Use quotes + dot notation:** `"parentField.childField"`

```javascript
// Find employees with rating > 4.3
db.emp.find({ "performance.rating": { $gt: 4.3 } });
```

**Rule:** Always use **double quotes** for nested properties!

---

### Basic Example

```javascript
// Document structure:
{
  name: "Bob",
  performance: {
    rating: 4.7,
    lastReviewDate: ISODate("2024-01-20")
  }
}

// ✅ CORRECT — With quotes
db.emp.find({ "performance.rating": { $gt: 4.5 } });

// ❌ WRONG — Without quotes
db.emp.find({ performance.rating: { $gt: 4.5 } });  // Syntax error!
```

---

### Fetching Nested Fields in Projection

**Problem:** You want to show only the rating, not the entire performance object.

```javascript
// Show only employee name and rating
db.emp.find(
  { "performance.rating": { $gt: 4.3 } },
  {
    empName: 1,
    "performance.rating": 1, // ← Nested in projection too!
    _id: 0,
  },
);

// Output:
[
  { empName: "Alice", performance: { rating: 4.5 } },
  { empName: "Bob", performance: { rating: 4.7 } },
];
```

---

### Multiple Levels Deep

**You can go as deep as needed:**

```javascript
// 3 levels deep
db.users.find({ "profile.settings.notifications.email": true });

// 4 levels deep
db.products.find({ "details.specs.dimensions.weight": { $lt: 5 } });
```

---

### 🎯 Quick Rules for Nested Properties

| Rule                     | Example                                         |
| ------------------------ | ----------------------------------------------- |
| **Use quotes**           | `"performance.rating"` not `performance.rating` |
| **Use dot notation**     | `"parent.child.grandchild"`                     |
| **Works in filters**     | `{ "address.city": "NYC" }`                     |
| **Works in projections** | `{ "address.city": 1 }`                         |

---

## 2. Date Queries — ISODate Format

### Understanding Dates in MongoDB

**MongoDB stores dates in a special format:** ISODate (ISO 8601 standard)

```
Full format: YYYY-MM-DDTHH:mm:ss.sssZ

Y = Year
M = Month
D = Day
T = Separator between date and time
H = Hour
m = Minute
s = Second
.sss = Milliseconds
Z = UTC offset (timezone)
```

---

### Basic Date Query

```javascript
// Find employees hired after Jan 31, 1982
db.emp.find(
  { hireDate: { $gt: ISODate("1982-01-31") } },
  { empName: 1, hireDate: 1, _id: 0 },
);

// Output:
[
  { empName: "Alice", hireDate: ISODate("1985-03-15") },
  { empName: "Bob", hireDate: ISODate("1990-07-20") },
];
```

**Format:** `ISODate("YYYY-MM-DD")`

---

### Date Range Queries

```javascript
// Find employees hired in 1985
db.emp.find({
  $and: [
    { hireDate: { $gte: ISODate("1985-01-01") } },
    { hireDate: { $lt: ISODate("1986-01-01") } },
  ],
});

// Or using shorthand:
db.emp.find({
  hireDate: {
    $gte: ISODate("1985-01-01"),
    $lt: ISODate("1986-01-01"),
  },
});
```

---

### Including Time

```javascript
// Specific date and time
db.logs.find({
  timestamp: { $gt: ISODate("2024-02-21T10:30:00Z") },
});

// With milliseconds
db.events.find({
  createdAt: ISODate("2024-02-21T14:25:30.500Z"),
});
```

---

### Common Date Queries

| Query            | Example                                     |
| ---------------- | ------------------------------------------- |
| **Before date**  | `{ date: { $lt: ISODate("2024-01-01") } }`  |
| **After date**   | `{ date: { $gt: ISODate("2024-01-01") } }`  |
| **Exact date**   | `{ date: ISODate("2024-01-01") }`           |
| **Date range**   | `{ date: { $gte: start, $lte: end } }`      |
| **Current year** | `{ date: { $gte: ISODate("2024-01-01") } }` |

---

## 3. ObjectId Queries — Finding by \_id

### What is ObjectId Again?

**Remember:** ObjectId is a **12-byte unique identifier** that MongoDB auto-generates.

```javascript
_id: ObjectId("66a23517b5c6990483c4e49b")
              └─────────┬────────────────┘
                   24 hex characters
```

---

### Querying by \_id

```javascript
// ✅ CORRECT — Wrap in ObjectId()
db.emp.findOne({ _id: ObjectId("66a23517b5c6990483c4e49b") });

// ❌ WRONG — String won't work
db.emp.findOne({ _id: "66a23517b5c6990483c4e49b" }); // Returns null
```

**Rule:** Always use `ObjectId()` wrapper for \_id queries!

---

### Why the ObjectId() Wrapper?

```javascript
// MongoDB stores _id as ObjectId type, not string
typeof ObjectId("66a23517b5c6990483c4e49b"); // "object"
typeof "66a23517b5c6990483c4e49b"; // "string"

// Comparing different types returns no matches
"66a23517b5c6990483c4e49b" === ObjectId("66a23517b5c6990483c4e49b");
// false (different types!)
```

---

### Finding Multiple by \_id

```javascript
// Find specific users by IDs
db.users.find({
  _id: {
    $in: [
      ObjectId("66a23517b5c6990483c4e49b"),
      ObjectId("66a23517b5c6990483c4e49c"),
      ObjectId("66a23517b5c6990483c4e49d"),
    ],
  },
});
```

---

### Real-World Example: Shopping Cart

```javascript
// User clicks "View Details" on product
const productId = "66a23517b5c6990483c4e49b"; // From URL or button

// Fetch product details
db.products.findOne({ _id: ObjectId(productId) });

// Add to cart with multiple products
const cartItems = [
  "66a23517b5c6990483c4e49b",
  "66a23517b5c6990483c4e49c",
  "66a23517b5c6990483c4e49d",
];

db.products.find({
  _id: { $in: cartItems.map((id) => ObjectId(id)) },
});
```

---

### 12-Byte Requirement

```javascript
// ✅ CORRECT — Exactly 24 hex characters (12 bytes)
ObjectId("66a23517b5c6990483c4e49b");

// ❌ WRONG — Too short
ObjectId("12345"); // Error!

// ❌ WRONG — Too long
ObjectId("66a23517b5c6990483c4e49b123456"); // Error!

// ❌ WRONG — Invalid characters (not hex)
ObjectId("66a23517b5c6990483c4e49z"); // Error! ('z' not hex)
```

---

## 4. Array Queries — Basic Filtering

### Documents with Arrays

```javascript
// Employee document:
{
  _id: 1,
  empName: "Alice",
  skills: ["JavaScript", "Python", "SQL"]
}
```

---

### Finding by Single Value

```javascript
// Find employees who know SQL
db.emp.find({ skills: "SQL" }, { skills: 1, _id: 0 });

// Output:
[
  { skills: ["JavaScript", "Python", "SQL"] },
  { skills: ["SQL", "MongoDB", "Node.js"] },
];
```

**How it works:** MongoDB checks if `"SQL"` exists **anywhere** in the array.

---

### The Wrong Way (Doesn't Work as Expected)

```javascript
// ❌ WRONG — This doesn't work as you'd expect
db.emp.find({ skills: "SQL", skills: "Python" }, { skills: 1, _id: 0 });
// Last condition wins! Only checks for "Python"

// ❌ WRONG — This checks for EXACT array match
db.emp.find({ skills: ["SQL", "Python"] }, { skills: 1, _id: 0 });
// Only matches if skills = ["SQL", "Python"] EXACTLY (same order!)
```

---

### The Right Way — Using $and

```javascript
// ✅ CORRECT — Find employees with BOTH SQL AND Python
db.emp.find(
  {
    $and: [{ skills: "SQL" }, { skills: "Python" }],
  },
  { skills: 1, _id: 0 },
);

// Output:
[
  { skills: ["JavaScript", "Python", "SQL"] },
  { skills: ["Python", "SQL", "MongoDB"] },
];
```

---

### Visual: Array Matching

```
Document: { skills: ["JavaScript", "Python", "SQL"] }

Query: { skills: "SQL" }
Check: Is "SQL" in array? ✅ YES → Match!

Query: { skills: ["SQL", "Python"] }
Check: Does array EXACTLY equal ["SQL", "Python"]? ❌ NO → No match
(Order matters! And "JavaScript" is extra)

Query: { $and: [{ skills: "SQL" }, { skills: "Python" }] }
Check: Is "SQL" in array? ✅ YES
       Is "Python" in array? ✅ YES → Match!
```

---

## 5. $all Operator — Match All Values

### What Does $all Do?

**Finds documents where an array contains ALL the specified values** (in any order).

Think of it like:

- **"Must have ALL of these ingredients"**
- **"I want developers who know BOTH JavaScript AND React"**
- **"Products that have ALL these features"**

---

### Syntax

```javascript
{
  fieldName: {
    $all: [value1, value2, value3];
  }
}
```

---

### Basic Example

```javascript
// Find employees who know BOTH SQL AND Python
db.emp.find(
  { skills: { $all: ["SQL", "Python"] } },
  { empName: 1, skills: 1, _id: 0 }
);

// Matches:
{ empName: "Alice", skills: ["JavaScript", "Python", "SQL"] }  ✅
{ empName: "Bob", skills: ["Python", "SQL", "MongoDB"] }       ✅
{ empName: "Charlie", skills: ["SQL", "Java"] }                ❌ (no Python)
{ empName: "David", skills: ["Python", "Ruby"] }               ❌ (no SQL)
```

---

### Order Doesn't Matter

```javascript
// These are IDENTICAL:
db.emp.find({ skills: { $all: ["SQL", "Python"] } });
db.emp.find({ skills: { $all: ["Python", "SQL"] } });

// Both match:
["Python", "SQL"]           ✅
["SQL", "Python"]           ✅
["JavaScript", "SQL", "Python"]  ✅
```

---

### Real-World Example: Job Board

```javascript
// Document:
{
  jobTitle: "Full Stack Developer",
  requiredSkills: ["JavaScript", "React", "Node.js", "MongoDB"]
}

// Find developers who know React AND Node.js
db.candidates.find({
  skills: { $all: ["React", "Node.js"] }
});

// Find frontend roles requiring HTML, CSS, and JavaScript
db.jobs.find({
  requiredSkills: { $all: ["HTML", "CSS", "JavaScript"] }
});
```

---

### E-Commerce Example

```javascript
// Product document:
{
  name: "Laptop",
  features: ["Touchscreen", "Backlit Keyboard", "SSD", "USB-C"]
}

// Find laptops with Touchscreen AND USB-C
db.products.find({
  features: { $all: ["Touchscreen", "USB-C"] }
});

// Customer says: "I want touchscreen, SSD, and USB-C"
db.products.find({
  features: { $all: ["Touchscreen", "SSD", "USB-C"] }
});
```

---

## 6. $in vs $all — The Difference

### The Critical Difference

```
$in  → Match ANY one (OR logic)
$all → Match ALL (AND logic)
```

---

### Side-by-Side Comparison

```javascript
// Sample documents:
{
  skills: ["JavaScript", "Python", "SQL"];
} // Doc A
{
  skills: ["Python", "Ruby"];
} // Doc B
{
  skills: ["SQL", "Java"];
} // Doc C
{
  skills: ["JavaScript", "TypeScript"];
} // Doc D
```

**Query 1: $in (OR logic)**

```javascript
db.emp.find({ skills: { $in: ["SQL", "Python"] } });

// Matches if array has SQL OR Python (or both)
// Results: Doc A ✅, Doc B ✅, Doc C ✅, Doc D ❌
```

**Query 2: $all (AND logic)**

```javascript
db.emp.find({ skills: { $all: ["SQL", "Python"] } });

// Matches only if array has SQL AND Python (both required)
// Results: Doc A ✅, Doc B ❌, Doc C ❌, Doc D ❌
```

---

### Real-World Example: Restaurant Menu

```javascript
// Menu items:
{ name: "Veggie Pizza", toppings: ["Mushrooms", "Peppers", "Onions"] }
{ name: "Supreme Pizza", toppings: ["Pepperoni", "Mushrooms", "Peppers", "Onions"] }
{ name: "Meat Lovers", toppings: ["Pepperoni", "Sausage", "Bacon"] }

// Customer 1: "I want ANY pizza with mushrooms OR peppers"
db.menu.find({
  toppings: { $in: ["Mushrooms", "Peppers"] }
});
// Returns: Veggie Pizza ✅, Supreme Pizza ✅, Meat Lovers ❌

// Customer 2: "I want pizza with BOTH mushrooms AND peppers"
db.menu.find({
  toppings: { $all: ["Mushrooms", "Peppers"] }
});
// Returns: Veggie Pizza ✅, Supreme Pizza ✅, Meat Lovers ❌
```

---

### When to Use Which

| Use Case            | Operator | Example                                        |
| ------------------- | -------- | ---------------------------------------------- |
| **"At least one"**  | `$in`    | "Find users who speak English OR Spanish"      |
| **"Must have all"** | `$all`   | "Find developers who know React AND Node.js"   |
| **"Any match"**     | `$in`    | "Products with red OR blue color"              |
| **"All required"**  | `$all`   | "Courses covering Python AND Django AND Flask" |

---

### Can $in Work on Non-Arrays?

```javascript
// ✅ YES! $in works on regular fields too
db.emp.find({ job: { $in: ["Manager", "Analyst"] } });

// ❌ $all ONLY works on arrays
db.emp.find({ job: { $all: ["Manager", "Analyst"] } }); // Wrong!
```

**Key difference:** `$in` is universal, `$all` is array-specific.

---

## 7. $size Operator — Array Length

### What Does $size Do?

**Finds documents where an array has a specific number of elements.**

Think of it like:

- **"Users who have exactly 3 friends"**
- **"Employees with exactly 2 skills"**
- **"Orders with exactly 5 items"**

---

### Syntax

```javascript
{
  fieldName: {
    $size: positiveInteger;
  }
}
```

---

### Basic Example

```javascript
// Find employees with exactly 2 skills
db.emp.find(
  { skills: { $size: 2 } },
  { empName: 1, skills: 1, _id: 0 }
);

// Matches:
{ empName: "Alice", skills: ["Python", "SQL"] }           ✅ (2 skills)
{ empName: "Bob", skills: ["JavaScript", "React"] }       ✅ (2 skills)
{ empName: "Charlie", skills: ["Python", "SQL", "Java"] } ❌ (3 skills)
{ empName: "David", skills: ["Node.js"] }                 ❌ (1 skill)
```

---

### Must Be Exact Match

```javascript
// ✅ WORKS
{
  skills: {
    $size: 2;
  }
} // Exactly 2

// ❌ DOESN'T WORK (can't use comparison operators with $size)
{
  skills: {
    $size: {
      $gt: 2;
    }
  }
} // Error!
{
  skills: {
    $size: {
      $gte: 2;
    }
  }
} // Error!
```

**Limitation:** `$size` only accepts exact numbers, not ranges!

---

### Real-World Example: Social Media

```javascript
// User document:
{
  username: "alice",
  friends: ["bob", "charlie", "david"]
}

// Find users with exactly 100 friends (milestone badge)
db.users.find({ friends: { $size: 100 } });

// Find users with no friends (prompt to add connections)
db.users.find({ friends: { $size: 0 } });

// Find users with exactly 1 friend (suggest more connections)
db.users.find({ friends: { $size: 1 } });
```

---

### E-Commerce Example: Shopping Cart

```javascript
// Cart document:
{
  userId: 123,
  cart: [
    { name: "Phone", price: 1200, qty: 2 },
    { name: "Laptop", price: 39999, qty: 1 }
  ]
}

// Find users with exactly 1 item in cart (upsell opportunity)
db.users.find({ cart: { $size: 1 } });

// Find users with empty cart (abandoned cart reminder)
db.users.find({ cart: { $size: 0 } });

// Find users with 5+ items (bulk discount eligible)
// ❌ Can't do: { cart: { $size: { $gte: 5 } } }
// ✅ Workaround: Use aggregation or extra field
```

---

### Workaround for Ranges

**Problem:** Can't do `$size: { $gt: 3 }`

**Solution 1:** Store array length separately

```javascript
{
  skills: ["Python", "SQL", "JavaScript"],
  skillCount: 3  // Store length
}

// Now you can query:
db.emp.find({ skillCount: { $gt: 2 } });
```

**Solution 2:** Use aggregation

```javascript
db.emp.aggregate([
  {
    $match: {
      $expr: { $gt: [{ $size: "$skills" }, 2] },
    },
  },
]);
```

---

## 8. $elemMatch — Complex Array Queries

### What Does $elemMatch Do?

**Matches documents where at least ONE array element satisfies ALL specified conditions.**

Think of it like:

- **"Find orders where at least one item costs > $100"**
- **"Users with at least one photo that's verified AND has > 100 likes"**
- **"Employees with at least one project that's completed AND high-priority"**

---

### When Do You Need $elemMatch?

**For arrays of objects** (not simple arrays like `["SQL", "Python"]`)

```javascript
// Simple array — NO need for $elemMatch
{
  skills: ["SQL", "Python", "JavaScript"];
}

// Array of objects — NEED $elemMatch
{
  cart: [
    { name: "Phone", price: 1200, qty: 2 },
    { name: "Charger", price: 25, qty: 1 },
  ];
}
```

---

### Syntax

```javascript
{ fieldName: { $elemMatch: { condition1, condition2, ... } } }
```

---

### Basic Example

```javascript
// Product catalog:
{
  productId: 1,
  cart: [
    { name: "Phone", price: 1200, qty: 2 },
    { name: "Laptop", price: 39999, qty: 1 },
    { name: "Mouse", price: 25, qty: 3 }
  ]
}

// Find users who have "Phone" in their cart
db.products.find({
  cart: { $elemMatch: { name: "Phone" } }
});
```

---

### Multiple Conditions

```javascript
// Find users with expensive items (price > $1000 AND qty >= 1)
db.products.find({
  cart: {
    $elemMatch: {
      price: { $gt: 1000 },
      qty: { $gte: 1 }
    }
  }
});

// Matches if ANY cart item satisfies BOTH conditions:
{ name: "Phone", price: 1200, qty: 2 }    ✅ (price > 1000 AND qty >= 1)
{ name: "Charger", price: 25, qty: 5 }    ❌ (price not > 1000)
```

---

### Real-World Example: Order Management

```javascript
// Order document:
{
  orderId: "ORD-001",
  items: [
    { product: "Laptop", price: 1500, quantity: 1, shipped: true },
    { product: "Mouse", price: 25, quantity: 2, shipped: false },
    { product: "Keyboard", price: 75, quantity: 1, shipped: true }
  ]
}

// Find orders with at least one unshipped item
db.orders.find({
  items: { $elemMatch: { shipped: false } }
});

// Find orders with expensive items (price > $100) that aren't shipped yet
db.orders.find({
  items: {
    $elemMatch: {
      price: { $gt: 100 },
      shipped: false
    }
  }
});

// Find orders with high-quantity items (qty > 5) of a specific product
db.orders.find({
  items: {
    $elemMatch: {
      product: "Laptop",
      quantity: { $gte: 5 }
    }
  }
});
```

---

### Social Media Example

```javascript
// User document:
{
  username: "alice",
  photos: [
    { url: "photo1.jpg", likes: 150, verified: true },
    { url: "photo2.jpg", likes: 50, verified: false },
    { url: "photo3.jpg", likes: 200, verified: true }
  ]
}

// Find users with at least one verified photo that has > 100 likes
db.users.find({
  photos: {
    $elemMatch: {
      verified: true,
      likes: { $gt: 100 }
    }
  }
});

// Returns: alice ✅ (photo1 and photo3 match)
```

---

### Why Not Just Use Multiple Conditions?

```javascript
// ❌ WRONG — Doesn't work as expected
db.products.find({
  "cart.name": "Phone",
  "cart.price": { $gt: 1000 },
});
// This checks if ANY item is "Phone" AND ANY item costs > $1000
// (Can be different items!)

// ✅ CORRECT — With $elemMatch
db.products.find({
  cart: {
    $elemMatch: {
      name: "Phone",
      price: { $gt: 1000 },
    },
  },
});
// This checks if THE SAME item is "Phone" AND costs > $1000
```

**Visual:**

```
Cart: [
  { name: "Phone", price: 500 },
  { name: "Laptop", price: 2000 }
]

Without $elemMatch:
  "cart.name": "Phone" → ✅ (Phone exists)
  "cart.price": { $gt: 1000 } → ✅ (Laptop is > 1000)
  Result: ✅ MATCH (but wrong! Phone is not > 1000)

With $elemMatch:
  Check if SAME item has name="Phone" AND price > 1000
  Result: ❌ NO MATCH (correct!)
```

---

## 9. $exists Operator — Field Presence

### What Does $exists Do?

**Checks whether a field exists or not** (regardless of its value, even if null).

Think of it like:

- **"Find users who have a phone number" (even if it's blank)**
- **"Find products missing a description"**
- **"Find employees with a performance review"**

---

### Syntax

```javascript
{
  fieldName: {
    $exists: true;
  }
} // Field exists
{
  fieldName: {
    $exists: false;
  }
} // Field does NOT exist
```

---

### Basic Example

```javascript
// Documents:
{ name: "Alice", email: "alice@example.com", phone: "123-4567" }
{ name: "Bob", email: "bob@example.com" }
{ name: "Charlie", phone: "987-6543" }

// Find users who HAVE a phone field
db.users.find({ phone: { $exists: true } });
// Returns: Alice ✅, Charlie ✅, Bob ❌

// Find users who DON'T have a phone field
db.users.find({ phone: { $exists: false } });
// Returns: Bob ✅, Alice ❌, Charlie ❌
```

---

### Exists vs Null

```javascript
// Documents:
{ name: "Alice", email: "alice@example.com" }
{ name: "Bob", email: null }
{ name: "Charlie" }

// Find documents where email field EXISTS (even if null)
db.users.find({ email: { $exists: true } });
// Returns: Alice ✅, Bob ✅ (even though Bob's email is null!), Charlie ❌

// Find documents where email is null (or doesn't exist)
db.users.find({ email: null });
// Returns: Bob ✅, Charlie ✅, Alice ❌
```

**Key difference:**

- `{ $exists: true }` → Field is present (value can be anything, even null)
- `{ field: null }` → Field is missing OR value is null

---

### Real-World Example: Data Validation

```javascript
// Product catalog with incomplete data:
{
  name: "Laptop",
  price: 1500,
  description: "High-performance laptop"
}
{
  name: "Mouse",
  price: 25
  // Missing description!
}

// Find products missing descriptions (need to add them)
db.products.find({ description: { $exists: false } });

// Find products that HAVE descriptions (quality check passed)
db.products.find({ description: { $exists: true } });
```

---

### User Profile Completion

```javascript
// User documents:
{
  username: "alice",
  email: "alice@example.com",
  bio: "Developer and coffee enthusiast",
  profilePicture: "alice.jpg"
}
{
  username: "bob",
  email: "bob@example.com"
  // Missing bio and profilePicture
}

// Find users with incomplete profiles (nudge them to complete)
db.users.find({
  $or: [
    { bio: { $exists: false } },
    { profilePicture: { $exists: false } }
  ]
});

// Find users with complete profiles (100% badge)
db.users.find({
  bio: { $exists: true },
  profilePicture: { $exists: true }
});
```

---

### Nested Fields

```javascript
// Employee documents:
{
  name: "Alice",
  performance: {
    rating: 4.5,
    lastReviewDate: ISODate("2024-01-15")
  }
}
{
  name: "Bob"
  // Missing entire performance object
}
{
  name: "Charlie",
  performance: {
    rating: 3.8
    // Missing lastReviewDate
  }
}

// Find employees WITH performance reviews
db.emp.find({ "performance": { $exists: true } });
// Returns: Alice ✅, Charlie ✅, Bob ❌

// Find employees missing their last review date
db.emp.find({ "performance.lastReviewDate": { $exists: false } });
// Returns: Bob ✅, Charlie ✅, Alice ❌
```

---

## 10. $type Operator — Data Type Checking

### What Does $type Do?

**Checks the data type of a field's value.**

Think of it like:

- **"Find documents where age is a number" (not a string like "25")**
- **"Find corrupted data where price is stored as string"**
- **"Find documents where date is actually a date object"**

---

### Syntax

```javascript
{
  fieldName: {
    $type: "datatype";
  }
}
```

---

### Common Data Types

| Type         | String Name              | Example Value           |
| ------------ | ------------------------ | ----------------------- |
| **String**   | `"string"`               | `"Hello"`               |
| **Number**   | `"number"` or `"double"` | `42`, `3.14`            |
| **Integer**  | `"int"`                  | `100`                   |
| **Boolean**  | `"bool"`                 | `true`, `false`         |
| **Array**    | `"array"`                | `[1, 2, 3]`             |
| **Object**   | `"object"`               | `{ key: "value" }`      |
| **Date**     | `"date"`                 | `ISODate("2024-01-01")` |
| **Null**     | `"null"`                 | `null`                  |
| **ObjectId** | `"objectId"`             | `ObjectId("...")`       |

---

### Basic Example

```javascript
// Documents with mixed types:
{ name: "Alice", age: 30 }           // age is number
{ name: "Bob", age: "25" }           // age is string!
{ name: "Charlie", age: null }       // age is null
{ name: "David" }                    // age doesn't exist

// Find documents where age is a number
db.users.find({ age: { $type: "number" } });
// Returns: Alice ✅, Bob ❌, Charlie ❌, David ❌

// Find documents where age is a string (data corruption!)
db.users.find({ age: { $type: "string" } });
// Returns: Bob ✅, Alice ❌, Charlie ❌, David ❌
```

---

### Data Validation Example

```javascript
// Product prices should be numbers, not strings
{
  name: "Laptop",
  price: 1500       // ✅ Correct (number)
}
{
  name: "Mouse",
  price: "25"       // ❌ Wrong (string!)
}
{
  name: "Keyboard",
  price: "99.99"    // ❌ Wrong (string!)
}

// Find products with incorrect price types (fix them)
db.products.find({ price: { $type: "string" } });
// Returns: Mouse ✅, Keyboard ✅, Laptop ❌

// Find products with correct price types
db.products.find({ price: { $type: "number" } });
// Returns: Laptop ✅, Mouse ❌, Keyboard ❌
```

---

### Date Type Checking

```javascript
// Subscription documents:
{
  userId: 1,
  endDate: ISODate("2024-12-31")  // ✅ Proper date
}
{
  userId: 2,
  endDate: "2024-12-31"           // ❌ String!
}
{
  userId: 3,
  endDate: null                   // ❌ Null
}

// Find subscriptions with proper date objects
db.subscriptions.find({ endDate: { $type: "date" } });
// Returns: userId 1 ✅, userId 2 ❌, userId 3 ❌

// Find corrupted date fields (stored as strings)
db.subscriptions.find({ endDate: { $type: "string" } });
// Returns: userId 2 ✅, userId 1 ❌, userId 3 ❌
```

---

### Array Type Checking

```javascript
// Documents:
{
  name: "Alice",
  skills: ["Python", "SQL"]       // ✅ Array
}
{
  name: "Bob",
  skills: "JavaScript"            // ❌ String!
}
{
  name: "Charlie",
  skills: { primary: "Python" }   // ❌ Object!
}

// Find documents where skills is an array
db.users.find({ skills: { $type: "array" } });
// Returns: Alice ✅, Bob ❌, Charlie ❌
```

---

### Multiple Type Checking

```javascript
// Find fields that are EITHER string OR number
db.users.find({
  age: { $type: ["string", "number"] },
});

// Find documents where status is null OR doesn't exist
db.orders.find({
  status: { $type: "null" },
});
```

---

### Real-World: Migration Validation

```javascript
// After data migration, verify all fields have correct types

// Check that all user IDs are ObjectId (not strings)
db.users.find({ _id: { $type: "string" } });
// Should return 0 documents (all IDs should be ObjectId)

// Check that all prices are numbers
db.products.find({ price: { $type: "string" } });
// Fix these documents (convert string to number)

// Check that all dates are date objects
db.orders.find({ createdAt: { $type: "string" } });
// Convert these to ISODate
```

---

## 11. Real-World Project Examples

### Project 1: E-Commerce Platform

```javascript
// Product Document:
{
  _id: ObjectId("66a23517b5c6990483c4e49b"),
  name: "Gaming Laptop",
  price: 1499.99,
  specs: {
    processor: "Intel i7",
    ram: 16,
    storage: 512
  },
  features: ["Backlit Keyboard", "Touchscreen", "USB-C", "SSD"],
  tags: ["Gaming", "Electronics", "Computers"],
  reviews: [
    { user: "john", rating: 5, verified: true, date: ISODate("2024-01-15") },
    { user: "sarah", rating: 4, verified: false, date: ISODate("2024-02-01") }
  ],
  stock: 25,
  dateAdded: ISODate("2023-12-01")
}

// Query 1: Find laptops with 16GB RAM and SSD
db.products.find({
  "specs.ram": 16,
  features: "SSD"
});

// Query 2: Find products added in last 30 days
const thirtyDaysAgo = new Date();
thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

db.products.find({
  dateAdded: { $gte: thirtyDaysAgo }
});

// Query 3: Find products with at least one verified 5-star review
db.products.find({
  reviews: {
    $elemMatch: {
      rating: 5,
      verified: true
    }
  }
});

// Query 4: Find products with exactly 3 tags (SEO check)
db.products.find({ tags: { $size: 3 } });

// Query 5: Find products missing stock information (needs update)
db.products.find({ stock: { $exists: false } });

// Query 6: Find products where price is stored as string (data error)
db.products.find({ price: { $type: "string" } });
```

---

### Project 2: HR Management System

```javascript
// Employee Document:
{
  _id: ObjectId("66a23517b5c6990483c4e49c"),
  empName: "Alice Johnson",
  email: "alice@company.com",
  hireDate: ISODate("2020-03-15"),
  department: {
    name: "Engineering",
    floor: 3,
    manager: "Bob Smith"
  },
  skills: ["JavaScript", "React", "Node.js", "MongoDB"],
  performance: {
    rating: 4.7,
    lastReviewDate: ISODate("2024-01-10"),
    projects: [
      { name: "Dashboard", status: "Completed", priority: "High" },
      { name: "API Rewrite", status: "In Progress", priority: "Medium" }
    ]
  },
  certifications: ["AWS", "MongoDB Professional"]
}

// Query 1: Find senior employees (hired before 2021)
db.emp.find({
  hireDate: { $lt: ISODate("2021-01-01") }
});

// Query 2: Find employees with performance rating > 4.5
db.emp.find({
  "performance.rating": { $gt: 4.5 }
});

// Query 3: Find developers who know React AND Node.js
db.emp.find({
  skills: { $all: ["React", "Node.js"] }
});

// Query 4: Find employees with exactly 2 certifications
db.emp.find({
  certifications: { $size: 2 }
});

// Query 5: Find employees with high-priority completed projects
db.emp.find({
  "performance.projects": {
    $elemMatch: {
      status: "Completed",
      priority: "High"
    }
  }
});

// Query 6: Find employees missing email (compliance issue)
db.emp.find({
  email: { $exists: false }
});

// Query 7: Find employees in Engineering on floor 3
db.emp.find({
  "department.name": "Engineering",
  "department.floor": 3
});
```

---

### Project 3: Social Media App

```javascript
// User Document:
{
  _id: ObjectId("66a23517b5c6990483c4e49d"),
  username: "alice_dev",
  email: "alice@example.com",
  profile: {
    bio: "Full-stack developer | Coffee addict",
    location: "San Francisco",
    website: "alice.dev"
  },
  posts: [
    {
      id: 1,
      content: "First post!",
      likes: 150,
      verified: true,
      date: ISODate("2024-01-20")
    },
    {
      id: 2,
      content: "Learning MongoDB",
      likes: 200,
      verified: true,
      date: ISODate("2024-02-15")
    }
  ],
  followers: ["bob", "charlie", "david"],
  following: ["eve", "frank"],
  interests: ["coding", "coffee", "travel"],
  accountType: "premium",
  joinDate: ISODate("2023-06-01")
}

// Query 1: Find users who joined in 2024
db.users.find({
  joinDate: {
    $gte: ISODate("2024-01-01"),
    $lt: ISODate("2025-01-01")
  }
});

// Query 2: Find users with viral posts (> 100 likes, verified)
db.users.find({
  posts: {
    $elemMatch: {
      likes: { $gt: 100 },
      verified: true
    }
  }
});

// Query 3: Find users with exactly 3 followers
db.users.find({
  followers: { $size: 3 }
});

// Query 4: Find users interested in coding AND coffee
db.users.find({
  interests: { $all: ["coding", "coffee"] }
});

// Query 5: Find users missing profile bio (prompt to complete)
db.users.find({
  "profile.bio": { $exists: false }
});

// Query 6: Find premium users in San Francisco
db.users.find({
  accountType: "premium",
  "profile.location": "San Francisco"
});
```

---

## 12. Complete Operator Reference

### Nested Properties

```javascript
{ "parent.child": value }
{ "level1.level2.level3": value }
```

### Dates

```javascript
{ date: ISODate("YYYY-MM-DD") }
{ date: { $gt: ISODate("2024-01-01") } }
{ date: { $gte: start, $lte: end } }
```

### ObjectId

```javascript
{
  _id: ObjectId("24-hex-chars");
}
{
  _id: {
    $in: [ObjectId("..."), ObjectId("...")];
  }
}
```

### Arrays (Basic)

```javascript
{
  array: "value";
} // Contains value
{
  $and: [{ array: "v1" }, { array: "v2" }];
} // Contains both
```

### Array Operators

```javascript
{
  array: {
    $all: ["v1", "v2"];
  }
} // Must have ALL
{
  array: {
    $in: ["v1", "v2"];
  }
} // Must have ANY
{
  array: {
    $size: 3;
  }
} // Exactly 3 elements
{
  array: {
    $elemMatch: {
      conditions;
    }
  }
} // Complex object matching
```

### Element Operators

```javascript
{
  field: {
    $exists: true;
  }
} // Field exists
{
  field: {
    $exists: false;
  }
} // Field doesn't exist
{
  field: {
    $type: "string";
  }
} // Type checking
{
  field: {
    $type: ["string", "number"];
  }
} // Multiple types
```

---

## 13. Summary — Key Takeaways

### 🎯 Operator Categories

| Category     | Operators               | Use Case              |
| ------------ | ----------------------- | --------------------- |
| **Nested**   | Dot notation            | Access nested objects |
| **Dates**    | ISODate()               | Query by date/time    |
| **Arrays**   | $all, $size, $elemMatch | Array filtering       |
| **Elements** | $exists, $type          | Field presence/type   |

---

### 🔍 Quick Decision Tree

```
Need to query nested field?
  → Use "parent.child" with quotes

Working with dates?
  → Use ISODate("YYYY-MM-DD")

Querying by _id?
  → Wrap in ObjectId()

Array of simple values?
  → Use $all (must have all) or $in (must have any)

Array of objects?
  → Use $elemMatch for complex conditions

Check if field exists?
  → Use $exists

Validate data types?
  → Use $type
```

---

## 14. Revision Checklist

### Nested Properties

- [ ] Can you query nested fields with dot notation?
- [ ] Do you remember to use quotes?
- [ ] Can you project nested fields?
- [ ] Can you go multiple levels deep?

### Dates

- [ ] Can you use ISODate("YYYY-MM-DD")?
- [ ] Can you query date ranges?
- [ ] Do you know dates are stored in UTC?
- [ ] Can you compare dates with $gt, $lt?

### ObjectId

- [ ] Do you wrap \_id in ObjectId()?
- [ ] Do you know it must be exactly 24 hex chars?
- [ ] Can you query multiple IDs with $in?

### Arrays (Basic)

- [ ] Can you find documents containing a value?
- [ ] Do you know why { arr: "v1", arr: "v2" } doesn't work?
- [ ] Can you use $and for multiple array values?

### $all Operator

- [ ] Can you find arrays containing ALL values?
- [ ] Do you know order doesn't matter?
- [ ] Can you explain $all vs $in?

### $size Operator

- [ ] Can you find arrays with exact length?
- [ ] Do you know $size can't use comparison operators?
- [ ] Can you find empty arrays with $size: 0?

### $elemMatch

- [ ] Can you query arrays of objects?
- [ ] Can you apply multiple conditions to same element?
- [ ] Do you know when to use $elemMatch vs regular query?

### $exists

- [ ] Can you check if field exists?
- [ ] Do you know the difference between $exists and null?
- [ ] Can you find missing fields?

### $type

- [ ] Can you check field data types?
- [ ] Can you validate data after migration?
- [ ] Can you find corrupted data?

---

> **🎤 Interview Tip — "How would you find all products with both 'touchscreen' and 'USB-C' features, where at least one review is 5 stars from a verified buyer?"**
>
> **Answer like this:**
>
> _"I'd use a combination of array operators. First, for finding products with both features, I'd use the $all operator since we need BOTH features present:_
>
> ```javascript
> db.products.find({
>   features: { $all: ["touchscreen", "USB-C"] },
> });
> ```
>
> _But we also need to check the reviews array for a 5-star verified review. Since reviews is an array of objects and we need multiple conditions on the SAME review (5 stars AND verified), I'd use $elemMatch:_
>
> ```javascript
> db.products.find({
>   features: { $all: ["touchscreen", "USB-C"] },
>   reviews: {
>     $elemMatch: {
>       rating: 5,
>       verified: true,
>     },
>   },
> });
> ```
>
> _The key is using $all for the simple array (features) and $elemMatch for the complex array of objects (reviews), ensuring all conditions on the review apply to the SAME review object, not different ones."_
>
> **Why this works:** Shows understanding of when to use $all vs $elemMatch, explains the reasoning clearly, provides working code, and demonstrates knowledge of MongoDB's array query behavior.
