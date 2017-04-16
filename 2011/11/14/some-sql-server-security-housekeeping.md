title: Some SQL Server Security Housekeeping
link: http://sev17.com/2011/11/14/some-sql-server-security-housekeeping/
author: Chad Miller
description: 
post_id: 10768
created: 2011/11/14 07:49:45
created_gmt: 2011/11/14 12:49:45
comment_status: open
post_name: some-sql-server-security-housekeeping
status: publish
post_type: post

# Some SQL Server Security Housekeeping

Managing SQL Server security changes in mass is something which screams automate it. Let's look a at few examples using either T-SQL, a [Centeral Management Server](http://msdn.microsoft.com/en-us/library/bb895144.aspx) and Powershell. 

## Task #1 Remove a SQL Server Login from the Sysadmin Role

Let's say you're given the task to remove a login from the sysadmin role on 50 servers.  For this task we can use the built-in system stored procedures [sp_dropsrvrolemember](http://msdn.microsoft.com/en-us/library/ms186270.aspx)  and [sp_helpsrvrolemember](http://msdn.microsoft.com/en-us/library/ms188772.aspx). Since we want to do this across multiple servers, we'll use a multiserver query from the registered central management: ![](http://images.sev17.com/multiquery-279x300.jpg) 1\. First create a before removal "backup" using sp_helpsrvrolemember: 
    
    
    sp_helpsrvrolemember 'sysadmin'

2\. Save the output 3\. Next run 
    
    
    EXEC sp_dropsrvrolemember 'CONTOSOSQL_SecurityAdmin', 'sysadmin'

Error handling is generally good thing, but there are cases were can do a task without much error handling especially if the task is interactive and you have backups. Although you can wrap some T-SQL code to check if the login exists on the server or if the login is member of the server before attempting to removing from the sysadmin, its not necessary any errors can safely be ignored. A Central Management Server is great at one off commands with good T-SQL coverage 

## Task #2 Remove Linked Server Login Mappings

After removing the login from the sysadmin you discover they are also mapped to sa on over 100  Linked Servers. Since you can't  remove the login from the server because they still need non-administration access, your task to to remove the linked server login mapping. Changing linked server security  in mass is something where Powershell provides a good solution. You'll probably want to backup the linked server by scripting out the linked server before any changes which is difficult to do in T-SQL, but easy with Powershell. Also removing a login mappings from all linked servers  is very procedural which is awkward in T-SQL. So here's a Powershell solution I created in the form of a few Powershell filters and functions called [LibraryLinkedServer](http://poshcode.org/3048). 
    
    
    . .LibraryLinkedServer.ps1
    $logins = @(
    'ContosoBill'
    'ContosoJohn'
    'ContosoJill'
    )
    Get-CMRegisteredServer "Z001SQL1" "PRD" | Backup-LinkedServer -LinkedServerLogins $logins
    Get-CMRegisteredServer "Z00SQL1" "PRD" | Remove-LinkedServerLogin -LinkedServerLogins $logins

I would suggest running the code from Powershell ISE so you can step through the code by highlighting and running each line. ![](http://images.sev17.com/powershell_ise_ls-300x105.jpg) The solution first sources the LibraryLinkedServer functions and filters i.e. loads the functions into our current Powershell session. Next we define our list of logins t removing from linked server mappings in an array called $logins. The final two steps involve obtaining a list of SQL Servers from the Central Managment Server Z001SQL1 and the server group PRD. Next we'll script out the linked servers using the Backup-LinkedServer function. At this stage I would manually verify the backup files before proceeding with removing the login mappings using Remove-LinkedServerLogin. 

## Task #3 Remove a Linked Server

You discover linked servers in development which point to production. The linked server should be removed entirely. Your task is to remove a specific linked server named PROD1  from all development servers.  Here again we'll use the [LibraryLinkedServer](http://poshcode.org/3048) functions: 
    
    
    . .LibraryLinkedServer.ps1
    Get-CMRegisteredServer "Z001SQL1" "DEV" | Backup-LinkedServer -LinkedServer "PROD1"
    Get-CMRegisteredServer "Z001SQL1" "DEV" | Remove-LinkedServer -LinkedServer "PROD1"

This task is very similar to task #2 and much of the code is the same, except here we're specifying a linked server name rather than an array of logins. Again I would suggest manually verifying the backup files before proceeding with removing the linked server. _Note: If you use this solution make sure you test, make backups and verify._