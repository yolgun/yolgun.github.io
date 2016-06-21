---
title: Cassandra Notes
layout: post
author: yunolgun
permalink: /cassandra-notes/
source-id: 1RkkdO-xYwgTH-gnugIPQe4dAUQztgXDB7Jkus7O2I-o
published: true
---
RDBMS--

* Consistency is violated at distributed systems because of replication lag.

* Third Normal Form doesn't scale. Denormalization is now norm. Dİsk seeks should be as low as possible.

* Sharding at RDBMS: No joins, no aggregations, denormalized, secondary indexes requires access to every shard, adding shards require moving data, schema changes are hard.

CASSANDRA INTRO

* Hash Ring: All nodes are equal. All nodes can read and write.

* Primary Key: Partition Key + Cluster Key

* Cassandra: No consistency, automated sharding-rebalancing, no master/slave, commodity hardware, denormalize at data modelling, goal is to always hit 1 machine.

* Cassandra chooses AP for CAP theorem.

* Consistency Levels: ALL, QUORUM, ONE. Can be set per query(read or write).

* Multi DC. Replicates async. Replication factor per keyspace. An example one DC for OLTP, one for OLAP.

* **Hinted handoff**: Used for synchronization of downed servers when they come up.

WRITE & READ

* Any node can write or read any query. No master or primary. 

* **Write**: Requested node is **coordinator**. First to **commit log**, an append-only file. Then to **memtable**, in-memory. Memtable is flushed periodically into an **SStable **and a new memtable is created. **SSTables** are compacted behind the scenes.

* No updates or delete in place. Deletes are **tombstone**. Immutability is key for SSTables.

* Every write includes timestamp. Merged through **compaction**. Only latest timestamp.

* **Read: **Requested node is coordinator. Contact related nodes. Pull data from SSTables and merge them. Repair in background if Consistency<ALL with **read_repair_chance**.

CASSANDRA CORE

* **Big Data**: **V**olume, **V**elocity, **V**ariety

* Cluster > Data Center > Rack > Node

INSTALLATION

* **NTP** is important. Cassandra is AP, eventually consistent and handles conflicts by using timestamps.

* Disable swap. **sudo swapoff --all**

* I installed DataStax Community Edition. I choose tarball.  http://www.planetcassandra.org/cassandra/?dlink=http://downloads.datastax.com/community/dsc-cassandra-2.2.4-bin.tar.gz

CONFIGURATION

* In my distro it is under /etc/cassandra. There are several files for configuration. The most important one is cassandra.yaml

* **If using HDDs, use different disks for data directory and commit log directory for performance.**

* Logs are stored under /var/log/cassandra/system.log

TOOLS

* Cassandra comes with many tools. Important ones are:

* **Nodetool:** Used for administration and monitoring of the cluster. status, info, ring and many other commands.

* **cqlsh: **Used as command-line shell for Cassandra.

* **stress: **Used for benchmarking.

* sstableloader, sstable2json, json2sstable

ARCHITECTURE

* Seeds are used to join a cluster at the beginning. After that it is not important. There should be more than one **seed **per cluster.

* The node taking read or write request is considered coordinator for this request. Any node can **coordinate** any request.

* Clients choose which node is going to be coordinator. Default policy is **TokenAware**.

* **Replication Factor (RF) **is set per keyspace.

* **Consistency Level (CL) **: How many nodes must **acknowledge **read or write request. Can be set per request. Possible values are: ANY, ONE, QUORUM, ALL.

* **Consistent Hashing: **Based on hashing partition key of the record a **token **is generated. Every node in the ring knows which nodes are responsible from a given token.

* A **partitioner **is used to for hashing. Default one is **Murmur3Partitioner**. All partitioners must be same across the cluster.

* Instead of one single token multiple **vnodes **are used to increase scalability. This way each physical node owns multiple tokens. Specified with **num_tokens **field in cassandra.yaml. Default value is 256.

REPLICATION

* Each node belongs to one rack and one data center. It can be specified using cassandra-rackdc.properties file.

* Replication factor is set when a keyspace is created. Common ones are **SimpleStrategy **(one factor for the cluster) or **NetworkTopologyStrategy **(one factor for each data center).

* All partitions are replicas. There are no primary or secondary replicas.

* **Replication factor of 3** is recommended minimum.

* **Hinted handoff**: Store the hint in system.hints table for downed nodes and replay them when downed node becomes available. It can be enabled (**hinted_handoff_enabled**) and **max_hint_window_in_ms** can be defined in cassandra.yaml. Longer downs should be handled by other repair mechanisms.

TUNABLE CONSISTENCY

* It means how many nodes should acknowledge a read or write request before responding.

