### Configure Cloud Services And Roles
  * Configuration of key items from __networking, storage, scaling, and caching__.

#### Configure HTTPS endpoint and upload an SSL certificate
  * Secure traffic between the public Internet and your role instances via __transport layer security (TLS) by using a certificate__.
  * Tasks
      1. Acquire a certificate.
        - From a certificate authority (production) or self generated (dev/test).
        ```cmd
        # SubjectName should be production domain name
        makecert -sky exchange -r -n “CN=<SubjectName>” -pe -a sha1 -len 2048 -sr LocalMachine -ss My “<PathTo>\<SubjectName>.cer”
        ```

        - This command will both create the *.cer file at the location you specified and __add it to your Local Machine\Personal keystore__.
      2. Convert the certificate into the __PFX format__ by exporting it from the keystore.
        - Run `mmc` to open an empty Microsoft Management Console > Add the certificates snap in > Export > Export The Private Key > give it a password (will use this when uploading to Azure) > Export as PFX format.
      3. __Upload__ the certificate to the __certificate store__ used by your cloud service. 
        - Go to the service, choose Certificates, upload the PFX file and provide the password.
      4. __Configure an endpoint__ in your role that uses the HTTPS protocol and is configured to use the certificate to secure the traffic.
        - Visual Studio > Solution Explorer > Certficates > lists the certificates present in the Local Machine\Personal keystore > Add an endpoint to the project > Choose port 443 for public and private port and select the certificate just added. > Save solution > deploy and should able to access the HTTPS endpoint.

#### Configure instance count and size, auto-scale
  * Can configure instance size and instance count, and you can use __auto-scale__ for the instance count.
  * Sizes: The __size of the local temp disk__, CPU, RAM, network performance.
  * Scaling the instance __size__ of a role instance __requires that you change the cloud service definition and re-deploy the package__.
  * Visual Studio > double click the role to open the __properties > Configuration tab__ > Choose desired VM size > Save and Publish. When the deployment completes, __instances of the affected role should be running on the new instance size__.
  * The role instance __count__ can be scaled using the existing management portal or by adjusting the count in the __cloud service configuration and re-deploying__. When the deployment completes, the number of instances requested will be deployed behind the load balancer
  * __Instance size = redeploy. Instance count = portal or redeploy.__
  * Auto-scale: based on load or according to a schedule.
  * Auto-scale is __configured at the level of the role__ (web/worker) within a cloud service, much like it is configured at the level of the web hosting plan for websites or at the availability set level for VMs.
  * Can auto scale by __CPU load or queue depth__ for a worker role. Can do by metric for by schedule. __Target Per Machine__, specify the target number of messages that Azure will scale to support per VM

#### Configure network access rules
  * Control or restrict how internal __communication between roles__ within a cloud service happens, effectively dictating which roles are allowed to communicate with the internal endpoints of another role within a cloud service.
  * Assuming you have defined an HTTP internal endpoint on each role, you can configure the service definition (the *.csdef because configures many roles). Use the `NetworkTrafficRules` schema.
  
  ```xml
  <Endpoints>
    <InternalEndpoint name="httpInternal" protocol="http" port="8080"/>
  </Endpoints>
  <NetworkTrafficRules>
    <OnlyAllowTrafficTo>
      <Destinations>
        <RoleEndpoint roleName="WorkerRole2" endpointName="httpInternal" />
      </Destinations>
      <WhenSource matches="AnyRule">
        <FromRole roleName="WorkerRole1" />
      </WhenSource>
    </OnlyAllowTrafficTo>
  </NetworkTrafficRules>
  ```
  * Restrict access to an endpoint within a cloud service, but the source requiring access is not another role in the same cloud service, use an Access Control List (ACL).
  * While you can use the management portal to specify ACLs for VMs, you __configure ACLs for cloud services by editing the service configuration__ file (*.Cloud.cscfg - for an individual VM) and then deploying. Use the `NetworkConfiguration` element.
  
  ```xml
  <NetworkConfiguration>
    <AccessControls>
      <AccessControl name="corpOnly">
        <Rule action="permit" description="allows corpnet" order="100" remoteSubnet="70.181.131.0/28" />
        <Rule action="deny" description="deny rest" order="200" remoteSubnet="0.0.0.0/0"/>
      </AccessControl>
    </AccessControls>
    <EndpointAcls>
      <EndpointAcl role="WorkerRole1" endPoint="httpInput" accessControl="corpOnly"/>
    </EndpointAcls>
  </NetworkConfiguration>
  ```
  * Currently, you must __perform a complete re-deployment if you enable ACLs__ on a service that already exists. An update deployment will fail with an error.
  * You can __join cloud services to an existing virtual network__; however, to do so you must edit the __service configuration file and re-deploy__.
  * Cloud services can be assigned a __reserved IP that keeps the VIP address from changing__, and role instances can be assigned public IPs that enable direct access to each instance without specifying a port. Configure the reserved and public IPs by editing the service configuration file.
  * Before you can configure a reserved IP on a cloud service, you __need to create a reserved IP__ in your Azure subscription. `New-AzureReservedIP -Location "<RegionName>" -ReservedIPName <ReservedIPName>`. Add a reference to the name via the NetworkConfiguration > ReservedIps > __ReservedIP__ element.
  * You enable a __public IP__ for a role instance by enabling it for __all instances in that role__. Each instance receives a unique public IP. The public IP is also configured within the service configuration. NetworkConfiguration > AddressAssignments > PublicIps > __PublicIP__ element.
  * Unlike a reserved IP, this __PublicIP resource is not created in advance__.
  * By using a public IP, you do not have to specify a port to reach an instance of a role.

