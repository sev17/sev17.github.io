title: Managing AlwaysOn with Powershell
link: http://sev17.com/2011/09/05/managing-alwayson-with-powershell/
author: Chad Miller
description: 
post_id: 10742
created: 2011/09/05 18:46:32
created_gmt: 2011/09/05 22:46:32
comment_status: open
post_name: managing-alwayson-with-powershell
status: publish
post_type: post

# Managing AlwaysOn with Powershell

Although you can use SQL Server Management Studio or T-SQL to manage AlwaysOn, SQL Server Denali CTP 3 includes 25 cmdlet which together provide complete coverage for creating, confiiguring and administering the AlwaysOn database feature. In this post we’ll look at using Powershell to perform various management tasks for AlwayOn.

_Note: This blog post describes features in SQL Server Denali CTP 3 which may change on final product release._

## Getting Started

You’ll need  a simple Windows 2008 R2 cluster with two standalone installs of SQL Server. I say simple because you don’t have to worry about shared storage, quorum disks and shared MSTDTC installations like you would in a traditional SQL Server installation on a Windows Server Failover Cluster. All you need are two servers running Windows Server 2008 R2 Enterprise Edition. For test purposes I’ve setup a two-node Windows Server Failover Cluster as follows:

1\. Configure a Private virtual machine network for intra-cluster communication. Note this step is optional and not really necessary for a bare minimum cluster, but I setup it up anyways to mimic close to what I’ll have in production. This network uses a separate IP subnet than the Internal only network I had already setup in Hyper-V. 

2\. Setup the private only network  which allows communication between virtual machines only.

  1. Right-click virtual machine 
  2. Select Virtual Network Manager > Select Private > Add 

![VirtualNetworkManager](http://images.sev17.com/VirtualNetworkManager_thumb.png)

3\. Add the private network to each of the virtual machines. 

    1. Shutdown each machine 
    2. Select machine in Hyper-V Manager 
    3. Select Settings 
    4. Select Add Hardware and choose Network Adapter and click Add 
    5. Select the private network you created from the Network drop down list and click OK 

![AddNetwork](http://images.sev17.com/AddNetwork_thumb.png)

4\. On the virtual machines assign IP addresses under Network and Sharing Center. Here’s a table of my setup:

**Machine**
**Internal Network**
**Private Network**

Node1
192.168.1.71
192.1.1.2

Node2
192.168.1.72
192.1.13

DC1
192.168.1.50
N/A

Clusterxm*
192.168.1.70
N/A

Availability Group* Listener
192.168.1.73
N/A

DC1= Domain Controller

Cluster1 = cluster management IP (assigned during cluster configuration)

Availability Group Listener (assigned during AlwaysOn  Availability Group Listener configuration)

*Don’t worry about these for now.

5\. Since we’re using a two-node cluster without a quorum disk it is suggested to use a Node and File Share Majority so I’ll setup network share which is read/write accessible by the Cluster Service account. For my testing purposes I created share on my DC1 machine called \DC1Share1 located on DC1 C:Share1 folder.

## Setting Up Windows Failover Clustering

1.  Add the Failover Cluster Manage feature to both modes by running the following  PowerShell commands
    
    
    import-module ServerManager
    Add-WindowsFeature -Name Failover-Clustering

2\. Create the cluster by running the following PowerShell commands on one node:
    
    
    import-module FailoverClusters
    new-cluster clusterxm -Node node1,node2 -StaticAddress 192.168.1.70 -NoStorage

3\. Set the quorum mode to Node and File Share Majority by running the following command on one node:
    
    
     Set-ClusterQuorum -NodeAndFileShareMajority "\\DC1\share1"

## Install SQL Server on Both Nodes

Install SQL Server and this important – **As a standalone instance. **Sorry no Powershell commands here just run through the installation screens. Be sure to set the SQL Server service account to a domain account (I had issues when using Local System). 

![DenaliInstall](http://images.sev17.com/DenaliInstall_thumb.png)

## Database Prerequisites

You need to have a database which is not already part of an AlwaysOn Availability Group in FULL recovery mode and has been backed up. As a test I’ll just use the [old school pubs sample database](http://www.microsoft.com/download/en/details.aspx?displaylang=en&id=23654). Run the instpubs.sql file and create a backup using Powershell.

Start SQL Server Management Studio and select “Start PowerShell” from Object Explorer. Run the following command to backup the database to the default backup directory:
    
    
    PS SQLSERVER:SQL\NODE1\DEFAULT\Databases\pubs> Backup-SqlDatabase -Database pubs

You’ll need to create a share accessible by both nodes for storing the SQL Server database and transaction log initialization backups. For my example I’ll create a folder called sqlrec under Node1’s C drive C:\sqlrec and share named sqlrec \\\node1\sqlrec

## AlwaysOn Powershell Documentation

The CTP3 version of Books Online contains some documentation and scripts for configuring AlwaysOn however as to be expected with pre-release software some topics are not covered and there are documentation errors in other sections. As I’ve encountered documentation errors I  submitted Connect Items  (see Connect Items below for details).  Relevant helps topics included in CTP3 are listed below:

[Specify the Endpoint URL When Adding or Modifying an Availability Replica](http://msdn.microsoft.com/en-us/library/ff878441\(v=sql.110\).aspx)

[Enable and Disable AlwaysOn](http://msdn.microsoft.com/en-us/library/ff878259\(v=SQL.110\).aspx)

[Create and Configure an Availability Group](http://msdn.microsoft.com/en-us/library/gg492181\(v=SQL.110\).aspx)