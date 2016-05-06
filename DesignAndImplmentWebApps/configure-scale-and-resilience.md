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
  * Links
    - [Cloud Services Sizes](https://azure.microsoft.com/en-us/documentation/articles/cloud-services-sizes-specs)

##Configure Traffic Manager
  * Traffic Manager is the __distribution of user traffic__. Increases apps __responsiveness/availability, cloud migration/hybrid on-premises, A/B testing, distribute traffic based on weighted values (based on geolocation)__.
  * Traffic Manager is a popular option for on-premises scenarios including burst-to-cloud, migrate-to-cloud, and failover-to-cloud. Applies intelligent policy engine to Domain Name System (DNS) queries for the domain names of your Internet resources such as directing end-users to the endpoint with the lowest network latency from the client.
  * Zero downtime: wait for the endpoint to complete the servicing of existing connections. When there is no more traffic to the endpoint, you update the service on that endpoint and test it, then re-enable it.
  * Configure to determine which endpoint should service the request based on a DNS query. The DNS resource record for the __company domain points to a Traffic Manager domain name__ maintained in Azure Traffic Manager. This is achieved by using a CNAME resource record that maps the company domain name to the Traffic Manager domain name. i.e. __contoso.com IN CNAME contoso.trafficmanager.net__.
  * Traffic Manager uses the specified traffic routing method and __monitoring status to determine which endpoint__ should service the request. Then  returns a CNAME record that maps the Traffic Manager domain name to the domain name of the endpoint. The user's DNS server resolves the endpoint domain name to its IP address and sends it to the user.
  * User continues to interact with the chosen endpoint until its __local DNS cache entry expires for the duration of their Time-to-Live (TTL)__. TTL value controls how often the client’s local caching name server will query the Azure Traffic Manager DNS system for updated DNS entries.
  * Seven steps to configure Traffic Manager
    - __Deploy Endpoints__
    - __Choose DNS name for Traffic Manager Profile__
    - __Choose the monitoring conifguration__
      + Wont route to endpoints that aren't available.
    - __Choose load balancing method__
      + Failover, performance, round robin.
    - __Create Traffic Manager profile__
    - __Test profile__
    - __Point the domain name to the Traffic Manager Profile__
  * Three routing method types:
    - __Failover__: Automatically redirect traffic when failure detected
    - __Performance__
    - __Weighted Round Robin__ 
  * Can configure Traffic Manager settings using the Azure classic portal, with REST APIs, and with Windows PowerShell cmdlets. Some settings are not available using just one method. i.e. configuring external endpoints (type = ‘Any’) for Round Robin routing.
  * Can use __Quick Create__ in Azure Portal to create Traffic Manager profile.
  * Links
    - [Traffic Manager](https://azure.microsoft.com/en-us/services/traffic-manager)
    - [What is Traffic Manager](https://azure.microsoft.com/en-us/documentation/articles/traffic-manager-overview)
###Left off on Best Pratices
    - [Nested Profiles](https://azure.microsoft.com/en-us/blog/new-azure-traffic-manager-nested-profiles)
    - [Manage an Azure Traffic Manager profile](https://azure.microsoft.com/en-us/documentation/articles/traffic-manager-manage-profiles/)
