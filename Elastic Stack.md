Document: JSON object that contains data stored in Elastic Search


Node: An instance of Elasticsearch
Unique ID and name
Belongs to a single cluster
Cluster: Group of nodes
Indices: used to group documents that are related to each other
Indices do not actually store documents but reference where documents are.

Shard: Where data is stored and where you store
One shard with one index by default.

Sharding: Distributing shards across nodes
When you create an index, one shard created by default and assigned to a node
Also where you run searches 

Number of documents a shard can hold depends on capacity on node

Sharding speeds up search
Can run through searches in parallel
Can search more data and search in scale 

Have primary and replica shards
P0 -> R0
P1 -> R1 
<img width="1844" height="815" alt="image" src="https://github.com/user-attachments/assets/fb63102e-c470-4a31-bf54-aa0e7bb777a8" />


CRUD Operations

Create
Read 
Update
Delete


Port 9200
Elasticsearch provides a REST API that allows you to commuinicate with your cluster

When you send requests to Elasticsearch, you're sending an HTTP REQUEST to this REST API

<img width="753" height="1513" alt="image" src="https://github.com/user-attachments/assets/3284a433-60e9-41a6-ad32-1805215b6f40" />

































