### Implement Azure Storage blobs and Azure Files

#### Read data
  * In a blob storage account, you can have many containers. Containers are similar to folders in that you can use them to logically group your files.
  * Each blob storage account can store up to __500 terabytes__ of data.
  * `http://<storage account name>.blob.core.windows.net/<container name>/<blob name>`
  * Create a container in the portal > Storage Account > Containers tab. Setup permissions for __Private__, __Public Container__, or __Public Blob__. Cannot list blobs in the container without authentication, but you can navigate to the blob URL, if you have it, and read it anonymously. Can change at anytime.
  * When set to Public Blob, only blobs can be accessed without credentials if the full URL is known.
  * To access your storage account, need account url and access key. Always use the __primary key for management activities__. 
  * Upload files to blob storage: AzCopy, Storage API with HTTP requests, __Storage Client Library__, which wraps the Storage API into a language and platform-specific library, powershell, Server Manager in Visual Studio 2013.
  
  ```cmd
  $ cd C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy
  $ AzCopy /Source:c:\test /Dest:https://myaccount.blob.core.windows.net/mycontainer2 /DestKey:key /Pattern:*.txt.
  ```

#### Change data
  * Any updates made to a blob are __atomic__. While an update is in progress, requests to the blob URL will always __return the previously committed version of the blob until the update is complete__.
  * For an application, in your app.config file, create a storage configuration string and entry for the account name and key, obtain the Microsoft.WindowsAzure.Storage.dll, 

  ```c#
  var storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["StorageConnectionString"]);

  CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
  CloudBlobContainer container = blobClient.GetContainerReference("files");
  container.CreateIfNotExists();
  container.SetPermissions(new BlobContainerPermissions {PublicAccess BlobContainerPublicAccessType.Blob });
  CloudBlockBlob blockBlob = container.GetBlockBlobReference(“myblob”); 
  
  // use the FileStream object to access the stream, and upload the file
  using (var fileStream = System.IO.File.OpenRead(@”path\myfile”))
  {
    blockBlob.UploadFromFileStream(fileStream);
  }
  ```
  * Can also ready block/page/directory blobs, download and delete blobs with similiar helper methods. 

#### Set metadata on a container
  * Blobs and containers have metadata attached to them
    - __System Properties__: _influence_ how the blob behaves. Access with __Properties__ member of the container.
    - __User Defined__: set of name/value pairs that your _applications can use_. Access with __Metadata__ member of the conatiner reference .
  * A __container has only readonly__ system properties, while blobs have both read-only and read-write properties.
  * Containers also have system properties and user-defined metadata.
  
  ```c#
  // setting user defined metadata
  CloudBlobContainer container = blobClient.GetContainerReference("files");
  files.Metadata["counter"] = "100";
  files.SetMetadata();
  ```
  * Blob metadata: read and write, valid HTTP headers, limited to __8 KB__ for the combination of name and value pairs, If the metadata key __doesn’t exist, an exception is thrown__.
  
  ```c#
  // getting system defined metadata
  CloudBlobContainer container = blobClient.GetContainerReference("files");
  Console.WriteLine("LastModifiedUTC: " + container.Properties.LastModified);
  Console.WriteLine("ETag: " + container.Properties.ETag);
  ```

#### Store data using block and page blobs
  * __Block__ blobs are great for streaming data __sequentially__ (videos). __Page__ blobs are great for __non-sequential__ reads and writes, like the VHD on a hard disk. Will mainly use block blobs.
  * __Blocks__: stored as a series of blocks, __up to 4MB each__. Block blobs can be __up to 200GB__. Blocks can be uploaded at random and assembled to a final blob later. After a week, an blocks not assoicated with a blob will be deleted.
  * __Pages__: comprised of __512-byte__ pages, __1TB total__ size max. Page blob writes are __done in place__ and are __immediately committed__ to the file. 
  * Page files are not faster for streaming files, but are very good for random I/O files like VHDs.
  * Block blobs have a maximum size of 200 GB. Page blobs can be 1 terabyte.

#### Stream data using blobs
  * Can stream blobs by downloading to a stream using the __DownloadToStream()__ API method.

  ```c#
  CloudBlockBlob blockBlob = container.GetBlockBlobReference("photo1.jpg");

  using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
  {
    blockBlob.DownloadToStream(fileStream);
  }
  ```

#### Access blobs securely
  * Azure Storage supports both HTTP and secure HTTPS requests. Can Authenticate 3 ways:
  * __Shared Key__: Constructed from a __set of fields related to the request__. Computed with a SHA-256 algorithm and encoded in Base64.
  * __Shared Key Light__: Like shared key but compatible with __previous versions of Azure Storage__. 
  * __Shared Access Signature__: Grants restricted access rights to containers and blobs, for users use don't want to give storage account keys. grant them __specific permissions to the resource for a specified amount of time__.
  
  ```c#
  string accountName = "ACCOUNTNAME";
  string accountKey = "ACCOUNTKEY";

  CloudStorageAccount storageAccount = new CloudStorageAccount(new StorageCredentials(accountName, accountKey), true);
  ```

