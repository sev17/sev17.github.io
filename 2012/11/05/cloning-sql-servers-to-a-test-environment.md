title: Cloning SQL Servers to a Test Environment
link: http://sev17.com/2012/11/05/cloning-sql-servers-to-a-test-environment/
author: Chad Miller
description: 
post_id: 10963
created: 2012/11/05 09:10:42
created_gmt: 2012/11/05 14:10:42
comment_status: open
post_name: cloning-sql-servers-to-a-test-environment
status: publish
post_type: post

# Cloning SQL Servers to a Test Environment

I've been involved in a project to set up a full test environment of dozens of SQL Servers. The requirements of the test environment where such that they needed to be cloned via V2V or P2V from source servers, the servers would then be renamed and located within a completely isolated network on a separate AD without a trust relationship. What follows are a few scripts I created to deal with some of the issues which arise given the requirements. These issues include: 

  * Dealing with orphaned AD SQL Server logins
  * Avoiding connection string changes
  * Avoiding Linked Servers changes

## Dealing with Orphaned AD SQL Server Logins

The test environment Active Directory logins and groups are copied from the source environment by AD administrators; however the SIDs will be different in the test environment which creates orphaned AD logins in SQL Server. I have to admit I haven't dealt with orphaned AD logins in SQL Server very much. I have dealt with orphaned SQL Server database users where the SQL Server login SID does not match between servers which often occur when restoring databases between servers. In researching this issue I was pleasantly surprised to see the same ALTER USER  ... WITH LOGIN syntax or the old style sp_change_users_login system stored procedure I had used to deal with orphaned SQL Server database users works with AD logins. Here's a T-SQL script to change AD logins from one domain to another and reassociate any database users. The script assumes the AD logins have been created in the test AD with the same name as the source domain: 
    
    
    SET NOCOUNT ON
    
    --#BEGIN LOGIN CREATION
    PRINT '--1. BEGIN LOGIN CREATION ' + @@SERVERNAME;
    
    DECLARE @name sysname, @default sysname
    
    DECLARE loginCursor CURSOR FOR 
    SELECT REPLACE(s1.name,'OldDomain','NewDomain'), s1.default_database_name 
    FROM sys.server_principals s1
     WHERE s1.type IN ('G','U') AND s1.name LIKE 'OldDomain%'
     AND s1.name NOT IN (SELECT s1.name FROM sys.server_principals s2 WHERE REPLACE(s1.name,'OldDomain','') = REPLACE(s2.name,'NewDomain',''))
    
    OPEN loginCursor
    FETCH NEXT FROM loginCursor INTO @name,@default
    
    WHILE @@FETCH_STATUS = 0
    BEGIN
    	PRINT 'CREATE LOGIN [' + @name + '] FROM WINDOWS WITH DEFAULT_DATABASE=[' + @default + '];'
    	EXEC ('CREATE LOGIN [' + @name + '] FROM WINDOWS WITH DEFAULT_DATABASE=[' + @default + '];')
    	FETCH NEXT FROM loginCursor INTO @name,@default
    END
    
    CLOSE loginCursor
    DEALLOCATE loginCursor;
    
    PRINT '--1. END LOGIN CREATION ' + @@SERVERNAME;
    --#END LOGIN CREATION
    
    --#BEGIN SERVER ROLE FIX
    PRINT '--2. BEGIN SERVER ROLE FIX ' + @@SERVERNAME;
    
    DECLARE @role sysname, @member sysname
    
    DECLARE roleCursor CURSOR FOR 
    SELECT SUSER_NAME(role_principal_id), REPLACE(SUSER_NAME(member_principal_id),'OldDomain','NewDomain')
     FROM sys.server_role_members
     WHERE SUSER_NAME(member_principal_id) LIKE 'OldDomain%'
    
    OPEN roleCursor
    FETCH NEXT FROM roleCursor INTO @role,@member
    
    WHILE @@FETCH_STATUS = 0
    BEGIN
    	PRINT 'EXEC sp_addsrvrolemember ''' + @member + ''', ''' + @role + ''';'
    	EXEC sp_addsrvrolemember @member, @role;
    	FETCH NEXT FROM roleCursor INTO @role,@member
    END
    
    CLOSE roleCursor
    DEALLOCATE roleCursor;
    
    PRINT '--2. END SERVER ROLE FIX ' + @@SERVERNAME;
    --#END SERVER ROLE FIX
    
    --#BEGIN USER FIX
    
    EXEC sp_MSforeachdb 
      @command1='PRINT ''--3. BEGIN USER FIX ?'''
    , @command2='USE [?] IF DATABASEPROPERTY(''?'',''IsReadOnly'') = 0
    BEGIN
    	DECLARE @uname sysname, @sname sysname
    
    	DECLARE userCursor CURSOR FOR
    	SELECT name, REPLACE(SUSER_SNAME(sid),''OldDomain'',''NewDomain'')
    	FROM sys.database_principals
    	WHERE type IN (''G'',''U'') AND SUSER_SNAME(sid) LIKE ''OldDomain%''
    
    	OPEN userCursor
    	FETCH NEXT FROM userCursor INTO @uname,@sname
    
    	WHILE @@FETCH_STATUS = 0
    	BEGIN
    		IF @uname = ''dbo''
                    BEGIN
                        PRINT ''ALTER AUTHORIZATION ON DATABASE::[?] TO ['' + @sname + ''];''
    		    EXEC (''ALTER AUTHORIZATION ON DATABASE::[?] TO ['' + @sname + ''];'')
                    END
                    ELSE
                    BEGIN
    		    PRINT ''ALTER USER ['' + @uname + ''] WITH LOGIN=['' + @sname + ''];''
    		    EXEC (''ALTER USER ['' + @uname + ''] WITH LOGIN=['' + @sname + ''];'')
                    END
    
    		FETCH NEXT FROM userCursor INTO @uname,@sname
    	END
    
    	CLOSE userCursor
    	DEALLOCATE userCursor
    END'
    , @command3 = 'PRINT ''--3. END USER FIX ?'''
    --#END USER FIX

