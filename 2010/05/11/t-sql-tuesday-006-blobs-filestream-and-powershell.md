title: T-SQL Tuesday #006 Blobs, FileStream and PowerShell
link: http://sev17.com/2010/05/11/t-sql-tuesday-006-blobs-filestream-and-powershell/
author: Chad Miller
description: 
post_id: 10162
created: 2010/05/11 16:07:03
created_gmt: 2010/05/11 20:07:03
comment_status: open
post_name: t-sql-tuesday-006-blobs-filestream-and-powershell
status: publish
post_type: post

# T-SQL Tuesday #006 Blobs, FileStream and PowerShell

In case you're wondering about the title [T-SQL Tuesday](http://tsql2sday.com/), it's a monthly collection of SQL Server related content where a bunch of bloggers contribute posts related a specific topic. This [month's topic hosted by Michael Coles is Large Object data types](http://sqlblog.com/blogs/michael_coles/archive/2010/05/03/t-sql-tuesday-006-what-about-blob.aspx) or also know as Blobs. In this blog post I'll demonstrate working with blob and filestream columns using PowerShell. 

## Blobs

Using the Knowledge Base article, [How to read and write a file to or from a BLOB column by using ADO.NET and Visual C# .NET](http://support.microsoft.com/kb/317016), as a guide I've loosely adapted the C# examples to PowerShell. I'm using the sample database AdventureWorks2008 which you can [download from CodePlex](http://msftdbprodsamples.codeplex.com/). _Note: traditional blob columns can be used in versions of SQL Server earlier than 2008 including SQL Server 2000_. Because we'll also be writing blobs we'll need to create a SQL table in the AdventureWorks2008 database as follows: 
    
    
    CREATE TABLE [Production].[ProductPhoto2](
    	[ProductPhotoID] [int] IDENTITY(1,1) NOT NULL,
    	[ThumbNailPhoto] [varbinary](max) NULL,
    	[ThumbnailPhotoFileName] [nvarchar](50) NULL)

First let's extract a bunch of images stored as blobs to the file system in a script called **sqlblob2file.ps1**. The script retrieves the first ten images and writes them to my _C:\Users\u00 directory_: 
    
    
    $server = "Z002\sql2k8"
    $database = "AdventureWorks2008"
    $query = "SELECT TOP 10 ThumbNailPhoto, ThumbnailPhotoFileName FROM Production.ProductPhoto"
    $dirPath = "C:\Users\u00"
    
    $connection=new-object System.Data.SqlClient.SQLConnection
    $connection.ConnectionString="Server={0};Database={1};Integrated Security=True" -f $server,$database
    $command=new-object system.Data.SqlClient.SqlCommand($query,$connection)
    $command.CommandTimeout=120
    $connection.Open()
    $reader = $command.ExecuteReader()
    while ($reader.Read())
    {
        $sqlBytes = $reader.GetSqlBytes(0)
        $filepath = "$dirPath{0}" -f $reader.GetValue(1)
        $buffer = new-object byte[] -ArgumentList $reader.GetBytes(0,0,$null,0,$sqlBytes.Length)
        $reader.GetBytes(0,0,$buffer,0,$buffer.Length)
        $fs = new-object System.IO.FileStream($filePath,[System.IO.FileMode]'Create',[System.IO.FileAccess]'Write')
    	$fs.Write($buffer, 0, $buffer.Length)
    	$fs.Close()
    }
    $reader.Close()
    $connection.Close()

Next we will write an image file on the file system to a SQL Server blob column in a script called **file2SqlBlob.ps1.** In this script we read the file into memory and write it to the blob column using an insert statement: 
    
    
    $server = "Z002\sql2k8"
    $database = "AdventureWorks2008"
    $query = "INSERT Production.ProductPhoto2 VALUES (@ThumbNailPhoto, @ThumbnailPhotoFileName)"
    $filepath = "C:\Users\u00\hotrodbike_black_small.gif"
    $ThumbnailPhotoFileName = get-childitem $filepath | select -ExpandProperty Name
    
    $connection=new-object System.Data.SqlClient.SQLConnection
    $connection.ConnectionString="Server={0};Database={1};Integrated Security=True" -f $server,$database
    $command=new-object system.Data.SqlClient.SqlCommand($query,$connection)
    $command.CommandTimeout=120
    $connection.Open()
    
    $fs = new-object System.IO.FileStream($filePath,[System.IO.FileMode]'Open',[System.IO.FileAccess]'Read')
    $buffer = new-object byte[] -ArgumentList $fs.Length
    $fs.Read($buffer, 0, $buffer.Length)
    $fs.Close()
    
    $command.Parameters.Add("@ThumbNailPhoto", [System.Data.SqlDbType]"VarBinary", $buffer.Length)
    $command.Parameters["@ThumbNailPhoto"].Value = $buffer
    $command.Parameters.Add("@ThumbnailPhotoFileName", [System.Data.SqlDbType]"NChar", 50)
    $command.Parameters["@ThumbnailPhotoFileName"].Value = $ThumbnailPhotoFileName
    $command.ExecuteNonQuery()
    
    $connection.Close()

## OpenRowset

While researching writing blob data I noticed T-SQL's OpenRowset supports a very simple way to load image data. This is documented in the Books Online topic, [OPENROWSET (Transact-SQL)](http://msdn.microsoft.com/en-us/library/ms190312.aspx). Here's an example that uses T-SQL to load a single image file into our ProductPhoto2 table: 
    
    
    USE AdventureWorks2008;
    INSERT INTO Production.ProductPhoto2 (ThumbNailPhoto, ThumbnailPhotoFileName)
       SELECT *, 'hotrodbike_black_small.gif' AS ThumbnailPhotoFileName
        FROM OPENROWSET(BULK N'C:\Users\u00\hotrodbike_black_small.gif', SINGLE_BLOB) AS ProductPhoto
    GO

It should be noted the sqlblob2file.ps1, file2sqlblob.ps1, OpenRowSet scripts will also work against filestreams columns. Filestreams support both the old T-SQL blob read/write syntax as well as the Win32 syntax which we will look at next. 

## FileStreams

Using the MSDN documentation, [FILESTREAM Data in SQL Server 2008 (ADO.NET)](http://msdn.microsoft.com/en-us/library/cc716724.aspx), as a guide I will loosely adapted the C# examples to PowerShell. We will again connect to the same AdventureWorks2008 database used in the blob demonstration. _Note: Filestream columns are only supported in SQL Server 2008 or higher. _There's a bit of setup required to enable filestream on a SQL Server 2008 or higher instance. The SQL Server installation provides the option to configure filestream on install. If you didn't turn on FileStream during setup, configuration is fairly simple and well documented in the Books Online topic, [Getting Started with FileStream](http://technet.microsoft.com/en-us/library/bb933995.aspx). Because we will also be writing data to a filestream we'll setup a separate database and table as follows: 
    
    
    CREATE DATABASE testFS
    GO
    ALTER database testFS
    ADD FILEGROUP FileStreamGroup
    CONTAINS FILESTREAM
    GO
    ALTER database testFS
    ADD FILE(NAME = testFS_FileStream, FILENAME = 'C:\Program Files\Microsoft SQL Server\MSSQL10.SQL2K8\MSSQL\DATA\testFS_FileStream.fs')
    TO FILEGROUP FileStreamGroup
    GO
    CREATE TABLE [dbo].[Document2](
    	[FileName] [nvarchar](50) NOT NULL,
    	[Document] [varbinary](max) FILESTREAM  NULL,
    	[rowguid] [uniqueidentifier] ROWGUIDCOL  NOT NULL UNIQUE DEFAULT NEWID()
    )

The following, script called **filestream2File.ps1** demonstrates how to read data from a filestream and write the bytes to a file on the file system: 
    
    
    $server = "Z002\sql2k8"
    $database = "AdventureWorks2008"
    $query = "SELECT TOP(10) Document.PathName(), GET_FILESTREAM_TRANSACTION_CONTEXT(), Title + FileExtension AS FileName FROM Production.Document WHERE FileExtension = '.doc'"
    $dirPath = "C:\Users\u00"
    
    $connection=new-object System.Data.SqlClient.SQLConnection
    $connection.ConnectionString="Server={0};Database={1};Integrated Security=True" -f $server,$database
    $connection.Open()
    $command=new-object system.Data.SqlClient.SqlCommand("",$connection)
    $command.CommandTimeout=120
    $tran = $connection.BeginTransaction([System.Data.IsolationLevel]'ReadCommitted')
    $command.Transaction = $tran
    $command.CommandText = $query
    $reader = $command.ExecuteReader()
    while ($reader.Read())
    {
        $path = $reader.GetString(0)
        [byte[]]$transactionContext = $reader.GetSqlBytes(1).Buffer
        $filepath = "$dirPath{0}" -f $reader.GetValue(2)
    
        $fileStream = new-object System.Data.SqlTypes.SqlFileStream($path,[byte[]]$reader.GetValue(1), [System.IO.FileAccess]'Read', [System.IO.FileOptions]'SequentialScan', 0)
        $buffer = new-object byte[] $fileStream.Length
        $fileStream.Read($buffer,0,$fileStream.Length)
        $fileStream.Close()
        [System.IO.File]::WriteAllBytes($filepath,$buffer)
    
    }
    
    $reader.Close()
    $tran.Commit()
    $connection.Close()

## Comments

**[Peter](#145 "2011-06-15 00:25:36"):** Love this article!! Only thing like it on the web. Very detailed, very helpful. In sqlblob2file.ps1, what is the purpose of the below line of code? (line 14) $sqlBytes = $reader.GetSqlBytes(0) When I try using the script, I get an error message saying "Exception calling "GetSqlBytes" with "1" argument(s): "Specified cast is not va lid."". If I simply comment out that line of code, everything works fine. Am I missing something?

**[Chad Miller](#146 "2011-06-15 07:18:26"):** Thanks Peter, Unfortunately I'm unable to reproduce the error you mention. I ran the sqlblob2file.ps1 code against the AdventureWorks database and it worked fine. As for the purpose, on line 16 , I define a byte array and part of the definition is the number of bytes obtained from $reader.GetSqlBytes(0)

**[Peter](#147 "2011-06-15 22:20:49"):** I figured out my error. I would normally not tell everyone and just go on my way, but I found this action of SQL Server very interesting. When I defined the table, I made the field a data type of varchar(max) NOT varbinary(max). Simple typo. Interesting enough all the code worked except for line 14 of sqlblob2file.ps1. I commented out line 14, and I was able to retrieve the file from SQL server. It was an executable that I had uploaded into SQL server. It worked after I extracted it from SQL Server. I am VERY surprised that it did not corrupt the file. Just thought I'd share.

**[Mike](#148 "2011-09-14 10:26:09"):** Peter, Great article. I'm not a .NET guy, but want to be a Powershell guy. I'm a SQL Server guy by trade. The code ran fine against my adventure works DB. I wanted to run it against a FileStream varbinary(max) column in my shop and it ran, kicked out the buffer size (I think that's what it is), but it just puts a System.Byte[] file out instead of the actual image file I'm trying to retrieve. Any ideas?

**[Chad Miller](#149 "2011-09-14 19:28:02"):** @Mike Is possible you don't have a FileStream and are working with a traditional blob? Have you tried using sqlblob2file.ps1?

**[Bill Chestnut](#300 "2013-01-17 16:40:45"):** This question is really only somewhat related to the article. It a more general problem, about BLOB storage used by Doc Mgmt Systems. Suppose I have two Word Docs: DocOne and DocTwo. Suppose that within the first one, I create a link to the second with a hyperlink, which of course specifies a full path to DocTwo. Now management installs a splendid new Doc Mgmt system that takes all the files and puts them in BLOBs in a database. Then it deletes all the files on the server. When DocOne is opened from a BLOB storage, it would still have the hyperlink reference to a file path for DocTwo that no longer exists. Broken hyperlink. How do they solve this problem with Doc Mgmt BLOB storage systems?

**[Chad Miller](#301 "2013-01-17 18:17:31"):** @Bill -- The code in this blog post won't help as it just demonstrates extracting and loading BLOBs. Seems like a tricky problem, I suppose you could re-extract the documents and either manually or through Office automation update links. The code in the post would help with the extract part and load part. I would also point that for most Doc Mgmt vendors like SharePoint this would NOT be a supported option. I would suggest opening a ticket with your document management vendor or perhaps posting a question to the Microsoft Office Technet forum: <http://social.technet.microsoft.com/Forums/en-US/officeitproprevious/threads>

