# Database Management Systems
## Distributed Databases
**Transparency** (User perspective)
- Data organization transparency
  - Location transparency: We don't care where the data comes from.
  - Naming transparency: Tables from multiple places could have different names, but users refer to them by only one logical name.
- Replication transparency: Users don't care about which copy of data they are talking to.
- Fragmentation transparency
  - Horizontal: break a relation into subsets of tuples.
  - Vertical: break a relation into subsets of columns. (must store the primary key with each fragment for reconstruction)

**Autonomy**: Can notes operate independently? Design / Communication / Execution autonomy.

Why do we want to distribute the data? For better **fault tolerance**.
**Reliability**: how long does it take something to fail on average. (metric of time)
**availability**: probability that the system is continuously available over a period of time (metric of probability).
Flexibility, improved performance, easier expansion.

Replication: more reliable, more available, more cost for keeping consistency.

**Schema Architecture**
[image](./picture/Schema-Arch.jpg)

**How the query works**
[image](picture/Component-Arch.jpg)
- Mapping
- Localization
- Global Optimization (Analyse data transfer costs)
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
- **CAP** (Consistency, Availablity, Partition tolerance) Theorem.
  - Partition tolerance: Will the network continue to function if it is split into pieces?
  - We can only have two of them in our database (trade-off). Partition tolerance should always be one of them.
  - If we want better consistency, we need to use locking transactions, meaning less availability.
  - If we want better availability, we allow everyone to read what they want (temporary consistency).

## MongoDB
https://www.mongodb.com/docs/manual/tutorial/getting-started/
- Collections (tables): groups of Documents. (No "explicit" schema) 
  - Sub-collections.
  - Documents (rows): key-value pairs.
    - Indexing: automatically created on ObjectID (primary key), or we can  create manually.
    - Design: embedded structure or referencing (foreign key)? It's all about how we want to access the data.
    - Objects: each object has a unique ObjectID (12 bytes).
      - ObjectID: TimeStamp(4B) + Machine(3B) + PID(2B) + Increment (3B, break ties).