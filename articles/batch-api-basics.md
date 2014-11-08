
<properties title="API basics for Azure Batch" pageTitle="API basics for Azure Batch" description="Concepts to introduce developers to the Azure Batch APIs and Batch service" metaKeywords="" services="batch" solutions="" documentationCenter=".NET" authors="yidingz, karran.batta" videoId="" scriptId="" manager="timlt" />

<tags ms.service="batch" ms.devlang="multiple" ms.topic="article" ms.tgt_pltfrm="na" ms.workload="big-compute" ms.date="10/27/2014" ms.author="yidingz, karran.batta" />


<!--The next line, with one pound sign at the beginning, is the page title--> 
# API basics for Azure Batch

The Azure Batch service provides a job-scheduling framework for scalable and distributed computation. The Batch service maintains a set of virtual machines that are located across different clusters and datacenters in Azure. The Batch service accomplishes distributed computation by running one or more programs either on demand or as scheduled for a specific time on a specified collection of these virtual machines. The Batch service manages these virtual machines to run your computation tasks according to the resource requirements, specifications, and constraints that you provide.

By using the Batch service, you can eliminate the need to write code for queuing, scheduling, allocating, and managing compute resources. You can then focus on the specific application and not worry about the complexity of job scheduling and resource management in the underlying platform. The Batch service can optimize the location of these jobs and their access to the data they must process.

The following are some of the scenarios that you can enable by using the Batch service:

- Computationally intensive parallel processing

- Daily cleanup of files

- Batch processing

You can learn more about the Batch service by reading the following sections:

