title: Give Me a Coconut and Six Months
link: http://sev17.com/2009/06/13/give-me-a-coconut-and-six-months/
author: Chad Miller
description: 
post_id: 9961
created: 2009/06/13 10:58:00
created_gmt: 2009/06/13 14:58:00
comment_status: open
post_name: give-me-a-coconut-and-six-months
status: publish
post_type: post

# Give Me a Coconut and Six Months

[Tim Benninghof](http://vsteamsystemcentral.com/cs21/blogs/timbenninghoff/archive/2009/06/07/islands-with-wifi.aspx) tagged me with the question.

"So You’re On A Deserted Island With WiFi and you’re still on the clock at work. Okay, so not a very good situational exercise here, but let’s roll with it; we’ll call it a virtual deserted island. Perhaps what I should simply ask is if you had a month without any walk-up work, no projects due, no performance issues that require you to devote time from anything other than a wishlist of items you’ve been wanting to get accomplished at work but keep getting pulled away from I ask this question: what would be the top items that would get your attention?"

Although blog tagging reminds me of the chain emails I tell my mother to stop forwarding to everyone, the question gives me a chance to jot down a few items from my Powershell projects list.

  1. Create a slick looking DBA dashboard and develop it using Powershell with WPF and [Powerboots](http://powerboots.codeplex.com/). Like most DBAs, money for tools isn't available. This will make a quick way to visual performance immediately.

  2. Create a [SQLPSX](http://sqlpsx.codeplex.com/) PowerPack for [PowerGUI](http://www.powergui.org/). I have yet to try PowerGUI, but moving some scripts into a clickable UI seems like it would be useful.

  3. Create a Powershell provider for SSIS. The provider will allow navigation of an MSDB package store like a drive. You'll be be able to list and copy SSIS packages between servers.  Moving SSIS packages between servers in bulk is painful. As part of SQLPSX I created a set of Powershell scripts to do this task, implementing a provider will take the scripts one step further.

  4. Re-write [SQLPSX](http://sqlpsx.codeplex.com/) for Powershell V2 and implement help files, parameter sets, SQL authentication. As I try to drive adoption of Powershell with DBAs, implementing things like native help that is available in Powershell V2 advanced functions will make working with Powershell and SQL even easier.

  5. Update [PoshRSS](http://poshrss.codeplex.com/) to use the [PowershellRSS](http://www.powershelltoys.com/powershellrss.aspx), so I can deliver a better no cost monitoring solution for my web and application peers.

My job is mixture of DBA and manager. As a DBA, I don't consider myself to be a developer, but I'll write few lines of Powershell or C# code to create something that will save time or money. As a manager, cost control, efficiency, doing more with less, is the mantra being delivered across every company. Today it is more important than ever to be productive, think about how you can get rid of low value work and measurably show expense reductions. Let's face it, a lot of us must fill out a performance appraisal at least once a year. Quantifying your contribution towards cost-savings is one way to keep you listed as a top performer.