# Aggregations_in_MongoDB

### Summary Table: MongoDB Operators vs. JavaScript Methods

| MongoDB Operator     | JavaScript Equivalent                | Description                                      |
|----------------------|--------------------------------------|--------------------------------------------------|
| `$lookup`            | `Array.map() + find()`              | Joining two arrays/collections                   |
| `$unwind`            | `Array.flatMap()`                   | Flattening arrays of arrays                      |
| `$addFields`         | `Array.map()`                       | Adding/modifying fields in each document         |
| `$map`               | `Array.map()`                       | Mapping over array elements for transformation   |
| `$mergeObjects`      | `Object.assign()` / `...` (spread)  | Merging multiple objects                         |
| `$arrayElemAt`       | `Array[index]`                      | Accessing a specific index of an array           |

---

### 1. `$lookup` vs. `Array.map() + find()`

#### MongoDB Example
```javascript
// MongoDB Input Collections
db.orders.insertMany([
  { orderId: 1, customerId: 101 },
  { orderId: 2, customerId: 102 }
]);

db.customers.insertMany([
  { _id: 101, name: "John Doe" },
  { _id: 102, name: "Jane Smith" }
]);

// MongoDB Aggregation Query
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customerInfo"
    }
  }
]);

// MongoDB Output
/*
[
  { orderId: 1, customerId: 101, customerInfo: [{ _id: 101, name: "John Doe" }] },
  { orderId: 2, customerId: 102, customerInfo: [{ _id: 102, name: "Jane Smith" }] }
]
*/
```

#### JavaScript Example
```javascript
// JavaScript Input
const orders = [
  { orderId: 1, customerId: 101 },
  { orderId: 2, customerId: 102 }
];

const customers = [
  { _id: 101, name: "John Doe" },
  { _id: 102, name: "Jane Smith" }
];

// JavaScript Transformation
const result = orders.map(order => ({
  ...order,
  customerInfo: customers.find(customer => customer._id === order.customerId) || null
}));

// JavaScript Output
console.log(result);
/*
[
  { orderId: 1, customerId: 101, customerInfo: { _id: 101, name: "John Doe" } },
  { orderId: 2, customerId: 102, customerInfo: { _id: 102, name: "Jane Smith" } }
]
*/
```

---

### 2. `$unwind` vs. `Array.flatMap()`

#### MongoDB Example
```javascript
// MongoDB Input Collection
db.inventory.insertMany([
  { item: "book", tags: ["fiction", "bestseller"] },
  { item: "pen", tags: ["stationery"] }
]);

// MongoDB Aggregation Query
db.inventory.aggregate([
  { $unwind: "$tags" }
]);

// MongoDB Output
/*
[
  { item: "book", tags: "fiction" },
  { item: "book", tags: "bestseller" },
  { item: "pen", tags: "stationery" }
]
*/
```

#### JavaScript Example
```javascript
// JavaScript Input
const inventory = [
  { item: "book", tags: ["fiction", "bestseller"] },
  { item: "pen", tags: ["stationery"] }
];

// JavaScript Transformation
const result = inventory.flatMap(doc => doc.tags.map(tag => ({ item: doc.item, tag })));

// JavaScript Output
console.log(result);
/*
[
  { item: "book", tag: "fiction" },
  { item: "book", tag: "bestseller" },
  { item: "pen", tag: "stationery" }
]
*/
```

---

### 3. `$addFields` vs. `Array.map()`

#### MongoDB Example
```javascript
// MongoDB Input Collection
db.students.insertMany([
  { firstName: "Alice", lastName: "Johnson" },
  { firstName: "Bob", lastName: "Smith" }
]);

// MongoDB Aggregation Query
db.students.aggregate([
  { $addFields: { fullName: { $concat: ["$firstName", " ", "$lastName"] } } }
]);

// MongoDB Output
/*
[
  { firstName: "Alice", lastName: "Johnson", fullName: "Alice Johnson" },
  { firstName: "Bob", lastName: "Smith", fullName: "Bob Smith" }
]
*/
```

