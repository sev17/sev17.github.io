title: The T-SQL Hammer
link: http://sev17.com/2010/02/16/the-t-sql-hammer/
author: Chad Miller
description: 
post_id: 9996
created: 2010/02/16 16:22:00
created_gmt: 2010/02/16 20:22:00
comment_status: open
post_name: the-t-sql-hammer
status: publish
post_type: post

# The T-SQL Hammer

The over-reliance on a familiar tool is best described with the quote, “if all you have is a hammer, everything looks like nail” and for database professionals this means using or sometimes misusing T-SQL. Whenever database administrators are presented with a scripting problem they instinctively reach for good-old-familiar T-SQL.  And why not? In many cases T-SQL provides an ideal solution to SQL Server administration scripting problems, however there are certain scenarios where another tool, Powershell provides a  more elegant solution. One such problem is scripting of database objects, T-SQL simply does not handle the complexities of objects script creation very well. In an attempt to use a familiar tool many people have written T-SQL scripts that trudge through system tables to produce an object creation script. The problem with such scripts is that they tend to be ugly, lengthy pieces of code that easily break. Experienced DBAs know querying system table directly is poor practice, yet they do it anyways in order to use their hammer T-SQL tool. There’s a better way, [SQL Server Management Objects](http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.aspx) (SMO) has taken care of many of these issues for us and all we need to do is write a little Powershell code. As an example let’s look at an object script problem from [Brent Ozar](http://www.brentozar.com/)’s post, “[How to REALLY Compress Your SQL Server Backups](http://www.brentozar.com/archive/2010/02/how-to-really-compress-your-sql-server-backups/).” He describes a database size compression solution that does the following: 

  1. Script out all of the non-clustered indexes in the database
  2. Save those definitions to a table (or a stored procedure)
  3. Create a stored proc that will loop through those index definitions and recreate them later
  4. Drop the indexes
Although Brent does not provide us with a script because in his words it’s a little duct-tape-y, he does link a to a couple of T-SQL based solutions: [Scripts to Drop and ReCreate Indexes in SQL Server 2005](http://samsudeenb.blogspot.com/2007/11/scripts-to-drop-and-recreate-indexes-in.html) [SQL Server 2005: Script all Indexes](http://www.sqlservercentral.com/scripts/Miscellaneous/31893/) The authors of the T-SQL scripts use a classic SQL administration scripting technique of building code by interrogating various system tables. The scripts are not without issue. A quick glance at the [comment section of the second script](http://www.sqlservercentral.com/Forums/Topic401784-562-1.aspx) reveals a dozen corrections were made to the original post after various users ran into things not accounted for. The abundance of corrections further illustrates T-SQL does not provide a good method to script out database objects. Now, let’s look at a more complete Powershell and SMO based solution. 

#### In order to store the index create and drop statements, we’ll need a table. I’m going to create the table in the same database:
    
    
    CREATE TABLE [dbo].[IndexDdl](
    [DdlType] [varchar](10) NOT NULL,
    [IndexScript] [varchar](4000) NOT NULL
    )
    

SQL Server 2008 provides a Powershell interface accessible from SQL Server Management Studio. Start Powershell from the Tables folder in SQL Server Management Studio: ![startps0](http://images.sev17.com/startps0_thumb.jpg) ![sqlpsidx](http://images.sev17.com/sqlpsidx_thumb.jpg)

#### Script out the non-clustered index drop statements and save them to the IndexDdl table:
    
    
    $scriptingOptions = New-Object Microsoft.SqlServer.Management.Smo.ScriptingOptions
    $scriptingOptions.DdlHeaderOnly =  $true
    $scriptingOptions.ScriptDrops = $true
    dir | foreach { $_.indexes } | where { $_.IsClustered -eq $false -and $_.IsXmlIndex -eq $false}|`
    foreach {$_.Script($scriptingOptions)} | foreach {invoke-sqlcmd -Query "INSERT dbo.IndexDdl VALUES('Drop','$_')"}
    

Using the SQL Server Powershell provider, we are able to get all indexes that are non-clustered and not an XML index and then call the SMO _script _method. Generating drop statements require a little extra setup in that we first need to create a _scripting options_ object and set the _DdlHeaderOnly_ and _ScriptDrops_ properties to true. The resulting script is then inserted into the _IndexDdl_ table. Note: Because the current connection is used the call to **invoke-sqlcmd** does not specify a server or database name. 

#### Script out the create index statements:
    
    
    dir |foreach {$_.indexes} | where {$_.IsClustered -eq $false -and $_.IsXmlIndex -eq $false} | `
    foreach {$_.Script() | foreach{invoke-sqlcmd -Query "INSERT dbo.IndexDdl VALUES('Create','$_')"
    

The create statement can use the default behavior of the _script_ method, no extra setup required. 

#### To execute the drop statements:
    
    
    $drops = Invoke-sqlcmd -Query "SELECT IndexScript FROM dbo.IndexDdl WHERE DdlType = 'Drop'"
    $drops | foreach { Invoke-sqlcmd -Query "$_.IndexScript"}
    

#### And finally if needed, to execute the create statements:
    
    
    $creates = Invoke-sqlcmd -Query "SELECT IndexScript FROM dbo.IndexDdl WHERE DdlType = 'Create'" $creates | foreach{Invoke-sqlcmd -Query "$($_.IndexScript)"}
    

### Observations

Although the command are run interactively the Powershell scripts can easily be incorporated into a SQL Server Agent Powershell step job. The solution works on down level versions of SQL Server as long as SMO 10 (SQL 2008) and sqlps (SQL Server Powershell) are available. The Powershell and SMO based solution is much less code, more easily understandable and since a standard SMO Script method is used, less prone to breakage. Not every SQL Server administration problem is a nail. Put down your T-SQL hammer and pick up Powershell!

## Comments

**[Chad Miller](#106 "2010-02-17 16:22:00"):** I agree that the uptake of Powershell by DBAs has been slow, but the reason is T-SQL. The so called hammer principle applied to technology is described as anti-pattern: "a familiar technology or concept applied obsessively to many software problems" (<http://en.wikipedia.org/wiki/Law_of_the_instrument>). We need to start somewhere and what better way to start by using Powershell than to implement very simple one to four line scripts that greatly reduce the complexity found in equivalent T-SQL based solutions?  
As for your point on Powershell only being applicable to SQL 2008, I disagree. In my environment although I'm primarily SQL 2005, I also have 2000 and 2008 to support. The solution listed in this blog post will work against 2000 or 2005 in any of these scenarios:  
Install SQL Server 2008 Management Studio + Powershell on your workstation and it will run against 2000 and 2005. Obviously this has some drawbacks for automation.  
Install SQL Server 2008 Management Studio + Powershell on the 2000/2005 server. Notice we’re not upgrading the database engine. I've done this on a few servers where I felt there was a benefit in having the tools.  
Run from a server with SQL Server 2008 Management Studio + Powershell. The SQL Server Powershell provider has built in remoting. I could have run the Powershell listed in this post without altering on another server.  
One other option is install using SQL Server Powershell and SMO 10 (2008) without SSMS whichare freely available from Microsoft download. My thought on this is that it really doesn't buy you much over simply installing SSMS 2008.  
In my environment I've opted to install Powershell on every server including Windows 2003/SQL 2000, Windows 2003/SQL 2005 and Windows 2008/SQL 2008. Again this isn't required, just increases the possibilities. You can still do a lot just having Powershell on your individual workstation.  
  
Interestingly enough I think it’s precisely the need to support 2000 and 2005 version of SQL Server that will drive more DBAs to use Powershell. For example if we look at Policy Based Management (PBM), a great tool that was added to SQL Server 2008, running against 2000 and 2005 is limited in that you can't automate and store the results of the policy evaluation as you can when running against SQL 2008. Although you can use the GUI version easily against 2000/2005 the underlying tables in MSDB are simply not available in 2000/2005 to allow storing the collected data. So, what do you do? Well, Buck Woody created CodePlex project, htt://sqlcms.codeplex.com/ to more fully support PBM against downward level versions. And how did he do it? He used Powershell. Powershell provides the DBA the ability to fill gaps in current tool sets with a little scripting. I've previously blogged about the scripter being a toolsmith in my post "The Value Proposition of Powershell to DBAs" <http://chadwickmiller.spaces.live.com/blog/cns!EA42395138308430!347.entry> and PBM is just one more example.

**[Brent Ozar](#107 "2010-02-17 16:22:00"):** Chad - great idea with the post. I would agree that this is something that could be done well with PowerShell, but there's two things to keep in mind.  
  
First, someone has to maintain your code. The vast majority of DBAs out there don't know PowerShell yet. If you implement a solution in a language that they don't know, you've just backed yourself into a maintenance corner. Since I'm only a part-time DBA for my clients, I have to put solutions into place that they can maintain after I leave. This is especially true for mission-critical components like backup and recovery.  
  
Second, as much as we'd all like to be on SQL 2008 and PowerShell, that's not the reality for most enterprise customers yet. Most of my clients still have SQL Server 2000 in production. When something works, they're not eager to change it, especially when changing it costs money and time.  
  
I really look forward to the day when everybody's on 2008 - but unfortunately, that's going to be in the year 2013-2014. Until then, we're stuck with doing some T-SQL now and then. Good to see guys like you are blazing the trail, though!

