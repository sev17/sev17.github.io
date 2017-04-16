title: sqlps.exe, the ShellId and Closed Shells
link: http://sev17.com/2008/10/20/sqlps-exe-the-shellid-and-closed-shells/
author: Chad Miller
description: 
post_id: 9919
created: 2008/10/20 19:27:00
created_gmt: 2008/10/20 23:27:00
comment_status: open
post_name: sqlps-exe-the-shellid-and-closed-shells
status: publish
post_type: post

# sqlps.exe, the ShellId and Closed Shells

Jaykul at [huddledmasses](http://huddledmasses.org/) has an interesting post on [Is PowerShell $ShellId too big a burden? ](http://huddledmasses.org/is-powershell-shellid-too-big-a-burden/)which explains why sqlps.exe, the PowerShell host provided by Microsoft in SQL Server 2008, is a closed shell and lacks support for Add-PSSnapin. To see the shellid run $shellid from the PowerShell host. In regular PowerShell the value is Microsoft.PowerShell and in sqlps.exe the value is Microsoft.SqlServer.Management.PowerShell.sqlps. Shellid is readonly and when you create a mini-shell/closed shell like sqlps.exe you implement the shellid as part of  the RunspaceConfiguration. When you implement your own host/shell you create your own RunspaceConfiguration inheriting from the base abstract class. The base class does not have a AddPsSnapIn method and cannot be overriden. Finally no AddPsSnapIn method means you've just created a closed shell! Read the comments section which includes my observation about sqlps.exe to get the full story.

As a result of his post I came away with another useful tip to use both a Microsoft.PowerShell_Profile.ps1 file and a Profile.ps1 file. By doing this you can avoid running incompatible commands such as Add-PSSnapin which is not supported in sqlps.exe. Put the host specific commands for example Add-PSSnapin only in the Microsoft.PowerShell_Profile file and then put the commands which can run in any host in Profile.ps1. You also eliminate the error message: "The term 'Add-PSSnapin' is not recognized as a cmdlet, function, operable program, or script file. Verify the term and try again." you receive launching sqlps when you have both [PowerShell Community Extensions ](http://www.codeplex.com/PowerShellCX)and sqlps.exe installed on the same machine. See [my post](http://www.codeplex.com/PowerShellCX/WorkItem/View.aspx?WorkItemId=18417) in the issue tracking area of PowerShell Community Extensions for more details on this error.