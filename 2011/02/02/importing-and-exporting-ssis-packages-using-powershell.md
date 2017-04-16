title: Importing and Exporting SSIS Packages Using PowerShell
link: http://sev17.com/2011/02/02/importing-and-exporting-ssis-packages-using-powershell/
author: Chad Miller
description: 
post_id: 10591
created: 2011/02/02 21:58:08
created_gmt: 2011/02/03 02:58:08
comment_status: open
post_name: importing-and-exporting-ssis-packages-using-powershell
status: publish
post_type: post

# Importing and Exporting SSIS Packages Using PowerShell

[SQL Server PowerShell Extensions](http://sqlpsx.codeplex.com/) (SQLPSX) includes a set of function for working with SSIS which among other things allow you to import and export SSIS packages between the file system and msdb. The functionality is best illustrated by looking a few examples.

## Creating an SSIS folder

_Note: The SSIS module supports SQL 2005 through 2008 R2. By default the module is setup to use the 2008  or 2008 R2 assembly, to switch to 2005, comment/uncomment the appropriate assembly at the top of SSIS.psm1 file in the ModulesSSIS folder. Once loaded an assembly can’t be unloaded (.NET thing), so you’ll need to start a new PowerShell host to switch between 2005 and 2008._

Use the new-isitem function to create a folder. The following example imports the SSIS module and creates a folder called sqlpsx off of the root msdb folder:
    
    
    import-module SSIS
    new-isitem 'msdb' 'sqlpsx' $env:computername

We can see the folder in SSMS:

![ImportSSIS1](http://images.sev17.com/ImportSSIS1_thumb.png)

## Importing SSIS Packages to MSDB

Having created a folder, next I want to import SSIS packages on the file system  to MSDB. In addition as part of the copy process I want to change the location where my SQL Server table-based Package Configuration points:

File System dtsx files:

![ImportSSIS2](http://images.sev17.com/ImportSSIS2_thumb.png)

I’ll use the copy-isitemfiletosql function…
    
    
    copy-isitemfiletosql -path "C:Program FilesMicrosoft SQL Server100DTSPackages*" -destination "msdbsqlpsx" -destinationServer "$env:computername" -connectionInfo @{SSISCONFIG=".SQLEXPRESS"}

Note: The SSIS copy-* functions include a progress bar indicator:

![ImportSSIS3](http://images.sev17.com/ImportSSIS3_thumb.png)

## Exporting SSIS Packages from MSDB

Now that I have SSIS packaged stored in MSDB, I’ll copy them back to the file system using the copy-isitemsqltofile function…
    
    
    copy-isitemsqltofile -path 'sqlpsx' -topLevelFolder 'msdb' -serverName "$env:computernamesql1" -destination 'c:UsersPublicbinSSIS' -recurse

Looking at the file system we see the dtsx files have been created:

![ImportSSIS4](http://images.sev17.com/ImportSSIS4_thumb.png)

_Note: The API ManagedDTS has some inconsistencies in usage, so the SQL Server instance ($env:computernamesql1) instead of just the computer name ($env:computername) is needed._

## Removing SSIS Packages and Folders from MSDB

Note: Like any delete operation be careful!

This isn’t a common operation, but for completeness I’ll remove the SSIS packages and folders I created. As a safety measure the remove and copy  functions support the standard PowerShell WhatIf and Confirm parameters, so first I’ll run the command with –WhatIf:
    
    
     get-isitem 'sqlpsx' 'msdb' "$env:computernamesql1"  | remove-isitem -WhatIf
     get-isitem '' 'msdb' "$env:computernamesql1" | ?{$_.name -like "sqlpsx*"} | remove-isitem -WhatIf

This produces the following output:
    
    
    What if: Performing operation "Remove-ISItem" on Target "RemoveFromDtsServer(msdbsqlpsxsqlpsx1,Z003)".
    What if: Performing operation "Remove-ISItem" on Target "RemoveFromDtsServer(msdbsqlpsxsqlpsx2,Z003)".
    What if: Performing operation "Remove-ISItem" on Target "RemoveFromDtsServer(msdbsqlpsxsqlpsx3,Z003)".
    ...
    What if: Performing operation "Remove-ISItem" on Target "RemoveFolderFromDtsServer(msdbSQLPSX,Z003)".

Satisfied with the results I’ll go ahead and remove the packages and folder:
    
    
     get-isitem 'sqlpsx' 'msdb' "$env:computernamesql1" | remove-isitem
     get-isitem '' 'msdb' "$env:computernamesql1" | ?{$_.name -like "sqlpsx*"} | remove-isitem

## Summary

Including the functions demonstrated in this post the SQLPSX SSIS module contains the following functions:

  * [Copy-ISItemSQLToSQL](http://www.sqlpsx.com/Copy-ISItemSQLToSQL.htm)
  *  [Copy-ISItemSQLToFile](http://www.sqlpsx.com/Copy-ISItemSQLToFile.htm)
  * [Copy-ISItemFileToSQL](http://www.sqlpsx.com/Copy-ISItemFileToSQL.htm)
  * [Get-ISData](http://www.sqlpsx.com/Get-ISData.htm)
  * [Get-ISItem](http://www.sqlpsx.com/Get-ISItem.htm)
  * [Get-ISPackage](http://www.sqlpsx.com/Get-ISPackage.htm)
  * [Get-ISRunningPackage](http://www.sqlpsx.com/Get-ISRunningPackage.htm)
  * [Get-ISSqlConfigurationItem](http://www.sqlpsx.com/Get-ISSqlConfigurationItem.htm)
  * [New-ISApplication](http://www.sqlpsx.com/New-ISApplication.htm)
  * [New-ISItem](http://www.sqlpsx.com/New-ISItem.htm)
  * [Remove-ISItem](http://www.sqlpsx.com/Remove-ISItem.htm)
  * [Rename-ISItem](http://www.sqlpsx.com/Rename-ISItem.htm)
  * [Set-ISConnectionString](http://www.sqlpsx.com/Set-ISConnectionString.htm)
  * [Set-ISPackage](http://www.sqlpsx.com/Set-ISPackage.htm)
  * [Test-ISPath](http://www.sqlpsx.com/Test-ISPath.htm)   


In addition to the online help, each function implement get-help with examples.

### Related Posts:

  * [T-SQL Tuesday #005: SSIS Reporting](/2010/04/t-sql-tuesday-005-ssis-reporting/)
  * [Adventures in Powershell SSIS Administration Programming](/2009/06/adventures-in-powershell-ssis-administration-programming/)
  * [Execute SQL Server Integration Services (SSIS) Package from Powershell](/2009/02/execute-sql-server-integration-services-ssis-package-from-powershell/)
  * [Providing Online Help for Powershell Modules](/2010/02/providing-online-help-for-powershell-modules/)

## Comments

**[Jamie Thomson](#212 "2011-02-03 07:53:25"):** Awesome stuff Chad. I've updated my psot with a link to this: http://sqlblog.com/blogs/jamie_thomson/archive/2011/02/02/export-all-ssis-packages-from-msdb-using-powershell.aspx

**[Chad Miller](#215 "2011-02-08 20:06:13"):** Thanks. As soon as I have some time I'm planning on building an SSIS provider so you can navigate and SSIS "drive" just as you would with SQLServer using sqlps. With the concept of packages and folders, SSIS fits well into the provider model.

**[Conor](#217 "2012-11-06 17:07:16"):** This is exactly what I need, but being a noob I have no idea how to use this. I have downloaded the SQLPSX but beyond that, I don't know how to start using your examples. In my job I find myself needing to export SSIS packages from SQL 2005 and import them into SQL 2008(R2). How do I find the query editor window you are using above?

**[Chad Miller](#218 "2012-11-06 22:14:40"):** The SSIS module includes documented help, for example you can run: `import-module ssis help copy-isitemsqltofile -full` Also my blog has a couple of examples using the SSIS module: <http://sev17.com/2011/02/importing-and-exporting-ssis-packages-using-powershell/> <http://sev17.com/2011/02/t-sql-tuesday-15-automation-and-the-ssis-dumper/> However the task you're trying automate can't be done using the SSIS module or dtutil.exe. To upgrade SSIS packages you'll need to open the 2005 SSIS packages and save them in BIDS (i.e. Visual Studio) 2008 R2. You could save them to an intermediate dtsx file using the copy-isitemsqltofile or dtutil, but the upgrade unfortunately is still manual through VS open and save. I'm not sure what you mean by query editor window. If you mean styling the code in my blog I'm using WordPress WP-Syntax.

**[Conor](#219 "2012-11-08 12:02:01"):** nevermind, I found it. but when I ran the very first example above, new-isitem I am getting the following error: New-ISItem : Cannot bind argument to parameter 'serverName' because it is an empty string. At line:2 char:11 \+ new-isitem <<<< 'msdb' 'sqlpsx' $env:HBI_2003 \+ CategoryInfo : InvalidData: (:) [New-ISItem], ParameterBindingValidationException \+ FullyQualifiedErrorId : ParameterArgumentValidationErrorEmptyStringNotAllowed,New-ISItem

**[Conor](#220 "2012-11-08 12:09:22"):** By the way, thank you so much for this amazing tool. I am slowly figuring it out.

**[Conor](#221 "2012-11-08 12:33:08"):** This is also on a 2005 server so to comment out the 2008 section I simply put a # in front of the 'add-type -AssemblyName......Version=10.0.0.0' line. Is that correct?

**[Chad Miller](#222 "2012-11-09 14:12:11"):** My guess is that $env:HBI_2003 is null, as the error messages indicates. What do you get when you run $env:HBI_2003?

**[Chad Miller](#223 "2012-11-09 14:18:16"):** You shouldn't need comment out to use 2005, because of the if/else logic at the top of the SSIS.psm1. The module is text file so you can ready it. To use 2005 you run: import-module SSIS -ArgumentList 2005 One very important thing to keep in mind. In .NET you can't unload an assembly so if you load the 2008/2008 R2 assembly in a Powershell session you'll need open a new Powershell Window to load 2005 assembly. The reverse applies also.

**[Conor](#224 "2012-11-09 14:49:18"):** it comes back with PS C:Documents and SettingsAdministrator> $env:hbi2003 I updated the name and took the underscore out, in case you were wondering why it looks different now. But it still gives me the same error.

**[Chad Miller](#225 "2012-11-09 15:52:59"):** So you have an environmental variable hbi2003? In any case new-isitem expects 3 parameters and it isn't receiving the server name. I would suggest as a test, don't use $env:hbi2003 and instead specify the servername directly: new-isitem 'msdb' 'sqlpsx' 'YourServerName'

**[Conor](#226 "2012-11-09 16:16:26"):** when you say "run $env:HBI_2003" do you just mean simply typing $env:HBI_2003 into line 1 and hitting play?

**[Chad Miller](#227 "2012-11-09 17:43:10"):** It would be the run script button which looks like a play button if you're in the editor. You should see a value. $env: is a prefix for environmental variable. If you're using cmd you'd use "set" to see all environment variable or echo %envname%. In Powershell the environnmental variables are accessed using $env: instead of %name%.

**[Conor](#228 "2012-11-12 10:54:21"):** $env:COMPUTERNAME this returns 'HBI2003'. However if I use HBI2003 in the new-isitem command, I get "New-ISItem : Cannot bind argument to parameter 'value' because it is an empty string." I have confirmed this is my server name, I must be missing something. Do I need to do anything special with the Path or tell the SSIS.psm1 file where to get the servername from? I am sorry if I am being a pest, just would really like to get this working.

**[Chad Miller](#229 "2012-11-12 11:06:49"):** So you've started a new Powershell.exe? It's complaining about the value parameter being null or empty and stated this was 2005 server? Let's try this. `import-module SSIS -ArgumentList 2005 new-isitem -path 'msdb' -value sqlpsx -serverName HBI2003`

**[Conor](#230 "2012-11-12 15:56:41"):** THAT DID IT!!!!!!!!!! I was able to create the folder and Import SSIS pacakges. I have also been able to Export them. I did just have one other question. The way this is setup the SSIS packages need to live under the sqlpsx folder under msdb. More often than not, I see the packages just under the root of msdb. how would I modify the command line to leave out the 'sqlpsx' value so it's just looking under the root of msdb? I can't thank you enough for taking the time to help me with this!

**[Chad Miller](#231 "2012-11-12 16:12:33"):** No problem, its usually better to use named instead of positional parameters anyways. I should update the documentation for new-isitem accordingly. The sqlpsx folder is just something I use for an example if you want to copy to the root msdb folder, just use msdb instead of 'msdbsqlpsx' here's an example: `Copy-ISItemFileToSQL C:UsersPublicbinSSISsqlpsx1.dtsx -destination msdb -destinationServer HBI2003`

**[Conor](#232 "2012-11-13 09:31:32"):** I think I should have been more specific. I am able to export the SSIS packages when they live under the sqlpsx folder. However, if they are just under the root of msdb, they don't get exported. Keeping that in mind, what value should I give the "-path" parameter if the packages are found just under the root of msdb? copy-isitemsqltofile -path 'sqlpsx' -topLevelFolder 'msdb' -serverName "$env:computername" -destination 'C:SSISSSISExport' -recurse Thanks!

**[Chad Miller](#233 "2012-11-13 09:41:08"):** -path '' everything else should be the same. There's an example of getting but not copying from the root using the get-isitem function, same concept though. help get-isitem -examples

**[Conor](#234 "2012-11-14 16:17:15"):** Chad, I can't thank you enough for all the time you took to walk me through this. I have it all setup and it works perfectly! This is a HUGE lifesaver. Thanks again, Conor

**[Chad Miller](#235 "2012-11-15 13:17:09"):** You're welcome and thanks for sticking with it.

**[Paul Hernandez](#308 "2013-03-27 06:57:31"):** Hi Chad, thanks so much for your amazing work. I just want to point out, that I had to slightly modify the input parameters for: .- copy-isitemfiletosql : -destination "msdbsqlpsx" -> -destination "msdb\sqlpsx" .- copy-isitemsqltofile: -topLevelFolder 'msdb' -> -topLevelFolder 'msdb\' I don't know exactly why this "\" problem, maybe something related to my specific configuration but I´m just guessing. Anyway it works. Kind Regard, Paul

