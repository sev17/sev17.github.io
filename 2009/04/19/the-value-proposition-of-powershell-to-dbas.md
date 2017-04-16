title: The Value Proposition of Powershell to DBAs
link: http://sev17.com/2009/04/19/the-value-proposition-of-powershell-to-dbas/
author: Chad Miller
description: 
post_id: 9953
created: 2009/04/19 13:33:00
created_gmt: 2009/04/19 17:33:00
comment_status: open
post_name: the-value-proposition-of-powershell-to-dbas
status: publish
post_type: post

# The Value Proposition of Powershell to DBAs

[Brent Ozar](http://www.brentozar.com/) posted the results of poll he conducted on Powershell adoption among database professionals in a post entitled, [Powershell poll results](http://www.brentozar.com/archive/2009/04/powershell-poll-results/). Of particular note, out of the 100 polled, which I assume are mostly database professionals only about 20% use Powershell. I wonder if the results of the poll are really that different when comparing DBAs with other system administration groups (Web, Exchange, Server, etc.). I mean Windows administrators really have never been heavy scripters. There is of course, a big opportunity for Powershell to change this. But, it wasn't so much the poll results which caught attention as it was the statements about Powershell benefits. You see I'm working on a Powershell presentation for SQL Server user groups and one thing I noticed is that we (DBAs who use Powershell) haven't done a great job of articulating the benefits of Powershell to our fellow SQL Server DBAs. I made a previous incomplete attempt to do so in my post, [Is PowerShell necessary for a DBA?](/2008/11/is-powershell-necessary-for-a-dba/).  DBAs are niche group of administrators and inevitably the Powershell discussion boils down to one question -- **"Why should I learn Powershell when I have a pretty good set of tools in T-SQL, SSMS and SSIS". **Notice the question isn't about VBScript vs. Powershell because DBAs like most Windows administration groups, have not used VBScript extensively. They do however, make heavy use of T-SQL. The use of a T-SQL as scripting language with near 100% adoption is what sets DBAs apart from their web, server or Exchange administrator counterparts. So the argument is really one of Powershell vs. T-SQL (and maybe SSIS), with that mind here is my attempt to better define the benefits of Powershell to DBAs:

  1. **Multi-server Automation** \-- Using Powershell you can do several things across multiple servers: execute a query, retrieve properties or even update configurations. Many of my [previous blog posts](http://chadwickmiller.spaces.live.com/?_c11_BlogPart_BlogPart=summary&_c=BlogPart) retrieve a list of SQL Servers from either a SQL table or text file and collect various information.
  2. **Retrieving Poperties is Easier** \-- Powershell makes getting properties easier than T-SQL or SSMS. For example this one line command exposes 97 properties of a SQL instance. It would take many SQL queries or lines of C# code to accomplish the same thing. $server = new-object (”Microsoft.SqlServer.Management.Smo.Server”) ‘Z002SQL2K8′ $server
  3. **Non-SQL tasks **\-- Powershell provides a better method for doing tasks outside of the SQL space. DBAs need to do things like check disk space, hotfixes, and delete files. These tasks are easy to accomplish with Powershell, but are impossible or ugly to do with T-SQL (hint: if you’re using xp_cmdshell, you probably should look into Powershell). Here’s an example getting disk space get-wmiobject win32_logicaldisk -computername ‘Z002′
  4. **Simple ETL** \-- While SSIS is great at complex ETL, Powershell makes it easy to to automate simple data loads. A Powershell script with a call to BULK INSERT or Data.SqlClient.SqlBulkCopy may be all that is required to load an CSV file. See [SQLServerCentral Article on Importing Powershell Output into SQL Server](/2008/12/sqlservercentral-article-on-importing-powershell-output-into-sql-server/) for several examples.
  5. **The non-DBA DBA** \-- A DBA is unlikely to use Powershell to do database backups or create tables, but for administrators thrust into a DBA role that must support databases yet are not DBAs, Powershell provides a common scripting language. Granted this really isn’t of a benefit to a DBA, but probably is for a administrator who doesn’t know T-SQL. See [Dan Jones post on this subject](http://blogs.msdn.com/dtjones/archive/2008/08/29/powershell-vs-t-sql-or-why-did-we-add-powershell-support-in-sql2k8.aspx): 
  6. **Toolsmithing **– The ability to quickly create useful tools that provide functionality you would normally have to purchase is especially important with the current economic conditions. Many IT departments are being told to only make purchases directly tied to business driven project. Powershell makes it very easy for a system administrator who may not have a developer background to quickly fashion useful tools. For examples see my previous blog postings on [Build your own SQL Dependency viewer](/2009/03/build-your-own-sql-server-2008-object-dependency-viewer/) and [Disk, database and table space charts](http://sev17.com/2009/04/making-disk-database-and-table-graphs-with-powershell/).
Getting started with Powershell SQL tasks can be a bit of a learning curve, if you’re looking at using SQL + Powershell you may want to check out my Codeplex project [SQL Server Powershell Extensions](http://sqlpsx.codeplex.com/) which provides over 100 functions for common SQL administration tasks. Can you think of any additional items or am I way off on my initial assumption (Powershell vs. T-SQL)?

## Comments

**[Chad Miller](#58 "2009-06-16 13:33:00"):** *** Post from Microsoft SMO developer Michiel Wories on using $error[0]|format-list -force to get back useful error messages from SMO. Nice! <http://blogs.msdn.com/mwories/archive/2009/06/08/powershell-tips-tricks-getting-more-detailed-error-information-from-powershell.aspx>

**[Chad Miller](#59 "2009-06-08 13:33:00"):** *** UPDATE *** Allen White has a nice write-up on error handling with the CheckTables method that shows you how to get back meaningful messages: <http://sqlblog.com/blogs/allen_white/archive/2009/06/08/handling-errors-in-powershell.aspx>

**[Chad Miller](#60 "2009-05-25 13:33:00"):** The reason I ask about the version is that SMO 10.0 which is 2008 has added some options over what was available in SMO 9.0 (2005). The target database being 2000 makes no difference for these methods since they are backwards compatible. As for the particular database question, like you, I was thinking what might be different, for example I've had issues running a reindex for particular databases even in T-SQL when dealing with computed columns. It seems like you've narrowed it down to a space issue. Keep in mind the lack of error messages is a SMO thing and not technically a Powershell issue. You might want to check $error[0], just to make sure an error message is not being set. As I mentioned you can get back detailed error messages in SMO 10.0 with the checktables method. Here's an example:  
  
$db.checktables("None",[Microsoft.SqlServer.Management.Smo.RepairOptions]"AllErrorMessages")  
  
Unfortunately, the various rebuild methods do not have error message options like checktables method.

**[Ian](#61 "2009-05-25 13:33:00"):** I may have worked out what is happening with the $tbl.RebuildIndexes(90) . I ran my script across another sql2005 instance and it died on one particular database table. I then ran DBCC REINDEX against the table in question in a query window and it failed, returning the error that my database had run out of space. I increased the space and the DBCC re-ran fine, after which $tbl.RebuildIndexes(90) did also. I think this may be the problem with my other reindexes, as both are on big tables. When they die they rollback and release the space they consumed. But that doesn't explain why DBCC REINDEX works (although it almost blows the space on my database), or why running $idx.Rebuild() on each index in turn doesn't blow the database limit either. The lack on meaningful information in powershell error messages is frustrating.

**[Ian](#62 "2009-05-24 13:33:00"):** Using 2005, but we're just introducing 2008 now so I will try it on that. Only getting the problem on certain databases. They both happen to be SQL2000 databases. Is this significant?

**[Chad Miller](#63 "2009-05-22 13:33:00"):** Are you using 2005 (9.0) or 2008 (10.0) SMO? In 2008, the checktables method has a repairoptions parameter which can be used to specify AllowErrorMessages. With 2005 there is no such option. Are you only having this issue with a certain database or all databases?

**[Ian](#64 "2009-05-22 13:33:00"):** Thanks, Chad. That is very helpful. I see the $idx.reorganize works, it just produces no output in powershell. As you said, I can see the $db.CheckTables () actually runs a DBCC CHECKDB WITH NO_INFOMSGS (which is probably why some error messages in Powershell are so uninformative). When I run CheckTables(0) against my database it died (as per last time), but when I then ran DBCC CHECKDB against the same database immediately afterward it ran fine. I then ran $db.checktables(0) again (to make sure the first time didn't somehow fix the problem even though it died) but it died again. This is very odd.

**[Chad Miller](#65 "2009-05-18 13:33:00"):** Running a profiler trace to capture the SQL statement which is sent to the server as a result of a Powershell SMO call can provide additional details. For instance:  
$tbl.RebuildIndexes(90) results in something like this DBCC DBREINDEX(N'[dbo].[ErrorLog]', N'', 90)  
$idx.Reorganize() results in this type of statement ALTER INDEX [PK_ErrorLog_ErrorLogID] ON [dbo].[ErrorLog] REORGANIZE WITH ( LOB_COMPACTION = ON )  
$db.CheckTables(0) results in this statement DBCC CHECKDB(N'AdventureWorks') WITH NO_INFOMSGS  
  
So, my question would be -- Does the same SQL statement run and if so are there any error messages returned?

**[Ian](#66 "2009-05-17 13:33:00"):** I am a SQL Server DBA who uses Powershell for all my admin tasks so I can run them from a central job scheduling service rather than rely on SQL Agent. However there are some frustrating things with database administation using Powershell that I can't understand or get around. eg I can't find a database method that corresponds to a DBCC CHECKDB. Database objects in Powershell have methods such as $db.checkallocations(0), $db.checktables(0) and $db.checkcatalog(), but no actual CHECKDB.   
Further to that, when I run CheckTables(0) on one of my databases I get the following error:  
Exception calling "CheckTables" with "1" argument(s): "Check tables failed for Database 'prd0Sylvan'. "  
At line:1 char:16  
\+ $db.CheckTables( <<<< 0)  
  
But if I run  
$tbls=$db.tables  
foreach ($tbl in $tbls) {$tbl.checktable()}  
  
it works fine.  
  
My lastest frustration is rebuilding indexes.   
If I run $tbl.RebuildIndexes(90) against a table in one particular database, it fails. But if I run the t-sql  
ALTER INDEX ALL ON port_sec_attrib REBUILD WITH (fillfactor = 90)  
in Management Studio it works fine.   
Oh, and the reason I run $tbl.RebuildIndexes(90) rather than check the fragementation level of each table and reorg rather than rebuild is it's quicker to rebuild all indexes than run $idx.EnumFragmentation('Fast') and check the averagefragmentation first. Which wouldn't matter anyway as $idx.Reorganize() doesn't appear to work.  
If anyone has any insights into what I'm doing wrong (or what Powershell is thinking) I'd love to hear them.

**[Chad Miller](#67 "2009-04-22 13:33:00"):** I forgot to mention, the wonderful SQL Server + Powershell talks Max Trinidad has been giving at various users groups and conferences. Check out his blog <http://max-pit.spaces.live.com/>

**[Chad Miller](#68 "2009-04-22 13:33:00"):** Richard Siddaway's brings up a couple of points I hadn't considered. Check out his blog post on PowerShell for DBAs <http://richardsiddaway.spaces.live.com/blog/cns!43CFA46A74CF3E96!2235.entry>

