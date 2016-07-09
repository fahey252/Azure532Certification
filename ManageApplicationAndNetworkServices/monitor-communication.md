### Monitor communication
  * Service Bus __pricing tier__, __scale__ Service Bus features, and __monitor__ communication.

#### Choose a pricing tier
  * A messaging tier for __brokered messaging entities__ and a __notification__ hubs tier.
  * Service Bus pricing tiers fall into a few categories: a Service Bus messaging tier, an event hub tier, and a notification hub tier.
  * Choose a messaging tier __for all entities that will belong to that namespace__.
  * __Basic tier__: __Queues and event hubs only__ (up to 100 connections)
  * __Standard tier__: Queues, event hubs, topics, and relays (up to 1000 connections). Supports advance brokered messaging features such as __transactions, de-duplication, sessions, and forwarding__.
  * Related to event hubs, the __basic tier only supports a single consumer__ group, so if you want to support parallelized processing across partitions, choose standard messaging tier.
  * Notification Hub Pricing:
    - __Free tier__: Up to 1 million messages per month; __no support for auto-scale__ nor a number of other enterprise features
    - __Basic tier__: 10 million messages per month plus unlimited overage for a fee; support for auto-scale; no support for other enterprise features
    - __Standard tier__: The same as basic tier with all enterprise features

#### Scale Service Bus
  * Service Bus entities scale based on a variety of properties: __Namespaces, Partitions, Message size, Throughput units, Entity instances__.
  * For relays, there is a __limit to the number of endpoints, connections__ overall, and listeners.
  * Number of topics and queues are limited, and separately a smaller number of partitioned topics and queues are supported.
  * Can avoid reaching some of these limits by __isolating entities that could be impacted into separate namespaces__.
  * Each entity has __slightly different requirements for scale__.
  * __Relays__
    - Scale relays for potential __namespace limitations__. Relay services are primarily __scaled by adding namespaces and listeners for processing messages__.
    - Relays are only supported in the standard messaging tier.
    - Relay endpoints have a __limited number of overall connections and listeners__ that can be supported per namespace.
    - Should __consider the number of concurrent connections__ that might be required for communicating with the endpoint.
    - __Design the solution to support publishing__ an instance of each relay service into multiple namespaces.
    - Design the solution so that clients sending messages to the relay service __can distribute calls across a selection of service instances__. This implies __building a service registry__.
  *  __Queues and topics__
    -  Queues and topics are primarily scaled by __choosing a larger storage size, adding partitions, and batching messages__ on both send and receive.
    - Neither is particularly bound by the namespace it belongs to except in the __total number of queues or topics__. Limited number of __partitioned__ queues and topics.
    - Must choose the maximum expected storage __size__ from 1 GB to 5 GB, and this __cannot be resized__.
    - Can have senders __batch__ messages to Service Bus and listeners batch receive or pre-fetch from Service Bus. __Reduces number of needed listeners__.
    - __Partitions__ increases the number of __message brokers__ available for incoming messages
  * __Event Hubs__
    - Each namespace can have multiple event hubs, but those event hubs __share the throughput units allocated to the namespace__.
    - If __a single event hub has the potential of scaling beyond__ the available throughput units for a namespace, you might consider creating a separate namespace for it.
    - Can request additional throughput units for a namespace by calling Microsoft support.
    - Primary unit of __scale for event hubs is throughput units__. You get a single throughput unit which provides ingress up to 1 MB per second, or 1,000 events per second, and egress up to 2 MB per second. Pre-purchase units and can by default configure __up to 20 units__.
    - __Single event hub partition can scale to a single throughput unit__; therefore, the number of partitions across event hubs in the namespace should be equal to or greater than the number of throughput units selected.
    - Number of __partitions should exceed the number of throughput units__ for optimal performance
  * __Notification Hubs__
    - With basic or standard tier, you can push an unlimited number of notifications, but the fees per notification will vary between these two tiers.
  * When entities reside in different namespaces, if a Event Hub needs to talk to a storage queue, it will require a connection used.
  * Links
    - [Service Bus Partitioning](https://azure.microsoft.com/en-us/documentation/articles/service-bus-partitioning/)

#### Monitor service bus queues, topics, relays, and notification hubs
  * __Service Bus Explorer__ is a tool used for managing and monitoring Service Bus features such as queues, topics, relays, and notification hubs. More information here than in the portal.
  * Can monitoring Service Bus instances in the portal via Monitor blade.
  * Useful to __compare the number of incoming and outgoing messages__ to note any behavior that indicates burst activities for messages received or reduced ability to process requests as they are received.
  * When the queue or topic length increases in bursts, it can be an indicator of a problem or an __indication that you should increase the number of listeners or optimize message processing__ using prefetch or batches.
  * Can monitor queue/topic length to note increases indicating requests not yet processed.
  * __Topics do not have a concept of ingress or egress__. You can monitor ingress and egress for event hubs/queues.
  * Can show relative or absolute values in the graph, and you can choose the number of hours or days to include based on a predefined list.
  * Can monitor Notification Hubs by device: You can monitor errors and other types of counters by device, which can be helpful in __isolating issues that are local to a particular device format or its notification service__.



