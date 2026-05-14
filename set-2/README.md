# Set 2

| S.No. | Question                                                                |
| ----- | ----------------------------------------------------------------------- |
| 1.    | [Does MongoDB enforce schema?](#question-1-does-mongodb-enforce-schema) |

## Question 1. Does MongoDB enforce schema?

MongoDB is **schema-flexible by default**, meaning it does **not strictly enforce a fixed schema** like traditional relational databases. Documents in the same collection can have different fields and structures.

However, modern MongoDB **does support schema validation** using **JSON Schema validators**, allowing you to enforce rules when needed.

### 1. Default Behavior: Flexible Schema

In MongoDB, documents inside a collection can vary.

Example:

```javascript
use interview_db;

db.employees.insertMany([
  {
    name: "Alice",
    department: "Engineering",
    salary: 75000
  },
  {
    name: "Bob",
    department: "Engineering",
    skills: ["MongoDB", "Node.js"]
  }
]);
```

Both documents are valid even though:

- Alice has `salary`
- Bob has `skills`
- Structures differ

This flexibility is useful for:

- Rapid development
- Evolving applications
- Semi-structured data
- Microservices
- IoT/event-driven systems

### 2. How MongoDB Can Enforce Schema

MongoDB provides **schema validation** using:

- `$jsonSchema`
- Validation rules
- Validation levels/actions

You can enforce:

- Required fields
- Data types
- Value ranges
- Array structures
- String patterns

### 3. Example: Schema Validation

Create a collection with validation:

```javascript
db.createCollection("employees", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "department", "salary"],
      properties: {
        name: {
          bsonType: "string",
          description: "Name must be a string",
        },
        department: {
          bsonType: "string",
        },
        salary: {
          bsonType: "number",
          minimum: 30000,
        },
        skills: {
          bsonType: "array",
          items: {
            bsonType: "string",
          },
        },
      },
    },
  },
  validationAction: "error",
});
```

### 4. Line-by-Line Explanation

#### `validator`

Defines validation rules for documents.

#### `$jsonSchema`

Uses JSON Schema standard for validation.

#### `bsonType: "object"`

Ensures inserted data must be a document object.

#### `required`

Mandatory fields:

- `name`
- `department`
- `salary`

#### `properties`

Defines field-level rules.

#### `minimum: 30000`

Salary cannot be below 30,000.

#### `validationAction: "error"`

Rejects invalid documents.

Alternative:

- `"warn"` → allows insert but logs warning

### 5. Valid vs Invalid Inserts

#### Valid

```javascript
db.employees.insertOne({
  name: "Charlie",
  department: "HR",
  salary: 50000,
  skills: ["Recruitment"],
});
```

#### Invalid

```javascript
db.employees.insertOne({
  name: "David",
  salary: "50000",
});
```

Why it fails:

- Missing `department`
- `salary` should be a number, not string

### 6. Performance Considerations

Schema validation adds **small write overhead** because MongoDB validates documents during inserts/updates.

Best practices:

- Keep validators simple
- Validate critical fields only
- Use application-level validation alongside DB validation
- Avoid overly complex nested schemas

### 7. Real-World Usage

Companies commonly use:

- Flexible schema during prototyping
- Strict validation in production

Typical pattern:

- MongoDB validation
- Plus application-layer validation (Node.js, Java, Python)

This gives both:

- Flexibility
- Data integrity

### 8. Important Interview Point

MongoDB is often called **“schema-less”**, but technically it is better described as:

> “Schema-flexible with optional schema enforcement.”

### Follow-Up Interview Questions

#### 1. What is the difference between `validationAction: "warn"` and `"error"`?

- `"error"` rejects invalid documents
- `"warn"` allows them but logs warnings

#### 2. Can schema validation be modified later?

Yes.

Use:

```javascript
db.runCommand({
  collMod: "employees",
  validator: { ... }
});
```

to update validation rules on an existing collection.