#### Configure local storage
  * Local storage provides __temporary disk space__ for your cloud service application - temporary location for file uploads, as storage for files that you have to generate.
  * Local storage can __persist between restarts__ of the VM that hosts your role. The data will be __lost if your role is migrated to a different VM host__. Should use Azure Blob storage if durability is needed.
  * Visual Studio > Roles > Role Properties > Local Storage > Add Local Storage > name it. 1 MB and the maximum size allotted for the VM size of the role you have configured.
  * Reach your maximum size of local storage, when you attempt to write again, you will get an out of disk space error message. Have a __method for deleting old files__ and clearing out space so that writing can continue.
  * The __Clean On Role Recycle__ option controls persistence between reboots.
  * On RoleEntryPointCode, access local storage path with: `LocalResource localResource = RoleEnvironment.GetLocalResource("LocalStorage1");`

#### Configure multiple websites in a web role
  * A single web role is __not limited to hosting only a single webs__. Aweb role can be configured, by editing the cloud __service definition__, to host multiple websites.
  * Before the closing `</Sites>` tag, add a new Site element. `name` attribute that uniquely identifies the site, `physicalDirectory` attribute. add a `Bindings` element with a single Binding element with same name and `endpointName` attribute values as those used by the original website. Add a `hostHeader` attribute that will differentiate requests intended for the original website versus the website just added.
  * An alternative to differentiating the websites __via host headers is to use different ports__. admin.contoso.com vs contoso.com vs contoso.com:8080.

#### Configure cloud service networking
  * Controlling communication between Internet traffic and the role instances and __between role instances__.
  * Configured to expose one of three types of endpoints
      - __Input endpoint__: Controls the transport protocol (http, https, tcp, or udp) and port used by traffic coming __from the Internet__ to instances of the role. Azure load balancer determines which role instance will receive a particular request. Traffic on the configured localPort number or on the one assigned automatically by the Azure fabric controller if a localPort is not provided.
      - __InstanceInput endpoint__: specific protocol (tcp or udp) and port that when used by __Internet traffic goes via port forwarding__ directly to a specific role instance.
      - __Internal endpoint__: dynamically allocated public port range that can be used for communication __between roles and role instances__ within a cloud service. Communicate with this endpoint is the __internal IP address__ assigned to each role instance, and the role instance to which the traffic goes will see the traffic on the configured private port.

#### Configure custom domains
  * Any roles for which you have configured an input endpoint will be accessible using the DNS name of the form __<cloudServiceName>.cloudapp.net__.
  * Resolve to the IP address of your cloud service, you have to configure DNS.
    - __A Record__: directly to the __VIP address__ of your cloud service for a domain.
    - __CNAME Record__: DNS entry that maps a subdomain to the __DNS name of your cloud service__ *.cloudapp.net.
  * The key difference between the two is that an A record maps a domain to an IP address and a __CNAME creates an alias that maps a domain name to another domain name__.
  * In most DNS registrars, the at symbol __(@) maps to the root of the domain__. Can use an asterisk (*) as the host name to map all subdomains to the IP address.
  * When using an A record, you have to be cautious that the __IP address value you map to does not change__. If you ever delete the deployment in the production slot of your cloud service and then later re-deploy, your VIP will have changed. It is a best practice to deploy to the staging slot first and then perform a VIP swap operation that moves your new deployment into production to ensure you don’t lose the VIP associated with production. Preserve the IP address of the cloud service is to assign a __reserved IP__ to your cloud service and then use that IP address in the A record. Another alternative is to not use an A record at all, and to instead map your custom domain using a CNAME.
  * Map www.contoso.com with CNAME: in registar map www -> contoso.cloudapp.net

