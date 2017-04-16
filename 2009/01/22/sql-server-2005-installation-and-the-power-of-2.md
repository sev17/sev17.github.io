title: SQL Server 2005 Installation and the Power of 2
link: http://sev17.com/2009/01/22/sql-server-2005-installation-and-the-power-of-2/
author: Chad Miller
description: 
post_id: 9936
created: 2009/01/22 13:19:00
created_gmt: 2009/01/22 17:19:00
comment_status: open
post_name: sql-server-2005-installation-and-the-power-of-2
status: publish
post_type: post

# SQL Server 2005 Installation and the Power of 2

My colleague John O'Shea ran into an unusual issue installing SQL Server 2005 on 4-socket, 6-core machines, which I thought I would share...

We've started using [DL 580 G5 4 socket 6-core ](http://h10010.www1.hp.com/wwpc/us/en/sm/WF06a/15351-15351-3328412-241644-3328422-3454575.html#)machines and deployed a few of these boxes running SQL Server 2008 with 64 GB of memory. We recently encountered a problem installing SQL Server 2005 on the 4-way 6-cores. Apparently the SQL Server 2005 installer verifies the number of processors is a power of 2. SQL Server 2008 does not do this. A 4-socket 6-core is 24 which is not a power of 2. We've been using SQL Server 2005 for nearly four years and this is the first model server we've encountered the issue. After reading through this [KB article ](http://support.microsoft.com/kb/954835/en-us)and opening a call with Premier we were able to workaround the issue by following the workaround documented in the KB article for Windows 2003-- edit boot.ini setting "**/NUMPROC= 1**"(sets Windows to a single processor), and then installing SQL Server 2005. The KB article stops there, but the next thing you need to do is install SQL Server 2005 Service Pack 2 or later and then you can remove the NUMPROC entry from boot.ini so all processors are recognized. The workaround works for both clustered and standalone machines alike.