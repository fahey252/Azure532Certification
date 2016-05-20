### Deploy Workloads On VMs
  * Azure has two different deployment models for creating and working with resources: __Resource Manager (modern) and classic__.
  * Need to maintain the virtual machine (IaaS) -- configuring, patching, and maintaining the software that runs on the virtual machine.
  * VM in Azure has an __operating system, storage and networking capabilities__ and can run a wide variety of applications. You can use an image provided by Azure or one of it's partners, or use your own.
  * Virtual machines use __virtual hard disks (VHDs)__ to store their operating system (OS) and data. VHDs are also used for the images you can choose from to install an OS.
  * If a physical server running a VM fails, Azure notices this, moves the VM to new hardware and restarts the VM. This process is sometimes called __service healing__. Azure also protects a virtual machine's data, by keeping redundant copies of the VHDs in blob storage
  * Links
  	- [About Linux virtual machines in Azure](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-about/)

#### Identify workloads that can and cannot be deployed
  * __Affinity groups__ are a legacy concept for a __geographical grouping__ of a customer’s cloud service deployments and storage accounts within Azure. Affinity groups are deprecated in the Azure Resource Manager.
  * Virtual networks are also at a regional scope, so an affinity group is no longer required when you use a virtual network.
  * Each data disk can be __up to 1 TB__. The number of data disks you can use depends on the size of the virtual machine.
  * Azure only supports fixed, VHD-format virtual hard disks.
  * Azure VM's are similiar to Hyper-V VM's but aren't the same. Both types provide virtualized hardware, and the VHD-format virtual hard disks are compatible. This means you can move them between Hyper-V and Azure. However, no console access, 1 virtual network adapter (1 public IP address) in an Azure VM.
  * Can use Azure Virtual Network to extend your existing infrastructure via VPN, branch office. 
  * Connect to the virtual machine by using Remote Desktop Connection for a Windows VM or a Secure Shell (SSH) for a Linux VM.
  * You __shouldn’t use the temporary disk__ (the D: drive by default for Windows or /dev/sdb1 for Linux) to store data. When the virtual machine moves to a different host (resizing, failure), temp data is lost.
  * __Don’t attempt to upgrade__ the guest OS while it resides on Azure. It isn’t supported because of the risk of losing access to the virtual machine. If you need to migrate the server using something like the Windows Server Migration Tools.
  * When you create virtual machine, you'll need to set the username and password. You can install and use the __VMAccess__ extension to fix login problems.
  * Linux VMs can run `sudo` commands. Root is disabled. Windows users are added to the Administrators group.
  * Azure offers several options for anti-virus solutions, but it’s up to you to manage it.
  * Azure Backup is available as a preview in certain regions. For backup, an additional option is to use the snapshot capabilities of blob storage. VM needs to be shutdown so file system is in a consistent state.
  * Azure charges an hourly price based on the VM’s size and operating system and for some pre-installed software. Azure __charges separately for storage for the VM’s operating system and data disks__. Temporary disk storage is free.
  * You are charged when the VM status is Running or Stopped, but you are __not charged when the VM status is Stopped (De-allocated)__.
  * Azure sometimes restarts your VM as part of regular, planned maintenance updates in the Azure datacenters.
  * Standalone VM (meaning the VM isn’t part of an availability set), an email is sent to administrator one week before planned maintenance.
  * To provide redundancy, put two or more similarly configured VMs in the same availability set. This helps ensure at least one VM is available during planned or unplanned maintenance.
  * Can view VM Reboot Logs for information on why a reboot happened.
  * __Classic__
  	- Also known as the __Service Management model__.
  	- Created from the Classic portal. Can also be created through the Preview Portal but not recommended or 
  	```powershell
  	Get-AzureDeployment
  	```
  	- If the resource was __created through classic deployment, you must continue to operate on it through classic operations__.
  * __Resource Manager__
  	- Deploys compute, storage and network as one logical unit, resource group.
  	- Created through the Preview Portal or:
  	```powershell
  	Get-AzureRmResourceGroupDeployment
  	```

  	```bash
  	$ azure config mode arm
  	```
  	- Resource Manager added the concept of the resource group. Every resource you create through Resource Manager exists within a resource group. Manage as a group instead individually.
  	- You can apply access control to all resources in your resource group because __Role-Based Access Control (RBAC)__
  	- Can apply __tags__ to resources to logically organize all of the resources in your subscription. You cannot apply tags to classic resources.
  	- In general, you __should not expect resources created through classic deployment to work with Resource Manager commands__. Only some _might_ work.
  * Virtual machines deployed with the classic deployment model cannot be included in a virtual network deployed with Resource Manager.
  * Virtual machines deployed with the Resource Manager deployment model __must be included in a virtual network__. Classic VM's don't have to be in a VNET.
  * Links
  	- [Frequently asked question about Azure Virtual Machines](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-classic-faq/)
  	- [Azure Resource Manager vs. classic deployment](https://azure.microsoft.com/en-us/documentation/articles/resource-manager-deployment-model/)

#### Run workloads including Microsoft and Linux
  * The Azure platform SLA applies to virtual machines running the Linux OS only when one of the endorsed distributions is used.
  * The VHDX format is not supported in Azure, only __fixed VHD__
  * All of the VHDs must have sizes that are multiples of 1 MB. Typically, VHDs created using Hyper-V should already be aligned correctly. 
  * The __Azure Linux Agent (waagent)__ is required to properly provision a Linux virtual machine in Azure.
  * Do not create swap space on the OS disk
  * Links
  	- [Information for Non-Endorsed Distributions](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-create-upload-generic/)

#### Create VMs
  * Quickly deploy a Linux Virtual Machine on Azure using the Azure CLI.
  ```bash
  $ azure config mode arm  # needs run at least once.
  $ azure vm quick-create
  # to attached your ssh key
  $ azure vm quick-create -M ~/.ssh/azure_id_rsa.pub -Q CoreOS
  ```

  * Can use other distrobution aliases such as RHEL, Debian, UbuntuLTS
  * Can SSH into your VM on the default SSH port 22 and the fully qualified domain name (FQDN)
  ```bash
  $ ssh ops@rhel-westu-1630678171-pip.westus.cloudapp.azure.com
  ```

  * __VM Agent__: provides an environment for extensions to be installed that can help with interacting or managing the virtual machine.
  * To create a VM with Powershell, see link below as there are many steps. Outline of steps:
  	1. Create a resource group
  	2. Create a storage account
  	  * needed to store the virtual hard disk that is used by the virtual machine
  	3. Create a virtual network
  	4. Create a public IP address and network interface
  	5. Create a virtual machine
  	6. You should see the resource group and all its resources in the Azure portal and a success status
  * Links
  	- [Create your first Windows virtual machine in the Azure portal](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-hero-tutorial/)
  	- [Create a Linux VM on Azure using the CLI](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-quick-create-cli/)
  	- [Create a Windows VM using Resource Manager and PowerShell](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-ps-create/)