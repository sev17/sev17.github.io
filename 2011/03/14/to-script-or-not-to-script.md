title: To Script or Not To Script
link: http://sev17.com/2011/03/14/to-script-or-not-to-script/
author: Chad Miller
description: 
post_id: 10611
created: 2011/03/14 23:01:19
created_gmt: 2011/03/15 03:01:19
comment_status: open
post_name: to-script-or-not-to-script
status: publish
post_type: post

# To Script or Not To Script

When approaching an automation problem the first questions you must answer is whether you should even invest the time to write a script. There are scenarios where it just isn’t worth your time. Depending on the tools available to you even the task of scripting out every database object could be accomplished easier and faster by not scripting. 

## Buy vs. Build vs. Borrow

When I look into my collection of SQL Server tools, I use combination of Microsoft shipped, free software, various scripts I’ve built or borrowed and products I've bought from various software vendors. One purchased product I use is from [Red Gate](http://www.red-gate.com/) called [SQL Compare](http://www.red-gate.com/products/sql-development/sql-compare/). Using SQL Compare I can quickly create database build scripts faster and better than I could if I created my own collection scripts. Given enough time I could have written my own tool, but why should I invest the time when there's a commercial tool that does it better? Scripting is about solving problems not adequately addressed by tools you own or can buy. Using SQL Compare for solely scripting out databases isn't worth the purchase price. The main reason I use Red Gate is for the comparison and build scripts it generates. The ability to script out database objects is just an added bonus. SQL Compare doesn’t include native PowerShell support, however the Pro edition includes a command-line utility called [sqlcompare](http://www.red-gate.com/supportcenter/Content?p=SQL%20Compare&c=SQL_Compare/help/9.0/SC_CL_Getting_Started.htm&toc=SQL_Compare/help/9.0/toc1413745.htm) which can be used within PowerShell. The following example scripts out an entire database including automatically creating the folder structures as shown below. No scripting required! 
    
    
    PS C:Program FilesRed GateSQL Compare 8>  ./sqlcompare.exe /s1 "Z003sql1" /db1 "pubs" /mkscr:"C:pubs"
    SQL Compare Command Line V8.1.0.360
    ==============================================================================
    Copyright c Red Gate Software Ltd 1999-2009
    
    Serial Number: XXXXXXXXXXXXXXXXXXX
    
    Creating folder of scripts 'C:pubs' from database 'pubs' on 'Z003sql1'...
    OK
    PS C:Program Files (x86)Red GateSQL Compare 8>

![sqlcompare](http://images.sev17.com/sqlcompare_thumb.png)