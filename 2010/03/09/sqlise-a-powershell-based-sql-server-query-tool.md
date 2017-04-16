title: SQLIse A Powershell Based SQL Server Query Tool
link: http://sev17.com/2010/03/09/sqlise-a-powershell-based-sql-server-query-tool/
author: Chad Miller
description: 
post_id: 10000
created: 2010/03/09 22:31:00
created_gmt: 2010/03/10 02:31:00
comment_status: open
post_name: sqlise-a-powershell-based-sql-server-query-tool
status: publish
post_type: post

# SQLIse A Powershell Based SQL Server Query Tool

[SQL Server Powershell Powershell Extensions](http://sqlpsx.codeplex.com/) (SQLPSX) has been updated to version 2.1. The most notable change is the addition of a Powershell Integrated Scripting Editor (ISE) module called SQLIse (pronounced “SQL Ice”). The module provides a basic IDE for T-SQL that includes the ability to edit, execute, parse and format SQL code from within Powershell ISE. 

### SQLIse Features

  * Offline parsing of T-SQL code
  * Formatting (prettifying) of T-SQL with an extensive customization abilities
  * Comment/Uncomment T-SQL code
  * Uppercase/Lowercase T-SQL code
  * Execute T-SQL code and output to grid, text, text file or CSV file
  * Apply any of the above actions to selections of code by highlighting

### SQLIse Requirements

SQLIse uses the following modules that part of the [CodePlex](http://www.codeplex.com/) project [SQLPSX](http://sqlpsx.codeplex.com/) as well as [PowershellPack](http://code.msdn.microsoft.com/PowerShellPack) available on [MSDN Code Gallery](http://code.msdn.microsoft.com/): 

  * SQLParser (SQLPSX)
  * AdoLib (SQLPSX)
  * IsePack (PowershellPack)
  * WPK (PowershellPack)
_NOTE: The use of external modules is a change for SQLPSX, however sometimes its important to leverage other people’s code to greatly simplify your own. So, in order to use SQLIse you’ll need to install both SQLPSX and the PowershellPack._

### Credits

A big thanks to [Mike Shepard](http://powershellstation.com/) for creating the AdoLib module and James Brundage for his excellent WPK and IsePack modules that make creating GUIs and customizing Powershell ISE seem easy. 

### Next Steps

Check out this [5 minute video of SQLIse](http://www.youtube.com/watch?v=1KcNSHn7oTA) and leave some [feedback on the SQLPSX site](http://sqlpsx.codeplex.com/Thread/View.aspx?ThreadId=204395):

## Comments

**[Chad Miller](#114 "2010-03-10 22:31:00"):** Oracle ISE, I like it! There was some talk of VSDB supporting Oracle, but even if it doesn't you could build the query piece.

**[Bernd Kriszio](#115 "2010-03-10 22:31:00"):** Looks good. Would be nice to extend it to Oracle.

**[Chad Miller](#116 "2010-03-10 22:31:00"):** lol -- Query Analyzer Lives!

**[Aaron Nelson](#117 "2010-03-10 22:31:00"):** This is AWESOME. Looks like you made Buck Woody's wish come true. Query Analyzer Lives!

