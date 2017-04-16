title: Dynamically Register SQL Instances in SQL Server Management Studio 2008
link: http://sev17.com/2009/01/27/dynamically-register-sql-instances-in-sql-server-management-studio-2008/
author: Chad Miller
description: 
post_id: 9938
created: 2009/01/27 15:18:00
created_gmt: 2009/01/27 19:18:00
comment_status: open
post_name: dynamically-register-sql-instances-in-sql-server-management-studio-2008
status: publish
post_type: post

# Dynamically Register SQL Instances in SQL Server Management Studio 2008

I've previously blog, about [programmatically registering SQL instances in SQL Server Management Studio (SSMS)](/2008/12/registering-sql-servers-in-2000-em-2005-ssms-and-2008-ssms/) and on [Dynamically discovering SQL instances through SMS/SCCM](http://sev17.com/2008/11/inventory-sql-server-databases-with-powershell/). This post combines the two techniques showing how to register any missing registrations from your SSMS 2008 or simply register all SQL instances in a new group.

Copy the following code to a file (for example C:usrbinregister10.ps1):

**UPDATED 1/28/2009 to check for existance of server within group before creating ***
    
    
    param($server,$group=’LostAndFound’,$path = ‘SQLSERVER:SQLRegistrationDatabase Engine Server Group’,[bool]$allowDups=$false)
    SET-LOCATION $path
    if (!(Test-Path $group))
    { New-Item $group }
    [bool]$isExists = $false
    Get-ChildItem -recurse | where {$_.Mode -eq ‘-’} | foreach {if ($_.Name -eq $server) {$isExists = $true}}
    SET-LOCATION $group
    if (!($isExists) -or $allowDups)
    {
    if(!(Test-Path $(Encode-Sqlname $server)))
    { New-Item $(Encode-Sqlname $server) -itemtype registration -Value “server=$server;integrated security=true” }
    }
    

Next launch the SQL Server 2008 Powershell host, sqlps.exe
    
    
    CD SQLSERVER: 

Next launch the SQL Server 2008 Powershell host, sqlps.exe 
    
    
    Invoke-SqlCmd -Query “SELECT instance_name FROM sms_sql_instance_vw” -ServerInstance ‘Z002SQL1′ -Database ‘DBAUtility’ | %{C:usrbinregister10.ps1 $_.instance_name}
    

The query uses a view against SMS/SCCM data described in the blog post [Inventory SQL Server Databases with PowerShell](/2008/11/inventory-sql-server-databases-with-powershell/). Since SSMS 2008 allows you to register the same SQL instance multiple times as long as they are not in the same server group, the Powershell script takes parameters to  register all instances. The following registers all SQL instances regardless of whether they are registered or not in a new group called 'All':
    
    
    Invoke-SqlCmd -Query “SELECT instance_name FROM sms_sql_instance_vw” -ServerInstance ‘Z002SQL1′ -Database ‘DBAUtility’ | %{C:usrbinregister10.ps1 -server $_.instance_name -group ‘All’ -allowDups $true}

## Comments

**[selvam](#21 "2011-12-01 09:28:44"):** hi there , how can we do this by vb.net using smo or dmo please note that in sql 2008 smo is obsolate thanks

**[Chad Miller](#22 "2011-12-01 11:45:14"):** SMO is not obsolete, you're thinking of SQL-DMO which was used in SQL Server 7.0 and SQL Server 2000. SMO was introduced in SQL 2005 as a replacement to SQL-DMO. SQL-DMO is the old COM-based technology while SMO is .NET and is the API on which SQL Server Management Studio is built. SMO continues to be supported in SQL 2012 and there hasn't been any indications of making it obsolete. As to how do do this in VB.NET, the code I posted is very much Powershell focus and uses the sqlps host which ships with SQL 2008 and higher. I don't code in VB.NET, but I would suggest looking at the MSDN documentation for the registered servers API. The API to interact with this in .NET can be found here: http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.registeredservers(v=SQL.105).aspx

