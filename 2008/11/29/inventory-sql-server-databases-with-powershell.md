title: Inventory SQL Server Databases with PowerShell
link: http://sev17.com/2008/11/29/inventory-sql-server-databases-with-powershell/
author: Chad Miller
description: 
post_id: 9927
created: 2008/11/29 17:46:00
created_gmt: 2008/11/29 21:46:00
comment_status: open
post_name: inventory-sql-server-databases-with-powershell
status: publish
post_type: post

# Inventory SQL Server Databases with PowerShell

A colleague of mine asked if I could provide an updated list of SQL Server instances and databases on an ongoing basis. Although we have several applications that are capable of doing this, they are either unreliable or haven't been fully adopted in our environment. So this seems like a perfect fit for a Powershell script. We use Microsoft SCCM/SMS so we have a list of SQL Server instances dynamically updated via SCCM. Since SCCM uses SQL Server as a backend data store I started by creating a view that lists all SQL Servers instances.:
    
    
    CREATE VIEW sms_sql_instance_vw
    AS
    SELECT
    DISTINCT ISNULL(virtual_name,SMS_R_System.Name0)
    + ISNULL(‘’ + instance_name,REPLACE(REPLACE(SMS_G_System_SERVICE.Name0,’MSSQLSERVER’,”),’MSSQL$’,'’))
    AS instance_name
    FROM SMS.dbo.System_DISC AS SMS_R_System
    LEFT JOIN DBAUtility.dbo.cluster_node_virtual_assn
    ON SMS_R_System.Name0 = node_name
    JOIN SMS.dbo.Services_DATA AS SMS_G_System_SERVICE
    ON SMS_G_System_SERVICE.MachineID = SMS_R_System.ItemKey
    JOIN SMS.dbo.System_DATA AS SMS_G_System_SYSTEM
    ON SMS_G_System_SYSTEM.MachineID = SMS_R_System.ItemKey
    WHERE SMS_G_System_SERVICE.Name0 LIKE ‘MSSQL%’
    AND SMS_G_System_SERVICE.Name0 NOT LIKE ‘MSSQLServerADHelper%’
    AND SMS_G_System_SERVICE.Name0 NOT LIKE ‘MSSQLServerOLAP%’
    AND SMS_G_System_SERVICE.Name0 NOT LIKE ‘MSSQLFDLauncher%’
    AND SMS_G_System_SERVICE.StartMode0 <> ‘Disabled’
    AND SMS_G_System_SERVICE.StartName0 = ‘’
    

In order to account for clustered SQL Servers which SCCM and SCOM are blissfully unaware of I use a table with all of my cluster information called cluster_node_virtual_assn. I will blog about a Powershell script I use to dynamically load all cluster management, nodes and virtual information into a series of SQL tables next week. In addition to restrict the view to only managed SQL Servers and not rogue SQL Servers I look for SQL Servers run by a particular service account. 

Next  create a SQL table to store the information:
    
    
    CREATE TABLE [dbo].[data_base](
    [instance_name] [varchar](50) NULL,
    [data_base_name] [varchar](100) NULL)
    GO
    

And now for the PowerShell script:
    
    
    $scriptRoot = Split-Path (Resolve-Path $myInvocation.MyCommand.Path)
    . $scriptRootLibrarySmo.ps1
    Set-Alias Test-SqlConn $scriptRootTest-SqlConn.ps1
    $destServer = ‘Z002SQLEXPRESS’
    $destDb = ‘DBAUtility’
    $destTbl = ‘data_base’
    $dt = Get-SqlData ‘SMSDBServer’ ‘DBAUtility’ “SELECT * FROM dbo.sms_sql_instance_vw” | foreach {$_.instance_name } | Test-SqlConn | foreach {Get-SysDatabases $_ }
    $connectionString = “Data Source=$destServer;Integrated Security=true;Initial Catalog=$destdb;”
    $bulkCopy = new-object (“Data.SqlClient.SqlBulkCopy”) $connectionString
    $bulkCopy.DestinationTableName = “$destTbl”
    $bulkCopy.WriteToServer($dt)
    

This script uses functions from [SQL Server Powershell Extensions](http://www.codeplex.com/sqlpsx). The script collects data from all SQL Servers into a DataTable called $dt. The source list of SQL instances is obtained from the view defined in the SCCM database. Each instance name is passed to Test-SqlConn to verify connectivity. And finally the instance name is passed to Get-SysDatabases to obtain a list of instance name/databases.  The data is then loaded into the table data_base using .NET SqlBulkCopy class.

As a finishing touch I saved the script as Write-DatabaseInfo.ps1 and setup a SQL job to load the information on daily basis from a SQL 2005 server with Powershell installed. The job consists of two job steps:

  1. T-SQL step -- Delete data_base
  2. CmdExec step -- C:WINDOWSsystem32WindowsPowerShellv1.0powershell.EXE -command "C:WINDOWSScriptWrite-DatabaseInfo.ps1"