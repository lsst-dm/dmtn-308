####################################################################
Database Platform Comparison for the Prompt Products Database (PPDB)
####################################################################

.. abstract::

   This note provides a comparison of database platform alternatives for implementing the Prompt Products Database (PPDB).

Overview
========

The Prompt Products Database (PPDB) will provide user access to level 1 data products, which are produced as a result of nightly processing.
The specifics of these data products, including the conceptual schemas, are covered in Section 3 of the `Data Products Definition Document <https://lse-163.lsst.io/>`_ .
Additionally, several tech notes have been written on specific aspects of the PPDB, including `DMTN-113`_ :cite:`DMTN-113`, `DMTN-268`_ :cite:`DMTN-268`, and `DMTN-293`_ :cite:`DMTN-293`.
These have covered performance of a PostgreSQL-based PPDB implementation, data ingestion, and system architecture, respectively.
The exact database platform used to implement the PPDB has not been determined, and this note provides a comparison of the alternatives.

Requirements
============

Data Volume & Retention
-----------------------

The Alert Production Database (APDB) is designed to retain data for a 1-year period.
The PPDB would ideally retain data for the lifetime of the project, which is currently planned for 10 years.
Based on scheduling considerations of Data Release Processing (DRP), a data retention of 2 years will be considered as a minimum requirement.
.. More information from Gregory here on why the 2 years is a minimum requirement

The following table provides an estimation of stored data volume for the PPDB.

.. list-table:: PPDB Data Volume Projects
   :header-rows: 1

   * - **Single Visit**
     - 190 MB
   * - **Nightly**
     - 190 GB
   * - **Yearly**
     - 70 TB
   * - **10 Years**
     - 700 TB

The exact size of the nightly data products which will be produced by LSSTCam is undetermined.
Data taking during the `ComCam On-Sky Campaign <https://sitcomtn-149.lsst.io/>`_ resulted in an average size per visit of approximately 9 MB with 9 active detectors.
Extrapolating to the full camera with 189 detectors results in an estimated single visit size of *189/9 * 9 MB = ~190 MB*.
Since LSSTCam is expected to produce approximately 1000 visits per night, this would result in a nightly data volume of approximately 190 GB.

These figures are almost certainly underestimated, because ComCam data processing resulted in sparse data products containing many `null` values.
Actual LSSTCam data products will likely be denser, and this density is expected to increase over time as pipeline algorithms are improved and more columns are filled.
Additionally, Solar System Processing (SSP) was not included in the ComCam data processing, and this is expected to increase the data volume by an unknown factor.

Deployment
----------

Two basic options exist for deploying the PPDB: on-premises at the US Data Facility (USDF) or in the cloud.
Cloud deployments would target `Google Cloud Platform <https://cloud.google.com/>`_ (GCP), which has been used for the Interim Data Facility (IDF) and already hosts many Rubin services.
Rubin has a long-term contract with Google for cloud services, which makes using other providers less attractive and feasible.

Query Performance & Latency
---------------------------

Query performance requirements for the PPDB are covered by *DMS-REQ-0355* in the `Data Management System Requirements <https://ls.st/LSE-61>`_.
These specify that the minimum number of simultaneous users should be 20, and that the maximum query time should be 10 seconds.
Query latency is highly dependent on the complexity of the query and the size of the database, so this requirement may not be satisfiable for all possible queries.
Given the expected data volumes, longer queries may be necessary to extract the desired information from the system.
The PPDB is expected to be used by a large number of users, and this may vary considerably depending on the time of day, the phase of the project, and other factors.

Scalability
-----------

Scalability is a multi-factor metric that includes the ability to scale out horizontally to handle large data volumes and high query loads.
While covered by other requirements, it is worth discussing and characterizing the overall scalability of each database platform.
The system should be able to handle the expected data volume of 700 TB over 10 years, as well as the expected query load of 20 simultaneous users, with as little latency as possible.

Operating Cost
--------------

Operatings costs include the cost of running the database platform, including storage, compute, and network costs.
Development and maintenance costs in terms of personnel time are not considered here.
Hardware purchase costs are considered broadly for on-premises deployments, but specific dollar amounts are not provided.
For on-premises deployment, it will be assumed that infrastructure costs such as cooling, power, and networking are already covered.
Cloud deployments will include discussion of the variable costs from running the database platform on GCP, but, again, specific dollar amounts are not provided.

Cost Predictability
-------------------

As a general rule, cloud deployments are less predictable in terms of cost than on-premises deployments.
The cost of running a database on GCP can vary depending on the amount of data stored, the number of queries run, and the amount of data transferred.

Maintenance Overhead
--------------------

Large, distributed databases generally require a significant amount of maintenance to keep them running smoothly and efficiently.
This may include monitoring, backup and recovery, and scaling to meet demand.
On-premises deployments require administrators to manage the infrastructure, while at least some of this burden is shifted to the provider in a cloud deployment.
Maintenance and development efforts may overlap significantly, especially in the early stages of building out the platform.

