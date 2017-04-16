title: Delegated SQL Server Administration with Powershell
link: http://sev17.com/2011/11/06/delegated-sql-server-administration-with-powershell/
author: Chad Miller
description: 
post_id: 10757
created: 2011/11/06 19:16:48
created_gmt: 2011/11/07 00:16:48
comment_status: open
post_name: delegated-sql-server-administration-with-powershell
status: publish
post_type: post

# Delegated SQL Server Administration with Powershell

Providing delegated administration to groups that need to perform various security functions has always been a difficult task, but thanks to Powershell V3 (currently in CTP 1 as of this blog post) and PowerGUI we have new tools to provide a solution.

## The Problem

You have groups outside of database administration that need to have the ability to manage security of SQL Server logins, users, and roles. Using the out-of-the-box SQL Server roles or permissions doesn’t quite handle all of these use cases and you want to avoid  putting these groups into the sysadmin server role on SQL Server for obvious reasons. One thing you may consider is placing these groups into the  securityadmin server role, but the problem with the securityadmin role is the lack of certain rights. The securityadmin role can only add logins to the instance and not users to databases or in turn users to database roles.  You could add your delegated security  group to every database as db_securityadmin, but this would be difficult to maintain in a highly dynamic environment with thousands of databases. There’s also a larger problem of auditing the actions your security administrators perform. Ideally you want to show every security change is related to specific documented change order. 

