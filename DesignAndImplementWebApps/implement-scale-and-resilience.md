#Design and implement applications for scale and resilience

##Select a Pattern
  * A cloud computing platform enables applications to be hosted in an __Internet-accessible virtual environment__ that supplies the necessary hardware, software, network, and storage capacities and provides for security and reliability, removing much of the burden of purchasing and maintaining hardware and software in-house. Pay for only what you use. 
  * Typical cloud platform: ![Cloud Platform](../images/cloud-platform.gif)
  * __Infrastructure as a Service (IaaS)__: gain access to the underlying physical resources without knowing the details of the underlying hardware and software and can control these systems efficiently through configuration.
  * Applications expose Web interfaces and Web Services for end users, enabling multitenant hosting models. Some functions include connecting disparate systems and leveraging cloud storage infrastructure to store documents. These services fall under the umbrella of Software as a Service (SaaS).
  * A common Windows fabric __abstracts the physical hardware and software platform and exposes virtualized compute and storage resources__.
  * Each instance of the application is monitored for availability and scalability, and automatically managed.
  * __Compute Patterns__: When planning your compute tasks is to remember to execute those tasks in such a way as to avoid moving large amounts of data around. (Worker role/WebJob).
  * __Storate Patterns__: Remote storage and abstracts the storage medium away from the users. The __table storage__ pattern allows the applications to store key/value pairs following a table structure while the __blob storage__ pattern can be used to store any data (mainly files).
  * __Communication Patterns__: message exchange (WCF, REST). Stateless applications.
  * __Management Patterns__: service deployment to organize service definition, configuration and monitoring. __Service level agreement__.
  * Inter-role communication (between services) is enabled through a Queue service, which provides an asynchronous message exchange. Storage services are exposed through REST-style interfaces for easier consumption by other applications and cloud providers.
  * Decouple applications from platform-specific APIs and help increased reusability between on-premise and cloud environment.
  * As a developer, you should accumulate a core set of patterns that will help you develop cloud applications most effectively.
  * Links
      - [Patterns For High Availability, Scalability, And Computing Power With Microsoft Azure](https://msdn.microsoft.com/en-us/magazine/dd727504.aspx)

##Implement transient fault handling for services
  * Makes your application more robust by providing the logic for __handling transient faults__.
  * Recent versions of SDKs for both __Azure Storage and Azure Service Bus__ natively support retries. It is recommended to use these instead of the Transient Fault Handling Application Block.
  * Transient faults typically occur very infrequently, and in most cases, only a few retries are necessary for the operation to succeed. Network faults, downtime, throttling.
  * No intrinsic way to distinguish between transient and non-transient faults.
  * Detection strategies contain built-in knowledge that is capable of identifying whether a particular exception is likely to be caused by a transient fault condition.
  * Block includes detection strategies for __SQL Database, Azure Service Bus, Azure Storage Service, Azure Caching Service__
  * __Retry strategy__ defines how many retries you want to make before you decide that the fault is not transient or that you cannot wait for it to be resolved, and what the intervals should be between the retries. _Fixed interval, incremental interval, random exponential interval._
  * All retry strategies specify a maximum number of retries after which the exception from the last attempt is allowed to bubble up to your application.
  * __Detection Strategy + Retry Strategy = Retry Policy__
  ```c#
  var retryStrategy = new Incremental(5, TimeSpan.FromSeconds(1), TimeSpan.FromSeconds(2));

  var retryPolicy = new RetryPolicy<SqlDatabaseTransientErrorDetectionStrategy>(retryStrategy);
  ```

  ```c#
  using (var config = new SystemConfigurationSource())
  {
  var settings = RetryPolicyConfigurationSettings.GetRetryPolicySettings(config);

  // Initialize the RetryPolicyFactory with a RetryManager built from the 
  // settings in the configuration file.
  RetryPolicyFactory.SetRetryManager(settings.BuildRetryManager());

  var retryPolicy = RetryPolicyFactory.GetRetryPolicy
  <SqlDatabaseTransientErrorDetectionStrategy>("Incremental Retry Strategy");   
   // Use the policy to handle the retries of an operation.

  }
  ```

  * You can define your __retry policies either in code or in the application configuration file__. Defining your retry policies in code is most appropriate for __small applications__ with a limited number of calls that require retry logic. Defining the retry policies in configuration is more useful if you have a __large number of operations__ that require retry logic, because it makes it easier to maintain and modify the policies.
  * Use the __ExecuteAction__ and __ExecuteActionAsync__ method to wrap the calls in your application that may be affected by transient faults
  * The ExecuteAction and ExecuteAsync methods automatically apply the configured retry strategy and detection strategy when they invoke the specified action.
  * Block rethrows the exception to your application if exceeds allowed number of retries.
  * The Transient Fault Handling Application Block is not a substitute for proper exception handling. Your application must still handle any exceptions that are thrown by the service you are using.
  * Links
    - [Using the Transient Fault Handling Application Block](https://msdn.microsoft.com/en-us/library/dn440719(v=pandp.60).aspx)

##Respond to throttling
  * Know the limits, requirements of the service being used that is subject to throtling.
  * Use a queue to store requests and competing instances that use the service.
  * Queue can take in requests a variable rate and then output at a constance rate acting as a buffer.
  * Throttling can be by: __Product, key-based, IP address, user identity.
  * May be subject to __renewal periods__. i.e. 10 calls per hour, 1000 calls per month.
    - `rate-limit-by-key` or `quota-by-key`
  * Can determine the user by extracting the Authorization header, and use that as a rate throttling key.
    - Can also introduce a custom header to allow the developer's client application to communicate the key to the API.
  * Throttling by product subscription key, is a great way to monetize based on usage.
  * Throttling can prevents one user's behavior from degrading the experience of another.

##Disable Application Request Routing (ARR) affinity
  * Application Request Routing IIS Extension to distribute your connecting users between your active instances.
  * Givens connecting user an affinity cookie so next request takes them to same server that originally responded.
  * Front end server recieves request, decides which instance to forward to and give __ARRAffinity__ cookie.
  * To disable, all you have to do to disable affinity is __make sure that Azure doesn’t give the cookies out__. Subsequent requests by the user will be treated as “new”, and instead of trying to route them to “their” server, ARR will use its normal load-balancing behavior to route the request to the best server.
    1. Disable in your __application__
    ```c#
    headers.Add("Arr-Disable-Session-Affinity", "True");
    ```
    2. `customHeaders` config in __web.config__
    ```
    <customHeaders>
      <add name="Arr-Disable-Session-Affinity" value="true" />
    </customHeaders>
    ```
  * With code based, can have cookie added in only certain sections of your website.
  * ARR Affinity cookie is normally included with the 1st response from any Azure Web Sites web site, and subsequently included with any request sent from the client and response received from the server. Inspect with network profiles such as Fiddler, Developer tools. 
  * If you see __both the cookie AND the disable header__, this means that __something is wrong__ with the way you set the header.
  * ARR relies on the _URL rewrite module_ to inspect incoming HTTP requests to make the routing decisions.
  * Links
    - [Disabling ARR’s Instance Affinity in Windows Azure Web Sites](https://azure.microsoft.com/en-us/blog/disabling-arrs-instance-affinity-in-windows-azure-web-sites/)



