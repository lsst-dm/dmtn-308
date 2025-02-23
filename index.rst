####################################################################
Database Platform Comparison for the Prompt Products Database (PPDB)
####################################################################

.. abstract::

   This note provides a comparison of database platforms for implementing the Prompt Products Database (PPDB). Requirements are described in detail, followed by a breakdown of the capabilities of each database platform for each requirement. Finally, recommendations are provided based on the comparison.

Introduction
============

The Prompt Products Database (PPDB) will provide user access to prompt data products, which are produced as a result of nightly processing by the Alert Production (AP) pipeline.
The specifics of these catalogs, including the conceptual schemas, are covered in Section 3 of the Data Products Definition Document :cite:`LSE-163`.
This does not include not include images or other raw data products that must be accessed using other services such as the `Data Butler <https://arxiv.org/abs/2206.14941>`_.
Several tech notes have been written on aspects of the PPDB, including *DMTN-113* :cite:`DMTN-113`, *DMTN-268* :cite:`DMTN-268`, and *DMTN-293* :cite:`DMTN-293`.
These have discussed performance of an early PostgreSQL-based PPDB implementation, data ingestion, and system architecture, respectively.
However, there is no definitive set of requirements for the system, detailed design document, or implementation plan.
The database platform which should be used to implement the PPDB is also yet to be determined, and this note provides a comparison of various alternatives, concluding with recommendations on which seem to be most suitable.

Requirements
============

Deployment
----------

Two basic options exist for deploying the PPDB: on-premises or in the cloud.
An on-premises solution would be deployed at the `US Data Facility <https://usdf-rsp.slac.stanford.edu/>`_ (USDF) at SLAC, where the Alert Production Database (APDB) is currently hosted.
Cloud deployments would use `Google Cloud Platform <https://cloud.google.com/>`_ (GCP), as Google is Rubin's contracted service provider.
For this reason, database services on other cloud platforms such as Amazon Web Services (AWS) or Microsoft Azure are not considered.
The choice of deployment platform will have a significant impact on the overall cost and complexity of the system.

There are a number of important trade-offs to consider when choosing between on-premises and cloud deployments.
Cloud solutions generally have much better elasticity and scalability, often through features which are built into the services.
However, this flexibility can result in higher operating costs as resources are provisioned and de-provisioned on the fly.
Massively scalable cloud-native platforms like BigQuery offer nearly unlimited scalability, but this feature may result in similarly unbounded and unpredictable financial costs.
On-premises deployments do not generally offer this level of flexibility because of infrastructure and purchasing limitations, especially when hardware is specifically provisioned for a given application; these clusters typically do not include enough spare capacity to scale out in an unlimited fashion as might a cloud-native platform.
If on-premises systems are designed to handle maximum load, then the classic problem of overprovisioning can occur, in which resources are underutilized during normal operation.
On the other hand, on-premises deployments will generally have higher cost predictability, as once dedicated hardware is paid for and installed, operating costs would presumably be covered by existing infrastructure and budget.

Data Volume & Retention
-----------------------

The Alert Production Database (APDB) is designed to retain data within a 1-year window, which is described under bullets 6 and 9 in section 3.2.1 of the *Data Products Definition Document* :cite:`LSE-163`.
The PPDB would ideally retain data for the lifetime of the project, which is currently planned at 10 years of survey operations.
Based on scheduling considerations of Data Release Processing (DRP) and the resultant availability of data products, a data retention of 2 years will be considered as a minimum requirement, with a goal of 10 years.

.. list-table:: Data Volume Projections
   :header-rows: 1

   * - **Visit**
     - 190 MB
   * - **Nightly**
     - 190 GB
   * - **Yearly**
     - 70 TB
   * - **10 Years**
     - 700 TB

The above table provides estimated data volumes for the PPDB across various time periods.
The exact size of the nightly data products which will be produced by LSSTCam is undetermined but can be roughly estimated based on the size of ComCam data products.
Data taking during the ComCam On-Sky Campaign :cite:`SITCOMTN-149` resulted in an average size per visit of approximately 9 MB with 9 active detectors.
Extrapolating to LSSTCam with 189 detectors results in an estimated single visit size of *189/9 * 9 MB = ~190 MB*.
Since the survey is expected to produce approximately 1000 visits per night, this results in a predicted nightly data volume of around 190 GB.

These figures are almost certainly underestimated, because ComCam data processing resulted in sparse data products containing many `null` values.
Actual LSSTCam data products will likely be denser, and this density is expected to increase over time as pipeline algorithms are improved and more columns are filled.
Additionally, `Solar System Processing <https://dp0-3.lsst.io/data-products-dp0-3/solar-system-processing-pipeline.html>`_ (SSP) was not included in AP pipeline processing runs during ComCam operations, and this would be expected to increase data volumes by an unknown factor.

The numbers presented here also represent a significant increase over previously predicted values, such as those used in *DMTN-135* :cite:`DMTN-135`, which estimated a yearly data volume of 24 TB for the APDB.
(This was before the APDB and PPDB were split apart as separate projects with the PPDB designed to handle user queries of prompt data products.)
These estimates were derived from *DMTN-113* :cite:`DMTN-113`, but the data schemas and processing algorithms have changed significantly since that time, and the new estimates are based on more recent data and processing runs.

Query Performance & Latency
---------------------------

Query performance, including the latency of returning results to a client, is a complex and multi-dimensional topic, depending on a multitude of physical factors such as hardware configuration, network latency, memory, and disk I/O.
Additionally, query complexity and the number of concurrent queries affecting the system load can have a significant impact.
The primary consideration in evaluating query performance and latency will be whether or not a given database platform can potentially meet the needs of the use case.

Query performance requirements for the PPDB are covered by *DMS-REQ-0355* in the *Data Management System Requirements* :cite:`LSE-61`.
These specify that the minimum number of simultaneous users should be 20, and that the maximum query time should be 10 seconds.
Given the expected data volumes, longer queries may be necessary to extract the desired information from the system, so the latter requirement may not be satisfiable in all cases.
The PPDB is expected to be used by a large number of users, and this may vary considerably depending on the time of day, the phase of the project, and other factors.

Scalability
-----------

Scalability is a multi-dimensional metric, including the ability to scale out horizontally to handle large data volumes and high query loads.
While specific aspects of scalability are also covered by other requirements, it is worth discussing and characterizing the overall scalability of each database platform.
The system should be able to handle the expected data volume and query load with as little latency as possible.
Ideally, system resoures could be reclaimed or provisioned on-the-fly to meet demand, but this is not a strict requirement.

Total Cost of Ownership (TCO)
-----------------------------

