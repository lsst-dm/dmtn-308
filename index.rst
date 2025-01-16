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
These have covered performance of a PostgreSQL-based PPDB implementation, data replication from the APDB, and system architecture, respectively.
The exact database platform used to implement the PPDB has not been determined, and this note provides a comparison of the alternatives.

Requirements
============

Data Volume & Retention
-----------------------

The APDB is designed to retain data for a 1-year period.
While, ideally, the PPDB would retain data for the lifetime of the project, which is currently planned for 10 years, this may turn out to be infeasible.
Based on scheduling considerations of Data Release Processing (DRP), a data retention of 2 years will be considered as a minimum requirement.

The following table provides an estimate of the data volume for the PPDB over the course of the project.

.. list-table:: PPDB Data Volume Projects
   :header-rows: 1

   * - **Visit**
     - 190 MB
   * - **Nightly**
     - 190 GB
   * - **Yearly**
     - 70 TB
   * - **10 Years**
     - 700 TB

The exact size of the nightly data products which will be produced by LSSTCam is not yet determined.
Data taking during the `ComCam On-Sky Campaign <https://sitcomtn-149.lsst.io/>`_ resulted in an average size per visit of approximately 9 MB with 9 active detectors.
Extrapolating to the full camera with 189 detector results in a single visit size of *189/9 * 9 MB = ~190 MB*.
Since LSSTCam is expected to produce approximately 1000 visits per night, this would result in a nightly data volume of 190 GB.

These sizes are almost certainly underestimated, because ComCam data processing resulted in sparse data products with many `null` values; actual LSSTCam data products will likely be denser, and this density is expected to increase over time as algorithms are improved and more columns are filled.
Additionally, Solar System Processing (SSP) was not included in the ComCam data processing, and this is expected to increase the data volume by an unknown factor.

Deployment
----------

Two options exist for the deployment of the PPDB: on-premises at the US Data Facility (USDF) or in the cloud.
Cloud deployments would use Google Cloud Platform (GCP), which has been used for the Interim Data Facility (IDF) and already hosts many Rubin services.
The USDF provides Kubernetes clusters for deploying services, and the IDF provides a similar service on GCP.
The USDF has a dedicated team of system administrators at SLAC who manage the infrastructure.

Query Performance & Latency
---------------------------

Query performance requirements for the PPDB are covered by *DMS-REQ-0355* in the `Data Management System Requirements <https://ls.st/LSE-61>`_.
These specify that the minimum number of simulataneous users should be 20, and that the maximum query time should be 10 seconds.
(Query latency is highly dependent on the complexity of the query and the size of the database, so this requirement may not be satisfiable for all possible queries.)
The PPDB is expected to be used by a large number of users, and the query latency should be low enough to provide a responsive user experience.
Given the expected data volumes, longer queries may be necessary to extract the desired information, but the system should be able to handle multiple queries concurrently.
We will use these metrics as a baseline for evaluation.

Scalability
-----------

The PPDB should be able to scale to meet the data volume and query performance requirements.
This includes the ability to scale out horizontally to handle large data volumes and high query loads.
The system should be able to handle the expected data volume of 700 TB over 10 years, as well as the expected query load of 20 simultaneous users.

Operating Cost
--------------

We consider only the overhead of running the database platform itself, not development or maintenance costs.
Nor are possible fixed costs such as hardware purchases considered.
For on-premises deployment, it will be assumed that costs are already covered by existing infrastructure and budget.
Cloud deployments will include the variable costs of running the database platform on GCP.

Cost Predictability
-------------------

As a general rule, cloud deployments are less predictable in terms of cost than on-premises deployments.
The cost of running a database on GCP can vary depending on the amount of data stored, the number of queries run, and the amount of data transferred.
Though an on-premises deployment could incur hardware purchase costs, these are fixed, and overhead is generally covered by lab budgets and existing infrastructure.

Maintenance Overhead
--------------------

Large databases require a significant amount of maintenance to keep running smoothly.
This includes monitoring, backup and recovery, and scaling the database to meet demand.
On-premises deployments require administrators to manage the infrastructure, while cloud deployments are managed by the cloud provider.
With modern devops methods, maintenance and development effort may overlap significantly, so this particular requirement is more about the amount of time and effort required to keep the database running smoothly rather than producing "configuration as code."

