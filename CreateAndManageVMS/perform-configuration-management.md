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

#### Configure VMs using a configuration management tool such as Puppet or Chef; 

#### Enable remote debugging