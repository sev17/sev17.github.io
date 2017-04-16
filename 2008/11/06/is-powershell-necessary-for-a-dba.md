title: Is PowerShell necessary for a DBA?
link: http://sev17.com/2008/11/06/is-powershell-necessary-for-a-dba/
author: Chad Miller
description: 
post_id: 9925
created: 2008/11/06 22:53:00
created_gmt: 2008/11/07 02:53:00
comment_status: open
post_name: is-powershell-necessary-for-a-dba
status: publish
post_type: post

# Is PowerShell necessary for a DBA?

I have been a SQL DBA for 9 years and in that time I have worked with many DBAs. Surprisingly only about 20% of them knew a dynamic programming language other than T-SQL. So, Is Powershell necessary for a DBA? Before we can answer this question we need to answer some more specific questions…

** **

**Why should I learn Scripting?**

** **

I consider scripting in Powershell, VBScript, or Perl when I need to pull data from many SQL Servers, or I need to implement a scriptable/repeatable solution which excludes the use of a GUI.  SQL Server Management Studio (SSMS), “the GUI,” does not scale well when you are working with hundreds of servers.

Why should I learn Powershell when I have T-SQL?

Some things T-SQL just cannot do or does not do very well. For example, to name a few tasks: enumerating AD groups, getting information from clusters, disk volumes, SANs, Windows services, getting NTFS/Share permissions, WMI, and monitoring SQL services. Furthermore, T-SQL is a single-server solution; you simply cannot write a T-SQL script that executes on multiple servers without using another tool or scripting language.

Why should I learn Powershell when I have VBScript? VBScript is a COM-based language in which you cannot use the .NET classes. From a DBA perspective, if you are using VBScript, more than likely you’ve used COM-based SQL-DMO. SQL-DMO was the preferred API for working with SQL Server for administrative tasks using script in SQL Server 7.0/2000. SQL Server 2005/2008 introduced a new .NET-only API called SMO. Because SMO is a .NET class, you cannot use it from VBScript. SMO can only be used from .NET languages, including Powershell. SQL-DMO is a legacy feature in which the new properties and methods introduced in SQL 2005/2008 are not accessible. The end of SQL-DMO is not bad thing, as I believe SMO is easier to use than SQL-DMO.

Why should I learn Powershell? Policy Based Management (PBM) and SQL Server 2008 Agent make use of Powershell. SQL Agent has a Powershell job command type that allows you to execute Powershell within a job. Of course you can do this in SQL 2000 and 2005 by calling powershell.exe with the -c switch, if you have Powershell installed on your SQL Server. Having Powershell on the SQL Server allows you to create jobs which load Powershell output into a SQL table.

Now we can answer the original question is Powershell necessary for a DBA? If you have not had any exposure to scripting before, I think you will find Powershell to be easier to master than a more verbose language like VBScript. And if you are a SQL DBA and you want to learn scripting, Powershell should be your first choice.