title: Importing CSV Files to SQL Server with PowerShell
link: http://sev17.com/2011/11/28/importing-csv-files-to-sql-server-with-powershell/
author: Chad Miller
description: 
post_id: 10792
created: 2011/11/28 08:02:51
created_gmt: 2011/11/28 13:02:51
comment_status: open
post_name: importing-csv-files-to-sql-server-with-powershell
status: publish
post_type: post

# Importing CSV Files to SQL Server with PowerShell

Ed Wilson ([Blog](http://technet.microsoft.com/en-us/scriptcenter/default.aspx)|[Twitter](http://twitter.com/scriptingguys/)) aka Scripting Guy is kicking off another guest blogger week  (Nov 28th 2011) with my guest blog post, [Four Easy Ways to Import CSV Files to SQL Server with PowerShell](http://blogs.technet.com/b/heyscriptingguy/archive/2011/11/28/four-easy-ways-to-import-csv-files-to-sql-server-with-powershell.aspx). The post demonstrates the following approaches to importing CSVs into a SQL Server table: 

  * T-SQL BULK INSERT command
  * LogParser command-line
  * LogParser COM-based scripting
  * A Windows Powershell-based approach using several functions

Most of the time I'll I use BULK-INSERT or the Windows Powershell-based approach, although as explained in the post the ability of LogParser to automatically create a SQL table based on a CSV is pretty handy.