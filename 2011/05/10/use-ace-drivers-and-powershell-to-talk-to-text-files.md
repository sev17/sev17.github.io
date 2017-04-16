title: Use ACE Drivers and PowerShell to Talk to Text Files
link: http://sev17.com/2011/05/10/use-ace-drivers-and-powershell-to-talk-to-text-files/
author: Chad Miller
description: 
post_id: 10642
created: 2011/05/10 08:27:01
created_gmt: 2011/05/10 12:27:01
comment_status: open
post_name: use-ace-drivers-and-powershell-to-talk-to-text-files
status: publish
post_type: post

# Use ACE Drivers and PowerShell to Talk to Text Files

As a follow up to my [SQLRally 2011 Scripting Guy Guest Blog Post](/2011/05/sqlrally-2011-scripting-guy-guest-blog-post/) which dealt with Excel and Access files, this post explorers working with delimited text files using the ACE driver.

## What about Import-CSV?

One of the first things I do when working with PowerShell is first look at the built-in cmdlets. PowerShell has a cmdlet for working with delimited text files  called import-csv. Using import-csv you can even specify a delimiter other than a comma using the –Delimiter parameter. Import-Csv is good enough for many scenarios which involve delimited text, but there a few areas like fixed width and write back which using an ACE and OLE DB handles and import-csv does not.

## Setup

Just as described in the my previous post you need to go to [Microsoft Access Database Engine 2010 Redistributable](http://www.microsoft.com/downloads/en/details.aspx?FamilyID=c06b8369-60dd-4b64-a44b-84b371ede16d&displaylang=en), and download AccessDatabaseEngine.exe or AccessDatabaseEngine_x64.exe, depending on your operating system. 

For testing purposes we’ll create a simple CSV file:
    
    
    get-psdrive | export-csv ./psdrive.csv -NoTypeInformation -Force

## CSV files

Connecting to a text file and querying the data is pretty easy using the ACE drivers and as long as the file is comma separated no special setup is required (more on other delimiters in moment):
    
    
    $filepath = "C:Usersu00bin"
    
    $connString = "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=`"$filepath`";Extended Properties=`"text;HDR=yes;FMT=Delimited`";"
    
    $qry = 'select * from [psdrive.csv]'
    
    $conn = new-object System.Data.OleDb.OleDbConnection($connString)
    $conn.open()
    $cmd = new-object System.Data.OleDb.OleDbCommand($qry,$conn) 
    $da = new-object System.Data.OleDb.OleDbDataAdapter($cmd) 
    $dt = new-object System.Data.dataTable 
    $null = $da.fill($dt)
    $conn.close()
    $dt

**Note: One of the odd things about using a text file as a data source is that unlike Excel and Access files in which you list the full path to the file, with text files you only specify the directory where the text file is located i.e. “C:Usersu00bin”. The text file is listed in the query i.e. “select * from [psdrive.csv]”. In this way the concept of the “database” is really the directory and files are analogous to tables.**

## Other Delimited Files

What if your text is delimited by something other than a comma? Let’s create a semi-colon delimited text file to test:
    
    
    get-psdrive | export-csv ./psdrive.csv -NoTypeInformation -Force  -Delimiter ";"

Because the delimiter is a semi-colon you’ll need to create a special file named **schema.ini** in order to describe the delimited file. Next you'll need to place the schema.ini file  in the same directory as the text file. Add the following to the ini file:
    
    
    [psdrive.csv]
    Format=Delimited(;)
    ColNameHeader=True

Now we can use the same code as demonstrated for a CSV file to query our semi-colon delimited file.

**Notes:**

  * You can specify multiple files in schema.ini with each file having a new section i.e. [filename] 
  * There are many additional options which can be used to describe the delimited file including fixed-width. See this on [Connecting to a Text File](http://en.csharp-online.net/Connect_Data_ADO_NET%E2%80%94Discussion_14) for details. 
  * If you’re a SQL Server professional you may wonder if can create a linked server using the ACE drivers, well you can but there seems to be bugs several bugs that prevent you from connecting to anything other than Excel files when running x64: <https://connect.microsoft.com/SQLServer/feedback/details/587897/connecting-via-a-linked-server-to-an-access-2010-database-file>. Despite making sure I had various settings specified I could not get the ACE driver to work against a text file through a linked server on an x64 machine while PowerShell + ACE worked fine.

## Comments

**[Chad Miller](#254 "2011-11-13 15:10:53"):** Note: Updating data through the ACE driver to text files is not supported. You'll see this error message if you try: Updating data in a linked table is not supported by this ISAM

**[Chad Miller](#255 "2011-11-13 15:15:44"):** Inserts work fine: $filepath = "C:UsersPublicbin" $connString = "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=`"$filepath`";Extended Properties=`"text;HDR=yes;FMT=Delimited`";" $qry = 'select * from [psdrive.csv]' $conn = new-object System.Data.OleDb.OleDbConnection($connString) $conn.open() $cmd = new-object System.Data.OleDb.OleDbCommand $cmd.Connection = $conn #$cmd.CommandText = @" #UPDATE [psdrive.csv] SET Name = 'ManOhMan' WHERE Name = 'WSMan' #"@ $cmd.CommandText = @" INSERT INTO [psdrive.csv] (Used, Free, CurrentLocation, Name, Provider, Root, Description, Credential) VALUES ('1','2','3','4','5','6','7','8') "@ $cmd.ExecuteNonQuery() $conn.close()

