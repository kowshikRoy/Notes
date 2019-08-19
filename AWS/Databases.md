# Amazon Databases

### Amazon Aurora

>   * MySQL and PostgreSQL compatible relational database engine
>   * 5 times faster than MySQL, 3 times faster than PostgreSQL
>   * distributed, fault-tolerant, self-healing storage system that auto-scales up to 64TB per database instance
>   * 15 low-latency read replicas, point-intime recovery, continuous backup to Amazon S3, and replication across three Availability Zones

### Amazon RDS
>* Relational Database service provided by Amazon
>* Several instance types:
>    * Amazon Aurora, MySQL, PostgreSQL, MariaDB, Oracle, SQL Server



### Amazon DynamoDB
>   *  key-value and document database
>   * single digit millisecond performance at any scale
>   * 10 trillion request per day, support peak of 20M RPS
>   * Wide adoption ( Lyft, Airbnb, Samsung)


### Amazon Elastic Cache
>   *  easy to deploy, operate,and scale an in-memory cache in the cloud
>   * improves the performance of web applications by allowing you to retrieve information from fast, managed, in-memory caches, instead of relying entirely on slower diskbased databases
>   * Support two open-source caching engine
>       - Redis
>           *  fast, open source, in-memory data store and cache
>           *  single- node and up to 15-shard clusters are available, enabling scalability to up to 3.55 TiB of in-memory data
>       - Memcached
>           * widely adopted memory object caching system

### Amazon Neptune
>   *  fast, reliable, fully-managed graph database service
>   * easy to build and run applications that work with highly connected
datasets
>   * The core of Amazon Neptune is a purpose-built, high-performance
graph database engine optimized for storing billions of relationships and
querying the graph with milliseconds latency


### Amazon Quantum Ledger Database (QLDB)
>   * fully managed ledger database that provides a transparent, immutable, and cryptographically verifiable transaction log owned by a central trusted authority
>   * With QLDB, your data’s change history is immutable – it cannot be altered or deleted – and using cryptography, you can easily verify that there have been no unintended modifications to your application’s data
>   *  QLDB uses an immutable transactional log, known as a journal, that tracks each application data change and maintains a complete and verifiable history of changes over time
>   * Serverless, scale as you need, pay what you use

### Amazon TimeStream
>   * fast, scalable, fully managed time series database
service
>   * store and analyze trillions of events per day at 1/10th the cost of relational databases


**Time-series data**
>   * typically arriving in time order form
>   * data is append-only
>   * queries are always over a time interval





