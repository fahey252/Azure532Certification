### Create and manage a VM image or virtual hard disk
  *  A VM image __includes an operating system and all disks attached__ to a virtual machine when the image is created. VM Image encompasses the full definition of a virtual machine’s storage, containing the OS disk and all data disks. 
  * A VM Image is a collection of metadata and pointers to a set of VHDs (one VHD per disk) stored as page blobs in Azure Storage.
  * There are two types of VM Images – __generalized__ and __specialized__.
  * __Generalized__ VM Image contains an OS disk, which, as the name suggests, has been generalized.  (for Windows, you have run Sysprep and for Linux, you have executed ‘waagent –deprovision’) and needs to be __provisioned during deployment time__.   Used as a “model” to quickly stamp out similar virtual machines.
  * A __specialized__ VM Image contains an OS disk, which is already provisioned. Used as a __“snapshot”__ to deploy a VM to a good known point in time. Not meant to be a mechanism to clone multiple identical virtual machines. In a virtual network and with computers linked to a directory, it is an issue if multiple computers have the same name.
  * If you want to use an image to create more than one virtual machine, you’ll need to prepare it for use as an image by generalizing it.
  * Need to use Sysprep for image replication.
  * __VHD vs Image__: VHD is a virtual space for storing your data or system for your Virtual Machines. VM uses VHD to store data in it. Image: It captures all the data that virtual machine will include. All the data, disks system is stored in image.
  * Links
  	- [VM Image](https://azure.microsoft.com/en-us/blog/vm-image-blog-post)

####Create specialized and reusable images
  * Once you have a virtual machine set up and configured as you want, you can capture the instance as a VM Image.
  * During capture, no in-memory state is saved.
  * Naming convention: `<VM Image Name>-os-YYYY-MM-DD<-ZZ>` and `<VM Image Name>-datadisk-<Lun>-YYYY-MM-DD(-ZZ)`. ZZ is only added as the number in case of collision.
  * If the OS has been generalized/deprovisioned, the virtual machine __must be shut down__ in order to capture it as a VM Image. Once the VM has been captured as a VM Image, the virtual machine will __automatically be deleted__.
  * If the OS is specialized, the virtual machine can be captured while it is running or shut down.
  * If the VM Image is specialized, no provisioning information is needed.
  * When a virtual machine is deployed from a VM Image, a copy of the VHDs are made for the new VM. The existing VHDs are not attached directly.
  ```powershell
  Save-AzureVMImage –ServiceName “myServiceName” –Name “myVMtoCapture” –OSState “Generalized” –ImageName “myAwesomeVMImage”
  Get-AzureVMImage  # get all images
  Get-AzureVMImage | select ImageFamily
  New-AzureQuickVM –Windows –Location “West US” –ServiceName “MySvc1” –Name “myVM1” –InstanceSize “Medium” –ImageName “myAwesomeVMImage” –AdminUsername “admin”–Password “adminPassword123” -WaitForBoot		#  using the VM Image ‘myAwesomeVMImage’
  Remove-AzureVMImage –ImageName "MyOldVmImage"
  ```

  *The `OSState` parameter is required if you want to create a VM image, which includes data disks as well as the operating system disk.
  * Images are available from several sources: Marketplace (more options for MSDN, pay-as-you-go subscribers), VM Depot, can store and use your own images in Azure storage, by either capturing an existing Azure virtual machine for use as an image or uploading an image.
  * To create a Linux image, depending on the software distribution, you’ll need to run a set of commands (Linux image with the waagent command instead of Sysprep.) that are specific to the distribution, as well as run the __Azure Linux Agent__.
  * Can create an __Azure Resource Manager template__ that deploys and provisions all of the resources in a single, coordinated operation
  * Doesn't include networking configurations, so you'll need to configure those when you create the other virtual machines that use the image.
  * To create an image from the portal, on the VM is stopped, click the Capture button, give it a name, check the I have run Sysprep checkbox. The new image is available and can be created from the gallery.
  * If you don't select the "I have run Sysprep checkbox", a specialized image is created instead of a generalized image.
  * Generalizing system is same like resetting machine identifiers (computer name and system identifier (SID)) and preparing it for reuse in another system.
  * Be aware that when you create a custom image, the image is stored in the storage account you created the VM in.
  * Links
  	- [VM Image](https://azure.microsoft.com/en-us/blog/vm-image-blog-post/)
  	- [About images for Virual Machines](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-classic-about-images/)
  	- [Introduction To Azure VM Image And VHD](http://www.c-sharpcorner.com/UploadFile/42ddd2/introduction-to-azure-vm-image-and-vhd-azure-deep-dive-c/)
  	- [Managing and Updating Custom Azure VM Images](http://michiel.vanotegem.nl/2015/02/managing-and-updating-custom-azure-vm-images/)

####Prepare images using SysPrep and Windows Agent (Linux)
  * Assumes installed a supported Linux operating system to a virtual hard disk. Multiple tools exist to create .vhd files such as Hyper-V.
  * Newer __VHDX format is not supported__ in Azure. You can convert the disk to VHD format using Hyper-V Manager or the `convert-vhd` cmdlet.
  * Azure __does not support uploading dynamic VHDs__, so you need to convert such disks to static VHDs before uploading.
  * Prepare the image to be uploaded (or use an Endorsed Distribution.).  
  * Make sure you are using the Azure CLI in the classic deployment model (`azure config mode asm`), then log in.
  * Will need a storage account to upload your VHD file to.
  ```bash
  $ azure vm image create <ImageName> --blob-url <BlobStorageURL>/<YourImagesFolder>/<VHDName> --os Linux <PathToVHDFile>
  ```

  * Before you upload the VHD to Azure, it needs to be generalized by using the Sysprep tool. Open a command prompt window as an administrator. Change the directory to `%windir%\system32\sysprep`, and then run `sysprep.exe`.
  * System Preparation Tool, select __Enter System Out of Box Experience (OOBE)__ and make sure that __Generalize is checked__.
  ```powershell
  Add-AzureVhd -Destination "<BlobStorageURL>/<YourImagesFolder>/<VHDName>.vhd" -LocalFilePath <PathToVHDFile>
  ```
  * Images includes the OS disk and the data disks that are attached to the virtual machine. It __doesn't include the virtual network resources__ that you'll need to create a Windows VM, so you'll __need to set those up__ before you create another virtual machine that uses the image.
  * Please note that the virtual machine cannot be logged in via RDP once it is generalized, since the process removes all user accounts. This is an irreversible change.
  ```powershell
  Set-AzureRmVm -ResourceGroupName YourResourceGroup -Name YourWindowsVM -Generalized

  Save-AzureRmVMImage -ResourceGroupName YourResourceGroup -VMName YourWindowsVM -DestinationContainerName YourImagesContainer -VHDNamePrefix YourTemplatePrefix -Path Yourlocalfilepath\Filename.json

  ```

  * Need to set the status of the virtual machine to Generalized. Note that you will need to do this because the generalization step above (`sysprep`) does not do it in a way that Azure can understand.
  * The __Azure Resource Explorer (Preview)__ is a new tool that you can use to manage Azure resources that have been created in the Resource Manager deployment model
  * 

  * Links
  	- [Creating and Uploading a Virtual Hard Disk that Contains the Linux Operating System](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-classic-create-upload-vhd/)
  	- [How to capture a Windows virtual machine in the Resource Manager deployment model](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-capture-image/)

###Copy images between storage accounts and subscriptions
  * Copy for doing backups, because your Windows Azure trial is ending, to get a copy of a client’s VM to investigate a problem.
  * You can only create a VM in the region where the image is stored. If you need it in other regions as well, you need to copy or recreate it there.
  ```bash
  $ azure vm disk upload <source-path> <target-blob-url> <target-storage-account-key>
  ```

  * Set the container with a __public access__ at the time of the transfer. You can use either the __primary or the secondary access key__.
  * Ability to copy VHDs (or any other blob for that matter) is built natively into the Windows Azure platform through the __Blob Copy API__.
  * Storage lives in various data centers throughout the world and within each data center WA Storage can be further divided into __multiple storage stamps__ (clusters of servers and disks). The location and stamp placement of your source and destination storage accounts matters greatly when using the blob copy operation. You can tell if storage accounts are in the same storage stamp if they share the same IP Address
  ```powershell
  $destContext = New-AzureStorageContext –StorageAccountName $storageAccount -StorageAccountKey $storageKey

  New-AzureStorageContainer -Name $containerName -Context $destContext

  blob1 = Start-AzureStorageBlobCopy -srcUri $srcUri -DestContainer $containerName -DestBlob "testcopy1.vhd" -DestContext $destContext

  ### Retrieve the current status of the copy operation ###
  $status = $blob1 | Get-AzureStorageBlobCopyState 
  ```

  * __AzCopy__ is a Windows command-line utility designed for copying data to and from Microsoft Azure Blob, File, and Table storage. AzCopy is not available for Mac/Linux OSs. (add to path: `%ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy`)
  * Can do all kinds of operations with AzCopy - download all files, upload files with pattern, modify files once uploaded etc.
  ```powershell
  AzCopy /Source:<source> /Dest:<destination> [Options]

  # download a file:
  AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:"abc.txt"

  # Copy single blob across Storage accounts
  AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt

  ```

  * Links
  	- [How to copy blobs or VHDs between different Windows Azure subscription](http://matricis.com/en/technical-article/how-to-copy-blobs-or-vhds-between-different-windows-azure-subscription/)
  	- [Transfer data with the AzCopy Command-Line Utility](https://azure.microsoft.com/en-us/documentation/articles/storage-use-azcopy/)

###Upload VMs
  * HD size must be fixed and a whole number of megabytes, i.e. a number divisible by 8.
  * Azure can accept images only for generation 1 virtual machines that are saved in the VHD file format. Convert VHDX's with Hyper-V or Convert-VHD PowerShell cmdlet.
  * Run `Sysprep` or waagent to prepare the machine. Make sure you have a storage account setup.
  * `Login-AzureRmAccount`. Make sure that you are using the right subscription by using `Set-AzureRmContext -SubscriptionId "xxxx-xxxx-xxxx-xxxx"`
  * 
  ```powershell
  # VHD (classic)
  Add-AzureVhd -Destination "<BlobStorageURL>/<YourImagesFolder>/<VHDName>.vhd" -LocalFilePath <PathToVHDFile>

  # VHD Resource Manager
  # Destination like: https://YourStorageAccountName.blob.core.windows.net
  Add-AzureRmVhd -ResourceGroupName YourResourceGroup -Destination "<StorageAccountURL>/<BlobContainer>/<TargetVHDName>.vhd" -LocalFilePath <LocalPathOfVHDFile>

  ```
  * Links
  	- [Upload a Windows VM image to Azure for Resource Manager deployments](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-upload-image/)