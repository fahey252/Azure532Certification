### Implement caching
  * Azure offers several options for __distributed, in-memory, and scalable caching__ solutions over time. Azure Redis Cache, Azure Managed Cache Service (deprecated), In-Role Cache (deprecated).
  * __Redis Cache is considered the modern and preferred approach__ for green field distributed cache implementations.

#### Implement Redis caching
  * Redis Cache is an open source project implemented in Microsoft Azure.
  * Offers many features beyond a traditional key-value pair cache, including __manipulating existing__ entries (Appending to a string, Incrementing or decrementing a value, Adding to a list, Computing results or intersections and unions, Working with sorted sets).
  * The large Redis Cache size is __53 GB__.
  * To create: Portal > Storage > Backup + Cache. Choose DNS name, pricing tier, location.
    - `domain.redis.cache.windows.net`.
  * Pricing: __Basic__ tier has several options that supports single node topology. __Standard__ tier supports __two-node master/subordinate topology__ with 99.9 percent SLA.
  * By default, the __cache requires SSL and assigns a port__. Get a __primary and secondary management__ keys.
  * Configure how the cache behaves when maximum memory is exceeded, select the __Maxmemory Policy__ box on the cache blade.
  * Can only create Redit Cache __in the new Portal__.
  * __Both HTTP and HTTPS are supported__; however, only HTTPS is enabled by default.
  * Add the __StackExchange.Redis NuGet__ package to the application.

  ```c#
  using StackExchange.Redis;
  ConnectionMultiplexer connection = ConnectionMultiplexer.Connect("solexpredis.redis.cache.windows.net,ssl=true,password=vPBvnDi5aa2QyxECMqFAEWe8d5Z2nXyLWhK+DAXwE6Q=");

  IDatabase cache = connection.GetDatabase();

  // with expiration policy.
  cache.StringSet("stringValue", "my string", TimeSpan.FromMinutes(30)););
  cache.StringSet("intValue", 42);

  // augment existing values
  cache.StringAppend("stringValue", "more added to my string");
  cache.StringIncrement("intValue");

  string entry = cache.StringGet("stringValue");
  entry = cache.StringGet("intValue");
  ```
  * You can __add any non-primitive type__ to the cache when it is serialized (JSON and other formats).
  * Can use Redis Cache to back your ASP.NET session state for a fast, in-memory distributed session shared by multiple servers.

  * Links
    - [Redis Cache Configuration](https://azure.microsoft.com/en-us/documentation/articles/cache-configure/)

#### Implement migrate solutions from Azure Cache Service to use Redis caching
  * Service provides a secure cache that you can leverage from your cloud applications in a region of your choice.
  * No longer offered through the portal - supporting existing customers.
  * Azure Managed Cache Service __can only be created using Windows PowerShell__ cmdlets. You can then manage the cache settings from the management portal.



