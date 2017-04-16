title: Check for Windows Server 2003 Scalable Networking Pack
link: http://sev17.com/2009/02/17/check-for-windows-server-2003-scalable-networking-pack/
author: Chad Miller
description: 
post_id: 9942
created: 2009/02/17 15:56:00
created_gmt: 2009/02/17 19:56:00
comment_status: open
post_name: check-for-windows-server-2003-scalable-networking-pack
status: publish
post_type: post

# Check for Windows Server 2003 Scalable Networking Pack

I've previously blogged about the [issues with Windows Server 2003 Scalable Networking Pack](/2008/12/disable-windows-server-2003-scalable-networking-pack/) included in Windows 2003 Service Pack 2. I keep finding servers in my environment with the feature enabled, so I created a short Powershell script to check for SNP features by querying the registry of the remote server. The registry keys involved are documented in the [Microsoft KB article 948496](http://support.microsoft.com/kb/948496).

To use, save the following Powershell code as a script file and pass the computer name as a parameter to the script:

**param** ($computer) $reg = **[Microsoft.Win32.RegistryKey]**::OpenRemoteBaseKey('LocalMachine',$computer) $regKey = $reg.OpenSubKey("SYSTEM\CurrentControlSet\Services\Tcpip\Parameters") **new-object** psobject | **add-member** -pass NoteProperty computer $computer | **add-member** -pass NoteProperty EnableTCPA $regKey.GetValue('EnableTCPA') | **add-member** -pass NoteProperty EnableRSS $regKey.GetValue('EnableRSS') | **add-member** -pass NoteProperty EnableTCPChimney $regKey.GetValue('EnableTCPChimney')