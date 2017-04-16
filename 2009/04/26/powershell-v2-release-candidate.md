title: Powershell V2 Release Candidate
link: http://sev17.com/2009/04/26/powershell-v2-release-candidate/
author: Chad Miller
description: 
post_id: 9954
created: 2009/04/26 11:40:00
created_gmt: 2009/04/26 15:40:00
comment_status: open
post_name: powershell-v2-release-candidate
status: publish
post_type: post

# Powershell V2 Release Candidate

The big news this week has been [the announcement of the Windows 7 Release Candidate availability on April 30th](http://windowsteamblog.com/blogs/windows7/archive/2009/04/24/windows-7-release-candidate-update.aspx?PageIndex=3). Upon seeing the [tweet](http://twitter.com/jamesoneill/statuses/1608708696) on Friday, April 24th, my first thoughts were of excitement and a bit of panic and not for the reasons you would typically think. The announcement is especially important to Powershell users because technically  this is also the first release candidate of Powershell V2. For me it's not about the upgrade from Vista to Windows 7, it is the upgrade from Powershell V1 to V2 that I'm focused on. I'm excited about all the new features in Powershell V2, but at the same time a little concerned that I haven't started using it yet.

Why haven't I started using Powershell V2 in CTP release? Well, part of my job is providing production support to several thousand SQL Server databases. I have Powershell V1 installed on SQL Server 2000, 2005 and 2008 machines all running Windows 2003 Server and my team also uses Powershell V1 on their Windows XP and Vista workstations. Running CTP or beta versions of Powershell just isn't an option for production support. That's not to say I haven't participated in various accelerated adoption programs around SQL Server. I used SQL Server 2008 a full 6 months prior to the release; however Microsoft rightly so, does not provide production support until after a product releases and the accelerated adoption programs are restricted to one or two systems.

Aside from my day job, I develop a CodePlex project called [SQL Server Powershell Extensions](http://www.codeplex.com/SQLPSX) built entirely with Powershell. As a tool developer creating a packaged product, I should be more bleeding edge then a production support role and look at new technology changes prior to general availability. There are many features I'm very excited about in Powershell V2 that I can't wait to apply to SQL Server Powershell Extensions. This week's announcement means I've got to start now and that causes a little panic when I think about all I need to do.

How much time do I have to get ready for Powershell V2? The Release Candidate status moves Powershell V2 one step closer to production usage, but how much closer? At this point general availability release dates are mere speculation, however based on the [Microsoft Watch blog post](http://www.microsoft-watch.com/content/windows_7/windows_7_rc_coming_around_april_30.html), it might be as early as June 30th. The Windows 7 release still [means several months until Powershell V2 is available outside of Windows 7 or Windows 2008 R2](http://tfl09.blogspot.com/2008/11/powershell-v2-release-imescales.html). So, my guess, best case we are looking right around the ***Updated 3 year release anniversary of Powershell V1 ***, September 30th. Production Windows 2003 or Windows XP/Vista workstations can then use Powershell V2. That gives me a little more time before upgrading production to Powershell V2, but not much time to get SQL Server Powershell Extensions ready. The next several months are going to be busy.

## Comments

**[Bernd Kriszio](#69 "2009-05-03 11:40:00"):** I hope you got to loading LibrarySmo.ps1 into ISE (perhaps using drag and drop), loaded it by pressing F5 or the green arrow in the editor pane. Finally typing Get-Command *et-Sql* | Select Name in the command pane produces a well known output.   
The graphical editor ISE is one off the important new features, advanced functions and modules are the other.

**[Chad Miller](#70 "2009-04-26 11:40:00"):** Corrected--Thanks

**[Richard Siddaway](#71 "2009-04-26 11:40:00"):** PowerShell v1 was released in November 2006 at Barcelona IT Forum

