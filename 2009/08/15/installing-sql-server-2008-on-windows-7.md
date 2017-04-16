title: Installing SQL Server 2008 on Windows 7
link: http://sev17.com/2009/08/15/installing-sql-server-2008-on-windows-7/
author: Chad Miller
description: 
post_id: 9974
created: 2009/08/15 08:48:00
created_gmt: 2009/08/15 12:48:00
comment_status: open
post_name: installing-sql-server-2008-on-windows-7
status: publish
post_type: post

# Installing SQL Server 2008 on Windows 7

When Installing SQL Server 2008 on Windows 7 RTM, I encountered a couple of issues. [Supposedly you can install SQL Server 2008 RTM on Windows 7](http://sqlblog.com/blogs/aaron_bertrand/archive/2009/08/14/is-sql-server-2008-supported-on-windows-7-windows-server-2008-r2.aspx). You'll receive various warnings about compatibility issues which can be ignored and are fixed with the application of a SQL Server 2008 service pack. So, I attempted to install SQL Server 2008 RTM, this was somewhat successful, the database engine, integration services and reporting services installed, however the various client tools including SQL Server Management Studio did not install. So, I tried to install SQL Server 2008 Service Pack 1 and again received an error. Next I tried adding client tools to SQL Server 2008 RTM -- no luck.

Rather than wade through the SQL Server installation logs, I decided to start fresh by uninstalling SQL Server 2008 and try slipstreaming. The slipstreaming ability is a new feature added to SQL Server 2008 were you can merge the RTM bits with and updated service pack. When you install a slipstreamed SQL Server installation, the service pack is included. I followed the instructions on [this msdn blog](http://blogs.msdn.com/petersad/archive/2009/02/25/sql-server-2008-creating-a-merged-slisptream-drop.aspx), re-ran setup using my new slipstreamed SQL Server 2008 drop and everything installed fine.