# Designing Data Intensive Applications - Notes
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Designing Data Intensive Applications - Notes](#designing-data-intensive-applications-notes)
	- [Chapter 1 - Reliable, Scalable and Maintainable Applications](#chapter-1-reliable-scalable-and-maintainable-applications)
		- [Reliability](#reliability)
		- [Scalability](#scalability)
			- [Approaches for Coping with Load](#approaches-for-coping-with-load)
		- [Maintainability](#maintainability)
			- [Operability](#operability)
			- [Simplicity](#simplicity)
			- [Evolvability](#evolvability)
	- [Chapter 2 - Data Models and Query Languages](#chapter-2-data-models-and-query-languages)
		- [Relational Model vs Document Model](#relational-model-vs-document-model)
		- [The Birth of NoSQL](#the-birth-of-nosql)
		- [The Object-Relational Mismatch](#the-object-relational-mismatch)
		- [Many-to-One and Many-to-Many Relationships](#many-to-one-and-many-to-many-relationships)
		- [Are Document Databases Repeating History?](#are-document-databases-repeating-history)
		- [Relational Versus Document Databases Today](#relational-versus-document-databases-today)
			- [Which has simpler application code?](#which-has-simpler-application-code)
			- [Schema flexibility in the document model](#schema-flexibility-in-the-document-model)
			- [Data locality for queries](#data-locality-for-queries)
			- [Convergence of document and relational dbs](#convergence-of-document-and-relational-dbs)
	- [Chapter 3 - Storage and Retrieval](#chapter-3-storage-and-retrieval)
		- [Data Structures That Power Your Database](#data-structures-that-power-your-database)
			- [Hash Indexes](#hash-indexes)
			- [SSTables and LSM-Trees](#sstables-and-lsm-trees)
			- [B-Trees](#b-trees)
			- [Keeping everything in memory](#keeping-everything-in-memory)
		- [Transaction Processing or Analytics?](#transaction-processing-or-analytics)
			- [Data Warehousing](#data-warehousing)
		- [Stars and Snowflakes: Schemas for Analytics](#stars-and-snowflakes-schemas-for-analytics)
		- [Column-Orientated Storage](#column-orientated-storage)
			- [Column Compression](#column-compression)
		- [Writing to Column-Orientated storage](#writing-to-column-orientated-storage)
	- [Chapter 4 - Encoding and Evolution](#chapter-4-encoding-and-evolution)

<!-- /TOC -->
## Chapter 1 - Reliable, Scalable and Maintainable Applications

Many factors influence the design of a system including:
- skills and experience of people involved
- legacy system dependencies
- timescale of delivery
- organisation's tolerance of risk
- regulatory constraints

### Reliability

- application performs the function that the user expected
- tolerate user mistakes or using the software in unexpected ways
- performance is good enough for the use case, under expected load and data volume
- system prevents unauthorized access and abuse

"continuing to work correctly, even when things go wrong"

fault - a component of the system deviating from spec
failure - the system as a whole stops providing the required service to the user

It can make sense to __increase__ the rate of faults by triggering them deliberately, for example by killing processes randomly without warning e.g. Netflix Chaos Monkey.

Hardware faults, software faults and human faults.

There are situations in which we choose to sacrafice reliability in order to reduce development cost (e.g. developing prototype for an unproven market) or operational cost (e.g. for a service with a very narrow profit margin), but we should be very concious when cutting corners.

### Scalability

The ability of a system to cope with increased load.

It is meaningless to say "X is scalable" or "Y doesn't scale". Rather, discussing scalability means consdiering questions like "if the system grows in a particular way, whow can we add compute resource to handle the additional load?"

Load can be described with a few numbers called __load parameters__, which depends on the architecture of your system e.g. requests per second to a server, the number of simultaneous active users in a chat room. Perhaps the average case matters, or perhaps your bottleneck is dominated by a small numver of extreme cases.

When you increase a load parameter and keep system resources unchanged, how is the performance of the system affected?

When you increase a load parameter, how much do you need to increase the resource if you want to keep performance unchanged?

Batch processing systems like Hadoop care about _throughput_, number of records we can process per second.

Online systems care about _response time_, the time between sending a request and receiving a response. _Response time_ will vary with identical calls, so should be thought of as a distribution of values, not a single number.

Percentiles are usually the best way to describe this distribution: p50, p95, p99, p999.

Amazon observed that a 100ms increase in response time reduces sales by 1%, others report that a 1-second slowdown reduces custoemr satisfaction metric by 16%.

Percentiles are often used in SLAs e.g. service is considered to be up if it has a median response time of less than 200ms and p99 < 1000ms.

When several backend calls are needed to serve a request, it takes just a single slow backend request to slow down the entire end-user request.

#### Approaches for Coping with Load

An architecture appropriate for one level of load is unlikely to cope with 10 times that load, the architecture will be rethought quite often.

vertical scaling (more powerful machine) / horizontal scaling (distributing load across multiple machines)

Good architectures involve a pragmatic mixture of the two approaches, several fairly powerful machines can be simpler and cheaper than a large number of small VMs.

_elastic systems_ add compute when a load increase is detected, whereas others are scaled manually. Manual scaling is simpler and has fewer operational suprises, elastic is good is load is highly unpredictable.

Distributing stateful systems across nodes can increase a lot of complexity, common wisdom says to keep you database on a single node until scaling cost or high availability requirements force you to make it distributed.

Architectures are highly specific to the application - there is no magic scaling sauce.

Architectures that scale well for a particular application are built around assumptions of which operations will be common and which will be rare - the load parameters.

### Maintainability

The majority of the cost of software is in its ongoing maintenance: fixing bugs, investigating failures, repaying tech debt and adding new features.

We can and should design software in such a way that will minimize pain during maintenance, avoiding creatign legacy software.

Three design principles for software systems:
1) Operability - easy for operations teams to keep the system running smoothly
2) Simplicity - easy for new engineers to understand the system
3) Evolvability - easy for engineers to make changes to the system in the future

#### Operability

Typically operations teams are responsbile for the following...
- Monitoring the health of a system and restoring service if it falls into a bad state
- Tracking the cause of problems like system failure or degraded performance
- Keeping software and platforms up to date, security patches
- Keeping tabs on how systems affect each other, so a problematic change can be avoided before it causes damage
- Establishing good practices and tools for deployment, config management
- Defining processes that make operations predictable and help keep production environments stable
- Preserving the organization's knowledge, even as individual people come and go

Good operations makes routine tasks easy, allowing the team to focus on high value tasks. Data systems can do the following to make routine tasks easy:
- Providing visibility into the runtime behaviour, with good monitoring
- Providing good support for automation and integration with standard tools
- Avoiding dependency on individual machines
- Providing good documentation and an easy-to-understand operational model
- Self-healing where appropriate, also giving admins control over system state when needed
- Predictable behaviour, minimizing surprises

#### Simplicity

Small software projects can be very simple and expressive, but as they get larger they often become very complex and difficult to understand. This complexity makes systems difficult to understand, slowing down everyone who needs to work on the system.

Simplicity should be a key goal for the systems we build.

Complexity is accidental if it is not inherent in the problem that the software solves but arises only from the implementation.

Good abstractions can hide a great deal of implementation detail behind a clean, simple-to-umderstand facade.

Finding good abstractions is very hard.

#### Evolvability

Extremely unlikley that your system's requirements will remain unchanged, more likley in a constant state of flux.

Agile working patterns provide a framework for adapting to change. There exist tools and patterns helpful when developing software in a frequently changing environment, such as TDD and refactoring.

The ease with which you can modify a data system and adapt to changing requirements is closely linked to its simplicity and its abstractions.

## Chapter 2 - Data Models and Query Languages

'The limits of my language mean the limits of my world' - Wittgenstein

Data models are how we _think about the problem_ that we are solving

Most applications are built by layering one data model on top of another. The key question is: how is it represented in terms of the next-lower layer?

The basic idea is: each layer hides the complexity of the layers below it by provdigin a clean data model. These abstractions allow different groups of people to work together effectively.

The data model has such a profound effect on what the software above it can do, so it is important to choose one appropriate to the application.

### Relational Model vs Document Model

SQL is the best-know data model today: data is organized into relations (tables in SQL), where each relation is an unordered collection of tuples (rows in SQL).

Roots of relational database lie in business data processing, use cases typically were _transaction processing_ (sales, banking transactions) and _batch processing_ (invoicing, payroll, reporting).

### The Birth of NoSQL

NoSQL is the latest attempt to overthrow the relational model's dominance.

Several driving forces behind the adoption of NoSQL databases including:
- Need for greater scalability than relational databases can easily achieve: large datasets or high write throughput
- Widespread preference for FOSS over commercial database products
- Specialized query operations that are nto well supported by the relational model
- Frustration with the restrictiveness of relational schemas, and a desire for a more dynamic and expressive data model

Different applications have different requirements, and the best choice of one technology for one use case may well be different from the best choice for another use case.

_Polyglot persistence_ relational and nonrelational datastores used in parallel

### The Object-Relational Mismatch

Most application development today is done in OO langauges, which leads to the criticism of the SQL data model: if data is stored in relational tables, and awkward translation layer is required between the objects in the application code and the db model of rows, tables and columns. _Impedance mismatch_.

For a Data Structure like a resume, wich is mostly a self-contained _document_, a JSON representation can be quite appropriate.

JSON representation has better _locality_ than the multi-table schema. To fetch a profile in the relational example, you need to either perform multiple queries or perform messy multi-way join between the users and subordinate tables.

### Many-to-One and Many-to-Many Relationships

If the UI has free-text fields for entering region and industry, it makes sense to store them as plain-text strings, but there are advantages to having standardized lists of geographic regions and industries:
- Consistent style and spelling across profiles
- Avoiding ambiguity
- Ease of updating
- Localization support e.g. when the site is transformed into other langauges
- Better search

Whether you store and ID or a text string is a question of duplication. When using and ID, the human-meaningful information is stored in only one place, and everything refers to it using an ID.

The advantage of an ID is that because it has no meaning to humans, it never needs to change.

Removing duplication is the key idea behind _normalization_ in databases.

### Are Document Databases Repeating History?

NoSQL reopened the debate on how best to represent many-to-many relationships in a database.

IBM's Information Management System used a simple data model called the hierarchical mode, which has remarkable similarities to the JSON model of used by document data-bases. Like document databases, it worked well for one-to-many relatinships but made many-to-many difficult.

A relational table is simple a collection of tuples, and that's it. The query optimizer automatically decides which parts of the query to execute in which order, and which indexes to use.

### Relational Versus Document Databases Today

Document data model pros: schema flexibility, bettwe perfance due to locality and close to data structures used by the application.

Relational data model pros: better support for joins, and many-to-one and many-to-many relationships.

#### Which has simpler application code?

If the data in your application has a document-like structure (tree of one-to-many relationships) then its a good idea to use a document model.

Document model has limitations, you cannot refer directly to a nested item within a document. As long as documents are not too deeply nested, not usually a problem.

Poor support for joins may or may not be a problemm. Many-to-many relationships may never be needed in an analytics applications that uses documents to record which events occurred at which time.

It's possible to reduce the need for joins by denormalizing in a relational model, but then the application needs to do additional work to keep the normalized data consistent.

It is not posssible to say in general which data model leads to simpler application code; it depends on the relationships that exist between data items. For highly connected data,  graph models are the most natural.

#### Schema flexibility in the document model

Most document dbs and JSON support in relational dbs do not enforce schema on the data.

Document dbs are sometimes called _schemaless_, but that is misleading. The code usually assumes some kind of structure that is not enforced by the db. _Schema-on-read_ is more accurate, compared to _schema-on-write_ of relational dbs.

_schema-on-read_ -> dynamic runtime type checking
_schema-on-write_ -> status compile-time type checking

Schema changes have a bad reputation of being slow and requiring downtime. MySQL the worst case.

Schema-on-read approach is advantageous if the items don't all have the same structure. Many different types of objects, not practical to put each type in its own table. Strucutre is determined by external systems over which you have no control, and may change at anytime.

Schemas useful mechanism for documenting and enforcing that structure.

#### Data locality for queries

Document is usuall stored as a single continuous string encoded as JSON, XML or BSON in Mongo.

If application often needs to access the entire document there is a performance advantage to storage locality, having the data split across multiple table can take much longer.

Locality advantage only applies if you need large parts of the document at the same time. DB typically needs to load the entire document even if you only access a small portion, wasteful in large documents.

Locality not just present in the document model. Cassandra and Bigtable have column friendly properties to manage locality.

#### Convergence of document and relational dbs

RethinkDB supports relational-like joins in its query language.

Document and reational dbs becoming more similar over time.

## Chapter 3 - Storage and Retrieval

How does a database store the data we give it, and does it find it again when asked.

In order to tune a storage engine to perform well on your kind of workload, you need to have a rough idea of what the storage engine is doing under the hood.

Big difference between storage engines optimized for transational workloads and those optimized for analytics.

### Data Structures That Power Your Database

Appending to a file is generally very efficient. Many database tables use a _log_, an append-only data file.

Logs have terrible, O(n) performance. To speed this up, we need a different data structure: an _index_. Indexes are additional structures derived from the primary data.

For writes, it's hard to beat the performance of appending to a file. Any kind of index slows down writes, because the index must be updated every time data is written.

Often indexes make the application developer choose indexes, using you knowledge of the typical query patterns.

#### Hash Indexes

Let's say our storage consists of appending to a file, and we want to create an index with a hash map.

Create an index by maintaining an in-memory hash-map where every key is mapping to a byte offset in the data file. Though simplistic, this is how Bitcask works.

This engine is well suited to situations where the value for each key is updated frequently. There are lots of writes, but not too many distinct keys.

How do we eventually avoid running out of disk space? Compaction. Throwing away duplicate keys in the log, keeping only the most recent update.

Other considerations? File format: binary preferred to csv, Deleting records, Crash recovery, Partially written records: checksums, Concurrency control.

Append only logs seem wasteful at first, but append-only is good for a number of reasons:
- Appending and segment merging are sequential operations, much faster than random writes.
- Concurrency and crash recovery are much simpler if segment files are append-only or immutable.

Hash table index also has limitations:
- Hash table must fit in memory. Could maintain on disk but super painful and slow.
- Range queries are not efficient, looking for sequences requires looking up every key.

#### SSTables and LSM-Trees

#### B-Trees

B-Trees are the standard index implementation in almost all relational databases, and many non-relational databases use them too.

Like SSTables, B-Trees keep key-value pairs sorted by key, which allows efficient key-value lookups and range queries.

B-Trees break the database down into fixed size _blocks_ or _pages_, traditionally 4kb in siz, and read or write one page at a time.

Each page can be identified using an address or location, which allows one page to refer to another, similar to a pointer.

The algorithm ensures that the tree remains _balanced_, n keys always has depth O(log n).

To make databases reliant to crashes, it is common for B-tree implementations to include _write ahead logs (WAL)_. Append-only file to which every B-Tree modification must be written before being applied to tree itself.

#### Keeping everything in memory

As RAM becomes cheaper, the cost-per-GB argument is eroded. Many datasets are simply not that big so it is feasible to keep everything in memory.

Memcached is an in-memory key-value store intended for caching use only, where it is ok for data to be lost if a machine is restarted.

When an in-memory DB is restarted, needs to reload state either from disk or over the network. Disk is merely used as an append-only log for durability and all reads are served entirely from memory.

Performance advantage of in-memory DB is not because they don't need to read from disk. Faster because they avoid the overheads of encoding in-memory data structures in a form that can be written from disk.

In-memory DBs can provide data models that are difficult to implement with disk-based indexes.

Research indicates in-memory DB can support datasets larger than RAM through _anti-caching_, evicting least recently used data to disk, and loading it back again when required. Analogous to operating systems swapping virtual memory in pages.

### Transaction Processing or Analytics?

Online transaction processing (OLTP).

Data analytics has very different _access patterns_ to transactional processing. Scans huge number of records, and calculates aggregate statistics, rather than returning raw data.

Property | OLTP | OLAP
--- | --- | ---
Read pattern | Small number of records per query, fetched by key | Aggregate over large number of records
Write pattern | Random-access low latency writes from user input | Bulk import (ETL) or event stream
Primary user | End-user / customer via web application | Internal analyst, for decision support
What data represents | Latest state of data | History of events over time
Size | GB/TB | TB/PB

#### Data Warehousing

OLTP systems expected to be highly available and low latency, critical to business operation. DBAs reluctant to let analysts run large ad hoc queries on OLTP DB.

_Data warehouse_ is a separate DB that analysts can query without affecting OLTP operations. Optimised for analytic access patterns.

On the surface, data warehouse and relational OLTP DB can look very similar (SQL interface), but the internals can be very different.

### Stars and Snowflakes: Schemas for Analytics

Data warehouses are used in a formulaic style know as a _star schema_.

Centre of the schema is the _fact table_, which represents an event that occurred at a particular time e.g. customer's purchase of a product, or click by a user.

TODO: p94 Example of a star schema image

### Column-Orientated Storage

If you have trillions of rows and petabytes of data in your fact tables, storing and querying them efficiently becomes challenging.

OLTP DBs and Document DBs are laid out in a similar fashion, each row or document is stored as one contiguous sequence of bytes.

Column orientated storage is simple, store all the values from each column together instead. Then you only need to read and parse thhose columns that are used in that query.

#### Column Compression

Column orientated storage leads itself well to compression.

_Bitmap encoding_ is particularly effective in data warehouses.\

### Writing to Column-Orientated storage

Column-orientated storage makes sense in analytical settings as the load consists of large read-only queries. However, they have the downside of making writes more difficult.

Update in place approach like B-Trees is not possible with compresed columns. Most likely have to write all the column files if you wanted to insert a row in the middle of a sorted table.

LSM Trees are a good solution. When enough writes have accumulated in memory, they are combined with an existing column files on disk and written to new files in bulk.

_Data cube_ is a common special case of a materialized view, it is s grid of aggregates grouped 	by different dimensions.

## Chapter 4 - Encoding and Evolution

Chapter 1 introduced the idea of _evolvability_: we should aim to build systems that make it easy to adapt to change.

Relational schemas assume all data conforms to one schema, which can be changed. Schema-on-read (document) databases don't enforce a schema and can contain a mixture of older and newer format.

When formats change, this normally requires a corresponding change in the application code. In large applications, this cannot usually happen instantaneously:
- Server side application, you may want to perform a _rolling upgrade_, deployign to a few nodes at a time, checking that things are running smoothly.
- Client side you are at the mercy of the user.

New and old version of the code may potentially co-exist at once. For this to work, we need to maintain copatibility in both directions:
- _Backward compatibility_: Newer code can read data written by older code. Not hard to achieve as you know the format of data.
- _Forward compatibility_: Older code can read data written by newer code. Harder to achieve, requries older code to ignore addition made by newer version.

### Formats for Encoding Data

Programs usually work with two different representations:
- In memory, data is kept in objects, structs, lists arrays etc. Optimised for efficient acces and manipulation by the CPU.
- When you want to store or send it over the network, you have to encode it as some kind of self-contained sequence of bytes (e.g. JSON).

Translating between representations is called _encoding_, _serialization_ or _marshalling_; and the reverse is called _decoding_, _parsing_, _deserialization_ or _unmarshalling_.

#### Language-specific Formats

Languages come with built-in support for encoding in-memory objects. Python has pickle etc. However, these have deep problems:
- Encoding is tied to a programming language. If you store or transmit the data, you are committing yourself to your programming language for a long time.

### Modes of Dataflow

#### Dataflow Through databases

The process that writes to the database encodes the data and the process that reads from the database decodes it.

Common for several different processes to be accessing a database at the same time. In this environment it is likley that some processes accessing the database will be running newer code, and some older.

Encoding formats can support preservation of unknown fields, but you need to take care at an application level.

##### Different values written at different times

Databases generally let you update any value at any time. You may entirely replace the old application with a new version, but the underlying data is still years old.

_data outlives code_

Migrating data to a new schema is certainly possible, though expensive to do on a large dataset. Most databases allow simple schema changes like adding a column, and fills NULL values in those that are missing.

LinkedIn's document database Espresso uses Avro for storage, allowing it to use Avro's schema evolution rules.

#### Dataflow Through Services: REST and RPC

When you need to communicate across a network, the most common arrangement has two roles: _client_ and _server_.

Web browsers are a type of client, and make AJAX requests to the service. Although HTTP is used as the transport protocol, the API implementation on top is application-specific, and the client and server need to agree on the details of the API.

The server itself is likely a client to another service. This approach is used to decompose a larger application into smaller services: _micro-services architecture_.

Key design goal of a micro-service architecture is to make the application easier to upgrade and maintain by making services independently deployable and evolvable. Expect older and newer versions to be running at the same time.


##### Web Services

Two popular approaches to web services: REST and SOAP.

REST is a design philosophy, not a protocol. Emphasises simple data formats, using URLs for identifying resources and HTTP features for cache control, authentication, and content type negotiation.

SOAP is an XML-based protocl for making network API requests. Deliberately independent from HTTP, and avoids using HTTP features.

SOAP API is described using an XML-based language called Web Services Description Language or WSDL. WSDL is not human readable, and users rely heavily on tool support, code generation, and IDEs.

##### The problems with remote procedure calls (RPCs)

Web services are merely the latest incarnation of technologies to make API requests across a network, many of which have serious problems.

Variants on the idea of a _remote proceedure call_ (RPC). RPC model tries to make a request to a remote network service look the same as calling a function, within the same process (location transparency). Although it appears convenient at first, the approach is fundamentally flawed.

Network request is very different from a local function call:
- Local function call is predictable and either succeeds or fails, depending on parameters under your control. Network request is unpredictable: request or response may be lost, remote machine may be slow or unavailable.

- Local function call either returns a result, or throws and exception, or never returns. A network request has an additional state, it may return without a result due to a _timeout_, meaning you don't know what happened.

- If you retry a failed network request, it could be that the requests are getting through but the results are getting lost. Retrying will case the action to be performed multiple times, unless you build a system for deduplication (idempotence).

- Local function calls take roughly the same time to execute, network calls are much slower and wildly variable

- Local function calls can take pointers to local memory addresses. When making a network request, the params need to be encoded into a byte sequence. Quickly becomes problematic with larger complex objects.

- Client and service likely to be implemented in different programming languages, so the RPC framework must translate datatypes. This can be ugly, since not all languages have the same types.

No point trying to make a remote service look too much like a local object in your programming language, as it is fundamentally different.

##### Current directions of RPC

RPC still proves popular today, new frameworks have been built on encodings mentioned above. Thrift and Avro have RPC support, gRPC is an RPC implementation on protobufs, Finagle uses Thrift, and Rest.li uses JSON over HTTP.

New generation of RPC frameworks are more explicit about the network request layer.

gRPC supports streams, where one call does not necessarily return one response, but a series of responses over time.

Frameworks also support _service discovery_, allowing a client to find out which IP address and port number they can find a service.

Custom RPC protocols with binary encoding can achieve better performance than something generic like JSON over REST. However, RESTful APIs are great for experimentation and debugging.

##### Data encoding and evolution of RPC

For evolvability, it is important that RPC clients and servers can be changed independently.

We can make the resonable assumption that all the servers will be updated first, and the client second. Thus you only need backward compatibility on requests and forward compatibility on responses.

Service compatibility made harder by the fact that RPC often used for communication across organizational boundaries, so the provider of a service has no control over its clients and cannot force them to upgrade.

Thus, compatibility often needs to be maintained for a long time.

#### Message-Passing Dataflow

_Asynchronous message-passing systems_ are somewhere between RPC and databases. The message is not sent via a direct network connection, but goes via an intermediary called a _message broker_ / _message queue_ / _message-orientated middleware_ which stores the message temporarily.

Advantages compared to direct RPC:
- Can act as a buffer if the recipient is unavailable or overloaded, and thus can improve reliability.
- Automatically re-delivers messages to a process that has crashed, and prevents messages from getting lost.
- Avoids the sender needing to know the IP address and port numver of the recipient.
- One message can be sent to several recipients.
- Logically decouples the sender from recipient.

Message-passing communication is usually one-way: sender normally doesn't expect to receive a reply, responses are done via a seperate channel.

##### Message brokers

Open source implementations include: RabbitMQ & Apache Kafka.

In general, the process is: one process sends a message to a named _queue_ or _topic_, and the broker ensures that the message is delivered to one or more _consumers_ / _subscribers_ to that queue or topic. Many to many relationship.

##### Distributed actor frameworks

The _actor_ model is a programming model for concurrency is a single process. Rather than dealing with threads directly (race conditions, deadlock, locking), logic is encapsulated in _actors_. Each actor represents one client or entity, and communicates with other actors by sending and receiving messages.

In _distributed actor frameworks_ this model is used to scale applications across multiple nodes. Location transparency (?) works better in the actor model than in RPC, as the actor model assumes that messages will be lost, even within a single process.

Akka, Orleans Erlang OTP are all _distributed actor frameworks_.
