title: The Powershell V1 to V2 Conversion
link: http://sev17.com/2010/02/01/the-powershell-v1-to-v2-conversion/
author: Chad Miller
description: 
post_id: 9994
created: 2010/02/01 12:58:00
created_gmt: 2010/02/01 16:58:00
comment_status: open
post_name: the-powershell-v1-to-v2-conversion
status: publish
post_type: post

# The Powershell V1 to V2 Conversion

This post is about my experience converting the CodePlex project, [SQL Server Powershell Extensions](http://sqlpsx.codeplex.com/) (SQLPSX) Powershell V1 function libraries into PowerShell V2 Advanced functions within modules. In order to provide context for people reading this blog post a quick timeline is needed: 

  * Powershell V1 was released in November 2006
  * SQLPS, the SQL Server Powershell host that ships with SQL Server 2008, is based on Powershell V1
  * Powershell V2 was released in October 2009
  * Everything you write in Powershell V1 should work in V2
  * SQLPSX is a CodePlex project I started for working with SQL Server and Powershell. The first release was July 2008 and it has frequently updated since. A Powershell V2 release was published on 12/31/2009

And with that hopefully the rest of this post makes sense. Let's take a look at my top six list of Powershell V2 improvements over V1 for script developers:

### **Modules**

Modules allow a script developer to package functions, scripts, format files into something very easy to distribute. In Powershell V1 I would create a function library which is just a script file with related functions. The function library would then need to be sourced to use:

**. ./librarySmo.ps1**

There were several problems with this approach:

  * Handling related related script files and separate function libraries is difficult -- usually solved by creating an initialization script and detailed instructions.
  * Loading assemblies
  * Appending format files

Modules make handling the distribution of a set of related files much easier. We simply place the module which is nothing more than the same old function library with .psm1 extension into a directory under DocumentWindowsPowerShellModules and optionally add a second special file called module manifest (more on this later). As an example I have sqlserver module in a directory DocumentWindowsPowerShellModulessqlserver. I can then import a module instead of sourcing the functions:

**import-module sqlserver**

** **

The module and manifest file contain the necessary information about processing assemblies, related script files, and nested modules. So, converting function libraries to modules involves little more than renaming .ps1 files to the module file extension .psm1 and placing the file into it's own directory under DocumentsWindowsPowershellModules. But, if that's all you are going to do there is little value in creating modules. Moving from Powershell V1 scripts to V2 modules should also include taking advantage of many of the Powershell V2 features described in this blog post.

**A word about binary modules:** SQLPSX is mostly implemented as Powershell script modules there are however a couple of compiled cmdlets used for parsing and formatting of T-SQL scripts: Test-SqlScript and Out-Sqlscript. Converting compiled snapin dll's to a module is just as easy as script based function libraries, you simply copy the snapin dll and any required assemblies to its own directory under DocumentsWindowsPowershellModules. This is exactly what I've done with the SQLParser module. I've also added a module manifest (psd1 file).

This brings us to module manifests which are basically processing instructions for moduels. Module manifests (psd1) files are created by **new-modulemanifest** cmdlet allow us to do several things:

  * Make functions private through by pattern matching the FunctionsToExport property. As an example in the SQLServer module I specify **FunctionsToExport = '*-SQL*' **\-- This tell Powershell to only export function that match -SQL prefix. I have several helper functions that I don't want to export, so I simply use a different prefix or none at all to avoid exporting.
  * Import assemblies automatically by making use of the **RequiredAssemblies **property
  * Nest modules i.e. import child modules with **NestedModules **property
The manifest files themselves are really easy to create. After you've created a module (.psm1), run new-modulemanifest and enter the information when prompted. 

### **Simplified Error Checking**

The try/catch error handling added to Powershell V2 is so much easier to work with and understand than its predecessor in Powershell V1 trap and thow. The construct is especially handy when dealing with [SMO errors that sometimes use nested error objects](/2009/06/powershell-smo-error-handling-tips/).

Both validatescript and validateset reduce input validation code I needed to write. I think this is best illustrated by a couple of examples from SQLPSX functions

The param section below uses ValidateSet to ensure values are either Data or Log:

param( [Parameter(Position=0, Mandatory=$true)] $sqlserver, [ValidateSet("Data", "Log")] [Parameter(Position=1, Mandatory=$true)] [string]$dirtype )

This second param section uses ValidateScript to check that the input object namespace is an SMO object.
    
    
    param(
    [Parameter(Position=0, Mandatory=$true, ValueFromPipeline = $true)]

## Comments

**[Mike Shepard](#105 "2010-02-02 12:58:00"):** Thank you for posting this. I have started slowly moving to the new features in 2.0, and I'm sure that your experiences will help me and others make the transition more quickly and easily.

