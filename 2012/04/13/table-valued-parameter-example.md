title: Table-Valued Parameter Example
link: http://sev17.com/2012/04/13/table-valued-parameter-example/
author: Chad Miller
description: 
post_id: 10896
created: 2012/04/13 15:34:53
created_gmt: 2012/04/13 19:34:53
comment_status: open
post_name: table-valued-parameter-example
status: publish
post_type: post

# Table-Valued Parameter Example

I wanted show someone how to use table-valued parameters available in SQL Server 2008 and higher. The main use case of table-valued parameters is for sending a list or array of items as parameter to a SQL Server stored procedure or function. This is more efficient than parsing strings or XML on the SQL Server side. I couldn't seem to find a complete example of [table-valued parameters](http://msdn.microsoft.com/en-us/library/bb510489.aspx) in the SQL Server documentation. The SQL Server docs only shows the T-SQL portion of the code and not the ADO.NET. I think its difficult to see how you would use this feature without having both the T-SQL and .NET code shown together so, here's a simple T-SQL and Powershell script demonstrating table-valued parameters: 
    
    
    <#
    /* FROM SSMS */
    USE AdventureWorks
    GO
    /* Create a CustomerList table type */
    CREATE TYPE Sales.CustomerList AS TABLE 
    ( CustomerID INT );
    GO
    
    /* Create a procedure to use new table type */
    CREATE PROCEDURE Sales.uspGetCustomer
        @TVP CustomerList READONLY
        AS 
        SET NOCOUNT ON
        SELECT c.*
        FROM Sales.Customer c
        JOIN @TVP t ON
        c.CustomerID = t.CustomerID;
     GO
    
    /* Test type and procedure in SSMS */
    
    /* Declare a variable that references the type. */
    DECLARE @CustomerTVP AS Sales.CustomerList;
    
    /* Add data to the table variable. */
    INSERT INTO @CustomerTVP (CustomerID)
    SELECT * FROM (
    	VALUES (1),(2),(3),(4),(5)
    ) AS v (CustomerID)
    
    /* Pass the table variable data to a stored procedure. */
    EXEC Sales.uspGetCustomer @CustomerTVP;
    GO
    #>
    
    #FROM Powershell
    #Create an ADO.NET DataTable matching the CustomerList Table Type:
    $dt = new-object Data.datatable  
    $col =  new-object Data.DataColumn  
    $col.ColumnName = 'CustomerID'  
    $col.DataType = [Int32]
    $dt.Columns.Add($Col)
    
    #Add a Row to the DataTable
    $dr = $dt.NewRow()
    $dr.Item('CustomerId') = 1   
    $dt.Rows.Add($dr)  
    
    #Add a 2nd Row to the DataTable
    $dr = $dt.NewRow()
    $dr.Item('CustomerId') = 2   
    $dt.Rows.Add($dr)  
    
    #Add a 3rd Row to the DataTable
    $dr = $dt.NewRow()
    $dr.Item('CustomerId') = 3   
    $dt.Rows.Add($dr)  
    
    #Connection and Query Info
    $serverName="$env:computernamesql1" 
    $databaseName='AdventureWorks' 
    $query='Sales.uspGetCustomer' 
    
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
    
    #Create and fill dataset
    $ds=New-Object system.Data.DataSet
    $da=New-Object system.Data.SqlClient.SqlDataAdapter($cmd)
    $null = $da.fill($ds)
    $conn.Close()
    
    #Return results
    $ds.Tables