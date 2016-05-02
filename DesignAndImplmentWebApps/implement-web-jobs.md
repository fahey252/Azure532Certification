#Implement web jobs

##Write web jobs using the SDK
  * Run a program or script in the same context as the App Service in the background. WebJobs SDK is to simplify the code you write for common task image processing (high CPU tasks)/queue processing (not making the user wait) etc.
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
  * SDK exposes an __extensibility model__ and allows you to write custom triggers and binders. Triggered jobs happen on a schedule or when some event happens and Continuous jobs basically run a while loop.
  	- __Trigger Bindings__ are bindings that monitor external event sources and cause a job function to be executed when they occur. i.e. QueueTrigger.
  	- __Non-Trigger Bindings__ are bindings to an external storage system. An example is the TableAttribute binding.
  * WebJobs don't need triggers/binder. You can manually invoke a function from the __Dashboard__.
  * Output from the `TextWriter` bound object shows up when you go to the Dashboard for a particular function invocation.
  * In a continuous WebJob, application logs show up in __/data/jobs/continuous/{webjobname}/job_log.txt__ in the web app file system.
  * Can dynamically disable and enable functions to control whether they can be triggered, by using a configuration switch that could be an app setting or environment variable name.
  * Links
  	- [What is the Azure WebJobs SDK](https://azure.microsoft.com/en-us/documentation/articles/websites-dotnet-webjobs-sdk)
  	- [Azure WebJobs documentation resources](https://azure.microsoft.com/en-us/documentation/articles/websites-webjobs-resources/)

##Package and deploy web jobs 
  * Can test the WebJob program locally on your development computer, and in production you can run it in a Cloud Service worker role or a Windows service.
  * Copies runtime files to a special location and setups a scheudler for triggers jobs.
  	- __App_Data/jobs/continuous__ for continuous WebJobs
  	- __App_Data/jobs/triggered__ for scheduled and on-demand WebJobs
  * Need `Microsoft.Web.WebJobs.Publish` NuGet package and `webjob-publish-settings.json` file in the project Properties folder. Contains deployment and scheduler settings. WebJobs Publishing NuGet package allows to publish from command line and VS. Does other helpful things such as create Job Scheduler collection if a scheduled job exists.
  * When you link a WebJobs-enabled project to a web project, Visual Studio stores the name of the WebJobs project in a `webjobs-list.json` file in the web project's Properties folder
  * Configure an existing Console Application project so that it automatically deploys as a WebJob when you deploy a web project (Add Existing Project > __Existing Project as Azure WebJob__ > Complete the wizard). Otherwise, deploy WebJob by itself for independent scaling (Right click > Pushlish as Azure WebJob).
  * Need to change the type of WebJob. You'll need to delete the `webjobs-publish-settings.json` file and create new. Change the run mode from continuous to non-continuous causes
  * Links
  	- [Deploy WebJobs using Visual Studio](https://azure.microsoft.com/en-us/documentation/articles/websites-dotnet-deploy-webjobs/)

##Schedule web jobs
  * `TimerTrigger` attribute gives you the ability to trigger functions to run on a schedule. Can schedule a WebJob through Azure Portal or schedule individual functions of a WebJob using the WebJobs SDK.
  ```c#
  public static void ProcessTimer([TimerTrigger("*/15 * * * * *", RunOnStartup = true)]
    TimerInfo info, [Queue("queue")] out string message)
  {
   	message = info.FormatNextOccurrences(1);
  }
  ```

  * Scheduled WebJob (not for continuous WebJobs),creates an __Azure Scheduler job collection__ if one doesn't exist yet, and it creates a job in the collection _WebJobs-{regionname}_. Scheduler job is named {webappname}-_{webjobname}_.
  * If you configure a Recurring Job and set recurrence frequency to a number of minutes, the Azure Scheduler service is not free. Other frequencies (hours, days, and so forth) are free.