Developer Effort
----------------

In addition to the database platform, the PPDB will require a TAP service to provide user access to the database.
Some of the database platforms do not have existing TAP implementations.
Additionally, the PPDB will require data replication from the APDB, which is currently implemented as a `COPY` operation in PostgreSQL.
The existing tools for data replication may not be compatible with all of the database platforms under consideration, so new tools may need to be developed.
Deployment and monitoring tools will need to be developed to manage the database platform, and these tools may need to be custom-built for the specific platform.
Finally, on-premises deployments may require configuration of the underlying infrastructure, such as Kubernetes clusters, to support the database platform.

TAP Service
-----------

Special consideration is given to the availability of a TAP service for the database platform.
The PPDB will require a TAP service to provide user access to the database, and this service will need to be developed and maintained.
The CADC has implemented a TAP service on top of PostgreSQL, which has been used for Rubin services.
The availability of a TAP service for the database platform will be a significant factor in the decision of which platform to use.

Data Replication
----------------

The system must be able to handle the ingestion of nightly data from the APDB.
This is currently implemented as a `COPY` operation in PostgreSQL using the `ppdb-replication` command line tool in the `dax_ppdb repository <https://github.com/lsst/dax_ppdb>`_.
When discussing data replication, we will only consider whether the target platform can support the existing data replication tools, rather than the performance of the replication itself, as this is especially difficult to estimate without a specific implementation, which does not exist for several of the platforms under consideration.

Ecosystem and Community
-----------------------

The ecosystem and community around the database platform are important factors to consider.
This includes availability of documentation, tutorials, and support forums, as well as the number of developers and companies using the platform.
A large ecosystem and community can provide valuable resources and support for developers, as well as a wide range of tools and libraries that can be used to extend the functionality of the database platform.

Database platforms
==================

Given the requirements above, we consider the following database platforms for the PPDB implementation:

PostgreSQL
----------

PostgreSQL is the current database platform used for the PPDB.
The `dax_ppdb <https://github.com/lsst/dax_ppdb>`_ repository contains command-line tools and APIs for creating the database schema in PostgreSQL from its `Felis representation <https://github.com/lsst/sdm_schemas/blob/main/python/lsst/sdm_schemas/schemas/apdb.yaml>`_ and replicating data from the APDB.
It has been used in numerous system tests at USDF and is well understood by the team.

Citus
-----
Citus is an open source extension that transforms PostgreSQL into a distributed database.
It is designed to scale out horizonally across multiple workers which are queried and managed by a controller instance.
Because Citus is an extension of PostgreSQL, it should be compatible with the existing PPDB schema and data replication tools.

Google AlloyDB for PostgreSQL
-----------------------------
AlloyDB is a distributed database that is compatible with PostgreSQL.
Though it has an on-premises version, it is primarily designed to run on Google Cloud Platform.
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

Platform Comparison
===================

The following table provides a comparison of the database platforms based on the requirements outlined above.

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

- According to its `published limits <https://www.postgresql.org/docs/current/limits.html>`_ , PostgreSQL has a maximum table size of 32 TB, which is insufficient for the 700 TB of data that will be generated over 10 years, and likely also insufficient for the 140 TB of data that will be generated over 2 years.
- Though PostgreSQL theoretically supports unlimited database size by using partitioning, practical constraints such as query performance degradation, index management overhead, and maintenance tasks (e.g., vacuum and analyze) make it impractical for datasets with a magnitude of hundreds of terabytes.
- Backup and restore operations for large datasets (e.g., > 100 TB) become increasingly time-consuming and operationally challenging.
- Overall, a single PostgreSQL instance cannot scale to the data volume requirements of the PPDB.

Citus
~~~~~

- Citus is designed to scale out horizontally to multiple petabytes, so it should be able to handle the data volume requirements of the PPDB.

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

PostgreSQL
~~~~~~~~~~

- PostgreSQL can be deployed on-premises at the USDF, where it is currently used for the PPDB.
- The USDF provides Kubernetes clusters for deploying services, and the PPDB could be deployed on these clusters.

Citus
~~~~~

- Citus can be deployed on-premises at the USDF.
- No standard Kubernetes operators or Helm charts seem to exist for Citus, or at least none are listed on the `Citus website <https://www.citusdata.com/>`_. These would need to be developed to deploy Citus on Kubernetes at the USDF.

