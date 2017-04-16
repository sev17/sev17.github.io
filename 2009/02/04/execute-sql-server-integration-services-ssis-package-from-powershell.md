title: Execute SQL Server Integration Services (SSIS) Package from Powershell
link: http://sev17.com/2009/02/04/execute-sql-server-integration-services-ssis-package-from-powershell/
author: Chad Miller
description: 
post_id: 9939
created: 2009/02/04 20:54:00
created_gmt: 2009/02/05 00:54:00
comment_status: open
post_name: execute-sql-server-integration-services-ssis-package-from-powershell
status: publish
post_type: post

# Execute SQL Server Integration Services (SSIS) Package from Powershell

***Updated 2/23/2009. After some additional testing I discovered you cannot change the location of SSISConfig without saving and reloading a package or exporting and importing a configuration, so I removed the functionality to reset the config location since there isn't an easy way of doing so. In addition errors were not being automatically enumerated. The script has been updated to return errors. One open question I have is whether executing SSIS packages using the ManagedDTS class executes under x64 or x86. I will post an update once I determine this. Based on the work I've done with Powershell and SSIS in [SQL Server Powershell Extensions](http://www.codeplex.com/SQLPSX), I created a standalone Powershell script to execute SSIS packages.   The script is available on [Poshcode](http://poshcode.org/) [here](http://poshcode.org/889).  The Powershell script offers several advantages over the dtexec: 

  1. Powershell is a .NET language so you can access SSIS programming model
  2. Logging to the Windows Application Event log is optional
  3. Full error messages are returned
  4. Return codes are set by the throw statement
  5. Execute both server and file based SSIS packages
  6. Verifies existence of package before execution
Note:  SSIS is NOT backwards compatible. At the beginning of the script you’ll need to comment/uncomment the specific assembly to load 2005 or 2008. Currently the script is set to 2005 Script Examples/Description: _ _ Executes SSIS package for both server and file system storage types. \-------------------------- EXAMPLE 1 -------------------------- ./RunSSIS.ps1 -path Z002_SQL1sqlpsx -serverName 'Z002SQL1' This command will execute package sqlpsx on the server Z002SQL1 \-------------------------- EXAMPLE 2 -------------------------- ./RunSSIS.ps1 -path Z002_SQL1sqlpsx -serverName Z002SQL1 -configFile 'C:SSISConfigsqlpsx.xml'  This command will execute the package as in Example 1 and process and configuration file \-------------------------- EXAMPLE 3 -------------------------- ./RunSSIS.ps1 -path 'C:SSISsqlpsx.dtsx' This command will execute the package sqlpsx.dtsx located on the file system \-------------------------- EXAMPLE 4 -------------------------- ./RunSSIS.ps1 -path 'C:SSISsqlpsx.dtsx -nolog This command will execute the package sqlpsx.dtsx located on the file system and skip Windows Event logging

## Comments

**[Chad Miller](#23 "2009-09-04 20:54:00"):** This is PowerShell Library, which is just a bunch of PowerShell functions intepreted at runtime from within the PowerShell host. It's not a C++ Library.

**[Greg](#24 "2009-09-04 20:54:00"):** I'm trying to use your latest script and when I run it using the file system and config option I get a Microsoft Visual C++ Runtime library error that states:   
Program:C:Windowssystem32WindowsPowerShellv1.0powershell.exe  
R6034   
An Application has mad an attempt to load the Cruntime Library incorrectly.

