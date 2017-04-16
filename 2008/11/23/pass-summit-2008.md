title: PASS Summit 2008
link: http://sev17.com/2008/11/23/pass-summit-2008/
author: Chad Miller
description: 
post_id: 9926
created: 2008/11/23 11:56:00
created_gmt: 2008/11/23 15:56:00
comment_status: open
post_name: pass-summit-2008
status: publish
post_type: post

# PASS Summit 2008

I attended the PASS (Professional Association for SQL Server) Summit 2008 in Seattle, WA from November 17th through November 21st. Powershell made a showing with three Powershell focused presentations. There were two sessions by [Allen White](http://sqlblog.com/blogs/allen_white/default.aspx) and one by [Buck Woody](http://search.live.com/results.aspx?q=Buck Woody). I was able to attend one session from each speaker. All presentations filled the available seats along with a row of attendees standing in the back of the room. For DBAs working with Powershell this is a good indication of the high level of interest, and I would not be suprised to see a larger number of Powershell focused presentations next year. I would describe the sessions as beginner level Powershell introductions which was fine considering over half the audience members had not worked with Powershell, according to an informal poll conducted by the presenters. Neither speaker focused on the Microsoft SQL Server 2008 Powershell Provider and instead used native SMO to accomplish most scripting demonstrations. In the session presented by Allen there were a lot of questions about the SMO object model that made me think about the level abstraction needed over SMO to make Powershell easier for the DBA--I've got a lot more work to do on [SQL Server Powershell Extensions](http://www.codeplex.com/SQLPSX).

Allen and Buck did a great job showing some practical scripts and techniques for DBAs new Powershell. For experienced Powershell scripters there were a couple takeaways:

  * SQL 2008 implements a SMOExtended assembly and moved several classes from the base SMO assembly. This reminds me of the DMO and DMO2 implementations in 7.0 and 2000 respectively where newer capabilities were only available in DMO2.
  * The SQL Server Powershell host, sqlps.exe will be available as a separate download so you do not have to install SQL 2008 to get it.

After seeing many announcements about the Powershell roadmap from TechED EMEA 2008, I was hoping for some similar announcments about SQL Server 2008 Provider. Unfortunately there was no news related to enhancements of the SQL Server 2008 Provider. I guess there's always next year.