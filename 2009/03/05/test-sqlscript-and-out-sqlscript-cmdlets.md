title: Test-SqlScript and Out-SqlScript cmdlets
link: http://sev17.com/2009/03/05/test-sqlscript-and-out-sqlscript-cmdlets/
author: Chad Miller
description: 
post_id: 9947
created: 2009/03/05 21:18:00
created_gmt: 2009/03/06 01:18:00
comment_status: open
post_name: test-sqlscript-and-out-sqlscript-cmdlets
status: publish
post_type: post

# Test-SqlScript and Out-SqlScript cmdlets

I 've been using  Visual StudioTeam System 2008 Database Edition (VSDB) and noticed an interesting post on [Gert Drapers' blog](http://blogs.msdn.com/gertd/default.aspx), entitled [Getting to the Crown Jewels](http://blogs.msdn.com/gertd/archive/2008/08/21/getting-to-the-crown-jewels.aspx). In the posting Gert demonstrates a basic C# WinForm application which parsers and formats T-SQL using the assemblies [Microsoft.Data.Schema.ScriptDom ](http://msdn.microsoft.com/en-us/library/microsoft.data.schema.scriptdom.aspx)and [Microsoft.Data.Schema.ScriptDom.Sql](http://msdn.microsoft.com/en-us/library/microsoft.data.schema.scriptdom.sql.aspx) included in VSDB. A subsequent posting states if you own an official copy of Visual Studio Team System 2008 Database Edition GDR, which I do, you are allowed to redistribute these assemblies. Looking at the WinForm code, I thought this would make a couple of interesting cmdlets. So, I created two cmdlets called Test-SqlScript and Out-Sqlscript. Test-SqlScript parses a T-SQL script and tests whether the script is valid. Out-SqlScript in addition to validating the script, re-formats the script output with 25 different formatting options. I created help files with the cmdlets, you can see the formating options by using get-help Out-SqlScript.

_ _You might ask, how can I use these cmdlets? Well I can immediately think of two use cases:

  1. Use Test-SqlScript in conjunction with a source control check-in to verify the script is valid
  2. Use Out-SqlScript to "pretty-print" ugly SQL script within SQL Server Management Studio
To setup the formatter within SQL Server Management Studio: 
  1. Install the SQLParser snapin using Init-SqlParser.ps1 script included in the [download ](http://cid-ea42395138308430.skydrive.live.com/self.aspx/Public/SQLParser.zip)
  2. Add Add-PSSnapin to your profile
  3. Create a bat file called formatSql.bat (also included in the [download](http://cid-ea42395138308430.skydrive.live.com/self.aspx/Public/SQLParser.zip))
  4. In SQL Server Management Studio go to Tools => External Tools
  5. Configure a new external tool [as shown](http://images.sev17.com/formatsql.jpg).
  6. To use the new external tool ensure you highlight the text in SQL Server Management Studio you want to format.
It would have been nice to take the T-SQL parsing routine a step further and build an object dependency list from a SQL script, but unfortunately the method/properties for getting to the referenced objects appear to be private. 

The cmdlets will be included in the next release of [SQL Server Powershell Extensions](http://sqlpsx.codeplex.com/). In the meantime I've included compiled cmdlets [here](http://cid-ea42395138308430.skydrive.live.com/self.aspx/Public/SQLParser.zip) until I can get a release packaged up. The source code is available on the [CodePlex project site Source Code section](http://sqlpsx.codeplex.com/SourceControl/ListDownloadableCommits.aspx).

## Comments

**[Chad Miller](#38 "2009-07-13 21:18:00"):** It depends on whether the T-SQL script returns a result set or takes parameters. For a script which does not return a result set as is the case with object creation. You could do this:  
$qry = [System.IO.File]::ReadAllText('C:usersu00documentssmoObjqry2.sql')  
Set-SqlData 'Z002SqlExpress' pubs $qry  
  
For scripts which return a result set use Get-SqlData instead of Set-SqlData. And finally for script which take parameters, embed the script into your Powershell script as a here document. I have an example of this in an SSC article <http://www.sqlservercentral.com/articles/Backup/66564/>

**[paul cassidy](#39 "2009-07-12 21:18:00"):** How can I execute the T-SQL Script in SQLPSX

**[Steven Murawski](#40 "2009-03-06 21:18:00"):** You rock! These are awesome cmdlets.

