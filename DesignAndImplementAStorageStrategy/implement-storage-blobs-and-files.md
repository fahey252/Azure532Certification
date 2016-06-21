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

#### Set metadata on a container
#### Store data using block and page blobs
#### Stream data using blobs
#### Access blobs securely
#### Implement async blob copy
#### Configure Content Delivery Network (CDN)
#### Design blob hierarchies
#### Configure custom domains
#### Scale blob storage
#### Implement Azure Premium storage