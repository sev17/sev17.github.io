title: Reconciling SPNs
link: http://sev17.com/2012/02/20/reconciling-spns/
author: Chad Miller
description: 
post_id: 10837
created: 2012/02/20 09:39:32
created_gmt: 2012/02/20 14:39:32
comment_status: open
post_name: reconciling-spns
status: publish
post_type: post

# Reconciling SPNs

As part of troubleshooting Kerberos authentication for SQL Server I had to verify SPNs so I thought I'd blog about the process I went through. Sometimes I think a post blog is nothing more than public documentation of  a complex problem ... 

## Get SQL Server SPNs from Active Directory

Before Powershell I would use the command-line utility setspn.exe to retrieve a list of SPNs for a given account. The syntax is: setspn -L AccountName Although this works reasonably well there are a couple of issues first the  output is text instead of objects which means I'd have to do a lot parsing and if you find you're parsing text in Powershell too much there's a high likelihood you're doing it the hard way!. The second issue with setspn -L is that it expects an account and doesn't retrieve ALL SPNs for a given service. For these reasons I created a script called Get-SqlSpn which will query Active Directory for all SQL Server SPNs. Querying AD for SPNs still requires a bit of parsing (I've taken care of this for you in the Get-SqlSpn function), but not near as much as starting from setspn. Let's look at how I use Get-SqlSpn... 

### Using Get-SqlSpn

1\. Download [Get-SqlSpn](http://poshcode.org/3234) 2\. Source the function 
    
    
    . ./Get-Sqlspn.ps1

#Get the SQL Server SPNs 
    
    
     $spns = Get-SqlSpn

If I'm interactively exploring data I like to use a tool for that specific purpose which for me is either loading the data into SQL Server or using Excel. For this particular I think Excel is a better fit. There is some tidying up and normalizing data that needs to be done which is really easy with Excel. Although you could mess with additional scripts to export data directly into Excel or or convert CSV files--a quick and dirty way to get data into Excel is simply copy and paste from out-gridview: 3\. Open a new Excel document and rename a worksheet servers 4\. Get Column Headers 
    
    
    $object = $spns | select -first 1
    $ht = @{}
    foreach($property in $object.PsObject.get_properties()) {
      $ht.add($property.Name.ToString(),$property.Name.ToString())
    }

5\. Copy/Paste heading row to Excel (Ctrl-A, Ctrl-C) 
    
    
    new-object psobject -Property $ht | out-gridview

6\. Copy/Paste spns to Excel (Ctrl-A, Ctrl-C) 
    
    
    $spns | out-gridview

Now that you have a list of all SQL Server SPNs from AD, let's grab some information from all of our SQL Servers to match against. For this purpose I created a script called Get-SqlWmi.. 

### Using Get-SqlWmi

1\. Download [Get-SqlWmi](http://poshcode.org/3235) and [Invoke-Sqlcmd2](http://poshcode.org/2279) 2\. Source the functions 
    
    
    . ./invoke-sqlcmd2.ps1
    . ./get-sqlwmi.ps1

I use [a  query against System Center Configuration Manager database](/2008/11/inventory-sql-server-databases-with-powershell/) to pull in a list of SQL Servers. A second approach, if you maintain a SQL Server Center Management Server (CMS)  is to query sysmanagement tables. I'll then pipe the list of servers to the get-sqlwmi function: 
    
    
    $servers = invoke-sqlcmd2 -ServerInstance 'smsserversql10' -Database 'dbautility' -query "select server_name AS 'ComputerName' from sms_sql_server_name_vw" | get-sqlwmi

3\. Rename Excel worksheet Sheet2 to servers 4\. Get Column Headers 
    
    
    $object = $servers | select -first 1
    $ht = @{}
    foreach($property in $object.PsObject.get_properties()) {
       $ht.add($property.Name.ToString(),$property.Name.ToString())
    }

5\. Copy/Paste heading row into Excel 
    
    
    new-object psobject -Property $ht | out-gridview

6\. Copy/Paste servers to Excel 
    
    
    $servers | out-gridview

Now that I collected the data, I start analyzing... 

## Analyzing SPNs

_Note: The steps which follow  involve using some basic data analysis functions in Excel. If you're not sure how to filter, define named ranges or use Excel functions you may brush up on Excel. I've also included an Excel document with the appropriate fields and formulas for [download here](https://skydrive.live.com/redir.aspx?cid=ea42395138308430&resid=EA42395138308430!1012&parid=EA42395138308430!194&authkey=!AJ59P92kso7I9hY)._ First I'll look for invalid SPNs. I've seen issues where people will manually create SPNs incorrectly. The correct format is: **MSSQLSvc/<SQL Server computer name>:1433 AccountName** **MSSQLSvc/<SQL Server FQDN>:1433  AccountName** Strangely enough I've seen semi-colons or commas instead of colons used which simply doesn't work. The easiest way to find these SPNs is to use the Text Filter using contains **;** or **,** ![](http://images.sev17.com/contains-150x150.jpg) The next thing I'll do is normalize the data. Get-SqlSpn lists the accounts without the domain prefix, while Get-SqlWmi does so I'll do a global replace to remove the domain slash. In order to define a unique key, I'll create a column which combines the SPN and service account name: = B6 & " "  &C6   I'll name the column searchSPN, this will allow me to do matching against the server list as we'll see in a moment. ![](http://images.sev17.com/searchSpn-150x150.jpg) On the servers worksheet I'll create two new columns of spn and combined as follows: = "MSSQLSvc/" & D2 & ".contoso.com:" &A2 = E2 & " " &C2 I'll then name the combine column (unique key) searchServer ![](http://images.sev17.com/searchServer-150x139.jpg) Next I'll add a column called matched to each worksheet and use the MATCH function to look for well, matches between SPN and Server lists: =MATCH(E2,searchServer,0) =MATCH(F2,searchSpn,0) If there's a match the match column will list the row number of corresponding match and if not you'll see #NA. You can then filter on #NA and try to figure out why there are mismatches. Did I miss a server in inventory? Are SPNs defined for servers no longer on the network, etc. And finally I'll add one more column of a setspn command to delete or add SPNs using a Excel formula: ="setspn -D " & E2 ="setspn -A " & F2 It may seem a little odd to build up commands in this way rather than using Powershell script, but I'm really paranoid about SPNs and want to verify each action. If you're not careful you can cause some authentication problems by deleting needed SPNs or adding duplicates. _Note: a quick way to find duplicate SPNs is to use the command-line **setspn -T * -X**. You can also use  Excel's conditional formatting to quickly identify duplicate values as described in [here](http://www.addictivetips.com/windows-tips/excel-2010-duplicate-unique-values/). _ I've seen duplicate SPNs cause SQL Server to fallback to NTLM instead of using Kerberos. Searching for duplicate SPNs should be part of  your reconciliation. To fix simply remove the duplicate SPN.__

## Summary

## Comments

**[PARAG](#281 "2012-03-07 23:52:56"):** Great post. very handy . I am thinking of doing the same to collect delegation info for each account.

