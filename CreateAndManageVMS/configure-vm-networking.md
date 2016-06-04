### Configure VM networking
  * Configure communication with your Azure VMs. VMs can be accessed using either the __cloud service DNS name__ and a port or the __VIP address and port__.
  * __Instance-level public IP addresses__ enable access to a VM using an IP address that will forward requests to the VM, __irrespective of the port used__.
  * To prevent unwanted IP address changes, configure __reserved IP addresses__.

#### Configure reserved IP addresses
  * After the cloud service has been created, the DNS name cannot be changed. Alternately, the VM can be accessed directly by the public virtual IP address.
  * Public endpoints created for a VM use __port forwarding__ to expose a single port on the publicly available virtual IP (VIP) assigned to the cloud service to which the VM belongs and map that public IP and port to a __private IP and port available on a single VM instance__. Can communicate directly with your VM instance using this public IP address (and any port) instead of (or in addition to) using the VIP address and a specific port.
  * Public virtual IP addresses (VIP) used to access your VM via an endpoint are __assigned to the cloud service to which that VM belongs, not to the VM itself__.
  * IP address remains unchanged until all of the VMs in that cloud service are stopped and de-allocated or deleted; If you want to __ensure that the IP address remains fixed__ and reflected as being owned by your Azure subscription, even if the cloud service it is associated with is deleted, then you need to configure a __reserved IP address__.
  * On the Reserved IP Address blade, click the Create A Reserved IP Address. Click Create to provision the new cloud service, new VM, and new reserved IP address.
  * When using external services would want to configure the VM endpoints with a reserved IP address so that the outbound IP address used by the requests from the __application VMs does not change__. Thereby differing from what is configured in the whitelist and breaking connectivity with the third party.

#### Network Security Groups (NSG)
  * Network security group (NSG) contains a __list of Access Control List (ACL) rules__ that allow or deny network traffic to your VM instances in a __Virtual Network__.
  * Associated with __either subnets or individual VM instances within that subnet__.
  * When a NSG is associated with a subnet, the __ACL rules apply to all the VM instances in that subnet__. Traffic to an individual VM can be restricted further by associating a NSG __directly to that VM__.
  * NSG have a name, region, resource group, rules.
  * __Endpoint-based ACLs and network security groups are not supported on the same VM instance__. If you want to use an NSG and have an endpoint ACL already in place, first remove the endpoint ACL.
  * Default __tags__ are system-provided identifiers to address a category of IP addresses. __VIRTUAL_NETWORK__ (all of your network address space), __AZURE_LOADBALANCER__ (Azure’s Infrastructure load balancer,  Azure datacenter IP where Azure’s health probes originate), __INTERNET__ (IP address space that is outside the virtual network and reachable by public Internet).
  * All NSGs contain a set of __default rules__. The default rules cannot be deleted, but because they are assigned the lowest priority, they can be overridden by the rules that you create.
  * You can associate an NSG to VMs (classic deployments only), NICs (Resource Manager deployments only), and subnets (all deployments), depending on the deployment model you are using.
  * Can only associate __a single NSG to a subnet, VM, or NIC__; you can associate the same NSG to as many resources as you want.
  * Since NSGs can be applied to subnets, you can minimize the number of NSGs by __grouping your resources by subnet, and applying NSGs to subnets__.
  * The NSG rules are always related to the original source and final destination of traffic, NOT the load balancer between the two.
  * Links
    - [What is a Network Security Group (NSG)?](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-nsg/)

#### DNS at the virtual network level
  * Azure provides an internal DNS that allows a __VM hosted within a cloud service to resolve the IP address of another VM__ also hosted within that cloud service using the target VM’s host name.
  * If you want to change the VM host name, you must log in to the VM and rename the host using the native operating system’s mechanisms. Using __Server Manager__, open the computer name properties, change the computer name, and restart.

#### Load balancing endpoints
  * If you have multiple VMs listening for requests on the same port and want to distribute external (or Internet) traffic between them, you can use the __Azure Load Balancing service__.
  * Distribute traffic internal to a cloud service or occurring within a VNET between the VMs, you can leverage the __Azure Internal Load Balancing (ILB)__ service.
  * Load balancer uses a hashing function to achieve a relatively even distribution of load between the VMs while also ensuring the subsequent requests use the same protocol, from the same source IP/source port to the same destination IP/destination port hash to the same value, and therefore map to the same VM and continue to be sent to the same VM as long as it remains available.
  * By default, Azure load balancer __distribute load randomly__. Not based on round-robin, network load or network response times.
  * To enable a VM endpoint to participate in load balancing, first create a __load balanced set__ as a part of creating your first load balanced endpoint, and then for each VM that should participate, __create a new endpoint and add it to the load balanced set__. For each VM that should participate, create a new endpoint and add it to the load balanced set.
  * Configure the __HTTP or TCP health probes__ that the load balancer uses to determine the availability of a VM.
  * Load balanced endpoints (public or internal) is functionality available only to __VMs in the Standard tier and not to VMs in the Basic tier__.

