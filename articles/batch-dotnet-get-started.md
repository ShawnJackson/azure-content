<properties title="Tutorial - Getting Started with the Azure Batch Library for .NET" pageTitle="Tutorial - Getting Started with the Azure Batch Library for .NET" description="required" metaKeywords="" services="batch" solutions="" documentationCenter=".NET" authors="yidingz, karran.batta" videoId="" scriptId="" manager="asutton" />

<tags ms.service="batch" ms.devlang="dotnet" ms.topic="article" ms.tgt_pltfrm="na" ms.workload="big-compute" ms.date="10/02/2014" ms.author="yidingz, karran.batta" />

#Getting Started with the Azure Batch Library for .NET  

This article contains the following two tutorials to help you start developing by using the Microsoft .NET library for the Azure Batch service.  

-	[Tutorial 1: Azure Batch Library for .NET](#tutorial1)
-	[Tutorial 2: Azure Batch Apps library for .NET](#tutorial2)  


For background information and scenarios for Azure Batch, see [Azure Batch technical overview](http://azure.microsoft.com/en-us/documentation/articles/batch-technical-overview/).

##<a name="tutorial1"></a>Tutorial 1: Azure Batch Library for .NET
  	
This tutorial will show you how to create a console application that sets up distributed computation among a pool of virtual machines by using the Azure Batch service. The tasks that are created in this tutorial evaluate text from files in Azure Storage and return the words that are most commonly used. The samples are written in C# code and use the Azure Batch Library for .NET.

>[WACOM.NOTE] To complete this tutorial, you need an Azure account. You can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial](http://www.windowsazure.com/en-us/pricing/free-trial/). 
>
>You need to use NuGet to obtain the **Microsoft.Azure.Batch.dll** assembly. After you create your project in the Microsoft Visual Studio development system, right-click the project in **Solution Explorer**, and then click **Manage NuGet Packages**. Search online for **Azure.Batch**, and then click **Install** to install the Azure Storage package and dependencies. 
>
Also, you can refer to the [Azure Batch Hello World sample](https://code.msdn.microsoft.com/Azure-Batch-Sample-Hello-6573967c) on MSDN for a sample similar to the code discussed here.

###Concepts
The Batch service is used for scheduling scalable and distributed computation. You can use it to run large-scale workloads that are distributed among multiple virtual machines. These workloads provide efficient support for intensive computation. When you use the Batch service, you take advantage of the following resources:  

-	**Account** – A uniquely identified entity within the service. All processing occurs through a Batch account.
-	**Pool** – A collection of task virtual machines on which task applications run.
-	**Task Virtual Machine** – A machine that is assigned to a pool and is used by tasks that are assigned to jobs that run in the pool.
-	**Task Virtual Machine User** – A user account on a task virtual machine.
-	**Workitem** – Specifies how computation is performed on task virtual machines in a pool.
-	**Job** – A running instance of a work item. It consists of a collection of tasks.
-	**Task** – An application that is associated with a job and runs on a task virtual machine.
-	**File** – Contains the information that a task processes.  

Let's start with the most basic usage.

###Create an Azure Batch account
You can use the management portal to create a Batch account. A key is provided to you after you create the account. For more information, see [Azure Batch technical overview](http://azure.microsoft.com/en-us/documentation/articles/batch-technical-overview/).  

###How to: Add a pool to an account
A pool of task virtual machines is the first set of resources that you must create when you want to run tasks.  

1.	In Microsoft Visual Studio 2013, on the **File** menu, click **New**, and then click **Project**.

2.	From **Windows**, under **Visual C#**, click **Console Application**, name the project **GettingStarted**, name the solution **AzureBatch**, and then click **OK**.

3.	Add the following namespace declarations to the top of Program.cs.

		using Microsoft.Azure.Batch;
		using Microsoft.Azure.Batch.Auth;
		using Microsoft.Azure.Batch.Common;

	Make sure that you reference the Microsoft.Azure.Batch.dll assembly.

4.	Add the following variables to the Program class.

		private const string PoolName = "[name-of-pool]";
		private const int NumOfMachines = 3;
		private const string AccountName = "[name-of-batch-account]";
		private const string AccountKey = "[key-of-batch-account]";
		private const string Uri = "https://batch.core.windows.net";
	Replace the following values:
	-	**[name-of-pool]** – The name that you want to use for the pool.
	-	**[name-of-batch-account]** – The name of the Batch account.
	-	**[key-of-batch-account]** – The key that was provided to you for the Batch account.
5.	Add the following code to Main, to define the credentials to use.

		BatchCredentials cred = new BatchCredentials(AccountName, AccountKey);
6.	Add the following code to Main, to create the client that is used to perform operations.

		IBatchClient client = BatchClient.Connect(Uri, cred);
7.	Add the following code to Main, to create the pool if it doesn’t exist.

		using (IPoolManager pm = client.OpenPoolManager())
		{
		   IEnumerable<ICloudPool> pools = pm.ListPools();
		   if (!pools.Select(pool => pool.Name).Contains(PoolName))
		   {
		      Console.WriteLine("The pool does not exist, creating now...");
		      ICloudPool newPool = pm.CreatePool(
		         PoolName, 
		         osFamily: "3", 
		         vmSize: "small", 
		         targetDedicated: NumOfMachines);
		       newPool.Commit();                  
		    }
		}
		Console.WriteLine("Created pool {0}", PoolName);
		Console.ReadLine();
8.	Save and run the program. The status is **Active** for a pool that was added successfully.  

###How to: List the pools in an account
If you don’t know the name of an existing pool, you can get a list of them in an account.  

1.	Add the following code to Main, to define the credentials to use.  

		BatchCredentials cred = new BatchCredentials(AccountName, AccountKey);
2.	Add the following code to Main, to create the client that is used to perform operations.

		IBatchClient client = BatchClient.Connect(Uri, cred);

3.	Update the Main proc to the following code that writes the names and states of all pools in the account and creates the pool if it doesn't exist.

		BatchCredentials cred = new BatchCredentials(AccountName, AccountKey);
		IBatchClient client = BatchClient.Connect(Uri, cred);
		
		using (IPoolManager pm = client.OpenPoolManager())
		{
		    IEnumerable<ICloudPool> pools = pm.ListPools();
		
		    Console.WriteLine("Listing Pools\n=================");
		    foreach (var p in pools)
		    {
		        Console.WriteLine("Pool: " + p.Name + " State:" + p.State);
		    }  
		
		    if (!pools.Select(pool => pool.Name).Contains(PoolName))
		    {
		        Console.WriteLine("The pool does not exist, creating now...");
		        ICloudPool newPool = pm.CreatePool(
		           PoolName,
		           osFamily: "3",
		           vmSize: "small",
		           targetDedicated: NumOfMachines);
		        newPool.Commit();
		    }
		}
		Console.WriteLine("Created pool {0}. Press <Enter> to continue.", PoolName);
		Console.ReadLine();

4.	Save and run the program. You'll see the following output:

		Listing Pools
		=================
		Pool: gettingstarted State:Active
		Created pool gettingstarted. Press <Enter> to continue.

###How to: List the work items in an account
If you don’t know the name of an existing work item, you can get a list of them in an account.  

1.	Add the following code to end of Main, to write the names and states of all work items in the account.

		using (IWorkItemManager wm = client.OpenWorkItemManager())
		{
		   Console.WriteLine("Listing Workitems\n=================");
		   IEnumerable<ICloudWorkItem> workitems = wm.ListWorkItems();
		   foreach (var w in workitems)
		   {
		      Console.WriteLine("Workitem: " + w.Name + " State:" + w.State);
		   }   
		}
		Console.WriteLine("Press <Enter> to continue."); 
		Console.ReadLine();
7.	Save and run the program. You'll probably see nothing because you haven't submitted any work item. We'll talk about adding a work item in the next section.  

##How to: Add a work item to an account
You must create a work item to define how the tasks will run in the pool.  

1.	Add the following variables to the Program class.

		private static readonly string WorkItemName = Environment.GetEnvironmentVariable("USERNAME") + DateTime.Now.ToString("yyyyMMdd-HHmmss");

2.	When a work item is created, a job is also created. You can assign a name to the work item, but the job is always assigned the name of **job-0000000001**. Add the following code to Main (before the list work-item code), to add the work item.

		using (IWorkItemManager wm = client.OpenWorkItemManager())
		{
		   ICloudWorkItem cloudWorkItem = wm.CreateWorkItem(WorkItemName);
           cloudWorkItem.JobExecutionEnvironment = new JobExecutionEnvironment() {PoolName = PoolName};
		   cloudWorkItem.Commit();
		}
		Console.WriteLine("Workitem successfully added. Press <Enter> to continue.");
		Console.ReadLine();
3.	Save and run the program. The status is **Active** for a work item that was added successfully. You should see the following output:

		Listing Pools
		=================
		Pool: gettingstarted State:Active
		Created pool gettingstarted. Press <Enter> to continue.
		
		Workitem successfully added. Press <Enter> to continue.
		
		Listing Workitems
		=================
		Workitem: yidingz20141106-111211 State:Active
		Press <Enter> to continue.

###How to: Add tasks to a job
A work item without a task will do nothing. After you create the work item and the job is created, you can add tasks to the job. Let's add a simple task to the job.  

1.	Add the following variables to the Program class.

		private const int Count = 4; // number of tasks that will run
		private const string JobName = "job-0000000001";

2.	Add the following code to Main. This code adds tasks to a job, waits for them to run, and reports the results.

		using (IWorkItemManager wm = client.OpenWorkItemManager())
		{
		    string taskName;
		    ICloudJob job = wm.GetJob(WorkItemName, JobName);
		    for (int i = 1; i <= Count; ++i)
		    {
		        taskName = "taskdata" + i;
		        string commandLine = String.Format("cmd /c echo I am {0}", taskName);
		        ICloudTask taskToAdd = new CloudTask(taskName, commandLine);
		        job.AddTask(taskToAdd);
		        job.Commit();
		        job.Refresh();
		    }
		
		    ICloudJob listjob = wm.GetJob(WorkItemName, JobName);
		    client.OpenToolbox().CreateTaskStateMonitor().WaitAll(listjob.ListTasks(),
		       TaskState.Completed, new TimeSpan(0, 30, 0));
		    Console.WriteLine("The tasks completed successfully. Terminating the workitem...");
		    wm.GetWorkItem(WorkItemName).Terminate();
		    foreach (ICloudTask task in listjob.ListTasks())
		    {
		        Console.WriteLine("Task " + task.Name + " says:\n" + task.GetTaskFile(Constants.StandardOutFileName).ReadAsString());
		    }
		    Console.ReadLine();
		}

3.	Save and run the program. The result should look like this:

		The tasks completed successfully. Terminating the workitem...
		Task taskdata1 says:
		I am taskdata1
		
		Task taskdata2 says:
		I am taskdata2
		
		Task taskdata3 says:
		I am taskdata3

###Create a task-processing program
Now that you can run Hello World on a virtual machine, let's do something real. You'll create a task-processing program in this section and upload it to a task virtual machine that runs tasks.  

1.	In **Solution Explorer**, create a new console application project named **ProcessTaskData** in the **AzureBatch** solution.

2.	Add the following namespace declarations to the top of Program.cs.

		using System.Net;

3.	Add the following code to Main, to process data.

		string blobName = args[0];
		int numTopN = int.Parse(args[1]);
		
		WebClient myWebClient = new WebClient();
		string content = myWebClient.DownloadString(blobName);
		
		string[] words = content.Split(' ');
		var topNWords =
		   words.
		   Where(word => word.Length > 0).
		   GroupBy(word => word, (key, group) => new KeyValuePair<String, long>(key, group.LongCount())).
		   OrderByDescending(x => x.Value).
		   Take(numTopN).
		   ToList();
		
		foreach (var pair in topNWords)
		{
		    Console.WriteLine("{0} {1}", pair.Key, pair.Value);
		}

4.	Save and build the project.  

###Prepare resources for running a task

You will use the following basic workflow when you create a distributed computational scenario by using the Batch service:  

1.	Upload the files that you want to use in your distributed computational scenario to an Azure storage account. These files must be in the storage account so that the Batch service can access them. The files are loaded onto a task virtual machine when the task runs.
2.	Upload the dependent binary files to the storage account. The binary files include the program that the task runs and the dependent assemblies. These files must also be accessed from storage and are loaded onto the task virtual machine.
3.	Create a pool of task virtual machines. You can assign the size of the task virtual machine to use when the pool is created. When a task runs, it is assigned a virtual machine from this pool.
4.	Create a work item. You can use a work item to manage a job of tasks. A job is automatically created when you create a work item.
5.	Add tasks to the job. Each task uses the program that you uploaded to process information from a file that you uploaded.
6.	Monitor the results of the output.  

We have shown steps 3 through 6. Let's see how to prepare Azure Storage for running the task.

####Getting a storage account
You will need a storage account to complete the rest of this tutorial. If you don't know how to do this, see [Create an Azure Storage account](#tutorial1_storage).

####Uploading data

1. In the Azure management portal, create a container called **gettingstarted** in your Azure storage account. Make sure you set the **ACCESS** field to **Public Container**.

>[WACOM.NOTE] In a production environment, we recommend that you use a shared access signature.

2. Upload ProcessTaskData.exe to the container.

3. Create three text files (taskdata1.txt, taskdata2.txt, taskdata3.txt), with each containing one of the following paragraphs, and upload them to the container:

		You can use Azure Virtual Machines to provision on-demand, scalable compute infrastructure when you need flexible resources for your business. From the gallery, you can create virtual machines that run Windows, Linux, and enterprise applications such as SharePoint and SQL Server. Or, you can capture and use your own images to create customized virtual machines.
		
		Quickly deploy and manage powerful applications and services by using Azure Cloud Services. Simply upload your application, and Azure handles the deployment details--from provisioning and load balancing to health monitoring for continuous availability. Your application is backed by an industry-leading 99.95 percent monthly service level agreement (SLA). You focus on the application and not the infrastructure.
		
		Azure Websites provide a scalable, reliable, and easy-to-use environment for hosting web applications. Select from a range of frameworks and templates to create a website in seconds. Use any tool or operating system to develop your site by using .NET, PHP, Node.js, or Python. Choose from a variety of source control options, including Team Foundation Server (TFS), GitHub, and BitBucket, to set up continuous integration and develop as a team. Expand your site functionality over time by using additional Azure managed services like storage, Content Delivery Network (CDN), and SQL Database.

>[WACOM.NOTE] The Azure Storage team has a [blog post](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx) that lists Azure Storage explorers that can help you upload files.


###Update task submission code

1.	Add the following variables to the Program class.

		private const string BlobPath = "[storage-path]"; 
	Replace the following values:
	-	**[storage-path]** – The path to the blob in storage. For example: http://yiding.blob.core.windows.net/gettingstarted/

2. Update the task submission code as follows.

		string taskName;
		ICloudJob job = wm.GetJob(WorkItemName, JobName);
		for (int i = 1; i <= Count; ++i)
		{
		    taskName = "taskdata" + i;
		    IResourceFile programFile = new ResourceFile(BlobPath + "ProcessTaskData.exe", "ProcessTaskData.exe");
		    IResourceFile supportFile = new ResourceFile(BlobPath + taskName + ".txt", taskName);
		    string commandLine = String.Format("ProcessTaskData.exe {0} {1}", BlobPath + taskName + ".txt", 3);
		    ICloudTask taskToAdd = new CloudTask(taskName, commandLine);
		    taskToAdd.ResourceFiles = new List<IResourceFile>();
		    taskToAdd.ResourceFiles.Add(programFile);
		    taskToAdd.ResourceFiles.Add(supportFile);
		    job.AddTask(taskToAdd);
		}
		job.Commit(); 

3. Save and run the program. You should see:

		Listing Pools
		=================
		Pool: gettingstarted State:Active
		Created pool gettingstarted. Press <Enter> to continue.
		
		Workitem successfully added. Press <Enter> to continue.
		
		Listing Workitems
		=================
		Workitem: yidingz20141106-132140 State:Active
		Press <Enter> to continue.
		
		The tasks completed successfully. Terminating the workitem...
		Task taskdata1 says:
		can 3
		you 3
		and 3
		
		Task taskdata2 says:
		and 5
		application 3
		the 3
		
		Task taskdata3 says:
		a 5
		and 5
		to 3

###How to: Delete the pool and work item
After your work item has been processed, you can release resources by deleting the pool and work item.

####Delete the pool
1.	Add the following code to the end of Main, to delete the pool.

		using (IPoolManager pm = client.OpenPoolManager())
		{
		   pm.DeletePool(PoolName);
		}
		Console.WriteLine("Pool successfully deleted.");
		Console.ReadLine();
2.	Save and run the program.


####Delete the work item
1.	Add the following code to Main, to delete the work item.

		using (IWorkItemManager wm = client.OpenWorkItemManager())
		{
		   wm.DeleteWorkItem(WorkItemName);
		}
		Console.WriteLine("Workitem successfully deleted.");
		Console.ReadLine();
2.	Save and run the program.

###<a name="tutorial1_storage"></a>APPENDIX: Create an Azure storage account
Before you can run the code in this tutorial, you must have access to a storage account in Azure.  

1.	Sign in to the [Azure management portal](http://manage.windowsazure.com/).
2.	At the bottom of the navigation pane, click **NEW**.  
![][1]
3.	Click **DATA SERVICES**, click **STORAGE**, and then click **QUICK CREATE**.
![][2]

4.	In **URL**, type a subdomain name to use in the Uniform Resource Identifier (URI) for the storage account. The entry can contain from 3 to 24 lowercase letters and numbers. This value becomes the host name within the URI that is used to address Blob, Queue, or Table resources for the subscription.
5.	In **LOCATION/AFFINITY GROUP**, select a group in which to locate the storage.
6.	Optionally, you can enable geo-replication.
7.	Click **CREATE STORAGE ACCOUNT**.  

For more information about Azure Storage, see [How to use Blob Storage from .NET](http://www.windowsazure.com/en-us/develop/net/how-to-guides/blob-storage/).  


##<a name="tutorial2"></a>Tutorial 2: Azure Batch Apps Library for .NET
This tutorial shows you how to run parallel compute workloads on Azure Batch by using the Batch Apps service.

Batch Apps is a feature of Azure Batch that provides an application-centric way of managing and executing Batch workloads.  By using the Batch Apps software development kit (SDK), you can create packages to Batch-enable an application, and then deploy them in your own account or make them available to other Batch users. Batch also provides data management, job monitoring, built-in diagnostics and logging, and support for inter-task dependencies. It additionally includes a management portal where you can manage jobs, view logs, and download outputs without having to write your own client.

In the Batch Apps scenario, you write code by using the Batch Apps Cloud SDK to partition jobs into parallel tasks, describe any dependencies between these tasks, and specify how to run each task. This code is deployed to the Batch account. Clients can then run jobs simply by specifying the kind of job and the input files to a REST API.

>[WACOM.NOTE] To complete this tutorial, you need an Azure account. You can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial](http://www.windowsazure.com/en-us/pricing/free-trial/). You can use NuGet to obtain both the <a href="http://www.nuget.org/packages/Microsoft.Azure.Batch.Apps.Cloud/">Batch Apps Cloud</a> assembly and the <a href="http://www.nuget.org/packages/Microsoft.Azure.Batch.Apps/">Batch Apps Client</a> assembly. After you create your project in Visual Studio, right-click the project in **Solution Explorer**, and then click **Manage NuGet Packages**. You can also download the Visual Studio Extension for Batch Apps, which includes a project template to cloud-enable applications and ability to deploy an application <a href="https://visualstudiogallery.msdn.microsoft.com/8b294850-a0a5-43b0-acde-57a07f17826a">here</a> or via searching for **Batch Apps** in Visual Studio via the **Extensions and Updates** menu item. You can also find <a href="https://go.microsoft.com/fwLink/?LinkID=512183&clcid=0x409">end-to-end samples on MSDN</a>.
>

###Fundamentals of Azure Batch Apps 
Batch is designed to work with existing compute-intensive applications. It uses your existing application code and runs it in a dynamic, virtualized, general-purpose environment. To enable an application to work with Batch Apps, you must do the following:

1.	Prepare a compressed (.zip) file of your existing application executable files--the same executable files that would be run in a traditional server farm or cluster--and any support files it needs. This .zip file is then uploaded to your Batch account via the management portal or REST API.
2.	Create a .zip file of the "cloud assemblies" that dispatch your workloads to the application, and upload it via the management portal or REST API. A cloud assembly contains two components that are built against the Cloud SDK:
	1.	Job splitter, which breaks the job down into tasks that can be processed independently. For example, in an animation scenario, the job splitter would take a movie job and break it down into individual frames. 
	2.	Task processor, which invokes the application executable file for a task. For example, in an animation scenario, the task processor would invoke a rendering program to render the single frame that the task specifies. 
3.	Provide a way to submit jobs to the enabled application in Azure. This might be a plug-in in your application UI, a web portal, or even an unattended service as part of your execution pipeline. For examples, see the <a href="https://go.microsoft.com/fwLink/?LinkID=512183&clcid=0x409">samples</a> on MSDN.



###Batch Apps key concepts
The Batch Apps programming and usage model revolves around the following key concepts.

####Jobs
A **job** is a piece of work that the user submits. When a job is submitted, the user specifies the type of job, any settings for that job, and the data required for the job. Either the enabled implementation can work out these details on the user’s behalf, or in some cases, the user can provide this information explicitly via the client. A job has results that are returned. Each job has a primary output and an optional preview output. Jobs can also return extra outputs if desired.

####Tasks
A **task** is a piece of work to be done as part of a job. When a user submits a job, it is broken up into smaller tasks. The service processes these individual tasks and then assembles the task results into an overall job output. The nature of tasks depends on the kind of job. The job splitter defines how a job is broken down into tasks, guided by the knowledge of what chunks of work the application is designed to process. Task outputs can also be downloaded individually and might be useful in some cases. For example, a user may want to download individual tasks from an animation job.

####Merge tasks
A **merge task** is a special kind of task that assembles the results of individual tasks into the final job results. For a movie rendering job, the merge task might assemble the rendered frames into a movie or put all the rendered frames in a single .zip file. Every job has a merge task, even if no actual merging is needed.

####Files
A **file** is a piece of data used as an input to a job. A job might have no input file associated with it, or it might have one or many. The same file can be used in multiple jobs. For example, for a movie rendering job, the files might be textures or models. For a data analysis job, the files might be a set of observations or measurements.

###Enabling the cloud application
Your application must contain a static field or property that contains all the details of your application. It specifies the name of the application and the job type or job types that the application handles. This is provided when you’re using the template in the SDK that can be downloaded via Visual Studio Gallery. 

Here is an example of a cloud application declaration for a parallel workload.

	public static class ApplicationDefinition
	{
	    public static readonly CloudApplication App = new ParallelCloudApplication
	    {
	        ApplicationName = "ApplicationName",
	        JobType = "ApplicationJobType",
	        JobSplitterType = typeof(MyJobSplitter),
	        TaskProcessorType = typeof(MyTaskProcessor),
	    };
	}

####Implementing a job splitter and a task processor
For embarrassingly parallel workloads, you must implement a job splitter and a task processor. 

####Implementing JobSplitter.SplitIntoTasks
Your SplitIntoTasks implementation must return a sequence of tasks. Each task represents a separate piece of work that a compute node will queue for processing. Each task must be self-contained and must be set up with all the information that the task processor will need. 

The tasks that the job splitter specifies are represented as TaskSpecifier objects. TaskSpecifier has a number of properties that you can set before you return the task:
  

-	TaskIndex is ignored but is available to task processors. You can use this to pass an index to the task processor. If you don’t need to pass an index, you don’t need to set this property.
-	Parameters is an empty collection by default. You can add to it or replace it with a new collection. You can copy entries from the job parameters collection by using the WithJobParameters or WithAllJobParameters method. 


RequiredFiles is an empty collection by default. You can add to it or replace it with a new collection. You can copy entries from the job files collection by using the RequiringJobFiles or RequiringAllJobFiles method.

You can specify a task that depends on a specific other task. To do this, set the TaskSpecifier.DependsOn property, passing the ID of the task it depends on.

	new TaskSpecifier {
	    DependsOn = TaskDependency.OnId(5)
	}

The task will not run until the output of the depended-on task is available.   

You can also specify that a whole group of tasks should not start processing until another group has completely finished. In this case, you can set the TaskSpecifier.Stage property. Tasks that have a particular stage value will not begin processing until all tasks that have lower stage values have finished. For example, Stage 3 tasks will not begin processing until all Stage 0, 1, or 2 tasks have finished. Stages must begin at 0, the sequence of stages must have no gaps, and SplitIntoTasks must return tasks in stage order. For example, it is an error to return a Stage 0 task after a Stage 1 task.

Every parallel job ends with a special task called the merge task. The merge task’s job is to assemble the results of the parallel processing tasks into a final result. Batch Apps automatically creates the merge task for you.  

In rare cases, you may want to explicitly control the merge task--for example, to customize its parameters. In this case, you can specify the merge task by returning TaskSpecifier with its IsMerge property set to true. This will override the automatic merge task. If you create an explicit merge task:  

-	You can create only one explicit merge task.
-	It must be the last task in the sequence.
-	It must have IsMerge set to true, and every other task must have IsMerge set to false.  


Remember, however, that normally you do not need to create the merge task explicitly.  

The following code demonstrates a simple implementation of SplitIntoTasks.  

	protected override IEnumerable<TaskSpecifier> SplitIntoTasks(
	    IJob job,
	    JobSplitSettings settings)
	{
	    int start = Int32.Parse(job.Parameters["startIndex"]);
	    int end = Int32.Parse(job.Parameters["endIndex"]);
	    int count = (end - start) + 1;
	 
	    // Processing tasks
	    for (int i = 0; i < count; ++i)
	    {
	        yield return new TaskSpecifier
	        {
	            TaskIndex = start + i,
	        }.RequiringAllJobFiles();
	    }
	}
####Implementing ParallelTaskProcessor.RunExternalTaskProcess

RunExternalTaskProcess is called for each non-merge task returned from the job splitter. It should invoke your application with the appropriate arguments, and return a collection of outputs that need to be kept for later use.

The following fragment shows how to call a program named application.exe. Note that you can use the ExecutablePath helper method to create absolute file paths.

The ExternalProcess class in the Cloud SDK provides useful helper logic for running your application executable files. ExternalProcess can take care of cancellation, translating exit codes into exceptions, capturing standard output and standard error, and setting up environment variables. You can also use the .NET Process class directly to run your programs, if you prefer.

Your RunExternalTaskProcess method returns TaskProcessResult, which includes a list of output files. This must include at minimum all files required for the merge. You can also optionally return log files, preview files, and intermediate files (for example, for diagnostic purposes if the task failed). Note that your method returns the file paths, not the file content.

Each file must be identified with the type of output it contains: output (that is, part of the eventual job output), preview, log, or intermediate. These values come from the TaskOutputFileKind enumeration. The following fragment returns a single task output, and no preview or log. The TaskProcessResult.FromExternalProcessResult method simplifies the common scenario of capturing exit code, processor output, and output files from a command-line program.

The following code demonstrates a simple implementation of ParallelTaskProcessor.RunExternalTaskProcess.

	protected override TaskProcessResult RunExternalTaskProcess(
	    ITask task,
	    TaskExecutionSettings settings)
	{
	    var inputFile = LocalPath(task.RequiredFiles[0].Name);
	    var outputFile = LocalPath(task.TaskId.ToString() + ".jpg");
	 
	    var exePath = ExecutablePath(@"application\application.exe");
	    var arguments = String.Format("-in:{0} -out:{1}", inputFile, outputFile);
	 
	    var result = new ExternalProcess {
	        CommandPath = exePath,
	        Arguments = arguments,
	        WorkingDirectory = LocalStoragePath,
	        CancellationToken = settings.CancellationToken
	    }.Run();
	
	    return TaskProcessResult.FromExternalProcessResult(result, outputFile);
	}
####Implementing ParallelTaskProcessor.RunExternalMergeProcess

RunExternalMergeProcess is called for the merge task. It should invoke the application to combine outputs of the previous tasks, in whatever way is appropriate to your application, and return the combined output. 

The implementation of RunExternalMergeProcess is very similar to the implementation of RunExternalTaskProcess, except that:  

-	RunExternalMergeProcess consumes the outputs of previous tasks, rather than user input files. You should therefore work out the names of the files you want to process based on the task ID, rather than from the Task.RequiredFiles collection.
-	RunExternalMergeProcess returns a JobOutput file, and optionally a JobPreview file.


The following code demonstrates a simple implementation of ParallelTaskProcessor.RunExternalMergeProcess.

	protected override JobResult RunExternalMergeProcess(
	    ITask mergeTask,
	    TaskExecutionSettings settings)
	{
	    var outputFile =  "output.zip";
	 
	    var exePath =  ExecutablePath(@"application\application.exe");
	    var arguments = String.Format("a -application {0} *.jpg", outputFile);
	 
	    new ExternalProcess {
	        CommandPath = exePath,
	        Arguments = arguments,
	        WorkingDirectory = LocalStoragePath
	    }.Run();
	 
	    return new JobResult {
	        OutputFile = outputFile
	    };
	}

###Submitting jobs to Batch Apps 
A job describes a workload to be run and must include all the information about the workload, except for file content. For example, the job contains configuration settings that flow from the client through the job splitter and on to tasks. The samples provided on MSDN are examples of how to submit jobs and provide multiple clients, including a web portal and a command-line client. 
 

<!--Anchors-->


<!--Image references-->
[1]: ./media/batch-dotnet-get-started/batch-dotnet-get-started-01.jpg
[2]: ./media/batch-dotnet-get-started/batch-dotnet-get-started-02.jpg
[3]: ./media/batch-dotnet-get-started/batch-dotnet-get-started-03.jpg
[4]: ./media/batch-dotnet-get-started/batch-dotnet-get-started-04.jpg


