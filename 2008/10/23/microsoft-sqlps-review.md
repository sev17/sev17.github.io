title: Microsoft SQLPS Review
link: http://sev17.com/2008/10/23/microsoft-sqlps-review/
author: Chad Miller
description: 
post_id: 9921
created: 2008/10/23 23:58:00
created_gmt: 2008/10/24 03:58:00
comment_status: open
post_name: microsoft-sqlps-review
status: publish
post_type: post

# Microsoft SQLPS Review

There have been several blog entries which about Microsoft's sqlps.exe, the PowerShell host provided with SQL Server 2008, most of which have centered around the SQL Server product teams' decision to implement a closed shell. It would seem after much [negative feedback](http://concentratedtech.com/content/index.php/2008/06/sql-server-2008-powershell-no-no-no-no-no/) on the mini-shell there is at least is some [acknowledgement by Microsoft](http://blogs.msdn.com/powershell/archive/2008/06/23/sql-minishells.aspx) this was a poor design decision. As a power PowerShell user, I too find the mini-shell usage to be very limiting, however beyond the mini-shell/closed shell question there are other implementation issues I find equally baffling.

The SQL Product Team basically only gives us one useful cmdlet, invoke-sqlcmd, the rest of implementation is based on a SqlServer PSDrive Provider. We can navigate SQL like a drive. Here's an example cd-ing into databases and doing an ls on the databases:

![sqlps](http://byfiles.storage.live.com/y1pTZ9thuCk_x0__RM4v11VYmuASGOG7xjq_9HNO0J1zT_4c1hzmwPhdjZqWtbcYlANGWtuKjC9YVo)

Working with sqlps.exe you quickly notice it has the same navigational feel as the GUI, SQL Server Management Studio (SSMS). In GUI navigation you click on a server, expand databases and then tables. In sqlps.exe you cd to the server then cd into databases then cd into tables. It's as if the SQL Product Team tried to reproduce SSMS on the PowerShell command-line. Although implementing sqlps.exe in this fashion will give DBAs a familiar experience they are used to when using SSMS, the model quickly breaks down when you want to do anything more than navigate or list properties. This is immediately apparent upon reading [SQL Server 2008 Books On Line](http://msdn.microsoft.com/en-us/library/bb543165.aspx) (BOL) [entry on Powershell](http://msdn.microsoft.com/en-us/library/cc281947.aspx). In this BOL excerpt the sample code uses native SMO syntax to create a database:

Set-Location SQLSERVER:SQLMyComputerDEFAULTDatabases $MyDBVar = New-Object [Microsoft.SqlServer.Management.SMO.Databases] $MyDBVar.Parent = (Get-Item ..) $MyDBVar.Name = "NewDB" $MyDBVar.Create() $MyDBVar.State

Based on a read of BOL and the lack of cmdlets it would seem the SQL Product Team expects a DBA to know the SMO object model to work in sqlps.exe. In [my article on SQLPSX ](http://www.sqlservercentral.com/articles/powershell/64316/)I expressed a concern about Powershell being a doubling steep learning curve for DBAs because of the added burden of having to learn SMO. The SSMS-style experience in sqlps.exe does not hold up when you are forced to learn SMO. This could have easily been solved by providing key cmdlets to abstract away SMO as was done in [SQLPSX](http://www.codeplex.com/SQLPSX). Instead working in sqlps.exe you feel like you are working with a product which is half finished you can navigate and list properties but anything more requires SMO scripting.

Using the SQL Server provider outside of sqlps.exe by loading the SQL Server provider extensions as suggested at [Michiel Wories' WebLog](http://blogs.msdn.com/mwories/default.aspx) blog [post](http://blogs.msdn.com/mwories/archive/2008/06/14/SQL2008_5F00_Powershell.aspx) does not resolve these issues. Since we do not have any cmdlets other than invoke-sqlcmd the scriptability of the SQL Server provider is akward and verbose. Here's a comparison between [SQL Server PowerShell Extensions (SQLPSX) ](http://www.codeplex.com/SQLPSX)and Microsoft SQL Server PowerShell provider (sqlps.exe) syntax to accomplish the same task:

**[Table: sqlpsx vs. sqlps.exe](http://cid-ea42395138308430.skydrive.live.com/self.aspx/Public/Blog/sqlpsx|_vs|_sqlps.htm)**

The SQL Product Team is [planning on releasing SQL Server cmdlets](http://blogs.msdn.com/mwories/archive/2008/06/25/what-no-cmdlets-sql-server-powershell.aspx), whether this will be done in SQL 2008 or the next version is unclear. In the meantime I have a lot of work to do on [SQLPSX](http://www.codeplex.com/SQLPSX). If you are a good SMO, Powershell and C# developer contact me as I could use the help.