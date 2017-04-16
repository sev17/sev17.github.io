title: Gettting and Setting SQL Data
link: http://sev17.com/2009/05/31/gettting-and-setting-sql-data/
author: Chad Miller
description: 
post_id: 9959
created: 2009/05/31 21:48:00
created_gmt: 2009/06/01 01:48:00
comment_status: open
post_name: gettting-and-setting-sql-data
status: publish
post_type: post

# Gettting and Setting SQL Data

Many of the Powershell scripts I write either retrieve data from SQL Server or execute a nonquery i.e. delete, insert, update against SQL Server. I find myself reusing a couple of simple functions often enough that posted Get-SqlData and Set-SqlData functions to [poshcode](http://poshcode.org/) as [LibrarySqlData.ps1](http://poshcode.org/1139):

####################### **function** Get-SqlData { **param**(**[string]**$serverName=$(**throw** 'serverName is required.'), **[string]**$databaseName=$(**throw** 'databaseName is required.'), **[string]**$query=$(**throw** 'query is required.')) **Write-Verbose** "Get-SqlData serverName:$serverName databaseName:$databaseName query:$query" $connString = "Server=$serverName;Database=$databaseName;Integrated Security=SSPI;" $da = **New-Object** "System.Data.SqlClient.SqlDataAdapter" ($query,$connString) $dt = **New-Object** "System.Data.DataTable" **[void]**$da.fill($dt) $dt } #Get-SqlData ####################### **function** Set-SqlData { **param**(**[string]**$serverName=$(**throw** 'serverName is required.'), **[string]**$databaseName=$(**throw** 'databaseName is required.'), **[string]**$query=$(**throw** 'query is required.')) $connString = "Server=$serverName;Database=$databaseName;Integrated Security=SSPI;" $conn = **new-object** System.Data.SqlClient.SqlConnection $connString $conn.Open() $cmd = **new-object** System.Data.SqlClient.SqlCommand("$query", $conn) **[void]**$cmd.ExecuteNonQuery() $conn.Close() } #Set-SqlData

To use soure the  . ./LibrarySqlData.ps1 file. And here are a couple examples:

Get-SqlData 'Z002SQLEXPRESS' 'master' 'SELECT @@servername' Set-SqlData 'Z002SQLEXPRESS' 'pubs' "update authors set au_lname = 'White' WHERE au_lname = 'White'"