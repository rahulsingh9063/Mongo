# MongoDB Aggregation Pipeline

---

## What is Aggregation?

Aggregation in MongoDB is like a data processing pipeline.

Imagine you have many documents (records), and you want to:

- filter them
- group them
- count them
- sort them
- calculate totals or averages
- join two collections

Instead of writing many queries, you use one single pipeline where data flows step by step.

**Important:** Whatever operations are performed using aggregate, it does NOT modify the original data.

---

## Pipeline Basics

```javascript
db.collection_name.aggregate([{$match}, {s2}, {s3}, {s4}, ...])
```

**Flow:**

- input to stage1 → collection
- input to stage2 → op of stage1
- input to stage3 → op of stage2

**Rule:** Each stage `{}` can only consist one aggregation operator ($match, $project, $addFields, etc.)

---

## $match

**Purpose:** Filter out the documents based on some conditions

```javascript
db.collection_name.aggregate([
  {
    $match: { conditions },
  },
]);
```

**Your Example:**

```javascript
//! display the details of emp whose age is greater than 30
db.emp.aggregate([
  {
    $match: {
      age: { $gt: 30 },
    },
  }, //? stage-1
]);
```

---

## $project

**Purpose:**

- Hide/display fields (1 to display, 0 to hide)
- Aliasing
- If key is present then it will be displayed otherwise not

```javascript
db.collection_name.aggregate([
  {
    $project: {
      key: 1,
      key2: 0,
      aliasName: "$originalFieldName",
    },
  },
]);
```

**Your Example:**

```javascript
//! display the age and name of emp whose age is greater than 30
db.emp.aggregate([
  {
    $match: {
      age: { $gt: 30 },
    },
  }, //? stage-1
  {
    $project: {
      username: "$empName", //? aliasing
      age: 1,
      _id: 0,
      isAged: { $gt: ["$age", 50] },
    },
  }, //? stage-2
]);

//! show only names job and salary of all the employees
//! find salesman with comm > 500 and display hireDate only.
//! find all remote emp names as "usernames"

db.emp.aggregate([
  {
    $project: {
      comm: 1,
      _id: 0,
      hireDate: 1,
    },
  },
]);
```

---

## $group

**Purpose:** Group documents based on some field and perform aggregate functions

**5 Operators (can ONLY be used inside $group):**

- $sum
- $avg
- $min
- $max
- (count)

```javascript
db.emp.aggregate([
  {
    $group: {
      _id: "$value",
      variableName: { $sum: "$value" }, // total / sum
      variableName1: { $sum: 1 }, // count
      variableName2: { $max: "$value" }, // max
      variableName3: { $min: "$value" }, // min
      variableName4: { $avg: "$value" }, // avg
    },
  },
]);
```

**Your Examples:**

```javascript
// count emp by contractType
// find avg totalHrsWorked by job role
// find max and min salary in each dept

// count the total emp in each dept
db.emp.aggregate([
  {
    $group: {
      _id: "$deptNo",
      empCount: { $sum: 1 },
      maxSal: { $max: "$sal" },
      minSal: { $min: "$sal" },
    },
  },
  {
    $project: {
      deptNo: "$_id",
      _id: 0,
      empCount: 1,
      maxSal: 1,
      minSal: 1,
    },
  },
]);

//? find the count of all emp along with total expenditure of money (sum of salary)
db.emp.aggregate([
  {
    $group: {
      _id: null,
      count: { $sum: 1 },
      totalEXP: { $sum: "$salary" },
    },
  },
]);

// find total sal expense in each dept
db.emp.aggregate([
  {
    $group: {
      _id: "$deptNo",
      totalSal: { $sum: "$salary" },
    },
  },
  {
    $project: {
      deptNo: "$_id",
      _id: 0,
      totalSal: 1,
    },
  },
]);
```

---

## $sort

**Purpose:** Sort documents in ascending or descending order (default is ascending order)

```javascript
db.collection_name.aggregate([
  {
    $sort: {
      fieldName: 1 / -1, // 1 for ascending, -1 for descending
      fieldName2: 1 / -1,
    },
  },
]);
```

