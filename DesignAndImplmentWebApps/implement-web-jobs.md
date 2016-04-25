#Implement web jobs

##Write web jobs using the SDK
  * Run a program or script in the same context as the App Service. WebJobs SDK is to simplify the code you write for common task image processing (high CPU tasks)/queue processing (not making the user wait) etc.
  * Visual Studio __Console Application__ project provide a framework that your code uses by decorating your methods with WebJobs SDK attributes.
  * __Dashboard__ allows to monitor and diagnose web jobs. <https://{webappname}.scm.azurewebsites.net/azurejobs/#/functions>
  * Use web jobs for __schedule tasks__ and __long running__ tasks in a background thread.
  * SDK ensures that your functions will be processed only once for each message or blob if you have multiple VM's.
  * Main method creats a `JobHost` object and the SDK provides __triggers__ that specify what conditions cause the function to be called, and __binders__ that specify how to get information into and out of method parameters. i.e. `QueueTrigger` attribute causes a function to be called when a message is received on a queue.
  * Many other triggers like: `QueueTrigger`, `FileTrigger`, `WebHookTrigger`, and `ErrorTrigger`.
  * WebJob code sample. SDK takes care of details to make triggers/processing happen like binding values to the parameters/dispatching the event.
  ```c#
  public static void Main()  //  polls a queue and creates a blob for each message
  {
    JobHost host = new JobHost(); //  container for a set of background functions
    host.RunAndBlock();  // run on the current thread or a background thread. Current thread in this case.
  }

  // trigger when new message in queue named 'webjobqueue'
  public static void ProcessQueueMessage([QueueTrigger("webjobsqueue")] string inputText, 
        [Blob("containername/blobname")]TextWriter writer)
  {
        writer.WriteLine(inputText);
  }
  ```

  * `JobHost` monitors functions, watches events, call proper function when event happens.
  * SDK is __extendable__ and allows you to write custom triggers and binders.
  * WebJobs don't need triggers/binder. You can manually invoke a function from the __Dashboard__.
  * Links
  	- [What is the Azure WebJobs SDK](https://azure.microsoft.com/en-us/documentation/articles/websites-dotnet-webjobs-sdk)

##Package and deploy web jobs 
  * Can test the WebJob program locally on your development computer, and in production you can run it in a Cloud Service worker role or a Windows service.

##Schedule web jobs
  * `TimerTrigger` attribute gives you the ability to trigger functions to run on a schedule. Can schedule a WebJob through Azure Portal or schedule individual functions of a WebJob using the WebJobs SDK.
  ```c#
  public static void ProcessTimer([TimerTrigger("*/15 * * * * *", RunOnStartup = true)]
    TimerInfo info, [Queue("queue")] out string message)
  {
   	message = info.FormatNextOccurrences(1);
  }
  ```
  * 
