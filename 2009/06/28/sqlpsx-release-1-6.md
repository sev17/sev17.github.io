title: SQLPSX Release 1.6
link: http://sev17.com/2009/06/28/sqlpsx-release-1-6/
author: Chad Miller
description: 
post_id: 9965
created: 2009/06/28 13:15:00
created_gmt: 2009/06/28 17:15:00
comment_status: open
post_name: sqlpsx-release-1-6
status: publish
post_type: post

# SQLPSX Release 1.6

I completed [Release 1.6 of SQLPSX](http://sqlpsx.codeplex.com/Release/ProjectReleases.aspx?ReleaseId=29380) which adds support for SQL Authentication and addresses several issues. SQLPSX consists of 106 functions, 2 cmdlets and 12 scripts for working with SMO, Agent, RMO, SSIS and SQL script files.

Here's an example of using SQL Authentication:

$server = **Get-SqlServer** 'Z002SQL2K8' 'sa' 'mypassword'

Because Get-SqlServer, Get-AgentJobServer, and Get-ReplServer support either Windows or SQL Authentication you can pass a server object to the additional functions. Here's an example of getting databases from the server variable creating above:

**Get-SqlDatabase** $server | Select name 

Many of the functions support shortcuts when using Windows authentication. Because Windows authentication is used we do not have to first get a reference to a server object. manually Here's the same example getting databases using Windows authentication:

**Get-SqlDatabase** 'Z002SQL2K8'

Other than the addition of SQL authentication support, SQLPSX 1.6 is largely a maintenance release. I've replaced calls to WMIC with Get-WmiObject and addressed issues documented in the [Issue Tracker](http://sqlpsx.codeplex.com/WorkItem/List.aspx). I've also incorporated the [error handling technique Allen White provided](http://chadwickmiller.spaces.live.com/blog/cns!EA42395138308430!458.entry) into several functions in order to provide better error reporting. I had hoped to have a provider for SSIS completed, however I ran into a few issues with [weak support of functionality in the SSIS API](/2009/06/adventures-in-powershell-ssis-administration-programming/). If the SQL product team doesn't include an SSIS provider in SQL Server 2008 R2, I'll look at creating one then.

With Release 1.6 complete I'm planning on the next release, 2.0, to re-implement the V1 functions as advanced functions in Powershell V2.