Developer Effort
----------------

Significant development effort may be required, depending on the database platform chosen.
This includes development of the database schema, data ingestion tools, TAP service, as well as deployment and monitoring tools.
Additionally, some options may require more development effort for the database platform itself, such as developing Kubernetes operators or Helm charts.

TAP Service
-----------

User access to the PPDB will be provided by an `IVOA TAP service <https://www.ivoa.net/documents/TAP/>`_ through the Rubin Science Platform (RSP) and HTTP APIs and other programmatic interfaces.
The availability of a TAP service for the database platform will be a significant factor in the decision of which platform to use.
Some of the database platforms do not have existing TAP implementations and may require significant effort to either developer a new implementation or adapt an existing one.
The `CADC TAP service <https://github.com/opencadc/tap>`_` runs on top of PostgreSQL and has been used for some existing Rubin services.
PostgreSQL compatibility is a significant advantage in this regard.

Data Ingestion
--------------

The PPDB will ingest data from the APDB on a nightly basis.
This is currently implemented by writing Parquet files to disk from the APDB and then using a  `COPY` operation in ingest the data into PostgreSQL.
We will primarily consider whether the target platform can support the existing data ingestion tools and if additional development effort would be required.
The performance of data ingestion would be difficult to estimate without a specific implementation, which does not exist for several of the platforms under consideration.

Ecosystem and Community
-----------------------

The ecosystem and community around the database platform are important factors to consider.
This includes availability of documentation, tutorials, and support forums, as well as the number of developers and companies using the platform.
A large ecosystem and community can provide valuable resources and support for developers, as well as a wide range of tools and libraries that can be used to extend the functionality of the database platform.

Database platforms
==================

Given the requirements outlined above, we consider the following database platforms:

PostgreSQL
----------

PostgreSQL is the current database platform that has been used for development and testing of the PPDB at USDF.
The `dax_ppdb <https://github.com/lsst/dax_ppdb>`_ repository contains command-line tools and APIs for creating the database schema in PostgreSQL from its `Felis representation <https://github.com/lsst/sdm_schemas/blob/main/python/lsst/sdm_schemas/schemas/apdb.yaml>`_, as well as ingesting data into a target PostgreSQL database from the APDB.

Citus
-----
`Citus <https://www.citusdata.com/>`_ is an open source extension that transforms PostgreSQL into a distributed database.
Citus uses a controller-worker model to distribute data across multiple nodes, allowing for horizontal scaling of both storage and compute.
Because Citus is an extension of PostgreSQL, it should be largely compatible with the existing PPDB schema and data ingestion tools.

Google AlloyDB for PostgreSQL
-----------------------------
`AlloyDB <https://cloud.google.com/products/alloydb>`_ is a distributed database that is compatible with PostgreSQL.
Though it has an on-premises version, it is primarily designed to run on GCP.
It is typically configured using a primary and replica setup, with the primary node handling writes and the replica nodes handling reads.
AlloyDB is also designed to be fully compatible with PostgreSQL, so it should be compatible with the existing PPDB schema and data replication tools.
Internally, AlloyDB uses the Google Colossus file system for storage, which may provide performance benefits over traditional PostgreSQL.

Google BigQuery
---------------
BigQuery is a fully managed, serverless data warehouse that is designed to scale out horizontally.
It is designed to handle large volumes of data and is optimized for fast query performance.
BigQuery is not compatible with the existing PPDB schema and data replication tools, so it would require a significant amount of work to migrate to this platform.

Qserv
-----

`Qserv <https://qserv.lsst.io/>`_ was developed to host the astronomical catalogs for the LSST Data Management System.
It is a distributed database that is designed to scale out horizontally across multiple nodes.
Qserv will be used to host the Data Release (DR) catalogs and has hosted, and will continue to host, Data Preview (DP) catalogs.
Rubin has a small team of developers who maintain the platform and develop new features.

Platform Comparison
===================

The following table provides a comparison of the database platforms based on the requirements outlined above.

.. TODO add color coding (Fritz)

.. list-table:: Platform Comparison Matrix
   :header-rows: 1

   * -
     - **PostgreSQL**
     - **Citus**
     - **Qserv**
     - **AlloyDB**
     - **BigQuery**

   * - **Data Volume & Retention**
     - No
     - Yes
     - Yes
     - No
     - Yes

   * - **Deployment**
     - USDF
     - USDF
     - USDF
     - GCP
     - GCP

   * - **Query Performance & Latency**
     - Medium
     - High
     - High
     - High
     - Very High

   * - **Query Latency**
     - Medium
     - Low to Medium
     - High (?)
     - Low
     - High (small queries)

   * - **Scalability**
     - Low
     - High
     - High
     - Medium
     - Very High

   * - **Operating Cost**
     - Low
     - Low
     - Low
     - Medium
     - High

   * - **Cost Predictability**
     - High
     - High
     - High
     - Medium
     - Low

   * - **Maintenance Overhead**
     - Medium
     - Very High
     - High
     - Medium
     - Low

   * - **Developer Effort**
     - Low
     - High
     - Very High
     - Medium
     - High

   * - **TAP Service**
     - Yes
     - Yes
     - Yes
     - No
     - No

   * - **Replication**
     - Yes
     - Yes
     - No
     - Yes
     - No

   * - **Ecosystem and Community**
     - Very good
     - Good
     - Limited
     - Good
     - Very Good

