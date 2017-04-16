title: Making A SQLPS Module
link: http://sev17.com/2010/07/10/making-a-sqlps-module/
author: Chad Miller
description: 
post_id: 10397
created: 2010/07/10 14:31:04
created_gmt: 2010/07/10 18:31:04
comment_status: open
post_name: making-a-sqlps-module
status: publish
post_type: post

# Making A SQLPS Module

If you’re working with PowerShell and SQL Server one of things you’ll want to to do is load the SQL Server 2008 provider and cmdlets into a regular PowerShell. [Michiel Wories](http://blogs.msdn.com/b/mwories/), the creator of SMO and sqlps, provides an initialization script in his blog post [SQL Server PowerShell is Here!](http://blogs.msdn.com/b/mwories/archive/2008/06/14/sql2008_5f00_powershell.aspx) The script will load SQL Server provider, cmdlets, required assemblies and set global variables expected by the SQL Server provider. 

## Creating the sqlps module

In PowerShell version 2, modules provide an alternative approach to initialization scripts used in PowerShell V1. To turn Michiel’s initialization script into a module simply create a folder called sqlps under _Documents\WindowsPowerShell\Modules_ and save the script as sqlps.psm1 instead of Initialize-SqlpsEnvironment.ps1. You can then execute: 
    
    
    import-module sqlps

You’ll notice the following warning: 
    
    
    WARNING: Some imported command names include unapproved verbs which might make them less discoverable. Use the Verbose parameter for more detail or type Get-Verb to see the list of approved verbs.

Running get-command -Module sqlps, you’ll notice the Encode-SqlName and Decode-SqlName cmdlets and since neither is an approved verb – hence the warning. To avoid the warning message when loading your new sqlps module use 
    
    
    import-module sqlps –DisableNameChecking

## An Alternative Approach

Rather than turning the original initialization script into a module, we could create a more structured implementation making use of a PowerShell manifest file (psd1) file. Using this approach we’ll need to copy the following snapins related files/folders from _C:Program FilesMicrosoft SQL Server100ToolsBinn_ to  _DocumentsWindowsPowerShellModulessqlps_ folder. 

  * en
  * Microsoft.SqlServer.Management.PSProvider.dll
  * Microsoft.SqlServer.Management.PSSnapins.dll
  * SQLProvider.Format.ps1xml
  * SQLProvider.Types.ps1xml
Next we’ll create a sqlp.psd1 manifest which contains the instructions for processing our new module: 
    
    
    @{
    ModuleVersion="0.0.0.1"
    Description="A Wrapper for Microsoft's SQL Server PowerShell Extensions Snapins"
    Author="Chad Miller"
    Copyright="© 2010, Chad Miller, released under the Ms-PL"
    CompanyName="http://sev17.com"
    CLRVersion="2.0"
    FormatsToProcess="SQLProvider.Format.ps1xml"
    NestedModules="Microsoft.SqlServer.Management.PSSnapins.dll","Microsoft.SqlServer.Management.PSProvider.dll"
    RequiredAssemblies="Microsoft.SqlServer.Smo","Microsoft.SqlServer.Dmf","Microsoft.SqlServer.SqlWmiManagement","Microsoft.SqlServer.ConnectionInfo","Microsoft.SqlServer.SmoExtended","Microsoft.SqlServer.Management.RegisteredServers","Microsoft.SqlServer.Management.Sdk.Sfc","Microsoft.SqlServer.SqlEnum","Microsoft.SqlServer.RegSvrEnum","Microsoft.SqlServer.WmiEnum","Microsoft.SqlServer.ServiceBrokerEnum","Microsoft.SqlServer.ConnectionInfoExtended","Microsoft.SqlServer.Management.Collector","Microsoft.SqlServer.Management.CollectorEnum"
    TypesToProcess="SQLProvider.Types.ps1xml"
    ScriptsToProcess="Sqlps.ps1"
    }

Notice a sqlps.ps1 script file is referenced which  is used to set the variables needed by the provider: 
    
    
    Set-Variable -scope Global -name SqlServerMaximumChildItems -Value 0
    Set-Variable -scope Global -name SqlServerConnectionTimeout -Value 30
    Set-Variable -scope Global -name SqlServerIncludeSystemObjects -Value $false
    Set-Variable -scope Global -name SqlServerMaximumTabCompletion -Value 1000

Since the sqlps licensing terms from Microsoft allow redistribution as long as the original installer is included, I’ve included a [sqlps module zip file](http://cid-ea42395138308430.office.live.com/self.aspx/Public/SQLPS%5E_Module.zip). 

### Notes

  1. SQL Server Management Studio is not required to run sqlps or the sqlps module demonstrated in this post as long as the required assemblies are installed
  2. Like sqlps, [Microsoft SQL Server 2008 Management Objects](http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=b33d2c78-1059-4ce2-b80d-2343c099bcb4) and [Microsoft Core XML Services (MSXML) 6.0](http://go.microsoft.com/fwlink/?LinkId=3999) are required.
  3. If you’d also like to also run sqlps host without install SQL Server Management Studio download the installation from the [SQL Server 2008 or R2 Feature Pack](http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=ceb4346f-657f-4d28-83f5-aae0c5c83d52).

## Comments

**[Robin](#303 "2013-03-08 07:50:13"):** Hi Chad, The link to Microsoft SQL Server 2008 Management Objects is dead. When I google for it, the top result is http://www.microsoft.com/en-gb/download/details.aspx?id=16978 but I'm not sure if that's a suitable replacement. Infact, there is actually an SP2 version of the above: http://www.microsoft.com/en-gb/download/details.aspx?id=30440 Any ideas? Thanks, Robin

**[Chad Miller](#304 "2013-03-08 08:25:45"):** The link you provided to SQL Server 2008 R2 SP2 is the latest for SQL Server 2008 R2. This of course will change whenever new Service Packs are release. I would also--SQL Server 2012 now ships a sqlps module you can install instead of following instructions in this blog: <http://www.microsoft.com/en-us/download/details.aspx?id=35580>

**[Laura Cerniglia](#325 "2013-06-23 18:35:27"):** So then can the full path name be like this: D:\CMS\Documents\WindowsPowerShell\Modules

**[Laura Cerniglia](#321 "2013-06-21 13:45:33"):** When you write this, DocumentsWindowsPowerShellModules, what is the full path name? I only see C:\Windows\System32\WindowsPowerShell\v1.0\Modules

**[Chad Miller](#322 "2013-06-21 19:13:00"):** When I moved my blog some of the backslashes were automatically removed. Should be fixed now. If you don't have a WindowsPowershell\Modules in your documents you need to create one.

**[Chad Miller](#326 "2013-06-23 18:57:00"):** It should be the path in your $env:PSModulePath i.e. run $env:PSModulePath from Powershell.

**[Laura Cerniglia](#327 "2013-06-23 19:14:58"):** Ok. I am starting to get somewhere. When I merge Out-DataTable.ps1 and FailedBackups.ps1 together as one script, FailedBackups.ps1, and at the cmd window I run sqlps "& 'D:\CMS\Scripts\PowerShell\FailedBackups.ps1' " I can get it to work. I am hoping that you can help me with this part. But when I try to get it to work in SQL Server Agent, it still doesn't work. I was hoping to schedule it as a SQL Server Agent instead of a Windows Schedule Task. I get the following error in SQL Server Agent Message Executed as user: NT Service\SQLSERVERAGENT. dir : Cannot find path 'SQLSERVER:\SQLRegistration\Central Management Server Gr oup\CMSNameHere\' because it does not exist. At D:\CMS\Scripts\PowerShell\FailedBackups.ps1:74 char:32 + foreach ($RegisteredSQLs in dir <<<< -recurse SQLSERVER:\SQLRegistration\'Ce ntral Management Server Group'\CMSNameHere\ | where {$_.Mode -ne "d"} ) + CategoryInfo : ObjectNotFound: (SQLSERVER:\SQLR...CMSNameHere\:String) [Get-ChildItem], ItemNotFoundException + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetCh ildItemCommand. Process Exit Code 0. The step succeeded. Why when it is a sql server agent job is it not importing sqlps?

**[Chad Miller](#328 "2013-06-24 08:28:50"):** I'm assuming CMSNameHere is your real CMS server name. So, one thing you'll need to do (one-time) when working with SQLRegistration provider from SQL Agent is to run SQL Server Management Studio as the account which runs your SQL Agent service and register your CMS in SSMS for that account. The CMS registration is tied to a user account. After you've done that you can access "...SQLServerRegistration\Central Management Server..." from a SQL Agent job.

**[Laura Cerniglia](#329 "2013-06-24 10:23:24"):** I hope you don't mind answering one more question. I don't want the modules for SQLPSX under my document folder incase I win the lottery. I ran $env:PSModulePath and the Modules are under C:\Users\MyLogonIDHere\Documents\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules\;D:\Program Files (x86)\Microsoft SQL Server\110\Tools\PowerShell\Modules\ How do I change C:\Users\MyLogonIDHere\Documents\WindowsPowerShell\Modules to C:\SQLPSX\Documents\WindowsPowerShell\Modules I can do that yes?

**[Chad Miller](#330 "2013-06-24 11:08:28"):** Sure you need to modify environmental variable. See documentation: http://msdn.microsoft.com/en-us/library/windows/desktop/dd878326(v=vs.85).aspx You could just use the GUI to modify also. Under System Properties > Environmental Properties > PSModulePath.

**[Gary](#331 "2013-06-24 12:14:17"):** Unfortunately, this only works for me in Powershell 2.0 Is there any way to get 'Invoke-SqlCmd' to work with PS 3?

**[Chad Miller](#332 "2013-06-24 12:19:30"):** I haven't had issues with invoke-sqlcmd and Powershell v3. Do you have details?

**[Laura Cerniglia](#333 "2013-06-24 12:32:16"):** "I’m assuming CMSNameHere is your real CMS server name. So, one thing you’ll need to do (one-time) when working with SQLRegistration provider from SQL Agent is to run SQL Server Management Studio as the account which runs your SQL Agent service and register your CMS in SSMS for that account. The CMS registration is tied to a user account. After you’ve done that you can access “…SQLServerRegistration\Central Management Server…” from a SQL Agent job." Yes, CMSNameHere is my real CMS server name. I just didn't want to post it for confidentially reasons. So how do you run SQL Server Management Studio as NT Service\SQLSERVERAGENT (This is the account that runs my SQL Agent service)?

**[Chad Miller](#334 "2013-06-24 13:41:48"):** That could be a problem, not sure how you would start SSMS one-time as NT Server\SQLSERVERAGENT. I tend to use normal domain accounts for SQLAgent and as a one-time setup while RDP'd on the actual CMS server box do a start run as the domain-level service account of SSMS. I would think not using domain account would cause other issues like connecting to remote SQL Servers. If you can't use a domain account you could try setting up a proxy account in SQL Agent, starting up SSMS one-time as the account, and registering the CMS under that account.

**[Laura](#335 "2013-06-25 15:10:34"):** Now I am trying to create my own module to collect data from a Central Management Server. I am trying to follow the example that John Sterrett provided in his web site http://johnsterrett.com/2011/05/12/passed-my-sqluniversity-powershell-midterm/ However, I am getting the following error PS SQLSERVER:\> D:\CMS\Scripts\PowerShell\RegisteredServers.ps1 Connected Exception calling "ExecuteNonQuery" with "0" argument(s): "There are not enough fields in the Structured type. Structured types must have at least one field." At D:\CMS\Scripts\PowerShell\RegisteredServers.ps1:93 char:1 \+ $cmd.ExecuteNonQuery() | out-Null \+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ \+ CategoryInfo : NotSpecified: (:) [], MethodInvocationException \+ FullyQualifiedErrorId : ArgumentException I have gone over it and I don't see the issue. Here is how the database objects are coded USE [CMSRepository] GO /****** Object: UserDefinedTableType [dbo].[RegisteredServers] Script Date: 6/25/2013 5:27:10 PM ******/ CREATE TYPE [dbo].[RegisteredServers] AS TABLE( [ServerName] [varchar](250) NOT NULL, [ProductVersion] [varchar](250) NOT NULL, [ProductLevel] [varchar](250) NOT NULL, [Edition] [varchar](250) NOT NULL, [EngineEdition] [varchar](250) NOT NULL, [RunDate] [smalldatetime] NOT NULL, [RowError] [varchar](1000) NULL, [RowState] [varchar](1000) NULL, [Table] [varchar](1000) NULL, [ItemArray] [varchar](1000) NULL, [HasErrors] [varchar](1000) NULL ) GO /****** Object: StoredProcedure [dbo].[usp_InsertRegisteredServers] Script Date: 6/25/2013 5:27:10 PM ******/ SET ANSI_NULLS ON GO SET QUOTED_IDENTIFIER ON GO CREATE PROCEDURE [dbo].[usp_InsertRegisteredServers] @TVP RegisteredServers readonly AS BEGIN INSERT INTO dbo.RegisteredServers (ServerName, ProductVersion, ProductLevel, Edition, EngineEdition, RunDate) SELECT ServerName, ProductVersion, ProductLevel, Edition, EngineEdition, RunDate FROM @TVP END GO /****** Object: Table [dbo].[RegisteredServers] Script Date: 6/25/2013 5:27:10 PM ******/ SET ANSI_NULLS ON GO SET QUOTED_IDENTIFIER ON GO SET ANSI_PADDING ON GO CREATE TABLE [dbo].[RegisteredServers]( [ServerName] [varchar](250) NOT NULL, [ProductVersion] [varchar](250) NOT NULL, [ProductLevel] [varchar](250) NOT NULL, [Edition] [varchar](250) NOT NULL, [EngineEdition] [varchar](250) NOT NULL, [RunDate] [smalldatetime] NOT NULL, CONSTRAINT [PK_RegisteredServers] PRIMARY KEY CLUSTERED ( [ServerName] ASC )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY] ) ON [PRIMARY] GO SET ANSI_PADDING OFF GO Here is how the PowerShell Script is coded: ####################### function Out-DataTable { [CmdletBinding()] param([Parameter(Position=0, Mandatory=$true, ValueFromPipeline = $true)] [PSObject[]]$InputObject) Begin { $dt = new-object Data.datatable $First = $true } Process { foreach ($object in $InputObject) { $DR = $DT.NewRow() foreach($property in $object.PsObject.get_properties()) { if ($first) { $Col = new-object Data.DataColumn $Col.ColumnName = $property.Name.ToString() $DT.Columns.Add($Col) } if ($property.IsArray) { $DR.Item($property.Name) =$property.value | ConvertTo-XML -AS String -NoTypeInformation -Depth 1 } else { $DR.Item($property.Name) = $property.value } } $DT.Rows.Add($DR) $First = $false } } End { Write-Output @(,($dt)) } } #Out-DataTable Import-Module “sqlps” -DisableNameChecking foreach ($RegisteredSQLs in dir -recurse SQLSERVER:\SQLRegistration\'Central Management Server Group'\CMSNameHere\ | where {$_.Mode -ne "d"} ) { $dt = Invoke-sqlcmd -ServerInstance "$($RegisteredSQLs.ServerName)" -Database "tempdb" -InputFile "D:\CMS\Scripts\T-SQL\RegisteredServers.sql" | out-DataTable $dt # Write data table to database using TVP $conn = new-Object System.Data.SqlClient.SqlConnection("Server=CMSNameHere;DataBase=CMSRepository;Integrated Security=SSPI") $conn.Open() | out-null "Connected" $cmd = new-Object System.Data.SqlClient.SqlCommand("dbo.usp_InsertRegisteredServers", $conn) $cmd.CommandType = [System.Data.CommandType]'StoredProcedure' #SQLParameter $spParam = new-Object System.Data.SqlClient.SqlParameter $spParam.ParameterName = "@TVP" $spParam.Value = $dt $spParam.SqlDbType = "Structured" #SqlDbType.Structured $spParam.TypeName = "RegisteredServers" $cmd.Parameters.Add($spParam) | out-Null $cmd.ExecuteNonQuery() | out-Null $conn.Close() | out-Null } I have masked my server and Central Management Server names with CMSNameHere. Can anyone tell me what this error means and how to fix it? Have I not change something that I should have or did I change something that I should not have? I got John Sterrett's code to work but I don't understand what I have done wrong in my code that it doesn't work. I forgot. Here is my RegisteredServers.sql script: BEGIN DECLARE @date SMALLDATETIME SET @date = GETDATE() DECLARE @RegisteredServers TABLE (ServerName VARCHAR(255), ProductVersion VARCHAR(255), ProductLevel VARCHAR(255), Edition VARCHAR(255), EngineEdition VARCHAR(255), RunDate SMALLDATETIME) INSERT INTO @RegisteredServers (ServerName, ProductVersion, ProductLevel, Edition, EngineEdition, RunDate) SELECT CONVERT(VARCHAR(250), SERVERPROPERTY('ServerName')) AS ServerName, CONVERT(VARCHAR(250), SERVERPROPERTY('ProductVersion')) AS ProductVersion, CONVERT(VARCHAR(250), SERVERPROPERTY('ProductLevel')) AS ProductLevel, CONVERT(VARCHAR(250), SERVERPROPERTY('Edition')) AS Edition, CONVERT(VARCHAR(250), SERVERPROPERTY('EngineEdition')) AS EngineEdition, @date AS RunDate END

**[Chad Miller](#336 "2013-06-25 19:08:47"):** Not sure why you're converting your call to invoke-sqlcmd to out-datatable -- unnecessary. The TVP approach is also overly complex. Instead use a simple BulkCopy as follows: ` $dt = Invoke-sqlcmd -ServerInstance “$($RegisteredSQLs.ServerName)” -Database “tempdb” -InputFile “D:\CMS\Scripts\T-SQL\RegisteredServers.sql” $conn.Open() | out-null $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $conn $bulkCopy.DestinationTableName = 'RegisteredServers' $bulkCopy.WriteToServer($dt) $conn.Close() `

**[Laura](#337 "2013-06-26 08:49:26"):** I am running this on a Central Management Server. Will this work? It doesn't say where I am inserting the data into what instance and what table in the PowerShell script. Here is my PowerShell code currently: Import-Module “sqlps” -DisableNameChecking foreach ($RegisteredSQLs in dir -recurse SQLSERVER:\SQLRegistration\'Central Management Server Group'\CMSNameHere\ | where {$_.Mode -ne "d"} ) { $dt = Invoke-sqlcmd -ServerInstance "$($RegisteredSQLs.ServerName)" -Database "tempdb" -InputFile "D:\CMS\Scripts\T-SQL\RegisteredServers.sql" $conn.Open() $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $conn $bulkCopy.DestinationTableName = 'RegisteredServers' $bulkCopy.WriteToServer($dt) $conn.Close() } Here is my SQL Server Code: BEGIN DECLARE @date SMALLDATETIME SET @date = GETDATE() \--DECLARE @RegisteredServers TABLE (ServerName VARCHAR(255), ProductVersion VARCHAR(255), ProductLevel VARCHAR(255), Edition VARCHAR(255), EngineEdition VARCHAR(255), RunDate SMALLDATETIME) INSERT INTO InstanceName.CMSRepository.dbo.RegisteredServers (ServerName, ProductVersion, ProductLevel, Edition, EngineEdition, RunDate) SELECT CONVERT(VARCHAR(250), SERVERPROPERTY('ServerName')) AS ServerName, CONVERT(VARCHAR(250), SERVERPROPERTY('ProductVersion')) AS ProductVersion, CONVERT(VARCHAR(250), SERVERPROPERTY('ProductLevel')) AS ProductLevel, CONVERT(VARCHAR(250), SERVERPROPERTY('Edition')) AS Edition, CONVERT(VARCHAR(250), SERVERPROPERTY('EngineEdition')) AS EngineEdition, @date AS RunDate END When I try to run it, I get the following error messages: Invoke-sqlcmd : Invalid object name 'CMSRepository.dbo.RegisteredServers'. At line:8 char:7 \+ $dt = Invoke-sqlcmd -ServerInstance "$($RegisteredSQLs.ServerName)" -Database "t ... \+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ \+ CategoryInfo : InvalidOperation: (:) [Invoke-Sqlcmd], SqlPowerShellSqlExecutionException \+ FullyQualifiedErrorId : SqlError,Microsoft.SqlServer.Management.PowerShell.GetScriptCommand You cannot call a method on a null-valued expression. At line:9 char:1 \+ $conn.Open() \+ ~~~~~~~~~~~~ \+ CategoryInfo : InvalidOperation: (:) [], RuntimeException \+ FullyQualifiedErrorId : InvokeMethodOnNull new-object : Constructor not found. Cannot find an appropriate constructor for type Data.SqlClient.SqlBulkCopy. At line:10 char:13 \+ $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $conn \+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ \+ CategoryInfo : ObjectNotFound: (:) [New-Object], PSArgumentException \+ FullyQualifiedErrorId : CannotFindAppropriateCtor,Microsoft.PowerShell.Commands.NewObjectCommand Property 'DestinationTableName' cannot be found on this object; make sure it exists and is settable. At line:11 char:1 \+ $bulkCopy.DestinationTableName = 'RegisteredServers' \+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ \+ CategoryInfo : InvalidOperation: (:) [], RuntimeException \+ FullyQualifiedErrorId : PropertyNotFound You cannot call a method on a null-valued expression. At line:12 char:1 \+ $bulkCopy.WriteToServer($dt) \+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ \+ CategoryInfo : InvalidOperation: (:) [], RuntimeException \+ FullyQualifiedErrorId : InvokeMethodOnNull You cannot call a method on a null-valued expression. At line:13 char:1 \+ $conn.Close() \+ ~~~~~~~~~~~~~ \+ CategoryInfo : InvalidOperation: (:) [], RuntimeException \+ FullyQualifiedErrorId : InvokeMethodOnNull Invoke-sqlcmd : Invalid object name 'CMSRepository.dbo.RegisteredServers'. At line:8 char:7 \+ $dt = Invoke-sqlcmd -ServerInstance "$($RegisteredSQLs.ServerName)" -Database "t ... \+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ \+ CategoryInfo : InvalidOperation: (:) [Invoke-Sqlcmd], SqlPowerShellSqlExecutionException \+ FullyQualifiedErrorId : SqlError,Microsoft.SqlServer.Management.PowerShell.GetScriptCommand You cannot call a method on a null-valued expression. At line:9 char:1 \+ $conn.Open() \+ ~~~~~~~~~~~~ \+ CategoryInfo : InvalidOperation: (:) [], RuntimeException \+ FullyQualifiedErrorId : InvokeMethodOnNull new-object : Constructor not found. Cannot find an appropriate constructor for type Data.SqlClient.SqlBulkCopy. At line:10 char:13 \+ $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $conn \+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ \+ CategoryInfo : ObjectNotFound: (:) [New-Object], PSArgumentException \+ FullyQualifiedErrorId : CannotFindAppropriateCtor,Microsoft.PowerShell.Commands.NewObjectCommand Property 'DestinationTableName' cannot be found on this object; make sure it exists and is settable. At line:11 char:1 \+ $bulkCopy.DestinationTableName = 'RegisteredServers' \+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ \+ CategoryInfo : InvalidOperation: (:) [], RuntimeException \+ FullyQualifiedErrorId : PropertyNotFound You cannot call a method on a null-valued expression. At line:12 char:1 \+ $bulkCopy.WriteToServer($dt) \+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ \+ CategoryInfo : InvalidOperation: (:) [], RuntimeException \+ FullyQualifiedErrorId : InvokeMethodOnNull You cannot call a method on a null-valued expression. At line:13 char:1 \+ $conn.Close() \+ ~~~~~~~~~~~~~ \+ CategoryInfo : InvalidOperation: (:) [], RuntimeException \+ FullyQualifiedErrorId : InvokeMethodOnNull

**[Chad Miller](#338 "2013-06-26 09:08:08"):** Looks like you removed the portion of code which creates you connection object `$conn = new-Object System.Data.SqlClient.SqlConnection(“Server=CMSNameHere;DataBase=CMSRepository;Integrated Security=SSPI”) $conn.Open() | out-null $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $conn $bulkCopy.DestinationTableName = 'RegisteredServers' $bulkCopy.WriteToServer($dt) $conn.Close() `

