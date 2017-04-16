title: Using RSS for System Administration
link: http://sev17.com/2008/11/05/using-rss-for-system-administration/
author: Chad Miller
description: 
post_id: 9924
created: 2008/11/05 09:48:00
created_gmt: 2008/11/05 13:48:00
comment_status: open
post_name: using-rss-for-system-administration
status: publish
post_type: post

# Using RSS for System Administration

Using RSS for system administration is a novel but extremely useful technique to consolidate information across many servers. I'm using RSS Server software [PoshRSS](http://www.codeplex.com/PoshRSS) and [xSQLSoftware RSS Reporter](http://www.xsqlsoftware.com/Product/Sql_Server_Rss_Reporter.aspx) along with my favorite client-side aggregator, [RSSOwl](http://www.rssowl.org) to pull together: 

  * Databases missing backups
  * SQL Replication Issues including down or latency problems
  * SQL Error Log for Error Messages
  * Open Microsoft Operation Manager Alerts
  * Open Help desk tickets
  * Drives with less than 20% free space
  * Uptime
RSS for system administration is a better solution than building scripts which generate yet another report. There are several problems with report and email based approach. 
  * The report information is static only reflecting data at the time report was generated. RSS Server software like PoshRSS present real-time information when you hit the RSS feed link the Powershell script/command executes.
  * Each report you create you need to collect the data, possibly store, format and deliver usually via email. One of the great things about RSS is the collection, format and delivery are handled for you. RSS is versatile you can consume the data in a number of ways including via an aggregator, Internet Explorer, Outlook, SharePoint. You can even convert RSS to email if you really wanted to.
  * Each report or email requires that you open the report and view it, which is just as inefficient as viewing a website instead of subscribing to a RSS feed of a website when all you care about is new information.
Not only can you monitor systems with RSS and you can also consolidate information from other monitoring systems. I have two help desk systems and Microsoft Operations Manager Alerts to monitor. I've setup 3 RSS feeds which check for "new" information every one minute. This provides a single portal, my RSS aggregator, to view instead of having to log into 3 systems. The important thing to keep in mind is that the RSS feeds I create are all by based on exception conditions so the volume of data is not overwhelming. In addition the frequency which the data is updated is controlled by the client-side aggregator. For some things like help desk tickets, I'll check for new information every minute. For things like backup information once every 24 hours is sufficient. For more information, here's a link to a short presentation I gave about using RSS for system administration: [RSS Presentation](/2008/10/tampa-bay-sql-user-group-presentation-oct-21-2008/) If you're used to subscribing to a bunch of blogs with an RSS aggregator, you'll probably find applying the same RSS concept to system administration data collection as you do for blogs to be just as addicting and time saving