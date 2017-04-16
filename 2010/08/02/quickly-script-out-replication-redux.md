title: Quickly Script Out Replication Redux
link: http://sev17.com/2010/08/02/quickly-script-out-replication-redux/
author: Chad Miller
description: 
post_id: 10406
created: 2010/08/02 23:04:28
created_gmt: 2010/08/03 03:04:28
comment_status: open
post_name: quickly-script-out-replication-redux
status: publish
post_type: post

# Quickly Script Out Replication Redux

Dave Levy ([Blog](http://adventuresinsql.com/)|[Twitter](http://twitter.com/Dave_Levy)) posted a script in [a blog post](http://adventuresinsql.com/2010/07/how-can-i-quickly-script-out-replication/) in which he uses a bit of [SQL PowerShell Extensions](http://sqlpsx.codeplex.com) (SQLPSX) and some [Replication Management Objects](http://msdn.microsoft.com/en-us/library/microsoft.sqlserver.replication.aspx) (RMO) to script out SQL Server replication.  Overall Dave's script is a good use of PowerShell and RMO. Scripting out objects is much easier to handle in PowerShell than a T-SQL solution, however some improvements to the script can be made. **EDIT:** _Dave pointed out I was missing the Append parameter for Out-File. The script below has been corrected._ One of the goals of SQLPSX is simplify SQL Server PowerShell scripting by providing functions over common tasks. I think this important as the PowerShell host which ships with SQL Server, sqlps, does not cover replication. The original script can be refactored to use SQLPSX instead of working with the RMO classes directly as well reduce some of the code as follows: 
    
    
    param ($sqlServer,$path,[switch]$scriptPerPublication)
    Import-Module Repl
    
    if ($sqlServer -eq "")
    {
        $sqlserver = Read-Host -Prompt "Please provide a value for -sqlServer"
    }
    
    if ($path -eq "")
    {
        $path = Read-Host -Prompt "Please provide a value for output directory path"
    }
    
        $scriptOptions = New-ReplScriptOptions
        $scriptOptions.IncludeArticles = $true
        $scriptOptions.IncludePublisherSideSubscriptions = $true
        $scriptOptions.IncludeCreateSnapshotAgent = $true
        $scriptOptions.IncludeGo = $true
        $scriptOptions.EnableReplicationDB = $true
        $scriptOptions.IncludePublicationAccesses = $true
        $scriptOptions.IncludeCreateLogreaderAgent = $true
        $scriptOptions.IncludeCreateQueuereaderAgent = $true
        $scriptOptions.IncludeSubscriberSideSubscriptions = $true
    
        $distributor = Get-ReplServer $sqlserver
    
    if($distributor.DistributionServer -eq $distributor.SqlServerName)
    {
    	$distributor.DistributionPublishers | ForEach-Object {
    		$distributionPublisher = $_
    		if($distributionPublisher.PublisherType -eq "MSSQLSERVER")
    		{
    			$outPath =  "{0}from_{1}{2}"  -f $path,$distributionPublisher.Name.Replace("","_"),$((Get-Date).toString('yyyy-MMM-dd_HHmmss'))
    			New-Item $outPath -ItemType Directory | Out-Null
    			Get-ReplPublication $distributionPublisher.Name | ForEach-Object {
    				$publication = $_
    				$fileName = "{0}{1}.sql" -f $outPath,$publication.DatabaseName.Replace(" ", "")
    				if($scriptPerPublication)
    				{
    					$fileName = "{0}{1}_{2}.sql" -f $outPath,$publication.DatabaseName.Replace(" ", ""),$publication.Name.Replace(" ", "")
    				}
    				Write-Debug $("Scripting {0} to {1}" -f $publication.Name.Replace(" ", ""),$fileName)
    				Get-ReplScript -rmo $publication -scriptOpts $($scriptOptions.ScriptOptions) | Out-File $fileName -Append
    			}
    		}
    	}
    }
    else
    {
        $outPath =  "{0}from_{1}{2}"  -f $path,$distributor.SqlServerName.Replace("","_"),$((Get-Date).toString('yyyy-MMM-dd_HHmmss'))
        New-Item $outpath -ItemType Directory | Out-Null
        Get-ReplPublication $distributor.SqlServerName | ForEach-Object {
    		$publication = $_
    		$fileName = "{0}{1}.sql" -f $outPath,$publication.DatabaseName.Replace(" ", "")
    		if($scriptPerPublication)
    		{
    			$fileName = "{0}{1}_{2}.sql" -f $outPath,$publication.DatabaseName.Replace(" ", ""),$publication.Name.Replace(" ", "")
    		}
    		Write-Debug $("Scripting {0} to {1}" -f $publication.Name.Replace(" ", ""),$fileName)
    		Get-ReplScript -rmo $publication -scriptOpts $($scriptOptions.ScriptOptions) | Out-File $fileName -Append
    	}
    }

Save the script as a .ps1 file, scriptPublications.ps1 and invoke with: 
    
    
    .scriptPublications.ps1 -sqlserver "Z002sql2k8" -path "C:Usersu00repl"
    .scriptPublications.ps1 -sqlserver "Z002sql2k8" -path "C:Usersu00repl" -scriptPerPublication

### Comments:

  * Notice the use of New-ReplScriptOptions. This is a helper function to handle the replication scripting options.
  * Replace a bool with a switch param. A switch is kind of like a bool, but you don't specify $true or $false. In PowerShell scripting switch should be used over bool.
  * Removed unneeded code