#### HTTP and TCP health probes
  * Configure the health probe by specifying the protocol used for probes, the health probe path (if you selected HTTP for the health probe protocol), the health probe port, the__ probe interval__ (seconds between load balancer health probes), and the __number of retries__ (before the load balancer decides an endpoint is unavailable). 

#### Public IPs
  * Public IP address (PIP)/Reserved IP advantages: removing the need to define ports - rely on choosing ports dynamically and PIP to uniquely identify your VM on outgoing requests to external services that have access control or firewall rules that allow or deny based on IP address.
  * Ensure that your VM is deployed to a regional virtual network (VNET) since a __PIP cannot be assigned to VMs that do not belong to a VNET__.
  * Create a VNET. After you have created a regional VNET, be sure to __select that VNET when you provision your VM__ since you cannot easily move a VM into a VNET after creating it.
  * In the IP Addresses blade, under the Instance IP Address blade, click __Instance IP Address to turn on the option__.
  * Instance-level public IP (PIP) address endpoints: All requests flow through the load balancer. You __can use both the PIP and VIP plus port__. PIPs enable you to communicate across all ports. 
  * PIPs free you from having to expose a VM endpoint on specific ports. 
  * A VM __must belong to a regional VNET in order to be assigned a PIP__. PIP can be assigned only to VMs that are a part of a regional virtual network.

#### Firewall rules
  * For Windows Server 2012 running a VM, the firewall is configured using the __Windows Firewall with Advanced Security__, which requires you to remote desktop into the VM and then run it from the Start screen by searching for “Firewall” or entering `wf.msc` in the Run dialog box.
  * VMs can now be created within the portal with security extensions from Trend Micro or Symantec, which provide additional firewall functionality (as well as antivirus and malware and intrusion detection functions). These VM extensions can be added from the Gallery. Listed as check boxes you can select on the last page of the Create a Virtual Machine dialog box.
  * Access control list (ACL) allows you to restrict access to your VMs to specific ranges. Defined on a VM endpoint or load balanced set and apply only to external traffic. They are not applied to internal traffic and cannot be applied to a VNET or to a subnet within a VNET.
  * ACL rule evaluation proceeds in priority order, where rules with the __lowest order value have highest priority__ and are evaluated before rules with higher order values that have a lower priority. First rule matched is applied and stops further rule evaluation.
  * Add any third party connections to the IP whitelist.

#### Direct server return and Keep-alive
  * Direct Server Return (which enables a VM to reply to a request directly to the client instead of through the load balancer) and keep-alive (which controls how long an idle connection is maintained open) are two TCP settings that certain server applications may require to function properly.
  * When responding to a request from a client, the typical flow is for the request to enter through the load balancer, which then forwards the request to a VM. That __VM processes the request and responds to the load balancer, which then forwards the request to the client.__ DSR enables the VM to return the __response directly to the client__ instead of sending it through the load balancer. DSR is most commonly needed to support __SQL Server AlwaysOn Availability Groups__.
  * Direct Server Return can be enabled only __during the creation__ of an endpoint, and it __requires that the public and private ports have the same value__.
  * On the Endpoints blade, be sure to select __Enable Direct Server Return__.
  * Keep-alives are intended to keep the TCP connection with the VM open even in the absence of application communication.
  * Periodically sending a keep-alive packet from the client application to the server-side application, which instructs both the server-side application and any load balancers along the way not to close the idle connection.
  * TCP keep-alive packet is simply an __ACK with the sequence number set to one less than the current sequence number__ for the connection. A host receiving one of these ACKs __responds with an ACK__ for the current sequence number. Keep-alives can be used to __verify__ that the computer at the remote end of a connection is __still available__.
  * In order for a TCP session to stay idle, there should be no data sent or received.
  * There’re 3 registry keys where you can affect TCP Keepalive mechanism on Windows systems: 
    - __KeepAliveInterval__: waits for a TCP Keepalive ACK for the duration of time specified in this registry entry. 
    - __KeepAliveTime__: how often TCP attempts to verify that an idle connection is still intact by sending a keep-alive packet.
    - __TcpMaxDataRetransmissions__: number of times that TCP retransmits an individual data segment (not connection request segments) before aborting the connection
  ```powershell
  # idle timeout can only be configured on  the endpoint at creation time.
  Get-AzureVM -ServiceName "<CloudServiceName>" -Name "<VMName> " | 
  Add-AzureEndpoint -Name "<EndpointName>" -Protocol "tcp" -PublicPort <PublicPort> -LocalPort <PrivatePort> -IdleTimeoutInMinutes <NumMinutes> |
  Update-AzureVM

  Set-AzureLoadBalancedEndpoint -ServiceName "<CloudServiceName>" -LBSetName "<SetName>" -Protocol tcp -LocalPort <PrivatePort> -ProbeProtocolTCP -ProbePort <ProbePort> -IdleTimeoutInMinutes <NumMinutes>
  ```

  ```c#
  public void SetTcpKeepAlive(
    bool enabled,
    int keepAliveTime,
    int keepAliveInterval
  )
  ```
* Links
  - [Things that you may want to know about TCP Keepalives](https://blogs.technet.microsoft.com/nettracer/2010/06/03/things-that-you-may-want-to-know-about-tcp-keepalives/)
