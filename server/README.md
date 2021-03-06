Simba Cloud
===========

Prerequisites
-------------
Before running the Simba Cloud you must have operational deployments of Apache Cassandra and OpenStack Swift.  

Configuration
-------------
You must set certain configuration options in the following files:  
```
./common/src/resources/client.properties  
./gateway/src/resources/gateway.properties  
./simbastore/src/resources/simbastore.properties  
```

In `client.properties`, set:  
  * `gateways`: Comma-separated list of gateway nodes.  
  * `consistency`: Consistency level used by test client (Default: `CAUSAL`). _optional_.  

In `gateway.properties`, set:  
  * `simbastores`: Comma-separated list of store nodes.  
  * `backend.server.thread.count`: Number of Gateway backend server threads. Set equal to number of store nodes.   

In `simbastore.properties`, set:  
  * `simbastores`: Comma-separated list of store nodes.  
  * `cassandra.keyspace`: Keyspace name for Simba metadata. Must also create it in Cassandra (Default: `simbastore`).  
  * `cassandra.seed`: Set Cassandra seed node host or IP.  
  * `swift.container`: Container name for Simba objects. Must also create it in Swift (Default:  `simbastore`).  
  * `swift.identity`: Swift account name.   
  * `swift.password`: Pass key for Swift account.  
  * `swift.proxy.url`: Swift proxy URL.  

Cassandra Setup
---------------
Prepare your Cassandra deployment by creating the following keyspace and tables.  

```  
CREATE KEYSPACE simbastore WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};  
CREATE TABLE simbastore.metadata (key text PRIMARY KEY, consistency text);  
CREATE TABLE simbastore.subscriptions (key uuid PRIMARY KEY, subscriptions list<blob>);  
```

This assumes you have set `cassandra.keyspace=simbastore` inside `simbastore.properties`. Otherwise, replace the keyspace name to match the name you have chosen.  

Swift Setup
-----------
Prepare your Swift deployment by creating the container with the name you have set for `swift.container` inside `simbastore.properties`.  

Using the Swift command-line tool, the command is:  
```  
swift -A ${swift.proxy.url} -U ${swift.identity} -K ${swift.password} post ${swift.container}  
```  

Compilation
-----------
To compile, run:  
```
./compile.sh  
```
JARs with dependencies will be built in folders ./{common,gateway,simbastore}/target/.

Starting Simba Store
--------------------
NOTE: Must start Store(s) before Gateway(s).  
```
java -jar simbaserver-simbastore-0.0.1-SNAPSHOT-jar-with-dependencies.jar -p simbastore.properties   
```

Starting Simba Gateway
----------------------
```
java -jar simbaserver-gateway-0.0.1-SNAPSHOT-jar-with-dependencies.jar -p gateway.properties  
```

Running Linux Test Client for Simba
-----------------------------------
```
java -cp simbaserver-common-0.0.1-SNAPSHOT-jar-with-dependencies.jar com.necla.simba.testclient.Client <workload file> <output folder> <prefix>  
```

Deployment on PRObE
-------------------
Want to deploy Simba on PRObE? Find out more [here](scripts/probe/README.md)!

Scripts for generating tables/workloads
---------------------------------------
Can be found in ./scripts/  

