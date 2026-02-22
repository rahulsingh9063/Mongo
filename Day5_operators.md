# MongoDB Operators Reference

> All MongoDB operators are prefixed with `$`. This reference covers operators commonly used in real-world production projects, organized to match the **emp** and **dept** practice dataset.

---

## Quick Cheat Sheet

```
QUERY         → $eq $ne $gt $gte $lt $lte $in $nin
                $and $or $not $nor
                $exists $type
                $regex $expr $mod $text
                $all $elemMatch $size

UPDATE FIELD  → $set $unset $rename $setOnInsert
                $inc $mul $min $max $currentDate

UPDATE ARRAY  → $push $addToSet $pop $pull $pullAll
                $each $slice $sort $position
                $  $[]  $[<identifier>]

AGG STAGES    → $match $group $project $unwind $lookup
                $sort $limit $skip $count $sample
                $addFields $set $unset $replaceRoot $replaceWith
                $facet $bucket $bucketAuto
                $out $merge $unionWith $graphLookup
                $setWindowFields

ACCUMULATORS  → $sum $avg $first $last $max $min
                $push $addToSet $mergeObjects

EXPRESSIONS   → $add $subtract $multiply $divide $mod $abs
                $ceil $floor $round $pow $sqrt
                $concat $toLower $toUpper $trim $split
                $substrCP $strLenCP $regexMatch $replaceOne $replaceAll
                $cond $ifNull $switch
                $year $month $dayOfMonth $hour $dayOfWeek
                $dateToString $dateFromString $dateAdd $dateSubtract $dateDiff
                $filter $map $reduce $size $arrayElemAt $concatArrays
                $indexOfArray $range $reverseArray $in
                $toString $toInt $toLong $toDouble $toDecimal $toBool $toDate $convert $type
                $mergeObjects $objectToArray $arrayToObject $getField
                $literal $let $cmp
```

---

## 1. Query Operators

_Used inside filter objects in `find()`, `findOne()`, `updateOne()`, `aggregate()` `$match`, etc._

---

### 1.1 Comparison Operators

| Operator | Definition                                                                                                    | Example                                    |
| -------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| `$eq`    | Matches documents where the field **equals** the specified value. Implicit when you write `{ field: value }`. | `{ job: { $eq: "clerk" } }`                |
| `$ne`    | Matches documents where the field does **not equal** the specified value.                                     | `{ contractType: { $ne: "contract" } }`    |
| `$gt`    | Matches documents where the field is **greater than** the specified value.                                    | `{ sal: { $gt: 2000 } }`                   |
| `$gte`   | Matches documents where the field is **greater than or equal to** the specified value.                        | `{ age: { $gte: 35 } }`                    |
| `$lt`    | Matches documents where the field is **less than** the specified value.                                       | `{ sal: { $lt: 1500 } }`                   |
| `$lte`   | Matches documents where the field is **less than or equal to** the specified value.                           | `{ totalHoursWorked: { $lte: 1900 } }`     |
| `$in`    | Matches documents where the field value **exists in** the given array.                                        | `{ deptNo: { $in: [10, 30] } }`            |
| `$nin`   | Matches documents where the field value does **not exist in** the given array.                                | `{ job: { $nin: ["clerk", "salesman"] } }` |

**Dataset examples:**

```js
// Find all managers and analysts
db.emp.find({ job: { $in: ["manager", "analyst"] } });

// Find employees NOT in dept 30
db.emp.find({ deptNo: { $nin: [30] } });

// Find departments with budget >= 180000
db.dept.find({ budget: { $gte: 180000 } });

// Find employees with grade not equal to "B"
db.emp.find({ grade: { $ne: "B" } });
```

---

### 1.2 Logical Operators

| Operator | Definition                                                                                          | Example                                                    |
| -------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| `$and`   | All conditions must be **true**. Use explicitly when multiple conditions target the **same field**. | `{ $and: [{ age: { $gte: 30 } }, { age: { $lte: 45 } }] }` |
| `$or`    | **At least one** condition must be true.                                                            | `{ $or: [{ job: "manager" }, { job: "president" }] }`      |
| `$not`   | **Inverts** the effect of a query expression. Wraps a single condition.                             | `{ sal: { $not: { $lt: 2000 } } }`                         |
| `$nor`   | Matches documents that **fail all** given conditions (opposite of `$or`).                           | `{ $nor: [{ job: "clerk" }, { deptNo: 10 }] }`             |

**Dataset examples:**

```js
// $and — employees in dept 20 with sal > 2000
db.emp.find({ $and: [{ deptNo: 20 }, { sal: { $gt: 2000 } }] });

// $or — managers or analysts
db.emp.find({ $or: [{ job: "manager" }, { job: "analyst" }] });

// $not — employees whose sal is NOT less than 2000
db.emp.find({ sal: { $not: { $lt: 2000 } } });

// $nor — neither a clerk NOR in dept 10
db.emp.find({ $nor: [{ job: "clerk" }, { deptNo: 10 }] });
```

---

### 1.3 Element Operators

| Operator  | Definition                                                                                       | Example                            |
| --------- | ------------------------------------------------------------------------------------------------ | ---------------------------------- |
| `$exists` | Matches documents where the specified field **exists** (`true`) or does **not exist** (`false`). | `{ incentive: { $exists: true } }` |
| `$type`   | Matches documents where the field is of the specified **BSON type**.                             | `{ skills: { $type: "array" } }`   |

**Common BSON type strings:** `"string"`, `"double"` (number), `"array"`, `"bool"`, `"null"`, `"date"`, `"int"`, `"objectId"`

**Dataset examples:**

```js
// Employees who HAVE an "incentive" field
db.emp.find({ incentive: { $exists: true } });

// Employees who do NOT have a "territory" field
db.emp.find({ territory: { $exists: false } });

// Departments where headOfDept is null (type null)
db.dept.find({ headOfDept: { $type: "null" } });

// Employees where skills is an array
db.emp.find({ skills: { $type: "array" } });

// Employees where hireDate is a date type
db.emp.find({ hireDate: { $type: "date" } });
```

---

### 1.4 Evaluation Operators

| Operator | Definition                                                                                                     | Example                                  |
| -------- | -------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| `$regex` | Matches documents where the field value **matches a regular expression**.                                      | `{ empName: { $regex: /^m/i } }`         |
| `$expr`  | Allows **aggregation expressions** inside query filters. Useful for comparing two fields in the same document. | `{ $expr: { $gt: ["$sal", "$bonus"] } }` |
| `$mod`   | Matches documents where the field value **divided by a divisor** has the specified remainder.                  | `{ sal: { $mod: [2, 0] } }`              |
| `$text`  | Performs a **full-text search** on fields with a text index. Requires a text index on the collection.          | `{ $text: { $search: "analyst" } }`      |