Total cost of ownership may include operating expenses, such as those from storage, compute, and networking, as well as capital expenditure on hardware purchases for on-premises deployments.
Development and maintenance costs in terms of personnel time are not specifically quantified but could vary significantly depending on the platform chosen and may be non-negligible.
Hardware purchase costs are considered and discussed for on-premises deployments, but specific dollar amounts are not provided.
For on-premises deployment, it is assumed that cooling, power, and networking are already covered by existing infrastructure and budget.
Cloud deployments will include some discussion of billing from operating costs, but specific dollar amounts are also not provided.
An attempt will be made to characterize the relative costs of each platform rather than provide specific dollar amounts.

Cost Predictability
-------------------

As a general rule, cloud deployments are less predictable in terms of operating costs than for on-premises.
The cost of running a database on the cloud can vary depending on the amount of data stored, the number of queries run, and the amount of data transferred.
On-premises deployments would likely incur fixed costs that could be calculated accurately in-advance, e.g., hardware purchases.
It is assumed that the operating costs of running the database on-premises at the USDF would be covered by existing infrastructure and budget.

Maintenance Overhead
--------------------

Large, distributed databases can require a significant amount of administrative effort to keep them running smoothly and efficiently.
This typically includes monitoring, backup and recovery, and periodic maintenance operations such as storage vacuuming and index rebuilding.
On-premises deployments would require personnel to manage the low-level infrastructure, while at least some of this burden is shifted to the provider in a cloud deployment.
Maintenance and development efforts may overlap significantly, especially in the early stages of building out the platform.

Database migrations, upgrades, and backups are a particularly important aspect of maintenance and can be complicated and time-consuming for large databases.
For instance, adding columns which calculate new values based on existing data can be quite time-consuming, requiring extensive downtime.
Backups and restoration may be complicated by data being distributed across multiple nodes, which can complicate these operations significantly.
Finally, schema migrations on large, distributed databases can be complex and time-consuming, often requiring significant planning and testing to ensure that they are performed correctly.

Developer Effort
----------------

Significant development effort on software enhancements may be required, depending on the database platform, including, but not necessarily limited to development of the database schema, data ingestion tools, TAP service, deployment code and monitoring tools.
The TAP service and data ingestion are discussed under their own requirements, as these are both potentially significant development efforts in and of themselves.
Additionally, some options may require more effort in developer operations (devops) or "configuration as code," especially for on-premises solutions.
Given the time constraints and the need to have the PPDB operational in a timely manner, the amount of developer effort required to implement the system is a significant factor in the decision of which platform to use.

TAP Service
-----------

User access to the PPDB will be provided by an `IVOA TAP service <https://www.ivoa.net/documents/TAP/>`_ through the Rubin Science Platform (RSP), allowing users to query the database using `Astronomical Data Query Language <https://www.ivoa.net/documents/ADQL/>`_ (ADQL).
The availability of a compatible TAP service will be a significant factor in the decision of which platform to use.
Some of the database platforms do not have a compatible TAP implementation and may require significant effort to either develop a new implementation or adapt an existing one.
The `CADC TAP service <https://github.com/opencadc/tap>`_ runs on top of PostgreSQL and has been used for some existing Rubin services.
PostgreSQL compatibility of the potential platform is a significant advantage in this regard.

The TAP service must support spherical geometry operations, which are used in ADQL queries.
For PostgreSQL databases, this is currently provided by the `PgSphere extension <https://pgsphere.github.io/>`_.
When using non-spherical spatial indexing, such as that provided by `PostGIS <https://postgis.net/>`_, it is typically necessary to apply a "cut" to the data returned by the spatial index in order to ensure that only the correct values are returned.
Implementing these operations can be non-trivial and may require significant development effort to implement correctly and test thoroughly, if this type of spatial indexing is used rather than spherical geometry and a suitable adapter does not exist.

Data Ingestion
--------------

The PPDB will ingest data from the APDB on a nightly basis and must make this data available for user querying within 24 hours, as described in section 3.3 of the *Data Products Definition Document* :cite:`LSE-163`.
The data ingestion is currently implemented as a long-running "daemon" process which writes Parquet files to disk from the APDB and then copies them over the network to a target PostgreSQL database using the `COPY` command.
Of primary consideration is whether a given platform can support the existing data ingestion tools, and, if not, what additional development effort would be required in order to implement the required functionality.
The potential performance of data ingestion is difficult to estimate if there is not an existing solution which can be tested and benchmarked, so this is not used as a point of comparison.

Ecosystem and Community
-----------------------

The ecosystem and community around the database platform are important factors to consider.
This includes availability of documentation, tutorials, and support forums, as well as the number of developers and companies using the platform.
A large ecosystem and community can provide valuable resources and support for developers, as well as a wide range of tools and libraries that can be used to extend the functionality of the database platform.
Similarly, a lack of timely support can make it difficult to find solutions to problems or to get help when needed.

Database Platforms
==================

Given the requirements outlined above, the following database platforms are initially considered:

PostgreSQL
----------

PostgreSQL is the current database platform that has been used for development and testing of the PPDB at USDF, typically running in a Kubernetes cluster in single server mode.
The `dax_ppdb <https://github.com/lsst/dax_ppdb>`_ repository contains command-line tools and APIs for creating the database schema in PostgreSQL from its `Felis representation <https://github.com/lsst/sdm_schemas/blob/main/python/lsst/sdm_schemas/schemas/apdb.yaml>`_, as well as ingesting data into a target PostgreSQL database from the APDB.

Citus
-----

`Citus <https://www.citusdata.com/>`_ is an open source extension that transforms PostgreSQL into a distributed database.
Citus uses a controller-worker model to distribute data across multiple nodes, allowing for horizontal scaling of both storage and compute.

Qserv
-----

`Qserv <https://qserv.lsst.io/>`_ was developed to host the astronomical catalogs for the LSST Data Management System.
It is a distributed database that is designed to scale out horizontally across multiple nodes.
Qserv will be used to host the Data Release (DR) catalogs and has hosted, and will continue to host, Data Preview (DP) catalogs.

AlloyDB
-------

`AlloyDB <https://cloud.google.com/products/alloydb>`_ is a distributed database that is compatible with PostgreSQL.
Though it has an on-premises version, it is primarily designed to run on GCP.
It is typically configured using a primary and replica setup, with the primary node handling writes and the replica nodes handling reads.

BigQuery
--------

`BigQuery <https://cloud.google.com/bigquery>`_ is a fully managed, serverless data warehouse designed for unlimited horizontal scalability.
It can handle huge data volumes and is optimized for fast response of analytical queries on massive datasets.

Platform Comparison
===================

The following table provides a comparison of the database platforms based on the above requirements.

.. Color coding
.. role:: red
.. role:: green
.. role:: yellow

