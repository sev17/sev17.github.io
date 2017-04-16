title: Registering SQL Servers in 2000 EM, 2005 SSMS, and 2008 SSMS
link: http://sev17.com/2008/12/17/registering-sql-servers-in-2000-em-2005-ssms-and-2008-ssms/
author: Chad Miller
description: 
post_id: 9931
created: 2008/12/17 16:25:00
created_gmt: 2008/12/17 20:25:00
comment_status: open
post_name: registering-sql-servers-in-2000-em-2005-ssms-and-2008-ssms
status: publish
post_type: post

# Registering SQL Servers in 2000 EM, 2005 SSMS, and 2008 SSMS

I support hundreds of SQL Servers and rather than manually registering SQL instances through the GUI I prefer to register SQL instances programmatically for each version of SQL Server. This is particuarlly true with SQL Server 2000 which offers no built-in way to import/export server registrations. SQL Server 2005/2008 introduced the capability to import/export an XML .regsrvr file, so I find less of a need. However being able to programmatically register SQL instances is still important, since you need to have one person register all SQL instances in order to export the registrations to share with other DBAs. SQL Server 2008 offers a new feature called "Central Management Server" that allows you to centrally register all of your SQL Servers and then as an administrator you would connect your SSMS to the Central Management Server to obtain the server registrations. Since SSMS 2008 and SSMS 2005 support registering lower version of SQL Server I'll probably find less of a need to maintain server registrations in older tools, but small areas of incompabilities in lower versions of SQL Server with higher versions of the management tool has meant maintaining three tools (2000, 2005, 2008). The following provides Powershell code and examples for registering servers in 2000 Enterprise Manager and 2005/2008 SQL Server Management Studio...

## SQL Server Management Studio (SSMS) 2008

The SQL Server 2008 Powershell provider includes new-item functionality which easily allows you to create server groups and server registration using a directory/file analogy. Microsoft makes registering SQL Servers in SSMS 2008 via Powershell very simple. Thank You Microsoft! 

  1. Start the SQL Server 2008 Powershell host, SQLPS.exe
  2. **CD 'SQLSERVER:sqlregistrationDatabase Engine Server Group'**
  3. **new-item MyGroup** \-- (Creates new server group)
  4. CD **MyGroup**
  5. **New-Item $(Encode-Sqlname MyServer) -itemtype registration -Value "server=MyServer;integrated security=true" **\-- (registers new server)
_**UPDATE: Use Encode-Sqlname cmdlet to deal with named instances, since adding Encode-Sqlname doesn't effect default instances use it all the time.** _ **new-item $(Encode-Sqlname MyServer) -itemtype registration -Value "server=MyServer;integrated security=true"**

## SQL Server Management Studio (SSMS) 2005

SQL Server 2005 uses the .NET Microsoft.SqlServer.Management.Smo.RegisteredServers classes. Save the following code as register2005.ps1: **[reflection.assembly]::Load("Microsoft.SqlServer.Smo, Version=9.0.242.0, Culture=neutral, PublicKeyToken= 89845dcd8080cc91") > $null** **####################### function Add-SSMSServerGroup { param($groupname)** **$ServerGroup = new-object ("Microsoft.SqlServer.Management.Smo.RegisteredServers.ServerGroup") "$groupname" $ServerGroup.Create()** **return $ServerGroup }** **####################### function Add-SSMSRegisteredServer { param($ServerGroup, $servername)** $server = new-object ("Microsoft.SqlServer.Management.Smo.RegisteredServers.RegisteredServer") $ServerGroup, "$servername" $server.ServerInstance = "$servername" $server.LoginSecure = $true $server.Create() } To use the functions, source the file: **. ./register2005.ps1** Here is an example which registers a list of SQL Server from a text file, servers.txt under a new group called "All". Keep in mind unlike SQL 2000 in 2005/2008 you can register a server multiple times under different groups. You can even nest groups. **$ServerGroup = Add-ServerGroup "All" Get-Content ./servers.txt |foreach { Add-SSMSRegisteredServer $ServerGroup $_ } **

## SQL Server Enterprise Manager (EM) 2000

_I orginally published a script entitled __[Import/Export SQL Server 2000 Enterprise Manager Registered Servers_](http://www.sqlservercentral.com/scripts/powershell/62275/)_ on __[SQLServerCentral_](http://www.sqlservercentral.com/)_. The script/notes are reposted below._ _Note: Prior to Powershell I used a VBScript/Perl version of the script._ Imports or exports SQL Server 2000 Enterprise Manager group and server registrations. Note: SQL Server 2000 Enterprise Manager should be installed and then launched to initialize the registry keys. Close Enterprise Manager prior to running script. To use you'll need to source the Powershell Script. Save the code below as sqlem.ps1 **$global:appl = New-Object -comobject "SQLDMO.Application"** **####################### function Export-RegisteredServers { param($file)** **#Dumps current SQL Server Enterprise Manager Groups and servers to specific $appl.ServerGroups | %{$group = $_.Name; $_.RegisteredServers | % {$group + " " + $_.Name}} >> $file** **}** **####################### function Add-ServerGroup { param($groupname)** **$ServerGroup = New-Object -comobject "SQLDMO.ServerGroup" $ServerGroup.Name = $groupname trap {$script:Exception = $_; continue} $appl.ServerGroups.Add($ServerGroup) $ServerGroup = $appl.ServerGroups.Item($groupname)** **return $ServerGroup** **}** **####################### function Add-RegisteredServer { param($ServerGroup, $servername)** **$RegisteredServer = New-Object -comobject "SQLDMO.RegisteredServer" $RegisteredServer.Name = $servername $RegisteredServer.UseTrustedConnection = 1 $RegisteredServer.PersistFlags = 1 trap {$script:Exception = $_; continue} $ServerGroup.RegisteredServers.Add($RegisteredServer)** **}** **####################### function Import-RegisteredServers { param($file)** **#Registers each group/server in enterprise manager based on file generated from write-registeredServers function** **Get-Content $file |foreach {$groupname, $servername = $_.split(); $ServerGroup = Add-ServerGroup $groupname; Add-RegisteredServer $ServerGroup $servername}** **} ** Launch Powershell cd to the file location Source the Powershell script **. ./sqlem.ps1 ** Example usage to export and import server registrations to c:serves.txt file: **Export-RegisteredServers c:servers.txt ** or **Import-RegisteredServers c:servers.txt **