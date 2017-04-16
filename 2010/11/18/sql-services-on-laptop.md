title: SQL Services on Laptop
link: http://sev17.com/2010/11/18/sql-services-on-laptop/
author: Chad Miller
description: 
post_id: 10464
created: 2010/11/18 20:44:44
created_gmt: 2010/11/19 01:44:44
comment_status: open
post_name: sql-services-on-laptop
status: publish
post_type: post

# SQL Services on Laptop

I run several SQL Server instances on my laptop, however I’ll keep the services shutdown to conserve resources and then start up the instance when needed. Doing this type of stuff through the GUI results in several clicks

  1. Right click on SQL instance in SSMS 
  2. Select Service Control >> Start 
  3. Then you’re of course prompted for UAC 
  4. A second prompt asking if you really want to Start SQL Server 

This is kind of a pain and much easier to do through a simple PowerShell script called Start-Sql
    
    
    Start-Process powershell.exe -argumentlist '-noprofile -command get-service -Name "MSSQL`$R2" | ? {$_.Status -eq "Stopped"} | Start-Service' -verb runas

The script starts a SQL Server service named “MSSQL$R2” using the the cmdlets Get-Service and Start-Service. A little trick of using start-process with the –verb runas command kicks of a powershell host as administrator. **Edit: Because SQL Agent and other dependent services must stopped, need -force switch. **To stop service the service I’ll use a stop-sql script:   

    
    
    Start-Process powershell.exe -argumentlist '-noprofile -command get-service -Name "MSSQL`$R2" | ? {$_.Status -eq "Running"} | Stop-Service -Force' -verb runas

Note: I’ve hardcoded the instance service name so I don’t have to specify a parameter each time. You’ll need to change “MSSQL$R2” to your instance.

## Comments

**[Paul Hiles](#187 "2010-11-19 08:22:56"):** Great tip

**[Matt Velic](#188 "2010-11-24 08:10:09"):** Nice, I'm going to have to give this a try, because it kind of annoys me to turn them on and off in the SQL Services GUI.

**[Davepen](#189 "2011-01-18 23:21:29"):** I do the same on my laptop (only starting services when I need them). What are the advantages of this over a cmd file that executes the appropriate NET START/STOP commands?

**[Chad Miller](#190 "2011-01-19 06:58:08"):** A couple of advantages. First the PowerShell version checks whether or not the service is running before attempting to start or stop and second the script invokes run as administrator which takes care of the UAC in Vista/Win7.

