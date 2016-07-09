### Configure a virtual network
  * If you have a private network at your organization, you might want to access servers hosted by Azure as if they are part of your local network.
  * Create a virtual network for a collection of servers, including Azure virtual machines, web roles, or websites in your system topology.
  * __Joining on-premises machines and servers hosted in Azure to the same virtual network__.
  * You can deploy a VM or a Cloud Service into a virtual network.
  * An Azure virtual network (VNet) is a __representation of your own network in the cloud__.
  * A virtual network allows your services to communicate securely without granting public Internet access to VMs. By default, VNets have public internet access.
  * Virtual networks can be used by __virtual machines, cloud services, and websites__.
  * You can use virtual networks to make __resources in separate regions behave as if they are on one network__. Same VNet accross several data centers.
  * Resources on a virtual network __don’t go through endpoints for connectivity__.
  * Virtual networks will make Azure VMs and cloud services available as if they were local to the on-premises users.
  * Links
    - [Virtual Networks Overview](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-overview/)

#### Creating a Virtual Network
  * Portal > New > Network Services > Virtual Network > Custom Create.
    - Leave the defaults for DNS Servers and VPN Connectivity.
    - Set IP range if need be. Control the subnet __IP range to match existing VMs on-premises__ when moving to the cloud or to add additional VMs to an existing range that conjoins with on-premises VMs.
  * Using shared DNS servers with a virtual network allows resources on the network to __use a DNS server that you specify__. You 
  * Can use DNS that Azure provides, but if you need more control, __can specify your own DNS servers__. DNS servers are not round-robin load balanced. The __first DNS server on the list will always be used__.
  * Virtual networks require __Address Space CIDR Block (10.0.0.0/16) and Subnet CIDR block (10.0.0.0/24)__.
  * Control your Azure network settings and define DHCP address blocks, DNS settings, security policies, and routing.
  * __Firewalls can be substituted by Network Security Groups (NSGs)__ applied to each individual subnet.
  * Classic VNets could be added to an affinity group, or created as a regional VNet. __If you have a VNet in an affinity group, it is recommended to migrate it to a regional VNet__.
  * __VNets are completely isolated from one another__. That allows you to create disjoint networks for development, testing, and production that use the same CIDR address blocks.
  * __VNets can be connected to each other__, and even to your on-premises datacenter, by using a site-to-site VPN connection, or ExpressRoute connection
  * __PaaS role instances and IaaS VMs can be launched in the same virtual network__ and they can connect to each other using private IP addresses even if they are in different subnets without the need to configure a gateway or use public IP addresses.
  * Make sure you __create a VNet before deploying any IaaS VMs or PaaS__ role instances to your Azure environment. __ARM based VMs require a VNet__, and if you do not specify an existing VNet, Azure creates a default VNet that might have a __CIDR address block clash__ with your on-premises network. Making it impossible for you to connect your VNet to your on-premises network.
  * A virtual __appliance__ is just another VM in your VNet that __runs a software based appliance function__, such as firewall, WAN optimization, or intrusion detection.
  * When extending on-premsis, make sure VNet in Azure does not conflict with on premisis subnet ranges.

#### Deploy a VM into a virtual network
  * When you create a new VM, you can easily add it to an existing virtual network. Portal > New VM > When selecting the region/location for the VM, You can choose the VNet to add it to instead. VNet determines the location/which data center.
  * You __can move a VM to a previously created virtual network__, but it requires some downtime. Virtual networks are typically chosen when you create a VM. To change the virtual network of an existing VM, you can __remove it from the first network__ and then __create a new VM with the same VHD__ as part of the new network. There is no method to move the VM itself to a different network after it is created.

#### Deploy a cloud service into a virtual network
  * You can __only deploy a cloud service to a virtual network through its service configuration__, not through the management portal.
  * Open the service configuration file __(.cscfg)__. 

  ```xml
  <NetworkConfiguration>
    <VirtualNetworkSite name="SampleNetwork" />
    <AddressAssignments>
      <InstanceAddress roleName="ContactManager.Web">
        <Subnets>
          <Subnet name="FrontEnd" />
        </Subnets>
      </InstanceAddress>
    </AddressAssignments>
  </NetworkConfiguration>
  ```




3. When should you create a VM on your virtual network? (Choose all that apply.) C
A. Create the VM first, then create the virtual network, and then migrate it.
B. Create the virtual network at the same time you create the VM.
C. Create the virtual network first, and then create the VM.
D. The order you create them in doesn’t matter.












