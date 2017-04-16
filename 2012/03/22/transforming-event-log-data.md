title: Transforming Event Log Data
link: http://sev17.com/2012/03/22/transforming-event-log-data/
author: Chad Miller
description: 
post_id: 10882
created: 2012/03/22 20:49:28
created_gmt: 2012/03/23 00:49:28
comment_status: open
post_name: transforming-event-log-data
status: publish
post_type: post

# Transforming Event Log Data

Several months ago I described a solution for [Delegated SQL Server Administration with Powershell](/2011/11/delegated-sql-server-administration-with-powershell/). In the solution, the SqlProxy module audits all security administration activity to a custom Windows Event log. In this blog post, I'll described a process to transform and incrementally load the audit data into a SQL Server table for reporting purposes. 

## Writing to the Event Log

First a quick review of how  SqlProxy module writes messages to the event log. This is important because as we'll see in a moment, how the message is constructed helps in extracting Event log data. In the SqlProxy module I use a standard template in each function for logging messages to the Event log: 
    
    
    $PSUserName = $PSSenderInfo.UserInfo.Identity.Name
    $logmessage =  "PSUserName=$PSUserName" + $($psBoundParameters.GetEnumerator() | %{"`n$($_.Key)=$($_.Value)"})
    write-sqlproxylog -eventID $eventID."$($myinvocation.mycommand.name)" -message $logmessage

The message is constructed using several built-in variables written as key/value pairs. 

  1. The $PSSenderInfo variable is available inside of remote session and returns information about the user who started the PSSession. Since I'm using runas credentials I'll grab the name of the person who is connected.
  2. $psBoundParameters contains a hashtable of the parameters and their values for the current function.
  3. This code may look a little odd, $eventID."$($myinvocation.mycommand.name)". I created hashtable called $eventid in the SqlProxy module to translate a function name from a an EventId. Since $myinvocation.mycommand.name returns the function name I'll use this as the hashtable key as shown below:
    
    
    $EventID = @{
    "Add-SqlDatabaseRoleMember"=0
    "Add-SqlLogin"=1
    "Add-SqlServerRoleMember"=2
    "Add-SqlUser"=3
    "Remove-SqlDatabaseRoleMember"=4
    "Remove-SqlLogin"=5
    "Remove-SqlServerRoleMember"=6
    "Remove-SqlUser"=7
    "Rename-SqlLogin"=8
    "Set-SqlLogin"=9
    "Set-SqlLoginDefaultDatabase"=10
    }

The write-sqlproxlog function is just a wrapper around write-eventlog as follows:  
    
    
    function Write-SqlProxyLog
    {
        param(
        [Parameter(Position=0, Mandatory=$true)] $EventID,
        [Parameter(Position=1, Mandatory=$true)] $Message,
        [Parameter(Position=2, Mandatory=$false)] $EntryType='SuccessAudit'
        )
    
        write-eventlog -logname SqlProxy -source SqlProxy -eventID $eventID -message $message -EntryType $EntryType
    
    } #Write-SqlProxyLog

