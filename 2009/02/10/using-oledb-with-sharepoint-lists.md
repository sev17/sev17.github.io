title: Using OLEDB  with SharePoint Lists
link: http://sev17.com/2009/02/10/using-oledb-with-sharepoint-lists/
author: Chad Miller
description: 
post_id: 9940
created: 2009/02/10 20:52:00
created_gmt: 2009/02/11 00:52:00
comment_status: open
post_name: using-oledb-with-sharepoint-lists
status: publish
post_type: post

# Using OLEDB  with SharePoint Lists

I noticed the folks at [connectionstrings.com ](http://www.connectionstrings.com/)provide an example connection string which uses [OleDb to connect to a SharePoint list.](http://www.connectionstrings.com/sharepoint) Being a database person, the idea of querying a SharePoiint list like a table sounded interesting, so I set out to write a few lines of Powershell to test the idea:

The only install needed is the [2007 Office System Driver: Data Connectivity Components](http://www.microsoft.com/downloads/details.aspx?FamilyID=7554F536-8C28-4598-9B72-EF94E038C891&displaylang=en#Requirements). Note: This step is not necessary if you have Office 2007. Next I created a new SharePoint list called "test" with a default column of "Title" and added a couple of items to the list using the SharePoint UI.  Using the technique described in [Finding the Id (Guid) for a SharePoint List](http://nickgrattan.wordpress.com/2008/04/29/finding-the-id-guid-for-a-sharepoint-list/), I obtained the GUID of SharePoint list which is used to construct  the connection string.

The following Powershell commands illustrate selecting, updating and deleting items from a SharePoint list.

#Select $connString = 'Provider=Microsoft.ACE.OLEDB.12.0;WSS;IMEX=2;RetrieveIds=Yes; DATABASE=<http://sharepoint.acme.com/IT/DBAdmin/;LIST={a113df9b-e56e-49d2-b786-03d170d18dbc};>' $spConn = **new-object** System.Data.OleDb.OleDbConnection($connString) $spConn.open() $qry='Select * from list' $cmd = **new-object** System.Data.OleDb.OleDbCommand($qry,$spConn) $da = **new-object** System.Data.OleDb.OleDbDataAdapter($cmd) $dt = **new-object** System.Data.dataTable $da.fill($dt) > $null $dt #Update $connString = 'Provider=Microsoft.ACE.OLEDB.12.0;WSS;IMEX=2;RetrieveIds=Yes; DATABASE=<http://sharepoint.acme.com/IT/DBAdmin/;LIST={a113df9b-e56e-49d2-b786-03d170d18dbc};>' $spConn = **new-object** System.Data.OleDb.OleDbConnection($connString) $spConn.open() $qry = "UPDATE LIST SET Title = 'Test1' WHERE Title = 'Title1'" $cmd = **new-object** System.Data.OleDb.OleDbCommand($qry,$spConn) $cmd.ExecuteNonQuery() #Delete $connString = 'Provider=Microsoft.ACE.OLEDB.12.0;WSS;IMEX=2;RetrieveIds=Yes; DATABASE=<http://sharepoint.acme.com/IT/DBAdmin/;LIST={a113df9b-e56e-49d2-b786-03d170d18dbc};>' $spConn = **new-object** System.Data.OleDb.OleDbConnection($connString) $spConn.open() $qry = "DELETE FROM LIST WHERE Title='Test'" $cmd = **new-object** System.Data.OleDb.OleDbCommand($qry,$spConn) $cmd.ExecuteNonQuery()  #Insert. NOTE: THIS DOES NOT WORK! $connString = 'Provider=Microsoft.ACE.OLEDB.12.0;WSS;IMEX=2;RetrieveIds=Yes; DATABASE=<http://sharepoint.acme.com/IT/DBAdmin/;LIST={a113df9b-e56e-49d2-b786-03d170d18dbc};>' $spConn = **new-object** System.Data.OleDb.OleDbConnection($connString) $spConn.open() $qry = "INSERT INTO LIST (Title) VALUES ('Test3')" $cmd = **new-object** System.Data.OleDb.OleDbCommand($qry,$spConn) $cmd.ExecuteNonQuery() 

For some reason the select, update and delete work perfectly, however the insert statement produces the following error:

Exception calling "ExecuteNonQuery" with "0" argument(s): "Cannot update 'Title'; field not updateable. At line:1 char:21 \+ $cmd.ExecuteNonQuery( <<<< )

I've tried different variations of the insert command including adding a row to a DataTable and updating the DataAdapter and using OleDbcommandbuilder all with the same result. After a few hours of trying to make the insert work and web searches I noticed [other people have reported the same issue](http://social.msdn.microsoft.com/Forums/en-US/sharepointdevelopment/thread/4230eeb2-d66e-4013-a723-12721c2c97e1). Lacking support for adding items to a list is big deal and greatly limits the applicability of using OleDB against a SharePoint list, nonetheless I may still use the select technique to return a list as a DataTable from Powershell.

## Comments

**[Chad Miller](#26 "2009-02-18 20:52:00"):** Sorry I haven't seen that issue. My usage of OLE DB to SharePoint has been limited. I do wonder if there is some kind of a timeout. It doesn't appear this is settable option for a SharePoint connection string.

**[Frank Daske](#27 "2009-02-18 20:52:00"):** Hi Chad,   
we offer the SharePoint Business Data List Connector to connect SharePoint lists to external data with update:  
<http://www.layer2.de/en/products/pages/sharepoint-business-data-list-connector.aspx>  
Generally, as you describe, OLEDB can be used to connect SharePoint lists to SharePoint lists too, e.g. cross site-collection.  
  
But sometimes we see a stange error message there:  
  
Cannot connect to the Sharepoint site '<http://myserver/sites/test/Lists/MyList/>'. Try again later.   
  
Any idea, what happens there?

**[Chad Miller](#28 "2009-07-10 20:52:00"):** Thanks for the info, I'll try it.

**[Praful Nama](#29 "2009-07-08 20:52:00"):** Hey Chad,  
Try this - Open Internet Explorer with your service account credentials(using Run as). Under Internet Options, on the security tab, click on Local intranet. Click on sites, advanced and add your sharepoint site to the list. Then click on Custom level... and in the User Authentication section, under Logon, select the Automatic logon with current user name and password option. Try scheduling your script now. (For some reason I couldn't add sites to my Local intranet zone so I configured the Internet zone to use the current username and password. It seems to work). Let me know if it works or not!

**[Chad Miller](#30 "2009-07-06 20:52:00"):** I noticed the same problem in my envrionment. Our web folks put SSL on our SharePoint servers a couple of months after I created the Powershell and like you, my script stopped working. I'm also prompted for usernae/password. I've tried to workaround the issue, but haven't come up with a solution. I've noticed that once you've logged into a desktop session you are not prompted nor are you prompted after the second login prompt when using run as. You could run the script from locked desktop running under the service account, but that is not ideal. At this point I'm just running my script interactively rather than scheduled. If you find a solution, please post.

**[Praful Nama](#31 "2009-07-06 20:52:00"):** Thanks for the excellent post. This is exactly what I am trying to do. There is one particular thing that is not clear and I would be thankful if you could shed some light on it. I am trying to read a sharepoint list and when I execute the powershell script as myself, it works perfectly. But if I launch powershell (using run as) using a service account ( that has access to the list), after the da.fill() statement, I get a prompt asking me to enter the credentials for accessing the sharepoint site. When I enter the credentials for the service account, it works. This is preventing me from being able to schedule this as a job on a server 2003 machine since, it keeps prompting me for credentials. I don;t understand as to why is it behaving differently for the service account. Any ideas?  
  
Thanks!!  
-p

**[Max](#32 "2009-02-15 20:52:00"):** Just wanted to thank you for an Excellent blog. I gave myself a 1% chance of getting bulksqlcopy to work on a discountasp.net server but your tips worked fabulously!

**[Max](#33 "2012-09-19 06:45:01"):** IMEX=0 is supposed to make writes work. Have a look at (the updated reference) http://www.connectionstrings.com/sharepoint