#### JavaScript Example
```javascript
// JavaScript Input
const students = [
  { firstName: "Alice", lastName: "Johnson" },
  { firstName: "Bob", lastName: "Smith" }
];

// JavaScript Transformation
const result = students.map(student => ({
  ...student,
  fullName: `${student.firstName} ${student.lastName}`
}));

// JavaScript Output
console.log(result);
/*
[
  { firstName: "Alice", lastName: "Johnson", fullName: "Alice Johnson" },
  { firstName: "Bob", lastName: "Smith", fullName: "Bob Smith" }
]
*/
```

---

### 4. `$map` vs. `Array.map()`

#### MongoDB Example
```javascript
// MongoDB Input Collection
db.students.insertMany([
  { name: "Alice", scores: [80, 85, 90] },
  { name: "Bob", scores: [75, 88, 92] }
]);

// MongoDB Aggregation Query
db.students.aggregate([
  {
    $project: {
      testScores: {
        $map: {
          input: "$scores",
          as: "score",
          in: { $multiply: ["$$score", 1.1] }
        }
      }
    }
  }
]);

// MongoDB Output
/*
[
  { name: "Alice", testScores: [88, 93.5, 99] },
  { name: "Bob", testScores: [82.5, 96.8, 101.2] }
]
*/
```

#### JavaScript Example
```javascript
// JavaScript Input
const students = [
  { name: "Alice", scores: [80, 85, 90] },
  { name: "Bob", scores: [75, 88, 92] }
];

// JavaScript Transformation
const result = students.map(student => ({
  ...student,
  testScores: student.scores.map(score => score * 1.1)
}));

// JavaScript Output
console.log(result);
/*
[
  { name: "Alice", testScores: [88, 93.5, 99] },
  { name: "Bob", testScores: [82.5, 96.8, 101.2] }
]
*/
```

---

### 5. `$mergeObjects` vs. `Object.assign()` / Spread Operator

#### MongoDB Example
```javascript
// MongoDB Input Collection
db.employees.insertMany([
  {
    contactInfo: { phone: "123-456", email: "alice@example.com" },
    jobInfo: { title: "Developer", department: "Engineering" }
  }
]);

// MongoDB Aggregation Query
db.employees.aggregate([
  { $project: { fullInfo: { $mergeObjects: ["$contactInfo", "$jobInfo"] } } }
]);

// MongoDB Output
/*
[
  { fullInfo: { phone: "123-456", email: "alice@example.com", title: "Developer", department: "Engineering" } }
]
*/
```

#### JavaScript Example
```javascript
// JavaScript Input
const employees = [
  {
    contactInfo: { phone: "123-456", email: "alice@example.com" },
    jobInfo: { title: "Developer", department: "Engineering" }
  }
];

// JavaScript Transformation
const result = employees.map(employee => ({
  fullInfo: { ...employee.contactInfo, ...employee.jobInfo }
}));

// JavaScript Output
console.log(result);
/*
[
  { fullInfo: { phone: "123-456", email: "alice@example.com", title: "Developer", department: "Engineering" } }
]
*/
```

---

### 6. `$arrayElemAt` vs. `Array[index]`

#### MongoDB Example
```javascript
// MongoDB Input Collection
db.students.insertMany([
  { name: "Alice", scores: [95, 85, 78] },
  { name: "Bob", scores: [80, 88, 92] }
]);

// MongoDB Aggregation Query
db.students.aggregate([
  { $project: { firstScore: { $arrayElemAt: ["$scores", 0] } } }
]);

// MongoDB Output
/*
[
  { name: "Alice", firstScore: 95 },
  { name: "Bob", firstScore: 80 }
]
*/
```

#### JavaScript Example
```javascript
// JavaScript Input
const students = [
  { name: "Alice", scores: [95, 85, 78] },
  { name: "Bob", scores: [80, 88, 92]

 }
];

// JavaScript Transformation
const result = students.map(student => ({
  name: student.name,
  firstScore: student.scores[0]
}));

// JavaScript Output
console.log(result);
/*
[
  { name: "Alice", firstScore: 95 },
  { name: "Bob", firstScore: 80 }
]
*/
```

--- 

This side-by-side guide should help in understanding how to achieve similar outcomes between MongoDB and JavaScript.