A typical event log entry will look like this: ![SqlProxEvtLog](http://images.sev17.com/SqlProxEvtLog_thumb.jpg)

## Extracting Data from the Eventlog

In order to load the Eventlog data into a SQL Server table I created a module called SqlTools which is collection of functions I use frequently for querying and loading data. I've posted the module [here](https://skydrive.live.com/redir.aspx?cid=ea42395138308430&resid=EA42395138308430!1014&parid=EA42395138308430!194). The initial load script as shown below makes use of SqlTools module: 
    
    
    import-module SqlTools
    
    $ComputerName = 'Z001'
    $ServerInstance = 'Z002sql1'
    $Database = 'SqlProxy'
    
    $dt = Get-EventLog -LogName SqlProxy -ComputerName $ComputerName -EntryType 'SuccessAudit' | % { $ht = ($_.Message -replace "\","/") |
        ConvertFrom-StringData; $xml = new-object psobject -Property $ht | ConvertTo-Xml -NoTypeInformation -As String;
        new-object psobject -Property @{'Index' = $_.Index; 'TimeGenerated'=$_.TimeGenerated;
        'EventId'=$_.EventId; 'MessageXml'=$xml} } | Out-DataTable
    
    Add-SqlTable -ServerInstance $ServerInstance -Database $Database -TableName 'SqlProxyLog' -DataTable $dt -AsScript | clip

At this point I'll paste the T-SQL script into SSMS, modify column data types, null/not null, add primary key and finally create the table:  
    
    
    CREATE TABLE [dbo].[SqlProxyLog](
    	[EventId] [int] NOT NULL,
    	[TimeGenerated] [datetime] NOT NULL,
    	[MessageXml] [xml] NOT NULL,
    	[Index] [int] NOT NULL,
     CONSTRAINT [PK_SqlProxyLog] PRIMARY KEY CLUSTERED
    (
    	[TimeGenerated] ASC,
    	[Index] ASC
    )
    )

Then I'll return to Powershell to execute the write-datatable function: 
    
    
    Write-DataTable -ServerInstance $ServerInstance -Database $Database -TableName 'SqlProxyLog' -Data $dt

In order to incrementally load only new events, I'll modify the get-sqlproxylog.ps1 to first grab the max timegenerated or 1900-01-01 if its null and then use value for the -After param of the Get-Eventlog cmdlet: 
    
    
    import-module SqlTools
    
    $ComputerName = 'Z001'
    $ServerInstance = 'Z002sql1'
    $Database = 'SqlProxy'
    
    $query = "SELECT ISNULL(MAX(TimeGenerated),'1900-01-01') AS TimeGenerated FROM dbo.SqlProxyLog"
    $maxDtm = invoke-sqlcmd2 -ServerInstance $ServerInstance -Database $Database -Query $query | select -ExpandProperty TimeGenerated
    
    $dt = Get-EventLog -LogName SqlProxy -ComputerName $ComputerName -EntryType 'SuccessAudit' -After $maxDtm | % { $ht = ($_.Message -replace "\","/") |
        ConvertFrom-StringData; $xml = new-object psobject -Property $ht | ConvertTo-Xml -NoTypeInformation -As String;
        new-object psobject -Property @{'Index' = $_.Index; 'TimeGenerated'=$_.TimeGenerated;
        'EventId'=$_.EventId; 'MessageXml'=$xml} } | Out-DataTable
    
    if ($dt) {
        Write-DataTable -ServerInstance $ServerInstance -Database $Database -TableName 'SqlProxyLog' -Data $dt
    }

Some interesting points about this script: 

  * Since the message data is stored as key/value pairs, the built-in ConvertFrom-StringData cmdlet is used to create the hashtable $ht
  * The hashtable is then used to create a psobject
  * The psobject is converted into XML using ConverTo-Xml.
One minor issue I ran into is with unenclosed backslashes. If backslashes are enclosed in quotes its fine, if not it causes an error with ConvertFrom-StringData, so I replace them with forward slashes. 

## Data Is Loaded, Now What?

After I've loaded the data I'll use XQuery to shred the message XML column into a relational data set. I created a function and view in SQL Server for this purpose: 
    
    
    CREATE FUNCTION [dbo].[ufn_GetEventMessage] (@MessageXml XML)
    RETURNS @Message TABLE
    (
    	 ChangeOrder VARCHAR(128) NULL
    	,dbname  VARCHAR(128) NULL
    	,PSUserName   VARCHAR(255) NULL
    	,name   VARCHAR(128) NULL
    	,rolename   VARCHAR(128) NULL
    	,[Description]   VARCHAR(128) NULL
    	,sqlserver   VARCHAR(128) NULL
    	,loginame   VARCHAR(128) NULL
    	,[login]   VARCHAR(128) NULL
    	,DefaultDatabase VARCHAR(128) NULL
    )
    AS
    BEGIN
    	INSERT @Message (ChangeOrder,dbname,PSUserName,name,rolename,Description,sqlserver,loginame,[login],DefaultDatabase)
    	SELECT
    	 Objects.Object.query('Property[@Name="ChangeOrder"]').value('.', 'varchar(128)') AS ChangeOrder
    	,Objects.Object.query('Property[@Name="dbname"]').value('.', 'varchar(128)') AS dbname
    	,Objects.Object.query('Property[@Name="PSUserName"]').value('.', 'varchar(128)') AS PSUserName
    	,Objects.Object.query('Property[@Name="name"]').value('.', 'varchar(128)') AS name
    	,Objects.Object.query('Property[@Name="rolename"]').value('.', 'varchar(128)') AS rolename
    	,Objects.Object.query('Property[@Name="Description"]').value('.', 'varchar(128)') AS [Description]
    	,Objects.Object.query('Property[@Name="sqlserver"]').value('.', 'varchar(128)') AS sqlserver
    	,Objects.Object.query('Property[@Name="loginame"]').value('.', 'varchar(128)') AS loginame
    	,Objects.Object.query('Property[@Name="login"]').value('.', 'varchar(128)') AS [login]
    	,Objects.Object.query('Property[@Name="DefaultDatabase"]').value('.', 'varchar(128)') AS DefaultDatabase
    	FROM @MessageXml.nodes('/Objects/Object') AS Objects(Object)
    RETURN
    END;
    GO
    
    CREATE VIEW [dbo].[vw_SqlProxyLog]
    AS
    SELECT l.*, m.*

## Comments

**[Paul](#282 "2012-03-28 04:20:09"):** It would be worthwhile here to comment on the consideration that there is an event trigger available to do all of this and more in Windows 2008. It just seems that everyone is trying to find a use for the OTT Powershell API :)

**[Chad Miller](#283 "2012-03-28 08:17:45"):** Sure there are Event Log triggers in Windows 2008, but it that doesn't handle writing Event Log data to a SQL Server table which is the whole point of the blog post. Nor does it handle parsing out a specially crafted message into something more meaningful.

