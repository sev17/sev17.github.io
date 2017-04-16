title: Dynamically Discover Microsoft Cluster Server Information
link: http://sev17.com/2008/12/09/dynamically-discover-microsoft-cluster-server-information/
author: Chad Miller
description: 
post_id: 9928
created: 2008/12/09 21:39:00
created_gmt: 2008/12/10 01:39:00
comment_status: open
post_name: dynamically-discover-microsoft-cluster-server-information
status: publish
post_type: post

# Dynamically Discover Microsoft Cluster Server Information

As mentioned in my post [Inventory SQL Server Databases with PowerShell](/2008/11/inventory-sql-server-databases-with-powershell/) most server management tools including SCCM/SMS are unware of Microsoft Cluster Servers. This presents a problem as I need an up-to-date list of clusters, physical nodes, and virtuals and this should be something discovered automatically.  One of the things I've always appreciated about scripting languages like Powershell and Perl is the ability to glue together the output of various utilities to create a new tool.  So, I created several Microsoft Cluster Service (MSCS) functions that parse the output of cluster.exe and the WMI MSCluster class. With these function I can dynamically discover and load cluster information into SQL Server tables.

First create two SQL Server tables to store the cluster information:

** **

**CREATE TABLE [dbo].[cluster_node]( [cluster_name] [varchar](50) NOT NULL, [node_name] [varchar](50) NOT NULL, CONSTRAINT [PK_cluster_node] PRIMARY KEY CLUSTERED ( [cluster_name] ASC, [node_name] ASC ) ) ;**

**CREATE TABLE [dbo].[cluster_virtual]( [cluster_name] [varchar](50) NOT NULL, [instance_name] [varchar](50) NULL, [virtual_name] [varchar](50) NOT NULL, CONSTRAINT [PK_cluster_virtual] PRIMARY KEY CLUSTERED ( [cluster_name] ASC, [virtual_name] ASC ) );**

Next Download [LibraryMSCS](http://www.poshcode.org/724) and save as LibraryMSCS.ps1 and then download Out-[DataTable](http://thepowershellguy.com/blogs/posh/archive/2007/01/21/powershell-gui-scripblock-monitor-script.aspx) function from and save as LibraryDataTable.ps1

Next add Write-DatabaseTableToDatabase function to LibraryDataTable.ps1:

**function Write-DataTableToDatabase { param($destServer,$destDb,$destTbl) process { $connectionString = "Data Source=$destServer;Integrated Security=true;Initial Catalog=$destdb;" $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $connectionString $bulkCopy.DestinationTableName = "$destTbl" $bulkCopy.WriteToServer($_) }**

**}# Write-DataTableToDatabase**

And now for the Powershell script (As you probably tell I'm a big fan of creating libraries of resusable functions and then creating specific scripts from the functions. This simplies the the code considerable):

**. ./LibraryMSCS.ps1 . ./LibraryDataTable.ps1 Get-ClusterList | Get-ClusterToNode | Out-DataTable | Write-DataTableToDatabase 'Z002SqlExpress' 'dbautility 'cluster_node' Get-ClusterList | Get-ClusterToVirtual | Out-DataTable | Write-DataTableToDatabase 'Z002SqlExpress' 'dbautility 'cluster_virtual'**

****

And finally with the tables loaded I'll create a view which joins the cluster information:

**CREATE VIEW [dbo].[cluster_node_virtual_assn_vw] AS SELECT n.cluster_name, n.node_name, v.virtual_name, v.instance_name FROM cluster_node n JOIN cluster_virtual v ON n.cluster_name = v.cluster_name**

Like the [inventory database solution](/2008/11/inventory-sql-server-databases-with-powershell/) as a finishing touch I save the script as Write-ClusterInfo.ps1 and create a SQL Agent job with three jobs:

  1. T-SQL step -- DELETE cluster_node
  2. T-SQL step -- DELETE cluster_virtual
  3. CmdExec step -- C:WINDOWSsystem32WindowsPowerShellv1.0powershell.EXE -command "C:WINDOWSScriptWrite-ClusterInfo.ps1"