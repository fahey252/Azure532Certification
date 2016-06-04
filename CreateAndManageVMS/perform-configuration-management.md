### Perform Configuration Management

#### Automate configuration management by using PowerShell Desired State Configuration (DSC) and VM Agent (custom script extensions); 
  * DSC enables deploying and __managing configuration data for software__ services and managing the __environment__ in which these services run. Used for configuration, deployment, and management of systems.
  * Declaratively specify how you want your software environment to be configured.
  * Can manage things such as registry settings, processes/services, environment variables etc.
  * __Local Configuration Manager__ (LCM) will continue to ensure that machines are configured in whatever state the configuration declares. If the system is out of state, the LCM uses more logic inside of the resources to __“make it so”__ according to the Configuration declaration.
  * `PSDesiredStateConfiguration` module (including `Start-DscConfiguration`, `Set-DscLocalConfigurationManager`, and `Get-DscResource`)
  * DSC configurations are PowerShell scripts that define a __special type of function__. To define a configuration, you use the PowerShell keyword `Configuration`. Save the file as a .ps1
  ```powershell
  Configuration MyDscConfiguration {
    Node "TEST-PC1" {  # (computers or VMs) that you are configuring

    	# resource blocks. This is where the configuration sets the properties for the resources
        WindowsFeature MyFeatureInstance {
            Ensure = "Present"
            Name =    "RSAT"
            DependsOn = "[Group]GroupExample"
        }}}
  ```

  * Before you can enact a configuration, you have to compile it into a __MOF document__. A MOF file will be created for each node.
  * __DependsOn__ specifies which resources depend on other resources, and the LCM ensures that they are applied in the correct order, regardless of the order in which resource instances are defined.
  * Desired State Configuration (DSC) __Resources provide the building blocks__ for a DSC configuration. A resource __exposes properties that can be configured__ (schema) and contains the PowerShell script functions that the Local Configuration Manager (LCM) calls to "make it so". A resource can model something as generic as a file or as specific as an IIS server setting.
  * The LCM runs on every target node, and is responsible for parsing and enacting configurations that are sent to the node.
  * You can use the Azure PowerShell DSC cmdlets to upload and apply a PowerShell DSC configuration to an Azure VM by __enabling the VM Agent__ and the PowerShell DSC extension.
  * The __VM Agent__ is a light weight process intended to bootstrap additional solutions. The status of the VM and its extensions can be viewed and managed from a single location.
  * __VM Extensions__ are software components that extend the VM functionality and simplify various VM management operations. Turns features on and off.
  ```powershell
  # sets the BGInfo extension on a VM (helpful information on desktop)
  Get-AzureVM –ServiceName –Name | Set-AzureVMBGInfoExtension | Update-AzureVM

  ```

  * Enabling VM extensions has one simple pre-requisite – create an IaaS VM by enabling the VMAgent. VMAgent is enabled by default.
  * VMAccess extension provides a mechanism for the user to get back on the VM by resetting the password and also user name when locked out.
  * __Custom Script Extensions__ can automatically download scripts and files from Azure Storage and launch a PowerShell script on the VM which in turn can install additional software components. Just like with any other VM Extension, this can be added during VM creation or after the VM has been running.
  ```powershell
  $vm = Set-AzureVMCustomScriptExtension -VM $vm -ContainerName $container -FileName 'start.ps1'
  New-AzureVM -ServiceName $servicename -Location $location -VMs $vm
  ```

  * Links
  	- [Windows PowerShell Desired State Configuration Overview](https://msdn.microsoft.com/en-us/powershell/dsc/overview)
  	- [DSC Configurations](https://msdn.microsoft.com/en-us/powershell/dsc/configurations)
  	- [Configuring the Local Configuration Manager](https://msdn.microsoft.com/en-us/powershell/dsc/metaconfig)
  	- [Automating VM Customization tasks using Custom Script Extension](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-classic-extensions-customscript/)

#### Configure VMs using a configuration management tool such as Puppet or Chef
  * Puppet Enterprise, you can easily manage, configure and provision your Windows infrastructure. Offered in the Microsft Azure Marketplace. Can only run a Puppet master on a Linux image. Open __port 8140__ for puppet agents. Can only use for Classic deployment model.
  * One of the problems Puppet solves is __configuration drift__. Over time, the configuration of a host under manual management tends to wander, specifically as the gap grows between __how you think it’s configured, how your colleagues think it’s configured and how it’s actually configured__. The wider that gap, the more often you'll get unexpected issues and failed deployments. A better approach is "infrastructure as code." The desired state of Azure VMs can be described in Puppet.
  * Chef is a great tool for delivering automation and desired state configurations. Creating a __policy or “CookBook”__ and then deploying this cookbook to an Azure virtual machine.
  * The __Chef Server__ is the management point and there are two options for the Chef Server: a hosted solution or an on-premises solution. The __Chef Client (node)__ is the agent that sits on the servers you are managing. The __Chef Workstation__ is our admin workstation where we create our policies and execute our management commands. We run the knife command from the Chef Workstation to manage our infrastructure.
  * Create a directory called C:\chef. Put the Azure publish settings in this directory.
  ```powershell
  > puppet agent --configprint server.
  azure_vm { 'sample':   
    location => 'eastus'  
    image    => 'canonical:ubuntuserver:14.04.2-LTS:latest',   
    user     => 'azureuser',   
    password => 'Password',    
    size     => 'Standard_A0',   
  }
  chef gem install knife-azure ––pre
  knife azure image list
  chef generate cookbook webserver
  knife cookbook upload webserver

  ```
  * Links
    - (Automating Azure virtual machine deployment with Chef)[https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-chef-automation/]

#### Enable remote debugging
  * You can debug these errors if you enable remote debugging when you publish your service and then attach the debugger to a role instance. The emulator simulates the Azure Compute service and runs in your local environment so that you can test and debug your cloud service before you deploy it.
  * Required services (__msvsmon.exe__, for example) are installed on the virtual machines that run your role instances. If you didn't enable remote debugging when you published the service, you __have to republish the service with remote debugging enabled__.
  * Can enable __IntelliTrace__ for any roles in that service that target the .NET Framework.
  * May adversely affect users if you enable remote debugging on the Production environment.
  * Select the __Enable Remote Debugger for all roles__ check box on the Advanced Settings.
  * In Server Explorer, expand the node for your cloud service. Open the shortcut menu for the role or role instance to which you want to attach, and then click __Attach Debugger__.
  * You can debug programs that run on Azure virtual machines by __using Server Explorer__ in Visual Studio. When you enable remote debugging on an Azure virtual machine, Azure installs the remote debugging extension on the virtual machine. Then, you can attach to processes on the virtual machine and debug as you normally would.
  * Virtual machines created through the Azure resource manager stack can be remotely debugged by using __Cloud Explorer in Visual Studio 2015__. Enabled Debugging options then choose __Attached to Process__.
  * __Azure Diagnostics__ to log detailed information from code running within roles, whether the roles are running in the development environment or in Azure.

