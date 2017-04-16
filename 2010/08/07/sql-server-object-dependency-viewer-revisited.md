title: SQL Server Object Dependency Viewer Revisited
link: http://sev17.com/2010/08/07/sql-server-object-dependency-viewer-revisited/
author: Chad Miller
description: 
post_id: 10419
created: 2010/08/07 16:55:59
created_gmt: 2010/08/07 20:55:59
comment_status: open
post_name: sql-server-object-dependency-viewer-revisited
status: publish
post_type: post

# SQL Server Object Dependency Viewer Revisited

A year ago I blogged about [building a SQL Server 2008 Object Dependency Viewer](/2009/03/build-your-own-sql-server-2008-object-dependency-viewer/) based on [a script](http://dougfinke.com/blog/index.php/2009/01/26/powershell-visualize-the-peanut-butter-recall-data) by PowerShell MVP, Doug Finke ([Blog](http://www.dougfinke.com/blog/)|[Twitter](http://twitter.com/dfinke)). Since then Doug has created an [alternative solution based on the new Microsoft Automatic Graph Layout features in Visual Studio 2010](http://www.dougfinke.com/blog/index.php/2009/12/20/tainted-peanut-butter-the-sequel-powershell-dgml-and-visual-studio-2010/). The approach Doug takes builds a DGML XML file using PowerShell which then can be opened in Visual Studio 2010. I thought it would be interesting to create an updated version of the SQL Server Object Dependency Viewer using DGML. Rather than simply running Doug’s script as-is I decided to develop an alternative solution targeted for SQL Server. Because the new solution only requires creating a DGML XML file and SQL Server can emit XML natively I used the following T-SQL/XQuery (no PowerShell required! Nonetheless I included a one-liner at the end of this post) 

## Requirements:

  * SQL Server 2008 or higher database
  * Visual Studio 2010
Run the following script from SQL Server Management Studio 
    
    
    ;WITH
    xmlnamespaces
    (
    DEFAULT 'http://schemas.microsoft.com/vs/2009/dgml'
    )
    ,Links AS
    (SELECT 1 AS 'DirectedGraph')
    ,Link AS
    (SELECT DISTINCT
    OBJECT_NAME(referencing_id) AS [Source],
    COALESCE(referenced_server_name + '.','') + COALESCE(referenced_database_name + '.','')
    + COALESCE(referenced_schema_name + '.','') + referenced_entity_name AS [Target],
    o.type_desc AS SourceType
    FROM sys.sql_expression_dependencies AS sed
    INNER JOIN sys.objects AS o ON sed.referencing_id = o.object_id
    AND o.type_desc != 'CHECK_CONSTRAINT')
    
    SELECT
    (
    	SELECT [Source] AS "@Source", [Target] AS "@Target"
    	FROM Link for xml path('Link'), type
    )
    FROM Links for xml AUTO, root('DirectedGraph'), type

Save the output as a dgml file, for example AdventureWorksLT.dgml. Next double-click to open the file in Visual Studio. You should see a dependency graph similar to this: ![vsdepends1](http://images.sev17.com/vsdepends1_thumb.jpg) If you want to automate a the steps of saving and opening the DGML file. Save the T-SQL script above as dgml.sql and create a PowerShell script you can then call from sqlps host: 
    
    
    $fileName = "C:Usersu00DesktopAdventureWorks.dgml"
    Invoke-Sqlcmd -ServerInstance "Z003R2" -Database "AdventureWorksLT" -InputFile "C:Usersu00Desktopdgml.sql" -MaxCharLength 8000 | Select -ExpandProperty Column1 | Set-Content $fileName
    invoke-item $fileName