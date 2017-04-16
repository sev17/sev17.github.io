title: SQLPSX 2.3 Release
link: http://sev17.com/2010/11/06/sqlpsx-2-3-release/
author: Chad Miller
description: 
post_id: 10460
created: 2010/11/06 18:11:15
created_gmt: 2010/11/06 22:11:15
comment_status: open
post_name: sqlpsx-2-3-release
status: publish
post_type: post

# SQLPSX 2.3 Release

Just in time for [PASS Summit 2010](http://www.sqlpass.org/Events/PASSSummit.aspx), the CodePlex project SQL Server PowerShell Extensions ([SQLPSX](http://sqlpsx.codeplex.com/)) has been updated . Here’s a rundown of what’s new in release 2.3… 

### Added MSI-based installer

The installer was built using the Windows Installer XML ([WiX](http://wix.codeplex.com/)) and this was an exercise in learning WiX. I hope to blog about building an MSI the command-line way soon. 

### Added PBM Module

The PBM module adds functions which use the out-of-box cmdllet **[Invoke-PolicyEvaluation](http://technet.microsoft.com/en-us/library/cc645987.aspx)** provided in the sqlps mini-shell. In addition to the PowerShell functions, tables and sample policies are provided that work with SQL 2000, 2005 or 2008 or 2008 R2. The module is similar to [EPM Framework](http://epmframework.codeplex.com/), but a much simpler implementation. One of things the PBM module provides is the ability to write policy evaluations to both a consolidated SQL table and if desired to the Windows EventLog. Why write the policy evaluation to the EventLog? Well, this is how you can integrate PBM with [SCOM](http://technet.microsoft.com/en-us/systemcenter/om/default.aspx). Just have your SCOM guys pick up the particular EventID. 

### Modified adolib module

We added invoke-bulkcopy which uses [SqlBulkCopy](http://msdn.microsoft.com/en-us/library/system.data.sqlclient.sqlbulkcopy.aspx) class and allows column mapping. I’m a big fan of using the SqlBulkCopy class to load data using PowerShell and now [Mike Shepard](http://powershellstation.com/) has given us a nice function to handle both simple and more complex data loading. 

### Modified OracleIse and SQLIse modules

#### Both

We moved saved preference to [Isolated Storage](http://msdn.microsoft.com/en-us/library/3ak841sy%28v=VS.100%29.aspx) instead of module directory, this means makes it easier to use system instead of user module location if so desired. We modified the prompt to show both Oracle and SQL Server connections. That’s right you can load OracleIse and SQLIse to query both data sources at the same time! We added auto, table, list, and isetab output options and moved output option to separate dialog box. The output option enhancements are pretty slick. I find send query output to a new tab especially useful. 

#### OracleIse Specific

  * Added PoshMode to OracleIse (already existed in SQLIse) which allows embedding PowerShell variables in queries

#### SQLIse Specific

  * Added print and raiserror handling.
  * Added multi-query handling

### Modified SqlServer module

We changed statement timeout from the SMO default of 10 minutes to unlimited. This was needed for backup and other long operations. We fixed issues/added functionality in specifying database objects. This includes new pattern matching object names and speeding up retrieval of specific database objects in most of the **get-** functions. We added progress indicator, percent complete and success messages to backup and restore functions. You’ll now see a progress bar when doing a backup or restore. In addition if the _–verbose_ option is specified you’ll see the percent complete and success messages you’re used to seeing in SQL Server Management Studio. 

### Credits

My thanks to 

  * [Bernd Kriszio](http://pauerschell.blogspot.com/) for his work on OracleIse and many of the enhancements to SQLIse
  * [Mike Shepard](http://powershellstation.com/) for adding functionality to adolib

### Next Steps

Download [SQLPSX](http://sqlpsx.codeplex.com/) and leave some [feedback on the SQLPSX site](http://sqlpsx.codeplex.com/discussions)