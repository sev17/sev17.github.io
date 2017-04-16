title: SQL Agent Visual Job Schedule Viewer
link: http://sev17.com/2010/05/15/sql-agent-visual-job-schedule-viewer/
author: Chad Miller
description: 
post_id: 10220
created: 2010/05/15 14:54:18
created_gmt: 2010/05/15 18:54:18
comment_status: open
post_name: sql-agent-visual-job-schedule-viewer
status: publish
post_type: post

# SQL Agent Visual Job Schedule Viewer

If you have experience with the SQL Server Agent you quickly realize the difficulty in managing job schedules. On a busy server with many SQL jobs scheduled to run throughout the day at various times finding clashing job schedules and long running jobs is hard to do using the native SQL Server tools. 

To address this issue I’ve been looking for a simple way to visualize the SQL Agent job schedule information and I was just about to write my own tool before finding a very nice product called [SQLJobVis](http://sqlsoft.co.uk/sqljobvis.php) from [SQLSoft](http://sqlsoft.co.uk/). Using SQLJobVis I was able to quickly see jobs that were long running and jobs with overleaping schedules:

![SQLjobvis](http://images.sev17.com/SQLjobvis_thumb.jpg)

SQLJobVis  is a great example of how data visualization in the right way makes finding information much easier. At this point, the utility is freely available from SQLSoft and they are just  looking feedback and feature requests. So, give SQLJobVis a try and be sure to let them know if you find the tool useful.

## Comments

**[Will Alber](#150 "2010-06-11 15:21:35"):** Hi there - and thanks for blogging on SQLjobvis - I'm glad you're finding it useful and I'm very grateful for the positive feedback, always good to see our software being used. My intention is to keep SQLjobvis free - whether at some point a 'pro' version comes along or not is down to my getting enough free time to dedicate to the software, however there'll always be a free version and I certainly expect to be increasing the functionality well into the future. A soon-to-be-released version adds tabbed window support so you can have you job viewers maximized and still be able to navigate between servers with ease. Any and all feedback is accepted with pleasure - anything that anyone has to suggest, or any bug reports can only go into making SQLjobvis a better tool for everyone! Thanks again, and we look forward to hearing from you! Will Alber SQLsoft

**[Thomas Ash](#151 "2011-04-19 08:09:26"):** I've loaded SQLJOBVIS and find it very useful but I'm not sure of the significance of a clashing job or the definition of a clashing job. While SQLJOBVIS higlights clashing jobs I see multiple jobs running in the same time slot that aren't highlighted as clashing. So what is a clashing job? Love the tool as it was a great aid in rearranging my job schedules to obtain much greater efficiency.

