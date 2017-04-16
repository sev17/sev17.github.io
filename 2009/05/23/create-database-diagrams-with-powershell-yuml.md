title: Create Database Diagrams with Powershell + yUML
link: http://sev17.com/2009/05/23/create-database-diagrams-with-powershell-yuml/
author: Chad Miller
description: 
post_id: 9958
created: 2009/05/23 16:44:00
created_gmt: 2009/05/23 20:44:00
comment_status: open
post_name: create-database-diagrams-with-powershell-yuml
status: publish
post_type: post

# Create Database Diagrams with Powershell + yUML

Powershell MVP, [Doug Finke](http://dougfinke.com/blog/) has an interesting [post](http://dougfinke.com/blog/index.php/2009/05/06/use-powershell-and-yuml-to-create-diagrams/) demonstrating how to create UML class digrams from Powershell using the website [yUML](http://yuml.me/). So, I thought it would be a fun exercise to create a UML class diagram of database tables. It might seem a little odd to represent database tables in UML, however it can done and a few commerical allow you to do so. Basically tables are represented as classes, columns as attributes, constraints and indexes are behaviors and foreign key relationships are associations. For a good overview of how to model databases as UML class diagrams see [Database Modelling in UML](http://www.methodsandtools.com/archive/archive.php?id=9) by By Geoffrey Sparks.

The yUML website generates UML through a simple syntax, where parameters are appended to the URL string ([See Doug's blog for an example](http://dougfinke.com/blog/index.php/2009/05/06/use-powershell-and-yuml-to-create-diagrams/)). The hard part for generating class digrams from databases is getting the meta data about SQL Server tables in a usable format. In fact, the T-SQL query is much larger than the acommpanying Powershell code and also took me longer to figure out. The query [sqlmeta.sql](http://cid-ea42395138308430.skydrive.live.com/self.aspx/Public/Blog/sqlmeta.sql) returns meta data about a SQL table and makes use of SQL 2005/2008 CTE's and XPath. A single XML document is returned with column, primary key, constraint, index, trigger, and relationship information. For example using the sample AdventureWorks database and the Sales.Store table as parameters the following XML is returned:

<root> <class> <table>Sales.Store</table> <columns> <column>PK CustomerID: int</column> <column>Name: nvarchar</column> <column>SalesPersonID: int</column> <column>Demographics: xml</column> <column>rowguid: uniqueidentifier</column> <column>ModifiedDate: datetime</column> </columns> <relations> <relation>[Sales.Customer]</relation> <relation>[Sales.SalesPerson]</relation> </relations> <operations> <operation>FK: FK_Store_Customer_CustomerID</operation> <operation>FK: FK_Store_SalesPerson_SalesPersonID</operation> <operation>Index: AK_Store_rowguid</operation> <operation>Index: IX_Store_SalesPersonID</operation> <operation>Index: PXML_Store_Demographics</operation> <operation>PK: PK_Store_CustomerID</operation> <operation>Trigger: iStore</operation> </operations> </class> </root>

The Powershell script, [yuml.ps1](http://cid-ea42395138308430.skydrive.live.com/self.aspx/Public/Blog/yuml.ps1) includes function to execute a query and return a DataTable called Get-SqlData:

**function** Get-SqlData { **param**(**[string]**$serverName=$(**throw** 'serverName is required.'), **[string]**$databaseName=$(**throw** 'databaseName is required.'), **[string]**$query=$(**throw** 'query is required.')) **Write-Verbose** "Get-SqlData serverName:$serverName databaseName:$databaseName query:$query" $connString = "Server=$serverName;Database=$databaseName;Integrated Security=SSPI;" $da = **New-Object** "System.Data.SqlClient.SqlDataAdapter" ($query,$connString) $dt = **New-Object** "System.Data.DataTable"