# Database Management Systems
## Transactions
- Concurrent operations may cause problems.
  - Lost Update Problem: T2 updates the data between the time when T1 reads the data and updates it so that T1's update is lost.
  - Temporary Update Problem: T2 reads T1's update, but T1 is reverted after that.
  - Incorrect Summary Problem: T1 is applying aggregation while T2 is updating these data.
  - Unrepeatable Read Problem: When the same value is read twice but modified in between.
- A transaction is a logical grouping of operations.
- ACID 
  - Atomicity: run every operation or nothing (rollback).
  - Consistency: database must start and end in consistent states (it may be in an inconsistent state when a transaction is not committed).
  - Isolation: every transaction is unaware of any others.
  - Durability: If a transaction has been committed, it must be permanent. (use log files for rollback and recovery)
- Conflicts: Read/Write and Write/Write conflicts.
- Serializable Schedule: non-serial schedule but equivalent to a serial schedule (equivalent: have the same conflict).
  - Serializable Test by the precedence graph
  - Used to maintain the consistency of the database.
- Violation (with different Isolation Levels)
  - Dirty Read (READ UNCOMMITTED): allows transactions to read data from other transactions that have not been committed.
  - Nonrepeatable Read (READ COMMITTED): One transaction attempts to access the same data twice and a second transaction modify the data in between. It causes the transaction 1 read two different values for the same data.
  - Phantom (REPEATABLE READ): One transaction has read a piece of data, and then another transaction comes and deletes it, so the piece of data been read that noew looks like it doesn't exist anymore.
  - SERIALIZABLE Isolatoin Level has none of these violations. 

## Locks
- Binary Lock: too restrictive (read query is non-conflicting)
- Shared(read) lock, Exclusive(write) lock.
- Two Phase Locking: acquisition phase + release phase. Guarantees serializability(sacrifice performance).
  - Basic
  - Conservative: Acquire all locks that I need before starting the transaction. (prevent deadlock)
  - Strict: I'm gonna hold all write locks until the very end of the transaction. (prevent Temporary Update, generate serializable schedule)
- Deadlock: Both transactions are waiting for each other's locks. (detect by a graph)
  - Prevention: wait-die, wound-wait, no-wait, cautious wait.
