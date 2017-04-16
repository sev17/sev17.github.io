title: Using Powershell to Import Excel file into SQL Server
link: http://sev17.com/2009/05/12/using-powershell-to-import-excel-file-into-sql-server/
author: Chad Miller
description: 
post_id: 9956
created: 2009/05/12 20:51:00
created_gmt: 2009/05/13 00:51:00
comment_status: open
post_name: using-powershell-to-import-excel-file-into-sql-server
status: publish
post_type: post

# Using Powershell to Import Excel file into SQL Server

This is a quick script which demonstrates how easy it is to import an Excel file into a SQL Server table using Powershell. The script is posted on [PoshCode ](http://poshcode.org/1098)also:

#Change these settings as needed $filepath = 'G:PowershellvTSQLbackupset.xlsx' #Comment/Uncomment connection string based on version #Connection String for Excel 2007: $connString = "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=`"$filepath`";Extended Properties=`"Excel 12.0 Xml;HDR=YES`";" #Connection String for Excel 2003: #$connString = "Provider=Microsoft.Jet.OLEDB.4.0;Data Source=`"$filepath`";Extended Properties=`"Excel 8.0;HDR=Yes;IMEX=1`";" $qry = 'select * from [backupset$]' $sqlserver = 'Z002SQLEXPRESS' $dbname = 'SQLPSX' #Create a table in destination database with the with referenced columns and table name. $tblname = 'ExcelData_fill' ####################### **function** Get-ExcelData { **param**($connString, $qry='select * from [sheet1$]') $conn = **new-object** System.Data.OleDb.OleDbConnection($connString) $conn.open() $cmd = **new-object** System.Data.OleDb.OleDbCommand($qry,$conn) $da = **new-object** System.Data.OleDb.OleDbDataAdapter($cmd) $dt = **new-object** System.Data.dataTable **[void]**$da.fill($dt) $conn.close() $dt } #Get-ExcelData ####################### **function** Write-DataTableToDatabase { **param**($dt,$destServer,$destDb,$destTbl) $connectionString = "Data Source=$destServer;Integrated Security=true;Initial Catalog=$destdb;" $bulkCopy = **new-object** ("Data.SqlClient.SqlBulkCopy") $connectionString $bulkCopy.DestinationTableName = "$destTbl" $bulkCopy.WriteToServer($dt) }# Write-DataTableToDatabase ####################### $dt = **Get-ExcelData** $connString $qry **Write-DataTableToDatabase** $dt $sqlserver $dbname $tblname