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

When you send requests to Elasticsearch, you're sending an HTTP REQUEST to this REST API whose endpoint is localhost on port 9200 {127.0.0.1:9200}

<img width="753" height="1513" alt="image" src="https://github.com/user-attachments/assets/3284a433-60e9-41a6-ad32-1805215b6f40" />

## Kibana

We see that the local host is running on port 5601

copy & paste to browser

<img width="1104" height="130" alt="image" src="https://github.com/user-attachments/assets/511b8d9e-9931-4a05-8aca-b311bb90c04e" />

#### Kibana is a web interface for Elasticsearch. Kibana ships with its backend server that communicates with Elasticsearch.

When you start up a node, cluste rins formed automatically.

Best practice is to name cluster and node to something that is meaningful.
Reason is that as apps get bigger, may be working with multiple clusters with multiple nodes that are responsible for different things

### Kibana GUI 
https://www.elastic.co/guide/en/elasticsrarch/reference/current/rest-apis.html

<img width="2144" height="1118" alt="image" src="https://github.com/user-attachments/assets/d2997a37-658f-482f-a471-db755dc69078" />

<img width="802" height="409" alt="image" src="https://github.com/user-attachments/assets/2f9ff607-63be-4786-a07e-0d3de6787576" />


















