- Granularity: Fields, Columns, Pages, Tables.
  - Smaller granularity -> better performance for higher Concurrency, but lock management will be more complicated.
  - Granularity Tree. traverse: db->files->pages->rows...
    - Intention locks: indicates that locks exist in my children. (no need to traverse the tree and make sure every row isn't locked before acquiring lock)

## Distributed Databases
**Transparency** (User perspective)
- Data organization transparency
  - Location transparency: We don't care where the data comes from.
  - Naming transparency: Tables from multiple places could have different names, but users refer to them by only one logical name.
  - Replication transparency: Users don't care about which copy of data they are talking to.
- Fragmentation
  - Horizontal: break a relation into subsets of tuples.
  - Vertical: break a relation into subsets of columns. (must store the primary key with each fragment for reconstruction)

**Autonomy**: Can notes operate independently? Design / Communication / Execution autonomy.

Why do we want to distribute the data? For better **fault tolerance**.
**Reliability**: how long does it take something to fail on average. (metric of time)
**Availability**: probability that the system is continuously available over a period of time (metric of probability).
Flexibility, improved performance, easier expansion.

Replication: more reliable, more available, more cost for keeping consistency.

**Schema Architecture**
[image](./picture/Schema-Arch.jpg)

**How the query works**
[image](picture/Component-Arch.jpg)
- Mapping
- Localization
- Global Optimization (such as Analysing data transfer costs, which )
- Local Optimization (index, query optimization, ...)

**Decomposition** (Global transaction -> Local transactions)
- If multiple copies of data (metadata from catalog) exist, which one should we read from? (This is what Global optimizer does. i.e., how close they are? how is network traffic condition?)
- If we need to update the data, update all of them. (trade-off between performance and availability / reliability)
- If one of the local transactions fails, rollback all local transactions.

**Concurrency**
- Problems
  - Failure and recovery of individual site / communication.
  - Distributed commits
  - Distributed deadlock.
- **Distinguished Copy** (to track locks for distributed items)
  - For distributed items, choose one of them as a distributed copy, and all locking requests are sent to it.
  - **Coordinator site** tracks where does this distinguished copy live.
    - Primary Site: One site has all the distinguished copies. (with backup)

- **Voting Method**
  - Lock requests are sent to all sites with the item. Must receive a majority of locks (more than 50%).
    - nobody else can get more locks than mine so that I can update the data from all sites.
    - Multiple locking requests: More network traffic.

**Distributed Catalog**
- Centralized: Accessing the catalog could be the bottleneck. 
- Replicated: Access quickly. Difficult to keep consistency if the catalog is changed frequently.

## NoSQL
### Scalability
- Distribution: Sharding (split up the data by considering what data will commonly be used together).
- Primary-Secondary Replication (more data consistency and data resilience)
  - Reads can be done from master or slaves.
  - All updates are made to the master and be propagated to slaves. (performance bottleneck)
- P2P Replication
  - Data can be read and written from any node.
  - Nodes notify each other if there is any update. 
### Schemaless
- Pros: More flexibility if I decide to change the data format that the DB takes.
- Cons: Under risk of storing data that is not validated.
- Schemaless means that DB hasn't explicitly told me the schema, but now the application is responsible for ensuring that all data are in the proper format (transfer the burden of the schema from DB into the application) known as **implied schema**.

### Consistency
- Pessimistic (Lock)
- Optimistic (Inconsistency window: allowing temporary consistency, similar to uncommitted read)
  -  When the reader finds out that the data is not consistent(how?), read it again.
  -  Sacrifice some read performance to gain some write performance that we lost due to locking.
  -  Replication causes a larger inconsistency window, for it takes more time to update all the replications.
  -  Write consistency: Read-your-write consistency (be able to see my own changes. sticky session: always talks to the same server, regardless of how many replicas there are).

#### Relaxing Consistency
- Sacrifice consistency for better performance.
- **CAP** (Consistency, Availability, Partition tolerance) Theorem.
  - Consistency: Every read receives the most recent write or an error.
  - Availability: Every request receives a (non-error) response, without the guarantee that it contains the most recent write.
  - Partition tolerance: Will the network continue to function if it is split into pieces?
  - We can only have two of them in our database (trade-off). Partition tolerance should always be one of them.
  - If we want better consistency, we need to use locking transactions, meaning less availability.
  - If we want better availability, we allow everyone to read what they want (temporary consistency).

## MongoDB
https://www.mongodb.com/docs/manual/tutorial/getting-started/
- Collections (tables): groups of Documents. (No "explicit" schema) 
  - Sub-collections.
  - Documents (rows): key-value pairs.
    - Indexing: automatically created on ObjectID ("_id" primary key), or we can create manually.
    - Design: embedded structure or referencing (foreign key)? It's all about how we want to access the data.
    - Objects: each object has a unique ObjectID (12 bytes).
      - ObjectID: TimeStamp(4B) + Machine(3B) + PID(2B) + Increment (3B, break ties).
```
// Insert documents (rows) into the collection (table)
db.<collection>.insertOne(<JSON>)  db.<collection>.insertMany(<JSON array>)
// Relational select, projection
// SELECT title FROM movies WHERE key1=value1 AND ...;
db.movies.find( {key1: value1}, {key2: value2}, ..., {title: 1});
// Sort by field.
db.movies.find( { "awards.wins": { $gt: 100 } } ).sort({<field>: 1});  // $gt, $lt, $in, $gte, $eq, $ne, $nin,...
// COUNT()
db.movies.countDocuments();
```
## Redis
- key-value pairs. In-memory database (performance boost, more availability).
- Use cases: storing session information, user profiles, shopping cart data, ...
- Persistence: Snapshots, Append only log files.
- Not to use: Relationship between data, Multioperation transactions, query by data (index needed), Operation by sets.
### Commands

key-value
``` 
SET <key> <value> GET/DEL/EXISTS <key>
```

Hash Set
```redis
HSET student:1 "name" "jushen" "age" 22 "gender" "male"  // 
HSET student:2 "name" "ruiqi" "age" 22 "gender" "female"
HGET student:2 "name"  // "ruiqi"
HGETALL student:2 #   // Print the whole document
HSET student:1 "age" 23  // Update/Create the age field of student:1
HDEL student:1 "gender"  // Delete a field
```
Index
```
FT.CREATE idx:students ... ON HASH PREFIX 1 "student:" ...
FT._LIST  // Return all indices.
FT.SEARCH idx:students "jushen" RETURN 2 name age  // Query based on the index
FT.SEARCH idx:students "@name:jushen" RETURN 2 name age  // Search from the name field. Project 2 columns, show 2 results.
FT.SEARCH idx:students "@age:[0,100] | @name:ruiqi"
```
### Graph DB: Neo4j
- Consistency, Transactions, Availability, Scaling.
- For connected data, Recommendation engines, and routing, dispatch, location-based services.

### Cassandra
- Wide-column store.
- Consistency: uses timestamps. Tunable. Failure detection: Gossip.
- Data Model: Query-driven model (no joins, denormalized). (choice of keys is essential)

### Object Databases
Store data as object. it meets the need of coupling object-oriented programming languages with a database.

### Multi-value DB
- Denormalization by allowing lists.
- More flexible, easier to query the data. High performance and scalability (relational DB includes numerous JOINs).