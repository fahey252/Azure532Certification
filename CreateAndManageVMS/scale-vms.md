### Scale VMs
  * Websites which can automatically provision new instances as a part of scale out, Virtual Machines __must be pre-provisioned__ in order for auto-scale to turn instances on or off during a scaling operation.

#### Scale up and scale down VM sizes
  * Scale VM sizes up or down to alter the capacity of the VM (when changing instance size): __number of VHD disks__ that can be attached and the total IOPS capacity, __size of the local temp disk__, number of CPU cores, amount of RAM memory available, __network performance__.  .
  * Cannot control the size of attachable disks for an  instnace size (i.e. A2). The maximum size of an attached disk is __controlled by Azure Storage page blob size constraints__ (1 terabyte) and not by the VM size.
  * Two pricing teirs: __Basic and Standard__. The Standard tier offers load balancing, auto-scale, and a larger set of VM sizes (high memory, SSD-based D-series) as compared to the Basic tier, which lacks these.
  * In the VM blade, choose Usage area and click the Pricing Tier tile, select the new size and tier and Apply.
  
  ```powershell
  Get-AzureVM -ServiceName "<CloudServiceName>" -Name "<VMName>" | 
  Set-AzureVMSize -InstanceSize "<InstanceSize>" |  # i.e. 'A2', 'Small', 'Standard_D14'
  Update-AzureVM
  ```

#### Configure auto-scale
  * Auto-scale automatically adjusts the number of VM instances running within an availability set __based on load or according to a schedule__. Or with respect to the queue depth of an Azure Storage queue or Service Bus queue.
  * These machines must have already been provisioned, but they can be in the stopped (de-allocated) state as they wait for auto-scale to turn them on.
  * All VMs involved must be in the __Standard tier__ (the Basic tier does not provide auto-scale support).
  * All the VMs you want to turn on or off with auto-scale must belong to the __same availability set and same cloud service__. All VMs in the availability set __must be of the same instance size__.
  * In portal for VM, choose Scale Blade
    - Schedules: Usage of the day/night recurring schedule, weekday/weekend schedule, date ranges, specific dates.
    - Size: Instance Count/CPU target (lower threshold below which scale-down actions will be taken.)
    - Apply the schedule to a size configuration. You can combine schedule and metric-based scaling.
    - Define the minimum and maximum number of instances that auto-scale can reach at any point in time.
    - __Cool-off period__ to wait after scaling up from the Scale Up Wait Time.
  * Auto-scale for VMs relies on availability sets to contain the __list of pre-provisioned VMs that auto-scale__ can start up to scale up or shut down to scale down.
  * __Configure auto-scale on the availability set__ resource and applies to everything in the set. VMs must belong to an availability set to benefit from auto-scale.

#### Configure availability sets
  * Availability sets enable you to improve the availability of VMs deployed to your cloud service by identifying to Azure a __group of VMs that should never be brought down simultaneously__ during __updates__ (update domain) and that should be __physically separated__ (that is, connected to a separate power source and network switch - fault domain) so that the failure of a host __does not cause all of the VMs in that group to fail__.
  * Availability sets list VMs that __should never be updated (or fail) together__, such that within the set, a VM is always available. You should typically create an availability set for __each logical tier of your application__.
  * If architecture has two tiers, you should ensure that you create two availability sets, one for each. You could use one cloud service for both the availability sets or provision two cloud services.
  * Set of VMs that Azure will respect to ensure that the service provided by the VMs remains available because at no point in time should all VMs in the set be offline. For a single availability set, you would __need to pre-provision enough VMs to cover your anticipated peak load__.
  * Constrain how Azure locates your VM in update and fault domains.
  * __Update Domains__: constrains how Azure performs updates to the underlying host machine that is running your VM. Azure provides a fixed set of five update domains in which it places your VMs in a __round-robin update process__. Azure places the first five VMs in separate update domains, then continues to distribute additional VMs across update domains.
  * Azure will __never bring down more than one update domain at a time__, effectively ensuring that when Azure updates the host machines, never more than 50 percent of your VMs will be affected.
  * __Fault domains__ consider isolation in terms of power and network (separate server rack). When you add VMs to an availability set, they are __distributed__ between two __fault domains in round-robin fashion__.
  * Strategic placement of your VMs across update and fault domains is controlled simply by their membership in an availability set.
  * For multi-tier applications (such as those having separate front-end, middle, and back-end tiers), it is a best practice to place all the VMs belonging to a single tier in a single availability set and to have separate availability sets for each application tier. At no point will all instance of one tier is down.
  * When creating a VM or existing, you can specify an existing availability set and add the new VM to it.
  * The primary requirement of an availability set is that __all member VMs must belong to the same cloud service__.
  * When creating a new VM, Click __Optional Configuration__ to display that blade. On the Availability Set blade, to use an existing availability set or create new.
  * VMs in an availability set may traverse several update domains for fault tolerance.

  ```powershell
  Get-AzureVM -ServiceName "<CloudServiceName>" -Name "<VMName>" |
  Set-AzureAvailabilitySet -AvailabilitySetName "<AvailabilitySetName>" |
  Update-AzureVM
  ```
