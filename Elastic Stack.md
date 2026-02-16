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

Observability
Use beats to store data and represent data about uptime and how different platforms are behaving and different metrics, how users are interacting (click throughs, how much time peoople are spending on pages.)

Security
Overview page that shows different alerts from internal and external, different type of events, host events can send auditbeat ,elastic endpoint, filebeat and winlogbeat

## KQL Queries

Baseline hunting filter (clean starting point)
Start with:

### event.kind:event

Security Onion sends lots of:
metrics
internal service logs
enrichment data

Removes:
metrics
pipeline/system logs
enrichment artifacts

Then add:

### event.category:network OR event.category:process OR event.category:file

Why?

These are usually the most valuable behavior categories:
network = connections, DNS, HTTP
process = execution activity
file = creation/modification

This already cuts a lot of noise.

## Network-only baseline

### event.kind:event AND event.category:network AND event.type:connection

What this gives you:

Clean Zeek-style network telemetry
Removes Suricata alerts and SOC service logs
Easier pivoting by protocol

## Then pivot into specific protocols

Now you can layer on things like:

## DNS hunting
###network.protocol:dns

## HTTP hunting
### network.protocol:http

## TLS hunting
###network.protocol:tls

Notice:
👉 Protocol filters come last, not first.

Your Discover view contains mixed datasets:

soc (system logs)
zeek.*
suricata.*
endpoint.*

Starting with event.kind:event prevents you from accidentally hunting through SOC internal logs — which is exactly what you were seeing earlier.


## Add this field as a column in Discover:
### data_stream.dataset

When you hunt you’ll instantly see:

zeek.dns
zeek.conn
suricata.alert
soc

### NOT data_stream.dataset:soc

What disappears:
Security Onion system logs
sensoroni logs
internal services
agent lifecycle noise

### agent.type:(filebeat OR elastic-agent OR packetbeat)
This limits results to real collectors.

Put them together (baseline clean view)

Try this:

### event.kind:event AND NOT data_stream.dataset:soc AND event.category:network

Then pivot with:

### network.protocol:dns

or

### network.protocol:http


## Other KQL Queries

### http.response.status_phrase : ok

## Hunt for Partial HTTP responses
A lot of suspicious activity can live in:

206 → partial downloads (chunked payloads, staged exfil, resumable transfers)
304 → caching tricks
302/301 → redirects used in phishing chains

### network.protocol:http AND http.response.status_code >= 200 AND http.response.status_code < 300

or 

### network.protocol:http

and then pivot by status code visually


### Discover

<img width="505" height="161" alt="image" src="https://github.com/user-attachments/assets/2df5f7db-0f13-4261-8204-c0b2ca8a5f16" />

<img width="469" height="438" alt="image" src="https://github.com/user-attachments/assets/d943e39e-581c-4d76-9a3e-f78b4f8b485c" />

<img width="470" height="441" alt="image" src="https://github.com/user-attachments/assets/c9c36bf5-ef6f-4bb9-9268-e4293f01db2e" />

### Example Hunt

Found a suspected malicious hash

ran it as a query
<img width="2014" height="121" alt="image" src="https://github.com/user-attachments/assets/27840cfc-488d-4aea-b634-49a421575bef" />

Filtered on hostname and file hash
File executed onto a box and we can see if anyone else had this file execute 

<img width="2024" height="932" alt="image" src="https://github.com/user-attachments/assets/2d35ff7d-2f0d-4e24-bddd-dd4e8ef5da06" />

 ### Next we changed to winlogbeat
Want to look at any files that were created 

We found that suspected malicious files were created inside of "C:\Users\packetpub\AppData\Roaming" and combined that with event.type:creation
Will show any files created in that directory
<img width="881" height="100" alt="image" src="https://github.com/user-attachments/assets/31210e8e-4081-4cfc-856f-7e682021ffe9" />

<img width="2030" height="946" alt="image" src="https://github.com/user-attachments/assets/262a5844-00d7-4be4-b306-cd27eabdf56e" />

From here we can add addtional information as columns to see what was created.

host.hostname
file.directory
file.name
event.type

We can add much more information if we wanted via flyout

<img width="2026" height="626" alt="image" src="https://github.com/user-attachments/assets/4092a491-ffe7-4016-806a-4b23346be2a0" />

### Next we went back to the Logs index pattern

Created a filter to look at network data and endpoint data

event.category: network
process.executable is 

<img width="780" height="427" alt="image" src="https://github.com/user-attachments/assets/a2c6bf69-e5bc-4082-840c-8ea16dd364c1" />

Need endpoint data to do this

In the demo we just wanted the suspicious file that was created
<img width="628" height="245" alt="image" src="https://github.com/user-attachments/assets/29296775-8d51-4f6c-9942-2b7e41236b71" />


We also added dns.answers.data & dns.question.name and could search for either of these

<img width="2062" height="831" alt="image" src="https://github.com/user-attachments/assets/c2deba9d-bb3b-4ba2-a91c-c51441b3f699" />

then we added process.executable to see which process we're tracking
<img width="2026" height="873" alt="image" src="https://github.com/user-attachments/assets/494fc7a1-d44b-41e8-9874-846c9c4b10aa" />


As well as destination.port
<img width="2019" height="907" alt="image" src="https://github.com/user-attachments/assets/41b65894-01d4-417f-8377-26fe9565b2d1" />

From here we see traffic going to destination port 587

so we filtered on event.category:network and also filtered on process.executable: and added a destination port column and saw that it is reaching out to port 587 which is suspicious because it is not a mail client

<img width="681" height="543" alt="image" src="https://github.com/user-attachments/assets/444e7a27-ff60-4dd1-87cf-d33ec0702134" />

<img width="696" height="729" alt="image" src="https://github.com/user-attachments/assets/b5fd3981-a3d5-40c2-9e44-2e5be8af3f35" />

<img width="639" height="251" alt="image" src="https://github.com/user-attachments/assets/d75fc54b-feed-49cd-8e92-c20bdae15fbf" />






