Data Volume & Retention
-----------------------

PostgreSQL
~~~~~~~~~~

- According to its `published limits <https://www.postgresql.org/docs/current/limits.html>`_ , PostgreSQL has a maximum table size of 32 TB, which is insufficient for estimated data volumes in any realistic data retention scenario.
- Though PostgreSQL theoretically supports unlimited database size by using partitioning, practical constraints such as query performance degradation, index management overhead, and maintenance tasks (e.g., vacuum and analyze) make it impractical for datasets with a magnitude of hundreds of terabytes.
- Backup and restore operations for large datasets (e.g., > 100 TB) become increasingly time-consuming and operationally challenging.
- Vertical scaling of PostgreSQL is limited by hardware constraints, such as I/O, memory, and CPU, which can be a bottleneck for large datasets.
- Overall, a single PostgreSQL instance cannot scale to the data volume requirements of the PPDB.

Citus
~~~~~

- Citus is designed to scale out horizontally to multiple petabytes, so it should be able to handle the data volume requirements of the PPDB.
- Low-level configuration is required to optimize performance for large datasets, including sharding and indexing.
  - For instance, the shard count is a settable parameters that would to be tested and tuned.
- So while Citus can handle the data volume requirements, it would require additional development effort to optimize performance for the expected data volume.

Qserv
~~~~~

- Qserv is a MPP system designed to scale to multiple petabytes of data, and so it should be able to handle the data volume requirements of the PPDB.

AlloyDB
~~~~~~~

- AlloyDB has a maximum storage capacity of 128 TiB per primary instance, which is insufficient for the 700 TB of data that will be generated over 10 years, and also less than the 140 TB of data that will be generated over 2 years.
- Given that final data volumes could be 700 TB or greater, AlloyDB is not a suitable platform for the PPDB.

BigQuery
~~~~~~~~

- BigQuery can handle petabytes of data, so it should be able to handle the data volume requirements of the PPDB.


Deployment
----------

We assume that single server PostgreSQL, Citus, and Qserv would all run on-premises at the USDF.
AlloyDB and BigQuery are cloud-native platforms that would run on GCP.
While the on-premises solutions could technically be deployed on GCP, we do not consider these scenarios here.
AlloyDB also has an on-premises option, but we do not consider this either.
Finally, BigQuery is cloud-native with no on-premises option.

PostgreSQL
~~~~~~~~~~

- PostgreSQL can be deployed on-premises at the USDF, where it is currently already being used for development and testing of the PPDB.
- `CloudNativePG <https://cloudnative-pg.io/>`_ has been used at USDF to deploy PostgreSQL on Kubernetes, including some existing PostgreSQL servers used for PPDB development.
  - This provides a suite of tools for managing PostgreSQL on Kubernetes, including monitoring, backup and recovery, and scaling.

Citus
~~~~~

- Citus can be deployed on-premises at the USDF.
- No standard Kubernetes operators or Helm charts seem to exist for Citus, or at least none are listed on the `Citus website <https://www.citusdata.com/>`_. These would need to be developed to deploy Citus on Kubernetes at the USDF.
- Hardware requirements would need to be considered for Citus, as it is a distributed database that requires multiple nodes to operate.
  - Assuming the need to service 20 simultaneous users and therefore concurrent connections, as well as server overhead, a reasonable estimate for the number of vCPUs required would be around 24. PostgreSQL forks a new process for every connection, so this would be the minimum number of vCPUs required to meet the requirement.
  - This is achievable on commodity hardware, but Kubernetes configuration would be required to ensure that the Citus controller and worker nodes are distributed across multiple physical machines, do not run on the same physical machine, and have sufficient memory and disk I/O to meet the requirements of the PPDB.
  - While SLAC has a large computing cluster dedicated to USDF, it is generally shared amongst many different services and projects, so it is not clear that the necessary resources would be available to deploy Citus on-premises without additional hardware allocation.

Qserv
~~~~~

- Qserv is already deployed on-premises at the USDF.
- PPDB could be deployed on the same infrastructure as Qserv, and the same team of system administrators could manage both services.

Query Performance & Latency
---------------------------

.. TODO: latency from hardware configuration, network latency, memory, disk I/O, and query complexity
.. Will the database meet the needs of the use case?
.. multi-dimensional metric - pick between databases

