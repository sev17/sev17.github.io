title: Disable Windows Server 2003 Scalable Networking Pack
link: http://sev17.com/2008/12/18/disable-windows-server-2003-scalable-networking-pack/
author: Chad Miller
description: 
post_id: 9932
created: 2008/12/18 20:46:00
created_gmt: 2008/12/19 00:46:00
comment_status: open
post_name: disable-windows-server-2003-scalable-networking-pack
status: publish
post_type: post

# Disable Windows Server 2003 Scalable Networking Pack

For the last several months I've been having Windows 2003 Server Pack 2 applied to all SQL Server in my envronment in order to bring them inline with a single build and have ran into an issue with Windows Server 2003 Scalable Network Pack (SNP). Windows 2003 SP2 introduces the "helpful feature" of the TCP Chimney Offload Engine (TOE.) The TOE feature is supposed to increase performance by allowing some TCP functionality to be handled by the network driver/adapter instead of the Windows TCP/IP stack itself.  This functionality is enabled by default in Windows 2003 Service Pack 2.

Unfortunately several systems I support had quite the opposite effect and I noticed issues of network latency, unable to connect and high paging. Several people have blogged about SNP/TOE issues in SQL Server and Exchange environments. I even saw a tip  to disable TOE in the PASS 2008 Summit session, "SQL Server Tips, Tricks & Techniques from Inside Microsoft ," I haven't seen the issue on every SQL Server just some of the more active ones. Once TOE was disabled the problems went away.

**Lession learned -- You should always disable TOE**. The risks in not doing so far outweigh any potential benefits. The easiest way to disable TOE (or more accurately the problematic features in TOE) is to immediately apply Hotfix [948496](http://support.microsoft.com/kb/948496) after installing Windows 2003 Service Pack 2.

## Comments

**[Chad Miller](#13 "2008-09-17 20:46:00"):** I can't say seen the issues after applying the QFE 948496 and disabling through HP utility. One suggestion check the registry key just to ensure TOE/RSS is disable. This post uses a Powershell script to check registry keys and can be executed from your workstation:  
[http://sev17.com/2009/02/check-for-windows-server-2003-scalable-networking-pack/](http://chadwickmiller.spaces.live.com/blog/cns!EA42395138308430!283.entry)

**[Andrew Catchpole](#14 "2008-09-17 20:46:00"):** I currently have the same problem on 6 HP servers with NC373i Gigabiut NICS. If you run netstat -s the number of TCP segment retranmissions start to increase over time up to 5% - this is significant for the application! Ping is alway 100% so the network is good.  
  
I have run all the patches from HP and turned off TOE and RSS on the HP Network tool AND in Windows but I still have the same problem - TCP retransmissions!!!  
  
It does seem to be all servers with SQL and/or multiple remote client and/or multiple internal connections - Servers without SQL or loading don't have this problem.  
  
Is your problem cured? do you have TCP problems  
  
Has anyone else got this problem???

**[Chad Miller](#15 "2008-01-12 20:46:00"):** Even after applying hotfix 948496, we've still noticed problems with some servers. TOE still shows as being enabled in the HP NCU (Network Configuration Utility) while RSS shows as disabled. But RSS still shows as enabled in the Windows Network Configuration NIC properties. To be safe we disabled both TOE and RSS in both the HP and Windows utilities:  
In NCU -- Go to Advanced => Receive Side Scaling => Set to Disabled  
In Windows Network Configuration NIC properities -- Go to Advanced Setting => TCP Offload Engine => Uncheck Enabled

