title: SQLPSX 1.5 Release
link: http://sev17.com/2009/03/28/sqlpsx-1-5-release/
author: Chad Miller
description: 
post_id: 9949
created: 2009/03/28 12:48:00
created_gmt: 2009/03/28 16:48:00
comment_status: open
post_name: sqlpsx-1-5-release
status: publish
post_type: post

# SQLPSX 1.5 Release

I completed [Release 1.5 of SQLPSX](http://sqlpsx.codeplex.com/Release/ProjectReleases.aspx?ReleaseId=25394) which adds 31 new functions for working with database maintenance (CHECKDB, Index rebuilds, backup and restore) as well as login, user, role and permission management. With this release there are now 104 total functions, 2 cmdlets and 12 scripts around SMO, Agent, RMO, and SSIS.

Here's a few examples working with database maintenance functions:

#Get a database object $db = **get-sqldatabase** 'Z002SqlExpress' pubs #Run a checkdatabse **invoke-sqldatabasecheck** $db $db | **invoke-sqldatabasecheck** #Get index defrag information for all indexes $db | **get-sqltable** | **get-sqlindex** | **get-sqlindexfragmentation** #Run an index defrag operation against all indexes $db | **get-sqltable** | **get-sqlindex** | **invoke-sqlindexdefrag** #Run an reindex operation against all indexes $db | **get-sqltable** | **get-sqlindex** | **invoke-sqlindexrebuild** #Run an update statistics operations against all statistics $db | **get-sqltable** | **get-sqlstatistic** | **update-statistic** #Get a server object $server = **Get-SqlServer** 'Z002SqlExpress' #Return log and data directory information: **Get-SqlDefaultDir** 'Z002SqlExpress' #Create a new database **Add-sqldatabase** 'Z002SqlExpress' test #Remove a database **Remove-sqldatabase** 'Z002SqlExpress' test #Add a WindowsGroup login **add-sqllogin** 'Z002SqlExpress' 'Z002TestGrp1' -logintype 'WindowsGroup' #Add a SqlLogin **add-sqllogin** 'Z002SqlExpress' test5 test5 -logintype 'SqlLogin' #Add a Windowsuser login **add-sqllogin** 'Z002SqlExpress' 'Z002testuser1' -logintype 'WindowsUser' #Add a User **add-sqluser** 'Z002SQLEXPRESS' pubs test5 #Add Windows user **add-sqluser** 'Z002SQLEXPRESS' pubs 'testuser1' 'Z002testuser1' #Remove a user **remove-sqluser** 'Z002SQLEXPRESS' pubs 'testuser1' #Remove a login **remove-sqllogin** 'Z002SqlExpress' test6 #Add a role member to the bulkadmin server role **add-sqlserverrolemember** 'Z002SqlExpress' 'test5' bulkadmin #Remove a role member from the bulkadmin server role **remove-sqlserverrolemember** 'Z002SqlExpress' 'test5' bulkdmin #Add a database role **add-sqldatabaserole** 'Z002SqlExpress' pubs testrole3 #Remove a database role **remove-sqldatabaserole** 'Z002SqlExpress' pubs testrole3 #Add a database role member  **add-sqldatabaserolemember** 'Z002SqlExpress' pubs test5 testrole3 #Remove a database role member **remove-sqldatabaserolemember** 'Z002SqlExpress' pubs test5 testrole3 #Get schemas from a database $db | **get-sqlschema** $db | **get-sqlschema** -name dbo #Return current processes **Get-SqlProcess** 'Z002SqlExpress' | ft #Return active transaction in the tempdb database **get-sqltransaction** 'Z002SqlExpress' tempdb #Return the current ErrorLog **get-sqlerrorlog** 'Z002SqlExpress' #Set server level permission **set-sqlserverpermission** 'Z002SqlExpress' AlteAnyLogin test5 Grant #Set database level permission **set-sqldatabasepermission** 'Z002SqlExpress' pubs CreateTable test5 Grant #Set object level permission $db | **get-sqlschema** -name dbo | **set-sqlobjectpermission** -permission Select -name test5 -action Grant