title: SQLServerCentral Article on Backup Monitoring and Reporting
link: http://sev17.com/2009/06/04/sqlservercentral-article-on-backup-monitoring-and-reporting/
author: Chad Miller
description: 
post_id: 9960
created: 2009/06/04 07:16:00
created_gmt: 2009/06/04 11:16:00
comment_status: open
post_name: sqlservercentral-article-on-backup-monitoring-and-reporting
status: publish
post_type: post

# SQLServerCentral Article on Backup Monitoring and Reporting

The article, [Backup Monitoring and Reporting](http://www.sqlservercentral.com/articles/Backup/66564/), demonstrates a SQL Server backup reporting solution I use in my production environment. Some highlights of the solution:

  * Powershell is used to collect backup information from a list of SQL Servers which is loaded into a consolidated reporting database
  * A series of queries are used to look for missing backups and report various backup completion statistics
  * Using SQL Server Reporting Services (SSRS) is quick way to provide self-service reports and several SSRS reports are included.

See [the full article](http://www.sqlservercentral.com/articles/Backup/66564/) for details.