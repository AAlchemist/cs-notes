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
**Availability**: probability that the system is continuously available over a period of time (metric of probability).
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
- If we need to update the data, update all of them. (tradeoff between performance and availability / reliability)
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