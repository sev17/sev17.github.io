title: SQLPSX 2.3.2.1 Release
link: http://sev17.com/2011/03/13/sqlpsx-2-3-2-1-release/
author: Chad Miller
description: 
post_id: 10604
created: 2011/03/13 15:01:43
created_gmt: 2011/03/13 19:01:43
comment_status: open
post_name: sqlpsx-2-3-2-1-release
status: publish
post_type: post

# SQLPSX 2.3.2.1 Release

We’ve released a minor update to [SQLPSX](http://sqlpsx.codeplex.com/) which includes includes several bug fixes/enhancements as well as one new module. Here’s an excerpt from the release notes:

## Added MySQLLib Module

[Mike Shepard](http://powershellstation.com/) created a MySQL module for querying MySQL databases. This is similar in concept to the adolib and OracleClient modules.

## Modified adolib Module

  * Changed SQLBulkCopy to use System.Data.Common classes rather than SQLClient-specific classes for platform interoperability
  * Fixed issue in new-sqlcommand

## Modified OracleClient Module

Added OracleBulkCopy which allows you to bulk load data into Oracle   


## Modified SQLServer Module

Added FileListOnly to Invoke-SqlRestore function

## Modified PBM Module

Fixed issue in PBM module when writing to Windows Event log

## Modified SSIS Module

Added ProtectionLevel parameter.  The package protection level can now be changed/specified as part of the copy process 

## Credits

My thanks to

  * Bernd Kriszio ([blog](http://pauerschell.blogspot.com/)|[twitter](http://twitter.com/#%21/bernd_k)) for his OracleClient enhancements as well as feedback on adoblib 
  * [Mike Shepard](http://powershellstation.com/) for adding functionality and maintaining his adolib as well as branching out SQLPSX to include basic query support for MySQL.
  * Eric Humphrey ([blog](http://www.erichumphrey.com/)|[twitter](http://twitter.com/#!/lotsahelp)) for contributing a patch to the SSIS module which adds support for specifying the package protection level during copy processes.

## Comments

**[Mark](#239 "2011-10-04 23:19:58"):** Chad, Can I used SQLPSX on a system without having SMO if I want to use it just for getting (select) and input data to a SQLServer? I am looking for a moduale where it have a few functions like get data and save data to SQLServer without having someone to load SMO just to talk to a SQLServer. Where do I find a few nice functions that can do this that should not need to load a lot of 3rd-party software. Thanks for your help!

**[Chad Miller](#240 "2011-10-06 13:03:45"):** Mark, There's a few ways to handle this. SMO ultimately executes T-SQL for many of its tasks and you can see this by running a trace. If all you want to do is execute queries without relying on SMO then have a look at the adolib module which provides many Powershell functions for working with ADO.NET which is on every machine with .NET. There's also a much simplier route where you can use a function called invoke-sqlcmd2: http://poshcode.org/2279 Lately I've been working a SQL Proxy application which uses Powershell remoting. The client machines that use the proxy application don't need SMO as they just communicate with the proxy server via Powershell remoting. I'll definitely will share code and blog about the setup, but its a few weeks out.

