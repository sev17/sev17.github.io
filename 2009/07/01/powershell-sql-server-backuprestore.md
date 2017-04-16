title: Powershell SQL Server Backup/Restore
link: http://sev17.com/2009/07/01/powershell-sql-server-backuprestore/
author: Chad Miller
description: 
post_id: 9966
created: 2009/07/01 22:32:00
created_gmt: 2009/07/02 02:32:00
comment_status: open
post_name: powershell-sql-server-backuprestore
status: publish
post_type: post

# Powershell SQL Server Backup/Restore

I posted a [script on Poshcode for doing backups and restores of SQL Server databases](http://poshcode.org/1188) using [SMO](http://msdn.microsoft.com/en-us/library/ms162127.aspx). The script is adapted from SQL Server Powershell Extensions functions of the same name. SMO 9.0 (2005) and 10.0 (2008) have slightly different methods and properties at times. For the most part there is very little difference, however for the backup class there is one big difference--the backup class was moved from the base SMO assembly to the SMOExtended assembly. In order to account for using either assembly, the script does a couple of things:

  * Load the SMO version into a global variable $smoVersion which is then used in later sections of the code
  * Load the SMOExtended assembly in all cases. On a system with just SMO 9.0 the SMO Extended assembly will not be present, but that's OK. Loading a non-existent assembly does not produce an error in Powershell. Surprising, but true, try it yourself by misspelling an assembly name.

The other thing the script does is make use of the [special error handling](/2009/06/powershell-smo-error-handling-tips/) needed for SMO due to the use of innerExceptions.

Looking at the Powershell + SMO script you can't help but think, how much simplier backup and restores are in T-SQL.  Here's the equivalent T-SQL command to backup a database:

BACKUP DATABASE AdventureWorks TO DISK = 'C:backupadventureworks.bak' WITH FORMAT; 

In this case T-SQL would be much easier and less code to accomplish the same task of backing up or restoring a database.