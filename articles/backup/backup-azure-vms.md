<properties
	pageTitle="Azure virtual machine backup - Backup"
	description="Learn how to backup an Azure virtual machine after registration"
	services="backup"
	documentationCenter=""
	authors="aashishr"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="05/26/2015"
	ms.author="aashishr"/>


# Back up Azure virtual machines

This article is the essential guide to backing up Azure virtual machines. Before you proceed, ensure that all the [prerequisites](backup-azure-vms-introduction.md#prerequisites) have been met.

Backing up Azure virtual machines involves three key steps:

![Three steps to back up an Azure virtual machine](./media/backup-azure-vms/3-steps-for-backup.png)

## Discover Azure virtual machines
The discovery process queries Azure for the list of virtual machines in the subscription, along with additional information like the cloud service name and the region.

> [AZURE.NOTE] The discovery process should always be run as the first step. This is to ensure that any new virtual machines added to the subscription are identified.

To trigger the discovery process, do the following steps:

1. Navigate to the backup vault, which can be found under **Recovery Services** in the Azure portal, and then click the **Registered Items** tab.

2. Choose the type of workload in the drop-down menu as **Azure Virtual Machine**, and then click the **Select** button.
  ![select workload](./media/backup-azure-vms/discovery-select-workload.png)

3. Click the **DISCOVER** button at the bottom of the page.
  ![discover button](./media/backup-azure-vms/discover-button.png)

4. The discovery process can run for a few minutes while the virtual machines are being tabulated. A toast notification at the bottom of the screen appears while the discovery process is running.
  ![discover vms](./media/backup-azure-vms/discovering-vms.png)

5. After the discovery process is complete, a toast notification appears.
  ![discover-done](./media/backup-azure-vms/discovery-complete.png)


## Register Azure virtual machines
Before a virtual machine can be protected, it needs to be registered with the Azure Backup service. The registration process has two primary goals:

1. To have the backup extension plugged in to the VM agent in the virtual machine

2. To associate the virtual machine with the Azure Backup service

Registration is typically a one-time activity. The Azure Backup service seamlessly handles the upgrade and patching of the backup extension without requiring any cumbersome user intervention. This relieves the user of the “agent management overhead” that is typically associated with backup products.

### To register virtual machines

1. Navigate to the backup vault, which can be found under **Recovery Services** in the Azure portal, then click the **Registered Items** tab.

2. Choose the type of workload in the drop-down menu as **Azure Virtual Machine**, and then click the **Select** button.
  ![select workload](./media/backup-azure-vms/discovery-select-workload.png)

3. Click the **REGISTER** button at the bottom of the page.
  ![register button](./media/backup-azure-vms/register-button.png)

4. In the **Register Items** pop-up, choose the virtual machines that you want to register. If there are two or more virtual machines with the same name, use the cloud service to distinguish between the virtual machines.

    The **Register** operation can be done at scale, which means that multiple virtual machines can be selected at one time to be registered. This greatly reduces the one-time effort spent in preparing the virtual machine for backup.

    >[AZURE.NOTE] Only the virtual machines that are not registered, and are in the same region as the backup vault, will show up.

5. A job is created for each virtual machine that should be registered. The toast notification shows the status of this activity. Click **View Job** to go to the **Jobs** page.
  ![register job](./media/backup-azure-vms/register-create-job.png)

6. The virtual machine also appears in the list of registered items, and the status of the registration operation is shown
  ![Registering status 1](./media/backup-azure-vms/register-status01.png)

7. After the operation is completed, the status in the portal will change to reflect the registered state.
  ![Registration status 2](./media/backup-azure-vms/register-status02.png)

## Back up Azure virtual machines
Protecting a virtual machine involves setting up a backup and retention policy for it. To protect a virtual machine, do the following steps:

1. Navigate to the backup vault, which can be found under **Recovery Services** in the Azure portal, and then click the **Registered Items** tab.
2. Choose the type of workload in the drop-down menu as **Azure Virtual Machine**, and then click the **Select** button.
  ![Select workload in portal](./media/backup-azure-vms/select-workload.png)

3. Click the **PROTECT** button at the bottom of the page.

4. This will bring up a Protect Items wizard, where the virtual machines to be protected can be selected. If there are two or more virtual machines with the same name, use the cloud service to distinguish between the virtual machines.

    The **Protect** operation can be done at scale, which means that multiple virtual machines can be selected at one time to be registered. This greatly reduces the effort spent in protecting the virtual machine.

    >[AZURE.NOTE] Only the virtual machines that have been registered correctly with the Azure Backup service, and are in the same region as the backup vault, will show up here.

5. On the second screen of the Protect Items wizard, choose a backup and retention policy to back up the selected virtual machines. Pick from an existing set of policies or define a new one.

    >[AZURE.NOTE] For preview, up to 30 days of retention and a maximum of once-a-day backup are supported.

    In each backup vault, you can have multiple backup policies. The policies reflect the details about how the backup should be scheduled and retained. For example, one backup policy could be for daily backup at 10:00 PM, while another backup policy could be for weekly backup at 6:00 AM. Multiple backup policies allow flexibility in scheduling backups for your virtual machine infrastructure.

    Each backup policy can have multiple virtual machines that are associated with the policy. The virtual machines can be associated with only one policy at any point in time.

6. A job is created for each virtual machine to configure the protection policy and to associate the virtual machines with the policy. Click the **Jobs** tab and choose the right filter to view the list of **Configure Protection** jobs.
  ![Configure protection job](./media/backup-azure-vms/protect-configureprotection.png)

7. Once completed, the virtual machines are protected with a policy and must wait until the scheduled backup time for the initial backup to be completed. The virtual machine will now appear under the **Protected Items** tab and will have a Protected Status of *Protected* (pending initial backup).
    >[AZURE.NOTE] Starting the initial backup immediately after configuring protection is not available as an option today.

8. At the scheduled time, the Azure Backup service creates a backup job for each virtual machine that needs to be backed up. Click the **Jobs** tab to view the list of **Backup** jobs. As a part of the backup operation, the Azure Backup service issues a command to the backup extension in each virtual machine to flush all writes and take a consistent snapshot.
  ![Backup in progress](./media/backup-azure-vms/protect-inprogress.png)

9. Once completed, the protection status of the virtual machine on the **Protected Items** tab will appear as *Protected*.
  ![Virtual machine is backed up with recovery point](./media/backup-azure-vms/protect-backedupvm.png)

## Viewing backup status and details
After a virtual machine is protected, the virtual machine count increases in the **Dashboard** page summary. In addition, the **Dashboard** page shows the number of jobs from the last 24 hours that were successful, have failed, and are still in progress. Clicking any one category will drill down into that category on the **Jobs** page.
  ![Status of backup in Dashboard page](./media/backup-azure-vms/dashboard-protectedvms.png)

## Troubleshooting errors
You can use the information in the following table to troubleshoot errors encountered while using Azure Backup.

| Backup operation | Error details | Workaround |
| -------- | -------- | -------|
| Discovery | Failed to discover new items - Microsoft Azure Backup encountered and internal error. Wait for a few minutes and then try the operation again. | Retry the discovery process after 15 minutes.
| Discovery | Failed to discover new items – Another Discovery operation is already in progress. Please wait until the current Discovery operation has completed. | None |
| Register | Azure VM role is not in a state to install extension – Please check if the VM is in the Running state. Azure Recovery Services extension requires the VM to be Running. | Start the virtual machine, and when it is in the running state, retry the register operation.|
| Register | Number of data disks attached to the virtual machine exceeded the supported limit - Please detach some data disks on this virtual machine and retry the operation. Azure backup supports up to 5 data disks attached to an Azure virtual machine for backup | None |
| Register | Microsoft Azure Backup encountered an internal error - Wait for a few minutes and then try the operation again. If the issue persists, contact Microsoft Support. | You can get this error due to one of the following unsupported configurations: <ol><li>Premium LRS <li>Multi NIC <li>Load balancer </ol> |
| Register | VM Guest Agent Certificate not found | Follow these instructions to resolve the error: <ol><li>Download the latest version of the VM Agent from [here](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). Ensure that the version of the downloaded agent is 2.6.1198.718 or higher. <li>Install the VM Agent in the virtual machine.</ol> [Learn](#validating-vm-agent-installation) how to check the version of the VM Agent. |
| Register | Registration failed with Install Agent operation timeout | Check whether the operating system version of the virtual machine is supported. |
| Register | Command execution failed - Another operation is in progress on this item. Please wait until the previous operation is completed | None |
| Backup | Copying VHDs from backup vault timed out - Please retry the operation in a few minutes. If the problem persists, contact Microsoft Support. | This happens when there is too much data to be copied. Check whether you have fewer than six data disks. |
| Backup | Snapshot VM sub task timed out - Please retry the operation in a few minutes. If the problem persists, contact Microsoft Support | This error is thrown if there is a problem with the VM Agent or network access to the Azure infrastructure is blocked in some way. <ul><li>Learn about [debugging VM Agent issues](#Troubleshooting-vm-agent-related-issues) <li>Learn about [debugging networking issues](#troubleshooting-networking-issues) </ul> |
| Backup | Backup failed with an internal error - Please retry the operation in a few minutes. If the problem persists, contact Microsoft Support | You can get this error for two reasons: <ol><li> There is too much data to be copied. Please check whether you have fewer than six disks. <li>The original VM has been deleted and therefore backup cannot be taken. To keep the backup data for a deleted virtual machine but stop the backup errors, unprotect the virtual machine and choose the option to keep the data. This will stop the backup schedule and the recurring error messages. |
| Backup | Failed to install the Azure Recovery Services extension on the selected item - VM Agent is a pre-requisite for Azure Recovery Services Extension. Please install the Azure VM agent and restart the registration operation | <ol> <li>Check whether the VM Agent has been installed correctly. <li>Ensure that the flag on the VM config is set correctly.</ol> [Read more](#validating-vm-agent-installation) about VM Agent installation, and how to validate the VM Agent installation. |
| Backup | Command execution failed - Another operation is currently in progress on this item. Please wait until the previous operation is completed, and then retry | An existing backup or restore job for the virtual machine is running, and a new job cannot be started while the existing job is running. <br><br>If you want the option to cancel an ongoing job, add your vote to the [Azure Feedback forum](http://feedback.azure.com/forums/258995-azure-backup-and-scdpm/suggestions/7941501-add-feature-to-allow-cancellation-of-backup-restor). |

### Troubleshooting VM Agent issues

#### Setting up the VM Agent
Typically, the VM Agent is already present in virtual machines that are created from the Azure Marketplace. However, virtual machines that are migrated from on-premises datacenters would not have the VM Agent installed. For such virtual machines, the VM Agent needs to be installed explicitly. Read more about [installing the VM Agent on an existing VM](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx).

For Windows virtual machines:

- Download and install the [agent MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). You will need administrator privileges to complete the installation.
- [Update the VM property](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx) to indicate that the agent is installed.


#### Updating the VM Agent
Updating the VM Agent is as simple as reinstalling the [VM Agent binaries](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). However, you need to ensure that no backup operation is running while the VM Agent is being updated.


#### Validating VM Agent installation
To check for the VM Agent version on Windows virtual machines:

- Log on to the Azure virtual machine and navigate to the folder C:\WindowsAzure\Packages.
- You should find the WaAppAgent.exe file present.
- Right-click the file, go to **Properties**, and then click the **Details** tab.
- The **Product Version** field should be 2.6.1198.718 or higher.

### Troubleshooting networking issues
Like all extensions, the backup extension needs access to the public Internet to work. Not having access to the public Internet can manifest itself in a variety of of ways:

- The extension installation can fail.
- The backup operations (like disk snapshot) can fail.
- Displaying the status of the backup operation can fail.

The need for resolving public Internet addresses has been articulated [here](http://blogs.msdn.com/b/mast/archive/2014/06/18/azure-vm-provisioning-stuck-on-quot-installing-extensions-on-virtual-machine-quot.aspx). You will need to check the DNS configurations for the virtual network and ensure that the Azure URIs can be resolved.

After the name resolution is done correctly, access to the Azure IPs also needs to be provided. To unblock access to the Azure infrastructure, follow these steps:

1. Get the list of [Azure datacenter IPs](https://msdn.microsoft.com/library/azure/dn175718.aspx) to be whitelisted.
2. Unblock the IPs by using the [New-NetRoute](https://technet.microsoft.com/library/hh826148.aspx) cmdlet. Run this cmdlet within the Azure virtual machine, in an elevated Azure PowerShell window (run as administrator).


## Consistency of recovery points
When dealing with backup data, customers worry about the behavior of the virtual machine after it has been restored. The typical questions that customers ask are:

- Will the virtual machine boot up?
- Will the data be available on the disk (or) is there any data loss?
- Will the application be able to read the data (or) is the data corrupted?
- Will the data make sense to the application (or) is the data self-consistent when read by the application?

The following table explains the types of consistency that are encountered during backup and restore of an Azure virtual machine:

| Consistency | VSS based | Explanation and details |
|-------------|-----------|---------|
| Application consistency | Yes | This is the ideal place to be for Microsoft workloads, because it ensures that:<ol><li> The virtual machine *boots up*. <li>There is *no corruption*. <li>There is no *data loss*. <li> The data is consistent with the application that uses the data, by involving the application at the time of backup - using the Volume Shadow Copy Service (VSS).</ol> VSS ensures that data is written correctly to the storage. Most Microsoft workloads have VSS writers that do workload-specific actions related to data consistency. For example, Microsoft SQL Server has a VSS writer that ensures the writes to the transaction log file and the database are done correctly.<br><br> For Azure virtual machine backup, getting an application consistent recovery point means that the backup extension was able to invoke the VSS workflow and finish *correctly* before the snapshot of the virtual machine was taken. Naturally, this means that the VSS writers of all the applications in the Azure virtual machine have been invoked as well.<br><br>Learn the [basics of VSS](http://blogs.technet.com/b/josebda/archive/2007/10/10/the-basics-of-the-volume-shadow-copy-service-vss.aspx) and dive deep into the details of [how it works](https://technet.microsoft.com/library/cc785914%28v=ws.10%29.aspx). |
| File system consistency | Yes - for Windows-based machines | There are two scenarios where the recovery point can be file-system consistent:<ul><li>Backup of Linux virtual machines in Azure, because Linux does not have an equivalent platform to VSS.<li>VSS failure during backup for Windows virtual machines in Azure.</li></ul> In both of these cases, the best that can be done is to ensure that: <ol><li> The VM *boots up*. <li>There is *no corruption*. <li>There is no *data loss*.</ol> Applications need to implement their own "fix-up" mechanism on the restored data.|
| Crash consistency | No | This situation is equivalent to a machine experiencing a "crash" (through either a soft or hard reset). There is no guarantee around the consistency of the data on the storage medium. Only data that already exists on the disk at the time of backup is what gets captured and backed up. <ol><li>Although there are no guarantees, in most cases the operating system will boot.<li>This is typically followed by a disk-checking procedure like chkdsk to fix any corruption errors.<li> Any in-memory data or writes that have not been completely flushed to the disk will be lost.<li> The application typically follows with its own verification mechanism in case data rollback needs to be done. </ol>For Azure virtual machine backup, getting a crash consistent recovery point means that Azure Backup gives no guarantees around the consistency of the data on the storage - either from the perspective of the operating system or from the perspective of the application. This typically happens when the Azure virtual machine is shut down at the time of backup.<br><br>As an example, if the transaction log has entries that are not present in the database, the database software does a rollback till the data is consistent. When dealing with data spread across multiple virtual disks (like spanned volumes), a crash-consistent recovery point provides no guarantees for the correctness of the data.|

## Next steps
To learn more about getting started with Azure Backup, see:

- [Restore virtual machines](backup-azure-restore-vms.md)
- [Manage virtual machines](backup-azure-manage-vms)
 
