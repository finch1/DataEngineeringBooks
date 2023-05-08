# A data intensive applications
<p><b>Databases</b> - Store data so that other applications can find it again later</p>
<p><b>Cache</b> - Remember thre results of an expensive operation to speed up reads</p>
<p><b>Index</b> - Allow users to search data by keyword or filter</p>
<p><b>Stream Processing</b> - Send a message to another process to be handled async</p>
<p><b>Batch Processing</b> - Periodically crunch a large amount of accumulated data</p>

## Tools
<p><b>Redis</b> - In-memory database also used as message queues</p>
<p><b>Kafka</b> - Message queues with database like durability guarantees</p>
<p><b>Memcached</b> - Distributed key value store, in-memory data store. Mature scalable, open source solution for delivering sub millisecond respnse times making it useful as a cache or session store</p>
<p><b>Hadoop</b> - In a batch processing system such as Hadoop, we usually care about <i>throughput - the number of records we can process per second or the total time it takes to run a job on a dataset of a certain size. Hadoop is a framework that manages big data storage in a distributed way and processes it parallely.</i>
<p> Hadoop Distributed File System (HDFS) is cabable of storing large and various data sets in comodity hardware.</p>
<p> Hadoop MapReduce is a programming technique where huge data is processed in a parallel and distributed fashion, on the slave nodes. It extracts keys from data and stores the reference as the value.</p>
<p> Hadoop Yet Another Resource Negotiator (YARN) acts like an OS for Hadoop. It manages cluster resources and does job scheduling.</p>
</p>
<p><b>HBase</b> - </p>
<p><b>Hive</b> - </p>

<img src="Figure D.png"> <img src="Figure E.png"> <img src="Figure F.png"> <img src="Figure G.png"> <img src="Figure H.png"> <img src="Figure I.png"> <img src="Figure J.png"> <img src="Figure K.png">

## Reliability - Tolerating hardware and software faults - human error
- The system should continue to work *correctly* (performing the correct function at the desired level of performance) even in the face of *adversity* (hardware or software faults, and even human error).
- It is usually best to design fault-tolerance mechanisms that prevent faults from causing failures. Many critical bugs are actually due to poor error handling. Deliberately inducing faults, ensures fault-tolerante systems can handle faults.
- Provide guarantee - A message sueue, the number of incoming messages matches the outgoing. It can constantly check itself and raise an alert if a discrepancy is found.
- Minimize opportunities for error: well-designed abstractions, APIs, make it easy to "do the right thing and intutive". 
- Provide tools to recompute data in case it turns out that the old computation was incorrect.
- *Telemetry*-monitoring show early warning signs and allow us ti check whether any assumptions or constraints are being violated.

