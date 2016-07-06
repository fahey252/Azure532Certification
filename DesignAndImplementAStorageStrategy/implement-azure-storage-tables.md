### Implement Azure storage tables
  * Azure Storage Tables is a __non-relational (NoSQL)__ entity storage service
  * Accessed via `http://<storage account name>.table.core.windows.net/<table name>`
  * Many forms of NoSQL databases:
    - Key-Value pairs: key per record and often allow for _jagged_ entries where __each row might not have a complete set of values__
    - Documents: semi-structured, easy-toquery documents. Usually stored in JSON.
    - Columnar: organize large amounts of distributed information
    - Graph: do not use columns and rows, use a graph model for storage and query.
  * Table storage is a key-value store that uses a __partition key__ to help with scale out distribution of data and a row key for unique access to a particular entry.
  * The partition key is used to logically group rows that are related; the row key is a unique entry for the row.
  * The Table service uses the partition key for distributing collections of rows across physical partitions in Azure to automatically scale out the database as needed.
  * Table Storage can use Zone redundant storage, Read access geo-redundant storage and Geo-redundant storage. Transactional replication is for SQL server only.

#### Implement CRUD with and without transactions
  * Access table storage programmatically.
  * Add keys to `app.config`

  ```XML
  <appSettings>
    <add key=”StorageConnectionString” value=”DefaultEndpointsProtocol=https;AccountName=<your account name>;AccountKey=<your account key>” />
  </appSettings>
  ```
  * Install Azure Storage .dlls via NuGet: `Install-Package windowsazure.storage –version 3.0.3`

  ```c#
  var storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["StorageConnectionString"]);

  CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
  CloudTable table = tableClient.GetTableReference("customers");
  table.CreateIfNotExists();

  TableOperation insertOperation = TableOperation.Insert(someEntity);
  table.Execute(insertOperation);

  TableBatchOperation batchOperation = new TableBatchOperation();  // batch operations
  batchOperation.Insert(newOrder1);
  batchOperation.Insert(newOrder2);
  table.ExecuteBatch(batchOperation);

  // query
  TableQuery<OrderEntity> query = new TableQuery<OrderEntity>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Lana"));

  // upsert
  TableOperation insertOrReplaceOperation = TableOperation.InsertOrReplace(updateEntity);

  // delete - get entity first, then delete.
  TableOperation deleteOperation = TableOperation.Delete(deleteEntity);
  table.Execute(deleteOperation);
  ```
  * To add __entries to a table__, you create objects based on the __TableEntity__ base class and __serialize__ them into the table using the Storage Client Library.
  * Use the __partition key, row key, timestamp, ETag__ in the base class to interact with entities.
  * An entity can __appear only once__ in the transaction, and only __one operation__ may be performed against it.
  * The _combination of partition key and row key_ must be unique within the table. This combination is used for _load balancing and scaling_, as well as for __querying and sorting__ entities.
  * You can group inserts and other operations into a single __batch transaction__. Must take place on the __same partition__, at most __100 entities__, total batch payload size __cannot be greater than 4 MB__.
  * You can batch transactions that belong to the same table and partition group for insert, update, merge, delete, and related actions programmatically.
  * You can select all of the entities in a partition or a range of entities by partition and row key. Should try to __query with the partition key and row key__. Querying entities by other properties does not work well because it __launches a scan of the entire table__.
  * Within a table, entities are __ordered within the partition key__.
  * Within a partition, __entities are ordered by the row key__.
  * If you are using a __date value for your RowKey__ property use the following order: year, month, day i.e. __20160704__.

#### Query using OData
  * Table storage __does not support anonymous access__, so you must supply credentials using the account key or a Shared Access Signature (SAS).
  * `https://myaccount.table.core.windows.net/Tables` - list of all tables created.
  * `https://myaccount.table.core.windows.net/Tables('MyTable')` - to get a table by name.
  * `https://myaccount.table.core.windows.net/MyTable()` - get all entities in a table - notice no Partition/RowKey.
  * `https://<your account name>.table.core.windows.net/<your table name>(PartitionKey='<partition-key>',RowKey='<row-key>')?$select=<comma separated property names>` - query entities. Query results are sorted by PartitionKey, then by RowKey. Ordering results in any other way is not currently supported.
  * The result is limited to __1,000 entities per request__, and the query will run for a __maximum of five seconds__.
  * Table service REST API supports __ATOM and JSON__ as OData payload formats. You __must use JSON with version 2015-12-11 and later__. ATOM is supported for versions earlier than 2015-12-11.
  * A request that returns more than the default maximum or specified maximum number of results __returns a continuation token for performing pagination__.
  * The PartitionKey and RowKey properties form an entity's primary key.
  * `https://myaccount.table.core.windows.net/Customers()?$top=10` - example using selectors such as __$top, $select, $filter__.
  * Links
    - [Quering Tables and Entities](https://msdn.microsoft.com/en-us/library/azure/dd894031.aspx)

#### Design, scale and manage partitions
  * To handle massive amounts of structured data and billions of records, tables are partitioned.
  * Table service will spread your table to multiple servers and key all rows with the __same partition key co-located__.
  * Three types of partition keys:
    - __Single__: one partition key for the entire table. __Good for batch operations__ as all entities must be part of same partition. __Bad for scaling__ because must be on same server.
    - __Multiple__: Might put each partition on its __own server__, making it easier for Azure to load balance. Might make __partitions slow__ as they grow in size and may make further partitioning necessary.
    - __Unique__: Many small partitions but __does not work with batching__.
  * For query performance, you should use the partition key and row key together when possible - for exact match.
  * Next best thing is to have an exact partition match with a row range so dont have to scan the entire table.
  * Partition keys can be unique, but only in rare circumstances. Use the same partition key for all records only in very small entity sets. Batches take place only in the same partition. Find an even way to split entities so that you have relatively even partition sizes.
  * Operations that affect more than one partition can run in parallel.
  * 3 methods for partitioning. Can use several methods together at the same time.
    - __Horizontal (sharding)__: Each partition is its own data source with same schema. Spreads work accross many servers.
    - __Vertical__: Subset of fields, broken up columns - frequently accessed fields held together. Limits the amount of I/O, separate frequently changing data with constant data, separate credit card #'s, from secruity verification numbers.
    - __Functional__:  Data aggregated based on how it is used in the system - i.e. customer data held separatly from inventory, read-write data from read only data for reporting purposes.
  * Links
    - [Data Partitioning Guidance](https://azure.microsoft.com/en-us/documentation/articles/best-practices-data-partitioning/)
