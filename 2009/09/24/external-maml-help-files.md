title: External MAML Help Files
link: http://sev17.com/2009/09/24/external-maml-help-files/
author: Chad Miller
description: 
post_id: 9981
created: 2009/09/24 10:36:00
created_gmt: 2009/09/24 14:36:00
comment_status: open
post_name: external-maml-help-files
status: publish
post_type: post

# External MAML Help Files

One of the really handy improvevements in Powershell V2 is around creating help information for your script users. I often find myself having to read through script and function definitions to figure out how to use them (and this is for scripts I write!), which becomes a real pain as your functions libraries grow. Starting in version 2 we can use [comment-based help in function or scripts](http://technet.microsoft.com/en-us/library/dd819489.aspx).

Comment-based help is great for creating simple help information for scripts and smaller collections of functions, however for large groups of functions you should consider using external help files. The format of the external help files is called [Microsoft Assistant Markup Langauge](http://en.wikipedia.org/wiki/MAML) (MAML). The XML-based, MAML help file format is used by compiled snapins and modules to produce all of the wonderful help information you see when you type **get-help <command name>**.

Using external help files provide several benefits over comment-based help:

  * Changes in help files do not require changing your script files
  * [Use language settings](http://msdn.microsoft.com/en-us/library/dd878343\(VS.85\).aspx)
  * [Easily create HTML or CHM standalone help documention](http://poshcode.org/1136)

For the reasons above and since my CodePlex project, [SQL Server Powershell Extensions](http://sqlpsx.codeplex.com/) contains over 100 functions, I've decided to create all of my help documentation in MAML files. For compiled cmdlets there is a very nice utility called [Cmdlet Help Editor by Wassim Fayed](http://blogs.msdn.com/powershell/archive/2007/09/01/new-and-improved-cmdlet-help-editor-tool.aspx) that autogenerates MAML files. Unfortunately the utility does not work for scripts, so, I created a script called [New-MAML](http://poshcode.org/1338) that accepts either CommandInfo or FunctionInfo objects emitted from Get-Command. The New-MAML script is posted on [Poshcode](http://poshcode.org/).

New-MAML uses a function called [New-XML](http://poshcode.org/1244) created by [Joel Bennet](http://huddledmasses.org/). The New-XML function leverages LINQ to make quick work out of generating XML documents, in this case a MAML file. Here's an example of using the New-MAML script for a function called Test-ISpath that is part of the SSIS related functions provided in [SQLPSX](http://sqlpsx.codeplex.com/):

Having sourced the function test-ispath, call the New-MAML function:

**$xml = ./new-maml test-ispath $xml.Declaration.ToString() | out-file ./test-ispath.ps1-help.xml -encoding "UTF8" $xml.ToString() | out-file ./test-ispath.ps1-help.xml -encoding "UTF8" -append**

Once the XML/MAML file is generated, you'll need to manually edit the MAML by filling in the TODO items and the parameters options that are defaulted to false. The position parameter option will need to be changed in the generated MAML also. For compiled cmdlets place the MAML file in the same directory as the binary module or snapin dll. For script modules/functions include a reference to the External MAML file for each function. Unfortunately script modules require that you [specify the path to your MAML for EVERY function](http://stackoverflow.com/questions/1432717/powershell-v2-external-maml-help).  **Note: You can have multiple function help items within the same MAML file.**

Finally edit the the Test-IsPath function by adding the path to your XML file.

#  .ExternalHelp C:Usersu00binTest-ISPath.psm1-help.xml **function** Test-ISPath

_ ..Rest of function ..._

__

**Note: when you specify an external help file, you need to use an absolute path ([except for language subfolder](http://msdn.microsoft.com/en-us/library/dd878343\(VS.85\).aspx)) with no variables as part of the path name. **For instance I'll often use _$scriptRoot = Split-Path (Resolve-Path $myInvocation.MyCommand.Path) _in my scripts, however this will NOT work:

#  .ExternalHelp $scriptRootTest-ISPath.psm1-help.xml

Interestingly enough, you will still get default help generated even though it will fail to use the help file.

Having specified an absolute path, if you re-source the function or reload the module you'll be able to call **get-help test-ispath**

![](http://images.sev17.com/newMaml.jpg)

## Comments

**[Chad Miller](#88 "2009-03-05 10:36:00"):** Chris -- Thanks for comment. Its always nice to know when someone finds a post useful.

**[Chris](#89 "2009-03-05 10:36:00"):** Thank you, this saved me a great deal of time.

**[Chad Miller](#90 "2009-02-26 10:36:00"):** I don't believe there is way to remove commonparameters. Per get-help about_commonparameters: "The common parameters are a set of cmdlet parameters that you can use with any cmdlet. They are implemented by Windows PowerShell, not by the cmdlet developer, and they are automatically available to any cmdlet."   
  
You may want to double-check by posting a question to StackOverflow.

**[No name](#91 "2009-02-25 10:36:00"):** In your screenshot above there is the section about CommonParameters. Is there a way to get this out of the output produced by get-help? I'm trying to use a MAML file to document a script, but the common parameters are not germane to this script.  
  
Thanks.

**[Chad Miller](#92 "2009-11-14 10:36:00"):** As a workaround I thought I would try creating a empty binary module, implementing a minimalist cmdlet and then pointing to a MAML file I created for a script module. The idea being I could import-module for my binary empty cmdlet. Unfortunately I could not get this to work as I thought. First for compiled cmdlet help you have to implement a cmdlet for each, well cmdlet. For instance you couldn't implement a dummy cmdlet then have the MAML file work. It would seem you have to actually have binary cmdlets implemented for each item in the MAML. Here's the code for implementing a minimal cmdlet. As noted this workaround does not work.   
  
$source = @"  
using System.Management.Automation;   
  
namespace ISExternalHelp  
{  
  
[Cmdlet("Copy", "ISItemSQLToSQL")]  
public class CopyISItemSQLToSQL : Cmdlet  
{  
protected override void ProcessRecord()  
{  
WriteObject("Empty Cmdlet used to implement external help for SSIS script module.");  
}  
}  
}  
"@  
add-type -TypeDefinition $source -OutputAssembly ISExternalHelp.dll -OutputType Library  
  
Oh well, I thought it was worth a try.

