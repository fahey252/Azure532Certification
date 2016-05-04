#Configure Web Apps for Scale and Resilience
  * Use the Azure Portal to scale your App Service plan from Free mode to Shared, Basic, Standard, or Premium mode then configuring certain settings. Should first remove the spending caps in place or risk your web app becoming unavailable if you reach your caps before the billing period ends.
  * Does not require your code to be changed or your applications to be redeployed. What are all the components of your application? Where are the bottle necks in the application? When load is applied to your app, what will break first? Number of users, location of users, peak times.
  * Scaling considerations: Is your Application Statefull? Stateless?
  * Links
      - [Scale a web app in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/web-sites-scale/)

##Configure auto-scale using built-in and custom schedules
  * In the Portal, can _configure_ scale by choosing __Add Rule__ 'schedule and performance rules' such as 'CPU percentage > 80 (increase count by 1)' and 'CPU Percentage < 60 (decrease instance size by 1)'.
  * Can move the two sliders to define the minimum and maximum number of instances to scale automatically.
  * If you have one or more __linked SQL Server databases__ linked to your web app (regardless of App Service plan mode), you can quickly scale them based on your needs.
      - Setup __geo-replication__ to increase the high availability and disaster recovery capabilities in the Geo Replication blade.
  * __Bitness__ Basic, Standard, and Premium modes support 64-bit and 32-bit applications. The Free and Shared plan modes support 32-bit applications only.

##Configure by metric
  * Can create scale rules based on resource, metric name (__CPU, memory, disk queue, HTTP queue__), operator (greater than), threshold (80%), duration (10 minutes), Time aggregation (average), __Action__ (increase count by), Value (1), Cool down (10 minutes.)
  * Examples: Scale up by 1 instance if CPU is above 80% in the last 10 minutes, Scale down by 1 instance if CPU is below 50% in the last 30 minutes.

##Change the size of an instance
  * Choose Web App > Settings > Scale Up > choose priceing tier.
  * Scale out > Scable by: 'Manual' Then increase slider to number of instances (based on app service plan).
  * __D-series__ VM instances are designed to run applications that demand higher compute power and temporary disk performance.
  * __Dv2-series__, a follow-on to the original D-series, features a __more powerful CPU__.
  * The __A8/A10__ and __A9/A11__ virtual machine sizes have the __same capacities__. 
    - The A8 and A9 virtual machine instances include an __additional network adapter__ that is connected to a remote direct memory access (RDMA) network for fast __communication between virtual machines__. 
    - A10 and A11 virtual machine instances do not include the additional network adapter, do not require constant and low-latency communication between nodes.
  * General purpose __XS-XL__ scaled based on CPU, Memory, RAM
  * Memory intesive applications should use __A5 - A7__
  * Netowork heavy: __A8/A9__
  * CPU heavy: __A10/A11__
  * D-Series: Optimized computer, faster processors, feature solid state drives. __D1-D4__ General purpose, __D11-D14__ are memory intensive. __Dv2__ have more powerful CPUs
  * Can specify the Virtual Machine size of a role instance as part of the service model described by the service definition file:
  ```xml
  <WebRole name="WebRole1" vmsize="Standard_D2">
  ...
  </WebRole>
  ```
  
  * All machine sizes provide an application disk that stores all the files from your cloud service package; it is around 1.5 GB in size.
  * 
  * Links
    - [Cloud Services Sizes](https://azure.microsoft.com/en-us/documentation/articles/cloud-services-sizes-specs)

##Configure Traffic Manager