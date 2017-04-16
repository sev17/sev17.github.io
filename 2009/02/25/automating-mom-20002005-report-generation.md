title: Automating MOM 2000/2005 Report Generation
link: http://sev17.com/2009/02/25/automating-mom-20002005-report-generation/
author: Chad Miller
description: 
post_id: 9944
created: 2009/02/25 14:05:00
created_gmt: 2009/02/25 18:05:00
comment_status: open
post_name: automating-mom-20002005-report-generation
status: publish
post_type: post

# Automating MOM 2000/2005 Report Generation

Every month I go through a  capacity planning exercise looking at things like processor, memory, storage and backup success rates. As part of the review I'll run two simple Microsoft Operations Manager 2005 SQL Server Reporting Services reports for the past 30 day period:

  * Microsoft Operations Manager Reporting > Microsoft Windows Base Operating System > Performance History > Performance History Processor-Percent Processor Time
  * Microsoft Operations Manager Reporting > Microsoft Windows Base Operating System > Performance History > Performance History Paging File-Percent Usage

The reports aren't perfect as the data summarized into hourly data points over the 30 day period you loose some details, but they work remarkably well at spotting performance trends and developing a performance baseline for a particular server over time. This is especially true with database servers. If the processor usage on the chart increases by a double digit percentage, usually this is a  direct result of a sudden event, meaning one day it suddenly jumps. In the database area by using additional tools we can often tie the performance change directly to configuration changes, new code deployments, increased usage, or business processing (month-end, quarter-end, etc.). When I notice sudden performance changes I will research why the change occurred, if a cause can be identified and usually it can, I will work to remediate the SQL code, change a configuration, order a larger server or simply continue to observe the performance.

To generate the MOM reports I created  a Powershell script .Over the past several years the Powershell script is my third version of a MOM report/IE automation script, the first two version were Perl scripts using Win32::SAM or Win32::IEAutomation. There are better ways at automating MOM reports, for instance it would be much easier to create your own customized version of the report based on the out of the box report and then create subscriptions in SQL Server Reporting Services to deliver the reports on a monthly basis. There are also some nice frameworks like WASP and WAITN which should make IE automation in Powershell easier. Again using SQL Server Reporting Services direclty is the best approach, but hey that wouldn't give me an excuse to write a Powershell script to automate IE to execute a report :). Whenever I have the time I will create the customized report I need, but for now this script works for me so I continue to use it.

A few of notes about the Powershell script:

  * Uses COM based InternetExplorer.Application
  * To determine the report control names I looked at the page source i.e. view page source and then had to play around with the control names and figuring out which methods to call to automate selecting from drop down list sand clicking button controls.
  * Uses a simple server name comma 0 or 5 text file as input, where 0 is SQL 2000 group and 2005 is SQL 2005 group

#get-content servers.txt | % {$server = $_.split(",")[0]; $serverType = $_.split(",")[1]; ./mom.ps1 $server $serverType '5' '2008'} **param** ($server=$(**throw** '`$server is required.'),$serverGroup,$month=$(**throw** '`$month is required.'),$Year=$(**throw** '`$year is required.')) **function** runReport { **param** ($server,$group,$beginDT,$endDT,$url) $ie = **new-object** -com "InternetExplorer.Application" $ie.navigate($url) $ie.visible = $true **Write-Host** "$server" **while** ($ie.Busy) { **[System.Threading.Thread]**::Sleep(500) } $doc = $ie.Document $begin = $doc.getElementByID("ctl143_ctl00_ctl03_txtValue") $begin.value = $beginDT $end = $doc.getElementByID("ctl143_ctl00_ctl05_txtValue") $end.value = $endDT $groupList = $doc.getElementByID("ctl143_ctl00_ctl07_ddValue") $groupValue = $groupList | **where** {$_.text -**eq** $group } $groupList.value = $groupValue.value $groupList.FireEvent('onchange') **while** ($ie.Busy) { **[System.Threading.Thread]**::Sleep(500) } #Sleep for 10 seconds to allow postback to fully refresh **[System.Threading.Thread]**::Sleep(10000) $computerList = $doc.getElementByID("ctl143_ctl00_ctl09_ddValue") $computerValue = $computerList | **where** {$_.text -**eq** "MyDomain$server" } $computerList.value = $computerValue.value **if** ($computerList.value -**ne** 0) { $viewReport = $doc.getElementByID("ctl143_ctl00_ctl00") $viewReport.Click() #Sleep for 5 minutes **[System.Threading.Thread]**::Sleep(300000) } } **switch** ($serverGroup) {