.. list-table:: Platform Comparison Matrix
   :header-rows: 1

   * -
     - **PostgreSQL**
     - **Citus**
     - **Qserv**
     - **AlloyDB**
     - **BigQuery**

   * - **Deployment**
     - USDF
     - USDF
     - USDF
     - GCP
     - GCP

   * - **Data Volume & Retention**
     - :red:`No`
     - :green:`Yes`
     - :green:`Yes`
     - :red:`No`
     - :green:`Yes`

   * - **Query Performance**
     - :red:`Low`
     - :green:`High`
     - :green:`High`
     - :green:`Medium`
     - :green:`Very High`

   * - **Query Latency**
     - :green:`Low`
     - :green:`Low to Medium`
     - :yellow:`Medium`
     - :green:`Low`
     - :green:`Low`

   * - **Scalability**
     - :red:`Low`
     - :green:`High`
     - :green:`High`
     - :yellow:`Medium`
     - :green:`Very High`

   * - **TCO**
     - :green:`Low`
     - :green:`Medium`
     - :green:`Medium`
     - :yellow:`Medium`
     - :red:`High`

   * - **Cost Predictability**
     - :green:`High`
     - :green:`High`
     - :green:`High`
     - :yellow:`Medium`
     - :red:`Low`

   * - **Maintenance Overhead**
     - :yellow:`Medium`
     - :red:`Very High`
     - :red:`High`
     - :yellow:`Medium`
     - :green:`Low`

   * - **Developer Effort**
     - :green:`Low`
     - :red:`High`
     - :red:`Very High`
     - :yellow:`Medium`
     - :red:`High`

   * - **TAP Service**
     - :green:`Fully Supported`
     - :green:`Fully Supported`
     - :green:`Fully Supported`
     - :red:`Not Supported`
     - :yellow:`In Progress`

   * - **Data Ingestion**
     - :green:`Fully Supported`
     - :green:`Fully Supported`
     - :red:`Not Supported`
     - :green:`Fully Supported`
     - :red:`Not Supported`

   * - **Ecosystem and Community**
     - :green:`Excellent`
     - :yellow:`Somewhat Limited`
     - :red:`Very Limited`
     - :green:`Good`
     - :green:`Excellent`

Deployment
----------

We assume that PostgreSQL, Citus, and Qserv would all run on-premises at the USDF.
AlloyDB and BigQuery are cloud-native platforms that run on GCP.
While the on-premises solutions could technically be deployed on the cloud, we do not consider those scenarios here.
AlloyDB has an on-premises option, but we also do not consider this.
Finally, BigQuery is cloud-native with no on-premises option.

PostgreSQL
~~~~~~~~~~

- PostgreSQL can be deployed on-premises at the USDF, where it is currently already being used for development and testing of the PPDB.
- `CloudNativePG <https://cloudnative-pg.io/>`_ has been used at the USDF to deploy PostgreSQL on Kubernetes, including existing PostgreSQL instances used for PPDB development.
   - This provides a suite of tools for managing PostgreSQL on Kubernetes, including monitoring, backup and recovery, and scaling.
- Maintenance and administration of PostgreSQL seems to be well-understood and managed at the USDF, with a dedicated team of system administrators who manage the infrastructure.

Citus
~~~~~

- Citus would be deployed on-premises at the USDF.
   - Dedicated nodes would be required for the controller and worker nodes, with the controller node acting as a proxy for the worker nodes.
   - Sufficient memory, CPU, and storage would need to be provisioned for each node to meet the expected data volume and query load.
- Hardware requirements would need to be carefully considered, as Citus is a distributed database requiring sufficient, provisioned resources to handle system load.
   - Assuming the need to service 20 simultaneous users and therefore concurrent connections, as well as server overhead, a reasonable estimate for the number of vCPUs per worker would be 24. PostgreSQL forks a new process for every connection, so this would be approximately a minimum amount of compute for satisfying the requirement.
   - Given that 16-core machines with hyperthreading are relatively common, this configuration is achievable on commodity hardware. But Kubernetes configuration would be required for ensuring that the Citus controller and worker nodes were distributed across multiple physical machines, did not run on the same physical machine, and had sufficient memory and disk I/O to meet the requirements of the PPDB.
   - While SLAC has a large computing cluster dedicated to USDF, it is generally shared amongst many different services and projects, so it is not clear that the necessary resources would be available to deploy Citus on-premises without additional hardware allocation.
   - The required hardware would need to be purchased and provisioned, which could be a significant limiting factor in deploying Citus on-premises, especially since the scale could be quite large.
- No standard Kubernetes operators or Helm charts seem to exist for Citus, or at least none are listed on the `Citus website <https://www.citusdata.com/>`_. These would need to be developed or found and adapted in order to deploy Citus at the USDF on Kubernetes.


Qserv
~~~~~

- Qserv is already deployed on-premises at the USDF.
- PPDB could be deployed on the same infrastructure as Qserv, and the same team of system administrators could manage both services.
- However, given that Qserv itself has complex provisioning requirements that are an ongoing challenge to satisfy, an additional requirement of hosting the PPDB on the same infrastructure would be a significant burden.

AlloyDB
~~~~~~~

- AlloyDB is a cloud-native platform that runs on GCP.
- Tools are provided for monitoring, backup and recovery, and scaling through a web interface.
- Deployment on GCP would require provisioning of resources through the GCP console or API, and configuration of the database through the web interface.

BigQuery
~~~~~~~~

- BigQuery is a cloud-native platform that runs on GCP.
- It is a fully managed, serverless data warehouse that requires no infrastructure management.
- Some database operations still need to be configured and managed, such as backups.

Data Volume & Retention
-----------------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL has been used to store PPDB data at the USDF but not at the expected data volumes. At most, a few months of data have been stored, though there is an ongoing system test to generate and store a years worth of data.
- The PostgreSQL database engine running in a single server mode has a number of inherent limitations which would prevent it from effectively scaling to the required data volumes of any retention scenario currently being considered.
   - According to its `published limits <https://www.postgresql.org/docs/current/limits.html>`_ , PostgreSQL has a maximum table size of 32 TB, which given an estimated data volume of 70 TB per year, would be exceeded in the first few years of operations.
   - Though theoretically supporting unlimited database size with table partitioning, practical constraints such as query performance degradation, index management overhead, and maintenance tasks (e.g., vacuum and analyze) make the platform impractical for datasets with a magnitude of hundreds of terabytes. Practical experience suggests that the system would actually degrade significantly before coming close to the theoretical limits.
   - Backup and restore operations for large datasets (e.g., > 100 TB) become increasingly time-consuming and operationally challenging and may actually become technically infeasible at a certain point.
   - Vertical scaling of PostgreSQL is limited by hardware constraints, such as I/O, memory, and CPU, which can be a bottleneck for large datasets.
- Overall, given these constraints and limitations, **a single PostgreSQL instance cannot scale to the data volume requirements under any retention scenario which is being considered.**

Citus
~~~~~