PostgreSQL
~~~~~~~~~~

- PostgreSQL has medium latency for small to medium datasets, typically ranging from milliseconds to a few seconds for indexed queries. However, it struggles with datasets larger than 10-20 TB on a single instance due to high I/O and memory constraints.
- Performance degrades with high concurrency or large joins across large tables.
- Index maintenance and vacuum operations can impact performance on large datasets.

.. not degredation with large datasets; cite Andy's tech note

Citus
~~~~~

- Citus has high query performance for large datasets, as it is designed to scale out horizontally across multiple nodes. Sub-second performance can be achieved for most queries distributed across worker nodes.
- With proper sharding and indexing, Citus can achieve high query performance for large datasets.

Qserv
~~~~~

- Qserv is highly performant for large datasets, as it is designed to scale out horizontally across multiple nodes.
- Tables are spatially sharded, and low latency can be achieved for restricted spatial queries (cone searches).
- High latency can be experienced for full table scans.
- Long-running queries may effect other user's queries, introducing higher latency for those users.

AlloyDB
~~~~~~~

- AlloyDB has low latency, with sub-millisecond response times for cached queries.
- Read replicas can improve query scalability.

BigQuery
~~~~~~~~

- BigQuery has high latency for small queries, from several to tens of seconds, due to the serverless nature of the platform, which requires provisioning of resources for each query, as well as optimization and planning within the execution engine.
- Designed for extreme horizontal scalability, it is very efficient and performant for large-scale analytical queries on petabyte-scale data.
- Caching mechanisms and optimization techniques can be used to improve query performance.
  - For instance, BigQuery can cache results of queries for up to 24 hours, which can significantly reduce query latency for repeated queries.
- Performance of spatial queries is not inherently optimized, as BigQuery does not support spatial indexing.
  - However, spatial queries can be optimized by using hierarchical mesh indexing, which can reduce the amount of data scanned by the query engine.
  - This can significantly improve query performance for spatial queries, but it requires additional development effort to implement.

Scalability
-----------

PostgreSQL
~~~~~~~~~~

- PostgreSQL can scale vertically to a certain extent, but it is not designed to scale out horizontally.
- While PostgreSQL can be used in a master-slave configuration for read scaling, it is not designed to scale out horizontally across multiple nodes.

Citus
~~~~~

- Citus is designed to scale out horizontally across multiple nodes, so it should be able to handle the data volume and query performance requirements of the PPDB.

.. add to cost or overview?
.. "multi-node, single-use appliance"
.. discuss I/O, memory, and CPU scaling
.. locally attached SSD storage
.. can specify nodes to select specific hardware
.. also, don't put 2 on the same physical machine
.. wouldn't dynamically auto-scale

Qserv
~~~~~

- Qserv is designed to scale out horizontally across multiple nodes, so it should be able to handle the data volume and query performance requirements of the PPDB.

AlloyDB
~~~~~~~

- AlloyDB uses a primary and replica setup, with the primary node handling writes and the replica nodes handling reads. This allows AlloyDB to scale out horizontally to multiple nodes.
- AlloyDB does not sufficiently scale in terms of storage capacity, as it has a (previously mentioned) maximum storage capacity of 128 TiB per primary instance.

.. TODO: add BigQuery

Operating Cost & Cost Predictability
------------------------------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL has low operating costs for on-premises deployments, as the overhead of running the database would presumably be covered by existing infrastructure and budget.
- Cost predictability is high for on-premises deployments, as the costs are fixed and known in advance.

Citus
~~~~~

- Citus has low operating costs for on-premises deployments, as the overhead of running the database would presumably be covered by existing infrastructure and budget.
- Cost predictability is high for on-premises deployments, as the costs are fixed and known in advance.

Qserv
~~~~~

- Qserv has low operating costs for on-premises deployments, as the overhead of running the database would presumably be covered by existing infrastructure and budget.
- Cost predictability is high for on-premises deployments, as the costs are fixed and known in advance.

AlloyDB
~~~~~~~

- `AlloyDB pricing <https://cloud.google.com/alloydb/pricing>`_ includes separate charges for CPU and memory, storage, backup storage and networking.
- CPU and memory charges by vCPU hour may be decreased with longer commitments.
- Storage is priced by GB hour, though, according to the pricing page, an "intelligent regional storage system" scales up and down. Storage prices depend on the region where the instance is located.
- Backup storage is priced by GB hour, and backups are billed from the time of completion until the end of their retention period.
- Data transfer into AlloDB is free. Outbound data transfer is priced by GB, with variable pricing depending on the source and destination regions.
- Hourly charges may be incurred for using certain network services such as Private Service Connect.
- The `Pricing Calculator <https://cloud.google.com/products/calculator>`_ can be used to estimate costs.
- Cost predictability is medium for AlloyDB, as the costs are variable and depend on the amount of data stored, the number of queries run, and the amount of data transferred.

