# MongoDB Cheat Sheet

## Basic Commands

### Connect to MongoDB
Different ways to connect using Mongoshell:

```bash
# Connect using URI
mongosh "URI"

# Connect with host and port
mongosh --host mongodb0.example.com --port 28015

# Connect with authentication
mongosh "mongodb://mongodb0.example.com:28015" --username alice --authenticationDatabase admin
```

### Database Operations

```bash
# Show databases
show dbs

# Switch database
use <database_name>

# Create a collection
db.createCollection("<collection_name>")

# Show collections in the current database
show collections
```

## Document Operations

### Insert Documents

```javascript
// Insert a single document
db.<collection_name>.insert({ field1: value1, field2: value2, ... })

// Insert multiple documents
db.<collection_name>.insertMany([document1, document2, ...])
```

### Find Documents

```javascript
// Find all documents
db.<collection_name>.find()

// Filter documents with a query
db.<collection_name>.find({ field: value })

// Equality query
db.<collection_name>.find({ field: "value" })
```

## Querying

### Range Queries

```javascript
// Less than
db.<collection_name>.find({ field: { $lt: value } })

// Greater than
db.<collection_name>.find({ field: { $gt: value } })

// Between (less than AND greater than)
db.<collection_name>.find({ field: { $lt: value, $gt: value } })
```

### Logical Operators

```javascript
// AND query
db.<collection_name>.find({ field1: value1, field2: value2 })

// OR query
db.<collection_name>.find({ $or: [ { field1: value1 }, { field2: value2 } ] })
```

### Sorting

```javascript
// Sort ascending
db.<collection_name>.find().sort({ field: 1 })

// Sort descending
db.<collection_name>.find().sort({ field: -1 })
```

## Update and Delete Operations

### Update Documents

```javascript
// Update one document
db.<collection_name>.updateOne({ field: value }, { $set: { new_field: new_value } })

// Update multiple documents
db.<collection_name>.updateMany({ field: value }, { $set: { new_field: new_value } })
```

### Delete Documents

```javascript
// Delete one document
db.<collection_name>.deleteOne({ field: value })

// Delete multiple documents
db.<collection_name>.deleteMany({ field: value })
```

## Aggregation

### Aggregation Pipeline

```javascript
db.<collection_name>.aggregate([
    { $match: { field: value } },
    { $group: { _id: "$field", total: { $sum: 1 } } }
])
```

## Indexing

```javascript
// Create a single field index
db.<collection_name>.createIndex({ field: 1 })

// Create a compound index
db.<collection_name>.createIndex({ field: 1, another_field: 1 })

// List all indexes
db.<collection_name>.getIndexes()
```

## Data Export and Import

### Export Data

```bash
# Export data to JSON
mongoexport --db <database_name> --collection <collection_name> --out <output_file.json>
```

### Import Data

```bash
# Import data from JSON
mongoimport --db <database_name> --collection <collection_name> --file <input_file.json>
```