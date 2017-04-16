title: SQLServerCentral SQLPSX Articles
link: http://sev17.com/2008/10/19/sqlservercentral-sqlpsx-articles/
author: Chad Miller
description: 
post_id: 9918
created: 2008/10/19 09:19:00
created_gmt: 2008/10/19 13:19:00
comment_status: open
post_name: sqlservercentral-sqlpsx-articles
status: publish
post_type: post

# SQLServerCentral SQLPSX Articles

I wrote a two part article on SQL Server PowerShell Extensiosn ([SQLPSX](http://www.codeplex.com/sqlpsx)) installation and usage.

[SQLPSX Part1](http://www.sqlservercentral.com/articles/powershell/64316/)

[SQLPSX Part2](http://www.sqlservercentral.com/articles/powershell/64350/)

Check out the discussion area for each article:

[Part 1 Discussion](http://www.sqlservercentral.com/Forums/Topic580775-106-1.aspx)

[Part 2 Discussion](http://www.sqlservercentral.com/Forums/Topic582357-106-2.aspx)

In Part  1 a reader had a question on creating PowerShell scripts to execute SQL queries with parameters using the SQLPSX function Get-SqlData. This is trival to do using the SQLPSX function Get-SqlData. Incidentally the MS SQL Server 2008 PowerShell cmdlet invoke-sqlcmd offers nearly identical functionality. I'm reposting my response here:

#When I need to execute a query with parameters I just create a PowerShell script for that specific purpose and load LibrarySmo #at the beginning of the script. Here's a simple example. **param**($au_lname) $scriptRoot = **Split-Path** (**Resolve-Path** $myInvocation.MyCommand.Path) . $scriptRootLibrarySmo.ps1 $srcServer = 'Z002SqlExpress' $qry = @" SELECT * FROM dbo.authors WHERE au_lname = '$au_lname' "@ **Get-SqlData** $srcServer 'pubs' $qry #To execute save the code as a ps1 file (here I'm using getAuthor.ps1). You'll also need to change the $srcServer variable or add #it as a parameter to script and pass the server name. ./getAuthor "White" #Also when I need to load the contents of a file such as a .sql file, I'll use the .NET ReadAllText method: $qry = **[System.IO.File]**::ReadAllText("c:usersu00scriptspubqry.sql")

In Part 2 a reader had a question about not seeing the SQL Agent job output in a particular step. This is due to my implementation of what I call basic threading which is really just launching child processe using the NET System.Diagnostics.ProcessStartInfo. The benefit to using this approach is that long running processes complete faster by launching additional PowerShell consoles. The drawback is the original session which started the additional processes has no knowledge of status of the new processes. See [my response](http://www.sqlservercentral.com/Forums/Topic582357-106-2.aspx) for a more detailed explanation. I think the implementation of background jobs in V2 should address the need to start child processes.