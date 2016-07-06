### Implement Azure storage queues
  * Reliable inter-application messaging to support asynchronous distributed application workflows.
  * Can add messages to a queue, processing those messages __individually or in a batch__, and scaling the service.
  * Creating a backlog of work to process asynchronously and passing messages from an Azure web role to an Azure worker role.
  * `http://<storage account>.queue.core.windows.net/<queue>` such as `http://myaccount.queue.core.windows.net/images-to-resize'
  * All __messages must be in a queue__. Note that the queue name must be __all lowercase__.
  * Links
    - [How to Use Queues](https://azure.microsoft.com/en-us/documentation/articles/storage-dotnet-how-to-use-queues/)

#### Add and process messages
  * Can add messages manually or via your program application workflow (.NET Storage Client Library or other library depending on language).

  ```c#
  CloudQueueClient queueClient = account.CreateCloudQueueClient();
  CloudQueue queue = queueClient.GetQueueReference("workerqueue");

  queue.CreateIfNotExists();
  queue.AddMessage(new CloudQueueMessage("Queued message 1"));
  ```
  * Queue service __assigns a message identifier__ to each message when it is added to the queue. Use by Storage Client Library to get, process and delete the message.
  * Limit of __64 KB per message__ stored in a queue. Any additional data should be kept in a storage account. 
  * Queued message can e__xpire after seven days__ if not processed. Message __expiry can be modified__ while the message is in the queue.
  * Messages are typically published by a separate application.
  * Can peak at messages in the queue without removing them.

  ```c#
  CloudQueueMessage peekedMessage = queue.PeekMessage();    // doesn't remove

  CloudQueueMessage message = queue.GetMessage(new TimeSpan(0, 5, 0));  //dequeue
  if (message != null)
  {
    string theMessage = message.AsString;
    // your processing code goes here
  }

  message.SetMessageContent("Updated contents.");
  queue.UpdateMessage(message,
      TimeSpan.FromSeconds(60.0),  // Make it visible for another 60 seconds.
      MessageUpdateFields.Content | MessageUpdateFields.Visibility);

  queue.DeleteMessage(message);

  // Fetch the queue attributes.
  queue.FetchAttributes();
  // Retrieve the cached approximate message count.
  int? cachedMessageCount = queue.ApproximateMessageCount;
  ```
  * _Visibility_: When you de-queue a message, it is __invisible to the queue for 30 seconds__. Can set the timeout to a value between one second and seven days. Once visibility time has expired, any messages which have not been deleted will become visible again.
  * Queue can contain millions of messages, up to the total capacity limit of a storage account. Can be accessed via HTTP or HTTPS.
  * To access your storage account, be sure to add a connection string to your `app.config` such as:

  ```xml
  <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key" />

  <add key="StorageConnectionString" value="UseDevelopmentStorage=true;" />
  ```
  * You __can change the contents of a message__ in-place in the queue such as updating the status. Can set the visibility timeout to extend another 60 seconds. Could use this technique to track multi-step workflows on queue messages. Would keep a __retry count__ as well, and if the message is retried more than _n_ times, you would delete it.
  * A message returned from __GetMessage__ becomes invisible to any other code reading messages from this queue. Default invisible for 30 seconds. Finish removing the message from the queue, you __must also call DeleteMessage__. 
  * Storage queue messages __are not durable as they expire after seven days__, unlike __Service Bus Queue messages__, which are __persisted until explicitly read and removed__.
  * The message identifier should be __considered opaque to the client__ (__client application shouldn't use the queue message ID__), although it is returned from the AddMessage() method. When retrieving messages from the queue for processing, the message identifier is provided so that you can use it to subsequently delete the message.

#### Retrieve a batch of messages
  * _Queue listener_ can be implemented as __single-threaded__ (processing one message at a time) or __multi-threaded__ (processing messages in a batch on separate threads).
  * One message processed per thread.
  * Can retrieve __up to 32 messages__ from a queue using the GetMessages() method.
  
  ```c#
  IEnumerable<CloudQueueMessage> batch = queue.GetMessages(10, new TimeSpan(0, 5, 0));    // 10 is the number of messages.
  foreach (CloudQueueMessage batchMessage in batch)
  {
  Console.WriteLine(batchMessage.AsString);
  }
  ```
  * Consider the overhead of message processing before deciding the appropriate number of messages to process in parallel (how much CPU, disk, network each message needs to process).  Throttle numberof messages appropriately.
  * Messages are not deleted when the message is read. Messages __must be explicitly deleted__.

#### Scale queues
  * Each individual queue has a target of approximately __2,000 messages per second__ (assuming a message is within 1 KB).
  * Can partition your application to use multiple queues to increase this throughput value.
  * More cost effective and efficient to pull multiple messages from the queue for processing in parallel. Likely need to __scale out compute nodes__.
  * Can configure VMs or cloud services to auto-scale by queue - average number of message per instance.
  * Should implement a back off polling algorithm for queue message processing to control storage costs.
  * Can use __async-await pattern to process messages__. When an async method is used, the async-await pattern suspends local execution until the call completes. This behavior allows the current thread to do other work, which helps avoid performance bottlenecks and improves the overall responsiveness of your application.  `await queue.AddMessageAsync(cloudQueueMessage);` and `await queue.GetMessageAsync();`
  * __Smaller message size can increase throughput__ since the storage service can support more requests per second when those requests hold a smaller amount of data.
  * A single compute instance can process as many messages as its resources allow for. If memory intensive, can process as many messages at the same time as memory capacity allows.
  * Can scale by spreading messages accross several queues. Can not scale websites by queue size. Can scale Cloud Services and VM's by number of items in the queue.

