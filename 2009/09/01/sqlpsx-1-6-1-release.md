title: SQLPSX 1.6.1 Release
link: http://sev17.com/2009/09/01/sqlpsx-1-6-1-release/
author: Chad Miller
description: 
post_id: 9978
created: 2009/09/01 21:52:00
created_gmt: 2009/09/02 01:52:00
comment_status: open
post_name: sqlpsx-1-6-1-release
status: publish
post_type: post

# SQLPSX 1.6.1 Release

I completed a maintenance release of [SQL Server PowerShell Extensions](http://sqlpsx.codeplex.com/), which address all known open issues. This release is still PowerShell V1, but I plan on working on a V2 release soon. The only notable change is in LibraryShowMbrs, which is used to recursively enumerate local and AD groups I know Quest makes [a nice set of free cmdlets](http://www.quest.com/powershell/activeroles-server.aspx) that provide this funcitonality. The Quest cmdlets are very good, I use them, however I chose to roll my own for two reasons. First, I did not want to build in a dependency for 3rd party cmdlets. Second, I want to be able to use SQLPSX from within the SQL Server 2008 PowerShell host, which does not support additional cmdlets--only scripts like SQLSPX.

The change in LibraryShowMbrs is in the way used to obtain group members. For several years I've used the WMI class [Win32_GroupUser](http://msdn.microsoft.com/en-us/library/aa394153\(VS.85\).aspx). Until recently this had been a reliable WMI class, however for some reason the class simply stopped returning even the name of sub groups and instead only user accounts are returned. I'm not really sure why and I haven't been able to find documentation on this change. I guess not finding documentation is not too suprising since I doubt many folks use WMI to enumerate local and AD groups. The library was re-written to use [WinNT Provider](http://msdn.microsoft.com/en-us/library/aa772237\(VS.85\).aspx) and I found [this post](http://operatingquadrant.com/2009/08/20/recursively-listing-security-group-members-with-powershell/) from [Kristopher Bash](http://operatingquadrant.com/) helpful in creating the code.

I want to thank [Jorge Seggara](http://sqlchicken.com/) ([@sqlchicken on Twitter](http://twitter.com/sqlchicken)) for helping to find several of the bugs addressed in this release. One of the issues fixed was a missing assembly that I did not find because I was loading it in my profile. I load a lot of things in my profile including initializing the SQL Server 2008 cmdlets and providers in my regular PowerShell using [this script](http://blogs.msdn.com/mwories/archive/2008/06/14/SQL2008_5F00_Powershell.aspx) that contained the assembly I needed to include in one of the SQLPSX scripts. Due to this issue, I've learned a couple of things, first I need to test using the -noprofile switch. This will best mimic a clean PowerShell installation and I would encourage all script developers to do the same with scripts they distribute. Second, I need to do a better job testing. As scripts become complex its time to look at a more displined form of testing. One tool I learned about in [a recent Episode](http://powerscripting.wordpress.com/) of the [PowerScripting Podcast](http://powerscripting.wordpress.com/) called [PSUnit](http://www.psunit.org/). I'm going to check it out.