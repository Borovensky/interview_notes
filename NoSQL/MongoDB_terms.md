| Term | Definition |
|------|------------|
| Aggregation pipeline | The aggregation pipeline in MongoDB allows for data transformation and processing using a series of stages, including filtering, grouping, sorting, and projecting. The aggregation pipeline is a powerful tool for expressive data manipulation. |
| B+ Tree | The B+ Tree is a data structure commonly used in database indexing to efficiently store and retrieve data based on ordered keys. |
| Code-first | Code-first refers to a development approach where developers create the application code first and let the code define the database schema. In MongoDB, this means that the schema is flexible and adapts to evolving application needs. |
| Collection | In MongoDB, a collection is a group of MongoDB documents. Collections are analogous to tables in a relational database and store related data documents in a schema-free, JSON-like format. |
| CRUD | CRUD is an acronym for create, read, update, and delete, which are the basic operations for interacting with and manipulating data in a database. |
| Database | A database in MongoDB is a logical container for one or more collections. It provides an isolation mechanism for collections and their associated data. |
| Document | A NoSQL database model that stores data in semi-structured documents, often in formats like JSON or BSON. These documents can vary in structure and are typically grouped within collections. |
| Election | In a MongoDB replica set, an election is the process of selecting a new primary node when the current primary becomes unavailable. |
| Extract, transform, and load (ETL) | A process of extracting data from its sources, transforming the data into a business-usable format, and then loading the data into a target database, often used with MongoDB for data integration. |
| Expressive querying | Expressive querying refers to the ability to write complex and flexible queries that address data retrieval and manipulation needs, often facilitated by MongoDB's query language and aggregation framework. |
| High availability (HA) | High availability (HA) in MongoDB refers to the ability of the database system to maintain near-continuous operation and data accessibility, even in the face of hardware failures or other issues. High availability is often achieved through features like replication and failover. |
| Horizontal scaling | The process of adding more machines or nodes to a NoSQL database to improve its performance and capacity. This is typically achieved through techniques like sharding. |
| Idempotent changes | Idempotent operations are those that can be safely repeated multiple times without changing the result. MongoDB encourages idempotent operations to ensure data consistency. |
| Indexing | The creation of data structures that improve query performance by allowing the database to quickly locate specific records based on certain fields or columns. |
| JSON | JSON is an acronym for JavaScript Object Notation, a lightweight data-interchange format used in NoSQL databases and other data systems. JSON is human-readable and easy for machines to parse. |
| Mongo shell | The MongoDB shell, known as mongo shell, is an interactive command-line interface that allows users to interact with a MongoDB server using JavaScript-like commands. The mongo shell is a versatile tool for administration and data manipulation. |
| MongoClient | MongoClient is the official MongoDB driver that provides a connection to a MongoDB server and allows developers to interact with the database in various programming languages. |
| MQL | MongoDB Query Language is a query language specific to MongoDB used to retrieve and manipulate data in the database. |
| NoSQL | NoSQL stands for "not only SQL." A type of database that provides storage and retrieval of data that is modeled in ways other than the traditional relational tabular databases. |
| Operational data | Operational data in MongoDB refers to the data that the application actively uses and manipulates, as opposed to historical or archived data. |
| Oplog | The Oplog is a special collection that records all write operations in a primary node. It is used to replicate data to secondary nodes and recover from failures. |
| Primary node | In a MongoDB replica set, the primary node is the active, writable node that processes all write operations. |
| Replication | Replication involves creating and maintaining copies of data on multiple nodes to ensure data availability, reduce data loss, fault tolerance (improve system resilience), and provide read scalability. |
| Replication lag | Replication lag refers to the delay in data replication from a primary node to its secondary nodes in a replica set. Replication lag can impact the consistency of secondary data. |
| Secondary | Secondary nodes replicate data from the primary and can be used for read-operations. |
| Sharding | Refers to the practice of partitioning a database into smaller, more manageable pieces called shards to distribute data across multiple servers. Sharding helps with horizontal scaling. |
| Unstructured data | Unstructured data in MongoDB is data that does not adhere to a fixed schema. MongoDB allows for flexible and unstructured data storage, making MongoDB suitable for semi-structured or rapidly changing data. |
| Vertical scaling | Vertical scaling involves upgrading the resources (For example, CPU and RAM) of existing machines to improve performance. |