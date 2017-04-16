title: The Truth about SQLPS and PowerShell V2
link: http://sev17.com/2010/05/27/the-truth-about-sqlps-and-powershell-v2/
author: Chad Miller
description: 
post_id: 10223
created: 2010/05/27 21:30:38
created_gmt: 2010/05/28 01:30:38
comment_status: open
post_name: the-truth-about-sqlps-and-powershell-v2
status: publish
post_type: post

# The Truth about SQLPS and PowerShell V2

With the release of SQL Server 2008 R2 there have been claims that sqlps is really PowerShell V1 under the covers or that sqlps is PowerShell V2 because it returns $psversiontable information. Although, technically, sqlps neither PowerShell V1 nor V2, the answer to this question is a little more complicated and a closer look into the inner workings of sqlps is needed. Before we can look into the implementation details of sqlps let’s review how various Microsoft products implement custom shells. 

## Custom Shell Implementation and History

Microsoft products that provide PowerShell integration excluding the sqlps implementation which will cover next; provide a custom shell/console in one of two ways: (Note: profiles are not mentioned because they are not used in package products) 

  1. Use a console file
  2. Call a PowerShell script on startup

### Console Files

A console file is a special XML file with a psc1 extension that stores the configuration information for a PowerShell session including all the custom Snapins loaded. Creating a console file is simple and anyone can do it calling the cmdlet **Export-Console. **You can then use the console file when starting PowerShell by specifying the –PSConsolefile parameter: Powershell.exe –PSConsoleFile _path_to_consolefile_ See [Richard Siddaway’s post on console files](http://richardsiddaway.spaces.live.com/blog/cns!43CFA46A74CF3E96!216.entry) for more information. If you look at the shortcut properties of the following Microsoft products you’ll find the use of console files with the following syntax: 

**_Product_**
**_Syntax_**

_Exchange 2007_
_C:\WINDOWS\system32\WindowsPowerShell\v1.0\PowerShell.exe -PSConsoleFile "C:\Program Files\Microsoft\Exchange Server\bin\exshell.psc1" -noexit -command ". 'C:\Program Files\Microsoft\Exchange Server\bin\Exchange.ps1'"_

_System Center Virtual Machine Manager 2008_
_C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -PSConsoleFile "C:\Program Files\Microsoft System Center Virtual Machine Manager 2008 R2\bin\cli.psc1" -NoExit_

_Operations Manager_
_C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -PSConsoleFile Microsoft.EnterpriseManagement.OperationsManager.ClientShell.Console.psc1 -NoExit .Microsoft.EnterpriseManagement.OperationsManager.ClientShell.Startup.ps1_

### Startup Scripts

PowerShell console files have a couple of limitations first they don’t support modules and second they require starting PowerShell with the PSConsolefile parameter. There are also non-technical issues in the use of console files—many PowerShell users simply don’t like custom consoles and console files don’t lend themselves to reuse in a regular PowerShell console. Looking at Exchange 2010 we see the console file used in Exchange 2007 has been replaced with a startup script. The script is nothing more than a plain old PowerShell script with a ps1 extension. The properties of Exchange 2010 console looks like this: _C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -noexit -command ". 'C:\Program Files\Microsoft\Exchange Server\V14\bin\RemoteExchange.ps1'; Connect-ExchangeServer -auto"_ I’m not sure if other products use startup scripts. There doesn’t seem to be much documentation on best practices for custom console creation, but I suspect because of the limitations of console files we will see more products use startup scripts. 

### SQLPS

This brings us to sqlps, how does SQL Server provide a custom console? Well, sqlps is implemented as something called a minishell. The  sqlps use of a minishell is unique among Microsoft products. In fact sqlps is the only product to develop PowerShell integration in this fashion and will probably be last. The use of a minishell [was not well received upon the initial release of SQL Server 2008](http://blogs.msdn.com/b/powershell/archive/2008/06/23/sql-minishells.aspx). (Note: the intention of this post is not to critique sqlps usage of minishell, but rather to provide historical summary to the reader, so please no flaming sqlps). Unfortunately the release of SQL Server 2008 R2 includes zero changes to sqlps so, we’ll have to look to the next release of SQL Server to get it fixed. In the next section we’ll create our own minishell in order to see demonstrate sqlps is implemented 

### Creating A MiniShell

To really understand how a program works we must delve into the code. _Note: The demonstration that follows uses a deprecated tool called make-shell. This tool should not be used for creating PowerShell solutions as part of a product distributed. The purpose of the sample code is illustrate how sqlps minishell is created._ According to [Michiel Wories](http://blogs.msdn.com/b/mwories/)’ (author of sqlps and creator of SMO) blog post [SQL Server Powershell is here!](http://blogs.msdn.com/b/mwories/archive/2008/06/14/sql2008_5f00_powershell.aspx) sqlps was created via a utility called make-shell. Because make-shell has been deprecated the MSDN documentation has been pulled. A Bing search turns up little other an old post from PowerShell V1 pre-release days which seems to indicate that [make-shell was used in beta builds of PowerShell to create cmdlets](http://bartdesmet.net/blogs/bart/archive/2006/05/07/3943.aspx). The approach demonstrated is even more awkward than the installutil plus add-pssnapin approach to adding cmdlets adopted in V1 and much more complex than the simple import-module approach used in PowerShell V2. We should be thankful make-shell didn’t make into mainstream PowerShell V1 development and how truly simple it is to add cmdlets in PowerShell V2. Although the make-shell tool has largely disappeared from online docs, the utility and documentation are still available as part of the Windows SDK (both 6.0A and [7.0](http://blogs.msdn.com/b/windowssdk/archive/2009/08/07/released-windows-sdk-for-windows-7-and-net-framework-3-5-sp1.aspx) versions). Once you install the SDK (any version higher than 6.0A), you can simply copy the standalone make-shell executable to any machine. We’re now ready to build our own SQL Server minishell. 

#### Setup

First you’ll need to install SQL Server 2008 or SQL Server 2008 R2. The Express edition will suffice. Next we need to setup our build directory. To keep things simple and because the paths are a little tricky, we’ll create a directory called C:makeshell and copy the following files from C:\Program Files\Microsoft SQL Server\100\Tools\Binn, C:\Windows\System32\WindowsPowerShell\v1.0 and C:\Program Files\Microsoft SDKs\Windows\v7.0\Bin. The contents of C:\makeshell should contain: en<DIR>* Certificate.format.ps1xml DotNetTypes.format.ps1xml FileSystem.format.ps1xml Help.format.ps1xml make-shell.exe Microsoft.SqlServer.Management.PSProvider.dll Microsoft.SqlServer.Management.PSSnapins.dll PowerShellCore.format.ps1xml PowerShellTrace.format.ps1xml Registry.format.ps1xml SQLProvider.Format.ps1xml SQLProvider.Types.ps1xml types.ps1xml *The en directory contains the MAML help file for the SQL Server provider and cmdlets (Microsoft.SqlServer.Management.PSProvider.dll-Help.xml and Microsoft.SqlServer.Management.PSSnapins.dll-Help.xml)

## Comments

**[Chad Miller](#153 "2010-06-14 07:41:30"):** Correction SQL 2008 R2 includes two new folders under SQLSERVER: for DAC and UCP. The other pieces of sqlps remain unchanged from SQL Server 2008.

**[Corwin](#343 "2013-07-11 13:30:42"):** Something interesting I ran into, SQL Server 2008 installs SQLPS v10 and SQL 2012 installs SQLPS v11. Installing SQL Server 2012 did not seem to upgrade my SQLPS to v11. SQLPS v11 seems to handle off of the CMDLETS that I had problems with in v10.

**[Chad Miller](#344 "2013-07-11 14:34:44"):** SQL Server 2012 and higher implements sqlps differently (now a regular Powershell host) than described in this blog post which pertains to SQL Server 2008/2008 R2. I blogged about sqlps 2012 here: http://sev17.com/2011/07/18/denali-sqlps-first-impressions/ One way to check if you're using 2008/2008 R2 sqlps.exe or 2012 sqlps.exe is to look at the $shellid In 2008/20082 it will be Microsoft.SqlServer.Management.PowerShell.sqlps in 2012 it will be Microsoft.PowerShell

