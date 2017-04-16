title: Gaining SQL Server SysAdmin Access
link: http://sev17.com/2011/10/22/gaining-sql-server-sysadmin-access/
author: Chad Miller
description: 
post_id: 10748
created: 2011/10/22 15:51:18
created_gmt: 2011/10/22 19:51:18
comment_status: open
post_name: gaining-sql-server-sysadmin-access
status: publish
post_type: post

# Gaining SQL Server SysAdmin Access

I’ve seen this come a few times at work and I’m sure most you have experienced something similar.

> Someone or an application installs SQL Server, doesn’t grant access to the DBA group and asks for DBA support.

In SQL Server 2008 and higher the built-in local administrators group is no longer automatically part of the SQL Server sysadmin role. You should add necessary logins to the sysadmin role as part of your SQL Server installation. Not automatically granting local administrators access to SQL Server is generally a good thing, however when the SQL Server installation is done by say another application then we see issues were support groups do not have access to SQL Server even though they are local administrators on the box. In the last few months I’ve seen this scenario several times and so has Argenis Fernandez ([blog](http://sqlblog.com/blogs/argenis_fernandez/default.aspx)|[twitter](http://twitter.com/#!/DBArgenis)) as he has a helpful blog post entitled  “[Think Your Windows Administrators Don’t Have Access to SQL Server 2008 by Default? Think Again](http://sqlblog.com/blogs/argenis_fernandez/archive/2011/07/10/think-your-windows-administrators-don-t-have-access-to-sql-server-2008-by-default-think-again.aspx).”  The post describes a technique of using the Sysinternals tool [PsExec](http://technet.microsoft.com/en-us/sysinternals/bb897553) to gain SQL sysadmin access to SQL Server on which you already have local administrator access. The  post also links to a documented way of starting SQL Server in single user mode in order to gain SQL Server sysadmin access (see “[Troubleshooting: Connecting to SQL Server When System Administrators Are Locked Out](http://msdn.microsoft.com/en-us/library/dd207004.aspx)).” And finally the post mentions, but does not demonstrate a method of using the Windows Task Scheduler. If you’re interested in the how’s and why’s this works and how different versions of SQL Servers are well, different in security settings defaults I encourage you to give the post and comments a read.

Armed with information on how to gain SQL Server administration I looked at the various options. The psexec utility is blacklisted in my environment, blocked from download and listed as an untrusted application. The approach of starting SQL Server in single user mode requires taking SQL Server down and since the application is a quasi-production system restarting SQL Server would have to be coordinated or done after hours. So, I chose to to use the Windows Scheduler trick. This should work on Windows 2003/XP and higher and I’ve tested on Windows 2008 and Windows 7. The typical UAC things apply—you’ll need to run as administrator. The script I came up with is listed below. I figured if I’m getting a server were some other group performed the installation or configuration there’s no guarantee PowerShell will be installed, so I’m going old school Windows batch file on this. It feels wrong to post a Windows batch file  instead of a Powershell script on my blog, but I think using a batch file is the best approach for the problem. To use, save the script as AddDBA.bat and see the example syntax. The script must be run locally.
    
    
    @echo off
    @if "%1"=="?" goto Syntax
    @if "%1"==""  goto Syntax
    @if "%2"==""  goto Syntax
    rem **********************************
    rem Script AddDBA.bat
    rem Creation Date: 10/21/2011
    rem Last Modified: 10/21/2011
    rem Author: Chad Miller
    rem ***********************************
    rem Description: Adds a Windows Account to SQL Sysadmin Role
    rem Use when you have local Windows admin access but lost SQL Sysadmin access
    rem ***********************************
     
    @echo ************************
    @echo *** ServerInstance: %1
    @echo ************************
    set TMPFILE=%TMP%AddDBA-%RANDOM%-%TIME:~6,5%.tmp
    schtasks  /Create /TN AddDBA /SC Once /ST 12:00 ^
    /TR "sqlcmd -S %1 -Q \"CREATE LOGIN [%2] FROM WINDOWS; EXEC sp_addsrvrolemember [%2],[sysadmin];\" -o \"%TMPFILE%\" -e" ^
    /RU "NT AUTHORITY\SYSTEM" /F
    schtasks /Run /TN AddDBA
    schtasks /Query /TN AddDBA /V /FO List
    rem Wait 5 seconds
    PING 127.0.0.1 -n 6  >NUL
    rem Display output file
    type %TMPFILE%
    schtasks /Delete /TN AddDBA /F
    goto :EXIT
     
    :Syntax
    @echo Syntax: AddDBA ServerInstance WindowsGroupOrLogin
    @echo Example: AddDBA Z001\SQL1 Contoso\DBAGroup
    goto :EXIT
     
    :EXIT