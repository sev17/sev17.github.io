title: T-SQL Tuesday #005: SSIS Reporting
link: http://sev17.com/2010/04/12/t-sql-tuesday-005-ssis-reporting/
author: Chad Miller
description: 
post_id: 10160
created: 2010/04/12 23:01:16
created_gmt: 2010/04/13 03:01:16
comment_status: open
post_name: t-sql-tuesday-005-ssis-reporting
status: publish
post_type: post

# T-SQL Tuesday #005: SSIS Reporting

Automating SQL Server Integration Services (SSIS) administration through PowerShell is very different than writing scripts against the core database engine. Once you move outside the core database engine the .NET classes for working with SQL Server features like SQL Server Integration Services vary greatly in the way they are implemented. The core database engine has a very rich and well-laid out object model in [SQL Server Management Objects or SMO](http://msdn.microsoft.com/en-us/library/cc285859.aspx). Although SSIS  does not  have as rich or as intuitive an object model as SMO, we can still accomplish many administration tasks using a namespace called [Microsoft.SqlServer.Dts.Runtime Namespace (ManagedDts)](http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.dts.runtime\(SQL.90\).aspx). 

The CodePlex project, [SQL Server PowerShell Extensions](http://sqlpsx.codeplex.com/) includes a PowerShell V2 module which uses ManagedDts for accomplishing many common tasks in deploying and administering SSIS. In the spirit of [TSQL Tuesday #005](http://sqlvariant.com/wordpress/index.php/2010/04/t-sql-tuesday-005-reporting/) this is my contribution—**reporting and changing SSIS configurations and Connections**…

The use of configurations in SSIS <strike>provides DBAs with a challenging twist in troubleshooting SSIS packages at 3 in the morning</strike> allows developers the flexibility to dynamically set connection strings at runtime. So, being able to report and change configurations and connection information via scripts is a useful task. The first thing we need to do after downloading and installing SQLPSX is to modify the SSIS.psm1 file to either work against 2005 or 2008. This is because unfortunately ManagedDts is not backwards compatible. In PowerShell the # symbol is a comment--comment/uncomment the line for 2005 or 2008 to suite your environment.
    
    
    #add-type -AssemblyName "Microsoft.SqlServer.ManagedDTS, Version=9.0.242.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    add-type -AssemblyName "Microsoft.SqlServer.ManagedDTS, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    

We can then import the module into our current PowerShell session:
    
    
    import-module SSIS

In this example we are going to determine the configuration and connection settings for 24 SSIS packages stored on the file system:
    
    
    $packages = dir "C:Program FilesMicrosoft SQL Server100DTSPackages*" | select -ExpandProperty Fullname | foreach {get-ispackage -path $_ }
    $packages | foreach {$package = $_; $_.Configurations | Select @{n='Package';e={$Package.DisplayName}}, Name,ConfigurationString}
    $packages | foreach {$package = $_; $_.Connections | Select @{n='Package';e={$Package.DisplayName}}, Name,ConnectionString}
    

 

This produces the following output:

![ssisConfigurations](http://images.sev17.com/ssisConfigurations_thumb.jpg)

![ssisConnections](http://images.sev17.com/ssisConnections_thumb.jpg)

To make things more interesting and illustrate that you can not only report but also change properties with ManagedDts/PowerShell we’ll change the connection string for the configuration SSISCONFIG as part of a copy process. We’ll also create new folder called sqlpsx on the root of the SQL Server SSIS package store:
    
    
    new-isitem "msdb" "sqlpsx" "Z002"
    copy-isitemfiletosql -path "C:Program FilesMicrosoft SQL Server100DTSPackages*" -destination "msdbsqlpsx" -destinationServer "Z002" -connectionInfo @{SSISCONFIG=".SQLEXPRESS"}
    

Packages stored on SQL Server can be worked with just as like SSIS packages stored on the file system. The following code returns the configuration and connection information of our newly copied SSIS packages:
    
    
    $packages = get-isitem -path 'sqlpsx' -topLevelFolder 'msdb' -serverName "Z002SQL2K8" | where {$_.Flags -eq 'Package'} | foreach {get-ispackage -path $_.literalPath -serverName $_.Servername}
    $packages | foreach {$package = $_; $_.Configurations | Select @{n='Package';e={$Package.DisplayName}}, Name,ConfigurationString}
    $packages | foreach {$package = $_; $_.Connections | Select @{n='Package';e={$Package.DisplayName}}, Name,ConnectionString}
    

A few notes:

  * PowerShell version 2 is required. The code will not work in sqlps or PowerShell V1. To obtain a V1 compatible script you can download version 1.61 of SQLPSX
  * The example use Z002 as the SSIS server—modify accordingly.
  * Unlike SQL Server Management Studio (SSMS), ManagedDts requires full instance name for some of the underlying method calls, hence Z002SQL2K8 instead of just Z002.
  * Notice the topLevelFolder parameter, unlike SSMS, ManagedDts does not provide a property or method to determine the toplevelfolder setting. See [this post](/2009/06/adventures-in-powershell-ssis-administration-programming/) for more information.
  * The PowerShell code in this blog post wraps lines, however the full sample script can be downloaded from [here](http://poshcode.org/1769)