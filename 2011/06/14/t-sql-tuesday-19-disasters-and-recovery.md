title: T-SQL Tuesday 19 Disasters and Recovery
link: http://sev17.com/2011/06/14/t-sql-tuesday-19-disasters-and-recovery/
author: Chad Miller
description: 
post_id: 10667
created: 2011/06/14 08:24:29
created_gmt: 2011/06/14 12:24:29
comment_status: open
post_name: t-sql-tuesday-19-disasters-and-recovery
status: publish
post_type: post

# T-SQL Tuesday 19 Disasters and Recovery

![TSQL2sDay150x150](http://images.sev17.com/TSQL2sDay150x150.jpg) This post is my contribution to [T-SQL Tuesday](http://tsql2sday.com/), hosted this month by Allen Kin ([blog](http://www.allenkinsel.com/) | [twitter](http://twitter.com/sqlinsaneo)). A first step in any disaster recovery planning is inventorying your database servers. Although having having an up-to-date list of SQL Servers sounds simple enough the reality is in an enterprise environment SQL Servers are bit like mushrooms—they suddenly crop up and when a server is no longer needed they are taken down. A secondary issue to deal with is verifying firewall settings. In an enterprise environment or really in any company that cares about security, internal layered data center firewalls are used between networks, web servers and user segments. The addition of layered firewalls has added a new troubleshooting requirement for DBAs--not only ensuring a server is up, but also verifying various machines can connect to needed endpoints. The point is once you inventory and collect your list of SQL Servers you need to check they are still alive and various services can be reached by remote machines. Once you find servers which cannot be connected to, then its time to research why. Has the server been decommissioned? Is it connection issue? And finally if the server has purposely been taken down then we need to remove it from inventory. What we need is a dynamic inventory and a way to automatically check endpoints... 

## Adding Servers to Your Inventory

In order to handle the inventory piece we can leverage [Central Management Server](http://msdn.microsoft.com/en-us/library/bb895144.aspx) (CMS). I’ve previously blogged about several ways you can use PowerShell and T-SQL to dynamically add servers to your CMS inventory and even update your RDCMan list: 

  * [Dynamically Register SQL Instances in SQL Server Management Studio 2008](/2009/01/dynamically-register-sql-instances-in-sql-server-management-studio-2008/)
  * [Building A Remote Desktop Connection Manager List](/2010/06/building-a-remote-desktop-manager-connection-list/)
  * [Building A Remote Desktop Manager Cluster List](/2010/06/building-a-remote-desktop-manager-cluster-list/)
The post on ﻿dynamically ﻿﻿﻿registering SQL Servers is interesting in that I'm leveraging System Center to find new SQL Servers and add them to a LostAndFound CMS  group. I'll then manually go through and move the servers to their appropriate groups. There also are many other uses for a CMS, you can even use CMS list to drive your Policy-Based Management collection as described here: [PBM and PowerShell](/2010/12/pbm-and-powershell/)

## Verifying Servers in Your Inventory

As for verifying connectivity and checking whether the server is still alive and connect through various firewalls, I’ve a new script called [Test-SqlConnection.ps1](http://poshcode.org/2732) and posted to [PoshCode](http://poshcode.org/). The script executes against the CMS list and can perform several tests: 

  * Ping
  * WMI
  * SMB
  * Port
  * SQL
  * Database
  * SSIS
  * Agent
Aside from Database and Agent tests each checks for connectivity by making a connection to the service, pinging an IP or connecting to a port. A simple colorized HTML report is produced (see sample output below) for each test. 

## Getting Started

  1. Download [Test-SqlConnection](http://poshcode.org/2732) and save as Test-SqlConnection.ps1
  2. Change the script level variable $Script:CMServer to your CMS name
  3. Source the functions
    
    
    . ./Test-SqlConnection.ps1

## Running Tests

Execute tests as follows: 
    
    
    Test-Main “Ping”
    Test-Main “WMI”
    Test-Main “SMB”
    Test-Main “SQL”
    Test-Main “Database”
    Test-Main “Port”
    Test-Main “SSIS”
    Test-Main “Agent”

Sample output: 

Args Message Result Test

sql1
 
True
Test-SQL

sql2
 
True
Test-SQL

sql3
Exception calling "Open" with "0" argument(s): "A network-related or instance-specific error occurred ...
False
Test-SQL
Testing a specific CMS group is supported. In this example I'm testing all servers in a CMS group named prod: 
    
    
    Test-Main "SQL" "prod"

You can also test a single server or list of servers using the individual functions without using a CMS: 
    
    
    Test-SQL 'Z003SQL1'
    get-content ./servers.txt | Test-SQL
    Test-SQL 'Z003SQL1','Z002SQL2'

Additional sample output can be found [here](http://cid-ea42395138308430.office.live.com/self.aspx/Public/Blog/Test-SqlConnection.zip). Failed connections should be researched and servers no longer online should be removed from inventory. Because there's a bit of research involved, I choose to remove servers manually.