title: T-SQL Tuesday #15 Automation and the SSIS Dumper
link: http://sev17.com/2011/02/08/t-sql-tuesday-15-automation-and-the-ssis-dumper/
author: Chad Miller
description: 
post_id: 10598
created: 2011/02/08 08:13:55
created_gmt: 2011/02/08 13:13:55
comment_status: open
post_name: t-sql-tuesday-15-automation-and-the-ssis-dumper
status: publish
post_type: post

# T-SQL Tuesday #15 Automation and the SSIS Dumper

This post is my contribution to [T-SQL Tuesday](http://tsql2sday.com), hosted this month by Pat Wright ([blog](http://sqlasylum.wordpress.com/) | [twitter](http://twitter.com/SQLAsylum)). 

![tsql2sday](http://images.sev17.com/tsql2sday.jpg)

In my previous post [Importing and Exporting SSIS Packages Using PowerShell](/2011/02/importing-and-exporting-ssis-packages-using-powershell/) I showed you how to import/export SSIS stored in MSDB to and from the file system. In this post we’ll look at exporting all SSIS packages from multiple servers.

## Getting Started

1\. I’ve included a [zip file](http://cid-ea42395138308430.office.live.com/self.aspx/Public/Blog/SSIS.zip), (be sure to unblock before extracting if you download) but you’ll need to make a few changes to the files along with inserting data into a SQL Server table as explained below…

2\. Create a SQL Server table in utility database called ssis:
    
    
    CREATE TABLE dbo.ssis(
    	serverName varchar(255) NOT NULL,
    	topLevelFolder varchar(255) NOT NULL,
    	ssisVersion char(4) NOT NULL,
    	isEnabled bit NOT NULL,
     CONSTRAINT PK_ssis PRIMARY KEY CLUSTERED 
    	( serverName ASC )
    ) 
    GO
    
    ALTER TABLE dbo.ssis ADD CONSTRAINT DF_ssis_isEnabled  DEFAULT ((1)) FOR isEnabled
    GO

3\. Insert rows into SQL Server table for each SSIS server (include instance name if named instance).
    
    
    INSERT INTO ssis VALUES('Z003R2','MSDB','2008', 1)

4\. Create a folder for the SSIS PowerShell scripts. I’ll use C:SSIS

5\. Normally you would store PowerShell modules in one of the default directories defined in $env:PSModulePath. For example on my computer the module directories are: 
    
    
    $env:PSModulePath -split ";"
    C:Usersu00DocumentsWindowsPowerShellModules
    C:Windowssystem32WindowsPowerShellv1.0Modules

However when running scripts on a server sometimes I’ll chose to keep the module file together in the same folder as the scripts. In this example I’ll place my SSIS module  SSIS.psm1, which is part of [SQLPSX](http://sqlpsx.codeplex.com/) in the C:SSIS folder created previously. To import a module stored in a non-default path you simply provide the path parameter: import-module C:SSIS.psm1

6\. Copy SSIS.psm1 to C:SSIS folder

If you work with SSIS you know the SSIS 2005 and SSIS 2008/2008 R2 versions are not compatible (yes 2005 version can be upgraded). If you’re stuck managing both SSIS 2005 and SSIS 2008, you need BIDS 2005 and 2008. When using PowerShell to manage SSIS the same rules apply only that you need to load either the 2005 or 2008 assembly. Another important caveat --within the same powershell.exe process you can not unload an assembly (this is really a .NET thing). What this means is you need fire up powershell.exe, load either SSIS 2005 or SSIS 2008 assemblies but not both. Fortunately we can control which assembly is loaded by adding some logic to the module. PowerShell module supports argument as part of the import-module command.

The following example works as long as both SSIS 2005 and SSIS 2008 assemblies are loaded on the same machine. 

**import-module  SSIS  -ArgumentList 2005** OR **import-module SSIS -ArgumentList 2008**

You can now control which assemblies are loaded (wouldn’t it be cool if SSMS could this!)

7\. Create a new script file called setupSSISFolders.ps1 with the following PowerShell code. Change the servername and databasename to the location where ssis SQL Server table is located.
    
    
    $scriptRoot = 'C:SSIS'
    import-module $scriptRootSSIS.psm1
    cd $scriptRoot
    Get-ISData -serverName Z003R2 -databaseName dbutility -query "SELECT * FROM dbo.ssis WHERE isEnabled =1" | `
    foreach { $folder= "{0}_{1}" -f $($_.serverName -replace "\","_"), $_.topLevelFolder;  if (!(test-path $folder)){mkdir $folder} } 

What this code does is iterate through the list of SSIS servers defined in our ssis SQL Server table and create the folder structure to store our SSIS packages for each server if the folder doesn’t already exist. As an example running the setupSSISFolders.ps1 script creates a directory C:SSISZ003_R2_MSDB

8\. Create script file called Backup-SSIS.ps1 with the following PowerShell code. Change the configServer and configDatabase to the location where ssis SQL Serve table is located.
    
    
    param ($ssisVersion)
    
    $Script:scriptRoot = Split-Path (Resolve-Path $myInvocation.MyCommand.Path)
    $Script:configServer = "Z003R2"
    $Script:configDatabase = "dbutility"
    
    import-module "$Script:scriptRootSSIS.psm1" -ArgumentList $ssisVersion -Force
    
    #######################
    function Copy-SSIS
    {
        param ($serverName,$topLevelFolder)
    
    $folder = "{0}_{1}" -f $($serverName -replace "\","_"), $topLevelFolder
    
    copy-isitemsqltofile -path '' -topLevelFolder $topLevelFolder -serverName $serverName -destination $("{0}{1}" -f $Script:scriptRoot,$folder) -recurse -force 2>&1 | out-file -filePath  $("{0}{1}{2}.log" -f $Script:scriptRoot,$folder,$folder)
    
    } #Copy-SSIS
    
    Get-ISData -serverName $Script:configServer -databaseName $Script:ConfigDatabase -query "SELECT * FROM dbo.ssis WHERE isEnabled = 1 AND ssisVersion = '$ssisVersion'" | % { Copy-SSIS $($_.serverName) $($_.topLevelFolder) }

The Back-SSIS.ps1 script does the actual work of recursively copying all SSIS packages and folder structure from MSDB to the file system. The script iterates through each entry defined in the ssis SQL Server table.

You should have a folder C:SSIS with the following files:

![SSISFolders](http://images.sev17.com/SSISFolders_thumb.png)


## Running Scripts

Execute the following commands to run the scripts for all 2008/2008 R2 servers:
    
    
    u00@Z003 C:SSIS>.setupSSISFolders.ps1
    
        Directory: C:SSIS
    
    Mode                LastWriteTime     Length Name
    ----                -------------     ------ ----
    d----          2/8/2011   7:17 PM            Z003_R2_MSDB
    
    
    u00@Z003 C:SSIS>.Backup-SSIS.ps1 2008
    u00@Z003 C:SSIS>
    

To run the script for all 2005 serves defined in the ssis SQL Server table use  **Backup-SSIS.ps1 2005**

## **Scheduling Scripts in SQL Agent**

To schedule the in SQL Server create a job with three Operating System (cmdExec) jobs steps as follows:
    
    
    C:WindowsSystem32WindowsPowerShellv1.0powershell.exe -NoProfile -Command C:SSISsetupSSISFolders.ps1
    C:WindowsSystem32WindowsPowerShellv1.0powershell.exe -NoProfile -Command "C:SSISBackup-SSIS.ps1 2005"
    C:WINDOWSSysWOW64windowspowershellv1.0powershell.exe -NoProfile -Command "C:SSISBackup-SSIS.ps1 2008"