# Designing Data Intensive Applications - Notes 

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

### Hash Indexes

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

### SSTables and LSM-Trees

### B-Trees

B-Trees are the standard index implementation in almost all relational databases, and many non-relational databases use them too.

Like SSTables, B-Trees keep key-value pairs sorted by key, which allows efficient key-value lookups and range queries.

B-Trees break the database down into fixed size _blocks_ or _pages_, traditionally 4kb in siz, and read or write one page at a time.

Each page can be identified using an address or location, which allows one page to refer to another, similar to a pointer. 
