### Design and implement a communication strategy
  * Microsoft __Azure Service Bus__ is a hosted infrastructure service that provides multi-tenant services for __communications between applications__. Events at scale.
  * __Relays__: Expose secure endpoints for __synchronous__ calls to service endpoints across a __network boundary__, for example to expose on-premises resources to a remote client
  * __Queues__: Implement brokered messaging patterns where the message sender can __deliver a message even if the receiver is temporarily offline__.
  * __Topics and subscriptions__: Implement __publish and subscribe__ patterns where messages can be received by __more than one receiver__ (subscriber)
  * __Event hubs__: Implement scenarios where receivers choose __events to register__ for, to support __high-volume__ message processing at scale
  * __Notification hubs__: __Push__ notifications to mobile devices.
  * Relays are used for relayed, synchronous messaging. The remaining scenarios are a form of brokered, __asynchronous messaging patterns__.
  * Queues and topics are message brokering features of Service Bus that provide a __buffer for messages, partitioning options for scalability, and a dead letter feature for messages that can’t be processed.__
  * Queues support __one-to-one__ message delivery while topics support __one-to-many__ delivery.
  * Service Bus features can __require authentication using a key__. You can create multiple keys to isolate the key used for management and usage __patterns, such as send and receive__.
  * Each Service Bus entity __provides a state property that can be disabled__ in the management portal or programmatically through management tools like Windows PowerShell. This allows you to __disable processing while still maintaining__ the actual entity for later use.
  * __Partitions can be set when you create the queue or topic__. This allocates additional message brokers for handling incoming messages to the entity, and this __increases scale__.

#### Create service bus namespaces and choose a tier
  * The__ Service Bus namespace is a container__ for Service Bus resources including queues, topics (/subscriptions), relays, notification hubs, and event hubs.
  * Group these resources into a single namespace or __separate them according to management and scale requirements__.
  * Create a name space: Portal > Create Service Bus > Type (messages|notifications) > Messaging Tier (basic|standard).
  * The namespace type was introduced to allow splitting the functionality between messaging and notification hubs, to optimize functionality for the latter. You __can still include queues, topics, and event hubs in a namespace that is designed to support notification hubs__.
  
  ```powershell
  New-AzureSBNamespace –Name sol-exp-svcbus –Location West US   # defaults to messaging
  New-AzureSBNamespace –Name sol-exp-svcbus –Location West US –NamespaceType NotificationHub
  ```
  * The recommended way to __secure Service Bus resources is with SAS tokens__. Can indicate an ACS namespace for securing the resources in the namespace via powershell.
  * Communication Protocols: 
    - __SBMP__: Ports 9350-9354 (for relay) 9354 (for brokered messaging). Service Bus Messaging Protocol (SBMP), is a proprietary __SOAP-based__ protocol that typically relies on __WCF__ under the covers to implement messaging with between applications through Service Bus.
    - __HTTP__: Ports 80, 443. HTTP protocol can be used for relay services when one of the __HTTP relay bindings are selected__ and the Service Bus environment is set to use HTTP connectivity. 
    - __AMQP__: Ports 5671, 5672. Advanced Message Queuing Protocol (AMQP) is a modern, __cross-platform asynchronous__ messaging standard. The brokered messaging client library uses this protocol if the connection string indicates TransportType of Amqp.
  * Advanced Message Queuing Protocol __(AMQP) is the recommended protocol__ to use for brokered message exchange if __firewall rules are not an issue__. Connectivity issues are common for on-premises environments that disable ports other than 80 and 443. For this reason, it is still often __necessary for portability to use HTTP protocol for brokered messaging__.