- Citus addresses the shortcomings of PostgreSQL in scaling to large data volumes by distributing data across multiple nodes.
   - Tables are sharded across worker nodes, with each shard containing a subset of the data.
   - The controller node routes queries to the appropriate worker nodes, which execute the query in parallel and return the results to the controller node for aggregation.
   - To clients, Citus appears as a single PostgreSQL instance, with the controller node acting as a proxy for the worker nodes.
   - These features allow Citus to scale out horizontally to multiple petabytes of data (see `Architecting petabyte-scale analytics by scaling out Postgres on Azure with the Citus extension <https://techcommunity.microsoft.com/blog/adforpostgresql/architecting-petabyte-scale-analytics-by-scaling-out-postgres-on-azure-with-the-/969685>`_ for a case study).
- Though it can theoretically scale to multiple petabytes, there is a considerable amount of technical work involved in deploying and configuring Citus to achieve this.
   - The shard count used to partition tables across workers is a settable parameter in Citus, and the number of shards must be carefully chosen to balance query performance and data distribution.
   - Both the controller and worker nodes must be provisioned with sufficient hardware resources to handle the expected query load and data volume. These would need to be determined through operational testing and benchmarking.
   - Usage of network file systems can introduce latency and bottlenecks, so local storage is recommended for worker nodes. This would need to be configured and provisioned on dedicated machines and might introduce complications in system maintenance, such as backup and recovery.
   - The above is not an exhaustive list, as Citus is not a service, but a software extension to PostgreSQL that requires careful configuration and tuning to achieve optimal performance.
- **Citus should be able to handle the data volume requirements under any retention scenario that is being considered.**

Qserv
~~~~~

- Qserv has been designed to scale to multiple petabytes of data for hosting the DR catalogs.
   - Spatial sharding is used to distribute data across nodes, with each node responsible for a subset of the data.
   - System tests have been performed with ~40 TB of data, with testing on multi-petabyte data volumes planned for the near future.
   - Dedicated hardware has been purchased for Qserv at USDF, including locally attached SSD storage, to ensure performance is adequate for the expected data volumes.
- **Qserv should be capable of handling the data volumes expected for the PPDB under any retention scenario being considered.**
- Similar to Citus, significant technical work is required to deploy and configure Qserv to achieve this, including provisioning of hardware, configuration of the database, and tuning of the system for optimal performance.

AlloyDB
~~~~~~~

- AlloyDB has distributed scaling through read replicas, but it has limitations which would prevent it from scaling to the data volumes required for the PPDB.
   - AlloyDB has a maximum storage capacity of 128 TiB per primary instance, which is insufficient for the 700 TB of data that will be generated over 10 years, and also less than the 140 TB of data projected for 2 years. While data could be sharded across multiple primary instances, this would introduce complexity and operational overhead.
   - For very large datasets in the hundreds of terabytes, complex analytical queries would likely result in high latency due to the limitations of vertical scaling on the replica nodes and the absence of distributed query execution.
   - Managing backups, replication and recovery at this scale would be complex and challenging, with backup and restore operations for multi-terabyte datasets being time-consuming and operationally challenging. Index maintenance and vacuum operations would also be similarly challenging and time-consuming.
   - Storing hundreds of terabytes on AlloyDB would be expensive, as standard rates per GB hour are high.
- The above is not a comprehensive list of all limiting features, but it is clear that **AlloyDB would not be suitable for the data volumes required by the PPDB.**

BigQuery
~~~~~~~~

- BigQuery is a massively parallel database engine designed for unlimited scalability.
   - Storage and compute are decoupled, with data stored in Google's Colossus file system.
   - Stored data can be scaled to multiple petabytes without impacting query performance.
   - Queries can be scaled dynamically, regardless of the amount of data stored.
   - Data is partitioned and indexed automatically, with the query engine optimizing query plans for performance.
- Overall, **BigQuery should easily be able to meet the data volume requirements of the PPDB.**

Query Performance & Latency
---------------------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL has low to medium latency for small to medium datasets, typically ranging from milliseconds to a few seconds for indexed queries
- However, it struggles with datasets larger than 10-20 TB on a single instance.
   - I/O and memory constraints can become bottlenecks.
   - Performance degrades with high concurrency or joins across large tables.
   - Index maintenance and vacuum operations can impact performance on large datasets.
- Internal benchmarking and testing indicates that query performance scales roughly linearly with data volume, with query times increasing by a factor of 10 for every order of magnitude increase in data volume *DMTN-113* :cite:`DMTN-113`.
   - This implies that performance would degrade significantly as the PPDB grows to hundreds of terabytes.
- **A single-node PostgreSQL server cannot achieve adequate query performance on the expected data volumes.**

Citus
~~~~~

- Citus can achieve high query performance on large datasets, as it is designed to scale out horizontally across multiple nodes.
   - Queries are executed in parallel, with the controller node aggregating results from worker nodes.
   - Sub-second performance can be achieved for most queries distributed across worker nodes.
   - Proper sharding and indexing, as well as table partitioning, can improve query performance significantly.
   - Citus employs adaptive query optimization, optimizing for minimal data movement and efficient execution.
      - Queries are rewritten to leverage parallelism and avoid unnecessary computation.
      - Joins are optimized by pushing computations to worker nodes to reduce cross-shard communication.
- Columnar storage is supported for analytical workloads, allowing for efficient scanning of required coumns, which can improve performance for large-scale queries, e.g., full table scans.
- Citus supports several sharding strategies including hash-based and range-based (time-series).
- Local and global indexes can be created on sharded tables, with global indexes being replicated across all worker nodes.
- Overall, with optimized configuration and adequate hardware, **Citus should be able to achieve high query performance for the data volumes expected for the PPDB.**

Qserv
~~~~~

- Qserv is highly performant for large datasets, as it is designed to scale out horizontally across multiple nodes.
   - Tables are spatially sharded, and low latency can be achieved for restricted spatial queries.
- Query performance may degrade under certain circumstances.
   - High latency can be experienced for full table scans.
   - Long-running queries may effect other user's queries, introducing higher latency for those users.
- Testing shows that Qserv achieves faster results than BigQuery for some queries, in particular spatially oriented searches (see results from `TAP BigQuery <https://docs.google.com/document/d/1CigD-6xJgTWZV82HUEYZBCxhgwfpOHfu8ABwmjLxZ4E>_` by Burwood Group, Inc. for comparison).
- **Qserv should be able to achieve adequate query performance for the data volumes expected for the PPDB.**

AlloyDB
~~~~~~~

- AlloyDB has low latency, with sub-millisecond response times for cached queries.
- Read replicas can improve query scalability.
- However, AlloyDB is not designed for large-scale analytical queries on petabyte-scale data.
- Given the inherent degradation of latency as data volume increase on a single PostgreSQL server, **AlloyDB would not be able to achieve adequate query performance for the data volumes expected for the PPDB.**

BigQuery
~~~~~~~~

