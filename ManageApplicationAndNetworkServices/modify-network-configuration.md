### Modify network configuration
  * After creating a virtual network, you are able to modify the network configuration. Add DNS, point-to-site connectivity, VNet space, add subnets, subnet space, prefixes (i.e. -AddressPrefix '10.0.1.0/28').
  * Can modify through Portal or powershell.
  * __If you run out of IP addresses__, you can either add IP addresses to an existing subnet, or you can add add subnets.
  * Should use a shared DNS server so names are resolved consistenly.

#### Modify a subnet
  * You __cannot modify or delete a subnet after services or VMs have been deployed to it__. You __can modify prefixes__ as long as the subnets containing existing VMs or services are not affected by the change.
  * __Can not expand existing subnets__.
  * Can add address space and subnets in the portal via the Virtual Network blade.
  * If you find it necessary to reorganize your network topology and the servers running within each subnet, you __can move existing VMs or cloud services from one subnet to another__.

  ```powershell
  Get-AzureVM –ServiceName TestVMCloud –Name TestVM | 
  Set-AzureSubnet –SubnetNames Subnet-2 | 
  Update-AzureVM    # restarts the VM
  ```

  * To move a cloud service role from one subnet to another, __edit the service configuration file (.cscfg)__.

#### Import and export network configuration
  * Can export your virtual network settings to a __network configuration file__ (XML).
  * Useful in situations where you accidentally modify or delete your network or when you want to __reproduce a similar environment with the same topology__.  (__Recovery__ operation or reproducing.)
  * Export the file via the portals in the Network section > Export.
  Export includes:
    - The __IP addresses of DNS servers__ to use for name resolution
    - The __IP address space and IP subnet__ definitions
    - The __local (on-premises)__ network site name and IP address space.
    - Your __VPN gateway__ IP address
    - The __region__ that you want to associate with your virtual network
  * Can __create a network configuration file manually__, or you can export it from an existing virtual network configuration. Once exported, you can __update changes to it prior to import__, to l__earn from it__ to make changes, simply to __back it up__ or use it as a __template__.
  * Can import a network configuration file via the portal: New > Virtual Network > Import Configuration.
  * When importing, a __list of potential changes__ to the virtual network configuration are presented. Browse through those changes and then click the check __mark to accept the changes and produce__ the new virtual network.


