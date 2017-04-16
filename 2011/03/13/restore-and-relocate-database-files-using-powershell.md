title: Restore and Relocate Database Files Using PowerShell
link: http://sev17.com/2011/03/13/restore-and-relocate-database-files-using-powershell/
author: Chad Miller
description: 
post_id: 10607
created: 2011/03/13 20:53:16
created_gmt: 2011/03/14 00:53:16
comment_status: open
post_name: restore-and-relocate-database-files-using-powershell
status: publish
post_type: post

# Restore and Relocate Database Files Using PowerShell

I was recently asked a question on restoring a database using PowerShell with the following requirements

  1. Take a database backup file i.e. DatabaseName.bak 
  2. Derive the database name from the backup file name 
  3. Disconnect any user connected to the database 
  4. Relocate (move) the physical files to SQL Serve r instance the default data and log directories 

If you had to develop a script like this from scratch you’ll find you would need to dig deep into various [SMO](http://msdn.microsoft.com/en-us/library/cc285859.aspx) classes—a non-starter for most novice PowerShell users. Fortunately thanks to the [CodePlex](http://www.codeplex.com/) project [SQL Server PowerShell Extensions](http://sqlpsx.codeplex.com/) (SQLPSX) there are base functions which make this task much easier. Let’s take a look at the script, Restore-Database.ps1 then an explanation. **Note: The following script requires ****[SQLPSX**](http://sqlpsx.codeplex.com/)** version 2.3.2.1 or higher:**
    
    
    param($sqlserver, $filepath)
    
    import-module sqlserver -force
    
    $server = get-sqlserver $sqlserver
    
    $filepath = Resolve-Path $filepath | select -ExpandProperty Path
    $dbname = Get-ChildItem $filePath | select -ExpandProperty basename
    
    $dataPath = Get-SqlDefaultDir -sqlserver $server -dirtype Data
    $logPath = Get-SqlDefaultDir -sqlserver $server -dirtype Log
    
    $relocateFiles = @{}
    Invoke-SqlRestore -sqlserver $server  -filepath $filepath -fileListOnly | foreach { `
        if ($_.Type -eq 'L')
        { $physicalName = "$logPath{0}" -f [system.io.path]::GetFileName("$($_.PhysicalName)") }
        else
        { $physicalName = "$dataPath{0}" -f [system.io.path]::GetFileName("$($_.PhysicalName)") }
        $relocateFiles.Add("$($_.LogicalName)", "$physicalName")
    }
    
    $server.KillAllProcesses($dbname)
    
    Invoke-SqlRestore -sqlserver $server -dbname $dbname -filepath $filepath -relocatefiles $relocateFiles -Verbose -force

What this script does is take two parameters, a SQL Server and the database back file. The line:
    
    
    $filepath = Resolve-Path $filepath | select -ExpandProperty Path 

Get the full file path so if the script was called with just a relative path i.e Restore-Database.ps1 databasename.bak. The script uses PowerShell Get-ChildItem cmdlet to grab the full path. The next line simply grabs the basename i.e. file name without path or extension. The next two lines use SQLPSX functions to grab the default data and log file directories for the SQL Server instance:
    
    
    $dataPath = Get-SqlDefaultDir -sqlserver $server -dirtype Data
    $logPath = Get-SqlDefaultDir -sqlserver $server -dirtype Log

The next several lines restore the file list only, this is an option to simply retrieve the physical and logical files that are part of the database backup. Because the requirement is relocate the files to the default directories, the script collects the logical and physical names into a hashtable called $relocationFiles. One thing to notice the script makes use of _[system.io.path]::GetFileName_ static method to extract just the file name portion of the files which is appended to default directory path.

Finally with the our new file location information, file path to the backup file, were ready to disconnect any users and restore the database with relocatefiles option:
    
    
    $server.KillAllProcesses($dbname)
    Invoke-SqlRestore -sqlserver $server -dbname $dbname -filepath $filepath -relocatefiles $relocateFiles -Verbose -force