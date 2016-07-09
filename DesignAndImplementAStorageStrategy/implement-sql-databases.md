### Implement SQL databases
  * PaaS offering.

#### Choose the appropriate database tier and performance level
  * This tiered pricing is called Service Tiers. Determines __stroage space__ and performance__. Three service tiers to choose from. Higher tiers give more predictability in performance.
  * A __DTU is a blended measure of CPU, memory, disk reads, and disk writes__. Select tier/level based on: __CPU Percentage, Physical Data Reads Percentage, Log Writes Percentage__
  * SQL Database is a__ shared resource with other Azure customers__, sometimes performance is not stable or predictable.
    - __Basic tier__: light workloads, __one performance__ level. Good for small use, new projects, testing, development, or learning.
    - __Standard tier__: most production online transaction processing (OLTP) databases. The performance is more predictable than the basic tier. Four performance levels under this tier, levels __S0 to S3__.
    - __Premium tier__: Performance is typically measured in seconds. Basic tier can handle 16,600 transactions per hour. The standard/S2 level can handle 2,570 transactions per minute. The top tier of premium can handle 735 transactions per second. That translates to 2,645,000 per hour in basic tier terminology.
  * Can __change service tiers at any time__ with minimal downtime to your application.
  * Manage multiple databases by creating an __elastic database pool__.
  * __Same features per tier__. Each tier has a 99.9 percent uptime SLA, backup and restore capabilities, access to the same tooling, and the same database engine features.
  * __Monitor tab to see the current load__ of your database and decide whether to scale up or down.
  * If you reach __80 percent__ of your performance metrics, it’s time to consider increasing your service tier or performance level. Less than __10 percent__, scale down. Be sure to consider spikes.
  * In client application code you should watch:
    - __Connection resiliency__, because you could failover to a replica.
    - __Transaction resiliency__ so you can resubmit a transaction in the event of a failover.
    - __Query auditing__ so you can baseline your current query times and know when to scale up the instance automatically.

#### Configure and perform point in time recovery
  * Azure SQL Database does a __full backup every week__, a __differential backup each day__, and an __incremental log backup every five minutes__.
  * Incremental log backup allows for a point in time restore, which means the database can be restored to any specific time of day. If accidently delete table, can restore with little data loss.
  * Depending on your service tier, you will have different backup retention periods. __Basic retains backups for 7 days, standard for 14 days, and premium for 35 days__.
  * Can restore a database from source to destination and by s__electing a point in time to restore (slider)__ in the portal.

#### Enable geo-replication
  * SQL Database subscription has built-in redundancy. Three copies of your data are stored __across fault domains__ in the datacenter to protect against server and hardware failure.
  * Configure two more fault-tolerant options: __standard geo-replication and active geo-replication__.
  * __Standard geo-replication__: fail over the database to a different region when a database is not available. Standard and premium service tiers. Does not allow clients to connect to the secondary server. It is offline until it’s needed to take over for the primary. The source and target servers must belong to the same subscription.
  * To create an offline secondary database, go to the Geo-Replication blade. 
  * For __upgrades__, you can terminate the relationship and then upgrade the primary database to a different schema to support a software upgrade. The __secondary database gives you a rollback option__.
  * __Active (online) geo-replication__: four secondary copies of your database, premium tier only, hybrid: 3 can be active and 1 offline/secondary. Can make one database read only or put secondary copies in different regions closer to users.
  * To create an online/active secondary prerequisites: must have __same name, separate servers, same subscription, same performance level__.
  * Creating an online or offline secondary can be done with Windows PowerShell.

  ```powershell
  Start-AzureSqlDatabaseCopy -ServerName "SecondarySrv" -DatabaseName "Flashcards" -PartnerServer "NewServer" –ContinuousCopy  # active/online

  Start-AzureSqlDatabaseCopy -ServerName "SecondarySrv" -DatabaseName "Flashcards" -PartnerServer "NewServer" –ContinuousCopy -OfflineSecondary   # offline
  ```

#### Import and export data and schema
  * For backup/restore between environments, archives.  Portal > Export > Supply: Name, Subscription, storage account, container, username/password.
  * Will __create a BACPAC file__ that can be used to create a database in another SQL server.
  * Import: Choose import, select a server, Service Tier, performance level and subscription.
  * Import process is faster if you use the standard service tier and at least the S2 performance level.

#### Scale SQL databases
  * You can easily scale out Azure SQL databases using the __Elastic Database tools__.  
  * Tools:
    - __Elastic Database client library__: allows you to create and maintain sharded databases.
    - __Elastic Database split-merge__ tool: moves data between sharded databases. 
    - __Elastic Database jobs__ (preview): manage large numbers of Azure SQL databases. Easily perform administrative operations such as schema changes.
    - __Elastic Database query__ (preview): run a Transact-SQL query that spans multiple databases.
    - __Elastic transactions__: allows you to run transactions that span several databases in Azure SQL Database.
  * Use the tools because difficult to scale. Managing __hotspots that may arise affecting a specific subset of data__ – such as a particularly busy end-customer (tenant).
  * Distributing data and processing across many identically-structured databases (a scale-out pattern known as "sharding") provides an alternative to traditional scale-up approaches both in terms of cost and elasticity.
  * Application may use __horizontal scaling (more databases/sharding)__ to provision new end-customers and __vertical scaling__ to allow each end-customer’s database to grow or shrink resources as needed by the workload.
  * Horizontal scaling is managed using the Elastic Database client library.
  * Vertical scaling is accomplished using Azure PowerShell cmdlets to change the service tier, or by placing databases in an elastic pool.
  * __Sharding reasons__: Too much __data__, too many __transactions__, customers must be __separated__, may need to reside in different geographies for __compliance__, performance or geopolitical reasons.
  * Sharding works best when every transaction in an application __can be restricted to a single value of a sharding key__. That ensures that all transactions will be local to a specific database. i.e. day of week.
  * __Single tenant sharding pattern__: specific tenant ID value (or customer key value), application’s responsibility to route each request to the appropriate database.
  * __multi-tenant sharding pattern__: application manages large numbers of very small tenants, rows in the database tables are all designed to carry a key identifying the tenant ID or sharding key.
  * Can __move data from multi-tenant to single tentant __i.e. shared trail to real customer. Use the __split-merge tool to move the data__ from the multi-tenant to the new single-tenant database.
  * Links
    - [Scaling out with Azure SQL Database](https://azure.microsoft.com/en-us/documentation/articles/sql-database-elastic-scale-introduction/)