BigQuery
~~~~~~~~

- `BigQuery pricing <https://cloud.google.com/bigquery/pricing>`_ has two main components: compute pricing and storage pricing.
- Compute pricing includes the cost to process queries, including "SQL queries, user-defined functions, scripts, and certain data manipulation language (DML) and data definition language (DDL) statements."
- BigQuery offers two compute pricing models for running queries:
  - On-demand pricing (per TiB) charges for the amount of data processed by the query, with a minimum of 10 MB per query.
  - Capacity pricing (per slot-hour) charges for the number of slots used by the query, with a minimum of 100 slots per query, and slots available in increments of 100. Billing is per second with a one-minimum.
- Storage pricing is the cost to store data that is loaded into BigQuery.
- BigQuery charges for other operations as well, such as streaming inserts and usage of integrated machine learning tools.
- The `Pricing Calculator <https://cloud.google.com/products/calculator>`_ can be used to estimate costs.
- Specific costing scenarios are beyond the scope of this document, but it is generally understood that BigQuery can be expensive for large datasets and high query volumes, with low cost predictability due to dynamic resource allocation and variable pricing.

.. in end analysis, BigQuery operating cost is negotiable with GCP; significant discount opportunities may exist given the scientific nature of our project

Maintenance Overhead
--------------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL has medium maintenance overhead, as it requires regular monitoring, backup and recovery, and scaling to meet demand.
- On-premises deployments require administrators to manage the infrastructure, including monitoring, backup and recovery, and scaling the database to meet demand.
- SLAC has a dedicated team of system administrators who manage the infrastructure at the USDF. This includes administration of a PostgreSQL development cluster for prompt processing.
- Compared with the two other on-premises options, PostgreSQL has a lower maintenance overhead, as it is a single-node database and does not require the same level of monitoring and management as a distributed database.

Citus
~~~~~

- Citus has very high maintenance overhead, as it requires regular monitoring, backup and recovery, and scaling to meet demand.
- Shards need to be periodically rebalanced to ensure even distribution of data across worker nodes.
- Distribution of data across worker nodes can be complex and require manual intervention. Distributed tables can complicate backup and recovery procedures.
- No official Kubernetes operators or Helm charts are available for Citus, at least not through their official documentation channels, so these would need to be developed to deploy Citus on Kubernetes at the USDF.
- Some significant fraction of a database administrator or similar expert would be required to manage an on-site Citus deployment.

Qserv
~~~~~

- As a distributed database, similar to Citus in many ways, Qserv has a high maintenance overhead.
- Additionally, since Qserv is a custom, in-house platform, it may require more maintenance effort than a more widely-used platform like Citus.
- Qserv will already be used to host the DP and DR catalogs, and it is unclear whether additional maintenance burden could be managed effectively by existing personnel.

AlloyDB
~~~~~~~

- AlloyDB has medium maintenance overhead, as it requires regular monitoring, backup and recovery, and scaling to meet demand.
- Google provides a suite of tools for managing AlloyDB, including monitoring, backup and recovery, and scaling.
- AlloyDB is designed to be fully compatible with PostgreSQL, so existing tools for monitoring and backup and recovery should work with AlloyDB.
- The maintenance overhead of AlloyDB is likely lower than that of Citus, as it is a fully managed service and does not require the same level of monitoring and management as an on-premises deployment.
- However, the maintenance overhead of AlloyDB is likely higher than that of PostgreSQL, as it is a distributed database and requires more monitoring and management than a single-node database. Primary and replica nodes need to be setup, managed, and monitored.

BigQuery
~~~~~~~~

- BigQuery has low maintenance overhead, as it is a fully managed service and does not require the same level of monitoring and management as an on-premises deployment.
- Google provides a suite of tools for managing BigQuery, including monitoring, backup and recovery, and scaling.
- BigQuery is designed to be fully compatible with SQL, so certain existing tools for monitoring and backup and recovery should work with BigQuery.
- Management of BigQuery would rely to some extend on expertise of Rubin personnel, who do not have much experience with the platform.

Developer Effort
----------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL has low developer effort, as the existing PPDB schema and data replication tools are compatible with PostgreSQL.
- Development effort would generally be limited to improving or resolving bugs with existing software, such as the replication tool.
- The CADC TAP server should work "out of the box" for a PostgreSQL-based PPDB, requiring little development effort unless new features were being added.

Citus
~~~~~

- As a fully compatible PostgreSQL extension, Citus should require low developer effort in terms of database tooling and TAP software, as the existing PPDB schema and data replication tools are compatible with PostgreSQL.
- However, Citus would require a significant amount of development effort to develop Kubernetes operators or Helm charts, backup and recovery solutions, and other tools to manage the distributed database. Some of these exist already but others would need to be adapted or developed.
- In theory, the CADC TAP server should work with Citus, but this would need to be tested and verified.

