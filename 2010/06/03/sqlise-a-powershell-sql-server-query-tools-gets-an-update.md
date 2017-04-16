title: SQLIse A PowerShell SQL Server Query Tools Gets an Update
link: http://sev17.com/2010/06/03/sqlise-a-powershell-sql-server-query-tools-gets-an-update/
author: Chad Miller
description: 
post_id: 10317
created: 2010/06/03 23:04:06
created_gmt: 2010/06/04 03:04:06
comment_status: open
post_name: sqlise-a-powershell-sql-server-query-tools-gets-an-update
status: publish
post_type: post

# SQLIse A PowerShell SQL Server Query Tools Gets an Update

After the first release of SQLIse (“SQL Ice”)--a basic IDE for T-SQL that includes ability to edit, execute, parse, format SQL code from within PowerShell ISE, several people contacted me to help add new features. The result is SQL Server PowerShell Extensions (SQLPSX) 2.2 with many enhancements to SQLIse… 

### SQLIse Features

  * Offline parsing of T-SQL code
  * Formatting (prettifying) of T-SQL with an extensive customization abilities
  * Comment/Uncomment T-SQL code
  * Uppercase/Lowercase T-SQL code
  * Execute T-SQL code and output to grid, text, text file or CSV file
  * Apply any of the above actions to selections of code by highlighting
  * Table and object browsers*
  * PoshMode (like sqlcmdmode)*
  * Save output to variable*
  * Save and manage connections*
  * Tab Expansion of schemas, tables, views, functions, procedures, columns and parameters.*
  * OracleIse module*
  * OracleClient module used by OracleIse*
  * Redistributed WPK and ISECreamBasic modules from PowerShellPack and ISECream projects*
  * Created an SQLPSX installer*
_* = New features for release 2.2_

### Credits

My thanks to 

  * [Steve Murawski](http://blog.usepowershell.com/) for his work creating most of the new GUIs (table/object browsers, saved connections, and output to variable).
  * [Bernd Kriszio](http://pauerschell.blogspot.com/) for his work in adding a slimmed down ISE menu module, IceCreamBasic, all of the Oracle components and his willingness to test and provide feedback to the rest us
  * [Max Trinidad](http://max-pit.spaces.live.com/) put a lot of work into creating an installer for SQLPSX built entirely in PowerShell. Max was very patient in our requests to build several interactions of the installer as we worked through various scenarios.

### Next Steps

  * Download [SQLPSX](http://sqlpsx.codeplex.com/)
  * Check out the [SQLIse screen shots](http://sqlpsx.codeplex.com/Project/Download/FileDownload.aspx?DownloadId=124413)
  * Watch the 10 minute [SQLIse video](http://www.youtube.com/watch?v=8WoZ5aUsNwM)
httpv://www.youtube.com/watch?v=8WoZ5aUsNwM

## Comments

**[Greg Duncan](#158 "2010-06-04 12:06:55"):** I think the final link on the page is jacked?

**[Chad Miller](#159 "2010-06-04 12:31:10"):** ahh, Trying to use SmartYouTube WP plugin for first time. I put the tag in the wrong place. I think its fixed now.

**[Amit Agarwal](#160 "2010-06-06 06:08:32"):** **SQuirrel SQL for Graphical interface to Oracle/MySQL and loads of other database -- OSS and free....** I found your entry interesting thus I've added a Trackback to it on my weblog :)...

**[Nick Wber](#161 "2010-06-14 12:27:39"):** I'm trying to re-create your demo in the video but having issues with it. Below is what i have. I enable PoshMode and output to Grib but I keep getting the follow message. ###################ERROR##### PS C:#[CONNECTED][dcsql8v100vm.master]: > $m = gmo SQLIse; & $m Invoke-ExecuteSql Exception calling "Fill" with "1" argument(s): "Incorrect syntax near '#'. Incorrect syntax near '1.0'." At C:UsersnwrDocumentsWindowsPowerShellModulesSQLPSXModulesadolibadolib.psm1:334 char:10 \+ $da.fill <<<< ($ds) | Out-Null \+ CategoryInfo : NotSpecified: (:) [], MethodInvocationException \+ FullyQualifiedErrorId : DotNetMethodException ##############SCRIPT######### !!setvar dirlist (dir | select name, lastwritetime, length | ConvertTo-Xml -NoTypeInformation -As string) declare @doc xml set @doc = '$dirList' CREATE TABLE #dirlist (filename varchar (128), LastWriteTime datetime, FileSize int) insert #dirlist SELECT item.ref.value('(Property/text())[1]', 'nvarchar(128)') AS Name ,item.ref.value('(Property/text())[2]', 'nvarchar(128)') AS LastWritetime ,item.ref.value('(Property/text())[3]', 'nvarchar(128)') AS Length FROM (SELECT @doc AS feedXml) feeds(feedXml) CROSS APPLY feedXml.nodes('/Objects/Object') AS item(ref) SELECT * FROM #dirlist What am I doing wrong?

**[Chad Miller](#162 "2010-06-14 13:11:13"):** Found a bug with PoshMode after we released 2.2.2. The PoshMode feature requires Beta version 2.2.3 or higher http://sqlpsx.codeplex.com/releases/view/46399 This version is not the default download. Are using the 2.2.3 version?

**[Nick Wber](#163 "2010-06-14 14:29:52"):** Just upgraded to the beta 2.2.3 and still getting the same error as above.

**[Chad Miller](#164 "2010-06-14 17:02:41"):** Hmm, I just re-downloaded 2.2.3 and copied/pasted and ran your script without issue. Can you verify 2.2.3, you should see this on line 33 of SQLIse.psm1 file in the SQLIse directory: Set-Alias Expand-String $psScriptRootExpand-String.ps1 Running this in the command-plane $options.PoshMode should return true. One other option, just to verify SQL (must be SQL 2005 or higher): From command pane run this after the dirList varaible is set (note clip is only available in Vista or higher OS): $dirList | clip paste the XML in SSMS and enclose in single quotes. You should be able to run the query in SSMS: declare @doc xml set @doc = '' CREATE TABLE #dirlist (filename varchar (128), LastWriteTime datetime, FileSize int) insert #dirlist SELECT item.ref.value('(Property/text())[1]', 'nvarchar(128)') AS Name ,item.ref.value('(Property/text())[2]', 'nvarchar(128)') AS LastWritetime ,item.ref.value('(Property/text())[3]', 'nvarchar(128)') AS Length FROM (SELECT @doc AS feedXml) feeds(feedXml) CROSS APPLY feedXml.nodes('/Objects/Object') AS item(ref) SELECT * FROM #dirlist

**[Nick Weber](#165 "2010-06-14 23:21:40"):** After a few more tests I decided to test it on another Dev VM and it worked. :) I'm not sure why my main laptop is throwing a error the only difference is that it has SMO for SQL 2008 R2. Thanks for your help!!! Nick Weber @Toshana

