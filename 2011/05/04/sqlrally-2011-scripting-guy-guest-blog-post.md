title: SQLRally 2011 Scripting Guy Guest Blog Post
link: http://sev17.com/2011/05/04/sqlrally-2011-scripting-guy-guest-blog-post/
author: Chad Miller
description: 
post_id: 10636
created: 2011/05/04 10:03:13
created_gmt: 2011/05/04 14:03:13
comment_status: open
post_name: sqlrally-2011-scripting-guy-guest-blog-post
status: publish
post_type: post

# SQLRally 2011 Scripting Guy Guest Blog Post

Ed Wilson ([Blog](http://technet.microsoft.com/en-us/scriptcenter/default.aspx)|[Twitter](http://twitter.com/scriptingguys/)) aka Scripting Guy has series of SQL Server related posts the week of  May 2nd 2011 including my guest blog post. The post, [Use ACE Drivers and PowerShell to Talk to Access and Excel](http://blogs.technet.com/b/heyscriptingguy/archive/2011/05/04/use-ace-drivers-and-powershell-to-talk-to-access-and-excel.aspx), demonstrates querying Excel and Access files from PowerShell and loading the data into a SQL Server table. There several ways to get data from Excel and Access, but I find using ADO.NET to be the most straight forward. An important consideration when using ADO.NET against Excel and Access files is selecting the right OLE DB drivers. In the post I talk about using Access Control Entry (ACE) drivers.  ACE is completely free, and it even includes a 64-bit version. For SQL Server professionals having a 64-bit driver for Excel and Access is a big deal as ACE's predecessor, JET only supported x86. ACE is included with Office 2007 or higher and Office 2010 has a 64-bit version. If you don't have Office or you're installing on a server go to [Microsoft Access Database Engine 2010 Redistributable](http://www.microsoft.com/downloads/en/details.aspx?FamilyID=c06b8369-60dd-4b64-a44b-84b371ede16d&displaylang=en), and download AccessDatabaseEngine.exe or AccessDatabaseEngine_x64.exe, depending on your operating system. I've mentioned this in the post, but I think this is a key takeaway which I'll restate--When you have **ACE drivers, there is no reason to use the old deprecated JET drivers—even for older versions of Microsoft Access and Excel.** A common mistake I see, even with seasoned developers, is to drop to JET for .mdb and .xls files when you don’t need to. I've even made this mistake myself. I found [a helpful blog post on MSDN](http://blogs.msdn.com/b/psssql/archive/2010/01/21/how-to-get-a-x64-version-of-jet.aspx) from the CSS SQL Server Engineers that talks about different data providers and discusses a migration strategy. My guest blog post [Use ACE Drivers and PowerShell to Talk to Access and Excel](http://blogs.technet.com/b/heyscriptingguy/archive/2011/05/04/use-ace-drivers-and-powershell-to-talk-to-access-and-excel.aspx) doesn't delve into other uses of the ACE driver including working with delimited text files which I'll blog about in a future post.

## Comments

**[kevin lan](#248 "2011-05-25 22:17:31"):** hi chad, when i used this script , i came cross a problem "Exception calling "Fill" with "1" argument(s): "Query must have at least one destination field." the excel file is empty. would you help me how to fix it. thanks

**[Chad Miller](#249 "2011-05-25 22:28:22"):** I can try, but I need more information or even a test file. What version of Excel? Are you specifying a query? Are you trying to query an empty Excel file?

**[kevin lan](#250 "2011-05-25 22:31:06"):** i want to catch the sql errorlog by using powershell script. sometimes the errorlog csv file is empty. here is my script: $10minutes = new-timespan -Minutes 10 $10minutesDiff=(get-date) - $10minutes Get-SqlErrorLog "SZPC750GMorningStar" $10minutesDiff |export-csv d:/sqlerrorlog.csv -NoTypeInformation -Force $filepath = "d:" $connString = "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=`"$filepath`";Extended Properties=`"text;HDR=yes;FMT=Delimited`";" $qry = 'select * from [sqlerrorlog.csv]' $conn = new-object System.Data.OleDb.OleDbConnection($connString) $conn.open() $cmd = new-object System.Data.OleDb.OleDbCommand($qry,$conn) $da = new-object System.Data.OleDb.OleDbDataAdapter($cmd) $dt = new-object System.Data.dataTable $null = $da.fill($dt) #$dataTable = Get-SqlData "SZPC750GMorningStar" Monitoring "SELECT top 10 * FROM dbo.DimDate" $connectionString = "Data Source=SZPC750GMorningStar;Integrated Security=true;Initial Catalog=Monitoring;" $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $connectionString $bulkCopy.DestinationTableName = "SqlErrorLog" $bulkCopy.WriteToServer($dt) thanks

**[Chad Miller](#251 "2011-05-26 12:45:36"):** One thing you could do is check your file is greater than 0 bytes: if ($filepath).length -gt 0 { } Another alternative which actually is easier. If your Get-SqlErrorLog function uses SMO or xp_readerrorlog it already returns an array of type dataRow, so you don't need to convert to CSV and read CSV you just do this: [reflection.assembly]::LoadWithPartialName("Microsoft.SqlServer.Smo") > $null $server = new-object ("Microsoft.SqlServer.Management.Smo.Server") 'Z002SQLEXPRESS' $dt = $server.ReadErrorLog(0) $connectionString = “Data Source=SZPC750GMorningStar;Integrated Security=true;Initial Catalog=Monitoring;” $bulkCopy = new-object (“Data.SqlClient.SqlBulkCopy”) $connectionString $bulkCopy.DestinationTableName = “SqlErrorLog” $bulkCopy.WriteToServer($dt)

**[kevin lan](#252 "2011-05-26 18:44:34"):** chad,thanks for you kind help. i encountered another question $10minutes = new-timespan -Minutes 10 $10minutesDiff=(get-date) - $10minutes $dt = Get-SqlErrorLog "SZPC750GMorningStar" $10minutesDiff $connectionString = “Data Source=SZPC750GMorningStar;Integrated Security=true;Initial Catalog=Monitoring;” $bulkCopy = new-object (“Data.SqlClient.SqlBulkCopy”) $connectionString $bulkCopy.DestinationTableName = “SqlErrorLog” if $dt.table(0).count>0 $bulkCopy.WriteToServer($dt) if there hasn't any errorlog in the time, it will raise an error. Multiple ambiguous overloads found for "WriteToServer" and the argument count: "1". thanks

**[Chad Miller](#253 "2011-05-26 19:24:28"):** Keep in mind PowerShell uses -gt instead of >. Also an array of datarows does not have a table property. What's the definition of your Get-SqlErrorLog function? The check should be written like this: if ($dt -ne $null) { $bulkcopy.writetoserver($dt) } or since you are just checking for NULL: if ($dt -ne $null) { $bulkcopy.writetoserver($dt) }

**[Aaron](#319 "2013-06-12 14:01:33"):** How would you add instance name and bulkcopy instance name and errorlog contents into destination table?

**[Chad Miller](#324 "2013-06-21 19:19:25"):** The easiest thing would be to use T-SQL with xp_readerrorlog into a temp table then add the two elements in select statement. From there run query with invoke-sqlcmd into the $dt variable.

