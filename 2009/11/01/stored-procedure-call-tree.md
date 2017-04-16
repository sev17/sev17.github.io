title: Stored Procedure Call Tree
link: http://sev17.com/2009/11/01/stored-procedure-call-tree/
author: Chad Miller
description: 
post_id: 9987
created: 2009/11/01 16:13:00
created_gmt: 2009/11/01 20:13:00
comment_status: open
post_name: stored-procedure-call-tree
status: publish
post_type: post

# Stored Procedure Call Tree

I was reading a post by [Linchi Shea](http://sqlblog.com/blogs/linchi_shea/default.aspx) in which he demonstrates a Perl script to [Find the complete call tree for a stored procedure](http://sqlblog.com/blogs/linchi_shea/archive/2009/10/23/find-the-complete-call-tree-for-a-stored-procedure.aspx) and thought, how would I do this in PowerShell? Before we dissect a PowerShell approach, let’s look at a basic Perl approach. In Perl you typically find a command-line tool that produces the output you want albeit not in a usable format (Sure, sometimes you might get lucky and find a Perl module that does what you want, however when I used Perl I was never that lucky). This may involve several command lines tools. You’ll then parse the output of the tool and perhaps even use the output as input to other command-line tools. This is one of the strengths of a scripting language, you can quickly glue together tools to make a new tool--The scripter is a tool smith. The Perl solution to getting a stored procedure call tree involves executing osql.exe and using some regular expressions to parse the output of sp_helptext, looking for EXECUTE statements. When I wrote Perl I would do much the same thing. As an example when I needed to [report the share and NTFS permissions](http://www.sqlservercentral.com/scripts/Miscellaneous/31745/) using Perl for all SQL Servers, I would execute rmtshare.exe parse the output which was then used as input to fileacl.exe and the output was then parsed. We could do much the same thing in PowerShell, execute osql.exe and parse the output, however in the vast majority of cases if you’re parsing something in PowerShell you’re doing it the hard way. This is because PowerShell works with .NET, WMI and COM and you’ll typically find a .NET/WMI class or COM interface to do exactly what you want to do. Of course the hard part is knowing which class to use, if you’re not sure search and then ask. If you’re unable to find the answer yourself, both [ServerFault](http://serverfault.com/) and [StackOverflow](http://stackoverflow.com/) are good places to post these types of questions. There is a nice set of .NET classes for parsing T-SQL code included with Visual Studio Team System 2008 Database Edition (VSDB) in the classes [Microsoft.Data.Schema.ScriptDom](http://msdn.microsoft.com/en-us/library/microsoft.data.schema.scriptdom.aspx) and [Microsoft.Data.Schema.ScriptDom.Sql](http://msdn.microsoft.com/en-us/library/microsoft.data.schema.scriptdom.sql.aspx). I’ve [blogged about these classes before](/2009/03/test-sqlscript-and-out-sqlscript-cmdlets/) and even use them in [SQL Server PowerShell Extensions](http://sqlpsx.codeplex.com/) (SQLPSX). At this point, you’re probably thinking I don’t have VSDB, well that’s OK, because [the assemblies are redistributable](http://blogs.msdn.com/gertd/archive/2008/08/22/redist.aspx) I’ve included them with SQLPSX and the accompanying download to this post. There’s only one problem with the VSDB assemblies, they are implemented as interfaces. Let me explain the issue. When writing PowerShell scripts you will often work with .NET classes. You’ll create an object from a .NET class and start working directly with it’s properties and methods. [SQL Server Management Objects or SMO](http://msdn.microsoft.com/en-us/library/cc285859.aspx) is great example of this. In one line of PowerShell code you have instant access to hundreds of properties and methods for a SQL Server object: 
    
    
    $srv = new-object ("Microsoft.SqlServer.Management.Smo.Server") Z002SQL2K8

As you can see with the example above its very simple to start working with a .NET class. There is however, one notable exception when dealing with interfaces.  This is kind of fringe use case, in most instances you won’t have to deal with interfaces in PowerShell. The best way I can describe an interface is a fancy abstraction thing developers use when creating classes. Only interfaces aren’t real classes and in order to use an interface you have to implement a class. The important thing to note, as a scripter an interface just means we need to create a class to use it and this requires a .NET language other than PowerShell. Fortunately PowerShell V2 through the **add-type** cmdlet provides a way for us to do this within PowerShell. Using **add-type** we can create a dynamic type in  .NET languages like C#. This means you can write C# within PowerShell. The first time I saw this I thought,”And why would I want to do that?” And then I remembered the problem with interfaces and in a way this functionality goes back to the main purpose of scripting, to glue together tools to create a new tool. Only in this case the tool is a snippet of C# code! So, I created a script called [SQLParser](http://poshcode.org/1445) that implements the Microsoft.Data.Schema.ScriptDom and Microsoft.Data.Schema.ScriptDom.Sql interfaces with a basic C# class. To use **SQLParser**, source the script file and create a **SQLParser** object by specifying the SQL version, whether quoted identifiers are used and some valid T-SQL: 
    
    
    . ./SQLParser.ps1
    
    
    $sqlparser = new-object SQLParser Sql100,$false,"Select * from dbo.authors"

The SQLParser class returns a fragment, which then is made up of batches and finally statements. We can iterate through the statements looking for a particular statement type. Having accomplished the hard part of finding and implementing a .NET class for T-SQL parsing, I then created a PowerShell script called [Get-ProcedureCallTree.ps1](http://poshcode.org/1446) to return a stored procedure call tree: If you run the script with the following parameters for HumanResources.uspUpdateEmplyeeHireInfo in the AdventureWorks database replacing Z002SQL2K8 with your server name, you’ll should see the following output: 
    
    
    .Get-ProcedureCallTree.ps1 uspUpdateEmployeeHireInfo "Z002SQL2K8" AdventureWorks HumanResources
    Server : Z002SQL2K8
    Database : AdventureWorks
    Schema : dbo
    Procedure : uspLogError
    Source : Z002SQL2K8.AdventureWorks.HumanResources.uspUpdateEmployeeHireInfo
    Target : Z002SQL2K8.AdventureWorks.dbo.uspLogError