* Most common ones are **LOCAL_ONE** and **LOCAL_QUORUM**.

* Immediate consistency requires (nodes_written + nodes_read > RF). 

WRITE ALL + READ ONE 

WRITE ONE + READ ALL 

WRITE QUORUM + READ QUORUM

* ALL consistency queries may fail if a node is down.

* Server clock synchronization is important because conflicts are resolved with timestamp.

* **Eventual consistency** is not bad. Netflix uses it. Only a few milliseconds of stale data.

* Using **nodetool getendpoints **you can see which partition key will be owned by which nodes.

ANTİ ENTROPY

* With CL ALL, a read query will repair inconsistencies before returning a result.

* **read_repair: **repair inconsistencies during read path using digests.

* **read_repair_chance**: For lower consistency level read queries, sometimes all nodes will compare their requested value and fix them. This happens asynchronously so the client may already got wrong answer.

* **nodetool repair **makes all data on a node consistent with the most current replicas in cluster. Think of it as synchronization. 

* Use nodetool repair sparingly. When recovering a failed node, bringing up a downed one. Run periodically on infrequently read data at least every **gc_grace_seconds** (default 10 days). Otherwise a deleted column may still live on a downed node which can repopulate live nodes with deleted column if it is repaired later. This is called **data resurrection**.

GOSSIP PROTOCOL

* This is how nodes in a cluster communicates each other to be aware of changes.

* Once per second each node contacts 1-3 other nodes and exchange information. Next time with different nodes. Deltas are used for optimization.

* **seeds **list is used for entering a cluster. Besides that it is also the preferred choice once per ten gossips. This way cluster data is propagated faster.

* **nodetool gossipinfo** can be used to get latest gossip information of nodes.

DATA MODELLING INTRO

* If you have a simple one primary key at your table, each row corresponds to one partition. This is called **narrow row**.

* When you have a partition key + cluster key as your primary key, each partition may contain multiple rows.

* A partition is minimum inseparable unit for distribution and replication.

* Cassandra provides **fast read** performance by making you provide one denormalized table per query type. This way each query needs only **one disk seek and one sequential read**. 

* Column family means like sparse database table.

* Each partition must be able to reside in one node.

* Maximum column per row is 2 billion. In practice 100 thousand. Because of compaction and read repairs.

* Maximum data size per cell, column value, is 2 GB. In practice 1 MB.

* A **static **column is one per partition. Each row in a partition shares this value. Physically, they exists only once per partition. primary key columns can not be static.

* **CLUSTERING ORDER BY **determines layout of fields in a partition, at table creation time. With **ORDER BY **clause, you can only reverse this order in a query. There is no sorting across partitions.

* **Secondary indexes **are for searching convenience and generally should be avoided. Normally only RF nodes are responsible from a query. With secondary indexes all nodes should be searched. **Low-cardinality **columns and smaller datasets are ok with secondary indexes. Counter column tables, frequently updated or deleted columns are not good. Also beware that each index increases write time. Generally **avoid them** in production code.

* **counter **data type is a special data type used for well, counting things. They should only be used at dedicated tables, primary key and counter. Beware that they are not 100% accurate. In case of false failures and retries, they can be updated twice.

* **UUID** and **TIMEUUID **are special data types used for unique identifiers. TIMEUUID also includes a timestamp in it which can be extracted. **TIMEUUID **is especially good when used in a cluster column since they will be sorted based on time.

* Collection types(**set, list, map**) are multi-value columns. There can be at most 64000 elements in a collection. Maximum size of a collection is 64 KB. Columns can not be part of primary key and they can not be nested in other collections. There can be secondary indexes on collections but for map it is only for key.

* User defined.types, **UDT**, can consist of other types.

* **frozen **keyword means part of a field can not be changed. All of it should change.

* Inserts are atomic and isolated at partition level.

* Each insert or update requires full primary key.

* Updates are inserts and inserts are updates. So, no need for an **upsert** operation.

* There is also **IF NOT EXISTS **clause. But if not exists clause requires read then write so it is slower.

* **Compare and set **operations enable you to say update if this is true.

* **TTL** can be specified per column or row. After TTL passes data becomes tombstone and gets removed within gc_grace_seconds period.

* A partition, row or column can be **deleted**.

* **Batch does not mean bulk loading**. All statements of a batch will receive the same timestamp. Be careful, since all statements has same timestamp inserting and deleting same column will be problematic. If any **LWT** fails all statements in a batch will fail.

* **ALLOW FILTERING** clause allows you to query with left-side clustering keys without specifying a partition key.

DATA MODELLING

* Write **every query** your application needs. Create tables specific to these queries to satisfy SLA.