## Avoiding Connection String Changes

In order to avoid changing connection strings I aliased the old server name to the new server name in a couple of ways, first by creating entries on each server's C:windowsSystem32driversetchosts file and second using SQL Server Client Aliases. I've found that creating aliases using both methods provides the highest likelihood of success. By aliasing the new server name I've avoided changing SSIS connections, linked server and really anything that runs externally on the particular SQL Servers. There are two scripts I've used. First a Powershell script to generate hosts file entries given a text file of just server names. The grid output is then copied (ctrl-A, ctrl-C) and pasted into a hosts file. I'll make the hosts file identical for each server in the test environment. 
    
    
    get-content ./servers.txt | 
    foreach { new-object PSObject -property @{'Computername'=$_; 'IP' = (Test-Connection -ComputerName $_ -Count 1).IPV4Address.IPAddressToString }} |
     out-gridview

To create the SQL Server Client alias using a GUI you would use cliconfg from the run command and manually create each SQL Server alias, but in order to script this, I created a script to add the registry entries which is really all the GUI does. For a single SQL Server Client Alias, I've posted a script to PoshCode, called [Add-SqlClientAlias](http://poshcode.org/3662), but rather than use that script I created a bulk alias script which goes against a SQL Server Central Management Server also setup in the test environment. The script assumes the old server name is the same as the new server name with a TEST prefix or suffix: 
    
    
    $query = @"
    SET NOCOUNT ON;
    SELECT DISTINCT s.name
    FROM msdb.dbo.sysmanagement_shared_registered_servers s
    JOIN msdb.dbo.sysmanagement_shared_server_groups g
    ON s.server_group_id = g.server_group_id
    "@
    
    if (!(test-path 'HKLM:SOFTWAREMicrosoftMSSQLServerClientConnectTo')) {
        new-item -path 'HKLM:SOFTWAREMicrosoftMSSQLServerClient' -name ConnectTo
    }
    
    sqlcmd -S myCMServerInstance -d msdb -Q $query -h -1 -W |
    foreach { Set-ItemProperty -Path 'HKLM:SOFTWAREMicrosoftMSSQLServerClientConnectTo' -Name $($_ -replace 'TEST') -Value "DBMSSOCN,$_" }
    
    }

## Avoiding Linked Server Changes

If you change the AD logins to the new domain any explicitly mapped Linked Server logins will break, so you'll need to re-map them using the new domain accounts. This process is a little manual in that the T-SQL script below generates a linked server login creation script which you'll need to copy and paste into a new query window and fill in passwords for the mapped AD to SQL accounts: 
    
    
    SET NOCOUNT ON
    
    --#BEGIN LOGIN CREATION
    PRINT '--1. BEGIN LINKED SERVER LOGIN CREATION ' + @@SERVERNAME
    
    CREATE TABLE #linkedsrvlogin
    (
    linked_server sysname NULL 
    ,local_login sysname NULL
    ,is_self_mapping SMALLINT NULL 
    ,remote_login sysname NULL
    ) 
    INSERT #linkedsrvlogin
    EXEC sp_helplinkedsrvlogin
    
    SELECT 'EXEC master.dbo.sp_addlinkedsrvlogin @rmtsrvname=N''' + 
    linked_server + ''',@useself=N''False'',@locallogin=N''' + 
    REPLACE(local_login,'OldDomain','NewDomain') +
    ''',@rmtuser=N''' + remote_login + 
    ''',@rmtpassword=''########'''
    FROM #linkedsrvlogin s1
    WHERE local_login LIKE 'OldDomain%'
    AND is_self_mapping = 0
    AND remote_login IS NOT NULL
    AND local_login NOT IN (SELECT local_login FROM #linkedsrvlogin s2 WHERE REPLACE(s1.local_login,'OldDomain','') = REPLACE(s2.local_login,'NewDomain',''))
    
    PRINT '--1. END LINKED SERVER LOGIN CREATION ' + @@SERVERNAME;
    --#END LOGIN CREATION
    
    SELECT * FROM #linkedsrvlogin