# Database Management Systems
## Distributed Databases
**Transparency** (User perspective)
- Data organization transparency
  - Location transparency: We don't care where the data comes from.
  - Naming transparency: Tables from multiple places could have different names, but users refer to that by only one logical name.
- Replication transparency: Users don't care about which copy of data they are talking to.
- Fragmentation transparency
  - Horizontal: break a relation into subsets of tuples.
  - Vertical: break a relation into subsets of columns. (store pk with each fragment for reconstruction)

**Autonomy**: Can notes operate independently? Design / Communication / Execution autonomy.

Why we want to distribute the data? For better **fault tolerance**.
**Reliability**: probability that a system is running.
**Availability**: probability that the system is continuously avaliable. (Replication)
Flexibility, improved performance, easier expansion.

**Schema Architecture**
[image](./picture/Schema-Arch.jpg)

**How the query works**
[image](picture/Component-Arch.jpg)
- Mapping
- Localization
- Global Optimization (Analyse data transfer costs)
- Local Optimization (index, query optimization, ...)