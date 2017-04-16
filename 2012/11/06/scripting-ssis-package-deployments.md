title: Scripting SSIS Package Deployments
link: http://sev17.com/2012/11/06/scripting-ssis-package-deployments/
author: Chad Miller
description: 
post_id: 10990
created: 2012/11/06 09:41:33
created_gmt: 2012/11/06 14:41:33
comment_status: open
post_name: scripting-ssis-package-deployments
status: publish
post_type: post

# Scripting SSIS Package Deployments

Before I delve into the subject of scripting SSIS package deployments, I'm going to take a slight detour and explain why the general subject of automating deployments is important. 

## Automating Deployments

One of the keys areas you should be looking at automation, possibly through scripting is deployments. If you're doing deployments and you're not using a scripted repeatable process, free of GUI and typing then you're doing it wrong. I've seen many times in my career where deployments too often rely on complex instructions rather than using a tested script-based approach. It is inevitable; relying on even the most detailed step-by-step manual instructions will lead to deployments errors because of the human operator factor. When it actually comes time to deploy changes there should be zero typing or clicking. And if it's not a fully automated deployment then any manual steps should be made as simple as possible such as "run this script with these parameters through copy and paste and report back results." _End Detour_. 

##  SSIS Package Deployments

My SSIS package deployment requirements: 

  1. The solution must support 2005, 2008, 2008 R2 and 2012 because I have a mixed environment
  2. The solution must support deploying to a SQL Server data storage in msdb from a dtsx file
  3. The solution must include verification of package installation
  4. The solution must be able to create any needed folder structures automatically
  5. The solution must include error handling and detailed output on operations performed
  6. The solution must support constrained parameters based on using SQL Server data store of a ServerInstance, the dtsx file and the full destination path on the SSIS server
