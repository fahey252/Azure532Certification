###Design and implement VM storage
  * The operating system disk is created from an image, and both the __operating system disk and the image are actually virtual hard disks (VHDs) stored in an Azure storage account__. Virtual machines also can have one or more data disks, that are also stored as VHDs.
  * The temporary disk is __automatically created for you__ for page and swap files only. On Windows virtual machines, this disk is labeled as the __D: drive__ and it used for storing `pagefile.sys``. On Linux virtual machines, the disk is typically __/dev/sdb__ and is formatted and mounted to __/mnt/resource__ by the __Azure Linux Agent__.
  * Azure creates an operating system disk when you create a virtual machine from an image. If you use an image that includes data disks, Azure also creates the data disks when it creates the virtual machine.
  * You can use a VHD that you’ve uploaded or copied to your storage account, or one that Azure creates for you. Attaching a data disk associates the VHD file from your storage account with the virtual machine, by placing a __‘lease’ on the VHD so it can’t be deleted from storage while it’s attached to a virtual machine__. Before you can delete a source .vhd file, you’ll need to __remove the lease by deleting the disk or image__.
  * The VHDs used in Azure are __.vhd files stored as page blobs__ in a __standard or premium storage__ account in Azure.
  * Azure supports VHD format, __fixed disks__. The fixed format lays the logical disk out __linearly__ within the file, so that disk offset X is stored at blob offset X. A small footer at the end of the blob describes the properties of the VHD. Often, the fixed format wastes space because most disks have large unused ranges in them. However, Azure stores .vhd files in a __sparse format, so you receive the benefits of both the fixed and dynamic__ disks at the same time. 
  * Source disks or images are read-only. Once copied can be read-only or read-and-write.
  * We have three main ways to store data in the Azure cloud – Files, Blobs and Disks.
  * Links
    - (About disks and VHDs for Azure virtual machines)[https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-about-disks-vhds/]

#### Configure disk caching
  * Each disk you attach to a VM has a host cache preference setting for managing a local cache (__reduce storage transaction costs by averting a read or write to Azure Storage__).
  * Local cache does not live within your VM instance; it is __external to the VM and resides on the machine hosting your VM__.
  * Cache options: __None__, __Read__ (read from Azure Storage and are then cached in local cache. Writes go directly to Azure Storage), __Read/Write__ (same as read but Writes go to the local cache and at some later point (determined by algorithms of the local cache)to Azure Storage).
  * Default is set to __Read/Write for operating system disks and None for data disks__.
  * The host caching __setting of None is not valid for operating system disks__.
  * Can configure the host cache preference when you create and attach an empty disk to a VM or __can change host cache settings after created__.
  * Portal: Use the __Host Cache Preference toggle __to configure the cache setting.

#### Plan for storage capacity
  * Storage capacity for VMs is dictated by the scalability limits __(IOPS, throughput, and maximum file size)__. Per-VM limits that adjust with the VM size (the number of VHD disks that can be attached - generally __2 per core)__.
  * A data disk is a VHD that’s attached to a virtual machine to store application data. Each data disk has a __maximum capacity of 1023 GB__.
  * VMs leverage a local disk provided by the host machine for the __temp drive (D) __and __Azure Storage for the operating system and data disks__, wherein each data disk is a VHD stored as a __blob in Blob storage__.
  * Local disk is shared among all the VMs running on the host. Can cause competition for read/write IOPS and bandwidth.
  * Performance (for example, IOPS and read/write throughput MB/s) and size (such as in GBs) is governed by the capacity of a single blob in Blob storage. __Maximum throughput of 60 MB/s, size of 1TB, 500 IOPS for Standard tier instances, 300 IOPS for Basic tier instance__.
  * An IOPS is a unit of measure counting the __number of input/output operations per second__ and serves as a useful measure for the number of read, write, or read/write operations that can be completed in a period of time for data sets of a certain size (usually 8 KB).
  * To get better performance, use multiple blobs, which means using multiple disks striped into a single volume. In Windows Server 2012 and later VMs, the approach is to use __Storage Spaces and create a storage pool__ across all of the disks.
  * General rule governing the __number of disks you can attach is twice the number of CPU cores__. Maximum number of disks that can currently be mounted to a VM is 16 (i.e. 16 terabytes of storage per VM).
  * Needs close to 8,000 IOPS during peak loads? Need VM with 16 disks (500 IOPS max per disk.) in an storage pool with VM size of A4, A7, A8, or A9.
  * The __size of the virtual machine__ determines __how many data disks__ you can attach to it and the __type of storage__ you can use to host the disks.

#### Configure storage pools
  * __Storage Spaces__ enables you to group together a set of disks and then create a volume from the available aggregate capacity. Create a storage pool from those disks, then create a storage space in that pool, and from that storage space, mount a volume you can access with a drive letter.
  * __Create a storage pool__: On the VM > Server Manager > File And Storage Services > Storage Pools > New Storage Pool > Select all the disks you want to include > Create.
  * Next, __create a new virtual disk using that pool__: Server Manager > Server Pools > select New Virtual Disk. VHDs are already __triple replicated by Azure Storage__, you do not need additional redundancy.
  * When the __New Virtual Disk Wizard closes, the New Volume Wizard__ appears: Assign drive letter and name. The new data disk will list the Partition as Unknown. Right-click the disk and select Initialize.
  * Applications running within your VM can use the new drive and benefit from the __increased IOPS and total storage capacity__ that results from having multiple blobs backing your multiple VHDs grouping in a storage pool.
  * D-series of VMs provide your VM an SSD drive mounted at D:\..
  * Valid replication option for the Azure Storage account that stores the VHD blobs: __Locally Redundent__. Geo-redundant, Read-access geo-redundant replication yields __corrupted disks__ if you try to restore from a replicated file.
  * __Zone redundant replication can only be used with block blobs__. VHDs are created using __page blobs__.

#### Configure shared storage using Azure File service
  * Azure File storage enables you to use network shares to provide __distributed access to files from your VMs.__
  * Local network on-premises to access files on a remote machine through a Universal Naming Convention (UNC) path like `\\server\share`, or if you have mapped a drive letter to a network share, you will find Azure File storage familiar.
  * Azure File storage enables your __VMs to access files using a share located within the same region__ as your VMs. Agnostic of storage account and subscription.
  * Azure File storage provides support for most of the __Server Message Block (SMB)__. Storage supports a subset of SMB - no named pipes, short file names like FAHEYC~1.txt. Azure File storage is compatible with __SMB 2.1__, not SMB 3.0.
  * Helpful for shared application configurations, common tools, log file centralization etc.
  * Azure File storage is built upon the same underlying infrastructure as Azure Storage.
  * Cannot mount shares across regions (even if you set up VNET-to-VNET connectivity) or access shares from your on-premises resources (if you are using a point-to-site or site-to-site VPN with a VNET). __Can connect from on-premisis if you are using ExpressRoute__.
  * __Requires an Azure Storage account__. Access is controlled with the __storage account name and key__.
  * REST APIs available through endpoints named `https://<accountName>.file.core.windows.net/<shareName>`` and through the SMB protocol.
  * Your web application can upload the files through the REST API to the share, but your back-end applications running on a VM can process those files by accessing them using a network share.
  * You may have to create a new Azure Storage account to be able to use Azure File storage. May not be available within older Azure Storage accounts.
  
  ```powershell
  $ctx = New-AzureStorageContext <Storage-AccountName> <Storage-AccountKey>
  $s = New-AzureStorageShare <ShareName> -Context $ctx
  New-AzureStorageDirectory -Share $s -Path testdir
  ```
  * To access the share within a VM, you mount it to your VM. So available across restarts, add your __Azure Storage account credentials to the Windows Credentials Manager__.
  
  ```cmd
  cmdkey /add:<Storage-AccountName>.file.core.windows.net /user:<Storage-AccountName> /pass:<Storage-AccountKey>
  net use z: \\<Storage-AccountName>.file.core.windows.net\<ShareName> /Persistent: YES
  net use   # to verify it was added
  ```
  * How to get your files and folders into that share: 
    - __RDP__: As a part of configuring your RDP session, you can __mount the drives from your local machine__ so that they are visible using Windows Explorer in the remote session. Copy and paste.
    - __AZCopy__: recursively upload __directories and files__
    - A__zure Powershell__: cmdlets to upload or download a __single file at a time__.  `Get-AzureStorageFileContent` and `Set-AzureStorageFileContent`
    - __Storage Client Library__: Application in .NET, convenience layer atop the REST API. `Microsoft.WindowsAzure.Storage.(CloudFileDirectory|CloudFile)`
    - __REST APIs__
  * Note that currently you cannot access files within a share using the existing tools commonly used for browsing Azure Storage blobs
  * Links
    - (Introducing Microsoft Azure File Service)[https://blogs.msdn.microsoft.com/windowsazurestorage/2014/05/12/introducing-microsoft-azure-file-service/]

#### Configure geo-replication
  * Can leverage geo-replication for blobs to maintain replicated copies of your VHD blobs in __multiple regions__ around the world in addition to three copies that are maintained within the datacenter.
  * If you configure the virtual machine for geo-replication, your VHD is also replicated to different sites more than 400 miles apart
  * Geo-replication __should not be used for Azure Storage accounts that store VHDs__ because the added redundancy does not provide additional protection against corrupting the data and may in fact result in data loss if you attempt to restore from a geo-replication.
  * Geo-replication is not synchronized across blob files. Writes could be replicated out of order. As a result, __if you mount the replicated copies__ to a VM, the disks will almost certainly be __corrupt__.
  * Configure the disks to use __locally redundant replication__ which does not add any additional availability and reduces costs (since geo-replicated storage is more expensive).