#### Develop messaging solutions using service bus queues, topics, relays, and notification hubs
  * __Service Bus Relays__
    - Service Bus Relay service is frequently used to __expose on-premises resources to remote client applications__ located in the cloud or across __network boundaries__.
    - Relay enables access to on-premises resources __without exposing on-premises services to the public Internet__. By default, all relay messages are sent through Service Bus (relay mode), but __connections might be promoted to a direct connection (hybrid mode)__.
    - Create __shared access policies__ to secure access to management.
    - Create a __service contract__ defining the messages, Create a __service implementation__ for that contract, __Host the service in any compatible WCF__ hosting environment, binding/credentials, Create a __client reference__ to the relay using typical WCF client channel features, __call methods__ on the service contract to invoke the service through the Service Bus relay.
    - Service Bus Relay was streamlined for use with the WCF ServiceModel library, so this is the easiest way to build a relay solution. It is also __possible to implement relay without WCF__ i.e. REST.
    - __Bindings__: BasicHttpRelayBinding, WS2007HttpRelayBinding, WebHttpRelayBinding, NetTcpRelayBinding, NetOneWayRelayBinding, NetEventRelayBinding.
    - Communication can be done via HTTP or TCP. TCP relay supports two connection modes: __relayed (the default) or hybrid__. In hybrid mode, communications are initially relayed, but if possible, a __direct socket connection is established between client and service (Hybrid)__, thus removing the relay from communications for the session.
    - __Relay credentials__ are managed on the Configure tab for the namespace.
    - Not recommended to use the __root shared access policy__. Create an entry for a receiver and sender __policy that respectively can only listen or send__. Policy has a __name, permission level and a primary/secondary key__.
    - Considered best practice to create __separate keys for the sender and receiver__.
    - Use the __Microsoft Azure Service Bus__ NuGet package for development.
    - You __can configure WCF relay endpoints programmatically or by using application configuration__ in the `<system.servicemodel>` section. The latter is more appropriate for dynamically configuring the host environment for production applications.
    - After you have created the relay service, defined the endpoint and related protocols, and noted the sender policy name and key, you can create a __client to send messages to the relay service__.
    - Practically speaking, __most systems today employ an asynchronous architecture__ that involves queues, topics, or event hubs as a way to queue work for on-premises processing from a remote application.
    - The __number of client and server listeners are restricted__ by the messaging tier chosen for the namespace. The limit is __100 for basic tier and 1,000 for standard tier.__
    - __No inbound communications__: Relays use an outbound connection to __establish communications with Service Bus__. No inbound communication is supported.
  * __Queues__
    - Brokered messaging service that supports __physical and temporal decoupling__ of a message producer (sender) and message consumer (receiver).
    - First In First Out (FIFO) buffer to the first receiver that removes the message. There is __only one receiver per message__.
    - __Storage Queues vs Service Bus Queues__: Azure queues are built on top of storage, while __Service Bus queues__ are built on top of a broader messaging infrastructure.
    - Queues: __size__ (1 GB to 5 GB.) and __partitions__ for scale out, message handling for __expiry__ and __locking__, and support for __sessions__, __TTL__ (14 days default).
    - _Sessions_: messages can be grouped into __sequential batches to guarantee ordered delivery__ of a set of messages.
    - Service Bus queue credentials should be Shared Access Policy with Send and Receiver policy name with keys.
    - To communicate with a queue, you provide __connection information including the queue URL and shared access credentials__ such as `Endpoint=sb://sol-exp-msg.servicebus.windows.net/;SharedAccessKeyName=Sender;SharedAccessKey=f9rWrHfJlns7iMFbWQxxFi2KyssfpqCFlHJtOuLS178=`
    - The connection string shown in the management portal for queues, topics, notification hubs, and event hubs __does not use AMQP protocol by default__. You must add a `TransportType=Amqp` string as query parameter to tell the client library to use AMQP.
    - The `BrokeredMessage` type can __accept any serializable object or a stream__ to be included in the body of the message. You can also set additional custom properties on the message and provide __settings relevant to partitions and sessions__.
    - Two modes for processing queue messages:
      + __ReceiveAndDelete__: Messages are __delivered once__, regardless of whether the receiver fails to process the message.
      + __PeekLock__: (default) Messages are __locked after they are delivered__ to a receiver so that other receivers do not process them unless they are unlocked through timeout or if the receiver that locked the message abandons processing.
    - Service Bus queues support __at-least-once processing__. A message might be redelivered and processed twice. To avoid duplicate messages, __use the MessageId of the message to verify that a message was not already processed by your system__.
    - __Dead letter queue__: Messages that cannot be processed are considered poison messages and should be removed from the queue and handled separately. Messages written to the dead letter sub-queue __do not expire__.
    - Theoretically, you __can build a publish-and-subscribe solution using queues__; however, the implementation requires quite a lot of custom coding, so that is not recommended
  * __Topics and Subscriptions__
    - Topics and subscriptions support __one-to-many communication__ in support of traditional publish and subscribe patterns in brokered messaging.
    - When messages are sent to a topic, a __copy is made for each subscription__.
    - Messages are not received from the topic; they are __received from the subscription__. __Receivers can listen to one or more subscriptions__ to retrieve messages.
    - By default, __there are no shared access policies created for a new topic__. You will usually create at least one policy per subscriber to isolate access keys and one for send permissions to separate key access between clients and services.
    - With topics and subscriptions, you __send messages to a topic and retrieve them from a subscription__.

    ```c#
    TopicClient topic = factory.CreateTopicClient(topicName);
    topic.Send(new BrokeredMessage(“topic message”));
    ```
    - Processing messages from a subscription is similar to processing messages from a queue. You can use ReceiveAndDelete or PeekLock mode. The latter is the preferred mode and the default.
    - Can __batch messages__ from a queue or topic client to a__void multiple calls to send messages to Service Bus__, including them in a single call.
    - Powerful features of topics and subscriptions is the __ability to filter messages based on certain criteria__. Determine which subscription should receive a copy of each message

    ```c#
    SqlFilter filter = new SqlFilter("Priority == 1");
    ns.CreateSubscription(topicName, "PrioritySubscription", filter);

    BrokeredMessage message = new BrokeredMessage("priority message");
    message.Properties["Priority"] = 1;
    ```
  * __Event Hubs__
    - Support very __high-volume message streaming__ as is typical of enterprise application logging solutions or Internet of Things (IoT) scenarios.
    - Ingesting message data at scale, Consuming message data __in parallel by multiple consumers__,  __Re-processing__ messages by restarting at any point in the message stream.
    - Messages to the event hub are FIFO and durable for up to __seven days__.
    - reconnect to the hub and __choose where to begin processing__, allowing for the re-processing scenario or for reconnecting after failure.
    - Event hubs differ from queues and topics in that there are __no enterprise messaging features__. There isn’t a Time-to-Live (TTL) feature for messages, no dead-letter sub-queue, no transactions or acknowledgements. The focus is __low latency, highly reliable, message streaming__ with order preservation and __replay__.
    - Event hubs can by default __handle 1-MB ingress per second, 2-MB egress per second per partition__. This can be __increased to 1-GB__ ingress per second and 2-GB egress per second through a support ticket.
    - __Partition Count__: Defaults to 8. Can be set to a value between 8 and 32 and __cannot be modified after it is created__. Support ticket you can increase that number up to 1,024.
    - __Message retention__: number of days a message will be retained before it is removed from the event hub. Can be between __1 and 7 days__.
    - With event hubs, you __send streamed messages as EventData instances__ to the event hub, and Service Bus leverages the allocated partitions to distribute those messages.
    ```c#
    EventHubClient client = factory.CreateEventHubClient(ehName);
    string message = "event hub message";
    EventData data = new EventData(Encoding.UTF8.GetBytes(message));
    client.Send(data);
    ```
    - Optionally __supply a partition key__ to group event data so that the data is collected in the same partition in sequence. Otherwise, data is sent in a __round-robin fashion__ across partitions. In addition, you can supply custom properties and a sequence number to event data prior to sending.
    - Can track event publishers with publisher policies. This creates a __unique identifier for each publisher for tracking purposes__.
    - __Consumer Groups__: consumers connect to a partition. A __default consumer group is created for each new event hub__, but you can optionally create multiple consumer groups (receivers) to __consume events in parallel__
    - Messages are stored in a buffer and __can be processed multiple times__.
    - It is true that __event hubs do not support a TTL__ for messages; however, __messages are not deleted__ as soon as they are consumed. Messages __remain in the buffer until the designated global expiry__ for the buffer.
    - Buffer allows for __different consumers to receive data from the buffer multiple times__ by requesting a particular part of the message stream
  * __Notification Hubs__
    - Service for push notifications to mobile devices, at scale. Notification hub to receive messages from the publisher and handle pushing those events in a platform-specific format to the mobile device.
