title: Building A Remote Desktop Manager Connection List
link: http://sev17.com/2010/06/15/building-a-remote-desktop-manager-connection-list/
author: Chad Miller
description: 
post_id: 10329
created: 2010/06/15 15:02:51
created_gmt: 2010/06/15 19:02:51
comment_status: open
post_name: building-a-remote-desktop-manager-connection-list
status: publish
post_type: post

# Building A Remote Desktop Manager Connection List

[Remote Desktop Connection Manager](http://www.microsoft.com/downloads/details.aspx?FamilyID=4603c621-6de7-4ccb-9f51-d53dc7e48047&displaylang=en) or RDCMan is a free download from Microsoft for managing multiple remote desktop connections. The functionality provided by RDCMan is above and beyond what you'll find in the built-in Remote Desktop MMC and comparable to other 3rd party Remote Desktop utilities. If you're connecting to multiple remote desktop sessions, RDCMan is a tool worth looking into. Because my environment has hundreds of servers, one of the things I look for in management tools is a method to import an existing list of servers.  Fortunately the XML file called an RDCMan file used for the connection list in RDCMan is a simple structure you can build yourself. A quick web search turns up a post by [Jan Egil Ring](http://blog.powershell.no/) titled [Dynamic Remote Desktop Connection Manager connection list](http://blog.powershell.no/2010/06/02/dynamic-remote-desktop-connection-manager-connection-list/). The approach Jan takes builds an RDCMan file from the computers in Active Directory using PowerShell which isn't quite what I want, but it gives me an idea on using T-SQL/XQuery... As a SQL Server DBA I maintain a list of SQL Servers and groups in a [Central Management Server](http://msdn.microsoft.com/en-us/library/bb895144.aspx) or CMS. A CMS is not only useful for maintaining a central list of SQL Servers, driving [Policy Based Managment](http://msdn.microsoft.com/en-us/library/bb510408.aspx), and executing multi-server queries but can also be used as input for other things including building an RDCMan file. Because the CMS data is already in SQL Server tables it's easier to use a T-SQL approach to shape the XML. The following T-SQL query creates an RDCMan XML file from CMS servers and groups. The same servers and groups in your CMS will be represented in the resulting RDCMan file: 
    
    
    DECLARE @xml XML
    
    ;WITH [file] AS
    (SELECT
    1 AS 'RDCMan'
    ,2.2 AS 'version'
    ,'CMS' AS 'name'
    ,'True' AS 'expanded'
    ,'Generated from CMS' AS 'comment'
    ,'FromParent' AS 'logonCredentials'
    ,'FromParent' AS 'connectionSettings'
    ,'FromParent' AS 'gatewaySettings'
    ,'FromParent' AS 'remoteDesktop'
    ,'FromParent' AS 'localResources'
    ,'FromParent' AS 'securitySettings'
    ,'FromParent' AS 'displaySettings'
    )
    ,grp AS
    (SELECT
     server_group_id
    ,name
    ,'True' AS 'expanded'
    ,'Generated from CMS' AS 'comment'
    ,'FromParent' AS 'logonCredentials'
    ,'FromParent' AS 'connectionSettings'
    ,'FromParent' AS 'gatewaySettings'
    ,'FromParent' AS 'remoteDesktop'
    ,'FromParent' AS 'localResources'
    ,'FromParent' AS 'securitySettings'
    ,'FromParent' AS 'displaySettings'
    FROM msdb.dbo.sysmanagement_shared_server_groups_internal
    WHERE is_system_object = 0)
    ,srv AS
    (SELECT
     DISTINCT server_group_id
    ,CASE
    	WHEN PATINDEX('%%',name) > 0 THEN
    		SUBSTRING(name,1, (PATINDEX('%%',name) -1 ))
    	WHEN PATINDEX('%,%',name) > 0  THEN
    		SUBSTRING(name,1, (PATINDEX('%,%',name) -1 ))
    	ELSE
    		name
    END AS name
    ,CASE
    	WHEN PATINDEX('%%',name) > 0 THEN
    		SUBSTRING(name,1, (PATINDEX('%%',name) -1 ))
    	WHEN PATINDEX('%,%',name) > 0  THEN
    		SUBSTRING(name,1, (PATINDEX('%,%',name) -1 ))
    	ELSE
    		name
    END AS 'displayName'
    ,'Generated from CMS' AS 'comment'
    ,'FromParent' AS 'logonCredentials'
    ,'FromParent' AS 'connectionSettings'
    ,'FromParent' AS 'gatewaySettings'
    ,'FromParent' AS 'remoteDesktop'
    ,'FromParent' AS 'localResources'
    ,'FromParent' AS 'securitySettings'
    ,'FromParent' AS 'displaySettings'
    FROM msdb.dbo.sysmanagement_shared_registered_servers_internal)
    
    SELECT @XML = (
    SELECT
    (SELECT
     name
    ,expanded
    ,comment
    ,(SELECT logonCredentials AS "@inherit" FOR XML PATH('logonCredentials'), TYPE)
    ,(SELECT connectionSettings AS "@inherit" FOR XML PATH('connectionSettings'), TYPE)
    ,(SELECT gatewaySettings AS "@inherit" FOR XML PATH('gatewaySettings'), TYPE)
    ,(SELECT remoteDesktop AS "@inherit" FOR XML PATH('remoteDesktop'), TYPE)
    ,(SELECT localResources AS "@inherit" FOR XML PATH('localResources'), TYPE)
    ,(SELECT securitySettings AS "@inherit" FOR XML PATH('securitySettings'), TYPE)
    ,(SELECT displaySettings AS "@inherit" FOR XML PATH('displaySettings'), TYPE)
    FROM [file]
    FOR XML PATH('properties'),TYPE)
    
    ,(SELECT 
    
     (SELECT
     g.name
    ,g.expanded
    ,g.comment
    ,(SELECT g.logonCredentials AS "@inherit" FOR XML PATH('logonCredentials'), TYPE)
    ,(SELECT g.connectionSettings AS "@inherit" FOR XML PATH('connectionSettings'), TYPE)
    ,(SELECT g.gatewaySettings AS "@inherit" FOR XML PATH('gatewaySettings'), TYPE)
    ,(SELECT g.remoteDesktop AS "@inherit" FOR XML PATH('remoteDesktop'), TYPE)
    ,(SELECT g.localResources AS "@inherit" FOR XML PATH('localResources'), TYPE)
    ,(SELECT g.securitySettings AS "@inherit" FOR XML PATH('securitySettings'), TYPE)
    ,(SELECT g.displaySettings AS "@inherit" FOR XML PATH('displaySettings'), TYPE)
    FROM grp g WHERE [group].server_group_id = g.server_group_id
    FOR XML PATH('properties'),TYPE)
    
    ,(SELECT
     s.name
    ,s.displayName
    ,s.comment
    ,(SELECT s.logonCredentials AS "@inherit" FOR XML PATH('connectionSettings'), TYPE)
    ,(SELECT s.connectionSettings AS "@inherit" FOR XML PATH('connectionSettings'), TYPE)
    ,(SELECT s.gatewaySettings AS "@inherit" FOR XML PATH('gatewaySettings'), TYPE)
    ,(SELECT s.remoteDesktop AS "@inherit" FOR XML PATH('remoteDesktop'), TYPE)
    ,(SELECT s.localResources AS "@inherit" FOR XML PATH('localResources'), TYPE)
    ,(SELECT s.securitySettings AS "@inherit" FOR XML PATH('securitySettings'), TYPE)
    ,(SELECT s.displaySettings AS "@inherit" FOR XML PATH('displaySettings'), TYPE)
    FROM srv s WHERE [group].server_group_id = s.server_group_id
    FOR XML PATH('server'),TYPE)
    FROM grp [group]
    ORDER BY [group].name
    FOR XML PATH('group'),TYPE)
    
    FROM [file]
    FOR XML AUTO, ELEMENTS, ROOT('RDCMan')
    )
    
    
    
    SET @XML.modify('
    insert 2.2 
    as first
    into (/RDCMan)[1]') 
    
    SET @xml.modify('insert attribute schemaVersion{"1"} as last into (RDCMan)[1]')
    
    SELECT @XML
    

To use run the query in the msdb database on your CMS and save the output as an **.rdg** file. Next, from Remote Desktop Connection Manager select _File->Open_ and select the RDCMan file. Once you've imported the list of servers, you'll want to take a look at the global settings and features of RDCMan. In particular you'll want to set Logon Credentials which will allow you to autologon to various servers. The code syntax highlighter I'm using seems to mess up the case of several items, so I'm including the SQL script for download:

## Comments

**[Steve Ledridge](#167 "2012-11-16 13:08:14"):** I found your script to be a great start but it did not handle nested groups and created a big mess with our data that uses heavily nested groups. This script resolves that and I hope it can be useful to others. GO USE [msdb] GO IF OBJECT_ID('dbo.RDCM_exstract_group_and_members') IS NOT NULL DROP function dbo.RDCM_exstract_group_and_members GO CREATE function dbo.RDCM_exstract_group_and_members(@key as int) returns xml begin return ( SELECT ( SELECT ( SELECT g.name ,g.expanded ,g.comment ,(SELECT g.logonCredentials AS "@inherit" FOR XML PATH('logonCredentials'), TYPE) ,(SELECT g.connectionSettings AS "@inherit" FOR XML PATH('connectionSettings'), TYPE) ,(SELECT g.gatewaySettings AS "@inherit" FOR XML PATH('gatewaySettings'), TYPE) ,(SELECT g.remoteDesktop AS "@inherit" FOR XML PATH('remoteDesktop'), TYPE) ,(SELECT g.localResources AS "@inherit" FOR XML PATH('localResources'), TYPE) ,(SELECT g.securitySettings AS "@inherit" FOR XML PATH('securitySettings'), TYPE) ,(SELECT g.displaySettings AS "@inherit" FOR XML PATH('displaySettings'), TYPE) FROM ( SELECT server_group_id ,parent_id ,name ,'True' AS 'expanded' ,'Generated from CMS' AS 'comment' ,'FromParent' AS 'logonCredentials' ,'FromParent' AS 'connectionSettings' ,'FromParent' AS 'gatewaySettings' ,'FromParent' AS 'remoteDesktop' ,'FromParent' AS 'localResources' ,'FromParent' AS 'securitySettings' ,'FromParent' AS 'displaySettings' FROM msdb.dbo.sysmanagement_shared_server_groups_internal \--WHERE is_system_object = 0 ) g WHERE g.server_group_id = @key FOR XML PATH('properties'),TYPE ) ,( SELECT dbo.RDCM_exstract_group_and_members(server_group_id) FROM msdb.dbo.sysmanagement_shared_server_groups_internal WHERE parent_id = @key FOR XML PATH(''),TYPE ) ,( SELECT s.name ,s.displayName ,s.comment ,(SELECT s.logonCredentials AS "@inherit" FOR XML PATH('connectionSettings'), TYPE) ,(SELECT s.connectionSettings AS "@inherit" FOR XML PATH('connectionSettings'), TYPE) ,(SELECT s.gatewaySettings AS "@inherit" FOR XML PATH('gatewaySettings'), TYPE) ,(SELECT s.remoteDesktop AS "@inherit" FOR XML PATH('remoteDesktop'), TYPE) ,(SELECT s.localResources AS "@inherit" FOR XML PATH('localResources'), TYPE) ,(SELECT s.securitySettings AS "@inherit" FOR XML PATH('securitySettings'), TYPE) ,(SELECT s.displaySettings AS "@inherit" FOR XML PATH('displaySettings'), TYPE) FROM ( SELECT DISTINCT server_group_id ,CASE WHEN PATINDEX('%%',name) > 0 THEN SUBSTRING(name,1, (PATINDEX('%%',name) -1 )) WHEN PATINDEX('%,%',name) > 0 THEN SUBSTRING(name,1, (PATINDEX('%,%',name) -1 )) ELSE name END AS name ,CASE WHEN PATINDEX('%%',name) > 0 THEN SUBSTRING(name,1, (PATINDEX('%%',name) -1 )) WHEN PATINDEX('%,%',name) > 0 THEN SUBSTRING(name,1, (PATINDEX('%,%',name) -1 )) ELSE name END AS 'displayName' ,'Generated from CMS' AS 'comment' ,'FromParent' AS 'logonCredentials' ,'FromParent' AS 'connectionSettings' ,'FromParent' AS 'gatewaySettings' ,'FromParent' AS 'remoteDesktop' ,'FromParent' AS 'localResources' ,'FromParent' AS 'securitySettings' ,'FromParent' AS 'displaySettings' FROM msdb.dbo.sysmanagement_shared_registered_servers_internal ) s WHERE s.server_group_id = @key FOR XML PATH('server'),TYPE ) FOR XML PATH('group'),TYPE ) ) end go DECLARE @xml XML ;WITH [file] AS ( SELECT 1 AS 'RDCMan' ,2.2 AS 'version' ,'CMS' AS 'name' ,'True' AS 'expanded' ,'Generated from CMS' AS 'comment' ,'FromParent' AS 'logonCredentials' ,'FromParent' AS 'connectionSettings' ,'FromParent' AS 'gatewaySettings' ,'FromParent' AS 'remoteDesktop' ,'FromParent' AS 'localResources' ,'FromParent' AS 'securitySettings' ,'FromParent' AS 'displaySettings' ) SELECT @XML = ( SELECT ( SELECT name ,expanded ,comment ,(SELECT logonCredentials AS "@inherit" FOR XML PATH('logonCredentials'), TYPE) ,(SELECT connectionSettings AS "@inherit" FOR XML PATH('connectionSettings'), TYPE) ,(SELECT gatewaySettings AS "@inherit" FOR XML PATH('gatewaySettings'), TYPE) ,(SELECT remoteDesktop AS "@inherit" FOR XML PATH('remoteDesktop'), TYPE) ,(SELECT localResources AS "@inherit" FOR XML PATH('localResources'), TYPE) ,(SELECT securitySettings AS "@inherit" FOR XML PATH('securitySettings'), TYPE) ,(SELECT displaySettings AS "@inherit" FOR XML PATH('displaySettings'), TYPE) FROM [file] FOR XML PATH('properties'),TYPE ) ,dbo.RDCM_exstract_group_and_members(1) FROM [file] FOR XML AUTO, ELEMENTS, ROOT('RDCMan') ) SET @XML.modify(' insert 2.2 as first into (/RDCMan)[1]') SET @xml.modify('insert attribute schemaVersion{"1"} as last into (RDCMan)[1]') SELECT @XML GO

**[Chad Miller](#168 "2012-11-16 17:33:22"):** @Steve -- Thanks for sharing your updated script. It took me a while to just figure out the basics without nested groups, so I know it must taken you a while to take it farther. On the subject of CMS organization my preference is to use a rather flat structure for a CMS without nested groups. My CMS server has basically dev, qa prod, and few other groups with servers directly under, but I understand that's not everyone's preference. Thanks again for sharing your more complete solution.