* Conceptual (Entity-Relationship) -> Logical (Column family) -> Physical (Implementation)

* Chen notation is advised for Entity-Relationship diagram.

* Chebotko Diagram for Column family diagram.

WRITE PATH

* RDBMS has preset disk locations for data. So it requires lots of disk seek and it is expensive.

* Cassandra is a log-structured storage engine. Writing is very fast, just append to commit log.

* 1. Write data to **CommitLog**.

2. In parallel, write data to **memtable**.

3. Periodically, turn memtable into **SSTable**s. SSTables are immutable. This step also clears memtable, JVM heap and commit log.

4. Periodically merge** **SSTables, which is called **compaction**.

* When a Cassandra node restarts, memtable is populated by using CommitLog data.

* memtable flush to disk when commitlog size reaches **commitlog_total_space_in_mb**.

* Best practice is to locate CommitLog on its own disk to minimize write head movement. **commitlog_directory**.

* Similarly commit log is either committed as **batch **or **periodic**.

	**batch: **writes are acknowledged after fsync which happens every **commitlog_sync_batch_window_in_ms**: 50 ms default

**	periodic: **writes are acknowledged immediately and data is fsynced every **commitlog_sync_period_window_in_ms: **10000 ms default

* This means if whole data center goes down there is a chance **we will lose acknowledged data**.

* memtable flush to disk when memtable size reaches **memtable_total_space_in_mb**.

* memtable flush to disk when **nodetool flush **command is used.

* There is **no two phase commit**. Which means unacknowledged writes may have happened. An operation is **Idempotent, **if it results same whether it happens once or multiple times. Counters are not idempotent.

* **sstable2json ** tool can be used to convert an sstable into json file.

READ PATH

* Read is more complex than write. A read may require multiple disk seeks.

* You can see details of a read by using **tracing on** command at cqlsh.

* row_cache is generally not desired and disabled by default.

* key_cache is useful and enabled by default.

* If location of a partition key is not found in key_cache, partition summary(off-heap) will be used to find approximate location of key in partition index(disk) to find data in partition.

* **nodetool cfstats <table_name> **can be used to get stats of a table.

DELETE, COMPACTION

* Deletes are just special writes.

* When a column gets deleted or TTL expires related column in memtable is marked. With memtable flush it becomes a tombstone. At each compaction, tombstones living longer than **gc_grace_seconds** gets evicted.

* This  eviction may cause zombie columns to resurrect. As a best practice, run **nodetool repair**, more frequently than gc_grace_seconds. Or, use nodetool repair when restoring failed nodes.

* Compaction is used to merge multiple SSTables into one and clear deleted data. It is I/O heavy but after it, reads will be optimized(less disk seeks, less fragmentation).

* Compaction is determined at table level.

* Size-Tiered:default(write-heavy and time-series), Leveled(read-heavy), Date-Tiered(TTL, time-series) compaction strategies.

* Size-Tiered compaction **requires 2 x disk space as largest CQL table**. Read latency may show spikes.

* Leveled compaction requires 10% free disk space. Predictable read latency. Generally used with SSDs. Each read touches only one SSTable.

* Date-tiered compaction, worst case still double disk space. With TTL it may drop whole SSTable at once. Good for TTL data.

* **compaction_throughput_mb_per_sec** can be set higher. Default is too low.

HARDWARE

* More memory is better for reads. Any available memory will be used by OS file system cache. Sweet spot is **16GB to 64GB**.

* Writes are CPU bound. Sweet spot is **8 cores**. But more is better of course.

* Size-tiered compaction requires **50%**, leveled compaction requires **10% free disk space**. Date-tiered somewhere between. Sweet spot is **500 GB to 1TB**. Maximum 3TB-5TB. Compaction becomes too expensive beyond that. For HDDs, **two drives **to avoid head movements. One for data, one for CommitLog. SSD is always better. SSD always has lower latency than a RAID.

* Internal communication uses listen_address and Thrift/RPC uses rpc_address. Bind interfaces to **separate NICs**. Gigabit ethernet or better.

* **Avoid**: SAN. Excessive heap space. Too many racks.

* Read this paper:

http://www.datastax.com/wp-content/uploads/2014/04/WP-DataStax-Enterprise-Best-Practices.pdf

CASSANDRA OPERATIONS&TUNING

* **terminator broadcast feature** or cssh is very good at managing multiple machines at the same time.

* **update-alternatives **program can be used to install a command.

* **":r !hostname -I" **can be used at vim to insert hostname into the document.

* **Ctrl+z** puts process into background. **%1 **lets you return to it.

* **clipit **utility helps you keep copy past history.

MANAGEMENT