#### Implement async blob copy
  * Can copy item in parrallel - copy to new URI, overwrite to same URI (replace, overwrite metadata, removed uncommitted blocks), __snap shot to base blob__, __copy new snapshot to new location; making it writable__.
  * Copy operation is always the entire length of the blob; you __can’t copy a range__.
  * A storage account can have multiple Copy Blob operations processing in parallel; however, an __individual blob can have only one pending copy operation__.

  ```HTTP
  PUT https://myaccount.blob.core.windows.net/mycontainer/myblob
  ```
  * The source blob for a copy operation may be a block blob, an append blob, or a page blob, or a snapshot.
  * Links
      - [Copy Blob](https://msdn.microsoft.com/en-us/library/azure/dd894037.aspx)

#### Configure Content Delivery Network (CDN)
  * Distributes content across geographic regions to edge nodes across the globe. Allows users to download static content with lower latency.
  * Blobs have a default __seven-day time-to-live (TTL)__ at the CDN edge node.
  * Must support __anonymous access__.
  * To enable the CDN for a storage account: App Services > Quick Create > select storage account to enable CDN > Create > CDN endpoints > 
  * Can take __60 minutes before the CDN is ready__ for use on the storage account.

  ```
  http://<your CDN subdomain>.vo.msecnd.net/<your container name>/<your blob path>
  https://<your domain>/<your container name>/<your blob path>
  ```

#### Design blob hierarchies
  * Storage account name, container/partitioning, blob name - can contain (/) to simulate folder structure.
  * You can use a blob naming convention akin to folder paths to create a __logical hierarchy for blobs__, which is useful for __query operations__.

  ```
  var list = images.ListBlobs(null, true, BlobListingDetails.All);
  var list = images.ListBlobs("en", true, BlobListingDetails.All);  // prefixed path/blob names with "en"

  //i.e. https://solexpstorage.blob.core.windows.net/images/en/bmp/logo.bmp
  ```

#### Configure custom domains
  * Default URl for blobs is `https://<your account name>.blob.core.windows.net`. Can map own __custom domain or subdomain__ to your storage account.
  * Two ways to point your custom domain to the blob endpoint for your storage account.
    - Create a __CNAME__ record mapping your custom domain and subdomain to the blob endpoint.
    - Process of mapping custom domain __can cause downtime__. If no downtime is required, can use the Azure intermediary __asverify__ subdomain to provide an intermediate registration step so that users will be able to access your domain while the DNS mapping takes place.
  * By creating a CNAME that points from `asverify.<subdomain>.<customdomain>`to `asverify.<storageaccount>.blob.core.windows.net`, you can pre-register your domain with Azure.
  * The asverify subdomain is a special subdomain recognized by Azure. Permit Azure to recognize your custom domain without modifying the DNS record for the domain. Create a new CNAME record in your domain name register, and provide a subdomain alias that includes the `asverify` subdomain.
  * CNAME format `asverify.www` or `asverify.photos` Then provide a host name, which is your Blob service endpoint, in the format `asverify.mystorageaccount.blob.core.windows.net`
  * At this point, your custom domain has been verified by Azure, but traffic to your domain is not yet being routed to your storage account. To complete the process, return to your DNS registrar's website, and create another CNAME record that maps your subdomain to your Blob service endpoint.
  * Can delete the CNAME record you created using asverify, as it was necessary only as an intermediary step.
  * Can unregsiter domains by going to __Manage Custom Domains__ and choosing unregister.
  * Links
    - [Configure a custom domain name for your Blob storage endpoint](https://azure.microsoft.com/en-us/documentation/articles/storage-custom-domain-name/)

#### Scale blob storage
  * Blobs are partitioned by __container name and blob name__. Can be distributed across many servers to scale access even though they are __logically grouped within a container__.
  * Avoid sudden spikes in the rate of traffic and ensure that traffic is well-distributed across partitions.
  * Can build your application to use multiple storage accounts, and partition your data objects across those storage accounts.
  * Every object that holds data that is stored in Azure Storage (blobs, messages, entities, and files) belongs to a partition, and is __identified by a partition key__.
  * Partition determines how Azure Storage load balances blobs, messages, entities, and files across servers to meet the traffic needs of those objects.
  * Entities that are in the same table but have different partition keys can be load balanced across different servers, making it possible to have greater scalability.
  * Every object that holds data that is stored in Azure Storage (blobs, messages, entities, and files) belongs to a partition, and is __identified by a partition key__.
    - _Blobs_: partition key for a blob is account name + container name + blob name.
    - _Files_: partition key for a file is account name + file share name
    - _Messages_: partition key for a message is the account name + queue name
    - _Entities_: partition key for an entity is account name + table name + partition key
  * Links
    - [Storage Scalability Targets](https://azure.microsoft.com/en-us/documentation/articles/storage-scalability-targets/)

#### Implement Azure Premium storage/Working with Azure File storage
  * VMs and cloud services as a mounted share, and applications can use the File Storage API to access File storage via SMB 2.1.
  * Any number of Azure virtual machines or roles can mount and access the File storage share simultaneously.

