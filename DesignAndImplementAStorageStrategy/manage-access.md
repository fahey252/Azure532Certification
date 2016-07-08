### Manage access
  * Secured via HTTPS, storage account keys, shared access keys - including __policies for keys__ and browser access.
  * You can use SAS tokens to __delegate access to storage account resources__ without sharing the account key.
  * Can generate a link to a __container, blob, table, table entity, file, or queue__.
  * Links
    - [Shared Access Signatures](https://azure.microsoft.com/en-us/documentation/articles/storage-dotnet-shared-access-signature-part-1/)

#### Generate shared access signatures - including client renewal and data validation
  * Only authenticated callers can access tables and queues. __Only blobs can optionally be exposed for anonymous access__.
  * To authenticate to any storage service, a __primary or secondary key__ is used, but this grants the caller__ access to all actions__ on the storage account.
  * SAS is used to delegate __access to specific storage account resources__ without enabling access to the entire account.  Includes expiration control lifetime, permissions, specific items.
  * SAS tokens are typically used to authorize access to the Storage Client Library.
  * Below code example is for blobs. Very similiar for queues and queues. Notice query parameter __sp__ for __rd__. Stand for read and delete.
  
  ```c#
  SharedAccessBlobPolicy sasPolicy = new SharedAccessBlobPolicy();
  sasPolicy.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(1);
  sasPolicy.SharedAccessStartTime = DateTime.UtcNow.Subtract(new TimeSpan(0, 5, 0));
  sasPolicy.Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Delete;
  string sasContainerToken = files.GetSharedAccessSignature(sasPolicy);

  // sample token:  ?sv=2014-02-14&sr=c&sig=B6bi4xKkdgOXhWg3RWIDO5peekq%2FRjvnuo5o41hj1pA%3D&st=2014-12-24T14%3A16%3A07Z&se=2014-12-24T15%3A21%3A07Z&sp=rd

  // access files
  StorageCredentials creds = new StorageCredentials(sasContainerToken);
  CloudBlobClient sasClient = new CloudBlobClient("https://<ACCOUNTNAME>.blob.core.windows.net/", creds);
  CloudBlobContainer sasFiles = sasClient.GetContainerReference("files");
  ```
  * __Renewal__: Should __limit the duration__ of an SAS token to limit access to controlled periods of time. Can __extend access by issuing new tokens__. 
  * __Validation__: Be sure to __validate system use__ of all resources exposed with SAS keys - specially for SAS with write permission. Don't leak keys.
  * Permission levels for containers:
    - __Full public read access__: Container and blob data can be read via anonymous request. Clients __can enumerate blobs__ within the container via anonymous request, but __cannot enumerate containers__ within the storage account.
    - __Public read access for blobs only__: Blob data within this container can be read via anonymous request, but __container data is not available__. Clients __cannot enumerate blobs__ within the container via anonymous request.
    - __No public read access__: Container and blob data can be __read by the account owner only__.

#### Create stored access policies
  * Stored access policies provide greater control over how you __grant access to storage resources__ using SAS tokens.
  * Can do the following after releasing an SAS token for resource access:
    - Change the __start and end time__ for a signature’s validity
    - Control __permissions__ for the signature
    - __Revoke access__
  * The stored access policy can be __used to control all issued SAS tokens__ that are based on the policy. You can update the policy without updating all the SAS tokens.
  * Use stored access policies wherever possible.
  * Note that when you __add an access policy to a container__, you must get the container's existing permissions, add the new access policy, and then set the container's permissions.
  * You can not add to the list of resources accessible by an SAS token. You can use the same stored access policy for multiple resources (such as multiple blobs, for example) but this is done at the time of producing the SAS token and associating the stored access policy to the token. __You cannot add resources at the policy level__.

#### Regenerate storage account keys
  * When you create a storage account, __two 512-bit storage access keys__ are generated for authentication to the storage account. This makes it possible to __regenerate keys without impacting application__ access to storage.
  * You typically use the __primary key when you first deploy applications__ that access the storage account. When time to update, update all applications to use secondary key, update primary, set applications back to primary and regenerate secondary.
  * Have a sound key management strategy. Applications should use  the primary key to facilitate regeneration process.
  * To reset keys, go to the portal > storage account > manage keys > click regenerate.

#### Configure and use Cross-Origin Resource Sharing (CORS)
  * Cross-Origin Resource Sharing (CORS) enables web applications running in the browser to __call web APIs that are hosted by a different domain__.
  * Blobs, tables, files, and queues __all support CORS__ to allow for access to the Storage API from the browser.
  * __CORS is disabled by default__ but can enable.
  * Note that a preflight request is __evaluated against the service__ (Blob, File, Queue, or Table) and __not against the requested resource__. The account owner must have enabled CORS as part of the account service properties in order for the request to succeed.
  * CORS rules are set at the service level, so you need to __enable or disable CORS for each service__ (Blob, File, Queue and Table) separately.
  * Try to avoid the use of CORS by using an alternate design if possible.
  * When using CORS, __generate an SAS token that will be included in any links__ for requesting the resource and limit the duration the token is valid either with a __short duration__ or a __stored access policy__ you can revoke when the user session ends.
  * Enable CORS in the service properties:

  ```xml
  <Cors>    
    <CorsRule>
      <AllowedOrigins>http://www.contoso.com, http://www.fabrikam.com</AllowedOrigins>
      <AllowedMethods>PUT,GET</AllowedMethods>
      <AllowedHeaders>x-ms-meta-data*,x-ms-meta-target*,x-ms-meta-abc</AllowedHeaders>
      <ExposedHeaders>x-ms-meta-*</ExposedHeaders>
      <MaxAgeInSeconds>200</MaxAgeInSeconds>
  </CorsRule>
<Cors>
  ```
  * Links
    - [Enabling CORS](https://msdn.microsoft.com/en-us/library/azure/dn535601.aspx)

