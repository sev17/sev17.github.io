title: ThrowAway Scripts
link: http://sev17.com/2009/08/12/throwaway-scripts/
author: Chad Miller
description: 
post_id: 9973
created: 2009/08/12 10:22:00
created_gmt: 2009/08/12 14:22:00
comment_status: open
post_name: throwaway-scripts
status: publish
post_type: post

# ThrowAway Scripts

Although I tend to write a lot of formal scripts, as part of the development process I'll explore an object looking at its data and available properties/methods using get-member. I'll then test individual pieces before glueing things together. However sometimes it isn't even necessary to write a well-crafted script and instead  I'll create a so-called "throwaway script" to accomplish a specific task. These scripts are especially useful when you will only need to run something once and the time to create a script out weighs the time to do the thing manually. Case in point, I had a need to extract the email addresses from an Exchange 2003 public folder with 100+ emails for an upcoming PowerShell club meeting.

I've done a little bit of work with Outlook COM objects. There's probably a way to do this on the Exchange side, but this works well enough and in about 10 minutes I created the following script:

$outlook = **new-object** -comobject "Outlook.Application" $nameSpace = $outlook.GetNamespace("MAPI") $folder = $nameSpace.GetDefaultFolder(18) $posh = $folder.Folders | ?{$_.name -**eq** "Technology"} | %{$_.Folders} | ?{$_.name -**eq** "Public"} | %{$_.Folders} | ?{$_.Name -**eq** "Powershell RSVPs"} $posh.Items | select SenderEmailAddress 

A quick note about the GetDefaultFolder method, 18 is the enum that represents public folders. You can change the value to another default folder including Inbox and Sent items, if needed. The list of available values can be found on [here](http://msdn.microsoft.com/en-us/library/bb208072.aspx).

I thought this was an interesting script, hopefully it saves someone a few minutes of time should they need to do a similar task.