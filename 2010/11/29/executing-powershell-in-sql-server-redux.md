title: Executing PowerShell in SQL Server Redux
link: http://sev17.com/2010/11/29/executing-powershell-in-sql-server-redux/
author: Chad Miller
description: 
post_id: 10509
created: 2010/11/29 20:01:24
created_gmt: 2010/11/30 01:01:24
comment_status: open
post_name: executing-powershell-in-sql-server-redux
status: publish
post_type: post

# Executing PowerShell in SQL Server Redux

A while ago I blogged about [using xp_cmdshell to execute a PowerShell script in SQL Server](/2009/04/executing-powershell-in-sql-server/) and return a result set. At that time I was using PowerShell V1, but now with PowerShell V2 I can clean this up a little. The improved version uses the built-in PowerShell V2 cmdlet **ConvertTo-XML** with the _–AsString_ parameter. Because SQL Server understands XML we can parse the XML using XQuery. Keep in mind this is still hacky and just as I mentioned last time its far better to execute T-SQL in PowerShell rather than use or more accurately misuse xp_cmdshell. Anyways here’s a improved version:
    
    
    -- To allow advanced options to be changed.
    EXEC sp_configure 'show advanced options', 1
    GO
    -- To update the currently configured value for advanced options.
    RECONFIGURE
    GO
    -- To enable the feature.
    EXEC sp_configure 'xp_cmdshell', 1
    GO
    -- To update the currently configured value for this feature.
    RECONFIGURE
    GO
    
    /*
    #COPY disk.ps1 to Public folder
    param ( [string]$ComputerName = "." )
    
    Get-WmiObject -computername "$ComputerName" Win32_LogicalDisk -filter "DriveType=3" | 
    foreach { add-member -in $_ -membertype noteproperty UsageDT $((Get-Date).ToString("yyyy-MM-dd"))
    add-member -in $_ -membertype noteproperty SizeGB $([math]::round(($_.Size/1GB),2))
    add-member -in $_ -membertype noteproperty FreeGB $([math]::round(($_.FreeSpace/1GB),2))
    add-member -in $_ -membertype noteproperty PercentFree $([math]::round((([float]$_.FreeSpace/[float]$_.Size) * 100),2)) -passThru } |
    Select UsageDT, SystemName, DeviceID, VolumeName, SizeGB, FreeGB, PercentFree
    */
    
    CREATE TABLE #output
    (line varchar(255))
    INSERT #output
    EXEC xp_cmdshell 'powershell -Command "C:UsersPublicdisk.ps1 | ConvertTo-Xml -NoTypeInformation -As string"'
    
    DELETE #output WHERE line IS NULL
    
    DECLARE @doc varchar(max)
    SET @doc = ''
    DECLARE @line varchar(255)
    DECLARE xml_cursor CURSOR
    FOR SELECT line FROM #output
    OPEN xml_cursor
    FETCH NEXT FROM xml_cursor INTO @line
    WHILE @@FETCH_STATUS = 0
    BEGIN
    SET @doc = @doc + @line
    FETCH NEXT FROM xml_cursor INTO @line
    END
    CLOSE xml_cursor
    DEALLOCATE xml_cursor
    DROP TABLE #output
    
    SELECT
    item.ref.value('(Property/text())[1]', 'datetime') AS UsageDT
    ,item.ref.value('(Property/text())[2]', 'nvarchar(128)') AS SystemName
    ,item.ref.value('(Property/text())[3]', 'nvarchar(128)') AS DeviceID
    ,item.ref.value('(Property/text())[4]', 'nvarchar(128)') AS VolumeName
    ,item.ref.value('(Property/text())[5]', 'nvarchar(128)') AS SizeGB
    ,item.ref.value('(Property/text())[6]', 'nvarchar(128)') AS FreeGB
    ,item.ref.value('(Property/text())[7]', 'nvarchar(128)') AS PercentFree
    FROM (SELECT CAST(@doc AS XML) AS feedXml) feeds(feedXml)
    CROSS APPLY feedXml.nodes('/Objects/Object') AS item(ref)