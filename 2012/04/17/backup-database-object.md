title: Backup Database Object
link: http://sev17.com/2012/04/17/backup-database-object/
author: Chad Miller
description: 
post_id: 10905
created: 2012/04/17 18:15:37
created_gmt: 2012/04/17 22:15:37
comment_status: open
post_name: backup-database-object
status: publish
post_type: post

# Backup Database Object

I saw this question in one of forums on backing up i.e. scripting out a database object. The problem is easy to solve, but only if you're familiar with SMO :). Even so, there some more obscure aspects of SMO like URNs which not many people are aware of. If you read the [MSDN docs on SMO](http://msdn.microsoft.com/en-us/library/ms162557.aspx) you'll find URNs are referenced in a few places. I haven't used them much, but for this case  it makes sense. Normally if you want to get to an object in SMO you'd reference the server, then the database then the object type collection (StoredProcedures, Views, etc.), and then the object;  however if you don't know the object type you can call EnumObject method on the database to get a list of objects with its URN. The URN is like a primary key of objects in SMO. So, here's my solution with with comments... 
    
    
    add-type -AssemblyName "Microsoft.SqlServer.ConnectionInfo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    add-type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    add-type -AssemblyName "Microsoft.SqlServer.SMOExtended, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    add-type -AssemblyName "Microsoft.SqlServer.SqlEnum, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    add-type -AssemblyName "Microsoft.SqlServer.Management.Sdk.Sfc, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
    
    #######################
    <#
    .SYNOPSIS
    Backs up a database object definition.
    .DESCRIPTION
    The Backup-DatabaseObject function  backs up a database object definition by scripting out the object to a .sql text file.
    .EXAMPLE
    Backup-DatabaseObject -ServerInstance Z002 -Database AdventureWorks -Schema HumanResources -Name vEmployee -Path "C:UsersPublic"
    This command backups up the vEmployee view to a .sql file.
    .NOTES
    Version History
    v1.0   - Chad Miller - Initial release
    #>
    function Backup-DatabaseObject
    {
        [CmdletBinding()]
        param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullorEmpty()]
        [string]$ServerInstance,
        [Parameter(Mandatory=$true)]
        [ValidateNotNullorEmpty()]
        [string]$Database,
        [Parameter(Mandatory=$true)]
        [ValidateNotNullorEmpty()]
        [string]$Schema,
        #Database Object Name
        [Parameter(Mandatory=$true)]
        [ValidateNotNullorEmpty()]
        [string]$Name,
        [Parameter(Mandatory=$true)]
        [ValidateNotNullorEmpty()]
        [string]$Path
        )
    
        $server = new-object Microsoft.SqlServer.Management.Smo.Server($ServerInstance)
        $db = $server.Databases[$Database]
    
        #Create a UrnCollection. URNs are used by SMO as unique identifiers of objects. You can think of URN like primary keys
        #The URN format is similar to XPath
        $urns = new-object Microsoft.SqlServer.Management.Smo.UrnCollection
    
        #Get a list of database object which match the schema and object name specified
        #New up an URN object and add the URN to the urns collection
        $db.enumobjects() | where {$_.schema -eq $Schema -and  $_.name -eq $Name } |
            foreach {$urn = new-object Microsoft.SqlServer.Management.Sdk.Sfc.Urn($_.Urn);
                     $urns.Add($urn) }
    
        if ($urns.Count -gt 0) {
    
            #Create a scripter object with a connection to the server object created above
            $scripter = new-object Microsoft.SqlServer.Management.Smo.Scripter($server)
    
            #Set some scripting option properties
            $scripter.options.ScriptBatchTerminator = $true
            $scripter.options.FileName = "$PathBEFORE_$Schema.$Name.sql"
            $scripter.options.ToFileOnly = $true
            $scripter.options.Permissions = $true
            $scripter.options.DriAll = $true
            $scripter.options.Triggers = $true
            $scripter.options.Indexes = $true
            $scripter.Options.IncludeHeaders = $true
    
            #Script the collection of URNs
            $scripter.Script($urns)
    
        }
        else {
            write-warning "Object $Schema.$Name Not Found!"
        }
    
    } #Backup-DatabaseObject

And here's example of sourcing and calling the function: 
    
    
    . ./Backup-DatabaseObject.ps1
    Backup-DatabaseObject -ServerInstance Z002 -Database AdventureWorks -Schema HumanResources -Name vEmployee -Path "C:UsersPublic"

I've posted the code on [PoshCode](http://poshcode.org/3367) also.

## Comments

**[Garry Bargsley](#285 "2012-04-17 21:24:07"):** I think this is in response to my question I posted on a couple powershell forums. I am so glad it intrigued you enough to figure it out. I sure would not have gotten all these pieces together to get it to work. I can't wait until the morning when I get back to the office to try it out. Thank you, Thank you. Garry

**[Chad Miller](#286 "2012-04-17 22:18:53"):** You're welcome. Although there's good coverage in Powershell with builtin cmdlets, SQL Server management is a bit tricky and requires much more programming/scripting. It can difficult just to figure out where to start. If you have problems with the script--let me know. I am loading some assemblies only available on 2008 and higher, but the server you connect to can be any version. If needed I adjust the assembly loading to work with SQL 2005 also.

**[Garry Bargsley](#287 "2012-04-18 09:41:02"):** That is awesome, just finished testing. Is there a way to load the function automatically when I open PowerShell? Then I can just type the command with parameters and I am good to go. Again, thanks so much for this working example...

**[Chad Miller](#288 "2012-04-18 13:57:29"):** Sure you can use [Powershell profiles](http://msdn.microsoft.com/en-us/library/windows/desktop/bb613488\(v=vs.85\).aspx). If you add sourcing the function to your profile, the function will be available whenever you start powershell and if you place the Backup-DatabaseObject.ps1 file in the same directory as your profile you can add this one line to your profile.ps1: `. $(Join-Path (split-path $profile) Backup-DatabaseObject.ps1)` Keep in mind the more things you add to your profile the slower powershell will start.