## Scalability - Measure load and performance - latency percentiles, throughput
- As the system *grows*(in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth.
- "If the system grows in a particular way, what are our options for coping with the growth?"
- "How can we add computing resources to handle the additional load?"
- Load: requests per second to a web server, the ratio of reads to writes in a database, number of simultaneously active users in a game, hit rate on a cache

### Twitter Fan-out
A hybrid was requried:
1-2 is used for users with millions of followers.
1-3 for regular users. Maintain a cache for each user's home timeline - like a mailbox of tweets for each recipient user. When a user posts a tweet, lookup all the users who follow that user, and insert the new tweet into each of their home timeline cache. **The request to read the home timeline is then cheap beacuse its result has been computed ahead of time.**

<img src="Figure 1-2.png"/>

<img src="Figure 1-3.png"/>

## Latency and Response time
The *response time* is what the client sees: besides the actual time to process the request (the service time), it includes network delays and queueing delays.
The *latency time* is the duration that a request is waiting to be handled - during which it is *latent* waiting to be serviced. 

Take a list of response times and sort it from fast to slow, then the median is the half way point. If the median is 200ms then half the reposes are returned in less then and the other half is greater than. 95th percentile response time is 1.5 sec, that means that 95 out of 100 requests take less then 1.5 seconds and 5 out of 100 take more. High percentile of response times, known as *tail latencies* are important as these effect the users with most data, high quality users. Using the median response time, we can determine if a fault happened if response is higher then median.

<img src="Figure 1-4.png"/>

Queueing delays often account for a large part of repsonse time, it only takes a small number of requests to hold up the processing of subsequent requests. Splitting messages by processing time into different message queues on different machines minimizes this phenomenon. Further reading *Percentiles in Practice pg.16*

Scaling up/vertically - moving to more powerful machines
Scaling out/horizontally - distributing the load across multiple machines
Elastic - machines automatically add resources when they detect load increase

## Maintainability - Operability, simplicity and evolvability
Over time, many different people will work on the system(engineering and operations, both maintaining current behaviour and adapting the system to new use cases), and they should all be able to work on it *productivly*.

### Operability
Make it easy for operations teams to keep things running smoothly such as not using legacy tools

### Simplicity
Make it easy for new engineers to understand the system, by removing as much complexity. In programming, one of the best tools we have to remove accidental complexity is *abstraction*. Abstraction hides implementation detail behind a clean,simple to use facade.

### Evolvability
Make it easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases as requirements change(*plasticity*)

# Data Models and Query Languages
In NoSQL, joins aren't recommended. But if we do, RethinkDB supports joins.

### Relational vs Document Databases
The main argument in favour of document data model are schema flexibility, better performance due to locality, and that for some applications it is closer to application structure.
The relational model counters by providing better support for joins, many-to-one and many-to-many relationships.

### Which data model?
If your data has a document like structure i.e. a tree of one-to-many relationship, where typically the entire tree is loaded at once, then it's probably a good idea to use document model. In resume example, splitting the document into tables (position, education, contact) can lead to cumbersome schema and joins. limitations are accessing nested data...

For highly interconnected data, the document model is awkward, relational is acceptable, and best are graphs.

Data conformity - schema on write in relational database ensures data matches the table. 

# Stream Processing
**Stream** refers to data that is incrementally made available over time. In reality, a lot of data is unbound because it arrives gradualy over time: users produced data yesterday and today, and they will continue to produce more data tomorrow. So the dataset is never "complete" in any meaningful way. 
<p>The idea behind *stream processing* is to process a second worth of data every second, or even continuously.</p>

### Transmitting Event Streams
Streaming:
- parse the input into a squence of records
- in streaming a record is commonly known as *event* - a small, self contained immutable object containing the details of something that happened at some point in time. 
- in a streaming system, related events are usually grouped together into a *topic* or *stream*.

### Messaging System
A common approach for notifying consumers about new events is to use a *messaging system:* a producer sends a message containing the event, which is then pushed to consumers. 
Unilike TCP - one to one -, a messaging system allows multiple producer nodes to send messages to the same topic and allows multiple consumer nodes to receive messages in a topic. 
- *What happens if the producers send messages faster then the consumers can process them?* There are 2 known options:
- - Buffer messages in a queue
- - Apply *backpressure* (also known as flow control; i.e. blocking the producer from sending more messages). For ex. Unix pipes and TCP use backpressure: they have a small fixed-size buffer and if it fills up, the sender is blocked until the recipient takes data out of the buffer. If messages are buffered in a queue, it is important to understand what happens as that queue grows. Does the system crash if the queue no longer fits in memory, or does it write messages to disk? If so, how does the disk access affect the performance of the messaging system?
- - *What happens if nodes crash or temprarily go offline-are any messages lost?* As with databases, durability may require some combination of writing to disk and/or replication, which has a cost.
- A nice property of batch processing systems is that they provide a strong reliability guarantee: failed tasks are auto retried, and partial output from failed tasks is auto discarded. This means the output is the same as if no failures happened.
- On a different note, if a consumer registers with a producer, producers can make a direct HTTP or RPC request to push messages to the consumer. This is the idea behind Webhooks - a callback makes a request to that URL whenever an event occurs.

### Message Brokers
<p>Also known as message queue, a kind of database optimized for handling message streams. Producers write messages to the broker while consumers receive them by reading from the broker. Some brokers only keep messages in memory, deleting a message when it has been successfully delivered to consumer, while others write to disk. Faced with slow consumers, they generally allow unbound queueing as opposed to dropping messages or backpressure.</p>

<p>A consequence of queueing is also that consumers are generally <i>async</i> when a producer sends a message, it only waits for broker reception confirmation.</p>

<p>Since they quickly delete messages, most borkers assume their working set is fairly small, i.e. the queue is fairly small. Brokers notify consumers when data changes (when new message becomes available)</p>

### Consumers reading messages in the same topic
<p><b>Load Balancing</b> Each message is delivered to <i>one</i> consumer, and consumers share the work of processing the messages in the topic. The broker may assign messages to consumer arbitrarily/multiple clients consume from the same queue. This pattern is useful when messages are expensive to process and additional consumers help parallelize the process.</p>

<p><b>Fan-out</b> Each message is delivered to <i>all</i> consumers. Several independent consumers receive the same messages.</p>

<p><b>Combination</b>Two separate groups of consumers may each subscribe to a topic, such that each group collectively receives all messages, but whitin each group, <b>only one of the nodes receives each message.</b></p>


<img src="Figure 11-1.png">

### Acknowledgments and redelivery
Consumers may crash at any time - a broker delivers a message but the consumer never processes it. Ensureing messages are not lsot, brokers use *acknowledgments*: a lient must explicitly tell the broker when it has finished processing a message so that the broker can *remove* it from the queue.
Then a gain, if a message is processed and acknowledgment not sent, the broker will resend and creates dublicate data. 

During load balancing, messages may lose order sequence. To avoid this, use a separate queue per consumer

<img src="Figure 11-2.png">

### Logs For Message storage
- A Hybrid - combining the durable storage approac of databases with the low latency notification of messaging.
- A log is a simple append only sequence of records on disk.
- With the same reasoning, a producer sends a message, the broker appends it to the end of the log and the consumer receives messages by reading the log sequentially. If a consumer reaches the end of the log, it waits for notifications that a new message has been appended. 
- Even though these message brokers write all messages to disk, they are able to achieve throughput of millions of messages per second by partitioning across multiple machines, and fault tolerance by replicating messages.

### Dual Writes
Race conditions where two clients want to update an item. Unless there is concurrency detection mechanism, one will not even notive that concurrent writes occured - one value will simply silenty overwrite another value. Or one write works and the other faults, still creating inconsistent replication.

### Change Data Capture
The process of observing all data changes written to database and extracting them in a form in which they can be replicated to other systems. CDC is interesting if changes are made available as a astream, immediately as they are written. 

### API support for change streams
<p><b>RethinkDB</b> - allows queried to subscribe to notifications when the results of a query changes.</p>
<p><b>Firebase and CouchDB</b> - provide data synchronization based on a change feed that is also made available to applications.</p>
<p><b>Meteor</b> - uses the MongoDB oplog to subscribe to data changes and updates the user interface.</p>
<p><b>VoltDB</b> - allows transactions to continuously export data from a database in the form of a stream.</p>
<p><b>Kafka Connect</b> - integrates CDC for a wide range of database systems with Kafka. Once the stream of change events is in Kafka, it can be used to update derived data systems and also feed into stream processing systems.</p>

### Commands and Events - betting
When a request from a user first arrives, it is initially a command: at this point, it may fail, for ex. because some integrity condition is violated. The application must first validate that it can execute the command. Any validation of a command needs to happen synchronously, before it becomes an event - ex. by using serializable transaction that atomically validates the command and publishes the event. If the validation is successful and the command is accepted, it becomes and event which is durable and immutable. 

Ex. If a user tries to register a particular username, or reserve a seat on an airplane, then the application needs to check that the username or seat is not already taken. When that check has succeeded, the application can generate an event.

At this point when the event is generated, it becomes a *fact*. Even if the customer later decides to change or cancel the reservation, the fact remains true that they formerly held a reservation for a particular seat, and the change or cancellation is a separate event that is added later.  

### Concurrency control
The biggest downside of event sourcing and CDC is that the consumers of event log are usually aysnc, so there is a possibility that a user may make a write to the log, then read from a log-derived view and find that their write has not yet been reflected in the read view. 

# What is a Log?
*A log is perhaps the simplest possible storage abstraction. It is an append-only, totally-ordered sequence of records ordered by time*

<img src="Figure A.png">

Records are appended to the end of the log, and reads proceed left to right. Each entry is assigned a unique Log Sequence Number.
The **ordering** of records defines a notion of "time" since entries to the left are defined to be older. This order decoupling from any particular physical clock will turn to be essential in distributed systems.
Logs have a spcific purpose: they record what happened and when. For **distibuted** systems, this is the very heart of the problem.

### Logs in databases
<p>The usage in databases has to do with keeping in sync the variety of data structures and indexes in the presence of crashes.</p>
<p>To make this atomic and durable, a databases uses a log to <b>write out</b> information about the records they will be modifying, <b>before</b> applying the changes to all the various data structures it maintains. The log is the record of what happened, and each table or index is a projection of this history into some useful data structure or index. Since the log is immediately persisted it is used as the authoriative source in restoring all other persistent structures in the event of a crash.</p>
<p>Over-time the usage of the log grew from an implementation detail of ACID to a method for replicating data between databases. Turns out that the sequence of changes that happened on the database is exactly what is needed to keep a remote replica database in sync.</p>
<p>Logs solve 2 problems - ordering changes and distributing data</p>
<p>The log centric approach to distributed systems imply: <i>If two identical, deterministic processes begin in the same state and get the same inputs in the same order, they will produce the same output and end in the same state.</i></p>
<p><b>Deterministic</b> means that the processing isn't timing dependent and doesn't let any other input influence its results</p>

<p>Hence the purpose of the log here is to squeeze all the non-determinism out of the input stream to ensure that each replica processing this input stays in sync.</p>

<p>Now, the timestamps that index the log act as the clock for the state of the replicas-you can describe each replica by a single number, the timestamp for the maximum log entry it has processed. This timestamp combined with the log uniqiely captures the entire state (syncronization with the leader) of the replica.</p>

<p>Multitude of ways of what to put in the log:
- incoming requests to a service
- state changes the service undergoes in response to request
- the transformation comands it executes</p>

<p><b>Physical logging</b> means logging the contents of each row that is changed.</p>
<p><b>Logical logging</b> means logging the SQL commands that lead to the row that is changed (insert, update, delete statements).</p>

<p>State-Machine Model (active-active) = incoming requests are kept in a log and each replica processes each request</p>
<p>Primary-backup model = elect one replica as the leader and allow this leader to process requests in the order they arrive and log out the changes to its state from processing the requests (like log shipping).</p>

<img src="Figure B.png">

### Tables and Events
Duality: The log is similar to the list of all credits and debits and bank processes; a table is all the current account balances capturing the current (latest) state. This process works in reverse too: if you have a table taking updates, you can record these changes and publish a "changelog" of all the updates. This changelog is exactly what you need to support near real time replicas. *Hence, tables support data at rest and logs capture change.*

### Data Integration
**Data integration is making all the data an organization has available in all its servies and systems.**

The log acts as a buffer that makes data production async from data consumption. This is important for a lot of reasons, but particularly when there are multiple that may consume at different rates Hadoop/realtime. Or a failure where on come back needs to catch up. Consumers can be added and removed with no change in the pipeline.

Pros this guy mentioned while integrating data for the organisation in one place.
- Making the data available in a new processing system (Hadoop) unlocked a lot of possibilities. 
- New computations were possible
- New analysis came from multiple pieces of data that had previously been locked in specialized systems
- Hadoop and Hive data loads fully automatic, so no manual effort needed adding new data sources or handling schema changes
- From trying to build a two way pipeline between each source to isolate each consumer from the source of the data by connecting to a single data repository that would give them access to everything.
- Kafka solved the issue. Keep in mind we are talking about logs not data

<img src="Figure C.png">

Concept of a data warehouse ia clean reportistory to support analysis. Involves periodically extracting data from source databases, transforming to fit in facts and dims - star or snowflake schema, columinar format,  and loading into data warehouse. It is a batch query infrastructure. 
Since  a DWH has to accomodate various sources, the central data team should propose a common API for anyone to interface with as a central point of integration.

Any kind of value-added transformation that can be done in real-tie should be done as post-processing on the raw log feed produced. This would include things like sessionization of event data, or the addition of the derived fields that are of general interest. The original log is still available, but this real-time processing produces a derived log containing augmented data.

*Sessionization is the **act of turning event-based data into sessions, the ordered list of a user's actions in completing a task.** It is widely used in several domains, such as: Web analytics. This is the most common use, where a session is composed of a user's actions during one particular visit to the website.*

### Log Files and Events
The typical approach to activity data in the web industry is to log it out to text files where it can be scrapped into a data warehouse or into HDFS for aggregation and querying. The **problem** with this is the same as with all **batch ETL**: it couples the data flow to the data warehouse's capabilities and processing schedule.

One solution is to use Kafka as the central, multi-subscriber event log, each capturing attributes on types of actions. Systems are decoupled, pulling only the data they need. In a huge scale system, few Kafka tricks are to:
- Partition the logs allows log appends to occur without co-ordintion between shards and allows the throughput of the system to scale linearly with the Kafka cluster size. Plus, partitions are replicated; if a leader failes, one of the replicas takes over. 
- Optimize throughout by batching reads and writes - Kafka guarantees that appends to a particular partition from a single sender will be delivered in the order they are sent - "state replication". Small reads and writes can be grouped together into larger, high-throughput operations by Kafka. Batching occurs from client to server when sending data, writes to dick, replication between servers, data transfer to consumer and in acknowledging commited data. 
- Avoiding needless data copies - Zero Copy Data Transfer

Logs provides buffering to the processes. If processing proceeds inan unsynchronized fashion it is likelt yo happen that an upstream data producing job will produce data more quickly than another downstream job can sonsume it. So Logs acts as a very very large buffer that allows process to be restarted or fail without slowing down other parts of the processing graph. There can't be a faulty job causeing backpressure that halts the entire flow. 

### Stateful Real-Time Processing
### Stream-table duality
KStream and ksqlDB can do the initial stateful processing of data. Ktable can be persisted to preserve state in case of failure. 

Stream data comes in -> events are agregated in table by Key -> logic AND previous state with new state -> If change, publish to change log stream.

### Log Compaction
Keeps value of last state to preserve storage.


# Map, Filter and Collect methods in Java Stream
<img src="Figure L.png">

The **Stream.map(Function mapper)** is a method in Stream class used to **transform one object into another by applying a function.** 
Ex. Convert a *List of String* into a *List of Integer* by applying the *Integer.valueOf()*

The **Stream.filter(Predicate condition) filters elements based upon a condition.**
Ex. Filter a list of numbers outputing only even numbers.

The **Stream.collect(Collectors.toList()) accumulates all even numbers into a List and returns.**

# Indexing Very Large Tables
Whenever we create an index, a copy of the indexed column + the PK are created on disk, and the index is kept in memory as much as possible. If the index has all the data required by a query, it will never go to the actual table. Ex. filter data on *customer_id* and group by *year* and selecting only these two columns, the query won't fetch the disk if the index is on *customer_id, year*. I think memory size = index size.
- More indexes means slower inserts.
- More indexes mean more disk & memory space occupied.
- Modify an existing index instead of adding more indexes. 
- Partitioning splits tables into smaller sub-tables so queries don't have ot performa full scan. 

#### The key, the whole key, nothing but the key, so help me Codd. The benefits of indexing foreign keys
A primary key is a constraint in SQL which acts to uniquely identify each row in a table, a unique index is created. The key can be defined as a single non-NULL column, or a  combination of non-null columns which generate a unique value, and is used to enforce entity integrity for a table. 
2 types of PK: **Natural Key** = primary key from the data source table. **Surrogate Key**: primary key generated by database when data is inserted. 
A foreign key is a column or combination of columns that are the same as the PK, but in a different table. FK enforce integrity between two tables. By default an index is not created for a FK, however it is not uncommon to add manually. 

Table Partitioning
Table partitioning refers to partitioning a large table physically into multiple small table storages in a database, but in logically, it is still a table partitioning method. Partitioning massive data table and storing them on different physical disk database files improve the database I/O operation efficiency.





# More of each system

### Big Data Vs     
- Volumne = size of the data
- Variety = various sources from which data is collected
- Velocity = the frequency at which data is getting generated
- Variability = how different is the data over a period of time     


# Hadoop
- RDBMS can be useful for single files and short data whereas Hadoop is useful for handling Big Data in one shot. Since data is stored in several nodes, it is better to process it in a distributed manner. RDBMS is not efficient when the data is large. Main components of Hadoop are HDFS used to store large databases and MapReduce used to analyze them. HDFS is filing system use to store large data files. It handles streaming data and running clusters on the commodity hardware. Features fault tolerance, high throughput, suitability for handling large data sets, and streaming access to file system data.
- Fault tolerance/Reliable/High Availability = by default, Hadoop creates three replicas for each block at different nodes. Also facilitates automatic recovery of data and node detection. On hardware failure, data can be accessed from a different path.
- Distributed Processing = Hadoop stores data in a distributed manner in HDFS, implying fast data processing. Also uses MapReduce for parallel processing of the data.
- Scalability = Easily add new hardware to the nodes in Hadoop.
### Hadoop Common, HDFS, YARN, MapReduce?
- HC = A set of utilities (Java libs and scripts) that offer support to the below three components.
- HDFS = Distribute storage of big data across cluster of computers. Stores data in the form of small mem blocks and distribute them across the cluster. 
- YARN = (Yet another resource negotiator) Manages the resources on your computing cluster. Decides who gets to run the tasks, when and what nodes are available or not for extra work.
- MapReduce = Mappers and Reducers functions/scripts. Executes M & R tasks in a parallel fashion, distributed as small blocks. 
### HDFS Master node, Data node, daemon
- HDFS replicates data multiple times to ensure availability. It has two daemons: Master Node - NameNode and their slave nodes - DataNode
- NameNode = runs on the master server and and regulates file access by the client. 
- DataaNode = runs on slave nodes and stores the business data.

### NameNode in HDFS, what data is stored in it and how does it communicate with the DataNode
### What is MapReduce? What is "map", "reducer", "combiner"? Compare MapReduce with Spark.
- Combiner = A mini version of a reducer that is used to perform local reduction process. The mapper sends the input as a specific node of the combiner, which then sends the respective output to the reducer.
- Map = handles data splitting and data mapping
- Reduce = handles shuffle and reduction in data
### Limitations
- Only one NameNode
- Suitable only for batch processing of large amount of data
- Only map or reduce jobs can be run by Hadoop
### Block and Block Scanner in HDFS
### Classification and Optimization-based Scheduling for Heterogeneous Hadoop systems
### How does a Block Scanner Handle corrupted files
### Hadoop Streaming and streaming access
- HS = permits any non-Java program that uses standard input and output to be used for map tasks and reduce tasks. 
### Name the XML configuration files present in Hadoop
- mapred-site.xml, core-site.xml, hdfs-site.xml, yarn-site.xml
### File System Check in HDFS
- A command used to check any data inconsistencies
### Some methods/functions of Reducer
### Different usage modes of Hadoop
### How is data security ensured in Hadoop
### Which are the default port numbers for Port Tracker, Task Tracker and NameNode in Hadoop
### What is meant by 'block' in HDFS? Can blocks be broken down by HDFS if a machine does not have the capacity to copy as many blocks as the user wants?
### What is the process of indexing in HDFS
### How is the distance between nodes defined when using Hadoop?
### Rack Awareness
### Heartbeat message
### Use of a Context Object in Hadoop
### Use of Hive in Hadoop and Metastore in Hive
- Hive is a tool in Hadoop, used for processing structured data stored in Hadoop. Developers write Hive queries almost smiilar to SQL statements that are given for analysis and querying data. 
### Components that are available in Hive data model?
### Can more than a single table be created for an individual data file?
### What is the meaning of Skewed tables in Hive?
### Collections present in Hive?
### What is SerDes in Hive?
### What are the table creation functions in Hive
### Role of the .hiverc file in Hive?

### What are the parameters of mappers and reducers?
### What is Shuffling and Sorting in MapReduce?
### What is Partitioner and its usage?
### What is Identity Mapper and Chain Mapper?
### What main configuration parameters are specified in MapReduce?
### Name Job control options specified by MapReduce
### What is InputFormat in Hadoop?
### What is the difference between HDFS block and InputSplit?
### What is JobTracker. Explain job scheduling through JobTracker
### What is SequenceFileInputFormat?
### How to set mappers and reducers for Hadoop Jobs?
### JobConf in MapReduce
### MapREduceCombiner
### RecordReader in a MapReduce
### Define Writable data types in MapReduce
### OutputCommitter
### Key differences between Pig and MapReduce?
### Partitioning
- A process to identify the reducer instance which would be used ot supply the mappers output. Before mapper emits the data (Key Value) pair to reducer, mapper identify the reducer as a recipient of mapper output. All the key, no matter which mapper has generated this, must lie with same reducer. 
### How to set which framework would be used to run mapreduce program?
### Compare HDFS and HBASE
| Criteria | HDFS | HBase |
|----------|------|-------|
|Data Write Process | Append method | Bulk incremental, random write |
|Data Read Process | Table Scan | Table scan/random read/small range scan |
|Hive SQL Query | Excellent | Average |
### Various components of HBase
- Hmaster = managed and coordinates the region server just like NameNode manages DataNode in HDFS
- Region Server = Possible to divide a table into multiple regions and server makes it possible to serve a group of regions to the clients
- Zookeper

### Syntax to run a MapReduce
- hadoop_jar_file.jar /input_path /output_path



# Spark
- In-memory caching computation, optimized query execution and cyclic data flow. Can access diverse data sources. 
- Components = Spark Core Engine, Spark Streaming, GraphX, Spark SQL
- Supports stream processing in real-time
### Explain key features of Spark
### What is Resilient Distributed Dataset and what operations does RDD support? How to create a RDD?
- The partitioned data in an RDD is immutable and distributed. 
- Transformations = produce a new RDD from an existing RDD, every time we apply a transformation to the RDD. It takes an RDD as input and ejects one or more RDD as output. Doesn't execute until an action occures
- Transformation functions = map() = iterates over every line in the RDD and splits into a new RDD. filter() = creates new RDD by selecting elements from the current RDD that pass the argument.
- Actions = used when we wish to use the actual RDD instead of working with a new RDD after we apply transformations. Actions bring back data from RDD to local machine. 
- Action functions = reduce() implements again and again until only one value is left. take() takes all the values from an RDD to the local node. 
- Creating an RDD:
```python
# Method 1
Data = Array(2,4,6,8,0)
RDD = sc.parallelize(Data)
# Method 2 : loading an external dataset
```

### What is RDD Lineage?
- Spark does not support data replication in memory and thus, if any data is lost, it is rebuilt using RDD lineage. 

### What are transformations and actions in Spark?
### What does Spark engine do?
- Responsible for scheduling, distributing and monitoring the data application across the cluster. 
- Run mappings in Hadoop clusters
- SQL Batch and ETL jobs, stream data from sensors
### Define partitions
- Smaller and logical division of data similar to a 'split' in MapReduce. 
- Partitioning is the process of deriving logical units of data to speed up data processing     
<img src="SparkPartition.png"/>

### Define functions of Spark Core
- Memory management, basic I/O functionalities, monitoring jobs,job scheduling, providing fault tolerance, interaction with storage system, distributed task dispatching

### What is Spark Driver?
- runs on the master node and declares transformations and actions on data RDDs.

### Spark Streaming
- Processing of live data
- Data form different sources: Kafka, Flume, Kinesis, HDFS, S3, Hbase, Cassandra is processed and then pushed to: file systems, live dashboards, databases
### Spark SQL, Parquet
- Parquet is a columnar format file where Spqrk SQL performs both read and write operations with Parquet file. 

### File systems does Spark Support
### DAG in Spark
- Not cyclic, unidirectional, only one flow. In this graph, the vertices represent RDDs and the edges represent the operations applied to the RDDs. 
- DAG is a scheduling layer that implements stage-oriented scheduling and converts a plan for logical execution to a physical execution plan.

### Roles of receivers in Spark Streaming?
- Special objects whose only goal is to consume data from different data sources and then move it to Spark. 
- You can create receiver objects by streaming contexts as long-running tasks on various executors.
- Reliable receivers: The receiver acknowledges data sources when data is received and replicated successfully in Spark Storage.
- Unreliable receiver: do not acknowledge data reception. 

### Spark Executor
- When SparkContext connects to Cluster Manager, it acquires an executor on the nodes in that cluster. Executors are Spark processes that run computations and store data on worker nodes. The final tasks by SparkContext are transferred to executors for their execution. A WorkerNode is any node that can run the application code in a cluster.       
<img src="WorkerNode.png"/>

### Types of Cluster Managers in Spark
- Standalone = a basic cluster manager ot set uup a cluster
- Apache Mesos =  running Hadoop MapREduce and other applications
- YARN

### Do you need to install Spark on all the nodes of the YARN cluster while running Spark on YARN?
No, because Spark runs on top of YARN.
### Some demerits of Spark.
Since Spark utilizes more storage space when compared to Hadoop and MapReduce, there might arise certain problems. Developers need to be careful while running their applications of Spark. To resolve the issue, distribute the workload over multiple clusters, instead of running everything on a single node. 
### What are Spark DataFrames?
- Equivalent to data table in RDBMS and DataFrame in Python, but optimized 
for big data.

### What are Spark Datasets
- Data structures in Spark, that provide the JVM object benefits of RDDs (the ability to manipulate data with lambda functions), alongside a Spark SQL-optimized execution engine. 

### What is in-memory processing?
### Lazy evaluation?
- If an RDD is created out of an existing RDD or a data source, the materialization of the RDD will not occur until the RDD needs to be interacted with. This ensures the avoidance of unnecessary memory and CPU usage.        

# KAFKA
### Elements of Kafka
- Topic = It is a bunch of similar kinds of messages
- Producer = Issue communications to the topic
- Consumer = 
- Broker = Where the issued messages are stored

### Partitioning key

### QueueFullException
- Happens when the manufacturer tries to transmit communications at a speed the broker can't handle. 

# SQL
# SQL Server Instance
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
