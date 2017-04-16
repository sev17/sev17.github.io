title: Using SMO Transfer Class to Script Database Objects
link: http://sev17.com/2011/07/24/using-smo-transfer-class-to-script-database-objects/
author: Chad Miller
description: 
post_id: 10722
created: 2011/07/24 20:51:56
created_gmt: 2011/07/25 00:51:56
comment_status: open
post_name: using-smo-transfer-class-to-script-database-objects
status: publish
post_type: post

# Using SMO Transfer Class to Script Database Objects

I’ve spent some time trying to get the [SMO Transfer class](http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.transfer.aspx) to bend to my will. I want to script out all objects of a certain type or a select list of objects. As we will see a moment this was a bit a of challenge and it wasn’t until I had given up , incorrectly figured it was a bug and filled [Connect Item](https://connect.microsoft.com/SQLServer/feedback/details/663078/microsoft-sqlserver-management-smo-transfer-ignores-options) did I learn the nuisances of the Transfer class.  So, the item closed with a comment telling me how to properly use the Transfer class. One thing to keep in is that this method creates a script in a single file and not a file per object. If you’re interested in creating a file per object then take a look at Aaron Nelson’s ([blog](http://sqlvariant.com/wordpress/)|[twitter](http://twitter.com/#!/sqlvariant)) [Use PowerShell to Script SQL Database Objects](http://blogs.technet.com/b/heyscriptingguy/archive/2010/11/04/use-powershell-to-script-sql-database-objects.aspx) post or [my post on using Red Gate SQL Compare](/2011/03/to-script-or-not-to-script/).

The Transfer class has many properties and options to set to either true or false which control the behavior of the emitted scripts or object/data transferred. Some of these properties default to true. We’d think that setting **CopyAllStoredProcedures** to true would copy all stored procedures—it doesn’t because two other properties override the setting **WithDependencies** and **CopyAllObjects** unless you change their values from true which is the default you’ll get either all objects or the stored procedures plus any objects the procedures depend on. Let’s look a few examples.

## All Stored Procedures Creation Script
    
    
    add-type -AssemblyName "Microsoft.SqlServer.ConnectionInfo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    add-type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    add-type -AssemblyName "Microsoft.SqlServer.SMOExtended, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    
    $sourceSrv = "$env:computernamesql1"
    $sourceDb = "Northwind"
    
    $server = new-object ("Microsoft.SqlServer.Management.Smo.Server") $sourceSrv
    $db = $server.Databases[$sourceDb]
    
    $transfer = new-object ("Microsoft.SqlServer.Management.Smo.Transfer") $db
    $transfer.Options.WithDependencies = $false
    $transfer.CopyAllObjects = $false
    $transfer.CopyAllStoredProcedures = $true
    $transfer.Options.ScriptBatchTerminator = $true
    $transfer.Options.FileName = "C:Usersu00binnorthwind.createprocedures.sql"
    $transfer.Options.IncludeIfNotExists = $true
    $transfer.ScriptTransfer()

What this script does is script out all stored procedures only. In addition to setting the properties  **WithDependencies**, **CopyAllObjects** and **CopyAllStoredProcedures** you’ll want to set the following as I have done in the script:

  * **ScriptBatchTerminator** – Includes a GO statement after each object. Note: batch terminators are only written to the file and not console output. 
  * **FileName** – Set this to create a sql file. You should always use file output for the Transfer class as this is the only way to get batch terminators 
  * **IncludeIfNotExists** – Adds an “IF NOT EXISTS … CREATE PROCEDURE” statement to each stored procedure or “IF NOT EXISTS…DROP PROCEDURE” statement when **ScriptDrops** property is true. 

## All Stored Procedures Drop Script 

If your goal is first drop all the procedures before creating them you’ll need to create a second script to run first. This script is exactly the same as the create script only change the **FileName** parameter and add **ScriptDrops:**
    
    
    $transfer.Options.FileName = "C:Usersu00binnorthwind.dropprocedures.sql"
    $transfer.Options.ScriptDrops = $true

## Specific Stored Procedures

If you want to script specific stored procedures using the Transfer class you’ll need to set the transfer object ObjectList property as follows:
    
    
    $objectList = New-Object -TypeName "System.Collections.ArrayList"
    $db.StoredProcedures | where {$_.name -like "Cust*"} | foreach {$null = $objectList.Add($_)}
    $transfer.ObjectList = $objectList

In this example I’ve created an arraylist of storedprocedure objects where the name is like “Cust*” and set the **ObjectList** property to the arraylist. Now only those specific procedures will be scripted.

## Miscellany Settings

Some other settings which you may want to consider.

Change the default file encoding from Unicode to ASCII:
    
    
    $encoding = new-object "System.Text.ASCIIEncoding"
    $transfer.Options.Encoding = $encoding

Suppress showing scripts on screen:
    
    
    $transfer.Options.ToFileOnly = $true