Qserv
~~~~~

- Qserv would require very high developer effort, initially on the order of 1 FTE or more, because it is missing many features that are required for the PPDB, including tooling to replicate data from the APDB.
- Qserv is not designed to handle inserts or updates and is primarily oriented towards bulk data loading, so enhancements would be required in order to support the incremental inserts and updating from the APDB.
- Given the existing commitments of the Qserv team, it is not clear that they would be able to devote the necessary resources to develop the required tooling for the PPDB on the required schedule.

AlloyDB
~~~~~~~


.. TODO: add AlloyDB and BigQuery


TAP Service
-----------

The TAP service also falls under developer effort but is given special consideration here because it is a critical component of the PPDB in terms of user accessibility.
It is planned that the PPDB will provide user access to the database through a TAP service, which will allow users to query the database using the Astronomical Data Query Language (ADQL).
Additionally, any interfaces on the Rubin Science Platform (RSP) which access the PPDB would be built on top of the TAP service.

PostgreSQL
~~~~~~~~~~

- Support for TAP services in PostgreSQL is provided by the CADC TAP implementation, with PgSphere providing spherical geometry support. This has already been used for Rubin services and should work with a PostgreSQL-based PPDB.

Citus
~~~~~

- In theory, as a fully PostgreSQL compatible platform, Citus should support existing TAP services, but this would need to be verified and tested.
- There could be unknown complexities and issues with the TAP service that would need to be resolved.

Qserv
~~~~~

- Qserv fully supports TAP services through customized implementations on top of the CADC TAP implementation.
- No problems would be expected running a TAP service on Qserv for the PPDB.

AlloyDB
~~~~~~~

- While AlloyDB is compatible with PostgreSQL, it does not support PgSphere, which is required for ADQL support in the CADC TAP implementation that has been used for Rubin services.
- AlloyDB does support the `PostGIS extension <https://postgis.net/>`_, which provides support for geospatial data. However, this does not provide the same functionality as PgSphere. Significant development effort would be needed to implement the required functionality for the TAP service using a PostGIS backend. And it is not clear that this would be possible given available software development resources.
- Additionally, the TAP service would realistically need to be run on GCP, which is certainly possible, but would require additional development effort.

.. When using spatial indexing that is not spherical, it may give you more data than you need, but as long as it returns the correct data, this could work. What would be needed in conjunction with this cut is an "and" with exact geometry to get the real answer. Need to apply precise spherical geometry predicate. PostGIS only solves the first part of this problem. Data has been reduced but can't just return all that data, because it is wrong answer (outside the cone). Have to refine returned data. This was part of the extra work on BigQuery - hierarchical mesh index.

BigQuery
~~~~~~~~

- BigQuery is not compatible with the CADC TAP implementation, so a TAP service would need to be developed.
- Work has been done in the past to implement a TAP service on top of BigQuery (see `TAP and ADQL on Google’s BigQuery Platform <https://assets.pubpub.org/rynkboj6/71582749259388.pdf#abs287.02>`_), but the status of this implementation and the location of the source code is unknown and would need to be investigated.

.. would like a TAP implementation on BigQuery; strategic considerations
.. Ross Thompson - TAP over BigQuery connection (does he still work for Google? what is the status of this project?)

Replication
-----------

.. actually call this Data Ingest

PostgreSQL
~~~~~~~~~~

- Existing replication tools are designed to copy data from Cassandra to PostgreSQL.
- These have been extensively tested on the USDF and found to be reliable, stable, and sufficiently performant.
- Additional testing is on-going to ensure that the replication tools can handle the expected data volume of the PPDB.

Citus
~~~~~

- In theory, as a PostgreSQL compatible database, the existing replication tools should be useable with Citus.
- However, no testing has been done with this platform, and the distribution of data across worker nodes could complicate the replication process. Additional testing would be required to ensure that the replication tools can handle the expected data volume of the PPDB.

Qserv
~~~~~

- No existing replication tools exist for Qserv, as it is not designed to handle inserts or updates.
- It would require a major "greenfield" development effort to implement data replication from the APDB to Qserv.
- Furthermore, since Qserv is not designed to handle incremental updates, a significant amount of development effort would be required in order to unblock implementation of these tools for the PPDB by adding support for incremental inserts and updates.

AlloyDB
~~~~~~~

- AlloyDB is fully compatible with PostgreSQL, so the existing replication tools should work with AlloyDB.
- Copying data from the on-premises APDB to AlloyDB on GCP may require additional development effort, as the existing tools are designed to copy data to PostgreSQL on-premises.
- It is possible that GCP connectivity tools could make this seemless, but this would need to be investigated and tested.

BigQuery
~~~~~~~~