- BigQuery is designed for extreme horizontal scalability, and it is very efficient and performant for large-scale analytical queries on petabyte-scale data.
- Caching mechanisms and optimization techniques can be used to improve query performance.
   - For instance, BigQuery can cache query results for up to 24 hours, which can significantly reduce latency when returning data to clients.
- BigQuery can potentially have high latency for small queries, from several to tens of seconds, due to the serverless nature of the platform, which requires provisioning of resources for each query, as well as optimization and planning within the execution engine.
   - However, some preliminary testing indicates that results on small datasets are returned quickly, with sub-second response times for simple queries.
   - How this operates at the expected data volumes is not clear; provisioning time could be a significant factor in query latency at scale given that more "slots" would be needed for each query.
- Performance of spatial queries is not inherently optimized, as BigQuery does not support spatial indexing.
   - However, spatial queries can be optimized by using hierarchical mesh indexing, which can reduce the amount of data scanned by the query engine. This can significantly improve query performance for spatial queries, but it requires additional development effort to implement.
   - Various techniques could also be developed for optimizing spatial queries on this platform, including use of the `HEALPix <https://healpix.sourceforge.io/>_` algorithm.
- Even with the above caveats, **BigQuery should be able to achieve high query performance on the expected data volumes, especially for large-scale analytical queries.**
   - BigQuery is particularly performant on full table scans, which are a common operation for analytical queries.

Scalability
-----------

PostgreSQL
~~~~~~~~~~

- PostgreSQL can scale vertically to a certain extent with hardware improvements, but it is not designed to scale horizontally to multiple compute nodes.
- Networking, memory, and I/O constraints can all come into play for large datasets on a single PostgreSQL instance, at well below the required data volume for the PPDB.
- **Neither storage nor compute scalability is achieveable at the level required.**

Citus
~~~~~

- Citus is designed to scale out horizontally across multiple nodes and would be configured as a multi-node, single-use appliance in Kubernetes.
- Compute and storage are not completely decoupled, as indivdual workers manage a specific set of table shards.
   - This means that worker nodes must be configured and provisioned to handle the expected query load, typically with a high number of vCPUs assigned to each worker.
- I/O, memory, and CPU scaling can be achieved by selecting specific hardware for each node, and nodes can be distributed across multiple physical machines to ensure that no two nodes run on the same physical machine.
- Elasticity can be achieved by adding nodes to the cluster or removing them, but these operations requires table resharding and balancing, which can be complex and time-consuming.
   - Though in theory Citus can be dynamically scaled, in practice it may be difficult to achieve this in a production environment.
- Even with the above limitations, **Citus should be adequately scalable.**


Qserv
~~~~~

- Qserv is designed to scale out horizontally across multiple nodes.
   - Additional nodes can be added to the cluster to increase storage and compute capacity.
- Sufficient hardware must be provisionined for each node to handle the expected data volume and query load, and elasticity is generally not supported.
- **It should be able to handle the data volume and query performance requirements of the PPDB.**

.. TODO: Add more on Qserv scalability, possibly with references to system benchmarks and tests, DP and DR catalog sizes, etc.

AlloyDB
~~~~~~~

- AlloyDB uses a primary and replica setup, with the primary node handling writes and the replica nodes handling reads. This allows AlloyDB to scale out horizontally to multiple nodes.
- AlloyDB does not sufficiently scale in terms of storage capacity, as it has a (previously mentioned) maximum storage capacity of 128 TiB per primary instance.
- This platform does not have true horizontal scalability, as it uses a primary and replica setup, which is not the same as sharding data across multiple nodes.
- Query volume scaling could be achieved by adding read replicas, but this would not address the storage capacity limitations. Additionally, read replicas require storing the data multiple times, which could be expensive.
- **AlloyDB likely does not have sufficient scalability for the PPDB.**

BigQuery
~~~~~~~~

- BigQuery is designed to scale out horizontally to multiple petabytes of data.
  - Storage and compute are decoupled, with data stored in Google's Colossus file system.
  - Compute resources, or "slots" in BigQuery terminology, are provisioned dynamically for each query, allowing for virtually unlimited, dynamic scaling to meet demand.
- Of all the systems under consideration, **BigQuery has the best scalability and most attractive feature set in this area.**

Operating Cost & Cost Predictability
------------------------------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL would have low operating costs for on-premises deployments.
- Cost predictability is high for on-premises deployments, as existing infrastructure and budget would cover the overhead of running the database at USDF.
- Hardware costs could be high for a single-node deployment, as it would need to be provisioned with sufficient memory, CPU, and storage to meet the expected data volume and query load.

Citus
~~~~~

- Citus would have low operating costs for on-premises deployments, as the overhead of running the database would presumably be covered by existing infrastructure and budget.
- Cost predictability would be high for on-premises deployments, as the costs are fixed and known in advance.
- However, Citus would incur much higher hardware costs than a single-node PostgreSQL deployment, as it would require multiple nodes to be provisioned with sufficient memory, CPU, and storage to meet the expected data volume and query load.
   - This would likely include new hardware purchases, as the existing infrastructure at USDF could likely not support the required number of nodes with the proper hardware configuration, especially given that a large number of nodes would need to be exclusively dedicated to Citus.
   - Lead-in time for hardware procurement and deployment would need to be considered and could be a significant limiting factor in deploying Citus on-premises.

Qserv
~~~~~

- Qserv costs are already included in the USDF budget, as it is used to host the DP and DR catalogs.
- A hardware cluster has been purchased and configured for Qserv and is already in operation.
- However, the added load of the PPDB would likely require additional hardware to be purchased, as the existing cluster may not be able to support the expected data volume and query load while also providing access to the DP and DR catalogs.
- Similar to Citus, hardware purchasing requirements could be a significant limiting factor in deploying Qserv on-premises.

AlloyDB
~~~~~~~

- `AlloyDB pricing <https://cloud.google.com/alloydb/pricing>`_ includes separate charges for CPU and memory, storage, backup storage and networking.
   - CPU and memory charges by vCPU hour may be decreased with longer commitments.
   - Storage is priced by GB hour, though, according to the pricing page, an "intelligent regional storage system" scales up and down. Storage prices depend on the region where the instance is located.
   - Backup storage is priced by GB hour, and backups are billed from the time of completion until the end of their retention period.
   - Data transfer into AlloDB is free. Outbound data transfer is priced by GB, with variable pricing depending on the source and destination regions.
   - Hourly charges may be incurred for using certain network services such as Private Service Connect.
- The GCP `Pricing Calculator <https://cloud.google.com/products/calculator>`_ can be used to estimate costs.
- Cost predictability is medium for AlloyDB, as the costs are variable and depend on the amount of data stored, the number of queries run, and the amount of data transferred.
- Overall, without favorable pricing agreements, AlloyDB would likely be a relatively expensive platform, incurring high operating costs, which would grow over time with more data and queries.