- [Resources of the Batch service](#resource)

- [Workflow of the Batch service](#workflow)

- [Files and directories](#file)

- [Scaling applications](#scaling)

- [Certificates for applications](#cert)

- [Scheduling priority](#scheduling)

- [Environment settings for tasks](#environment)

## <a name="resource"></a> Resources of the Batch service

When you use the Batch service, you take advantage of the following resources:

- [Account](#account)

- [Task virtual machine](#taskvm)

- [Pool](#pool)

- [Work item](#workitem)

- [Job](#job)

- [Task](#task)

### <a name="account"></a>Account

A Batch account is a uniquely identified entity within the Batch service. All processing occurs through a Batch account. When you perform operations by using the Batch service, you need the name of the account and the key for the account. Currently, you must contact the Azure Batch team to create a new account. After the account is created, a key is provided to you.

### <a name="taskvm"></a>Task Virtual Machine

A task virtual machine (TVM) is an Azure virtual machine that is dedicated to a specific workload for your application. The size of a TVM determines the number of CPU cores, the memory capacity, and the local file system size that is allocated to the TVM. A TVM can be a small, large, or extra-large virtual machine, as described in [Virtual Machine and Cloud Service Sizes for Azure](http://msdn.microsoft.com/en-us/library/dn197896.aspx).

The types of programs that a TVM can run include executable files (.exe), command files (.cmd), batch files (.bat), and script files. A TVM also has the following attributes:

- File system folders that are both task-specific and shared

- Stdout.txt and Stderr.txt files that are written to a task-specific folder

- Environment variables for processing

- Firewall settings that are configured to control access

### <a name="pool"></a>Pool

A pool is a collection of TVMs on which your application runs. You can create the pool, or the Batch service automatically creates the pool when you specify the work to be accomplished. You can create and manage a pool that meets the needs of your application. A pool can be used only by the Batch account in which it was created. A Batch account can have more than one pool.

Every TVM that is added to a pool is assigned a unique name and an associated IP address. When a TVM is removed from a pool, it loses the changes that were made to the operating system, all of its local files, its name, and its IP address. When a TVM leaves a pool, its lifetime is over.

You can configure a pool to allow communication between TVMs within it. If intra-pool communication is requested for a pool, the Batch service enables ports greater than 1100 on each TVM in the pool. Each TVM in the pool is configured to allow and restrict incoming connections to just this port range and only from other TVMs in the pool. If your application does not require communication between TVMs, the Batch service can potentially allocate a large number of TVMs across different clusters or datacenters to the pool to enable more parallel processing.

When you create a pool, you can specify the following attributes:

- The size of TVMs in the pool

- The operating system family and version that runs on the TVMs

- The target number of TVMs that should be available for the pool

- The scaling policy for the pool

- The communication status of the TVMs in the pool

- The start task for TVMs in the pool

When you create a pool, you can specify the storage account with which the pool should be associated. The Batch service allocates TVMs from the datacenters that have better network connectivity and bandwidth capacity to the specified storage account. This enables workloads to access data more effectively.

### <a name="workitem"></a>Work item

A work item specifies how computation is performed on TVMs in a pool. A work item includes the following configuration items:

1.The program that performs the processing on the TVM

2.The command-line parameters for the program

3.The priority of the computation

4.The schedule of the computation, which includes defining whether the computation should reoccur

A work item is always assigned to a pool. The pool can already exist, or it can be automatically created when the work item is created.

### <a name="job"></a>Job

A job is a running instance of a work item and consists of a collection of tasks. The Batch service creates and instance of a job based on the configuration of the work item. The job uses TVMs from the pool that is associated with the work item.

### <a name="task"></a>Task

A task is a unit of computation that is associated with a job and runs on a TVM. A task uses the following resources:

- The program that was specified in the work item.

- The resource files that contain the data to be processed. These files are automatically copied to the TVM from blob storage. For more information, see Files and directories.

- The environment settings that the program needs. For more information, see Environment settings for tasks.

- The constraints in which the computation should occur. For example, constraints can include the maximum time in which the task is allowed to run, the maximum number of times that a task should be tried again if it fails to run, and the maximum time that files in the working directory are retained.

In addition to tasks that you can define to perform computation on a TVM, you can use the following special tasks that the Batch service provides:

- [Start task](#starttask)

- [Job manager task](#jobmanagertask)

#### <a name="starttask"></a>Start task

You can configure the operating system of TVMs in a pool by associating a start task with the pool. Installing software and starting background processes are two of the actions that a start task can perform. The start task runs every time a TVM starts for as long as it remains in the pool.

A start task is defined by adding an XML section to the request body for the Add Pool operation. The following example shows a basic definition of a start task.

	{
		“commandLine”:”mypoolsetup.exe”,
		“resourceFiles”:
		[
			{
				“blobSource”:”http://account.blob.core.windows.net/container/myapp1.exe?st=2013-08-09T08%3a49%3a37.0000000Z&se=2013-08-10T08%3a49%3a37.0000000Z&sr=c&sp=d&si=YWJjZGTVMZw%3d%3d&sig= %2bSzBm0wi8xECuGkKw97wnkSZ%2f62sxU%2b6Hq6a7qojIVE%3d”,
				“filePath”:”mypoolsetup.exe”
			},
			{
				“blobSource”:”http://account.blob.core.windows.net/container/myapp2.exe?st=2013-08-09T08%3a49%3a37.0000000Z&se=2013-08-10T08%3a49%3a37.0000000Z&sr=c&sp=d&si=YWJjZGTVMZw%3d%3d&sig= %2bSzBm0wi8xECuGkKw97wnkSZ%2f62sxU%2b6Hq6a7qojIVE%3d”,
				“filePath”:”myapp2.exe”
			}
		],
		“maxTaskRetryCount”:0
	}


#### <a name="jobmanagertask"></a>Job manager task

A job manager task is started before all other tasks. The job manager task provides the following benefits:

- It is automatically created by the Batch service when the job is created.

- It is scheduled before other tasks in the job.

- Its associated TVM is the last to be removed from a pool when the pool is being downsized.

- It is given the highest priority when it needs to be restarted. If an idle TVM is not available, the Batch service may terminate one of the running tasks in the pool to make room for it to run.

- Its termination can be tied to the termination of all tasks in the job.

A job manager task in a job does not have priority over tasks in other jobs. Across jobs, only job-level priorities are observed. A job manager task is defined by adding an XML section to the request body for the Add Workitem operation. The following example shows a basic definition of a job manager task.

	{
		“name”:”jmTask”,
		“commandLine”:”myapp1.exe”,
		“resourceFiles”:
		[
			{
				“blobSource”:”http://account.blob.core.windows.net/container/myapp1.exe?st=2013-08-09T08%3a49%3a37.0000000Z&se=2013-08-10T08%3a49%3a37.0000000Z&sr=c&sp=d&si=YWJjZGTVMZw%3d%3d&sig= %2bSzBm0wi8xECuGkKw97wnkSZ%2f62sxU%2b6Hq6a7qojIVE%3d”,
				“filePath”:”myapp1.exe”
			},
			{
				“blobSource”:”http://account.blob.core.windows.net/container/myapp2.exe?st=2013-08-09T08%3a49%3a37.0000000Z&se=2013-08-10T08%3a49%3a37.0000000Z&sr=c&sp=d&si=YWJjZGTVMZw%3d%3d&sig= %2bSzBm0wi8xECuGkKw97wnkSZ%2f62sxU%2b6Hq6a7qojIVE%3d”,
				“filePath”:”myapp2.exe”
			}
		],
		“taskConstraints”:
		{
			“maxWallClockTime”:”PT1H”,
			“maxTaskRetryCount”:0,
			“retentionTime”:”PT1H”
		},
		“killJobOnCompletion”:true,
		“runElevated”:false,
		“runExclusive”:true
	}




## <a name="workflow"></a>Workflow of the Batch service

You need a Batch account to use the Batch service, and you use multiple resources of the service to schedule computation. You use the following basic workflow when you create a distributed computational scenario by using the Batch service:

1.Upload the files that you want to use in your distributed computational scenario to an Azure storage account. These files must be in the storage account so that the Batch service can access them. The Batch service loads them onto a TVM when the task runs.

2.Upload the dependent binary files to the storage account. The binary files include the program that the task runs and the dependent assemblies. These files must also be accessed from storage and are loaded onto the TVM.

3.Create a pool of TVMs. You can assign the size of the TVM to use when the pool is created. When a task runs, it is assigned a TVM from this pool.

4.Create a work item. A job is automatically created when you create a work item. You can use a work item to manage a job of tasks.

5.Add tasks to the work item. Each task uses the program that you uploaded to process information from a file that you uploaded.

6.Monitor the results of the output.

## <a name="files"></a>Files and directories

Each task has a working directory under which it creates zero or more directories and files for storing the program that a task runs, the data that a task processes, and the output of the processing that a task performs. These directories and files are then available for other tasks to use during the running of a job. A single user account owns all tasks, files, and directories on a TVM.

The Batch service exposes a portion of the file system on a TVM as the root directory. The root directory of the TVM is available to a task through the WATASK\_TVM\_ROOT\_DIR environment variable. For more information about using environment variables, see Environment settings for tasks.

The root directory contains the following subdirectories:

- **Tasks** – This location is where all of the files that belong to tasks that run on the TVM are stored. For each task, the Batch service creates a working directory that has a unique path in the form of %WATASK\_TVM\_ROOT\_DIR%/tasks/workitemName/jobName/taskName/. This directory provides read/write access to the task. The task can create, read, update, and delete files under this directory, and this directory is retained based on the RetentionTime constraint specified for the task.

- **Shared** – This location is a shared directory for all of the tasks under the account. On the TVM, the shared directory is at %WATASK\_TVM\_ROOT\_DIR%/shared. This directory provides read/write access to the task. The task can create, read, update, and delete files under this directory.

- **Start** – This location is used by a start task as its working directory. All of the files that the Batch service downloads to begin the start task are also stored under this directory. On the TVM, the start directory is at %WATASK\_TVM\_ROOT\_DIR%/start. The task can create, read, update, and delete files under this directory. Start tasks can use this directory to configure the operating system.

When a TVM is removed from the pool, all of the files that are stored on the TVM are removed.

## <a name="scaling"></a>Scaling applications

Your application can be automatically scaled up or down to accommodate the computation that you need. You can dynamically adjust the number of TVMs in a pool according to the current workload and resource usage statistics. You can also optimize the overall cost of running your application by configuring it to be automatically scaled. You can specify the scaling settings for a pool when it is created, and you can update the configuration at any time.

You specify automatic scaling of an application by using a set of scaling formulas. The formulas can be used to determine the number of TVMs that are in the pool for the next scaling interval. For example, assume that you need to submit a large number of tasks to be scheduled on a pool. You can assign a scaling formula to the pool to specify the size of the pool based on the current number of pending tasks and the completion rate of the tasks. The Batch service periodically evaluates the formula and resizes the pool based on the workload.

A formula can be based on the following metrics:

- **Time metrics** – Based on statistics collected every five minutes in the specified number of hours.

- **Resource metrics** – Based on CPU usage, bandwidth usage, memory usage, and number of TVMs.

- **Task metrics** – Based on the status of tasks, such as Active, Pending, and Completed.

For more information about automatically scaling an application, see Configure Autoscaling of Task Virtual Machines.

## <a name="cert"></a>Certificates for applications

You typically need to use certificates when you encrypt secret information. Certificates can be installed on TVMs. The encrypted secrets are passed to tasks in command-line parameters or embedded in one of the resources, and installed certificates can be used to decrypt them. An example of secret information is the key for a storage account.

You use the Add Certificate operation to add a certificate to a Batch account. You can then associate the certificate to a new or existing pool. When a certificate is associated with a pool, the Batch service installs the certificate on each TVM in the pool. The Batch service installs the appropriate certificates when the TVM starts up, before it begins any tasks, including start tasks and job manager tasks.

## <a name="scheduling"></a>Scheduling priority

When you create a work item, you can assign a priority to it. Each job under the work item is created with this priority. The Batch service uses the priority values of the job to determine the order of job scheduling within an account. The priority values can range from -1000 (the lowest priority) to 1000 (the highest priority). You can update the priority of a job by using the Update Job operation.

Within the same account, higher-priority jobs have scheduling precedence over lower-priority jobs. A job that has a higher priority value in one account does not have scheduling precedence over another job that has a lower priority value in a different account.

Job scheduling in different pools is independent. Across different pools, it is not guaranteed that a higher-priority job is scheduled first, if its associated pool is short of idle TVMs. In the same pool, jobs that have the same priority level have an equal chance of being scheduled.

## <a name="environment"></a>Environment settings for tasks

You can specify environment settings that can be used in the context of a task. Environment settings for a start task and for tasks running under a job are defined by adding an XML section to the request body of the Add Task or Update Task operation.

The following example shows the definition of an environment setting:

For every task that is scheduled under a job, the Batch service sets specific environment variables. The following table lists the environment variables that the Batch service sets for all tasks.

<table>
  <tr>
   <th>Environment variable name
   </th>
   <th>
   Description
   </th>
  </tr>
 <tr>
  <td>
  WATASK_ACCOUNT_NAME
  </td>
  <td>
  The name of the account to which the task belongs
  </td>
 </tr>
 <tr>
  <td>
  WATASK_WORKITEM_NAME
  </td>
  <td >The name of the work item to which the task belongs
  </td>
 </tr>
 <tr>
  <td>
  WATASK_JOB_NAME
  </td>
  <td >The name of the job to which the task belongs
  </td>
 </tr>
 <tr>
  <td>
  WATASK_TASK_NAME
  </td>
  <td >The name of the current task
  </td>
 </tr>
 <tr>
  <td>
  WATASK_POOL_NAME
  </td>
  <td >The name of the pool on which the task is running
  </td>
 </tr>
 <tr>
  <td>
  WATASK_TVM_NAME
  </td>
  <td >The name of the TVM on which the task is running
  </td>
 </tr>
 <tr>
  <td>
  WATASK_TVM_ROOT_DIR
  </td>
  <td >The full path of the root directory on the TVM
  </td>
 </tr>
 <tr>
  <td>
  WATASK_PORTRANGE_LOW
  WATASK_PORTRANGE_HIGH
</td>
<td>
  The port range (inclusive on both ends) that the task can use for communication when intra-pool communication is enabled
</td>
</table>


**Note** 

You cannot overwrite these system-defined variables.

You can retrieve the value of environment settings by using the Get Task operation.
