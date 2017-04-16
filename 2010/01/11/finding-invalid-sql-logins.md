title: Finding Invalid SQL Logins
link: http://sev17.com/2010/01/11/finding-invalid-sql-logins/
author: Chad Miller
description: 
post_id: 9990
created: 2010/01/11 21:04:00
created_gmt: 2010/01/12 01:04:00
comment_status: open
post_name: finding-invalid-sql-logins
status: publish
post_type: post

# Finding Invalid SQL Logins

As many of you know the system stored procedure [sp_validatelogins](http://msdn.microsoft.com/en-us/library/ms181728.aspx) is used for finding invalid logins. Although sp_validatelogins is useful there's one problem -- the output isn't always accurate. You see when you add a a Windows account to SQL Server the [SID](http://msdn.microsoft.com/en-us/library/aa379594\(VS.85\).aspx) as well as the domain (or computer name) slash account name are stored in master database, if the account is renamed in Active Directory or in the case of local users on the local system, the account stills retains access to SQL Server. How is this possible? That's because the SID is unchanged and that is what SQL Server uses. When you run sp_validatelogins the account name is validated but not the SID and a valid but rename account is returned.

So, what we need to do is make sp_validateLogins accurate by resolving the SID against Active Directory or the local system. As add bonus we should return the rename account name. Fortunately this is pretty easy with a little Powershell script. The following is a standalone excerpt from [SQL Server PowerShell Extensions](http://sqlpsx.codeplex.com/), edited to work with Microsoft's [sqlps](http://msdn.microsoft.com/en-us/library/cc280450.aspx):

function Get-InvalidLogins { param($ServerInstance) foreach ($r in Invoke-SqlCmd -ServerInstance $ServerInstance -Database 'master' -Query 'sp_validatelogins') { $NTLogin = $r.'NT Login' $SID = new-object security.principal.securityidentifier($r.SID,0) $newAccount = $null trap { $null; continue } $newAccount = $SID.translate([system.security.principal.NTAccount]) if ($newAccount -eq $null) { $isOrphaned = $true $isRenamed = $false } else { $isOrphaned = $false $isRenamed = $true } if ($NTLogin -ne $newAccount) { new-object psobject | add-member -pass NoteProperty NTLogin $NTLogin | add-Member -pass NoteProperty TSID $SID | add-Member -pass NoteProperty Server $ServerInstance | add-Member -pass NoteProperty IsOrphaned $isOrphaned | add-Member -pass NoteProperty IsRenamed $isRenamed | add-Member -pass NoteProperty NewNTAccount $newAccount } } } #Get-InvalidLogins

To use the script simply copy and paste the function defintion into a sqlps session or alternatively you can add the function to your [Windows Powershell profile](http://msdn.microsoft.com/en-us/library/bb613488\(VS.85\).aspx).

Next simply call the function specifying a SQL Server instance:

**Get-InvalidLogins "Z002SQL2K8"**

### 

### Credits and History

The original idea for the code came from a blog post which uses a [CLR solution](http://blogs.msdn.com/lcris/archive/2005/09/26/474202.aspx).  In my pre-Powershell days (2006) I created this [Perl script](http://www.sqlservercentral.com/scripts/Maintenance+and+Management/31782/).