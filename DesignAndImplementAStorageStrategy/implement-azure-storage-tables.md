### Implement Azure storage tables
  * Azure Storage Tables is a __non-relational (NoSQL)__ entity storage service
  * Accessed via `http://<storage account name>.table.core.windows.net/<table name>`
  * Many forms of NoSQL databases:
    - Key-Value pairs: key per record and often allow for _jagged_ entries where __each row might not have a complete set of values__
    - Documents: semi-structured, easy-toquery documents. Usually stored in JSON.
    - Columnar: organize large amounts of distributed information
    - Graph: do not use columns and rows, use a graph model for storage and query.
  * Table storage is a key-value store that uses a __partition key__ to help with scale out distribution of data and a row key for unique access to a particular entry.

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

#### Query using OData (page 265)
  * 

#### Scale tables and partitions
#### Design and manage partitions