Qserv
~~~~~

- Qserv is already deployed on-premises at the USDF.
- PPDB could be deployed on the same infrastructure as Qserv, and the same team of system administrators could manage both services.

Query Performance & Latency
---------------------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL has medium latency for small to medium datasets, typically ranging from milliseconds to a few seconds for indexed queries. However, it struggles with datasets larger than 10-20 TB on a single instance due to high I/O and memory constraints.
- Performance degrades with high concurrency or large joins across large tables.
- Index maintenance and vacuum operations can impact performance on large datasets.

Citus
~~~~~

- Citus has high query performance for large datasets, as it is designed to scale out horizontally across multiple nodes. Sub-second performance can be achieved for most queries distributed across worker nodes.
- With proper sharding and indexing, Citus can achieve high query performance for large datasets.

Qserv
~~~~~

- Qserv has high query performance for large datasets, as it is designed to scale out horizontally across multiple nodes.

AlloyDB
~~~~~~~

- AlloyDB has low latency, with sub-millisecond response times for cached queries.
- Read replicas can improve query scalability.

BigQuery
~~~~~~~~

- BigQuery can have high latency for small queries (seconds to tens of seconds), but it is very efficient for large-scale analytical queries on petabyte-scale data.
- The serverless nature of the platform requires that a full query execution environment is initialized for every query, which includes allocating and provisioning of resources, as well as optimization and planning across the distributed resources.
- Caching mechanisms and optimization techniques can be used to improve query performance. For instance, BigQuery can cache results of queries for up to 24 hours, which can significantly reduce query latency for repeated queries.

Scalability
-----------

PostgreSQL
~~~~~~~~~~

- PostgreSQL can scale vertically to a certain extent, but it is not designed to scale out horizontally.
- While PostgreSQL can be used in a master-slave configuration for read scaling, it is not designed to scale out horizontally across multiple nodes.

Citus
~~~~~

- Citus is designed to scale out horizontally across multiple nodes, so it should be able to handle the data volume and query performance requirements of the PPDB.

Qserv
~~~~~

- Qserv is designed to scale out horizontally across multiple nodes, so it should be able to handle the data volume and query performance requirements of the PPDB.

AlloyDB
~~~~~~~

- AlloyDB uses a primary and replica setup, with the primary node handling writes and the replica nodes handling reads. This allows AlloyDB to scale out horizontally to multiple nodes.
- AlloyDB does not sufficiently scale in terms of storage capacity, as it has a (previously mentioned) maximum storage capacity of 128 TiB per primary instance.

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

Maintenance Overhead
--------------------

PostgreSQL
~~~~~~~~~~

- PostgreSQL has medium maintenance overhead, as it requires regular monitoring, backup and recovery, and scaling to meet demand.
- On-premises deployments require administrators to manage the infrastructure, including monitoring, backup and recovery, and scaling the database to meet demand.
- SLAC has a dedicated team of system administrators who manage the infrastructure at the USDF. This includes administration of a PostgreSQL development cluster for prompt processing.
- `CloudNativePG <https://cloudnative-pg.io/>`_ has been used by USDF to deploy PostgreSQL on Kubernetes, and this could be used to deploy the PPDB. This provides a suite of tools for managing PostgreSQL on Kubernetes, including monitoring, backup and recovery, and scaling.
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
- AlloyDB does support the `PostGIS extension <https://postgis.net/>`_, which provides support for geospatial data. However, this does not provide the same functionality as PgSphere. Significant development effort would be needed to implement the required functionality for the TAP service using a PostGIS backend. And it is not clear that this would be possible.
- Additionally, the TAP service would realistically need to be run on GCP, which is certainly possible, but would require additional development effort.

BigQuery
~~~~~~~~

- BigQuery is not compatible with the CADC TAP implementation, so a TAP service would need to be developed.
- Work has been done in the past to implement a TAP service on top of BigQuery (see `TAP and ADQL on Google’s BigQuery Platform <https://assets.pubpub.org/rynkboj6/71582749259388.pdf#abs287.02>`_), but the status of this implementation and the location of the source code is unknown and would need to be investigated.


Replication
-----------

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


Summary & Conclusions
=====================

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

