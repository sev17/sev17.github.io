title: Adventures in Powershell SSIS Administration Programming
link: http://sev17.com/2009/06/19/adventures-in-powershell-ssis-administration-programming/
author: Chad Miller
description: 
post_id: 9963
created: 2009/06/19 12:44:00
created_gmt: 2009/06/19 16:44:00
comment_status: open
post_name: adventures-in-powershell-ssis-administration-programming
status: publish
post_type: post

# Adventures in Powershell SSIS Administration Programming

In January 2009, I released version 1.4 of [SQL Server Powershell Extensions](http://sqlpsx.codeplex.com/) which included a Library of functions for working with [SQL Server Integration Services (SSIS)](http://msdn.microsoft.com/en-us/library/ms141026.aspx). [This post](/2009/01/sqlpsx-1-4-release/) describes the functionality included in the script library including enumerating msdb package stores, getting packages from msdb and file stores as well as copying packages and folders among other things. At the time I felt the script library was just a way to prototype what should later be implemented as a [Powershell Provider](http://msdn.microsoft.com/en-us/library/ms714636\(VS.85\).aspx). I even named the functions after what I would later implement in a provider (Get-ISItem, Copy-ISItem, New-ISItem, Remove-ISItem, Rename-ISItem, Test-ISPath) prefixing eaching with IS for Integration Services.

Over the past week I've been thinking about how to code an SSIS Powershell provider by re-reading [Professional Windows Powershell Programming](http://www.amazon.com/Professional-Windows-PowerShell-Programming-Providers/dp/0470173939)* chapter on Providers, re-familiarizing myself with the SSIS Powershell code and pouring over the [Microsoft.SqlServer.Dts.Runtime Namespace (ManagedDts)](http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.dts.runtime\(SQL.90\).aspx) documentation. I haven't writen a line C# code for my SSIS provider project, yet I've learned a few things...

  * Writing a provider is hard or at least harder than writing a cmdlet. Haven written a few cmdlets myself, its really not that much more code than writing a classic console (exe) type application. If you can write a console application, you can write a cmdlet.
  * I have a new found respect for those who have written providers including the SQL Server 2008 provider. This is some advanced programming stuff and requires a lot more thought than writing a cmdlet. Although I feel a few dozen cmdlets for SQL Server is more useful than a provider interface, my hat is off to you, [Michiel Wories](http://blogs.msdn.com/mwories/default.aspx), for creating the SQL Server 2008 Powershell provider.
  * The more I use ManagedDts, the more I appreciate [SQL Server Management Server (SMO)](http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.aspx).  SMO is elegant, well-laid out and after a bit of a learning curve generally makes sense. ManagedDts on the other hand is missing core functionality which is exposed through Microsoft's own [SQL Server Management Studio (SSMS)](http://msdn.microsoft.com/en-us/library/ms174173.aspx), [Dtexec](http://msdn.microsoft.com/en-us/library/ms162810.aspx) or [BIDS](http://msdn.microsoft.com/en-us/library/ms173767.aspx). As someone who automate things through scripts, I find the limited support for SSIS administration scripting disappointing.

I don't want to turn this post into a rant, so instead of listing every SSIS administration and scripting issue of which there are many, I will illustrate just the top one that is preventing me from creating an SSIS Powershell provider. First, let's go over the setup I have. I'm using SQL Server 2005 Standard Edition named instance called Z002SQLEXPRESS. I know the name says SQLEXPRESS, but its not I just don't feel like changing demonstrate code and naming the instance other than SQLEXPRESS. Because I use a named instance I modified the MsDtsSrvr.ini.xml file located in C:Program FilesMicrosoft SQL Server90DTSBinn directory according to the [SQL Server documentation](http://msdn.microsoft.com/en-us/library/ms137789.aspx) and added the following entry:

<TopLevelFolders> <Folder **xsi**:**type**="SqlServerFolder"> <Name>Z002_SQLEXPRESS</Name> <ServerName>Z002SQLEXPRESS</ServerName> </Folder>

I then created a SSIS package called "test" and deployed to the SQL Server store. I can the connect to Integration Services through SSMS 2005 as shown the in following screen shots:

![](http://images.sev17.com/SSISConnect.jpg)

![](http://images.sev17.com/SSISExplorer.jpg)

Now let's try to connect to Integration Services using Powershell and enumerate the folders and packages.

First load the ManagedDTS assembly (note this should be a single line). I'm loading the 2005 instead of 2008 assembly.

**[reflection.assembly]**::Load("Microsoft.SqlServer.ManagedDTS, Version=9.0.242.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91") > $null

To load the 2008 assembly:

**[reflection.assembly]**::Load("Microsoft.SqlServer.ManagedDTS, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91") > $null

An Application is the main class in the ManagedDTS, most of what an administrator would be interested in is accessible via its methods. So, next we'll create an SSIS Application object:

$app =  **new-object** ("Microsoft.SqlServer.Dts.Runtime.Application")

And finally we will enumerate the folders and packages using the GetPackgeInfos method. $app.GetPackageInfos("",'Z002SQLExpress',$null,$null) The following information is returned: ![](http://images.sev17.com/SSISGetPackageInfos.jpg) **Problem #1: The call to GetPackgeInfos requires the SQL instance name i.e. Z002SQLExpress while the equivalent SSMS SSIS connection you specify just the server name i.e. Z002**