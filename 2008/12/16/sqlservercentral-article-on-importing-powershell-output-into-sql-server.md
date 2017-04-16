title: SQLServerCentral Article on Importing Powershell Output into SQL Server
link: http://sev17.com/2008/12/16/sqlservercentral-article-on-importing-powershell-output-into-sql-server/
author: Chad Miller
description: 
post_id: 9930
created: 2008/12/16 08:09:00
created_gmt: 2008/12/16 12:09:00
comment_status: open
post_name: sqlservercentral-article-on-importing-powershell-output-into-sql-server
status: publish
post_type: post

# SQLServerCentral Article on Importing Powershell Output into SQL Server

The question of how to load Powershell output into SQL Server has been frequently asked in Powershell forums. The article, 

[Loading Data With Powershell](http://www.sqlservercentral.com/articles/powershell/65196/) is my attempt to answer the question by providing three methods you can use to load the output of any Powershell command into a SQL Server table.

The methods are:

  1. Export-CSV/BULK INSERT
  2. XML/XMLBulkLoad
  3. DataTable/SQLBulkCopy

See [the article](http://www.sqlservercentral.com/articles/powershell/65196/) for details