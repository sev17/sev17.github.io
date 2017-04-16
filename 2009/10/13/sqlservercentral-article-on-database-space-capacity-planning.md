title: SQLServerCentral Article on Database Space Capacity Planning
link: http://sev17.com/2009/10/13/sqlservercentral-article-on-database-space-capacity-planning/
author: Chad Miller
description: 
post_id: 9984
created: 2009/10/13 10:25:00
created_gmt: 2009/10/13 14:25:00
comment_status: open
post_name: sqlservercentral-article-on-database-space-capacity-planning
status: publish
post_type: post

# SQLServerCentral Article on Database Space Capacity Planning

I wrote an article for SQLServerCentral entitled "[Database Space Capacity Planning](http://www.sqlservercentral.com/articles/powershell/68011/)" that demonstrates a database and volume (disk) capacity planning solution I use in my production environment. Some highlights of the solution: 

  * Powershell is used to collect database and volume space information from a list of SQL Servers, which is then loaded into a consolidated reporting database
  * A series of queries are used to calculate a days remaining metric i.e. the number of days until volume or database file runs out of space.
  * Uses SQL Server Reporting Services (SSRS)  to provide self-service reports
See the [full article](http://www.sqlservercentral.com/articles/powershell/68011/) for details.