#### Configure caching
  * Besides hosting websites and logic for background processing, a role can also be used to provide a __cache cluster using In-Role Cache for Azure Cache__.
  * __Distributed, in-memory cache__ can be used to store frequently accessed data so that access to it is fast and offloads the data retrieval work from other resources like your SQL database.
  * In-Role Cache is configured to run within a single role, instances of your cache role make up the cache cluster.
  * Cache can only be accessed from managed code running within role instances within the same cloud service deployment. VMs and websites cannot access the cache provided by In-Role Cache.
  * In-Role Cache also provides support for the __memcache protocol, which, when enabled, provides cache access to other non-.NET languages__, such as PHP, using the memcache client libraries for that language running on instnaces in the same cloud service.
  * In-Role Cache can be configured in the following two different topologies:
    - __Co-located cache__: Each role instance contributes a certain __percentage of its memory to the cache__ and joins in as part of the larger cache cluster. Provided in addition to whatever the primary function of the role is.
    - __Dedicated cache__: Worker roles that are __only used for caching__ and nothing else can form a dedicated cache.
    - _Name cache_: cache on which different policies are configured
  * You are paying for the __time the VM infrastructure__ supporting your role instances are running and not for any additional cache service.
  * In-Role Cache stores objects in serialized form within the in-memory cache. Serialization relies on the `NetDataContractSerializer` - stored objects must be serializable.
  * Configuring a _co-located_ versus a _dedicated cache_ are very similar.
  * Server Explorer > Role > enable In-Role Cache > properties > select topology (Co-located/Dedicated) > For a co-located topology, select Co-Located Role and then use the Cache Size slider to indicate the maximum percentage of memory from each instance in the role to contribute toward the cache cluster > Select the __storage account that the cache cluster will use to persist its runtime state__. (Note that this is not used to persist any data in the cache, which is purely maintained within instance memory.) > 
  * Cache Policies:
    - __High Availability__: any single piece of cache data is represented by __two copies__, each on a __separate instance__ within the cluster.
    - __Notifications__: enable cache clients to poll the cluster for notifications about __data that has changed__.
    - __Eviction Policy__: when __memory runs low__ (such that the least recently used objects are removed first) by selecting LRU. 
    - __Expiration Type__: how cache data is expired. __Absolute__ to remove anitem from the cache when the duration calculated from when the item was added to the cache exceeds the __time to live__. __Sliding Window__ to reset the countdown to the time to live on a given object in the cache __every time the object is accessed__.
    - __Time to Live__: duration in minutes used by the Absolute or Sliding Window expiration types.
  * __Named cache__: logical grouping of cache data that enables you to have another __cache on which different policies are configured__.
  * The number of __instances multiplied by the memory provided__ by each instance yields the total __capacity of the cache cluster__.
  * Write code within your web or worker role that uses the __caching API__ to read from and write to the cache. Include the appropriate assemblies in your project, add the caching NuGet package (Windows Azure Cache).
  * Place some configuration in web.config (for a web role) or app.config (for a worker role) to configure the cache client. Add the following configs:

  ```xml
  <dataCacheClients>
    <dataCacheClient name="default">
      <autoDiscover isEnabled="true" identifier="CacheWorkerRole1" />
    </dataCacheClient>
  </dataCacheClients>
  ```
  * Access in application code wrapping in `using`:

  ```c#
  DataCacheFactory dataCacheFactory = new DataCacheFactory();
  DataCache defaultCache = dataCacheFactory.GetDefaultCache();
  DataCache namedCache = dataCacheFactory.GetCache("default");    // if named
  defaultCache.Add("FrequentlyUsedItem", "Value of the item");  // throws error if exists.
  defaultCache.Put("FrequentlyUsedItem", "Value of the item");  // replace
  object cachedObject = defaultCache.Get("FrequentlyUsedItem");
  ```

  * If you are attempting to use the cache within a web role’s RoleEntryPoint logic, need to manually setup instead of using the *.config files.
  * Always __dispose of the DataCacheFactory__ (`using` statement) to free underlying resources when you are finished with using the factory reference. Each factory has a setting for the number of connections to the cache. 