BigQuery
~~~~~~~~

- `BigQuery pricing <https://cloud.google.com/bigquery/pricing>`_ has two main components: compute pricing and storage pricing.
- Compute pricing includes the cost to process queries, including "SQL queries, user-defined functions, scripts, and certain data manipulation language (DML) and data definition language (DDL) statements."
- BigQuery offers two compute pricing models for running queries:
   - On-demand pricing (per TiB) charges for the amount of data processed by the query, with a minimum of 10 MB per query.
   - Capacity pricing (per slot-hour) charges for the number of slots used by the query, with a minimum of 100 slots per query, and slots available in increments of 100. Billing is per second with a one-minimum.
- Storage pricing is the cost to store data that is loaded into BigQuery.
- BigQuery charges for other operations as well, such as streaming inserts and usage of integrated machine learning tools.
- Specific costing scenarios are beyond the scope of this document, but it is generally understood that BigQuery can be expensive for large datasets and high query volumes, with low cost predictability due to dynamic resource allocation for every query along with variable pricing.
- Though the default BigQuery pricing structure would likely result in very high (even prohibitively expensive) operating costs, it is possible that significant discounts could be negotiated, given the scientific nature of the project.

Maintenance Overhead
--------------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL has medium maintenance overhead, as it requires regular monitoring, backup and recovery, and scaling to meet demand.
   - On-premises deployments require administrators to manage the infrastructure, including monitoring, backup and recovery, and scaling the database to meet demand.
   - SLAC has a dedicated team of system administrators who manage the infrastructure at the USDF. This includes administration of a PostgreSQL development cluster for prompt processing.
- Administrators at USDF already have expertise with this platform, including the areas of maintenance operations, as well as configuration, maintenance, and deployment of new instances using standardized tools and procedures.
- Compared with the two other on-premises options, PostgreSQL would have a lower maintenance overhead, as it is a single-node database that does not require the same level of monitoring and management as a distributed database.

Citus
~~~~~

- An on-premises Citus deployment would likely incur very high maintenance overhead.
   - Shards need to be periodically rebalanced to ensure even distribution of data across worker nodes.
   - Distribution of data across worker nodes can be complex and require manual intervention. Distributed tables can complicate backup and recovery procedures.
   - No official Kubernetes operators or Helm charts are available for Citus, at least not through their official documentation channels, so these would need to be developed to deploy Citus on Kubernetes at the USDF.
   - Procedures and tools for monitoring, backup and recovery, and scaling would need to be developed or adapted.
- Some significant fraction of a database administrator or similar expert would be required to manage an on-site Citus deployment, making this a high-maintenance option.

Qserv
~~~~~

- As a distributed database, similar to Citus in many ways, **Qserv has a high maintenance overhead.**
- Additionally, since Qserv is a custom, in-house platform, it may require more maintenance effort than a more widely-used platform like Citus.
- Qserv will already be used to host the DP and DR catalogs, and it is unclear whether additional maintenance burden could be managed effectively by existing personnel. It is likely that additional manpower would be required to manage the PPDB on the same infrastructure.

AlloyDB
~~~~~~~

- AlloyDB has medium maintenance overhead, as it requires regular monitoring, backup and recovery, and scaling to meet demand.
   - Google provides a suite of tools for managing AlloyDB, including monitoring, backup and recovery, and scaling. These tasks are not necessarily done automatically, but the tools are available.
   - AlloyDB is designed to be fully compatible with PostgreSQL, so existing tools for monitoring and backup and recovery should work with AlloyDB.
   - The maintenance overhead of AlloyDB is likely lower than that of Citus, as it is a fully managed service and does not require the same level of monitoring and management as an on-premises deployment.
- However, the maintenance overhead of AlloyDB is likely higher than that of PostgreSQL, as it is a distributed database and requires more monitoring and management than a single-node database. Primary and replica nodes need to be setup, managed, and monitored.

BigQuery
~~~~~~~~

- BigQuery has low maintenance overhead, as it is a fully managed service and does not require the same level of monitoring and management as an on-premises deployment.
   - Google provides a suite of tools for managing BigQuery, including monitoring, backup and recovery, and scaling.
   - BigQuery is designed to be fully compatible with SQL, so certain existing tools for monitoring and backup and recovery should work with BigQuery.
- Management of BigQuery would still rely to some extend on expertise of Rubin personnel, who do not have much experience with the platform beyond a few pilot projects.

Developer Effort
----------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL would have low developer effort, as the existing schema and data ingestion tools are compatible and have been used and tested extensively in this environment.
- Development effort would generally be limited to improving or resolving bugs with existing software, such as the ingestion tools.

Citus
~~~~~

- As a fully compatible PostgreSQL extension, Citus should require relatively low developer effort, as the existing schema and data replication tools are, in theory, fully compatible.
- However, Citus would require a significant amount of development effort in devops, backup and recovery solutions, and other tools to manage the system, as mentioned in its maintenance overhead section.

Qserv
~~~~~

- Qserv would require very high developer effort, because it lacks some required features, including, but not limited to tooling for data ingestion.
   - Qserv does not support incremental inserts or updates, as it is primarily designed for loading data in bulk. Significant enhancements would be required to support nightly updates from the APDB.
- Given the existing commitments of the Qserv team, it is not clear that they would be able to devote the necessary resources to develop the required tooling on a reasonable timescale. It is likely that additional personnel would be required to develop these tools in a timely manner.

AlloyDB
~~~~~~~

- AlloyDB has been designed to be fully compatible with PostgreSQL, so most existing tools should work, including the schema and data ingestion tools.
   - Some additional overhead and configuration may been incurred by networking connectivity to GCP, but this is likely to be minimal.

BigQuery
~~~~~~~~

- BigQuery would likely require high developer effort, as the existing schema and data ingestion tools are not compatible.

TAP Service
-----------

PostgreSQL
~~~~~~~~~~

- Support for TAP services in PostgreSQL is provided by the CADC TAP implementation, with PgSphere providing spherical geometry functionality. This has already been used for Rubin services and should work with any PostgreSQL-based backend.

Citus
~~~~~

- In theory, Citus should be compatible with existing TAP services, but this would need to be verified and tested.
- There could be unknown complexities and issues with the TAP service running on a distributed system that would need to be resolved.

Qserv
~~~~~

- Qserv fully supports TAP services through a set of adapters on top of the CADC TAP implementation.
- No problems would be expected running a TAP service on Qserv, as this has been tested extensively on the RSP.

AlloyDB
~~~~~~~

- While AlloyDB is compatible with PostgreSQL, it does not support PgSphere, which is required for ADQL support in the CADC TAP implementation that has been used for Rubin services in the past.
- AlloyDB does support the `PostGIS extension <https://postgis.net/>`_, which provides support for geospatial data. However, this does not provide the same functionality as PgSphere. Significant development effort would be needed to implement the required functionality for the TAP service using a PostGIS backend. And it is not clear that this would be feasible given available software development resources and the operational schedule.