---

## $skip

**Purpose:** Skip the first n documents

```javascript
db.collection_name.aggregate([
  {
    $skip: n,
  },
]);
```

---

## $limit

**Purpose:** Restrict the number of documents to n

```javascript
db.collection_name.aggregate([
  {
    $limit: n,
  },
]);
```

---

## $unwind

**Purpose:** Flatten the array field (deconstruct array)

```javascript
db.collection_name.aggregate([
  {
    $unwind: "$arrayFieldName",
  },
]);
```

---

## $lookup

**Purpose:** Perform join operation between two or more collections

```javascript
db.collection_name.aggregate([
  //& local collection
  {
    $lookup: {
      from: "otherCollectionName", //& foreign collection
      localField: "fieldName", // field from input documents
      foreignField: "fieldName", // field from "from" collection
      as: "outputArrayFieldName", // output array field, normally same as localField
    },
  },
]);
```

---

## $addFields

**Purpose:** Add a field while fetching the documents

```javascript
db.emp.aggregate([
  {
    $addFields: {
      fieldName: value,
    },
  },
]);
```

**Your Examples:**

```javascript
//! get the emp names along with annual salary
db.emp.aggregate([
  {
    $addFields: {
      anSal: { $multiply: [12, "$salary"] },
    },
  },
  {
    $project: {
      empName: 1,
      anSal: 1,
      sal: 1,
      _id: 0,
    },
  },
]);

//! add field "isHighPerformer" to true if performance rating is greater than 4.5
db.emp.aggregate([
  {
    $match: {
      "performance.rating": { $gt: 4.5 },
    },
  },
  {
    $addFields: {
      isHighPerformer: true,
    },
  },
  {
    $project: {
      "performance.rating": 1,
      isHighPerformer: 1,
      _id: 0,
    },
  },
]);
```

---

## Arithmetic Operators

**$add:**

```javascript
{ $add: [num1, num2, .....] }
```

**$subtract:**

```javascript
{
  $subtract: [num1, num2];
} // num1 - num2
```

**$divide:**

```javascript
{
  $divide: [num1, num2];
} // num1/num2
```

**$mod:**

```javascript
{
  $mod: [num1, num2];
} // num1%num2
```

**$multiply:**

```javascript
{
  $multiply: [num1, num2];
} // num1*num2
```

---

## Date Operators

**$ifNull:**

```javascript
{
  $ifNull: ["$field", defaultValue];
}
// Assign a default value if the field is not present
```

**$year:**

```javascript
{
  $year: "$hireDate";
}
// Extract year from date input
```

**$month:**

```javascript
{
  $month: "$hireDate";
}
// Extract month from date input
// for filtering month values ranges from 1 (jan) to 12 (dec)
```

**$dayOfMonth:**

```javascript
{
  $dayOfMonth: "$hireDate";
}
// Extract day from date input
// for filtering day values ranges from 1 to 31
// for particular days 1(Sunday) to 7(Saturday)
```

---

## $bucket

**Purpose:** Group documents based on ranges

```javascript
db.collection_name.aggregate([
    {
        $bucket: {
            groupBy: "$fieldName",
            boundaries: [lowerLimit, upperLimit, upperLimit, ...],
            default: "default value",
            output: {
                count: { $sum: 1 },
                names: { $push: "$name" },
                age: { $push: "$age" }
            }
        }
    }
]);
```

**Parameters:**

- **groupBy:** The expression to group by
- **boundaries:** An array of the lower boundaries for each bucket
- **default:** The bucket name for documents that do not fall within specified boundaries
- **output:** Optional. The output object may contain a single or numerous field names used to accumulate values per bucket

---

## $bucketAuto

```javascript
db.collection_name.aggregate([
  {
    $bucketAuto: {
      groupBy: "",
      buckets: "",
      output: {},
    },
  },
]);
```

**Parameters:**

- **groupBy:** The expression to group by
- **buckets:** The desired number of buckets
- **output:** Optional. The output object may contain a single or numerous field names used to accumulate values per bucket
