<!-- # Designing Data Intensive Applications - Notes -->
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
- In order to restore data in the same types, the decoder must be able to instantiate arbitary classes - big security risk.
- Efficiency, time taken to encode & decode is notoriously bad.

Only use language's built-in encoding for transient purposes.

#### JSON, XML & Binary Variants

JSON, XML & CSV are textual formats, and thus relatively human-readable. But they also have problems...
- Ambiguity around the encoding of numbers. XML and CSV cannot distinguish between number and string. JSON does not distinguish ints and floats, or precision. Big problem when dealing with large numbers (>2^53).  
- JSON and XML have good support for unicode, but not for binary strings.
- Optional schema support for XML and JSON. These are powerful but complicated languages. Many JSON based tools don't bother using schemas.
- CSV has no schema, up to the application to define the meaning of each row and column. If an applicaiton change adds a new row or column, you have to handle manually.

These three are good enough for many purposes. As long as people agree on the format, it doesn't matter how pretty or efficient the format is. The difficulty of getting different organisation to agree on anything outweighs most other concerns.

##### Binary encoding

For internal data, there is less pressure to use 'lowest-common-denominator' encoding format. Can choose format that is more compact or fast to parse. Negligible gains in small data sets, but big impact in the terabytes.

JSON and XML use a lot of space compared to binary formats. This led to Binary JSON encodings (MessagePack, BSON, BISON) etc.

#### Thrift and Protocol Buffers

Both Thrift and Protobufs require a schema for any data that is encoded, described like...

```
message Person {
	required string user_name = 1;
	optional int64 favourite_number = 2;
	repeated string interests = 3;
}
```

Both come with a code generation tool that takes a definition, and produces classes that implement the schema in various languages. Your application code uses these to encode or decode records of the schema.

No field names, instead the encoded data uses _field tag_ integers used as an alias.

##### Field tags and schema Evolution

How do Thrift and Protocol Buffers handle schema changes while keeping forward and backward compatibility?

Each field is identified by its tag number, and annotated with its data type. If a field value is not set then it is omitted from the encoded record. Field tags are essential to the meaning of the encoded data. You can change the name of a field, but not a field's tag as that would make all existing encoded data invalid. You can add new fields to the schema, provided you give each field a new tag number.

This maintains forward compatibility, old code can read records written by new code.

As long as each field has a unique tag number, new code can always read old data, as the tag numbers will have the same meaning.

#### Avro

Apache Avro is another binary encoding format, started in 2009 by Hadoop.

### The Merits of Schemas
Schema languages are much simpler than XML Schema or JSON Schema. Protobufs,  Thrift and Avro have grown to support a fairly wide range of programming languages.

Many data systems also implement some kind of proprietary binary encoding. Most relational databases have a network protocol (sql:// ?) over which you can send queries to the database and get responses.

Although textual data formats are widespread, binary encodings based on schemas are a viable option. They have some nice properties:
- More compact than various 'BSON' variants since they can omit field names from the encoded data.
- Schema is a valuable form of documentation. Manually maintained documentation can easily diverge from reality.
- Database of schemas lets you check forward and backward compatibility before deployment.
- For statically typed languages, the generate code from schemas is useful since it allows type checking at compile time.
