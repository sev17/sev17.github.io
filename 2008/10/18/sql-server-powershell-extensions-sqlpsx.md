title: SQL Server PowerShell Extensions (SQLPSX)
link: http://sev17.com/2008/10/18/sql-server-powershell-extensions-sqlpsx/
author: Chad Miller
description: 
post_id: 9917
created: 2008/10/18 17:02:00
created_gmt: 2008/10/18 21:02:00
comment_status: open
post_name: sql-server-powershell-extensions-sqlpsx
status: publish
post_type: post

# SQL Server PowerShell Extensions (SQLPSX)

SQL Server PowerShell Extensions ([SQLPSX](http://www.codeplex.com/sqlpsx)) is open source project hosted on [CodePlex](http://www.codeplex.com/) for purpose of providing an intuitive PowerShell scripting experience when working with SQL Server.  The original reason I created SQLPSX was to collect, analyze and report SQL Server security and permission settings and since then it has expanded to work with many of the more common SQL Server tasks. Work on SQLPSX started in September 2007 and the first Release 1.0 was completed in July 2008. Release 1.1 followed in September 2008 and includes the folllowing functions and reusable scripts:

**Get-SqlServer**
Returns a Microsoft.SqlServer.Management.Smo.Server Object **Get-SqlDatabase**
Returns an SMO Database object or collection of Database objects **Get-SqlData**
Executes a query returns an ADO.NET DataTable
**Set-SqlData**
Executes a query that does not return a result set 
**Get-SqlShowMbrs**
Recursively enumerates AD/local groups handling built-in SQL Server Windows groups
**Get-SqlUser**
Returns a SMO User object with additional properties including all of the objects owned by the user and the effective members of the user. Recursively enumerates nested AD/local groups 
**Get-SqlDatabaseRole**
Returns a SMO DatabaseRole object with additional properties including the effective members of a role recursively enumerates nested roles, and users 
**Get-SqlLogin**
Returns a SMO Login object with additional properties including the effective members of the login 
**Get-SqlLinkedServerLogin**
Returns a SMO LinkedServerLogin object with additional properties including LinkedServer and DataSource 
**Get-SqlServerRole**
Returns a SMO ServerRole object with additional properties including the effective members of a role. Recursively enumerates nested AD/local groups 
**Get-SqlServerPermission**
Returns a SMO ServerPermission object with additional properties including the effective members of a grantee. Recursively enumeates nested roles and logins
**Get-SqlDatabasePermission**
Returns a SMO DatabasePermission object with additional properites including the effective members of a grantee. Recursively enumerates nested roles and users
**Get-SqlObjectPermission**
Returns a SMO ObjectPermission object with additional properties including the effective members of a grantee. Recursively enumerates nested roles and users
**Get-SqlTable**
Returns a SMO Table object with additional properties **Get-SqlStoredProcedure**
Returns a SMO StoredProcedure object with additional properties **Get-SqlView**
Returns a SMO View object with additional properties **Get-SqlUserDefinedDataType**
Returns a SMO UserDefinedDataType object with additional properites **Get-SqlUserDefinedFunction**
Returns a SMO UserDefinedFunction object with additional properites **Get-SqlSynonym**
Returns a SMO Synonym object with additional properites **Get-SqlTrigger**
Returns a SMO Trigger object with additional properites. Note: A Trigger can have a Server, Database or Table/View parent object.
**Get-SqlColumn**
Returns a SMO Column object with additional properites. Note: A Column can have either a Table or View parent object. **Get-SqlIndex**
Returns a SMO Index object with additional properites. Note: An Index can have either a Table or View parent object.
**Get-SqlStatistic**
Returns a SMO Statistic object with additional properites **Get-SqlCheck**
Returns a SMO Check object with additional properites. Note: A Check can have either a Table or View parent object.
**Get-SqlForeignKey**
Returns a SMO ForeignKey object with additional properites
**Set-SqlScriptingOptions**
Sets scripting option used in Get-SqlScripter function by reading in the text file scriptopts.txt
**Get-SqlScripter**
Returns a SMO Scripter object. Any function which returns a SMO object can pipe to Get-SqlScripter. For example to script out all table in the pubs database: Get-SqlDatabase MyServer | Get-SqlTable | Get-SqlScripter 
**Get-Information_Schema.Tables**
Returns the result set from INFORMATION_SCHEMA.Tables for the specified database(s) along with the Server name
**Get-Information_Schema.Columns**
Returns the result set from INFORMATION_SCHEMA.Columns for the specified database(s) along with the Server name **Get-Information_Schema.Views**
Returns the result set from INFORMATION_SCHEMA.Views for the specified database(s) along with the Server name **Get-Information_Schema.Routines**
Returns the result set from INFORMATION_SCHEMA.Routines for the specified database(s) along with the Server name
**Get-SysDatabases**
Returns the result set from sysdatases for the specified server along with the Server name
**Get-SqlDataFile**
Returns a SMO DataFile object with additional properties **Get-SqlLogFile**
Returns a SMO LogFile object with additional properties **Get-SqlVersion**
Returns a custom object with the Server name and version number **Get-SqlPort**
Uses SQL-DMO to return the port number of the specified SQL Server **Get-Sql**
Uses WMI to list all of the SQL Server related services running on the specified computer along with the service state and service account
**Get-ShowMbrs**
Recursivley enumerates local Windows and AD groups similar to the NT Resource utility showmbrs.exe
**Get-InvalidLogins.ps1**
Lists invalid AD/NT logins/groups which have been granted access to the specified SQL Server instance. Script calls the system stored procedure sp_validatelogins and validates the output by attempting to resolve the sid against AD. The second level of validation is done because sp_validatelogins incorrectly reports logins/groups which have been renamed in AD. SQL Server stores the AD sid so renamed accounts still have access to the instance. Renamed logins/groups are listed with the renamed value in the newAccount property. **Test-SqlConn.ps1**
Verifies Sql connectivity and writes successful connection to stdout and faiiled connections to stderr. Script is useful when combined with other scripts which would otherwise produce a terminating error on connectivity