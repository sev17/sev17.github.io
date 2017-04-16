title: Internet Explorer Automation with WatIN
link: http://sev17.com/2009/09/01/internet-explorer-automation-with-watin/
author: Chad Miller
description: 
post_id: 9977
created: 2009/09/01 10:14:00
created_gmt: 2009/09/01 14:14:00
comment_status: open
post_name: internet-explorer-automation-with-watin
status: publish
post_type: post

# Internet Explorer Automation with WatIN

At my workplace I use an IT Service Desk application, called well, "Service Desk" from CA. The system is web-based and provides various queues for Change Orders, Requests and Incidents. Service Desk is used as used as workflow system instead of email. I'm sure this is a pretty common practice for many large IT shops, although they may use a different application. The Service Desk application is one the applications I like to keep open all day, however there is a 60 minute inactivity timeout. By doing any activity within a 60 minute timeframe you avoid timing out, even something simple like clicking an update count button.

![](http://images.sev17.com/ServiceDesk.jpg)

So, I thought I would create a PowerShell that would click an update count button every 50 mintues. I've done some scripting using [Internet Explorer COM automation](/2009/02/automating-mom-20002005-report-generation/) and PowerShell. This involves creating an  InternetExplorer.Application object and reading through the page source to find the HTML elements I want to select. I soon discovered issues pragmatically getting to the button through this technique due to Service Desk's heavy used of nested frames within nested frames. I decided to go another route using [WaitIN](http://watin.sourceforge.net/), [WaitIN Recorder](http://watintestrecord.sourceforge.net/) and PowerShell.

[Joel Bennet](http://huddledmasses.org/) has a post on [Using PowerShell and WatiN](http://huddledmasses.org/using-powershell-and-watin-powerwatin/), which I found helpful. Although I did not use the functions he created, his post provides a quick introduction to PowerShell and WaitIN.  Running WaitIN requires starting PowerShell in STA mode, so this is a PowerShell V2 only script. To start PowerShell in STA mode run:

powershell.exe -STA

### Getting Started

  1. Download [WatIN](http://watin.sourceforge.net/)
  2. Install [WatIN Recorder](http://watintestrecord.sourceforge.net/)

Usually working with PowerShell you'll create an object and explore it's property and methods, however in this case trying to find the button name was a little difficult. This is where WatIN Recorder helps out. After you've installed WatIN Recorder, run as a Administrator and navigate to the URL you want to automate:

![](http://images.sev17.com/watinRecorder.jpg)

Next click the record button and click the HTML element you want to automate. Then stop the WatIN recorder and click copy code to clipboard icon. This will produce some C# code that just needs to be translated into PowerShell:

// Windows WatiN.Core.IE window = **new** WatiN.Core.IE(); // Frames Frame frame_sd_scoreboard = window.Frame(Find.ByName("sd") && Find.ByName("scoreboard")); // Model Element __imgBtn0_button = frame_sd_scoreboard.Element(Find.ByName("imgBtn0_button")); // Code __imgBtn0_button.Click(); window.Dispose(); 

So, I now know the name of the button and that it is 3 frames deep. A little WatIN object exploration later, I came up with the follow script, which clicks a button every 50 mintues.

#Requires -version 2.0 #powershell.exe -STA **[Reflection.Assembly]**::LoadFrom( "$ProfileDirLibrariesWatiN.Core.dll" ) | **out-null** $ie = **new-object** WatiN.Core.IE("<https://sd.acme.com/CAisd/pdmweb.exe>") $scoreboard = $ie.frames | **foreach** {$_.frames } | **where** {$_.name -**eq** 'sd'} |  **foreach** {$_.frames } | **where** {$_.name -**eq** 'scoreboard'} $button = $scoreboard.Element("imgBtn0_button") **while** ($true) { $button.Click() #Sleep for 50 minutes **[System.Threading.Thread]**::Sleep(3000000) }