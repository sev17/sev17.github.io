title: Scripting Linked Servers
link: http://sev17.com/2010/07/06/scripting-linked-servers/
author: Chad Miller
description: 
post_id: 10370
created: 2010/07/06 12:40:41
created_gmt: 2010/07/06 16:40:41
comment_status: open
post_name: scripting-linked-servers
status: publish
post_type: post

# Scripting Linked Servers

A co-worked asked me to look at a [T-SQL script I had written 8 years ago for scripting out linked servers and linked server logins](http://www.sqlservercentral.com/scripts/Miscellaneous/30620/) from SQL Server. The script wasn't working as expected. I hadn't seen the code in some time, but looking at it now the fact that it did not work wasn't surprising as the code uses some of bad practices like directly querying system tables which I've [discouraged in previous posts](/2010/02/the-t-sql-hammer/). So rather than fix the T-SQL script, I fired up sqlps PowerShell host and in 5 minutes I had a much simpler and working one-line PowerShell command: From SQL Server 2008 SSMS navigate to the Linked Servers folder in Object Explorer, right-click and start sqlps (PowerShell): **Script out Linked Servers and logins**
    
    
    PS SQLSERVER:SQLZ002SQL1LinkedServers get-childitem | %{$_.Script()}

A few things struck me about the PowerShell solution: 

  * This isn't something that can be done from the GUI (SQL Server Management Studio). The functionality simply isn't there nor should it be. Some advanced things are easier and better exposed in PowerShell than the GUI
  * PowerShell is certainly much easier than my old T-SQL solution.
  * Unlike the T-SQL solution, I doubt I will be asked to fix this simple PowerShell command in another 8 years.

## Comments

**[John Eisbrener](#173 "2011-08-19 12:59:05"):** Thanks Chad, this is exactly what I was looking for and it works great if I'm actively in the sqlps utility. I do have one question for you though. Is there a way to pass this to sqlps as a single-line command? I'm trying the following: sqlps -Command "}" but I receive this error: An expression was expected after '('. At line:1 char:64 \+ {cd SQL/ServerN/DEFAULT/LinkedServers;get-children | %{.Script( <<<< )};} \+ CategoryInfo : ParserError: (:) [], ParentContainsErrorRecordException \+ FullyQualifiedErrorId : ExpectedExpression Any help or insight would be appreciated. -John

**[Chad Miller](#174 "2011-08-20 11:05:32"):** I can get it to work by using the call operator & and double quotes as follows: `C:Usersu00>sqlps -command "&{cd SQLSERVER:SQLZ001SQL1LinkedServers; get-childitem | %{$_.Script()}}"` If you take a look a powershell /? you'll see several examples of passing a command to powershell.exe including the one I used above. Sqlps is missing the same level of detail help in its help when looking at sqlps /?, but it works.

**[John Eisbrener](#175 "2011-08-24 14:54:35"):** It's a shame the comments are cutting off the syntax, as that's the meat of the answer. I've started a thread over on the technet forums (http://social.technet.microsoft.com/Forums/en-US/sqltools/thread/3db76280-713c-4520-823a-a91357bceb9e), so hopefully you wouldn't mind posting the answer there as well. Thanks!

**[Chad Miller](#176 "2011-08-24 15:51:11"):** Ahh, just noticed that myself. Amperstand is special character with HTML. I'll reply to the forum thread and see I can get the comment to show correctly here.

