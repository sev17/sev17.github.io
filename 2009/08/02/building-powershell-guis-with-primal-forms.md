title: Building PowerShell GUIs with Primal Forms
link: http://sev17.com/2009/08/02/building-powershell-guis-with-primal-forms/
author: Chad Miller
description: 
post_id: 9971
created: 2009/08/02 11:41:00
created_gmt: 2009/08/02 15:41:00
comment_status: open
post_name: building-powershell-guis-with-primal-forms
status: publish
post_type: post

# Building PowerShell GUIs with Primal Forms

Although PowerShell is best suited for console applications there are times when a GUI interface just makes sense, however hand cranking PowerShell code to display a GUI seems almost anti-productive when we are used to rich IDE's with WYSIWIG development available in other programming languages. Fortunately there are several 3rd party tools like [PowerGUI](http://www.powergui.org/), CodePlex projects like [Powerboots ](http://powerboots.codeplex.com/)as well as many great data visualization scripts which reduce the amount of time to build GUIs in PowerShell. In this blog post we'll look at another option for buidling GUIs, a free utility from [SAPIEN ](http://www.sapien.com/)called [Primal Forms](http://www.primaltools.com/downloads/communitytools/signup.asp?tool=pforms).

To demonstrate Primal Forms, I thought it would be interesting to create a basic CRUD form for a SQL Server table, so I adapted the example on [this MSDN page](http://chadwickmiller.spaces.live.com/How to: Bind Data to the Windows Forms DataGridView Control) to PowerShell. Although the example uses the authors table in the sample pubs database, I can think of many real-world applications with tables used solely by sys admins to control user access or store configuration data about an application. In order to provide an interface to security and configuration tables a web front end could be developed, PowerShell scripts or cmdlets created or an MS Access front end could be used. All true, but I think the use of PowerShell and a WinForm datagridview provides a light-weight alternative that is completely accessible to PowerShell savvy admins.

Primal Forms is a simple IDE for building WinForms; if you've used Visual Studio, the form development feels similar. In the screenshot below I created a form with three controls: a dataGridView, and two buttons (reload and submit). Once you've created the form you can save the form definition in an XML format for later editing within Primal Forms. When you're ready to create a PowerShell script, select Export to PowerShell. Primal Forms creates all the necessary PowerShell code for the WinForm and controls which you can save to a .ps1 file. In the example that follows, I've named the script **dataGrid.ps1**.

**Primal Forms IDE**

![](http://images.sev17.com/PrimalForms.jpg)

All that's left to do is add event handling to the form load and various forms controls (buttons). _Note the actual user of the PowerShell script does not need to have Primal Forms installed. Primal Forms simply generates 100% compliant PowerShell code for WinForms. _Adding the event handling must be done in a text editor. Open the newly created dataGrid.ps1 file in your PowerShell script editor of choice. Primal Forms creates placeholders for the events. In this example the following TODO placeholders are generated:

#Provide Custom Code for events specified in PrimalForms. $Form1_Load= { #TODO: Place custom script here } $submitButton_Click= { #TODO: Place custom script here } $reloadButton_Click= { #TODO: Place custom script here } 

To populate the datagridview and events for the submit and reload buttons add the following code:

#endregion Generated Form Objects $bindingSource1 = **new-object** System.Windows.Forms.BindingSource $dataAdapter = **New-Object** System.Data.SqlClient.SqlDataAdapter $serverName = "$env:computernamesqlexpress" $databaseName = "Northwind" $query = 'select * from Customers' #---------------------------------------------- #Generated Event Script Blocks #---------------------------------------------- #Provide Custom Code for events specified in PrimalForms. $Form1_Load= { $dataGridView1.DataSource = $bindingSource1 $connString = "Server=$serverName;Database=$databaseName;Integrated Security=SSPI;" $dataAdapter.SelectCommand = **new-object** System.Data.SqlClient.SqlCommand ($query,$connString) $commandBuilder = **new-object** System.Data.SqlClient.SqlCommandBuilder $dataAdapter $dt = **New-Object** System.Data.DataTable **[void]**$dataAdapter.fill($dt) $bindingSource1.DataSource = $dt $dataGridView1.AutoResizeColumns(**[System.Windows.Forms.DataGridViewAutoSizeColumnsMode]**::AllCellsExceptHeader) } $submitButton_Click= { $dataAdapter.Update($bindingSource1.DataSource) } $reloadButton_Click= { $dataGridView1.DataSource = $bindingSource1 $connString = "Server=$serverName;Database=$databaseName;Integrated Security=SSPI;" $dataAdapter.SelectCommand = **new-object** System.Data.SqlClient.SqlCommand ($query,$connString) $commandBuilder = **new-object** System.Data.SqlClient.SqlCommandBuilder $dataAdapter $dt = **New-Object** System.Data.DataTable **[void]**$dataAdapter.fill($dt)

## Comments

**[Chad Miller](#85 "2009-12-16 11:41:00"):** Tracey, I see you added a function called OnApplicationLoad. I have no such function (or maybe the new version of Primal forms generates this code). At first glance it looks like a problem with variable scope. In my code I define the $bindingSource1, $dataAdapter, $serverName, $databaseName and $query variable as parent function level variables, so they are visible to the child functions. In your code it appears some of these variables are defined with OnApplicationLoad only. I would suggest starting with my code base and changing the database, query and server name variables.

**[tracy conley](#86 "2009-12-16 11:41:00"):** Can you help me out? I am getting the following error when i try to use your code pointed at my database.  
ERROR: Property 'SelectCommand' cannot be found on this object; make sure it exists an  
ERROR: d is settable.  
ERROR: At line:40 char:15  
  
I am using the full version of primal forms so i can edit the code directly in the application.  
  
function OnApplicationLoad {  
#Note: This function runs before the form is created  
#Note: To get the script directory in the Packager use: Split-Path $hostinvocation.MyCommand.path  
#Note: To get the console output in the Packager (Windows Mode) use: $ConsoleOutput (Type: System.Collections.ArrayList)  
#TODO: Add snapins and custom code to validate the application load  
  
return $true #return true for success or false for failure  
  
$bindingSource1 = new-object System.Windows.Forms.BindingSource  
$dataAdapter = New-Object System.Data.SqlClient.SqlDataAdapter  
$commandBuilder = new-object System.Data.SqlClient.SqlCommandBuilder $dataAdapter  
  
$SQLServer = "server"  
$databaseName = "testdb"  
$Query="SELECT * from queues"  
  
  
}  
  
$handler_form1_Load={  
#TODO: Place custom script here  
  
$dataGridView1.DataSource = $bindingSource1  
$connString = "Server=$serverName;Database=$databaseName;Integrated Security=SSPI;"  
  
$dataAdapter.SelectCommand = new-object System.Data.SqlClient.SqlCommand ($query,$connString)  
$commandBuilder = new-object System.Data.SqlClient.SqlCommandBuilder $dataAdapter  
$dt = New-Object System.Data.DataTable  
[void]$dataAdapter.fill($dt)  
  
$bindingSource1.DataSource = $dt  
  
$dataGridView1.AutoResizeColumns([System.Windows.Forms.DataGridViewAutoSizeColumnsMode]::AllCellsExceptHeader)  
  
}

**[Chris](#305 "2013-03-21 13:44:53"):** Hi, Loved this article! I am using PrimalForms Community Edition and was losing my mind trying to add a DataGridView to a form. Didn't realize you couldn't define it within the community edition IDE itself, but had to do it in the ps1 file itself. Your code was enough to get me pointed in the right direction. I know this is an old article and I'm not sure if you still even moderate/answer questions within the comments on it, but I did have a few I am hoping you (or someone) may know: 1\. Is it possible to omit certain columns from the DataGridView? Or is it an all or nothing thing? I have a view columns I would like to hide from the user, and manipulate behind the scenes. 2\. Is there a way to turn one column in the DataGridView into a combobox in terms of behavior? Hopefully someone knows. Thank you for the article though, has been a huge help!

**[Chad Miller](#306 "2013-03-21 19:45:33"):** It's been a while since I look at PrimalForms although I recently had to create a simple GUI for a script at work last week. As far as your questions--I'm not sure, but it seems reasonable. Often times when you're using Powershell more like C# you'll find more luck looking at the MSDN documentation search for (DataGridView) or searching for C# code samples. In addition or instead of doing that ask the question on StackOverflow--you'll probably get an answer within 30 minutes of posting it.

