title: SQLPSX 1.4 Release
link: http://sev17.com/2009/01/15/sqlpsx-1-4-release/
author: Chad Miller
description: 
post_id: 9935
created: 2009/01/15 23:11:00
created_gmt: 2009/01/16 03:11:00
comment_status: open
post_name: sqlpsx-1-4-release
status: publish
post_type: post

# SQLPSX 1.4 Release

I completed [Release 1.4 of SQLPSX ](http://www.codeplex.com/SQLPSX/Release/ProjectReleases.aspx?ReleaseId=21698)which adds 15 new functions for working with SQL Server Integration Services (SSIS). With this release there are now 74 total functions and 10 scripts around SMO, Agent, RMO, and SSIS. 

Here's a few example of working with SQL Server Integration Services:

Load an SSIS package and execute:

**$package = Get-ISPackage 'Z002_SQL1MyPackage' 'Z002'**

**$package.Execute()**

Recursively list all SSIS packages on the Z002SQL1 instance

**get-isitem '' 'Z002_SQL1' 'Z002SQL1' -recurse $true **

Recursively copy all SSIS packages and folder from the Integration Server Z002 to Z003. In addition change the Connection Manager named SSISCONFIG to the server Z003SQL2 during the copy process \--SQL to SQL, 

**copy-isitemsqltosql -path '' -topLevelFolder 'Z002_SQL1' -serverName 'Z002SQL1' -destination 'Z003_SQL2' -destinationServer 'Z003' -recurse $true -connectionInfo @{SSISCONFIG='Z003SQL2'}**

Recursively copy all SSIS packages and folder from the Integration Server Z002 to file system path C:usrbinSSIS  In addition change the Connection Manager named SSISCONFIG to the server Z003SQL2 during the copy process \--SQL to file

**copy-isitemsqltofile -path '' -topLevelFolder 'Z002_SQL1' -serverName 'Z002SQL1' -destination 'C:usrbinSSIS' -recurse $true -connectionInfo @{SSISCONFIG='Z002SQL2'}**

Recursively copy all SSIS packages and folder from file system path C:usrbinSSIS to the Integration Server Z003. In addition change the Connection Manager named SSISCONFIG to the server Z003SQL2 during the copy process

\--file to SQL **copy-isitemfiletosql -path 'C:usrbinSSIS' -destination 'Z003_SQL2' -destinationServer 'Z003' -recurse $true -connectionInfo @{SSISCONFIG='Z002SQL2'}**

List running packages on the Z002 server:

**Get-ISRunningPackage Z002**

The complete list of new functions added in the 1.4 Release:

**Copy-ISItemSQLToSQL** Copies a Package or SSIS folder from SQL to SQL **Copy-ISItemSQLToFile** Copies a Package or SSIS folder from SQL to file **Copy-ISItemFileToSQL** Copies a Package or SSIS folder from file to SQL **Get-ISData** Executes a query and returns an ADO.NET DataTable **Get-ISItem** Retrieves a list of SQL Server Integration Services folders and packages from the specified SQL Server instance. Returns a PackInfo Object. Note: Unlike the other SSIS functions this function requires a SQL instance name i.e. serverNameinstanceName **Get-ISPackage** Retrieves an SSIS package from the specified Integration Services server or file path. Returns a Package Object **Get-ISRunningPackage** Returns a list of running packages on the specified Integration Services server. Returns a RunningPackage object or collection of objects **Get-ISSqlConfigurationItem** Executes a query to retrieve a configuration item **New-ISApplication ** Base object for all other functions. Executes new-object ("Microsoft.SqlServer.Dts.Runtime.Application") **New-ISItem ** Creates a SQL storage folder for the specified Integration Services server **Remove-ISItem** Deletes a SQL storage folder or package on the specified Integration Services server **Rename-ISItem** Renames a SQL storage folder or package on the specified Integration Services server **Set-ISConnectionString** Sets the Connection string for an SSIS package. Useful for package configuration connection string which cannot be set dynamically at run or deploy time **Set-ISPackage** Saves an SSIS package to an Integration Services server or file path as a dtsx file. **Test-ISPath** Test the existance of a SQL storage folder or package on the specified Integration Services server

I choose to put the SSIS related functions into a separate Library file, LibrarySSIS.ps1. I did this because the SSIS related objects are in the Microsoft.SqlServer.Dts.Runtime namespace instead of the Smo or RMO namespace, so it made sense to use separate Library file. You'll need to source the additional Library file to load function definitions.  Also because SSIS 2005 and 2008 Microsoft.SqlServer.Dts.Runtime namespace are not compatible you'll need to change the load assembly to the specific 2005 or 2008 version.

With Release 1.4 complete, I'm starting work on the 1.5 Release which will add Remove, Add, and Update functions where appropriate (prior releases have focused soley on Get functions). My goal is to the 1.5 release be feature complete and then move to a 2.0 release which will re-implement the functions as proper C# cmdlets and providers OR re-implement everything as advanced functions for Powershell V2. I haven't decided yet, whether to go the C# route or the Powershell V2 route.