- No existing replication tools exist for BigQuery, as it is not compatible with the existing PPDB schema and data replication tools.
- A significant amount of development effort would be required to implement data replication from the APDB to BigQuery.
- This might take a much different form that the existing tools, as BigQuery is a fully managed service and does not support the same operations as a traditional database.
- For instance, data in Parquet format dumped from the APDB might be loaded into Google Cloud Storage, triggering an ETL process that loads the data into BigQuery, rather than using the streaming mechanisms in the current implementation.

Ecosystem and Community
-----------------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL is a flagship open source project with a large and active community.
- Its documentation is extensive and well-maintained, and there are many tutorials and support forums available.
- Many developers and companies use PostgreSQL, and there are a wide range of tools and libraries available that can be used to extend the functionality of the database platform.

Citus
~~~~~

- Citus is an open source project with a growing community.
- A complete set of documentation is available on the `Citus website <https://www.citusdata.com/>`_, and there are many tutorials and support forums available, including a dedicated `Slack workspace <https://slack.citusdata.com>`_.
- Though more limited than PostgreSQL, there are many developers and companies using Citus, and there are a range of tools and libraries available that can be used to extend the functionality of the database platform.
- Though more limited than PostgreSQL, this is probably not a significant limiting factor in terms of platform selection. The high quality of the documentation site in particular could be considered a significant advantage of using Citus.

Qserv
~~~~~

- As an in-house platform, Qserv has a limited ecosystem and community.
- Documentation is available on the `Qserv website <https://qserv.lsst.io/>`_, but it is not as extensive as that of PostgreSQL or Citus, nor is it complete.
- Qserv only has a few deployments, and there are no non-Rubin developers or companies using the platform.
- This could be considered a limiting factor in terms of platform selection.

.. few deployments
.. advantage of having in-house staff - we own the development; if we need something, we can add it

AlloyDB
~~~~~~~

- AlloyDB is a proprietary platform developed by Google, so its ecosystem and community are more limited than those of open source platforms like PostgreSQL and Citus.
- Documentation is available on the `Google Cloud website <https://cloud.google.com/alloydb>`_, but it is not as extensive as that of PostgreSQL or Citus.
- Support could be obtained through GCP support channels, if necessary.
- Though more limited than PostgreSQL and (likely) Citus, this is probably not a significant limiting factor in terms of platform selection.

BigQuery
~~~~~~~~

- BigQuery has a large and active community, with extensive documentation and tutorials available.
- Google Cloud Platform has a wide range of tools and libraries available that can be used to extend the functionality of BigQuery.
- Many developers and companies use BigQuery, and there are many support forums available, including the dedicated `BigQuery Slack workspace <https://cloud.google.com/blog/topics/inside-google-cloud/join-the-google-cloud-community-on-slack>`_.
- This is probably not a significant limiting factor in terms of platform selection, and the high quality of the available documentation and support could be considered a significant advantage of using BigQuery.

.. Performance
.. -----------

.. All of the database platforms should be able to meet the requirement of 20 simultaneous users.
.. For USDF-based PostgreSQL platforms, including a single server and Citus, a specific number of vCPUs would need to be allocated to meet the performance requirements.
.. PostgreSQL allocates a single process per connection, implying that nodes should be allocated at least 20 vCPUs to meet the requirement, and likely more to handle the overhead of the database, so 24 vCPUs is probably a reasonable estimate.
.. This is achievable on a single, dedicated node with commodity hardware; for example, 16 physical CPU cores with hyper-threading would translate to 32 vCPUs operating concurrently.
.. For a single PostgreSQL instance, an allocation of 24 vCPUs would be sufficient to meet the performance requirements in terms of simulataneous users, assuming 20 active connections with several processes dedicated to PostgreSQL overhead.
.. Similarily, for a Citus deployment, worker nodes would likely need to be allocated a similar number of vCPUs to meet the performance requirements as a single node, as full table scans across all shards would still be required and fairly common.
.. The Citus controller node would likely need to be allocated a similar number of vCPUs to handle the overhead of managing the worker nodes.
.. While 20 active queries is considered a minimum requirement, the actual number of queries will likely vary between being very low and very high, depending on the time of day and the number of users accessing the database.
.. Auto-scaling options would need to be considered in order to handle peak loads, as well as monitoring tools to track the number of active queries and the number of vCPUs in use.

.. AlloyDB is designed to scale out horizontally, so it should be able to meet the performance requirements in terms of simulataneous users.

.. Query response time is a more challenging requirement, as it is highly dependent on the complexity of the query and the size of the database.
.. A single node PostgreSQL instance would likely struggle to meet the 10 second query response time requirement given the expected data volume.
.. Cits would likely be able to meet the query response time requirement, as it is designed to scale out horizontally and should be able to handle the data volume and query performance requirements, though, again, this would be highly dependent on the complexity of the query.


Summary
=======

PostgreSQL
----------

