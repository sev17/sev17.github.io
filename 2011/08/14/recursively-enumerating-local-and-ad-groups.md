title: Recursively Enumerating Local and AD Groups
link: http://sev17.com/2011/08/14/recursively-enumerating-local-and-ad-groups/
author: Chad Miller
description: 
post_id: 10730
created: 2011/08/14 12:16:30
created_gmt: 2011/08/14 16:16:30
comment_status: open
post_name: recursively-enumerating-local-and-ad-groups
status: publish
post_type: post

# Recursively Enumerating Local and AD Groups

If you ever need to flatten out groups which may include nested local and AD groups there’s a really easy way to do this in  the [System.DirectoryServices.AccountManagement.GroupPrincipal GetMembers method](http://msdn.microsoft.com/en-us/library/bb339975.aspx). Here’s some PowerShell code which works against both local and AD groups. The code can easily be adapted into a function and in fact I’m using similar code in the SQLPSX project for [SQL Server permission auditing](http://sqlpsx.codeplex.com/SourceControl/changeset/view/62407#536173):
    
    
    add-type -AssemblyName System.DirectoryServices.AccountManagement
    
    $domain = "$env:computername"
    $groupname = "Administrators"
    
    #Determine if domain a machine or domain
    try {
        $domainName = [System.DirectoryServices.ActiveDirectory.Domain]::GetComputerDomain() | select -ExpandProperty Name
        $isDomain = $domainName -match "$domain."
    }
    catch {
        $isDomain = $false
    }
    
    if ($isDomain)
    { $ctype = [System.DirectoryServices.AccountManagement.ContextType]::Domain }
    else
    { $ctype = [System.DirectoryServices.AccountManagement.ContextType]::Machine }
    
    #Create objects to filter based on group name and ContextType--Domain or Machine
    $principal = new-object System.DirectoryServices.AccountManagement.PrincipalContext $ctype,$domain
    $groupPrincipal = new-object System.DirectoryServices.AccountManagement.GroupPrincipal $principal,$groupname
    $searcher = new-object System.DirectoryServices.AccountManagement.PrincipalSearcher 
    $searcher.QueryFilter = $groupPrincipal
    
    #Note GetMembers($true) recursively enumerates groups members while GetMembers() simply enumerates group members
    $searcher.FindAll() | foreach {$_.GetMembers($true)}

## Comments

**[Johnny Venter](#273 "2011-12-09 13:56:04"):** I used the script and it works pretty well. In the output, how can I separate the nested groups by their group name? Thank you,

**[Chad Miller](#274 "2011-12-09 21:18:35"):** It's much harder to do that. You'll need to walk the group membership and store the results in some kind of structure. Maybe a hash table. I don't have code to do that and have never needed to. For my purposes just flattening out membership was sufficient BTW the Quest AD cmdlets will also show nested groups however Quest takes the same approach of flattening out membership without preserving hierarchy.