**Dataset examples:**

```js
// $regex — employees whose name starts with "m" (case-insensitive)
db.emp.find({ empName: { $regex: /^m/i } });

// $regex — employees whose name contains "ar"
db.emp.find({ empName: { $regex: /ar/i } });

// $regex — departments whose name contains "ing"
db.dept.find({ dName: { $regex: /ing/i } });

// $expr — employees where sal > bonus
db.emp.find({ $expr: { $gt: ["$sal", "$bonus"] } });

// $expr — employees where (sal + bonus + comm) > 4000
db.emp.find({ $expr: { $gt: [{ $add: ["$sal", "$bonus", "$comm"] }, 4000] } });

// $expr — departments where budget > employeeCount * 30000
db.dept.find({
  $expr: { $gt: ["$budget", { $multiply: ["$employeeCount", 30000] }] },
});

// $mod — employees with even salary
db.emp.find({ sal: { $mod: [2, 0] } });

// $mod — employees whose empNo is divisible by 3
db.emp.find({ empNo: { $mod: [3, 0] } });

// $mod — departments whose dept number is divisible by 10
db.dept.find({ dept: { $mod: [10, 0] } });
```

---

### 1.5 Array Query Operators

| Operator     | Definition                                                                                                          | Example                                                   |
| ------------ | ------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| `$all`       | Matches documents where an array field **contains ALL** the specified elements (order doesn't matter).              | `{ skills: { $all: ["sql", "python"] } }`                 |
| `$elemMatch` | Matches documents where **at least one element** in an array satisfies **all** specified conditions simultaneously. | `{ weeklyHours: { $elemMatch: { $gte: 45, $lte: 55 } } }` |
| `$size`      | Matches documents where an array field has **exactly** the specified number of elements.                            | `{ skills: { $size: 3 } }`                                |

> **Note:** `$size` only supports exact matches. For greater/less than comparisons on array length, use `$expr` with `$size`.

**Dataset examples:**

```js
// $all — employees with both "sql" and "python" in skills
db.emp.find({ skills: { $all: ["sql", "python"] } });

// $all — employees working on both "project_alpha" and "research_initiative"
db.emp.find({ projects: { $all: ["project_alpha", "research_initiative"] } });

// $all — departments with both "conference_room" and "printer"
db.dept.find({ facilities: { $all: ["conference_room", "printer"] } });

// $elemMatch — employees where at least one weeklyHour is between 45 and 55
db.emp.find({ weeklyHours: { $elemMatch: { $gte: 45, $lte: 55 } } });

// $elemMatch — departments where a monthlyExpense is > 19000
db.dept.find({ monthlyExpenses: { $elemMatch: { $gt: 19000 } } });

// $size — employees with exactly 3 skills
db.emp.find({ skills: { $size: 3 } });

// $size — employees with exactly 0 certifications
db.emp.find({ certifications: { $size: 0 } });

// $expr + $size — employees with MORE than 3 skills (greater-than comparison)
db.emp.find({ $expr: { $gt: [{ $size: "$skills" }, 3] } });
```

---

## 2. Update Operators

_Used in the update document of `updateOne()`, `updateMany()`, `findOneAndUpdate()`, etc._

---

### 2.1 Field Update Operators

| Operator       | Definition                                                                                              | Example                                       |
| -------------- | ------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| `$set`         | **Sets** the value of a field. Creates the field if it doesn't exist.                                   | `{ $set: { grade: "A+" } }`                   |
| `$unset`       | **Removes** the specified field from the document entirely.                                             | `{ $unset: { incentive: "" } }`               |
| `$rename`      | **Renames** a field. Useful for schema migrations.                                                      | `{ $rename: { "sal": "salary" } }`            |
| `$setOnInsert` | Sets field values **only when a document is inserted** (with `upsert: true`). Has no effect on updates. | `{ $setOnInsert: { createdAt: new Date() } }` |

**Dataset examples:**

```js
// $set — update smith's salary and grade
db.emp.updateOne({ empName: "smith" }, { $set: { sal: 1500, grade: "B+" } });

// $set — update nested field (performance rating)
db.emp.updateOne({ empName: "smith" }, { $set: { "performance.rating": 4.0 } });

// $unset — remove incentive field from ward
db.emp.updateOne({ empName: "ward" }, { $unset: { incentive: "" } });

// $unset — remove territory from ALL salesmen
db.emp.updateMany({ job: "salesman" }, { $unset: { territory: "" } });

// $rename — rename sal to salary for all employees
db.emp.updateMany({}, { $rename: { sal: "salary" } });

// $rename — rename loc to location in all departments
db.dept.updateMany({}, { $rename: { loc: "location" } });

// $setOnInsert — upsert: if empNo 9999 not found, insert with createdAt
db.emp.updateOne(
  { empNo: 9999 },
  {
    $set: { empName: "newbie", job: "intern" },
    $setOnInsert: { createdAt: new Date(), deptNo: 10 },
  },
  { upsert: true },
);
```

---

### 2.2 Arithmetic Update Operators

| Operator       | Definition                                                                                     | Example                                      |
| -------------- | ---------------------------------------------------------------------------------------------- | -------------------------------------------- |
| `$inc`         | **Increments** a field by the specified amount. Use a negative value to decrement.             | `{ $inc: { sal: 500, overtimeHours: -10 } }` |
| `$mul`         | **Multiplies** the value of a field by the specified number.                                   | `{ $mul: { sal: 1.1 } }`                     |
| `$min`         | Updates the field **only if** the specified value is **less than** the current field value.    | `{ $min: { leaveBalance: 5 } }`              |
| `$max`         | Updates the field **only if** the specified value is **greater than** the current field value. | `{ $max: { totalHoursWorked: 2000 } }`       |
| `$currentDate` | Sets the field value to the **current date**, either as a `Date` or `Timestamp`.               | `{ $currentDate: { lastIncrement: true } }`  |

**Dataset examples:**

```js
// $inc — increase adams' salary by 500
db.emp.updateOne({ empName: "adams" }, { $inc: { sal: 500 } });

// $inc — increase bonus of all clerks by 300
db.emp.updateMany({ job: "clerk" }, { $inc: { bonus: 300 } });

// $inc — decrease overtimeHours of james by 10
db.emp.updateOne({ empName: "james" }, { $inc: { overtimeHours: -10 } });

// $mul — 10% salary raise for king
db.emp.updateOne({ empName: "king" }, { $mul: { sal: 1.1 } });

// $mul — 5% budget increase for all active departments
db.dept.updateMany({ isActive: true }, { $mul: { budget: 1.05 } });

// $min — set james' sal to 1000 only if 1000 < current sal
db.emp.updateOne({ empName: "james" }, { $min: { sal: 1000 } });

// $max — ensure adams has at least 2000 totalHoursWorked
db.emp.updateOne({ empName: "adams" }, { $max: { totalHoursWorked: 2000 } });

// $currentDate — set lastIncrement to today for all clerks
db.emp.updateMany({ job: "clerk" }, { $currentDate: { lastIncrement: true } });
```

---

### 2.3 Array Update Operators

#### $push — append elements

| Usage                           | Definition                                                                   | Example                                                 |
| ------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------- |
| `$push`                         | **Appends** a single value to an array.                                      | `{ $push: { skills: "mongodb" } }`                      |
| `$push` + `$each`               | Appends **multiple** values to an array at once.                             | `{ $push: { skills: { $each: ["a","b"] } } }`           |
| `$push` + `$each` + `$position` | Appends elements at a **specific index** instead of the end.                 | `{ $push: { skills: { $each: ["x"], $position: 0 } } }` |
| `$push` + `$each` + `$slice`    | Appends and **trims** the array to a specified size. Negative = keep last N. | `{ $push: { skills: { $each: ["x"], $slice: -5 } } }`   |
| `$push` + `$each` + `$sort`     | Appends and **sorts** the resulting array. `1` = asc, `-1` = desc.           | `{ $push: { skills: { $each: ["x"], $sort: 1 } } }`     |

```js
// $push — add single skill to smith
db.emp.updateOne({ empName: "smith" }, { $push: { skills: "mongodb" } });

// $push + $each — add multiple skills to turner
db.emp.updateOne(
  { empName: "turner" },
  { $push: { skills: { $each: ["angular", "vue", "svelte"] } } },
);

// $push + $each + $position — add "leadership" at start of skills for jones
db.emp.updateOne(
  { empName: "jones" },
  { $push: { skills: { $each: ["leadership"], $position: 0 } } },
);

// $push + $each + $slice — add skill and keep only last 5
db.emp.updateOne(
  { empName: "ward" },
  { $push: { skills: { $each: ["graphql"], $slice: -5 } } },
);

// $push + $each + $sort — add skill and sort array alphabetically
db.emp.updateOne(
  { empName: "allen" },
  { $push: { skills: { $each: ["animation"], $sort: 1 } } },
);

// All modifiers combined — add "team_lead" at position 0, keep last 6, sort ascending
db.emp.updateOne(
  { empName: "jones" },
  {
    $push: {
      skills: { $each: ["team_lead"], $position: 0, $slice: -6, $sort: 1 },
    },
  },
);
```

---

#### $addToSet — add unique elements only

| Usage                 | Definition                                                                 | Example                                           |
| --------------------- | -------------------------------------------------------------------------- | ------------------------------------------------- |
| `$addToSet`           | Adds a value **only if it doesn't already exist** — guarantees uniqueness. | `{ $addToSet: { skills: "python" } }`             |
| `$addToSet` + `$each` | Adds **multiple** unique values to an array.                               | `{ $addToSet: { skills: { $each: ["a","b"] } } }` |

```js
// $addToSet — add "python" to clark only if not already present
db.emp.updateOne({ empName: "clark" }, { $addToSet: { skills: "python" } });

// $addToSet + $each — add multiple unique skills to ward
db.emp.updateOne(
  { empName: "ward" },
  { $addToSet: { skills: { $each: ["react", "angular", "vue"] } } },
);

// $addToSet — add skill "html" to ALL dept 30 employees, no duplicates
db.emp.updateMany({ deptNo: 30 }, { $addToSet: { skills: "html" } });
```

---

#### $pop, $pull, $pullAll — remove elements

| Operator   | Definition                                                            | Example                                     |
| ---------- | --------------------------------------------------------------------- | ------------------------------------------- |
| `$pop`     | Removes the **first** (`-1`) or **last** (`1`) element from an array. | `{ $pop: { skills: 1 } }`                   |
| `$pull`    | Removes **all elements** matching a specified value or condition.     | `{ $pull: { skills: "html" } }`             |
| `$pullAll` | Removes **all occurrences** of each listed value.                     | `{ $pullAll: { skills: ["sql","excel"] } }` |

```js
// $pop — remove last skill from turner
db.emp.updateOne({ empName: "turner" }, { $pop: { skills: 1 } });

// $pop — remove first project from blake
db.emp.updateOne({ empName: "blake" }, { $pop: { projects: -1 } });

// $pull — remove "html" from jones' skills
db.emp.updateOne({ empName: "jones" }, { $pull: { skills: "html" } });

// $pull with condition — remove weeklyHours values < 40 from smith
db.emp.updateOne({ empName: "smith" }, { $pull: { weeklyHours: { $lt: 40 } } });

// $pull — remove projects ending in "_q1" from all salesmen
db.emp.updateMany(
  { job: "salesman" },
  { $pull: { projects: { $regex: /_q1$/ } } },
);

// $pullAll — remove multiple skills from james
db.emp.updateOne(
  { empName: "james" },
  { $pullAll: { skills: ["sql", "excel"] } },
);

// $pullAll — remove specific facilities from dept 40
db.dept.updateOne(
  { dept: 40 },
  { $pullAll: { facilities: ["storage", "loading_dock"] } },
);
```

---

### 2.4 Positional Operators

| Operator                                | Definition                                                                                   | Example                                                                              |
| --------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `$` (positional)                        | Updates the **first matching** array element. The query must also reference the array field. | `{ $set: { "skills.$": "python3" } }`                                                |
| `$[]` (all positional)                  | Updates **all elements** in an array field.                                                  | `{ $inc: { "weeklyHours.$[]": 2 } }`                                                 |
| `$[<identifier>]` (filtered positional) | Updates only array elements that **match the `arrayFilters` condition**.                     | `{ $set: { "weeklyHours.$[hr]": 45 } }` with `arrayFilters: [{ "hr": { $gt: 45 } }]` |

```js
// $ — update first "python" to "python3" in clark's skills
db.emp.updateOne(
  { empName: "clark", skills: "python" },
  { $set: { "skills.$": "python3" } },
);

// $ — update first "conference_room" in dept 20 facilities
db.dept.updateOne(
  { dept: 20, facilities: "conference_room" },
  { $set: { "facilities.$": "conference_room_A" } },
);

// $[] — add 2 to every weeklyHour for james
db.emp.updateOne({ empName: "james" }, { $inc: { "weeklyHours.$[]": 2 } });

// $[] — add 100 to every monthlySalaryHistory entry for smith
db.emp.updateOne(
  { empName: "smith" },
  { $inc: { "monthlySalaryHistory.$[]": 100 } },
);

// $[identifier] — cap all weeklyHours > 45 to 45 for jones
db.emp.updateOne(
  { empName: "jones" },
  { $set: { "weeklyHours.$[hr]": 45 } },
  { arrayFilters: [{ hr: { $gt: 45 } }] },
);

// $[identifier] — append "_priority" to all projects containing "sales" for blake
db.emp.updateOne(
  { empName: "blake" },
  { $set: { "projects.$[proj]": { $concat: ["$proj", "_priority"] } } },
  { arrayFilters: [{ proj: { $regex: /sales/ } }] },
);
// Note: use aggregation pipeline update syntax for $concat in $set

// $[identifier] — set all monthlySalaryHistory < 1300 to 1300 for smith
db.emp.updateOne(
  { empName: "smith" },
  { $set: { "monthlySalaryHistory.$[m]": 1300 } },
  { arrayFilters: [{ m: { $lt: 1300 } }] },
);

// $[identifier] — update across ALL employees (weeklyHours < 40 → set to 40)
db.emp.updateMany(
  {},
  { $set: { "weeklyHours.$[hr]": 40 } },
  { arrayFilters: [{ hr: { $lt: 40 } }] },
);
```

---

## 3. Aggregation Operators

_Used inside `aggregate()` pipelines._

---

### 3.1 Pipeline Stages

| Stage                           | Definition                                                                                     | Example                                                                                                               |
| ------------------------------- | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `$match`                        | **Filters** documents — equivalent to `find()`. Place early in the pipeline for performance.   | `{ $match: { job: "manager" } }`                                                                                      |
| `$group`                        | **Groups** documents by a key and computes accumulations.                                      | `{ $group: { _id: "$deptNo", total: { $sum: "$sal" } } }`                                                             |
| `$project`                      | **Reshapes** documents — include, exclude, rename, or compute new fields.                      | `{ $project: { empName: 1, annualSal: { $multiply: ["$sal", 12] } } }`                                                |
| `$sort`                         | **Sorts** documents. `1` = ascending, `-1` = descending.                                       | `{ $sort: { sal: -1 } }`                                                                                              |
| `$limit`                        | **Restricts** the number of documents to the next stage.                                       | `{ $limit: 5 }`                                                                                                       |
| `$skip`                         | **Skips** the first N documents. Use with `$limit` for pagination.                             | `{ $skip: 3 }`                                                                                                        |
| `$count`                        | **Counts** total documents; outputs a single document with the count.                          | `{ $count: "totalEmployees" }`                                                                                        |
| `$unwind`                       | **Deconstructs** an array field — outputs one document per array element.                      | `{ $unwind: "$skills" }`                                                                                              |
| `$lookup`                       | **Left outer join** to another collection.                                                     | `{ $lookup: { from: "dept", localField: "deptNo", foreignField: "dept", as: "deptInfo" } }`                           |
| `$addFields` / `$set`           | **Adds** new fields or overwrites existing ones without removing others. (`$set` is an alias.) | `{ $addFields: { annualSal: { $multiply: ["$sal", 12] } } }`                                                          |
| `$unset`                        | **Removes** specified fields from documents.                                                   | `{ $unset: ["_id", "mgr"] }`                                                                                          |
| `$replaceRoot` / `$replaceWith` | **Replaces** the root document with a specified embedded document or expression.               | `{ $replaceRoot: { newRoot: "$performance" } }`                                                                       |
| `$sample`                       | **Randomly selects** N documents.                                                              | `{ $sample: { size: 5 } }`                                                                                            |
| `$facet`                        | Runs **multiple sub-pipelines** on the same input in a single stage.                           | `{ $facet: { topEarners: [...], avgBySept: [...] } }`                                                                 |
| `$bucket`                       | Categorizes into **user-defined ranges**.                                                      | `{ $bucket: { groupBy: "$sal", boundaries: [0,1500,3000,6000] } }`                                                    |
| `$bucketAuto`                   | Like `$bucket` but **auto-determines** bucket boundaries.                                      | `{ $bucketAuto: { groupBy: "$age", buckets: 4 } }`                                                                    |
| `$out`                          | Writes pipeline results to a **new/existing collection** (replaces entirely).                  | `{ $out: "high_performers" }`                                                                                         |
| `$merge`                        | Writes results into a collection — can **merge/upsert** instead of replace.                    | `{ $merge: { into: "dept", on: "dept", whenMatched: "merge" } }`                                                      |
| `$unionWith`                    | **Combines** pipeline results with documents from another collection.                          | `{ $unionWith: { coll: "archived_emp" } }`                                                                            |
| `$graphLookup`                  | **Recursive lookup** — useful for org charts and hierarchical data.                            | `{ $graphLookup: { from: "emp", startWith: "$mgr", connectFromField: "mgr", connectToField: "empNo", as: "chain" } }` |
| `$setWindowFields`              | **Window functions** — rank, running totals, moving averages (MongoDB 5.0+).                   | `{ $setWindowFields: { partitionBy: "$deptNo", sortBy: { sal: -1 }, output: { rank: { $rank: {} } } } }`              |

**Dataset examples:**

```js
// $match + $group — total salary expense per department
db.emp.aggregate([
  { $match: { isRemote: false } },
  { $group: { _id: "$deptNo", totalSal: { $sum: "$sal" } } },
]);

// $sort + $limit — top 5 highest-paid employees
db.emp.aggregate([{ $sort: { sal: -1 } }, { $limit: 5 }]);

// $skip + $limit — pagination: records 4–7 by salary
db.emp.aggregate([{ $sort: { sal: -1 } }, { $skip: 3 }, { $limit: 4 }]);

// $unwind — one document per skill per employee
db.emp.aggregate([{ $unwind: "$skills" }]);

// $lookup — join employees with their department
db.emp.aggregate([
  {
    $lookup: {
      from: "dept",
      localField: "deptNo",
      foreignField: "dept",
      as: "deptInfo",
    },
  },
]);

// $graphLookup — trace management chain upward from smith
db.emp.aggregate([
  { $match: { empName: "smith" } },
  {
    $graphLookup: {
      from: "emp",
      startWith: "$mgr",
      connectFromField: "mgr",
      connectToField: "empNo",
      as: "managementChain",
    },
  },
]);

// $facet — top 5 earners AND avg salary by dept in one query
db.emp.aggregate([
  {
    $facet: {
      topEarners: [{ $sort: { sal: -1 } }, { $limit: 5 }],
      avgByDept: [{ $group: { _id: "$deptNo", avg: { $avg: "$sal" } } }],
    },
  },
]);

// $bucket — group employees by salary ranges
db.emp.aggregate([
  {
    $bucket: {
      groupBy: "$sal",
      boundaries: [0, 1500, 3000, 6000],
      default: "Other",
      output: { count: { $sum: 1 }, names: { $push: "$empName" } },
    },
  },
]);

// $bucketAuto — auto 3 buckets by salary
db.emp.aggregate([{ $bucketAuto: { groupBy: "$sal", buckets: 3 } }]);

// $out — save high performers to a new collection
db.emp.aggregate([
  { $match: { "performance.rating": { $gt: 4.5 } } },
  { $out: "high_performers" },
]);

// $merge — add avgSalary field to dept documents
db.emp.aggregate([
  { $group: { _id: "$deptNo", avgSalary: { $avg: "$sal" } } },
  { $merge: { into: "dept", on: "_id", whenMatched: "merge" } },
]);

// $setWindowFields — rank employees by sal within their department
db.emp.aggregate([
  {
    $setWindowFields: {
      partitionBy: "$deptNo",
      sortBy: { sal: -1 },
      output: { salRank: { $rank: {} } },
    },
  },
]);
```

---

### 3.2 Accumulator Operators

_Used inside `$group`. Some also work in `$project` and `$addFields`._

| Operator        | Definition                                                                           | Example                                         |
| --------------- | ------------------------------------------------------------------------------------ | ----------------------------------------------- |
| `$sum`          | Returns the **sum** of numeric values. Use `1` to count documents.                   | `{ count: { $sum: 1 } }`                        |
| `$avg`          | Returns the **average** of numeric values.                                           | `{ avgSal: { $avg: "$sal" } }`                  |
| `$first`        | Returns the value from the **first** document in the group (respects prior `$sort`). | `{ firstEmp: { $first: "$empName" } }`          |
| `$last`         | Returns the value from the **last** document in the group.                           | `{ lastHired: { $last: "$empName" } }`          |
| `$max`          | Returns the **maximum** value in the group.                                          | `{ maxSal: { $max: "$sal" } }`                  |
| `$min`          | Returns the **minimum** value in the group.                                          | `{ minSal: { $min: "$sal" } }`                  |
| `$push`         | Returns an array of **all values** in the group (including duplicates).              | `{ names: { $push: "$empName" } }`              |
| `$addToSet`     | Returns an array of **unique values** in the group.                                  | `{ uniqueJobs: { $addToSet: "$job" } }`         |
| `$mergeObjects` | **Merges** multiple documents into one.                                              | `{ merged: { $mergeObjects: "$performance" } }` |

**Dataset examples:**

```js
// Count employees per department
db.emp.aggregate([{ $group: { _id: "$deptNo", count: { $sum: 1 } } }]);

// Average salary per job role
db.emp.aggregate([
  {
    $group: {
      _id: "$job",
      avgSal: { $avg: "$sal" },
      maxSal: { $max: "$sal" },
      minSal: { $min: "$sal" },
    },
  },
]);

// Collect all employee names per department
db.emp.aggregate([
  { $sort: { empName: 1 } },
  {
    $group: {
      _id: "$deptNo",
      employees: { $push: "$empName" },
      firstAlpha: { $first: "$empName" },
    },
  },
]);

// Unique cities per job role
db.emp.aggregate([{ $group: { _id: "$job", cities: { $addToSet: "$city" } } }]);
```

---

### 3.3 Expression Operators

#### Arithmetic

| Operator    | Definition                                          | Example                                      |
| ----------- | --------------------------------------------------- | -------------------------------------------- |
| `$add`      | **Adds** numbers or adds milliseconds to a date.    | `{ $add: ["$sal", "$bonus"] }`               |
| `$subtract` | **Subtracts** the second from the first.            | `{ $subtract: ["$sal", "$comm"] }`           |
| `$multiply` | **Multiplies** two or more numbers.                 | `{ $multiply: ["$sal", 12] }`                |
| `$divide`   | **Divides** the first number by the second.         | `{ $divide: ["$sal", "$totalHoursWorked"] }` |
| `$mod`      | Returns the **remainder** of division.              | `{ $mod: ["$sal", 2] }`                      |
| `$abs`      | Returns the **absolute value**.                     | `{ $abs: { $subtract: ["$a", "$b"] } }`      |
| `$ceil`     | Rounds **up** to the nearest integer.               | `{ $ceil: "$hourlyRate" }`                   |
| `$floor`    | Rounds **down** to the nearest integer.             | `{ $floor: "$hourlyRate" }`                  |
| `$round`    | **Rounds** to a specified number of decimal places. | `{ $round: ["$hourlyRate", 2] }`             |
| `$pow`      | Raises a number to an **exponent**.                 | `{ $pow: ["$rating", 2] }`                   |
| `$sqrt`     | Returns the **square root**.                        | `{ $sqrt: "$area" }`                         |

```js
// Annual salary and total compensation
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      annualSal: { $multiply: ["$sal", 12] },
      totalComp: { $add: ["$sal", "$bonus", "$comm"] },
      hourlyRate: { $round: [{ $divide: ["$sal", "$totalHoursWorked"] }, 2] },
    },
  },
]);

// Achievement rate (taskCompleted / targetAssigned * 100)
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      achievementRate: {
        $multiply: [{ $divide: ["$taskCompleted", "$targetAssigned"] }, 100],
      },
    },
  },
]);
```

---

#### Date Operators

| Operator          | Definition                                              | Example                                                                        |
| ----------------- | ------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `$year`           | Extracts the **year**.                                  | `{ $year: "$hireDate" }`                                                       |
| `$month`          | Extracts the **month** (1–12).                          | `{ $month: "$hireDate" }`                                                      |
| `$dayOfMonth`     | Extracts the **day** (1–31).                            | `{ $dayOfMonth: "$hireDate" }`                                                 |
| `$hour`           | Extracts the **hour** (0–23).                           | `{ $hour: "$hireDate" }`                                                       |
| `$minute`         | Extracts the **minute** (0–59).                         | `{ $minute: "$hireDate" }`                                                     |
| `$second`         | Extracts the **second** (0–59).                         | `{ $second: "$hireDate" }`                                                     |
| `$dayOfWeek`      | Extracts the **day of week** (1=Sunday … 7=Saturday).   | `{ $dayOfWeek: "$hireDate" }`                                                  |
| `$dateToString`   | **Formats** a date as a string.                         | `{ $dateToString: { format: "%d-%m-%Y", date: "$hireDate" } }`                 |
| `$dateFromString` | **Parses** a date string into a Date object.            | `{ $dateFromString: { dateString: "$dateStr" } }`                              |
| `$dateAdd`        | **Adds** time (in a unit) to a date.                    | `{ $dateAdd: { startDate: "$lastIncrement", unit: "day", amount: 365 } }`      |
| `$dateSubtract`   | **Subtracts** time from a date.                         | `{ $dateSubtract: { startDate: "$lastAuditDate", unit: "month", amount: 6 } }` |
| `$dateDiff`       | Returns the **difference** between two dates in a unit. | `{ $dateDiff: { startDate: "$hireDate", endDate: "$$NOW", unit: "year" } }`    |

```js
// Hire year, month, and years of service
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      hireYear: { $year: "$hireDate" },
      hireMonth: { $month: "$hireDate" },
      hireDay: { $dayOfMonth: "$hireDate" },
      yearsOfService: {
        $dateDiff: { startDate: "$hireDate", endDate: "$$NOW", unit: "year" },
      },
      hireDateFormatted: {
        $dateToString: { format: "%d-%m-%Y", date: "$hireDate" },
      },
    },
  },
]);

// Find employees hired in April (month = 4)
db.emp.aggregate([
  { $match: { $expr: { $eq: [{ $month: "$hireDate" }, 4] } } },
  { $project: { empName: 1, hireDate: 1 } },
]);

// Count employees hired per year
db.emp.aggregate([
  { $group: { _id: { $year: "$hireDate" }, count: { $sum: 1 } } },
  { $sort: { _id: 1 } },
]);
```

---

#### String Operators

| Operator      | Definition                                                  | Example                                                                  |
| ------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------ |
| `$concat`     | **Concatenates** two or more strings.                       | `{ $concat: ["$empName", " - ", "$job"] }`                               |
| `$toLower`    | Converts to **lowercase**.                                  | `{ $toLower: "$empName" }`                                               |
| `$toUpper`    | Converts to **uppercase**.                                  | `{ $toUpper: "$dName" }`                                                 |
| `$trim`       | Removes **whitespace** (or specified chars) from both ends. | `{ $trim: { input: "$empName" } }`                                       |
| `$split`      | **Splits** a string into an array by a delimiter.           | `{ $split: ["$city", " "] }`                                             |
| `$substrCP`   | Returns a **substring** (Unicode-safe).                     | `{ $substrCP: ["$empName", 0, 3] }`                                      |
| `$strLenCP`   | Returns the **character count** of a string.                | `{ $strLenCP: "$empName" }`                                              |
| `$regexMatch` | Returns `true` if string **matches** a regex pattern.       | `{ $regexMatch: { input: "$empName", regex: /^[a-m]/ } }`                |
| `$replaceOne` | Replaces the **first** occurrence of a substring.           | `{ $replaceOne: { input: "$job", find: "man", replacement: "person" } }` |
| `$replaceAll` | Replaces **all** occurrences of a substring.                | `{ $replaceAll: { input: "$proj", find: "_", replacement: "-" } }`       |

```js
// Full title, name length, and name starts with a-m check
db.emp.aggregate([
  {
    $project: {
      fullTitle: { $concat: ["$empName", " - ", "$job"] },
      nameLength: { $strLenCP: "$empName" },
      first3Letters: { $substrCP: ["$empName", 0, 3] },
      startsAtoM: { $regexMatch: { input: "$empName", regex: /^[a-m]/ } },
      deptDisplay: {
        $concat: ["Dept ", { $toString: "$deptNo" }, " - Grade: ", "$grade"],
      },
    },
  },
]);

// Department names in uppercase, split city by space
db.dept.aggregate([
  {
    $project: {
      dNameUpper: { $toUpper: "$dName" },
      cityParts: { $split: ["$loc", " "] },
      deptLabel: { $concat: [{ $toUpper: "$dName" }, " - ", "$loc"] },
    },
  },
]);

// Replace underscores in project names with dashes
db.emp.aggregate([
  { $unwind: "$projects" },
  {
    $project: {
      empName: 1,
      projectDisplay: {
        $replaceAll: { input: "$projects", find: "_", replacement: "-" },
      },
    },
  },
]);
```

---

#### Array Expression Operators

| Operator        | Definition                                                                    | Example                                                                                        |
| --------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `$size`         | Returns the **number of elements** in an array.                               | `{ $size: "$skills" }`                                                                         |
| `$arrayElemAt`  | Returns the element at a **specified index**.                                 | `{ $arrayElemAt: ["$skills", 0] }`                                                             |
| `$first`        | Returns the **first element** of an array.                                    | `{ $first: "$monthlySalaryHistory" }`                                                          |
| `$last`         | Returns the **last element** of an array.                                     | `{ $last: "$monthlySalaryHistory" }`                                                           |
| `$slice`        | Returns a **subset** of an array.                                             | `{ $slice: ["$skills", 2] }`                                                                   |
| `$concatArrays` | **Merges** two or more arrays into one.                                       | `{ $concatArrays: ["$skills", "$certifications"] }`                                            |
| `$filter`       | Returns only elements **satisfying a condition**.                             | `{ $filter: { input: "$weeklyHours", as: "h", cond: { $gt: ["$$h", 42] } } }`                  |
| `$map`          | **Applies an expression** to each element; returns new array.                 | `{ $map: { input: "$skills", as: "s", in: { $toUpper: "$$s" } } }`                             |
| `$reduce`       | **Accumulates** a single result from an array.                                | `{ $reduce: { input: "$weeklyHours", initialValue: 0, in: { $add: ["$$value", "$$this"] } } }` |
| `$in`           | Returns `true` if a value **exists in** an array.                             | `{ $in: ["python", "$skills"] }`                                                               |
| `$indexOfArray` | Returns the **index** of the first occurrence of a value (`-1` if not found). | `{ $indexOfArray: ["$skills", "python"] }`                                                     |
| `$range`        | **Generates** an array of numbers from start to end.                          | `{ $range: [0, 5] }`                                                                           |
| `$reverseArray` | Returns array elements in **reverse order**.                                  | `{ $reverseArray: "$weeklyHours" }`                                                            |

```js
// skillCount, first skill, last project, combined arrays
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      skillCount: { $size: "$skills" },
      firstSkill: { $arrayElemAt: ["$skills", 0] },
      thirdWeekHour: { $arrayElemAt: ["$weeklyHours", 2] },
      lastProject: { $last: "$projects" },
      lastSalary: { $last: "$monthlySalaryHistory" },
      fullProfile: { $concatArrays: ["$skills", "$certifications"] },
    },
  },
]);

// $filter — keep only weeklyHours > 42
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      highHours: {
        $filter: {
          input: "$weeklyHours",
          as: "h",
          cond: { $gt: ["$$h", 42] },
        },
      },
      pythonSkills: {
        $filter: {
          input: "$skills",
          as: "s",
          cond: { $regexMatch: { input: "$$s", regex: /python/ } },
        },
      },
    },
  },
]);

// $map — skills to uppercase, double each weeklyHour
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      skillsUpper: {
        $map: { input: "$skills", as: "s", in: { $toUpper: "$$s" } },
      },
      doubledHours: {
        $map: { input: "$weeklyHours", as: "h", in: { $multiply: ["$$h", 2] } },
      },
    },
  },
]);

// $reduce — sum all weeklyHours, find max weeklyHour
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      totalWeeklyHours: {
        $reduce: {
          input: "$weeklyHours",
          initialValue: 0,
          in: { $add: ["$$value", "$$this"] },
        },
      },
      maxWeeklyHour: {
        $reduce: {
          input: "$weeklyHours",
          initialValue: 0,
          in: { $cond: [{ $gt: ["$$this", "$$value"] }, "$$this", "$$value"] },
        },
      },
    },
  },
]);
```

---

#### Conditional Operators

| Operator  | Definition                                                                                   | Example                                                                      |
| --------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `$cond`   | **Ternary** — if condition is true return first value, else second.                          | `{ $cond: { if: { $gt: ["$sal", 2500] }, then: "High", else: "Standard" } }` |
| `$ifNull` | Returns the first expression if **not null/missing**, otherwise the fallback.                | `{ $ifNull: ["$comm", 0] }`                                                  |
| `$switch` | Evaluates a series of **cases** and returns the first matching `then`. Supports a `default`. | See example below                                                            |

```js
// $cond — salary tier and bonus eligibility
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      salaryTier: {
        $cond: {
          if: { $gt: ["$sal", 2500] },
          then: "High Earner",
          else: "Standard",
        },
      },
      bonusEligible: {
        $cond: {
          if: { $gte: ["$performance.rating", 4.0] },
          then: true,
          else: false,
        },
      },
      remoteFlag: { $cond: ["$isRemote", "Remote", "On-site"] },
    },
  },
]);

// $switch — seniority level classification
db.emp.aggregate([
  {
    $addFields: {
      seniorityLevel: {
        $switch: {
          branches: [
            { case: { $lt: ["$experience", 2] }, then: "Fresher" },
            { case: { $lt: ["$experience", 7] }, then: "Junior" },
            { case: { $lt: ["$experience", 15] }, then: "Mid" },
          ],
          default: "Expert",
        },
      },
      performanceLabel: {
        $switch: {
          branches: [
            { case: { $gt: ["$performance.rating", 4.5] }, then: "Excellent" },
            { case: { $gte: ["$performance.rating", 3.5] }, then: "Good" },
          ],
          default: "Needs Improvement",
        },
      },
    },
  },
]);

// $ifNull — handle null/missing fields
db.dept.aggregate([
  {
    $project: {
      dName: 1,
      headDisplay: { $ifNull: ["$headOfDept", "No Head Assigned"] },
      certDisplay: { $ifNull: ["$certification", "Not Certified"] },
    },
  },
]);

db.emp.aggregate([
  {
    $project: {
      empName: 1,
      commDisplay: { $ifNull: ["$comm", 0] },
      incentiveDisplay: { $ifNull: ["$incentive", "No Incentive"] },
    },
  },
]);
```

---

#### Comparison Expression Operators

_Return `true`/`false` or `-1`/`0`/`1` for use inside expressions — not for query filters._

| Operator | Definition                                          | Example                                  |
| -------- | --------------------------------------------------- | ---------------------------------------- |
| `$eq`    | Returns `true` if two values are **equal**.         | `{ $eq: ["$job", "manager"] }`           |
| `$ne`    | Returns `true` if two values are **not equal**.     | `{ $ne: ["$contractType", "contract"] }` |
| `$gt`    | Returns `true` if first > second.                   | `{ $gt: ["$sal", "$bonus"] }`            |
| `$gte`   | Returns `true` if first >= second.                  | `{ $gte: ["$performance.rating", 4.5] }` |
| `$lt`    | Returns `true` if first < second.                   | `{ $lt: ["$age", 30] }`                  |
| `$lte`   | Returns `true` if first <= second.                  | `{ $lte: ["$leaveBalance", 5] }`         |
| `$cmp`   | Returns `-1` (less), `0` (equal), or `1` (greater). | `{ $cmp: ["$sal", "$bonus"] }`           |

```js
// Use comparison expressions inside $project / $addFields
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      isManager: { $eq: ["$job", "manager"] },
      isPermanent: { $eq: ["$contractType", "permanent"] },
      earnsMoreThanBonus: { $gt: ["$sal", "$bonus"] },
      isHighPerformer: { $gte: ["$performance.rating", 4.5] },
      isYoung: { $lt: ["$age", 30] },
      salVsBonusCmp: { $cmp: ["$sal", "$bonus"] }, // -1, 0, or 1
    },
  },
]);
```

---

#### Type Conversion Operators

| Operator      | Definition                                                                        | Example                                                             |
| ------------- | --------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| `$toString`   | Converts to a **string**.                                                         | `{ $toString: "$deptNo" }`                                          |
| `$toInt`      | Converts to a **32-bit integer**.                                                 | `{ $toInt: "$sal" }`                                                |
| `$toLong`     | Converts to a **64-bit integer**.                                                 | `{ $toLong: "$empNo" }`                                             |
| `$toDouble`   | Converts to a **double** (floating-point).                                        | `{ $toDouble: "$rating" }`                                          |
| `$toDecimal`  | Converts to a **128-bit decimal** (high-precision). Preferred for financial data. | `{ $toDecimal: "$sal" }`                                            |
| `$toBool`     | Converts to a **boolean**.                                                        | `{ $toBool: "$isActive" }`                                          |
| `$toDate`     | Converts a string, number, or ObjectId to a **Date**.                             | `{ $toDate: "$timestamp" }`                                         |
| `$toObjectId` | Converts a string to an **ObjectId**.                                             | `{ $toObjectId: "$idStr" }`                                         |
| `$convert`    | **General-purpose** conversion with `onError` and `onNull` fallbacks.             | `{ $convert: { input: "$val", to: "int", onError: 0, onNull: 0 } }` |
| `$type`       | Returns the **BSON type name** of a field's value as a string.                    | `{ $type: "$sal" }`                                                 |

```js
// Type conversions for display and safety
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      deptNoStr: { $toString: "$deptNo" },
      salDecimal: { $toDecimal: "$sal" },
      isActiveFlag: { $toBool: "$havingInsurance" },
      salType: { $type: "$sal" },
      // Safe conversion with fallback
      safeComm: {
        $convert: { input: "$comm", to: "int", onError: 0, onNull: 0 },
      },
    },
  },
]);

// Concatenate string with number (requires $toString)
db.emp.aggregate([
  {
    $project: {
      display: {
        $concat: ["Emp #", { $toString: "$empNo" }, " - ", "$empName"],
      },
    },
  },
]);
```

---

#### Object Operators

| Operator         | Definition                                                                       | Example                                               |
| ---------------- | -------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `$mergeObjects`  | **Merges** multiple objects into one. Later keys overwrite earlier ones.         | `{ $mergeObjects: ["$defaults", "$overrides"] }`      |
| `$objectToArray` | Converts a document to an **array of `{k, v}` pairs**. Useful for dynamic keys.  | `{ $objectToArray: "$address" }`                      |
| `$arrayToObject` | Converts an array of **`{k, v}` pairs** back into an object.                     | `{ $arrayToObject: "$pairs" }`                        |
| `$getField`      | Returns the value of a field — for **dynamic or special-character** field names. | `{ $getField: { field: "my.field", input: "$doc" } }` |

```js
// $objectToArray — inspect address sub-document as key-value pairs
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      addressPairs: { $objectToArray: "$address" },
    },
  },
]);

// $mergeObjects — merge employee + department info after $lookup
db.emp.aggregate([
  {
    $lookup: {
      from: "dept",
      localField: "deptNo",
      foreignField: "dept",
      as: "dept",
    },
  },
  {
    $replaceWith: { $mergeObjects: ["$$ROOT", { $arrayElemAt: ["$dept", 0] }] },
  },
]);
```

---

#### Variable & Literal Operators

| Operator   | Definition                                                                                   | Example                                                                                          |
| ---------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `$literal` | Returns a value **exactly as written** — prevents MongoDB from treating it as an expression. | `{ $literal: 0 }`                                                                                |
| `$let`     | Defines **local variables** for use within a sub-expression.                                 | `{ $let: { vars: { total: { $add: ["$sal", "$bonus"] } }, in: { $multiply: ["$$total", 2] } } }` |

```js
// $let — compute a variable and reuse it
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      doubleComp: {
        $let: {
          vars: { comp: { $add: ["$sal", "$bonus", "$comm"] } },
          in: { $multiply: ["$$comp", 2] },
        },
      },
    },
  },
]);

// $literal — project a literal value (e.g., a tag or flag)
db.emp.aggregate([
  {
    $project: {
      empName: 1,
      source: { $literal: "emp_collection" },
      defaultScore: { $literal: 0 },
    },
  },
]);
```

---

## 4. Projection Operators

_Used in the projection argument of `find()` and `findOne()`._

| Operator         | Definition                                                                                            | Example                                                |
| ---------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| `$` (positional) | Projects only the **first matching** array element. The array field must be in the query.             | `db.emp.find({ skills: "python" }, { "skills.$": 1 })` |
| `$elemMatch`     | Projects only the first element satisfying the given condition (can differ from query).               | `{ weeklyHours: { $elemMatch: { $gt: 45 } } }`         |
| `$slice`         | Limits the number of elements returned in an array. `N` from start, `-N` from end, `[skip, N]` range. | `{ skills: { $slice: 3 } }`                            |
| `$meta`          | Projects metadata about the document, most commonly `"textScore"` for full-text search ranking.       | `{ score: { $meta: "textScore" } }`                    |

```js
// $ positional — show only the matched skill
db.emp.find({ skills: "python" }, { empName: 1, "skills.$": 1 });

// $elemMatch — show first weeklyHour > 45
db.emp.find(
  { empName: "jones" },
  { empName: 1, weeklyHours: { $elemMatch: { $gt: 45 } } },
);

// $slice — show first 2 skills only
db.emp.find({}, { empName: 1, skills: { $slice: 2 } });

// $slice — show last 3 skills only
db.emp.find({}, { empName: 1, skills: { $slice: -3 } });

// $slice — skip 1, take next 2
db.emp.find({}, { empName: 1, projects: { $slice: [1, 2] } });
```

---

## 5. Text Search Operators

_Requires a text index on the collection._

| Operator / Option     | Definition                                                                              | Example                             |
| --------------------- | --------------------------------------------------------------------------------------- | ----------------------------------- |
| `$text`               | Performs a **full-text search** on text-indexed fields.                                 | `{ $text: { $search: "analyst" } }` |
| `$search`             | The string to search. Multiple words = OR by default; wrap in `"..."` for exact phrase. | `{ $search: "\"project alpha\"" }`  |
| `$language`           | Specifies the language for stemming and stop words.                                     | `{ $language: "english" }`          |
| `$caseSensitive`      | Enables case-sensitive matching (`false` by default).                                   | `{ $caseSensitive: false }`         |
| `$diacriticSensitive` | Treats `é` and `e` differently when `true` (`false` by default).                        | `{ $diacriticSensitive: false }`    |
| `$meta: "textScore"`  | Projects relevance score. Use with `$sort` to rank results.                             | `{ score: { $meta: "textScore" } }` |

```js
// Create a text index first
db.emp.createIndex({ empName: "text", job: "text" });

// Basic text search
db.emp.find({ $text: { $search: "analyst" } });

// Exact phrase search
db.emp.find({ $text: { $search: '"data science"' } });

// Text search with relevance score, sorted by score
db.emp
  .find(
    { $text: { $search: "manager analyst" } },
    { score: { $meta: "textScore" } },
  )
  .sort({ score: { $meta: "textScore" } });
```

---

## 6. Geospatial Operators

_Requires a geospatial index (`2dsphere` for GeoJSON, `2d` for legacy coordinates)._

| Operator         | Definition                                                                                         | Example                                                                                                     |
| ---------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `$geoWithin`     | Matches documents with coordinates **inside** a given shape.                                       | `{ location: { $geoWithin: { $centerSphere: [[-73.9, 40.7], 10/3963.2] } } }`                               |
| `$geoIntersects` | Matches documents whose geometry **intersects** with the specified GeoJSON object.                 | `{ area: { $geoIntersects: { $geometry: { type: "Point", coordinates: [-73.9, 40.7] } } } }`                |
| `$near`          | Returns documents **sorted by proximity** to a point. Requires a `2dsphere` index.                 | `{ location: { $near: { $geometry: { type: "Point", coordinates: [-73.9, 40.7] }, $maxDistance: 1000 } } }` |
| `$geoNear`       | Aggregation stage — like `$near` but outputs **distance as a field**. Must be the **first** stage. | See example below                                                                                           |

```js
// Add coordinates to departments first (example)
db.dept.updateOne(
  { dept: 10 },
  { $set: { location: { type: "Point", coordinates: [-74.006, 40.7128] } } },
);
db.dept.createIndex({ location: "2dsphere" });

// $geoNear — departments near a point, with distance field
db.dept.aggregate([
  {
    $geoNear: {
      near: { type: "Point", coordinates: [-74.006, 40.7128] },
      distanceField: "distanceFromHQ",
      maxDistance: 500000,
      spherical: true,
    },
  },
]);
```

---

_End of MongoDB Operators Reference_
