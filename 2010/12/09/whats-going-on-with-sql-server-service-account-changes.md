title: What’s Going on with SQL Server Service Account Changes?
link: http://sev17.com/2010/12/09/whats-going-on-with-sql-server-service-account-changes/
author: Chad Miller
description: 
post_id: 10517
created: 2010/12/09 22:44:27
created_gmt: 2010/12/10 03:44:27
comment_status: open
post_name: whats-going-on-with-sql-server-service-account-changes
status: publish
post_type: post

# What’s Going on with SQL Server Service Account Changes?

For years we’ve been told you should use Enterprise Manager in SQL Server 2000 or SQL Server Configuration Manager in SQL 2005 or higher to change the SQL Service account. Why you may ask well, because according to the [documentation](http://msdn.microsoft.com/en-us/library/ms143504.aspx) Configuration Manager does “magic” stuff to re-ACL things for the new account. I decided to put the notation that SQL Server files, folders or Registry keys get Re-ACL’d when Enterprise Manager (SQL 2000) or SQL Server Configuration Manager (SQL Server 2005 or higher) is used to change the SQL Server service account to a few tests… 

## Setup

  1. Use a test machine (even your local workstation or laptop will work)
  2. Note: Since I’m using test machines the SQL instances were setup and running under Local System account and then changed to an AD account with no assigned permissions or groups other than the default AD Users group.
  3. Create a test local or AD account, in my case I created an account named “ZZZ”
  4. We’ll need the SID for the account when we dump the ACLs, from a PowerShell host obtain the SID for the account using the following commands:
    
    
    $domain = "CORP"
    $user = "ZZZ"
    $ntAccount = new-object System.Security.Principal.NTAccount($domain, $user)
    $sid = $ntAccount.Translate([System.Security.Principal.SecurityIdentifier])
    $sid.value

> For my account this returns: **S-1-5-21-3799912004-3082830092-2763560395-1116**

## The Tests

  * For SQL Server version 2000, 2005 and 2008 I’ll use the recommended utilities to change the service account.
  * I’ll then run a the following PowerShell commands to export the ACL's for all SQL Server files, folders and registry keys:
    
    
    #FileSystem
    cd "C:program filesMicrosoft SQL Server"
    dir -recurse | get-acl | select pschildname, owner, sddl | export-csv -NoTypeInformation -Path $homesqlfile.csv
    #Registry
    cd "HKLM:SOFTWAREMicrosoftMicrosoft SQL Server"
    dir -recurse | get-acl | select pschildname, owner, sddl | export-csv -NoTypeInformation -Path $homesqlreg.csv

Finally I’ll look for any ACL’s assigned to the SID of my new service account (_S-1-5-21-3799912004-3082830092-2763560395-1116)._

### SQL Server 2000 Enterprise Manager

  1. From SQL Server 2000 Enterprise Manager change the service account to the new account. The GUI will prompt for SQL service restart.
  2. After SQL Server has restarted, run the PowerShell commands to export ACL’s as instructed in “The Tests” section.
_I’ve posted my output files SQL2K_sqlfilefull2.csv and SQL2K_sqlregfull2.csv in the [accompanying download](http://wth5na.blu.livefilestore.com/y1pbemWC1gYhha9uUp8C-iqCeR98rkYTXkiHB9RePvaD8JhCWO_X8ru_7lQCNfDiiac1_9kTvPd_V8nS0lTKFT6VwppLQfOrqru/chgsvcacct.zip?download&psid=1)._ Looking at the ACL’s in the CSV file we see, yes in fact Enterprise Manager did re-ACL both folders, files and registry keys. A typical entry includes the file/folder name and the following SDDL syntax with the service account’s SID: **…(A;OICI;FA;;;S-1-5-21-3799912004-3082830092-2763560395-1116)…** Checking off Enterprise, we’ll move to SQL 2005 and 2008… 

### SQL Server Configuration Manager (2005/2008)

Since the results of both SQL Server 2005 and 2008 were the same, I’ll summarize the steps and results together. 

  1. From SQL Server Configuration Manager change the service account to the new account. The utility will prompt for a SQL Server restart.
  2. After SQL Server restarts run the  PowerShell commands to export ACL’s as instructed in “The Tests” section.
_I’ve posted my output files SQL2K5_sqlfilefull2.csv, SQL2K8_sqlfilefull2.csv, and SQL2K5_sqlregfull2.csv and SQL2K8_sqlregfull2.csv in the [accompanying download](http://wth5na.blu.livefilestore.com/y1pbemWC1gYhha9uUp8C-iqCeR98rkYTXkiHB9RePvaD8JhCWO_X8ru_7lQCNfDiiac1_9kTvPd_V8nS0lTKFT6VwppLQfOrqru/chgsvcacct.zip?download&psid=1)._ This time, when we look at the ACL’s only two files show a change not in the permissions, but in the owner of the file ERRORLOG and log_*.trc (the current default trace file). The change in file ownership isn’t surprising as the new service account will create the files on restart which has nothing to do with re-ACL-ing. Also there aren’t any changes in ACL’s for the registry. As an additional check I verified the service account wasn’t auto-magically added to any of the built-in Windows groups SQL Server 2005 or higher creates on the local machine.** EDIT: ****[Dan Jones posted a blog entry clarifying the behavior**](http://blogs.msdn.com/b/dtjones/archive/2010/12/15/changing-service-account-amp-service-account-password.aspx)** which I verified in SQL Server 2005/Windows 2003 the account is added to SQLServer2005MSSQLUser% local group by SQLCM. As noted by Dan this is OS and SQL version specific behavior where if you are running pre-Vista (Windows 2003/XP) or SQL 2005 group membership is modified**. So, where’s the “magic” re-ACL-ing?

## Summary

According to my tests none of the ACL’s were changed in SQL 2005 or higher while in SQL 2000 they were. So is the BOL documentation wrong or am I missing something? Or is the “Only use SQL Server Configuration Manager” directive a “SQL Myth?” 

# [Download](http://wth5na.blu.livefilestore.com/y1pbemWC1gYhha9uUp8C-iqCeR98rkYTXkiHB9RePvaD8JhCWO_X8ru_7lQCNfDiiac1_9kTvPd_V8nS0lTKFT6VwppLQfOrqru/chgsvcacct.zip?download&psid=1)

## Comments

**[Scott](#196 "2010-12-10 09:07:11"):** Good article. I wonder how not changing the service account using the SQL Configuration Manager affects the the Service Master Key. http://technet.microsoft.com/en-us/library/ms187788.aspx http://support.microsoft.com/kb/283811

**[Chad Miller](#197 "2010-12-10 10:01:11"):** Good point about the encryption piece. I don't use TDE, but keep in mind the testing I've done is using SQL Configuration Manager. It should be an easy test to see what happens with a TDE database when changing password with Service Control instead.

**[Chad Miller](#198 "2010-12-10 21:21:07"):** Ran a quick test of Service Master Key. Test 1 -- Change account using SQL Configuration Manager, then run select * from sys.symmetric_keys. Noticed modify_date is updated. Test 2 -- Change account using Windows Service Control, again run select * from sys.symmetric_keys. Also notice modify_date is updated.

**[Scott](#199 "2010-12-10 21:59:07"):** It looks like both end with the same results. Good to know! Thanks for testing that Chad!

**[Chad Miller](#200 "2010-12-16 07:50:03"):** More information on "Service SID", apparently this feature was added to Windows 2008/Vista OS or higher. Allows assignment of ACL to service and not service account! Explains why SQL 2008 service account behaves the way it does. http://blogs.technet.com/b/askperf/archive/2008/02/03/ws2008-windows-service-hardening.aspx

**[David Lathrop](#201 "2011-01-25 15:45:14"):** With SQL Server 2005, MS started using the "principal of least privelege" with SQL Server, and changed how the service accounts are handled. If you check the Local Windows Groups after installing SQL Server, you will find a set of 7 groups are created that start with "SQLServer200. ACLs are created for these groups based on what they need. (I'll use 2005 names here, but same seems to apply to 2008.) SQLServer2005DTSUser$myserver (replace with your server name) is used for the SSIS Service which is one per server. Same for the SQL Browser group (SQLServer2005BrowserUser$myserver). The other groups are created for each SQL instance, and contain both the server name and the instance name (or MSSQLSERVER for the default instance). SQLServer2005MSSQLUser$myserver$myinstance is used to grant permissions to the SQL Server Engine, while SQLServer2005SQLAgentUser$myserver$myinstance grants permission to the corresponding SQL Server Agent. Similar groups for Full Text Engine, AD Helper and Notification services. If you check the ACL for specific SQL Server folders, you will find one or more of these groups granted permissions, not the actual service accounts. When we created a separate folder on another drive for Transaction Logs in 2005 after the initial install, we granted permissions to the SQL Engine and SQL Agent groups, and that has seemed to fine even with a change of service accounts. I haven't found anywhere that said the proper method of assigning permissions to files and folders, but based on observation I've assumed this is it. [A lot of the BoL, TechNet and MSDN notes and articles are from SQL 2000 and don't seem to have been rewritten for SQL 2005 and later to mention these changes.] I think the main thing Configuration Manager does when you change service accounts is change the membership of these local windows groups. [This will not affect a running service, because it's "Windoows security token" is created when the service starts.] As far as the encryption hierarchy, the only thing that I think should be affected is the copy of that instance's Service Master Key (SMK) that is encrypted and stored in the registry. This encryption is done using DPAPI using the SQL Engine service account's keys, and is entirely outside SQL Server. [This will not affect copies of the SMK stored anywhere in a database, including sys.symmetric_keys.] If this registry copy isn't re-encrypted for the new service account, then when you restart the SQL Engine, it will not have the proper SMK value. However, you may not notice unless you actually try to use something encrypted by the SMK. I'm not sure of the details of encrypting SQL objects, but credentials, linked server logins with SQL passwords, and a few other things might be impacted if you actually used them.

