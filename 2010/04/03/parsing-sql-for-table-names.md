title: Parsing SQL for Table Names
link: http://sev17.com/2010/04/03/parsing-sql-for-table-names/
author: Chad Miller
description: 
post_id: 10150
created: 2010/04/03 20:34:46
created_gmt: 2010/04/04 00:34:46
comment_status: open
post_name: parsing-sql-for-table-names
status: publish
post_type: post

# Parsing SQL for Table Names

Visual Studio Team System 2008 Database Edition (VSDB) ships with a .NET class for parsing T-SQL. I’ve previously blogged about producing a [Stored Procedure Call Tree](/2009/11/stored-procedure-call-tree/) and even built the [Test-SqlScript and Out-SqlScript cmdlets](http://sev17.com/2009/03/test-sqlscript-and-out-sqlscript-cmdlets/) included in [SQL Server PowerShell Extensions](http://sqlpsx.codeplex.com/) using the assemblies. Recently I’ve discovered another useful SQL parser called [MacroScope](http://macroscope.sourceforge.net/) that’s worthy of mention . A [short article on CodeProject](http://www.codeproject.com/KB/database/macroscope.aspx) describes MacroScope as an [Antlr](http://www.antlr.org/) based SQL parsing/transformation utility with support for Oracle, SQL Server, MySQL and even MS Access. Currently only CRUD (Select, Insert, Update, Delete) SQL parsing is implemented. Although MacroScope isn’t as full-featured as the VSDB classes, which supports the full range of T-SQL syntax, the fact that other DBMS types are covered is a plus. In addition MacroScope is licensed under GPL. To use MacroScope from PowerShell you’ll first need to [download the source code](http://macroscope.sourceforge.net/) and build the project. I’ve already done this, so alternately you can grab the assemblies from [here](http://cid-ea42395138308430.skydrive.live.com/embedicon.aspx/Public/Blog/macroscopeParser.zip)… As a test I’ve created a [PowerShell script for finding all table names and aliases within a SQL string](http://poshcode.org/1733): 
    
    
    param ($commandText)            
    
    #Assumes MacroScope and Antlr3 assemblies are in same directory
    add-type -Path $(Resolve-Path .MacroScope.dll | Select-Object -ExpandProperty Path)
    add-type -Path $(Resolve-Path .Antlr3.Runtime.dll | Select-Object -ExpandProperty Path)            
    
    #######################
    function Get-Table
    {
        param($table)            
    
        $table            
    
        if ($table.HasNext)
        { Get-Table $table.Next }            
    
    }            
    
    $sqlparser =[MacroScope.Factory]::CreateParser($commandText)
    $expression = $sqlparser.queryExpression()
    Get-Table $expression.From.Item | Select @{n='Name';e={$_.Source.Identifier}}, @{n='Alias';e={$_.Alias}}

Calling the PowerShell scripting with a simple SQL string which has multiple tables and aliases produces the following output: ![MacroScope](http://images.sev17.com/MacroScope_thumb.jpg) _Note: I haven’t done more complex testing—your mileage may vary._

## Comments

**[Rick](#128 "2012-04-26 16:10:35"):** Very nice! I was looking for something like this, and I think this will prove to be very helpful to some work I need to do. Before I dive in head first, is there an example of using MacroScope to provide the column names as well as the table names? Thanks!

**[Chad Miller](#129 "2012-04-26 22:39:31"):** I haven't tried with columns and last the bit of SQL parsing I've done I ended up using the VSDB assemblies since it understood more complex T-SQL.

**[Rick](#130 "2012-04-27 09:19:34"):** Chad, thanks very much for the quick reply! Do you know if there is some documentation for these interfaces? I've been looking through the C# source code, and I have to admit it's been rough sledding trying to figure out what the actual callable interfaces are. In fact, I'm not sure I could have figured out how to even get the table names if not for your example! Thanks! ~ Rick

**[Rick](#131 "2012-04-27 10:33:36"):** Or alternatively, since I have Sql Server 2008, SQL Server Management Studio, and PowerShell 2.0 already installed, maybe I could use the SQL Server PowerShell Extensions instead? My aim is to be able to parse SQL scripts offline and produce output showing all of the tables and columns used in the source SQL. (The SQL itself is designed to run on Oracle rather than SQL Server, but I'm hoping that would not make a difference for this application.) Would the PowerShell Extensions enable me to do this, and if so, is there perhaps some examples somewhere that would help me get started? Thanks! ~ Rick

**[Chad Miller](#132 "2012-04-27 10:59:24"):** I completely agree the VSDB and MacroScope classes are not very well documented. I've had use Reflector and play with using it Powershell. The VSDB classes are SQL Server specific and require a valid T-SQL statement or else parsing will fail. The MacroScope classes are database agnostic, but may not work on anything too complex.

**[Chad Miller](#133 "2012-04-27 11:14:54"):** There's nothing really in SQLPSX that will help for what you're trying to do. I only implemented parsing and formatting cmdlets called Test-SqlScript and Out-SqlScript which is part of the sqlparser module in SQLPSX. Test-SqlScript returns true/false on whether the script is valid and out-sqlscript applies formatting rules to the the SQL script after it parses correctly. For online parsing of existing SQL Server databases the sys.sql_expression_dependencies view hold a lot of promise for SQL Server 2008 and higher. You then use plain T-SQL to determine object dependency mapping by querying the view. Of course this is only an option if you can get the scripts to port and run in SQL 2008 and higher. You could just use some regular expressions to pull out table and view names from script files. Columns names would be difficult and this kind of Regex is difficult to get right. As far as commercial tools I've used Red Gate SQL Server Dependency Tracker. Its a SQL Server only tool. It does require an online database.

**[Rick](#134 "2012-04-27 17:20:27"):** Chad, thanks once again for the quick and informative replies! I've spent some more time with the Macroscope code, and I *thought* I had figured out where the columns would be stored: in the SelectItems property of the parser. I took your example and added a couple of things to test it out, but unfortunately I am getting nothing back. Here is what I did: * Used the $commandText and changed it to "select c1, c2 from authors a join titleauthors t on a.au_id = t.au_id" * created a function based on your Get-Table function: function Get-Columns { param($columns) $columns if ($columns.HasNext) { Get-Columns $columns.Next } } * Added this line of code to the script, again based on your Tables example: Get-Columns $expression.SelectItems.Item | Select-Object @{n='Name';e={$_.Source.Identifier}}, @{n='Alias';e={$_.Alias}} But, invoking this line does not return anything at all. Maybe I'm not quite understanding how the parser works, but I would have thought that the SelectItems property would contain the 'c1' and 'c2' values from the Select statement. Or, maybe I'm not accessing the property correctly? I imagine you don't have a lot of time to pursue this, but if I could get this part working, it would really help with the project I'm working on. Thanks! ~ Rick

**[Chad Miller](#135 "2012-04-27 19:43:39"):** This is a tricky problem as the object returned from parsers are jagged objects with varying levels of depth depending on the SQL statement parsed. Here's an example of parsing a simple statement for the items in the select list. The MacroScope parser has a HasNext property which indicates if there are additional items of a given type. For example notice "$expression | select -ExpandProperty SelectItems" has an Item and HasNext = True. To what you want to do you'll need to you use recursion looking for Item and HasNext = True. ` $commandText = "select au_lname, au_fname from dbo.authors" #Assumes MacroScope and Antlr3 assemblies are in same directory add-type -Path $(Resolve-Path .MacroScope.dll | Select-Object -ExpandProperty Path) add-type -Path $(Resolve-Path .Antlr3.Runtime.dll | Select-Object -ExpandProperty Path) $sqlparser =[MacroScope.Factory]::CreateParser($commandText) $expression = $sqlparser.queryExpression() $expression | select -ExpandProperty SelectItems | select -ExpandProperty Item | select -ExpandProperty Right Returns Identifier \--------- au_fname $expression | select -ExpandProperty SelectItems | select -ExpandProperty Next | select -ExpandProperty Item | select -ExpandProperty Right Returns Identifier \--------- au_lname `

**[Rick](#136 "2012-04-30 09:53:05"):** Chad, thank you once again for taking the time to reply to this, and for creating the example! This is invaluable, and sets me in the right direction to continue working on what I need to get done for my project. Thank you! ~ Rick

**[Rick](#137 "2012-04-30 17:27:28"):** Just a note, using your example as a base, I was able to implement a simple recursive function to parse out the column names: function Get-Columns { param($AliasedItem) $AliasedItem.Item.Right.Identifier if ($AliasedItem.HasNext) { Get-Columns $AliasedItem.Next } } $commandText = "select au_lname l, au_fname f, au_mname m from dbo.authors" #Assumes MacroScope and Antlr3 assemblies are in same directory add-type -Path $(Resolve-Path .MacroScope.dll | Select-Object -ExpandProperty Path) add-type -Path $(Resolve-Path .Antlr3.Runtime.dll | Select-Object -ExpandProperty Path) $sqlparser =[MacroScope.Factory]::CreateParser($commandText) $expression = $sqlparser.queryExpression() Get-Columns ($expression | select -ExpandProperty SelectItems) So, I have seen how to get the table name, and how to get the column names. Now, if I could only figure out how to relate a given column to the table it is part of, whene there are more than one table... ~ Rick

**[Chad Miller](#138 "2012-05-01 10:29:20"):** That's going to be difficult unless the column is prefixed with table or table alias. One thought -- Since you're working against Oracle, you could grab the columns from the live database. Here's a query I've used which has more information than you need: [code] SELECT DISTINCT 'oracle' as dbms_type_name ,(SELECT instance_name FROM v$instance) as dbms_name ,(SELECT instance_name FROM v$instance) as database_name ,a.owner as table_schema ,a.table_name as table_name ,a.column_name as column_name ,a.column_id as ordinal_position ,NULL as column_default ,NULL as is_identity ,NULL as is_nullable ,a.data_type as data_type ,a.data_length as character_maximum_length ,a.data_length as character_octet_length ,a.data_precision as numeric_precision ,a.data_precision as numeric_precision_radix ,a.data_scale as numeric_scale ,'0' as datetime_precision ,NULL as character_set_catalog ,NULL as character_set_schema ,NULL as character_set_name ,NULL as collation_catalog ,NULL as collation_schema ,NULL as collation_name ,NULL as domain_catalog ,NULL as domain_schema ,NULL as domain_name ,sysdate as load_dtm FROM dba_tab_columns a WHERE exists (SELECT 'x' FROM dba_tables b WHERE a.owner = b.owner AND a.table_name = b.table_name AND b.table_name not like 'BIN$%' AND b.owner not in ('SYS','SYSTEM','PUBLIC','CTXSYS','DIP','DMSYS','EXFSYS','MDDATA', 'MDSYS','MGMT_VIEW','OLAPSYS','ORACLE_OCM','ORDPLUGINS','ORDSYS','OUTLN','SCOTT','SYSMAN','TSMSYS' ,'WMSYS','XDB','TOAD','SOE','DBSNMP')) [/code] You'd then need to relate this to the tables in your query.

**[Rick](#139 "2012-06-04 17:27:46"):** Chad, sorry to keep bothering you, but one more thing that perhaps you can help me with. I have the parser working pretty well with even fairly complex sql, but for some reason, the parser generates an exception when it encounters a comment in the sql, as defined by either a leading -- or the /* */ pair. I looked in the grammar file (Macroscope.g) and I don't see anything obvious that seems to pertain to comments, whihc may be why they are not accounted for. Do you know if comments are not implemented in this grammar? I did come across another grammar file that seems more specific to Oracle (which is what I am using) at http://www.antlr.org/grammar/1209225566284/PLSQL3.g, but I am not sure how I would re-build things to use it. I know this is a bit far-afield from the things you are focused on, but any help or pointers would once again be most appreciated! ~ Rick

**[Chad Miller](#140 "2012-06-04 18:30:24"):** Compiling a new Antlr grammar file is bit beyond my abilities. I would suggest stripping out the comments through a regular expression. This will be a little tricky. I'm not an expert Regex guy and it usually takes me an embarrassing number of hours to get complex regex right. One thing I'd suggest is purchasing the book and downloading the Perl source code for "Real World SQL Server Administration with Perl". You can find the Perl source code on the Apress site. The author has an a Perl module which normalizes SQL. Take a look at SQLDBA.pm, there's some fairly involved Regex for identifying ANSI and SQL comments. The Perl regex will need to be translated to .NET equivalent for Powershell.

**[Rick](#141 "2012-06-05 09:51:30"):** Thanks as always, Chad, I appreciate the help! ~ Rick