When automating any task I'll see if there's already a solution either from Microsoft or third parties. I couldn't find anything that out-of-the-box does meet all my requirements, but I did find two ways which provide partial solutions. The first, writing Powershell code directly against Microsoft.SqlServer.ManagedDTS like I've done in the SSIS Powershell module I created for [SQL Server Powershell Extensions](http://sqlpsx.codeplex.com/). There's is a function in the SSIS module called _Copy-ISItemFileToSQL_, however it provides only part of the solution and there's a bigger problem of incompatibilities between versions to handle. The assembly for SSIS changes between 2005 and 2008/2008 R2 and 2012 which make crafting a complete solution difficult. I've given up on going down this path because it quickly becomes complex. The second option and the one I went with, is to use the command-line utility dtutil.exe. The nice thing about dtutil--its included with  SQL Server 2005 and higher, [well-documented](http://msdn.microsoft.com/en-us/library/ms162820.aspx) and removes some of complexity of coding against the SSIS classes directly.Although dtutil.exe only meets requirements 1 through 3 above, I can fill in the rest with a bit of Powershell code. I present my Powershell script solution [install-ispackage.ps1](http://poshcode.org/3745). 

### Using Install-ISpackage

To use install-ispackage simply [download the script and from PoshCode](http://poshcode.org/3745) and run by providing three parameters. Here's an example of installing a dtsx file to my SSIS server: 
    
    
    ./install-ispackage.ps1 -DtsxFullName "C:\Users\Public\bin\SSIS\sqlpsx1.dtsx" -ServerInstance "Z001\SQL1" -PackageFullName "SQLPSX\sqlpsx1"

### Install-ISPackage Explanined

The [install-ISPackage](http://poshcode.org/3745) script provides an example of how you can approach calling native console applications (exe's) from Powershell. You see error handling and handling output differs greatly when calling an exe vs. using cmdlets or .NET code. The former does not trigger errors and instead relies on exit codes defined by the console application developer. You have to check lastexitcode and read whatever documentation is provided with console application to determine what the exit codes mean. I'll step through a few things to explain: When I'm dealing with scripts that make changes I like to set _$ErrorActionPreference_ to _Stop_ instead of the default of _Continue_. This way I can wrap some error handling and logging around any errors and be assured the script won't proceed to the next step should an error occur. I also like to make the exit code more user friendly. I'll do this by reading the documentation for the command-line utility. On the [msdn page for dtutil there a nice table under dtutil Exit Codes](http://msdn.microsoft.com/en-us/library/ms162820.aspx) which I then create as a hashtable at the top of the script: 
    
    
    $exitCode = @{
    0="The utility executed successfully."
    1="The utility failed."
    4="The utility cannot locate the requested package."
    5="The utility cannot load the requested package."
    6="The utility cannot resolve the command line because it contains either syntactic or semantic errors"}

I can then return a more useful error message by using the hastable with the built-in variable $lasterrorcode: 
    
    
    throw $exitcode[$lastexitcode]

You'll notice in the Get-SqlVersion function I'm just using the classic sqlcmd.exe console application to run a query to get the SQL Server version number: 
    
    
    $SqlVersion = sqlcmd -S "$ServerInstance" -d "master" -Q "SET NOCOUNT ON; SELECT SERVERPROPERTY('ProductVersion')" -h -1 -W

I choose to use sqlcmd.exe instead of invoke-sqlcmd Powershell cmdlet because it's installed on every SQL 2005 machine and it's easier to use when I just want to return a single string: 
    
    
    C:Users\Public\bin\>Get-SqlVersion -ServerInstance Z001\sql1
    10.50.2550.0

The Set-DtutilPath function tries to find the "right" dtutil.exe based on the SQL version being deployed to. You see although parameters for dtutil.exe are identical between version the utility isn't backwards or forward compatible. You have to use the 9.0 version for 2005,  the 10.0 version for both 2008 and 2008 R2 and the 11.0 version for 2012. The rest of the functions follow a basic pattern: Run dtutil.exe and save the output to $result variable _$result_ will be an array of strings so create a single string separated by newlines: 
    
    
    $result = $result -join "`n"

Rather than returning an error on failure or nothing on success, instead return an object with details of what was run: 
    
    
    new-object psobject -property @{
    ExitCode = $lastexitcode
    ExitDescription = "$($exitcode[$lastexitcode])"
    Command = "$Script:dtutil /File `"$DtsxFullName`" /DestServer `"$ServerInstance`" /Copy SQL;`"$PackageFullName`" /Quiet"
    Result = $result
    Success = ($lastexitcode -eq 0)}

I really like using this technique so that if there are failures as part of troubleshooting you can just run the Command property and you get other relevant details. The key here is you can always get back to the base utility so if something doesn't work in the script you can prove it's not the script when you get the same error in the utility alone. _Note: I have seen errors a few times, usually because a developer will create an SSIS package in later version than the server being deployed to._ Check the _$lasterrorcode_ after calling the utility and returning an object with details: 
    
    
    if ($lastexitcode -ne 0) {
    throw $exitcode[$lastexitcode]
    }

Here I'll use the hashtable defined at the top of script to return a more friendly error message. If errors occur between the error returned and the detailed object I can troubleshoot any issues.

## Comments

**[Klaas](#302 "2013-02-20 04:27:41"):** Great, Chad ! I've scripted a lot of my SQL Management tasks, but I couldn't do this. You save me a lot of time and headaches again. Thank you very much.

**[Yong](#315 "2013-05-31 09:31:58"):** That clears my question. Thank you!

**[Chad Miller](#314 "2013-05-31 07:34:20"):** We'll use either server or workstation, however some or maybe all (I can't recall) versions of dtutil require a running SSIS service with the same version (2005, 2008, 2012) as the server. For this reason usually its better to run from server. In our environment we use a file share and then provide a UNC path instead of a local drive as a parameter to the script i.e. \\\codedeployserver\launchpad\application\sqlpsx.dtsx Every server in our environment has a standard location for Powershell and other host-based scripts, so each server will have the install-ispackage.ps1 script.

**[Yong](#313 "2013-05-31 00:07:53"):** Hi Chad Would you explain how you set up deployment process? 1\. Where did you place this script? Did you put this on the server that you're going deploy to? 2\. How did you upload package file to that server?

**[himanshu](#340 "2013-07-02 06:14:40"):** I want to call SSIS package installation wizard using powershell and for the same I want to double click SSISManifest fie from my deployment folder below script is executing SSIS EXECUTION WIZARD but I don't want that I need SSIS PACKAGE EXECUTION WIZARD could you please make changes below script to open right wizard : #call SSIS package installation wizard $ManifestFile = get-childitem "D:\Deployments\ssis_folder\Deployment_todaydate\\*.SSISDeploymentManifest" $baseFolder = [System.IO.Path]::GetDirectoryName($ManifestFile) [xml] $list = Get-Content $ManifestFile foreach($package in $list.DTSDeploymentManifest.Package){ $basePackage = [System.IO.Path]::GetFileNameWithoutExtension($package) # This might need to be a relative path $fullyQualifiedPackage = [System.IO.Path]::Combine($baseFolder, $package) $cmd = [string]::Format($format, $fullyQualifiedPackage, $destinationServer, $basePackage) cmd /c $cmd } #package installer wizard code of SSIS end

**[Chad Miller](#341 "2013-07-02 09:09:34"):** I have not used the dtsinstall.exe GUI with SSISDeploymentManifests as you described. The blog post shows how I'm using dtutil to deploy packages. You may want to ask your question on StackOverFlow forum with the SSIS, and Powershell tags.

