# Introduction to Amazon DynamoDB — NoSQL Database Basics

## 📌 Overview

This hands-on AWS lab demonstrates the fundamentals of **Amazon DynamoDB**, a fully managed NoSQL database service.

In this project, I created a DynamoDB table, configured a **partition key** and **sort key**, inserted multiple items, and queried the table to retrieve specific records.

The lab provided practical experience with DynamoDB's key-value and document data model and basic database operations through the AWS Management Console.

---

## 🎯 Objectives

By completing this lab, I learned how to:

* Create an Amazon DynamoDB table
* Configure a partition key
* Configure a sort key
* Understand how DynamoDB identifies items
* Insert items into a DynamoDB table
* Query DynamoDB data
* Filter data using key attributes
* Explore DynamoDB items using the AWS Management Console
* Understand basic NoSQL database concepts

---

## ☁️ AWS Service Used

| AWS Service     | Purpose                                       |
| --------------- | --------------------------------------------- |
| Amazon DynamoDB | NoSQL database for storing and querying items |

---

## 🌎 AWS Region

```text
US East (N. Virginia)
us-east-1
```

---

# 🏗️ DynamoDB Data Model

The table uses two key attributes:

```text
Partition Key: companyid
Type: Number

Sort Key: name
Type: String
```

Together, the partition key and sort key form the table's composite primary key.

```text
                 DynamoDB Table
                       │
              ┌────────┴────────┐
              │                 │
         Partition Key       Sort Key
          companyid             name
             │                   │
             ▼                   ▼
          Number               String
```

---

# 1. Create DynamoDB Table

I created a DynamoDB table named:

```text
mydynamodbtable
```

### Table Configuration

```text
Table Name: mydynamodbtable

Partition Key:
    companyid
    Number

Sort Key:
    name
    String
```

The combination of `companyid` and `name` uniquely identifies an item within the table.

---

# 2. Insert Items

I added multiple items to the DynamoDB table.

### Sample Data

| companyid | name  |
| --------: | ----- |
|         1 | John  |
|         2 | Kyle  |
|         3 | Will  |
|         4 | Sarah |
|         5 | Kane  |

Example DynamoDB item:

```json
{
    "companyid": 1,
    "name": "John"
}
```

Additional items were added to test DynamoDB's key structure and query functionality.

---

# 3. Explore Table Items

After creating the table, I opened:

```text
mydynamodbtable
        ↓
Explore table items
```

The table view allowed me to inspect the stored records and verify that the inserted items were successfully saved.

---

# 4. Query DynamoDB Data

I used the DynamoDB query functionality to search for specific records.

### Example

```text
Partition Key:
companyid = 1
```

The query returned the item associated with:

```text
companyid = 1
name = John
```

### Query Concept

```text
Query
  │
  ▼
Partition Key
companyid = 1
  │
  ▼
Matching Items
```

Because `companyid` is the partition key, DynamoDB uses it to determine which partition contains the relevant data.

---

# 🔑 Partition Key vs. Sort Key

## Partition Key

The **partition key** determines how DynamoDB distributes data across partitions.

In this lab:

```text
companyid
```

was used as the partition key.

Example:

```text
companyid = 1
```

---

## Sort Key

The **sort key** is used together with the partition key to organize and uniquely identify items within the same partition key value.

In this lab:

```text
name
```

was used as the sort key.

The composite primary key is:

```text
companyid + name
```


# 🧠 Key Concepts Learned

### NoSQL Database

DynamoDB is a managed NoSQL database designed for applications that require scalable and low-latency data access.

### Item

An item is a single record stored in a DynamoDB table.

Example:

```json
{
    "companyid": 1,
    "name": "John"
}
```

### Attribute

An attribute is a data element within an item.

For this project:

```text
companyid
name
```

are attributes.

### Partition Key

The partition key determines the logical location of an item in DynamoDB.

### Sort Key

The sort key works with the partition key to create a composite primary key and allows items with the same partition key value to be organized by the sort key.

### Query

A Query operation retrieves items based on their primary key attributes.

---

# 📊 Example Table

```text
┌───────────┬────────┐
│ companyid │ name   │
├───────────┼────────┤
│     1     │ John   │
│     2     │ Kyle   │
│     3     │ Will   │
│     4     │ Sarah  │
│     5     │ Kane   │
└───────────┴────────┘
```

Example query:

```text
companyid = 1
```

Result:

```text
companyid = 1
name = John
```

---

# 🔐 Security Considerations

For production applications:

* Follow the principle of least privilege with IAM
* Grant applications only the DynamoDB permissions they require
* Avoid using overly broad IAM policies
* Enable appropriate encryption and backup settings
* Monitor database activity when required
* Use appropriate capacity and access patterns for the workload

---

# 📸 Screenshots

The following screenshots document the implementation.

### 1. DynamoDB Table Created ⭐

Show:

```text
mydynamodbtable
```

and the table status.

---

### 2. Table Key Configuration ⭐⭐⭐

Show the table's key schema:

```text
Partition key: companyid
Type: Number

Sort key: name
Type: String
```

This is an important screenshot because it demonstrates the DynamoDB data model used in the project.

---

### 3. Explore Table Items ⭐⭐⭐

Show the inserted records:

```text
1 → John
2 → Kyle
3 → Will
4 → Sarah
5 → Kane
```

---

### 4. Query Result ⭐⭐⭐⭐⭐

Show the query:

```text
companyid = 1
```

and the returned item:

```text
companyid = 1
name = John
```

This is the most important validation screenshot because it demonstrates that the data can be queried using the table's key structure.

---

### 5. Final Table Overview

If possible, capture a clean view showing:

* Table name
* Key schema
* Item count or items
* Region



# 📚 AWS Services

* Amazon DynamoDB
* AWS IAM
