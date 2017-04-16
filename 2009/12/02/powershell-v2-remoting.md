title: PowerShell V2 Remoting
link: http://sev17.com/2009/12/02/powershell-v2-remoting/
author: Chad Miller
description: 
post_id: 9989
created: 2009/12/02 18:05:00
created_gmt: 2009/12/02 22:05:00
comment_status: open
post_name: powershell-v2-remoting
status: publish
post_type: post

# PowerShell V2 Remoting

To use remoting you'll need to install Powershell V2, unless you're running Windows 2008 R2 or Windows 7, you'll need to grab [the latest version of the Windows Management Framework](http://support.microsoft.com/kb/968929). [The Framework includes PowerShell V2 + WINRM.](http://blogs.msdn.com/powershell/archive/2009/11/20/windows-powershell-and-the-windows-management-framework.aspx) What's [WINRM](http://msdn.microsoft.com/en-us/library/aa384426\(VS.85\).aspx)? It's the service included in Windows 2008 and Vista that provides the remoting infrastructue used by PowerShell V2. The installation simply updates the binaries to Windows 2008 R2 and Windows 7 levels. The other thing to note is that Windows 2003 and XP are also now supported. Make sure you have V2 on both your server and client. From a security standpoint, remoting is disabled by default, requires an elevated administrator session to enable and once enabled only allows administrators to connect by default. If you're interested in security I suggest you reading about internals of WINRM and the underlying protocol, WS-MAN. In my environment I use SCCM to push out Microsoft updates to servers and since PowerShell V2 is available as a [Windows Update](http://support.microsoft.com/kb/968929), SCCM can push out the PowerShell V2. _Note: Unlike PowerShell V1, V2 requests a reboot, so schedule accordingly. _

In PowerShell V1 and V2 some cmdlets and .NET classess have always supported the concept of remote connections. For instance Get-WMIObject takes a computername parameter that does not rely on the new remoting infrastructure; when using the SMO Server class you can specify a remote SQL Server instance; and the same is true with ADO.NET. There are however cases where command-line programs don't provide native support for remoting. In these situations the remoting capabilities of PowerShell V2 is going to be very useful.

Before setting up remoting, like you I searched the internet looking for blogs, article and documentation. Unfortunately I wasn't that lucky when it comes to to finding PowerShell remoting topics. There are blog posts related to WS-MAN, WINRM, WINRS, the WS-MAN provider with different fringe use cases some which will lead you down a rabbit hole describing how to set up remoting in some obscure non-PowerShell, CTP or non-default manner. The only thing I want to do is enable plain vanilla remoting between domain attached computers.

This is unfortunate because setting up remoting is, as we will soon see, very simple. What's surprising is where I found the best documentation on remoting and PowerShell in general, my own PowerShell console using** Get-Help.** This was one of those duh moments.  We know that PowerShell has a verb-noun naming convention and you can discover commands based on naming convention. You can then  use **get-command **to see what you might be looking for and then use **get-help** to view the documentation. But, what if you're not sure of the command to execute and have a question on concepts? That's when you should, take a look the about_* topics. For remoting specifically look at **get-help about_remote_FAQ **(to see all about topics run **get-help about_***):

Reading through **about_remote_FAQ** you'll see a heading entiled _"HOW TO CONFIGURE YOUR COMPUTER FOR REMOTING"_, which is exactly what I was looking for.

From the server computer, run (**NOTE: You only have to run this command one time to enable remoting. It must be run from an elevated prompt):**

PS C:> **enable-psremoting**

_You should see the following output_

WinRM Quick Configuration Running command "Set-WSManQuickConfig" to enable this machine **for** remote management through WinRM service. This includes: 1. Starting **or** restarting (**if** already started) the WinRM service 2. Setting the WinRM service type to auto start 3. Creating a listener to accept requests on any IP address 4. Enabling firewall exception **for** **WS-Management** traffic (**for** http only). **Do** you want to **continue**? **[Y]** Yes  **[A]** Yes to All  **[N]** No  **[L]** No to All  **[S]** Suspend  [?] Help (**default** **is** "Y"):Y WinRM already **is** set up to receive requests on this machine. WinRM has been updated **for** remote management. Created a WinRM listener on [HTTP://*](http://*/) to accept **WS-Man** requests to any IP on thi s machine. 

_Then execute the following command to test remoting (this will connect to the local host): _PS C:> **new-pssession**

_You should see the following, showing an open local connection:_

Id Name            ComputerName    State    ConfigurationName     Availability \-- ----            ------------    -----    -----------------     ------------ 1 Session1        localhost       Opened   Microsoft.PowerShell     Available 

Having completed the server-side setup, next take a look at **get-help about_remote**. This document will walk you through the three main remoting scenarios of interactive session, remote command, commands in a session:

_**From the client machine (i.e. your workstation), start an interactive session with the server, in this case Z002  is the remote server.**_

** **

PS C:> **enter-pssession** Z002 **[Z002]**: PS C:> $env:computername Z002

**When finish close the remote connection:** **[Z002]**: PS C:> **Exit-PSSession** _**To execute a non-interactive remote command use the invoke-command cmdlet, specifying a remote server:**_