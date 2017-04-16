title: Executing Powershell in SQL Server
link: http://sev17.com/2009/04/05/executing-powershell-in-sql-server/
author: Chad Miller
description: 
post_id: 9951
created: 2009/04/05 13:23:00
created_gmt: 2009/04/05 17:23:00
comment_status: open
post_name: executing-powershell-in-sql-server
status: publish
post_type: post

# Executing Powershell in SQL Server

Let me put out a disclaimer the script and technique described in this post is a bit of a hack and I find loading SQL Server data from Powershell easier rather than executing Powershell within SQL Server to load data. The former is cleaner and uses standard .NET assemblies, XML or SQL Server utilities. See my previous post and the referenced article entitled [SQLServerCentral Article on Importing Powershell Output into SQL Server](http://chadwickmiller.spaces.live.com/blog/cns!EA42395138308430!236.entry) for details. But if you really want to run Powershell in SQL Server and get back a table result set here's one way to do so:

Copy the New-Xml script below which is simply a minor adaptation (I turned the original function into a script) of a function called New-Xml posted on the [Powershell Team blog ](http://blogs.msdn.com/powershell/default.aspx)blog entry [Using PowerShell to Generate XML Documents](http://blogs.msdn.com/powershell/archive/2007/05/29/using-powershell-to-generate-xml-documents.aspx). Save the code as New-Xml.ps1 and place in directory accessible by SQL Server service account on the SQL Server machine.

**param**($RootTag="ROOT",$ItemTag="ITEM", $ChildItems="*", $Attributes=$Null) Begin { $xml = "<$RootTag>`n" } Process { $xml += " <$ItemTag" **if** ($Attributes) { **foreach** ($attr **in** $_ | **Get-Member** -type *Property $attributes) { $name = $attr.Name $xml += " $Name=`"$($_.$Name)`"" } } $xml += ">`n" **foreach** ($child **in** $_ | **Get-Member** -Type *Property $childItems) { $Name = $child.Name $xml += " <$Name>$($_.$Name)</$Name>`n" } $xml += " </$ItemTag>`n" } End { $xml += "</$RootTag>`n" $xml } 

**Now, run the SQL script below from SQL Server Management Studio:**

**CREATE** TABLE #output (line **varchar**(255)) **INSERT** #output EXEC xp_cmdshell 'powershell.exe -c "get-service | c:usersu00binnew-xml.ps1 -childItems Name,Status"' **DELETE** #output WHERE line IS NULL \--The result set contains one row per line 255 characters we need to have a single variable with all lines \--Code below sets @doc to the entire contents of the #output table DECLARE @doc **varchar**(max) **SET** @doc = '' DECLARE @line **varchar**(255) DECLARE xml_cursor CURSOR FOR **SELECT** line FROM #output OPEN xml_cursor FETCH NEXT FROM xml_cursor INTO @line WHILE @@FETCH_STATUS = 0 BEGIN **SET** @doc = @doc + @line FETCH NEXT FROM xml_cursor INTO @line END CLOSE xml_cursor DEALLOCATE xml_cursor **DROP** TABLE #output \--Get the name and status out of the xml doc **SELECT** item.ref.value('(Name/text())[1]', 'nvarchar(128)') AS name, item.ref.value('(Status/text())[1]', 'nvarchar(128)') AS status FROM (**SELECT** CAST(@doc AS xml) AS feedXml) feeds(feedXml) CROSS APPLY feedXml.nodes('/ROOT/ITEM') AS item(ref) 

The script uses xp_cmdshell to execute powershell.exe and passes in a Powershell command or script file for execution (in this case Get-Service). Because xp_cmdshell returns the output of a command as a multiple rows of text up to 255 characters in length the output is first converted to XML using the New-Xml.ps1 script. The XML is then transformed into a relational result set using the XQuery extensions available in T-SQL. The solution is not without issues the output of Powershell commands may contain reserved words or special characters which the New-Xml script does not handle, but it's good enough for what we want to do.