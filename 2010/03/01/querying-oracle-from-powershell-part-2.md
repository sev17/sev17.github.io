title: Querying Oracle from Powershell Part 2
link: http://sev17.com/2010/03/01/querying-oracle-from-powershell-part-2/
author: Chad Miller
description: 
post_id: 9999
created: 2010/03/01 09:15:00
created_gmt: 2010/03/01 13:15:00
comment_status: open
post_name: querying-oracle-from-powershell-part-2
status: publish
post_type: post

# Querying Oracle from Powershell Part 2

In [part one](/2010/02/querying-oracle-from-powershell-part-1/) we installed and configured the Oracle client software, in this post we will query an Oracle database from Powershell.  In addition we’ll look at one way to handle storing sensitive password information. 

### Querying and Oracle Database

To query an Oracle database we’ll use a function called [Get-OLEDBData](http://poshcode.org/1591). The code listed below and is also available on [PoshCode](http://poshcode.org/): 
    
    
    function Get-OLEDBData ($connectstring, $sql) {
    $OLEDBConn = New-Object System.Data.OleDb.OleDbConnection($connectstring)
    $OLEDBConn.open()
    $readcmd = New-Object system.Data.OleDb.OleDbCommand($sql,$OLEDBConn)
    $readcmd.CommandTimeout = '300'
    $da = New-Object system.Data.OleDb.OleDbDataAdapter($readcmd)
    $dt = New-Object system.Data.datatable
    [void]$da.fill($dt)
    $OLEDBConn.close()
    return $dt
    }

The Get-OLEDBData function has been tested against SQL Server, Informix, Oracle and Excel data sources. In addition other data source can be addressed all that is needed is a valid connection string, the ability the data source to support OLEDB connections and of course the appropriate drivers. See [connectionstring.com](http://connectionstrings.com/) for a list of connection string examples. The hard part of querying an Oracle database from Powershell is setting up the Oracle Client software demonstrated in part one. Using the Get-OLEDBData function is simple, just pass a connection string and a query as follows: 
    
    
    $connString = "password=assword;User ID=SYSTEM;Data Source=XE;Provider=OraOLEDB.Oracle"
    $qry= "SELECT * FROM HR.DEPARTMENTS"
    ./Get-OLEDBData $connString $qry

This will return all rows from the DEPARTMENTS table in HR schema: ![oraclePowershell1](http://images.sev17.com/oraclePowershell1_thumb.jpg) As long as your Oracle client software is installed and configured correctly and you have a valid connection string, specifying a database user with sufficient rights the query works. One issue immediately apparent is the sensitive password information. This especially true if you intend to use this technique for automated batch jobs. To address the password issue we’ll need to encrypt the connection string and the store the password somewhere. Let’s a look at one solution… 

### Encrypting Connection Strings

Ideally everything would use Windows authentication and you wouldn’t need to store password information. The reality is this simple isn’t the case especially with Oracle databases. Unfortunately there aren’t any native encryption cmdlets in Powershell (I’d love to see a cmdlet that would use certificates in order to avoid pass phrases), there are however a very nice and set of Powershell encryption functions created by [Steven Hystad](http://lunex.spaces.live.com/default.aspx) called [Library-StringCrytpo](http://lunex.spaces.live.com/blog/cns!64CB3857E28BD106!344.entry). To use the encryption functions download the Powershell script and source the library, then call the Write-EncryptedString function passing our connection string we want to encrypt with a passphrase. To decrypt the connection string call the Read-EncryptedString function with the encrypted string and passphrase. 
    
    
    #Source Encryption Functions
    . ./Library-StringCrypto.ps1
    #encrypt string using passphrase
    $encrypt = Write-EncryptedString $connString "4#&7yaoff"
    #Show encrypted string
    $encrypt
    #Decrypt string
    Read-EncryptedString $encrypt "4#&7yaoff"

The encrypt functions work well, but I like to do is then take the encrypted string and store it in a SQL Server table that is locked down. To do we’ll need to first create a table in SQL Server database as follows: **CREATE** TABLE [dbo].[server_lku]( [server_name] [**varchar**](255) **NOT** NULL, [server_type] [**varchar**](25) **NOT** NULL, [connection_string] [**varchar**](2000) **NOT** NULL, [is_encrypted] [bit] **NOT** NULL,

## Comments

**[Tyler Powers](#113 "2010-04-02 12:01:07"):** Thank you very much for the info! Your contribution to the powershell community is greatly appreciated!

