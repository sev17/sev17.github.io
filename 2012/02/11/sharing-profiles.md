title: Sharing Profiles
link: http://sev17.com/2012/02/11/sharing-profiles/
author: Chad Miller
description: 
post_id: 10829
created: 2012/02/11 13:01:38
created_gmt: 2012/02/11 18:01:38
comment_status: open
post_name: sharing-profiles
status: publish
post_type: post

# Sharing Profiles

A question came up in a class I was teaching:  “How do you share your Powershell profiles across accounts?” Well, there’s documented way of sharing profiles on the same machine where you create an All Users profile. As stated in [help about_Profiles](http://technet.microsoft.com/en-us/library/dd315342.aspx) Powershell will look in the following locations for profiles: 

> 
>             Name                               Description
>             -----------                        -----------
>             $Profile                           Current User,Current Host
>             $Profile.CurrentUserCurrentHost    Current User,Current Host
>             $Profile.CurrentUserAllHosts       Current User,All Hosts
>             $Profile.AllUsersCurrentHost       All Users, Current Host
>             $Profile.AllUsersAllHosts          All Users, All Hosts

The All Users profile creation is a little odd in that you would need to create your profiles in the $pshome directory instead of the $home directory. The $pshome directory is the location where powershell.exe is installed i.e. C:WindowsSystem32WindowsPowerShellv1.0 while $home is under <user>DocumentsWindowsPowershell. I don’t like messing with storing my shared profiles in a system directory, so instead I use a different technique… 

## Enter Symlinks

I’ve [previously blogged about Symlinks](/2010/10/using-symlinks-in-powershell-scripts/), but I didn’t mention I use them for sharing profiles. _Note: This  requires Vista, Windows 7  or 2008 or higher OS_. So here’s the steps to share profiles using symlinks: 

  1. Create A WindowsPowershell folder or copy your existing WindowsPowershell to a shared location. I use C:UsersPublicDocuments.
  2. Make sure your accounts don’t have a WindowsPowershell folder under <user>Documents already.
  3. Start a classic Window command prompt as Administrator (that’s right don’t use Powershell to create symlinks)
  4. Change directories to the Documents folder for account you want to share profiles
  5. Run the following command
    
    
    c:Usersu00Documents>mklink /D WindowsPowerShell C:usersPublicDocumentsWindowsPowerShell
    symbolic link created for WindowsPowerShell <<===>> C:usersPublicDocumentsWindowsPowerShell

Repeat steps 4 and 5 for each account. You now can now share profiles and modules across accounts on the same computer.