You could use the various server and database level permissions over the fixed role approach and in fact that is what Books Online [recommends](http://msdn.microsoft.com/en-us/library/ms175892\(v=SQL.105\).aspx), however you’ll have the same issues of per database permissions and auditable actions to overcome. If this problem statement sounds unfamiliar to you then you probably haven’t had deal with segregation of duties and reducing administration access that has become prevalent in our post-SOX IT world.

## A Solution

Looking at web-based applications which run under a service account , you’ll notice  normal users and administrators users as part of almost any applications. Neither group has direct access to the database systems they touch instead a service account is used. A simple idea which can easily be applied to administration functions all we need to do is create a web-based application, assign a service account the necessary rights to perform administration tasks and restrict access to the web-based application. What’s that? You’re not web developer? Well, neither am I. Here’s where Powershell V3 and PowerGUI can help.

### Powershell V3 Delegated Administration

One of the simple, but really useful changes made to the Powershell V3 is the ability to delegate administration by setting up  runas credentials for a remoting configuration. In the CTP 1 download there’s an example script called runas.ps1 located under **SamplesWindowsPowerShellDelegatedAdmin** which demonstrates the functionality. This presents an interesting an idea, instead of having your security users connect to each SQL Server they could connect to a single “Proxy” server. The proxy server would then connect to various SQL Servers on their behalf.  Of course the account used for the delegated administration will need to have the necessary rights perhaps even sysadmin access. As an added bonus by using Powershell remoting the security administrator’s machine doesn’t need SQL Server tools or SMO installed. The only thing they needed is Powershell.

If you go this route you’ll want to create a distinct Session Configuration (endpoint) and ACL the configuration to the appropriate groups. You’ll l also need to create the various Powershell functions for the administration tasks on the remote server, here’s where I created a module called SqlProxy.

### SQLProxy Module

SQL Server lacks Powershell coverage for security administration, so I created [SqlProxy module](http://poshcode.org/3040) which provides various functions for managing SQL Server logins, users and roles. In addition, because auditability is a key requirement I’ve added logging to a custom Windows Eventlog called “SqlProxy” for every function which change security settings. The module can be used in Powershell V2 or V3 with or without remoting. When used within a delegated administration setting the module will be loaded into the remote session and anyone granted access to the remote endpoint will be able to execute the SqlProxy functions. We could say our problem of providing delegated administration is solved, however unless your security admins are very adapt at Powershell you’ll probably want to provide GUI.

### PowerGUI SQLProxy PowerPack

Because PowerGUI provides an easy way to create MMC-style UI’s over Powershell, I applied PowerGUI to the SQLProxy module and created the [SqlProxy PowerPack](http://www.powergui.org/entry.jspa?externalID=3742&categoryID=54). The security administrators can now use GUI-based tool without having to know Powershell. The complete solution does requires some additional setup and configuration as documented below.

## Putting it Together

![sqlproxydiag](http://images.sev17.com/sqlproxydiag_thumb.png)

_Diagram courtesy of __[yuml.me_](http://yuml.me)

A security administer will use PowerGUI with the SqlProxy PowerPack installed to connect to a proxy server via Poewershell remoting. The proxy server has a session configuration with delegated credentials. The proxy server also has a SqlProxy module which is then used to connect to the various SQL Servers through SMO. _Note Powershell remoting is not used between the proxy server and SQL Server. The only area where Powershell remoting is used is between the user’s machine and the proxy server._

### Setup

#### Setup Proxy Server

##### Requirements:

  * Windows 7 with Sp1, Windows 2008 R2 with SP1 or Windows 8. 
  * Powershell V3 CTP1 or higher 

##### 

##### 

##### 

##### Installation

1\. Install SMO or SQL Server clients tools (Express edition will work) 

2, Copy the SqlProxy.psm1 module from <http://poshcode.org/3040> to C:Windowssystem32WindowsPowerShellv1.0ModulesSqlProxy folder

3\. Create the SqlProxy Eventlog and Source by running following command 
    
    
    New-EventLog -LogName SqlProxy -Source SqlProxy

4\. Create the SqlProxy Session Configuration with delegated credentials
    
    
    $cred = Get-Credential
    Register-PSSessionConfiguration -Name "SqlProxy" -RunAsCredential $cred

5\. ACL the SqlProxy session configuration to the appropriate AD groups (by default only administrators on the proxy server will have access to the remote session):
    
    
    Set-PSSessionConfiguration -Name SqlProxy –ShowSecurityDescriptorUI

6\. **Optional**: As noted on the Powershell team blog [the default settings  of Powershell remoting are way too low](http://blogs.msdn.com/b/powershell/archive/2010/05/03/configuring-wsman-limits.aspx). The low default settings are particularly problematic in a fan-in scenario like SqlProxy. You should make Powershell remoting more robust by changing the default concurrent and memory settings:
    
    
    cd WSMan:\localhost\Shell
    Set-Item MaxShellsPerUser 25
    Set-Item MaxConcurrentUsers 25
    Set-Item MaxMemoryPerShellMB 1024
    
    #Do the same for the SqlProxy Session Configuration
    
    cd WSMan:\localhost\Plugin\SqlProxy\Quotas
    Set-Item MaxShellsPerUser 25
    Set-Item MaxConcurrentUsers 25
    Set-Item MaxMemoryPerShellMB 1024

#### Setup Client

##### Requirements:

  * Powershell V2 or higher (that’s right you can remote from V2 client to V3 server!)

## Comments

**[Bob Beauchemin](#275 "2011-11-09 12:04:11"):** Hi Chad, I'm curious about your statement: "...manage security of SQL Server logins, users, and roles. Using the out-of-the-box SQL Server roles or permissions doesn’t quite handle all of these use cases" When is this true? There are nice granular permissions around users, logins, and roles, you don't need to use sysadmin. And with the advent in SQL Server 2005 of signed procs and execute as, there's no permission (even DBCC and bulk copy, which still don't use the granular permissions) that you can't hide behind a stored procedure, and give user permission only to the procedure. Just curious... BTW, nice posting.

**[Chad Miller](#276 "2011-11-09 13:54:27"):** Thanks Bob. Let me walk through a scenario. My security user needs to be able to add/remove logins, add/remove users to databases, add/remove database users from database roles, and add/remove logins from server roles. Focusing on post SQL 2005 (incidentally I still have some 2000 to deal with) I can grant the user ALTER ANY LOGIN , but this has zero permissions to actually add users to databases. In order to allow my security user to also handle database security I would need to apply the following permissions to every database ALTER ANY APPLICATION ROLE, ALTER ANY ROLE, CREATE SCHEMA ON ANY DATABASE, VIEW DEFINITION. In an environment with thousands of databases many of which are installed by applications maintaining these permissions at the database level is very difficult. What I'd like and what's missing is the ability to say TO ANY DATABASE at the server level for the by-database permissions. This would be kind of like some of the ANY SCHEMA permissions in Oracle (SQL database ~= Oracle schema). The granularity I'm looking for is less granular than sysadmin, but not granular as by-database--is this medium grain :) ? A security admin that can only add logins to servers is useless and maintaining by database permissions is an administration nightmare. You could work around the issue in all kinds of creative ways. Maybe add the database permissions to model or setup a SQL Job or setup a PBM policy, etc. There are pros/cons to each approach. If we take your proposal as an example: Pros: Solve the problem, use least privilege Cons: No GUI, need to deploy procedure on every server, no auditing of actions (you could include logging in our customer procedure) The big draw back I see with a custom stored procedure is that you can no longer use SSMS and my security administrators will not have a non-GUI approach. The way my solution is designed there is little typing which equals fewer errors. So, you'll still need to layer a GUI on top of the custom procedure. You could use Powershell and PowerGUI to do that, but then the solution would start to look similar to what I've presented. I'm certainly open to other ideas on how on to solve this problem, but the solution needs to provide a GUI, be easily integrated into a large dynamic environment, provide for full auditing and support SQL 2000.

