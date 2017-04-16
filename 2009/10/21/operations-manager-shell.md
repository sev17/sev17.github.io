title: Operations Manager Shell
link: http://sev17.com/2009/10/21/operations-manager-shell/
author: Chad Miller
description: 
post_id: 9986
created: 2009/10/21 18:25:00
created_gmt: 2009/10/21 22:25:00
comment_status: open
post_name: operations-manager-shell
status: publish
post_type: post

# Operations Manager Shell

I've started using [SCOM 2007 R2](http://www.microsoft.com/systemcenter/operationsmanager/en/us/default.aspx), which has given me a chance to try out the [Operations Manager Shell](http://technet.microsoft.com/en-us/library/cc974484.aspx). The Ops Mgr Shell is a customized PowerShell initialized through a console file and startup script. The important thing to note is that it's still regular PowerShell.

What's interesting about the Ops Mgr shell is the use of  both a provider and cmdlets. You can navigate groups and objects like a file system drive plus there are some nicely thought out cmdlets that can act within the current context of objects returned by the provider through pipeline input.  As someone coming from SQL Server PowerShell I couldn't help but think this is how the SQL Server Powershell host should work i.e. cmdlets integrated with provider. Although many of the Ops Mgr cmdlets allow you to specify paths and objects as parameters for their cmdlets, I found it much easier to use the provider in conjunction with cmdlets--navigating to a group or object and then executing cmdlets to get or change properties.

I'm more of a SCOM user than a SCOM administrator, so my use cases are simple and have nothing to do with administering SCOM. The only things I need to do is get a list of alerts for a particular group or alert and put groups or machines in maintenance mode to avoid alerts during planned maintenance. Here are some scripts I've used:

#Navigate to the SQL Server computer group

PS Monitoring:sunfish1Microsoft.SQLServer.ComputerGroup

#Get all unresolved alerts for the current group get-alert -criteria "ResolutionState = 0" | Select Name, Description, MonitoringObjectPath, MonitoringObjectName, TimeRaised #Get alerts for where the alert name is logical disk frag and export results to CSV file

get-alert -criteria "Name like 'Logical Disk Frag%'" | Select Name, Description, MonitoringObjectPath, MonitoringObjectName | export-csv c:usersu00binDiskFrag.csv -noTypeInfo #Put an entire group in maintenance mode for one hour

get-monitoringobject | New-MaintenanceWindow -startTime:$(get-date) -endTime:$((get-date).AddHours(1)) -comment:"Testing PowerShell script"

#Put the Z002 computer in maintenance mode for one hour get-monitoringobject | where {$_.Name -eq 'Z002.acme.com'} | New-MaintenanceWindow -startTime:$(get-date) -endTime:$((get-date).AddHours(1)) -comment:"Testing PowerShell script" #Note as a DBA two other groups you should look at:

Microsoft.SQLServer.InstanceGroup Microsoft.Windows.Clusters

### A few tips

  * I find myself using the alert script just to export what I'm seeing in my Operations Console (GUI) to a CSV file. I'll then send the simple PowerShell generated CSV reports to my team to address. If you know of an easier way to do this through the GUI, let me know.
  * The maintenance mode scripts are not as useful, it's easier to use the GUI, right-click, and select maintenance mode. Even multiple computers can be selected. I suppose if I had a regularly scheduled maintenance, I would find more uses for a scripted maintenance window solution.
  * It is surprising what SCOM finds, the most common warning condition I've seen include databases with autoclose or autoshrink on. In an active environment with thousands of databases a few inherited databases from SQL Express installation are bound to exist with either autoclose or autoshrink on. Once identified these warning are easy to fix.
  * One other interesting warning is logical disk defragmentation. I had do a little research about this warning and [posted a question to ServerFault](http://serverfault.com/questions/74196/how-is-win32defraganalysis-filepercentfragmentation-calculated). I won't repost here; see the ServerFault Q&A for detailed explanation. As part of the question I created a short script to retrieve logical fragmentation through WMI and PowerShell:
$vols = Get-WmiObject -computername "Z002" Win32_Volume -filter "DriveType=3"

## Comments

**[Kendra Little](#100 "2009-11-05 18:25:00"):** Awesome! This is great, thanks, I can't wait to start using this. (kendra little, <http://littlekendra.com>)