BigQuery
~~~~~~~~

- BigQuery is not compatible with the CADC TAP implementation, so a TAP service would need to be developed.
- Work has been done in the past to implement a TAP service on top of BigQuery (see `TAP and ADQL on Google’s BigQuery Platform <https://assets.pubpub.org/rynkboj6/71582749259388.pdf#abs287.02>`_).
- A production TAP service does not currently exist but there is `work in progress <https://github.com/opencadc/tap/pull/172>`_ on adding one to the CADC TAP implementation, as part of Rubin's ongoing collaboration with CADC.
- While some development effort would be required, it is likely that a working TAP service could be implemented on BigQuery.


Data Ingestion
--------------

PostgreSQL
~~~~~~~~~~

- Existing data ingestion tools are designed to copy data from Cassandra to PostgreSQL.
   - These have been extensively tested on the USDF and found to be reliable, stable and performant.
- Additional testing is on-going to ensure that the ingestion tools can handle the expected data volume of the PPDB.
- Data ingestion is currently best-supported for single-node PostgreSQL deployments.

Citus
~~~~~

- In theory, as a PostgreSQL compatible database, the existing data ingestion tools should be useable.
- However, no testing has been done with this platform, and the distribution of data across worker nodes could complicate the process.
   - Additional testing would be required to ensure that the tools can handle the expected data volume with adequate throughput on this platform.
- Bottlenecks on the coordinator node could be a concern, as it would be responsible for managing ingestion while also servicing user queries, at least with a default configuration.

Qserv
~~~~~

- No existing data ingestion tools exist for Qserv, as it is not designed to handle incremental inserts or updates.
   - It would require a major "greenfield" development effort to implement data replication from the APDB to Qserv.
   - A significant amount of development effort would be required in order to unblock implementation of these tools by adding support for SQL insert and update operations.

AlloyDB
~~~~~~~

- AlloyDB is fully compatible with PostgreSQL, so the existing data ingestion tools should work.
- Copying data from the on-premises APDB to AlloyDB on GCP may require additional development effort, as the existing tools are designed to copy data to an on-premises rather than cloud database.
   - It is possible that GCP connectivity tools could make this seemless, but this would need to be investigated and tested.

BigQuery
~~~~~~~~

- No existing data ingestion tools exist for BigQuery, as it is not compatible with the existing software.
- A significant amount of development effort would be required to implement this functionality.
   - One option would be uploading Parquet files to Google Cloud Storage (GCS) where they could then be ingested by using a BigQuery API.
- Not having these tools available would be a significant initial roadblock in implementing the PPDB on BigQuery, but it is likely surmountable given enough development effort.

Ecosystem and Community
-----------------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL is a flagship open source project with a large and active community.
   - Its documentation is extensive and well-maintained, and there are many tutorials and support forums available.
   - Many developers and companies use PostgreSQL, and there are a wide range of tools and libraries available that can be used to extend the functionality of the database platform.
- The high quality of the documentation site in particular could be considered a significant advantage of using PostgreSQL.

Citus
~~~~~

- Citus is an open source project with a growing community.
   - Though more limited than PostgreSQL, there are many developers and companies using Citus, and there are a range of tools and libraries available that can be used to extend the functionality of the database platform.
- Complete documentation is available on the `Citus website <https://www.citusdata.com/>`_, and there are many tutorials and support forums available, including a dedicated `Slack workspace <https://slack.citusdata.com>`_.
- Citus has some shortcomings in its ecosystem, as standardized deployment scripts and configurations, backup tools, and monitoring tools are not provided out of the box.
  - These would all require development effort to implement, and it is not clear that they would be available in a timely manner.
- While Citus has an active community and high quality documentation, the lack of standardized tooling in its ecosystem could be considered a limiting factor.

Qserv
~~~~~

- As an in-house platform, Qserv has an extremely limited ecosystem and community compared with all of the other platforms.
   - Documentation is available on the `Qserv website <https://qserv.lsst.io/>`_, but it is not as extensive and complete as that of PostgreSQL or Citus.
   - Qserv only has a handful of deployments, and there are no developers or companies using the platform outside of Rubin.
   - Development relies on a few key individuals, who are heavily subscribed in terms of future commitments to the project and may not have the bandwidth to develop new features or tools.
- The lack of a wider ecosystem and community could be considered a major limiting factor in terms of platform selection.

AlloyDB
~~~~~~~

- AlloyDB is a proprietary platform developed by Google, so its ecosystem and community are more limited than those of open source platforms like PostgreSQL and Citus.
   - Documentation is available on the `Google Cloud website <https://cloud.google.com/alloydb>`_, but it is not as extensive as that of PostgreSQL or Citus.
- Support could be obtained through GCP support channels, if necessary.
- This is probably not a significant limiting factor in terms of platform selection, as the existing resources seem adequate.

BigQuery
~~~~~~~~

- BigQuery has a large and active community, with extensive documentation and tutorials available.
   - Google Cloud Platform has a wide range of tools and libraries available that can be used to extend the functionality of BigQuery.
   - Many developers and companies use BigQuery, and there are many support forums available, including the dedicated `BigQuery Slack workspace <https://cloud.google.com/blog/topics/inside-google-cloud/join-the-google-cloud-community-on-slack>`_.
- The high quality of the available documentation and support could be considered a significant advantage of using BigQuery.

Summary
=======

There is no clear winner across all categories, but several options can be eliminated which do not meet basic requirements.

The following summarizes the findings for each platform:

PostgreSQL
----------

PostgreSQL is an attractive RDMS platform in general, due to its feature set, excellent documentation, and large community. Rubin and SLAC also have extensive experience with PostgreSQL, and the existing PPDB is implemented on this platform.
Low development and maintenance effort would be required to implement the PPDB on PostgreSQL, as it has heretofore been the target platform for the PPDB implementation.
However, PostgreSQL is not designed to scale out horizontally, and it simply cannot handle the projected data volume and query performance requirements.
**A single PostgreSQL server is not a suitable platform for the PPDB and can be eliminated as a longterm viable option.**

Citus
-----

Citus brings with it all of the positive features of PostgreSQL, as it is an extension of that platform.
The platform is designed to scale out horizontally, and it should be able to handle the data volume and query performance requirements.
However, Citus would likely incur very high maintenance overhead, as it requires regular monitoring, backup and recovery, and scaling to meet demand. USDF personnel also have no experience with this platform and would presumably need to learn how to manage it.
Running Citus on-premises would require the development of Kubernetes operators or Helm charts, backup and recovery solutions, and other tools to manage the distributed database. This would necessitate a significant amount of development effort upfront.
A rough estimation is that at least one FTE or more would be required for the initial build out, testing, and deployment of Citus, and ongoing maintenance would require a significant fraction of time from a database administrator or similar expert.
Given these factors, **Citus is technically a viable option for the PPDB, but the maintenance overhead and effort required to develop configuration and monitoring tools would be considerable and should not be underestimated.**

