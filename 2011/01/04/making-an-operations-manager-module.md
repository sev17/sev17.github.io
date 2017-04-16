title: Making an Operations Manager Module
link: http://sev17.com/2011/01/04/making-an-operations-manager-module/
author: Chad Miller
description: 
post_id: 10549
created: 2011/01/04 07:27:57
created_gmt: 2011/01/04 12:27:57
comment_status: open
post_name: making-an-operations-manager-module
status: publish
post_type: post

# Making an Operations Manager Module

Operations Manager 2007 R2 ships with Operations Manager Shell which is simply a provider with associated cmdlets for working with System Center Operations Manager. The Operations Manager implementation of PowerShell uses [a console file which limits reusability](/2010/05/the-truth-about-sqlps-and-powershell-v2/), however we can easily build our own Operations Manager Module... 

## Creating the Module

This is a technique I learned from Joel Bennett ([blog](http://huddledmasses.org/)|[twitter](http://twitter.com/jaykul)) and previously applied to SQL Server in my post [Making A SQLPS Module](/2010/07/making-a-sqlps-module/). To create a module simply copy the [OpsMgr.psd1 file](http://poshcode.org/2425) to your _$homeDocumentsWindowsPowerShellOpsMgr_ directory and run the commented out code to copy the necessary files. As mentioned in the comments the files are available on any machine with the Operations Manager Shell installed, however once the module has been created you can copy the module folder and files to machines without the Operations Manager Shell or client installed. 

## How Do You Use the Operations Manager Shell?

Import the module and call the Start-OperationsManagerClientShell function: 
    
    
    import-module OpsMgr; Start-OperationsManagerClientShell -ManagementServerName: "MyOpsMgrServer" -PersistConnection: $true -Interactive: $true;

One "real-life" use is to put server in maintenance mode so alerts aren't generated while performing system maintenance. See [this post](/2009/10/operations-manager-shell/) for details on putting a server in maintenance mode.