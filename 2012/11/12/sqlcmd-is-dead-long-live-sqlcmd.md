title: Sqlcmd is dead. Long live Sqlcmd
link: http://sev17.com/2012/11/12/sqlcmd-is-dead-long-live-sqlcmd/
author: Chad Miller
description: 
post_id: 10991
created: 2012/11/12 09:41:33
created_gmt: 2012/11/12 14:41:33
comment_status: open
post_name: sqlcmd-is-dead-long-live-sqlcmd
status: publish
post_type: post


# Sqlcmd is dead. Long live Sqlcmd

SQL Server 2008 and higher ship with the invoke-sqlcmd cmdlet while SQL Server 2005 and higher includes the sqlcmd.exe utility. Not wanting to take a dependency on invoke-sqlcmd, a few years ago I’d written a simpler variation I called [invoke-sqlcmd2](http://poshcode.org/3448) which I’ve updated along with others.

You might think with later releases of SQL Server and the ability to create your own Powershell function that the old sqlcmd.exe utility isn’t needed, but you’d be wrong. There are still certain cases where using sqlcmd.exe is a better choice than invoke-sqlcmd or your own Powershell functions.

I reached this conclusion when converting some automated scripts from VBScript to Powershell used within our job scheduler. What I originally came up with was a simple variation of invoke-sqlcmd2 with some logging and error handling. I called this script invoke-sql.ps1 as posted on PoshCode. The script worked reasonably well for many scheduled jobs, but as I started looking at other existing VBScripts which used sqlcmd they relied on sqlcmd input and output options.

At first I thought I could reproduce the same output options as sqlcmd.exe including:

* Output rows affected
* Write T-SQL PRINT and RAISERROR statments to an output file
* The delimited output should not include header rows as export-csv does
* The delimited output shouldn’t quote all fields as export-csv does

Unfortunately, doing all the above isn’t possible using invoke-sqlcmd, although it is with my invoke-sqlcmd2 function through extending functionality. The main sticking points with invoke-sqlcmd is that rows affected are not returned which some folks have gotten use to seeing in their sqlcmd.exe based scripts and getting at T-SQL PRINT and RAISERROR statement output is only available through the -verbose parameter which is difficult to redirect to a file.

## SqlCmd Lives On

So, if I really want to get away from running sqlcmd.exe in favor of a Powershell cmdlet or function I’d have to write a lot code to duplicate all the output options, I’d have to test it and even then I’d have to provide fixes for things which inevitably happen in production as new issues are discovered. Just as developer sees the Base Class Library in .NET as code they don’t have to write I see native Windows console applications (or cmdlets) as portions of my script I don’t have to write or worry about working correctly.
sqlcmd.exe isn’t dead. The [documentation doesn’t list sqlcmd as deprecated](http://msdn.microsoft.com/en-us/library/ms162773.aspx) and there’s a [nice table which shows the missing features in invoke-sqlcmd](http://msdn.microsoft.com/en-us/library/cc281720.aspx) when compared to sqlcmd on the MSDN page. The documentation reinforces my own experience–invoke-sqlcmd lacks functionality in sqlcmd. Also the fact that sqlcmd isn’t marked for deprecation and has been enhanced (for example connecting to new SQL Server 2012 availability group listener) means I can continue to use it without concern it will go away in the next release or worry new functionality isn’t being added.

## Invoke-SqlCmdExe

Since sqlcmd.exe is here to stay and solves issues with output format requirements I just needed to write a quick wrapper around sqlcmd.exe to do error handling/logging and ensure Powershell doesn’t get messed up on the parameters to sqlcmd.exe. I came up with a script I call [Invoke-SqlCmdExe](http://poshcode.org/3756) and posted on PoshCode.

## Invoke-SqlCmdExe Explained

At the top of the script I create a new Eventlog source. If it already exists the command will fail, so I’ll suppress and clear the error then move on. As stated in the script you must run the script as administrator in order to create a new Eventlog source on a server with UAC enabled, but once the Eventlog source is created you do not need to run as administrator. This is one of the many, many annoyances of UAC running on a server. You could run just this section of code to register a new Eventlog source.

	#This must be run as administrator on Windows 2008 and higher!
	New-EventLog -LogName Application -Source $Application -EA SilentlyContinue
	$Error.Clear()

When running scheduled tasks, my preference is to use the Eventlog for logging messages. I’ll either setup a special Eventlog or use the Application log. The nice thing about using the Eventlog is that it’s always there, so you don’t have worry about messages not getting logged like you would if you were using a single remote database. Also a lot of monitoring tools like System Center or scheduling products include Eventlog  watchers, so you can build actions to respond to messages written to the Eventlog if needed. As part of using the Eventlog log for schedule task messages, you may want to create unique categories so your Eventlog watcher can look for particular events. The hashtable at the top of the script defines my unique categories:

	$events = @{"ApplicationStartEvent" = "31101"; "ApplicationStopEvent" = "31104"; "DatabaseException" = "31725"; "ConfigurationException" = "31705";"BadDataException" = "31760"}

Powershell tends to get a little confused when calling certain native console applications with complex parameters. To avoid this issue with sqlcmd.exe I’ll wrap the call using start-process and here strings. I’ll also grab the exitcode and send all output to a file:

	$exitCode = (Start-Process -FilePath "sqlcmd.exe" -ArgumentList @"
	$Options
	"@ -Wait -NoNewWindow -RedirectStandardOutput $tempFile -Passthru).ExitCode
If you use native console applications in Powershell you need to check exit codes as I’ve done in the script. If the error isn’t zero I’ll throw an error:

	if ($ExitCode -eq 0) {
	$msg = "ApplicationStopEvent"
	Write-Message -Severity Information -Category $events.ApplicationStopEvent -Eventid 99 -ShortMessage $msg -Context $Context
	}
	else {
	throw
	}

The catch and finally statement blocks will handle writing error messages based on what was written to the output files and clean up any temp files.

## Which Method to Use?

I haven’t given up on invoke-sqlcmd/invoke-sqlcmd2, but what I have done is come with a general rule on when to use one or the other. If I need to run a Powershell command and send the data to SQL or if I need to run a SQL command and send the data to Powershell I’ll use invoke-sqlcmd or some variation thereof. If just need to run a SQL query or produce an output file from a SQL query without a need to use the data in Powershell I’ll use sqlcmd.exe. You think of it like this my sqlcmd.exe based scripts are used for one-way operations, run a query which changes data or produces a file.


## Comments

**Justin Dearing November 12, 2012, 8:38 am**:
Chad,

Glad to see this, and glad to see you will continue to maintain Invoke-Sqlcmd.

Do you have a private scm for maintaining your own version of these tools, or do you just use poshcode as a poor mans scm.

Justin

**Chad Miller November 12, 2012, 9:02 am**:
I use Mercurial locally for scripts, but nothing external other than PoshCode.

**Boris November 13, 2012, 9:16 am**:
Nice post. There is still use for sqlcmd.exe.

I ran into an issue with invoke-sqlcmd and script execution. When trying to run a script (500 MB) which held data and ddl for a database, it threw a “System.OutOfMemoryException”. The machine I tried executing the script on had 3GB of memory at it’s disposal. With SSMS throwing an error as well when trying to open the .sql file, I thought of one more method of executing the script.
I tried sqlcmd.exe. After at least half an hour of churning through the script the database was successfully restored. Long live sqlcmd! 🙂

**Boris November 13, 2012, 9:22 am**:
Pardon my spelling in the above post.
Also forgot to mention I was working with SQL Server 2008 and Windows server 2003.

**Chad Miller November 13, 2012, 9:34 am**:
Good point, I would agree there are bugs in invoke-sqlcmd that don’t exist in sqlcmd.exe. The timeout settings issue is the biggest one that comes to mind and I’m still not sure it’s fixed in 2008 and 2008 R2: https://connect.microsoft.com/SQLServer/SearchResults.aspx?SearchQuery=invoke-sqlcmd