Qserv
-----

Qserv is a distributed database that is designed to scale out horizontally, and it should be able to handle the data volume and query performance requirements of the PPDB.
It has been used to host the data previews and will contain multi-petabyte DR catalogs.
However, Qserv would require very high developer effort, as it is missing many required features, including tooling to ingest data from the APDB.
**Qserv is a possibility for hosting the PPDB, but there are significant constraining factors including the high developer effort required to implement the required tooling, a limited developer ecosystem and community, and the existing commitments of the Qserv team.**

AlloyDB
-------

AlloyDB has an attractive set of features built on top of PostgreSQL, including compatibility with the existing PPDB schema and data replication tools.
It is designed to scale out horizontally, via read replicas, and so it would perform better than a single node PostgreSQL instance.
However, data volume requirements under the proposed scenarios would exceed its maximum storage capacity, and the platform still has many of the problems associated with a single-node database.
**The inability of AlloyDB to scale to the required data volumes makes it an infeasible choice for the PPDB.**

BigQuery
--------

BigQuery is a fully managed service with low maintenance overhead, excellent scalability, and good query performance.
It is designed for extreme horizontal scalability and can handle petabytes of data, so it should be able to meet the data volume requirements of the PPDB.
However, the developer effort required to migrate to this platform is significant, as the existing schema and data replication tools are not compatible.
The cost of running the service is also unknown, and it is possible that high operating costs would be incurred, which would grow over time with more data and queries.
**BigQuery is a good fit in terms of scalability and query performance, but the developer effort required to migrate to this platform is significant, and the cost of running the service is unknown.**

Recommendations
===============

With several options eliminated entirely, the following alternatives are recommended:

1. BigQuery
-----------

Of all the platforms, BigQuery offers the most attractive featureset in terms of meeting or exceeding the use case and has been designed from the ground-up to provide unlimited scaling of compute and storage resources.
It is a fully managed service, with low maintenance overhead, and has excellent scalability and query performance.
Support could be obtained through Rubin's existing GCP contract, and costs could be negotiated to be more favorable.

A pilot project by Rubin staff used BigQuery as part of *Google Cloud Engagement Results* :cite:`DMTN-125` and reported favorable results.

  The results for BigQuery show significant speedups for queries that retrieve a limited number of columns, as expected due to BigQuery’s columnar organization. Spherical geometry primitives were able to be adapted for use in astronomical queries. Proper data organization, in particular clustering the BigQuery tables by spatial index, along with the use of a spatial restriction primitive led to substantial improvements in query time for a near-neighbor query. Retrieval of individual objects was relatively slow, however, due to BigQuery’s startup time and lack of indexing. It seems clear that it is possible, with some work on ADQL (Astronomical Data Query Language) translation and possibly creation of auxiliary tables, for BigQuery to handle the largest-scale catalog queries.

While a TAP service does not currently exist, one is under development, and it is likely that this could be adapted to run on BigQuery once it is complete.
Data ingestion tools would also need to be written, but this should be a relatively straightforward process, as BigQuery has a well-documented API and many libraries available for interacting with the service.

Overall, while still requiring significant up-front development effort, BigQuery represents the best choice out of the available options for hosting a database at the required scale and query performance, with a minimum maintenance overhead.
This option also opens up possibilities for other institutions and individuals to access the data and pay for their own queries, which could be a significant advantage in the future.
Finally, the longevity of the platform would be improved compared to in-house options that might become technically obsolete or unsupported in the future.

2. Citus
--------

Citus has an attractive feature set, as it is an extension of PostgreSQL which is designed to scale out horizontally across multiple nodes.
Its documentation claims that petabyte scalability is achievable given the proper hardware and configuration.
Some existing tools that have already been developed for PostgreSQL should work with Citus, and the platform should be able to handle the data volume and query performance requirements of the PPDB.

However, maintenance overhead and developer effort incurred from such a complicated on-premises deployment would be considerable and likely quite challenging.
A significant amount of administrative and developer effort would be required to develop configuration and monitoring tools, deployment scripts, backup and recovery solutions, and other tools.
Furthermore, beyond the standard tools for PostgreSQL, there seems to be a lack of standardized tooling within the Citus ecosystem for common administrative and maintenance tasks.
It is not clear that there is sufficient manpower available to address these shortcomings, and the cost of purchasing the necessary hardware would likely be high.
The lead-in time for purchasing, configuring, and deploying hardware at SLAC would be long, as much as one year, and the operational schedule dictates that the PPDB must be operational before this.
If these challenges can be overcome, Citus could be a viable option for the PPDB, especially if an on-premises deployment is preferable in terms of costing or for other reasons.

3. Qserv
--------

Qserv should be able to handle the data volume and query performance requirements.
But the required developer effort for new tooling and capabilities would be very high, as data ingestion capabilities would need to be developed.
The existing commitments of the Qserv team might prevent them from devoting the necessary resources to develop the required tooling on a reasonable timescale.
The ecosystem and community are also quite limited.
For these reasons, Qserv is not recommended as a primary option, though in terms of technical capability, it could be a viable choice.

4. Interim Solution
-------------------

Given the constraints and requirements, it may be necessary to provide an interim solution using existing PostgreSQL-based tooling.
This would allow the PPDB to be operational in a timely manner, while the longer-term solution is developed and deployed.
Software has already been developed for data ingestion, which has been tested and found to be reliable, stable, and sufficiently performant at high data volumes.
Additionally, a TAP service could be configured and deployed to the RSP with minimal effort.
Vertical scaling could be used to address performance requirements initially.
This scheme would at least provide a working system that would allow the PPDB to be operational in a timely manner.

However, there are major issues with this approach, as PostgreSQL may struggle with query loads at a much lower data volume than would be expected even just for the first few months of operations.
Certainly, after several months of operations when the data volume is expected to exceed 100 TB, the system would likely be unable to handle the user query load without unacceptable latency.
Vertical scaling or other stopgap measures would only be able to address this issue for a limited time, and the system would likely need to be replaced or significantly reconfigured well within a year of deployment, and likely much sooner.

.. _DMTN-113: https://dmtn-113.lsst.io
.. _DMTN-125: https://dmtn-125.lsst.io
.. _DMTN-135: https://dmtn-135.lsst.io
.. _DMTN-268: https://dmtn-268.lsst.io
.. _DMTN-293: https://dmtn-293.lsst.io
.. _LSE-61: https://ls.st/LSE-61
.. _LSE-163: https://lse-163.lsst.io
.. _SITCOMTN-149: https://sitcomtn-149.lsst.io

References
==========

.. bibliography::
