title: Using Symlinks in PowerShell Scripts
link: http://sev17.com/2010/10/14/using-symlinks-in-powershell-scripts/
author: Chad Miller
description: 
post_id: 10454
created: 2010/10/14 20:05:36
created_gmt: 2010/10/15 00:05:36
comment_status: open
post_name: using-symlinks-in-powershell-scripts
status: publish
post_type: post

# Using Symlinks in PowerShell Scripts

[Symlinks](http://en.wikipedia.org/wiki/Symbolic_link) are heavily used in Linux/Unix environments to provide an abstraction for file and directory locations and with the introduction of symbolic links in Vista/Win7/Windows 2008 we can take advantage of this handy little feature. One use case for symlinks in Linux scripting is to setup a link to a script file and then based on the file name from which the script is called provide alternate code execution paths. This is a bit of an advanced scripting technique which I thought would be interesting to reproduce in PowerShell/Windows. Keep in mind, there are other ways to accomplish the same desired outcome in PowerShell including reusing functions… Let’s take a look at an example. First create a PowerShell script called Test-SymLink.ps1 as follows: 
    
    
    function Test-SymLink
    {
        $scriptName = [system.io.path]::GetFilename($myinvocation.ScriptName)
    
        if ($scriptName -eq 'link1.ps1')
        {Write-Host "$scriptName -- Do Stuff"}
        elseif ($scriptName -eq 'link2.ps1')
        {Write-Host "$scriptName -- Do Other Stuff"}
    
    }
    
    Test-SymLink

Then create two symlinks to the Test-SymLink.ps1 script file using the mklink.exe utility from the regular cmd.exe window run as administrator: ![TestSymlink](http://images.sev17.com/TestSymlink_thumb.jpg) Next within PowerShell call the script using the two symlinks: 
    
    
    u00@Z003 C:Usersu00bin>.link1.ps1
    link1.ps1 -- Do Stuff
    u00@Z003 C:Usersu00bin>.link2.ps1
    link2.ps1 -- Do Other Stuff

As you can see the function resolved the “[system.io.path]::GetFilename($myinvocation.ScriptName)” to the name of the link which then allows you to take different code paths. This could make for some obscure code, so if you use this technique be sure to comment your script.

## Comments

**[Mark Broadbent](#181 "2010-10-17 12:45:41"):** Hi Chad, firstly good bit of code. Just wanted to point out/ publicize that symbolic links have been around in Windows for a very long time contrary to the common perception, it was just not obvious to people how to do it. Various ways to do it, but by far the easiest used to be by using the linkd tool (provided by MS in a reskit/ download). Still now thats not necessary any longer with the newer Windows releases.. Regards, Mark.

**[Chad Miller](#182 "2010-10-17 13:21:39"):** From what I've read the support for things like junction points (kind of like symlinks) in Windows versions prior to Vista was a little half-baked. Here's an old post from Scott Hanselman that discusses junction points and some history. There's also a link to Wikipedia in the post: http://www.hanselman.com/blog/MoreOnVistaReparsePoints.aspx In any case symlinks are very powerful especially outside of scripting. I've seen Informix DBAs manage seamless upgrades by using symlinks on Unix systems. It would be nice if they were better integrated into Explorer and PowerShell.

**[Mark Broadbent](#183 "2010-10-17 14:29:34"):** Thanks for additional info. Yeah directory symbolic links (junctions) aren't the whole story and it does sound like mlink is a darn sight more powerful -will have a play at some point. Agree with you on the integration side, but things do seem to be improving though. Maybe next Win version or service pack...

**[Peter Kriegel](#185 "2012-02-15 12:54:59"):** The PowerShell Community Extensions (PSCX) having a set of cmdlets to deal with Symlinks, Junction Points and Hardlinks. See my German Blog Post (you can Translate it by Using the Translate Button on the Post Page) <http://www.admin-source.de/BlogDeu/38/verknuepfungen-im-ntfs-dateisystem-hardlink-junction-point-oder-symlink-mit-powershell>

