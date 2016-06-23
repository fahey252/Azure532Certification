### Implement Azure Storage blobs and Azure Files

#### Read data
  * In a blob storage account, you can have many containers. Containers are similar to folders in that you can use them to logically group your files.
  * Each blob storage account can store up to __500 terabytes__ of data.
  * `http://<storage account name>.blob.core.windows.net/<container name>/<blob name>`
  * Create a container in the portal > Storage Account > Containers tab. Setup permissions for __Private__, __Public Container__, or __Public Blob__. Cannot list blobs in the container without authentication, but you can navigate to the blob URL, if you have it, and read it anonymously. Can change at anytime.
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
  * __Pages__: comprised of __512-byte__ pages, 1TB total size max. Page blob writes are __done in place__ and are __immediately committed__ to the file. 

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
#### Configure Content Delivery Network (CDN)
#### Design blob hierarchies
#### Configure custom domains
#### Scale blob storage
#### Implement Azure Premium storage