* When adding a node it is important to change  **cluster_name, listen_address, rpc_address, seeds** fields at cassandra.yaml file.

* We have to add new nodes when there is too much data(at maximum 1 TB per server), too much traffic(throughput, latency), operational headroom(50% free disk space for compaction).

* Adding a new node requires moving some part of the token range to the new node. Using vnodes makes management easier when adding new nodes.

* If you are adding multiple nodes, it is better to add them all at once to avoid unnecessary data streaming.

* **nodetool decommission **is used to deactivate online nodes from the cluster by streaming out their data. Close the cassandra process from decommissioned node manually.

* **nodetool removenode **is used to remove offline nodes from other nodes. Useful when the node to be removed is not accessible anymore.

* **Chaos Monkey** can be used to test failures at a cluster.

* All additions and removals at single-node clusters requires using **nodetool move **to rebalance cluster.

* When joining a new cluster **bootstrap **is used to get its data from other nodes.

* When adding new node, streamed out data is not cleaned up immediately for safety and performance reasons. It is going to be removed at compaction phase. If you need it immediately run **nodetool cleanup** commands. It is like a single SSTable compaction.

* You can replace a dead node by inserting **JVM_OPTS="$JVM_OPTS -Dcassandra.replace_address=address_of_dead_node" **to cassandra-env.sh file.

MAINTENANCE

* If a node is down longer than **max_hint_window_in_ms**(default 3 hours), a manual repair is necessary before rejoining cluster

* Data can get inconsistent. **Repair **weekly with default **gc_grace_seconds**(10 days).

* Run repair one node at a time at low-usage hours. It is I/O intensive.

* If data is too big, you can divide repairs into subranges.

* **Snapshot **is just a hard link to immutable SSTables.

* **Incremental backups **create a hardlink to every SSTable upon flush.

* **Auto snapshot **takes a snapshot before tables are truncated or dropped.

* Both snapshots and incrementals are necessary for backup and recovery process. It is better to take backups at remote locations with a scheduled job.

* Recovery Process: Save schema. Take snapshots at every node. Copy snapshots to remote file systems periodically. When recovering, run schema. Then just copy old snapshots into appropriate directories at each node. Call **nodetool refresh**. And it is there.

* Cassandra supports client-to-node and node-to-node(necessary?) **security**.

* Superusers can be created which can create other users and grant them permission at table level.

* It is good practice to increase replication factor of **system_auth** keyspace to number of nodes for better performance.

* Change **authenticator **to **PasswordAuthenticator **and **authorizer **to **CassandraAuthorizer **in cassandra.yaml file.

* Secondary indexes may get out-of-sync(how?). Use nodetool **rebuild_index **to recreate indexes.  You have to run it at every node.

* Internally indexes are just tables. Hence you can still use old indexes while new ones are being built. Beware that this process is essentially a write and so it is I/O and cpu intensive.

TUNING

1. The ultimate goal is to increase **latency **and/or **throughput**.

2. A Well Defined Goal: type(read/write/delete), latency(percentile), throughput(op/s), size(op size), duration, scope

3. Verify: hooks in application, query trace, **jmeter**, customized stress tool (cassandra-stress)

* **nodetool tpstats **is a useful tool to monitor thread pool of Cassandra. Especially "All time blocked" and dropped fields are very important.

* Messages can be dropped in case of resource contention between different threads. If you see one, investigate it.

* **nodetool netstats **can show attempted and done read repairs. Number of read repairs shows how synced is your cluster.

* **OpsCenter** is a very useful tool to monitor and manage your cluster.

* **nodetool getlogginglevels and nodetool setlogginglevel **is very useful to change logging level dynamically to troubleshoot.

* Monitoring jvm stats is important. **jstats,**** jconsole****, jvisualvm **are good tools to use.

* Write is CPU-bound, read is memory-bound. Encryption and compression is also CPU-bound. By following CPU stats you can see your limit and if you can use compression and such.

* **dstat **is a good tool to analyze disk or net usage.

* **nodetool cfhistograms **is good to see latency for reads and writes.

* **nodetool proxyhistograms **shows this from view point of coordinator. So it also includes network latency.

* cql **tracing **helps you to identify which node and stage is responsible from latency.

* **nodetool settraceprobability **lets you keep track of latencies at a slower rate. Very useful.

* **Survey Mode** seems fun for benchmarking. Analyze it.

* **nodetool cfstats **is good to see some stats per table.

* Avoid creating more than 500 tables. Each table takes up 1MB at heap.

* Keep wide rows under 100 MB or 100k columns.

* TokenAware load balancing policy lets you avoid hops. Good.

* Prepared statements is good.