- PostgreSQL is an attractive RDMS platform in general, due to its feature set, excellent documentation, and large community. Rubin and SLAC also have extensive experience with PostgreSQL, and the existing PPDB is implemented on this platform.
- Low development and maintenance effort would be required to implement the PPDB on PostgreSQL, as it has heretofore been the target platform for the PPDB implementation.
- However, PostgreSQL is not designed to scale out horizontally, and it is unlikely that a single node database could handle the data volume and query performance requirements of the PPDB.
- Given the data volume requirements, a single PostgreSQL server is not a suitable platform for the PPDB and can be eliminated as a viable option.

Citus
-----

- Citus brings with it all of the positive features of PostgreSQL, as it is an extension of the platform.
- Citus is designed to scale out horizontally, and it should be able to handle the data volume and query performance requirements of the PPDB.
- However, Citus has a very high maintenance overhead, as it requires regular monitoring, backup and recovery, and scaling to meet demand.
- Running Citus on-premises would require the development of Kubernetes operators or Helm charts, backup and recovery solutions, and other tools to manage the distributed database. This would necessitate a significant amount of development effort.
- A rough estimation is that at least one FTE could be required for the initial build out, testing, and deployment of Citus, and ongoing maintenance would require a significant fraction of a database administrator or similar expert.
- Given these factors, Citus is a viable option for the PPDB, but the maintenance overhead and effort required to develop configuration and monitoring tools would be considerable.

AlloyDB
-------

- AlloyDB has an attractive set of features built on top of PostgreSQL, including compatibility with the existing PPDB schema and data replication tools.
- AlloyDB is designed to scale out horizontally, via read replicas, and so it would perform better than a single node PostgreSQL instance.
- However, data volume requirements under the proposed scenarios would exceed the maximum storage capacity of AlloyDB, which is 128 TiB per primary instance.
- The inability of AlloyDB to scale to the required data volume makes it an infeasible choice for the PPDB.

Qserv
-----

- Qserv is a distributed database that is designed to scale out horizontally, and it should be able to handle the data volume and query performance requirements of the PPDB.
- Though developed in-house, Qserv has been used to host the DP and DR catalogs, and it is a proven platform for hosting large astronomical catalogs.
- However, Qserv would require very high developer effort to implement the PPDB, as it is missing many required features, including tooling to replicate data from the APDB.

.. Need to add AlloyDB and BigQuery

Conclusions
============

.. first, second, and third picks with caveats attached
.. 1. BigQuery
.. 2. Citus
.. 3. Qserv

Data retention of 2 years or more is the most challenging requirement for the PPDB.
Given that 2 years of operations is expected to result in 140 TB of table data, and that this data volume is expected to increase over time, it is likely that the PPDB will need to be implemented on a distributed database platform.
Single node databases like PostgreSQL are unlikely to be able to handle the data volume and query performance requirements given these datasets.
Though it has horizontal and vertical scaling options, AlloyDB has a hard maximum storage capacity of 128 TiB per primary instance, which would be insufficient.
In theory, Citus is a promising option for the PPDB, as it is designed to scale out horizontally and should be compatible with the existing PPDB schema and data replication tools which target PostgreSQL.
However, the maintenance overhead of managing a distributed database on-premises should not be underestimated.
Scaling, monitoring, and backup and recovery procedures will need to be carefully considered; it is likely that custom tooling would need to be developed to manage these aspects of the database.
BigQuery is not a particularly good fit for the PPDB in terms of software compatibility, as it is not supported by the existing PPDB schema and data replication tools, and would require a significant amount of work to migrate to this platform.
But it is worth noting that BigQuery is a fully managed service, with low maintenance overhead, and has excellent scalability along with good query performance.
Additionally, a TAP service has been implemented on top of BigQuery, which could be used to provide user access to the PPDB.
Particular costing options should be explored with Google Cloud Platform to determine the feasibility of using BigQuery.

Overall, there is no clear winner among the database platforms considered, though given the requirements and constraints, PostgreSQL and AlloyDB can be eliminated as options, as they cannot scale to the required data volume.
Qserv can handle the data volume and query performance requirements, but the maintenance overhead, as well as developer effort for new tooling and capabilities like makes it an infeasible choice.
Citus is an excellent option, but the maintenance overhead and effort required to develop configuration and monitoring tools would be considerable, on the order of 1 FTE for a database administrator.
BigQuery is a good fit in terms of scalability and query performance, but the developer effort required to migrate to this platform is significant, and the cost of running the service is unknown.
The final decision should likely involve a cost-benefit analysis of on-premises Citus versus BigQuery, including financial costs, developer effort, and maintenance overhead.
The decision of which platform to use will depend on the trade-offs between these factors, as well as the availability of personnel to manage the database and the cost of running the service.

.. _DMTN-113: https://dmtn-113.lsst.io
.. _DMTN-268: https://dmtn-268.lsst.io
.. _DMTN-293: https://dmtn-293.lsst.io

References
==========

.. bibliography::

.. Citus: no spatial sharding, hashes based on distribution column
.. cloud --> elastic support
..
