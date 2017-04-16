title: SQL Server Powershell Extensions PowerGUI PowerPack
link: http://sev17.com/2009/07/24/sql-server-powershell-extensions-powergui-powerpack/
author: Chad Miller
description: 
post_id: 9969
created: 2009/07/24 11:54:00
created_gmt: 2009/07/24 15:54:00
comment_status: open
post_name: sql-server-powershell-extensions-powergui-powerpack
status: publish
post_type: post

# SQL Server Powershell Extensions PowerGUI PowerPack

If you've been working with Powershell, you most certainly have heard of [PowerGUI](http://www.powergui.org/index.jspa), for those of you who haven't PowerGUI is a free utility provided by Quest. It's both a script editor with PowerShell syntax highlighting and GUI builder for PowerShell scripts. The editor is a very usable standalone component which I recommend to folks looking for a good PowerShell script editor. The other standalone GUI component returns the results of PowerShell commands and scripts in a grid format,  additional actions and links can be created on the objects displayed. PowerGUI  provides a quick method  to create simple GUIs in PowerShell which kind of have the look and feel of MMCs. PowerGUI has a bunch of management packs they call [PowerPacks](http://www.powergui.org/kbcategory.jspa?categoryID=21) that address various technologies including AD, Exchange and SQL Server. All of this is usable out of the box without writing any code.

Should you want to build your own GUIs, there are a couple of ways you can approach PowerGUI development. You could create PowerShell scripts and functions directly in PowerGUI. Alternatively, simply call your functions which have been sourced in your profile. I've choosen to do the later and created a [PowerPack for SQL Server PowerShell Extensions](http://www.powergui.org/entry.jspa?externalID=2442&categoryID=54)[.](http://sqlpsx.codeplex.com/) Here's a few screenshots:

_Server_

![](http://images.sev17.com/PowerGUIServer.jpg)

_Databases_

![](http://images.sev17.com/PowerGUIDatabases.jpg)

_Integration Services_

![](http://images.sev17.com/PowerGUISSIS.jpg)

As a DBA, I can see the potential for using PowerGUI to create administration views for areas outside of Database Administration. For many years I've thought it would be nice to have a scaled down Enterprise Manager or SQL Server Managment Studio tailed for particular job functions such as junior DBAs, computer operators, or login provisioning. With PowerGUI it would trivial for a DBA to create such a view. So, try out PowerGUI yourself, customize it for your needs and leave some [feedback](http://www.powergui.org/entry.jspa?externalID=2442&categoryID=54).