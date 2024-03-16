### Cassandra

Cassandra is a distributed, decentralized, scalable, eventually consistent and highly available NoSQL database. It is designed to handle large and growing datasets by distributing the workload across multiple machines. Unlike traditional master-slave architectures, Cassandra operates in a decentralized manner, with all nodes being identical, ensuring fault tolerance and eliminating single points of failure.

Key features of Cassandra include its high availability and fault tolerance, which allow it to continue functioning even if individual nodes fail. It offers seamless scalability, allowing nodes to be added or removed without significant performance impact or manual intervention. Additionally, Cassandra provides high throughput for write operations, making it suitable for applications with demanding write workloads.

When considering whether to use Cassandra, factors to consider include the size and growth rate of your data, the importance of high availability, the dynamic nature of your data, and the tolerance for eventual consistency. Cassandra may not be suitable for applications requiring strict consistency guarantees or static data.

In summary, Cassandra offers a robust solution for distributed, scalable, and highly available data storage, particularly for applications with large and dynamic datasets and demanding write workloads.

### Keyspace
Keyspace that is nothing but a container for tables.


### Primary Key
The primary key in Cassandra comprises two parts: the partition key and the clustering key. The partition key determines data distribution across nodes, while the clustering key determines data storage within a single machine.

Data distribution is achieved by assigning each node a range of tokens, which determines the rows mapped to that node. Token ranges are generated using a hashing function, with each node receiving a portion of the token space. For example, if an employee record with a partition key of ID 20 is hashed to token 62, it will be mapped to the node responsible for tokens 61 to 80.

The clustering key dictates how data is stored within a single machine. Rows with the same partition key are stored together, with data sorted based on the clustering key values. For instance, if multiple rows share the same partition key PK1 and clustering keys K1 and K2, they will be stored consecutively, sorted by the clustering key values.


## Replication

The replication factor determines the number of copies of data stored in the cluster. For example, a replication factor of three means that three copies of data will exist across different nodes in the cluster. The replication strategy defines where these copies will be placed.

There are two main replication strategies: the simple strategy and the network topology strategy. In the simple strategy, replicas are placed at consecutive nodes. For instance, if the replication factor is three, the coordinator node assigns replicas to the two consecutive nodes following the primary node.

In contrast, the network topology strategy is used for multi-data center deployments. This strategy allows for different replication factors to be specified for different data centers. For example, one might specify a replication factor of three for a data center in the USA and a replication factor of two for a data center in Europe.

When a write request is initiated by the client, the coordinator node determines the primary node and replica nodes based on the chosen replication strategy. If using the network topology strategy, the coordinator may also communicate with remote coordinators in other data centers to ensure data replication across multiple locations.

### Wrtie consistency

1.  **One Consistency**:
    
    -   With One consistency, the coordinator node sends the write request to only one replica node, typically the closest or fastest node.
    -   Once the write is acknowledged by this single node, the coordinator considers the write successful.
    -   However, if the acknowledged node fails before replicating the data to other nodes, the data may be lost.
2.  **All Consistency**:
    
    -   All consistency requires the write to be acknowledged by all replica nodes in the cluster.
    -   The coordinator sends the write request to all replica nodes and waits for acknowledgments from each of them.
    -   This ensures that the data is replicated to all nodes before considering the write successful, providing strong consistency guarantees.
    -   However, it can be slower and less resilient to node failures compared to lower consistency levels.
3.  **Quorum Consistency**:
    
    -   Quorum consistency balances between One and All, offering both performance and durability.
    -   The coordinator sends the write request to a subset of replica nodes determined by the quorum value, which is typically set to a majority of nodes.
    -   Once the write is acknowledged by the required number of nodes (quorum), the coordinator considers the write successful.
    -   Quorum consistency provides strong consistency guarantees while still allowing for efficient writes and fault tolerance.
4.  **Local Quorum Consistency**:
    
    -   Similar to Quorum consistency, but limited to replicas within the same data center as the coordinator node.
    -   Write requests are sent to a subset of replica nodes within the local data center, providing strong consistency within the data center.
    -   This ensures that writes are resilient to data center failures while maintaining low latency and efficient replication within the local data center.

### Read Consistency
1.  **Level One Consistency**:
    
    -   The coordinator node sends requests only to the fastest replica node based on information from the Snitches program, which knows the network topology.
    -   Once the fastest node returns the requested data, the coordinator node checks for consistency among other replica nodes. If inconsistencies are found, a read repair request is initiated in the background.
2.  **Quorum Consistency**:
    
    -   The coordinator node sends read requests to two nodes: the fastest node for the actual data and the second fastest node for a hash of the data to save network bandwidth.
    -   Upon receiving the data and hash, the coordinator node compares them for consistency. If inconsistent, another read request is sent to the second fastest node for the actual data.
    -   The coordinator node then merges the data based on the latest timestamp and sends it to the client.
    -   A read repair request is initiated if inconsistencies are detected.
3.  **Local Quorum Consistency**:
    
    -   Used in multi-data center setups, where the coordinator sends read requests to replicas in the same data center.
    -   Similar to Quorum consistency, but limited to replicas within the coordinator's data center.
    -   Inconsistencies across data centers are also checked, and read repair requests are initiated if inconsistencies are found.

### Gossip Protocol
1.  **Gossip Protocol Overview**:
    
    -   Every node in the Cassandra cluster engages in gossiping, similar to human gossip, but with truthful information sharing.
    -   Gossiping facilitates the efficient dissemination of state information across the cluster, ensuring that all nodes are up-to-date.
2.  **Information Exchange**:
    
    -   Nodes initiate gossip sessions with each other, exchanging state information.
    -   For example, if node A starts gossiping with node D, node D responds by sharing its state information, which includes details about itself and other nodes it has knowledge of.
    -   This continuous exchange of information ensures that the cluster remains synchronized and informed about the state of all nodes.
3.  **Failure Detection**:
    
    -   If a node fails to respond during a gossip session, it is not immediately marked as dead.
    -   Instead, a suspicion level is assigned to the node, indicating the probability of it being dead.
    -   Cassandra employs a probabilistic failure detection model, known as the "killer" failure detection model, to assess the likelihood of node failure.
4.  **Gossip Class**:
    
    -   In addition to regular gossip sessions between nodes, Cassandra utilizes a gossip class to initiate random gossip sessions with nodes in the cluster.
    -   When a gossip session is initiated with a node (e.g., node F), it responds by sharing its state information, which is then updated and maintained locally by the gossip class.
