title: Build Your Own SQL Server 2008 Object Dependency Viewer
link: http://sev17.com/2009/03/14/build-your-own-sql-server-2008-object-dependency-viewer/
author: Chad Miller
description: 
post_id: 9948
created: 2009/03/14 19:43:00
created_gmt: 2009/03/14 23:43:00
comment_status: open
post_name: build-your-own-sql-server-2008-object-dependency-viewer
status: publish
post_type: post

# Build Your Own SQL Server 2008 Object Dependency Viewer

I saw a demonstration by  [Doug Finke](http://dougfinke.com/blog/) during the [Windows PowerShell Virtual User Group Meeting #9 ](http://marcoshaw.blogspot.com/2009/02/windows-powershell-virtual-user-group.html) for displaying network graphs and thought this would be a great technique for visualizing SQL Server object dependencies. The Powershell code is available on Doug's blog post entitled [PowerShell, Visualize the Peanut Butter Recall Data](http://dougfinke.com/blog/index.php/2009/01/26/powershell-visualize-the-peanut-butter-recall-data). The scripts he provides use the [NodeXL](http://nodexl.codeplex.com/) .NET class libraries for creating network graphs.

With the first piece of our do-it-yourself project in place, we need to get the dependency information in a simple source/target pair format to pass the data to the Show-NetMap function. Unfortunately obtaining reliable object dependencies in SQL Server is somewhat difficult due to deferred name resolution and other dependency tracking issues. [Aaron Bertrand](http://sqlblog.com/blogs/aaron_bertrand/default.aspx) has a very detailed post describing the problems with dependency tracking in SQL Server 6.5 through SQL Server 2008 entitled [Keeping sysdepends up to date in SQL Server 2008](http://sqlblog.com/blogs/aaron_bertrand/archive/2008/09/09/keeping-sysdepends-up-to-date-in-sql-server-2008.aspx). As a result of the SQL dependency tracking issues most SQL Server professionals have learned not to trust the sysdepends tables. The only truly reliable method of determining dependencies remains to be parsing SQL code or purchasing 3rd party dependency viewer tools. I had originally planned on writing my own dependency parsing cmdlet, leveraging Visual Studio Database Edition ScriptDom class libraries, but quickly discovered the properties and methods which would expose this information is private. Fortunately SQL Server 2008 has added a new system catalog view called sys.sql_expression_dependencies which solves some, but not all of the dependency tracking issues (see Aaron's post for details). The new system catalog view may still have some issues, but for the most part its good enough for getting SQL object dependencies without parsing SQL code, so this is what we will use.

Before we get started we'll need to download [Doug's functions](http://dougfinke.com/blog/index.php/2009/01/26/powershell-visualize-the-peanut-butter-recall-data) and [SQL Server Powershell Extensions](http://www.codeplex.com/SQLPSX). The following code below uses the [SQL Server Powershell Extensions](http://www.codeplex.com/SQLPSX) function, Get-SqlData to get the output of a query against sys.sql_expression_dependencies and sys.objects. In order to show a simplier graph I'm filtering out check constraint dependency information. The data is then piped to Doug's Show-NetMap Powershell function:

. .**Show-NetMap**

$qry = @" SELECT DISTINCT OBJECT_NAME(referencing_id) AS [Source],  COALESCE(referenced_server_name + '.','') + COALESCE(referenced_database_name + '.','') \+ COALESCE(referenced_schema_name + '.','') + referenced_entity_name AS [Target], o.type_desc AS SourceType FROM sys.sql_expression_dependencies AS sed INNER JOIN sys.objects AS o ON sed.referencing_id = o.object_id "@ **get-sqldata** 'Z002SQL2K8' AdventureWorksLT $qry | ? {$_.SourceType -**ne** 'CHECK_CONSTRAINT'} | Select Source, Target | **Show-NetMap** **F**

_Network graph, showing a diagram of SQL Server object dependencies in the AdventureWorksLT sample database:_

![](http://images.sev17.com/depends.jpg)_ _

_Alternative query that provides more detailed column level dependency information:_

$qry = @" SELECT  OBJECT_NAME(referencing_id) + COALESCE('.' + COL_NAME(referencing_id, referencing_minor_id),'') AS [Source],  COALESCE(referenced_server_name + '.','') + COALESCE(referenced_database_name + '.','') \+ COALESCE(referenced_schema_name + '.','') + referenced_entity_name \+ COALESCE('.' + COL_NAME(referenced_id, referenced_minor_id), '') AS [Target], o.type_desc AS SourceType FROM sys.sql_expression_dependencies AS sed INNER JOIN sys.objects AS o ON sed.referencing_id = o.object_id UNION ALL SELECT OBJECT_NAME(referencing_id) AS [Source], OBJECT_NAME(referencing_id) + '.' + COL_NAME(referencing_id, referencing_minor_id) AS [Target], o.type_desc AS SourceType FROM sys.sql_expression_dependencies AS sed INNER JOIN sys.objects AS o ON sed.referencing_id = o.object_id WHERE referencing_minor_id <> 0 "@