title: Denali SQLPS First Impressions
link: http://sev17.com/2011/07/18/denali-sqlps-first-impressions/
author: Chad Miller
description: 
post_id: 10705
created: 2011/07/18 08:15:16
created_gmt: 2011/07/18 12:15:16
comment_status: open
post_name: denali-sqlps-first-impressions
status: publish
post_type: post

# Denali SQLPS First Impressions

I’ve taken a few hours to try out Denali CTP 3 sqlps and noticed some welcome changes.  The biggest change for sqlps is that it has been implemented as module and plain old Powershell host--It’s no longer mini-shell! 

## SQLPS Host

SQLPS is now regular Powershell host implemented as the familiar sqlps.exe. Prior versions of sqlps were a  mini-shell, which is to say a Powershell host that implements RunsapceConfiguration with explicitly defined cmdlets and no support for add-pssnapin. The old implementation was limiting in that you couldn’t add cmdlets or providers. I’ve previously [written about the SQL Server 2008 and SQL 2008 R2 sqlps](/2010/05/the-truth-about-sqlps-and-powershell-v2/)  so I won’t spend much time on it here, but I will say that I really like what SQL Server product team has done with the Denali version of sqlps. There some 40 new cmdlets and 2 new “providers” over what was provided in SQL Server 2008 R2. Note: sqlps uses something called SqlServerProviderExtensions which are not like regular providers in that you can’t load them individually—so there’s really only a single “SQLServer” provider. Its interesting to see how the SQL Server product team has organized their provider so that they easily plug in these extensions. As far as I know this is unique among provider implementations. 

## Modules

Denali Powershell implementation also includes two modules, SQLASCMDLETS and to make things a little confusing there’s binary module called SQLPS which has the same name as the sqlps.exe host. Both modules are located under: 
    
    
    C:Program Files (x86)Microsoft SQL Server110ToolsPowerShellModules

## Cmdlets
    
    
    get-command -CommandType cmdlet -Module sqlps,sqlascmdlets | group-object -Property verb

**Count**
**Verb**
**Group**

3
Add
Add-RoleMember, Add-SqlAvailabilityDatabase, Add-SqlAvailabilityGroupListenerStaticIp}

2
Backup
Backup-ASDatabase, Backup-SqlDatabase

1
Convert
Convert-UrnToPath

1
Decode
Decode-SqlName

1
Disable
Disable-SqlHADRService

1
Enable
Enable-SqlHADRService

1
Encode
Encode-SqlName

6
Invoke
Invoke-ASCmd, Invoke-PolicyEvaluation, Invoke-ProcessCube, Invoke-ProcessDimension, Invoke-ProcessPartition, Invoke-Sqlcmd

1
Join
Join-SqlAvailabilityGroup

1
Merge
Merge-Partition

6
New
New-RestoreFolder, New-RestoreLocation, New-SqlAvailabilityGroup, New-SqlAvailabilityGroupListener, New-SqlAvailabilityReplica, New-SqlHADREndpoint

4
Remove
Remove-RoleMember, Remove-SqlAvailabilityDatabase, Remove-SqlAvailabilityGroup, Remove-SqlAvailabilityReplica

2
Restore
Restore-ASDatabase, Restore-SqlDatabase

1
Resume
Resume-SqlAvailabilityDatabase

4
Set
Set-SqlAvailabilityGroup, Set-SqlAvailabilityGroupListener, Set-SqlAvailabilityReplica, Set-SqlHADREndpoint

1
Suspend
Suspend-SqlAvailabilityDatabase

1
Switch
Switch-SqlAvailabilityGroup

3
Test
Test-SqlAvailabilityGroup, Test-SqlAvailabilityReplica, Test-SqlDatabaseReplicaState

## Providers
    
    
    PS SQLSERVER:> dir

**Name**
**Root**
**Description**

SQL
SQLSERVER:SQL
SQL Server Database Engine

SQLPolicy
SQLSERVER:SQLPolicy
SQL Server Policy Management

SQLRegistration
SQLSERVER:SQLRegistration
SQL Server Registrations

DataCollection
SQLSERVER:DataCollection
SQL Server Data Collection

XEvent
SQLSERVER:XEvent
SQL Server Extended Events

Utility
SQLSERVER:Utility
SQL Server Utility

DAC
SQLSERVER:DAC
SQL Server Data-Tier Application Component

IntegrationServices
SQLSERVER:IntegrationServices
SQL Server Integration Services

SQLAS
SQLSERVER:SQLAS
SQL Server Analysis Services
IntegrationServices and SQLAS are new to Denali.

## Using SQLPS

Just as in previous versions you launch the sqlps.exe host by right-clicking an object in SQL Server Management Studio Object Explorer and selecting “Start Powershell” or by typing sqlps.exe from the Run or command-prompt. Alternatively if you want to load the SQLPS module in your powershell.exe host then you can run: 
    
    
    $env:PSModulePath = $env:PSModulePath + ";C:Program Files (x86)Microsoft SQL Server110ToolsPowerShellModules"
    import-module sqlps

Since I’ve implemented my own backup and restore functions I thought I’d test the Backup-SqlDatabase cmdlet:

## Comments

**[Chad Miller](#269 "2011-07-18 19:54:39"):** To execute a package within IntegrationServices provider extension: PS SQLSERVER:IntegrationServicesSQL11DEFAULTCatalogsSSISDBFoldersSQLPSXProjectstestPackages> get-item sqlpsx1* | %{$_.Execute($false,$null)}

