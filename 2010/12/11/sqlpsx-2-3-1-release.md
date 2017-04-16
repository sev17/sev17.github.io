title: SQLPSX 2.3.1 Release
link: http://sev17.com/2010/12/11/sqlpsx-2-3-1-release/
author: Chad Miller
description: 
post_id: 10521
created: 2010/12/11 17:49:57
created_gmt: 2010/12/11 22:49:57
comment_status: open
post_name: sqlpsx-2-3-1-release
status: publish
post_type: post

# SQLPSX 2.3.1 Release

We’ve released a minor update to [SQLPSX](http://sqlpsx.codeplex.com/) which includes several bug fixes as well as two new modules. Here’s an excerpt from the release notes: 

## Added PerfCounters Module

The PerfCounters module contributed by Laerte Junior ([blog](http://laertejuniordba.spaces.live.com/)|[twitter](http://twitter.com/LaerteSQLDBA)), is used for working with Windows Performance Counters, collecting and importing into a SQL Server table. Checkout Laerte’s [Simple-Talk](http://www.simple-talk.com/default.aspx) article, [Gathering Perfmon Data with Powershell](http://www.simple-talk.com/sql/database-administration/gathering-perfmon-data-with-powershell/) for good overview of how you can use these functions. 

## Added SQLProfiler Module

Another contribution by Laerte Junior ([blog](http://laertejuniordba.spaces.live.com/)|[twitter](http://twitter.com/LaerteSQLDBA)), this module is used for reading and writing SQL trace files. Checkout Laerte’s article [Fun with SQL Server Profiler trace files and PowerShell](http://www.simple-talk.com/sysadmin/powershell/fun-with-sql-server-profiler-trace-files-and-powershell/) for more information. 

## Modified SQLIse Module

Fixed issues with output formatting of multi-queries and added SQL password encryption for saved connections using SQL authentication. 

## Modified adolib Module

Fixed issues with SQL authentication on some functions and added enhanced output types. 

## Credits

My thanks to 

  * Bernd Kriszio ([blog](http://pauerschell.blogspot.com/)|[twitter](http://twitter.com/#!/bernd_k)) for his continual support and enhancements to SQLIse.
  * [Mike Shepard](http://powershellstation.com/) for adding functionality and maintaining his adolib.
  * Laerte Junior ([blog](http://laertejuniordba.spaces.live.com/)|[twitter](http://twitter.com/LaerteSQLDBA)) for contributing two new modules SQLProfiler, PerfCounters as well as his previous work in creating a SQLMaint module (complete PowerShell-based SQL maintenance).

## Comments

**[Bernd Kriszio](#202 "2010-12-12 12:17:12"):** At daytime I work in a SCRUM team where we deal a lot with text and varchar(max) fields. We need to be able to extract the complete unmodified content in a painless way. Many of the modifications and enhancements I did to sqlise where influenced by work requirements. Now this release of sqlise is matured enough to be used as tool by its own in parallel with SSMS and sqlcmd. It fills some gaps where we have no influence on the other tools. My special thanks to Chad to let me add this stuff to the project and enhance sqlise this way. PS.: Some ORACLE improvements are planned for the next 3 month, but when I get no feedback, the progress is determined by my jobs needs.

