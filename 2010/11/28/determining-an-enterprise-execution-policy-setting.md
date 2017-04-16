title: Determining An Enterprise Execution Policy Setting
link: http://sev17.com/2010/11/28/determining-an-enterprise-execution-policy-setting/
author: Chad Miller
description: 
post_id: 10500
created: 2010/11/28 17:34:49
created_gmt: 2010/11/28 22:34:49
comment_status: open
post_name: determining-an-enterprise-execution-policy-setting
status: publish
post_type: post

# Determining An Enterprise Execution Policy Setting

Windows PowerShell has the concept of execution policy that determines in which cases script and configuration files are able to run. The various execution policy settings are described in about_Execution_Policies. Run Get-Help about_execution_policies or see the [online version](http://technet.microsoft.com/en-us/library/dd347641.aspx) for additional information.

The default setting for the execution policy is restricted which means PowerShell will not run scripts or load your profile. Of course, as a IT Pro the default setting is too restrictive for either your workstation or the servers you manage. When adopting PowerShell in an enterprise environment you’ll need to determine the most appropriate execution policy setting. In order to run scripts this means you’ll need to choose between AllSigned or RemoteSigned setting._ Note: Restricted, UnRestricted and ByPass settings are not appropriate for obvious reasons._

After research, testing and careful consideration I propose RemoteSigned enforced via Group Policy to be the most appropriate setting for an enterprise environment for the following reasons:

  1. Microsoft’s products do not support an AllSigned Policy 
  2. The Execution Policy is Not an Effective Security Feature 
  3. Using AllSigned is unnecessary in a well-managed environment 
  4. No one uses AllSigned 
  5. You are not a Software Vendor 

## Microsoft’s Products Do Not Support an AllSigned Policy

In the course of research and testing I’ve identified at least three products Exchange, IIS and PowerShell ISE where AllSigned isn’t supported. There are probably additional products and if you know of others please comment. The details of each products lack of AllSigned support is described below… 

### Exchange

All the files used by the Exchange Management Tools and are signed.  You would think EMS would run using an AllSigned policy, however, if you try launching the EMS with the execution policy set to ALLSIGNED you’ll see a series of prompts like this:

Do you want to run software from this untrusted publisher?

_File C:Program FilesMicrosoftExchange ServerV14binRemoteExchange.ps1 is published by CN=Microsoft Corporation, _

_OU=MOPR, O=Microsoft Corporation, L=Redmond, S=Washington, C=US and is not trusted on your system. Only run scripts  from trusted publishers._

_[V] Never run  [D] Do not run  [R] Run once  [A] Always run  [?] Help (default is "D"):_

This works up to the point where the remote module will be imported to the local session.  There are two format.ps1xml files created and used.  Since they are dynamically created   there are no signatures.  Hence the reason REMOTESIGNED works and ALLSIGNED does not.

The REMOTESIGNED requirement for Exchange is documented in two places:

**[Install Windows Management Framework](http://technet.microsoft.com/en-us/library/dd335147.aspx)**

**[Troubleshooting the Exchange Management Shell](http://technet.microsoft.com/en-us/library/dd351136.aspx)**

This applies to all machines where the Exchange Management tools are installed.

_Note: This is really not an Exchange implementation issue so much as a PowerShell remoting issue which means that this issue will come up again with other products._

### IIS

PowerShell was included in the Microsoft Common Engineering Criteria each product team is responsible for building their own PowerShell component invariably some inconsistencies creep in. One of those inconsistencies has been around script signing. It would seem some product teams are failing to sign their components. As an example IIS 7.5 is unsigned. The IIS 7.5 module will not run using an AllSigned policy.

### PowerShell ISE 

Did you know PowerShell ISE breaks script signing by default? The following connect item documents the issue:

 <https://connect.microsoft.com/PowerShell/feedback/details/483431/set-authenticodesignature-fails-on-scripts-created-from-ise>. 

The reason PowerShell ISE by default lacks support for script signing is due to the default encoding being Unicode Big Endian. As documented in the connect item, there is a workaround to change PowerShell ISE encoding, however this should be cause of concern since the issue shows how script signing isn’t being tested internally.

## The Execution Policy is Not an Effective Security Feature

That’s right, using AllSigned does not prevent scripts from being run. As demonstrated by Tome Tanasovski ([blog](http://powertoe.wordpress.com/)|[twitter](http://twitter.com/toenuff)) in his blog post [Attacking Execution Policy](http://powertoe.wordpress.com/2010/10/22/attacking-execution-policy/). The following PowerShell code will read in a PowerShell script called test.ps1 and execute even if the execution policy is set to AllSigned or Restricted:
    
    
    PowerShell -noprofile -Command "PowerShell -noprofile -encodedCommand ([Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes((Get-Content .test.ps1 |%{$_}|out-string))))"

As Tome notes “Microsoft doesn’t consider this an exploit because execution policy was never intended to be a security feature.  In a way this makes sense.  It’s real intention is to prevent a user from accidentally double-clicking on an untrusted PowerShell script.”

A second method for circumventing an AllSigned policy is to use the –ExecutionPolicy parameter for powershell.exe. Provided your execution policy isn’t being enforced through Group Policy:
    
    
    powershell.exe –executionpolicy -RemoteSigned

The above command will launch a powershell host with the execution policy set to remotesigned. This will override previous set powershell execution policy unless the execution policy is being enforced via Group Policy.

Note: Just because execution policy can be circumvented does not make PowerShell less secure than any other scripting or programming language. The concept of execution policy doesn't even exist in other scripting languages. Keep in mind you can only perform tasks you have permissions to in PowerShell. If you're using Vista or higher operating system and have UAC enabled you this means PowerShell scripts won't change settings without prompting for approval. What got scripting languages in trouble in the past has been scripts that automatically download and execute or scripts that are simply double-clicked and execute. These issues have been addressed many years ago even in VBScript. PowerShell does not automatically execute and by default a PowerShell script file (.ps1) is associated with Notepad, so double-clicking a file will not execute the script.

## Using AllSigned is Unnecessary in a Well-Managed Environment

## Comments

**[Jonathan Jespersen](#194 "2011-07-27 11:19:56"):** I work in an enterprise environment and we operate with an AllSigned policy enforced by Group Policy in our Internet-facing environment. It is used as a measure of protection from inadvertent execution. In our case, there is not much difference between using RemoteSigned or AllSigned as most of our scripts are run from a UNC path. In the small percentage of cases where the script is local to the machine, the additional overhead of signing is negligible given that we have a script for signing other scripts (and it's possible that such scripts could be executed from a remote share). In the case of Microsoft products, we address the unsigned files in IIS and SharePoint 2010 (signed with a now expired certificate) by signing them ourselves at the time the products are installed. That does not mean AllSigned is right for everyone or should be the best practice, but it's worked for us for more than 4 years.

**[Chad Miller](#195 "2011-07-27 12:30:13"):** That is an interesting way to deal with UNC paths. Signing some Microsoft products may work up to a point. I'm not sure the approach would work for Exchange.

