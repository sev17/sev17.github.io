title: Appending New Rows Only
link: http://sev17.com/2012/04/27/appending-new-rows-only/
author: Chad Miller
description: 
post_id: 10914
created: 2012/04/27 12:06:48
created_gmt: 2012/04/27 16:06:48
comment_status: open
post_name: appending-new-rows-only
status: publish
post_type: post

# Appending New Rows Only

I saw [a question in the forums](http://stackoverflow.com/questions/10323767/proper-usage-of-data-tables) related to inserting new rows into a SQL Server table only if they didn’t exist. The current solution was using an ADO.NET DataTable , checking for new rows and then pushing the rows back to SQL Server by calling the Update method on the DataAdapter. Although the solution works, the process becomes longer as each time the process is run the entire table is retrieved and compared. There’s a number of approaches you could take to solve this problem. One solution is to use [Table Valued Parameters which I’ve previously blogged](/2012/04/table-valued-parameter-example/) about to push a batch of rows to SQL Server and add only new rows. This does require creating both a table type and stored procedure on the SQL Server and only works for SQL Server 2008 and higher: 
    
    
    <#
    /* FROM SSMS */
    USE tempdb
    GO
    /* Create a Request table for testing purposes*/
    CREATE TABLE Request
    ( PKID INT,
     MessageText varchar(max));
    GO
    /* Create a RequestList table type */
    CREATE TYPE RequestList AS TABLE
    ( PKID INT,
     MessageText varchar(max));
    GO
    
    /* Create a procedure to use insert only new rows  */
    CREATE PROCEDURE uspSetRequest
        @TVP RequestList READONLY
        AS
        SET NOCOUNT ON
        INSERT Request
        SELECT tvp.PKID, tvp.MessageText
        FROM @TVP tvp
        LEFT JOIN Request r ON
        tvp.PKID = r.PKID
        WHERE r.PKID IS NULL;
     GO
     #>
    
    #FROM Powershell
    #Create an ADO.NET DataTable matching the RequestList Table Type:
    $dt = new-object Data.datatable
    $col =  new-object Data.DataColumn
    $col.ColumnName = 'PKID'
    $col.DataType = [Int32]
    $dt.Columns.Add($Col)
    $col =  new-object Data.DataColumn
    $col.ColumnName = 'MessageText'
    $col.DataType = [String]
    $dt.Columns.Add($Col)
    
    #BEGIN INSERT foreach Loops to add records to DataTable
    #Example below inserts only one record
    #Add a Row to the DataTable
    $dr = $dt.NewRow()
    $dr.Item('PKID') = 1
    $dr.Item('MessageText') = 'It worked!'
    $dt.Rows.Add($dr)
    #END INSERT foreach Loops to add records to DataTable 
    
    #Connection and Query Info
    $serverName="$env:computernamesql1"
    $databaseName='tempdb'
    $query='uspSetRequest' 
    
    #Connect
    $connString = "Server=$serverName;Database=$databaseName;Integrated Security=SSPI;"
    $conn = new-object System.Data.SqlClient.SqlConnection $connString
    $conn.Open()
    
    #Create Sqlcommand type and params
    $cmd = new-object System.Data.SqlClient.SqlCommand
    $cmd.Connection = $conn
    $cmd.CommandType = [System.Data.CommandType]"StoredProcedure"
    $cmd.CommandText= $query
    $null = $cmd.Parameters.Add("@TVP", [System.Data.SqlDbType]::Structured)
    $cmd.Parameters["@TVP"].Value = $dt
    
    #Execute Query and close connection
    $cmd.ExecuteNonQuery() | out-null
    $conn.Close()

## Comments

**[Chris Randall](#290 "2012-04-29 11:15:09"):** Why not use the T-SQL MERGE command?

**[Chad Miller](#291 "2012-04-29 12:11:22"):** You could use a Merge statement, however it's overkill for this specific scenario. The requirement is to only insert new rows and ignore updates. If we re-write the procedure using merge we end up with this: ` ALTER PROCEDURE uspSetRequest @TVP RequestList READONLY AS SET NOCOUNT ON MERGE Request AS target USING (SELECT tvp.PKID, tvp.MessageText FROM @TVP tvp) AS source ON (target.PKID = source.PKID) WHEN NOT MATCHED THEN INSERT (PKID, MessageText) VALUES (source.PKID, source.MessageText) ` Using MERGE is more lines of code and to me looks a little awkward when compared to the classic IS NULL check. Now, if the requirement was to to do a so called "Upsert" then, yes I would use a MERGE statement.

