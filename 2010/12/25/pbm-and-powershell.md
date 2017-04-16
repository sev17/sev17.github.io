title: PBM and PowerShell
link: http://sev17.com/2010/12/25/pbm-and-powershell/
author: Chad Miller
description: 
post_id: 10536
created: 2010/12/25 13:17:19
created_gmt: 2010/12/25 18:17:19
comment_status: open
post_name: pbm-and-powershell
status: publish
post_type: post

# PBM and PowerShell

The best new management feature added to SQL Server 2008 is [Policy Based Management](http://msdn.microsoft.com/en-us/library/bb510667.aspx) or PBM. PBM allows DBAs to automate many of the traditional daily (or more frequent) checklists*. Although PBM is useable out-of-box there are several key features missing:

  * **Unable to capture results of policy evaluations against downward-level versions** (2000 and 2005). The required system tables used for storing policy evaluation results are only available in SQL Server 2008 or higher. So, out-of-the-box you can only use the GUI (SQL Server Management Studio) to “see” the results of downward-level evaluations**. What’s needed is the ability to capture the results of downward level versions. 
  * **Lack of integration with enterprise monitoring tools**. Traditionally integration features for monitoring solutions are provided by SNMP traps or Ops Mgr Management Packs. Even writing messages to the Windows Event Log on policy evaluation failure is at least something, but this too is missing or incomplete. What’s needed is at minimum is the ability to write messages to the Windows Event Log. 
  * **Provides an Instance-level view instead of instead of an enterprise view**. Although you can visually see the results of policy evaluation within SQL Server Management Studio, again this is only applicable to 2008 or higher and the results are stored within tables on each instance. What’s needed is the ability to consolidate and report the results of policy evaluations across all instances for any version. 

How can these issues be addressed…?

## Enter PowerShell

One of the key strengths of any scripting language is the ability to glue together existing tools to make another tool or to fill gaps in current tools. This is especially relevant for database professional stuck on older versions of SQL Server. Support for new management features like PBM isn’t being provided to SQL 2000/2005. PowerShell provides great way to bridge these gaps and provide PBM support for older versions of SQL Server. I’ve often heard remarks a reason for not using PowerShell by DBAs is that they do not use SQL Server 2008, so PowerShell is less relevant to them, but on the contrary PowerShell is necessary to allow you to address with missing functionality. 

### Invoke-PolicyEvaluation cmdlet

The SQL Server 2008 (or 2008 R2) host, sqlps provides a cmdlet called [Invoke-PolicyEvaluation](http://msdn.microsoft.com/en-us/library/cc645987.aspx) which does exactly as the name implies, evaluates policies. To create a solution, all that is needed is a few functions around Invoke-PolicyEvaluation which will insert the policy evaluation result to SQL table and optionally write an error message to the Windows Event log.

### Extending Invoke-PolicyEvaluation

Release 2.3.1 of the CodePlex project, [SQLPSX](http://sqlpsx.codeplex.com/) adds a PBM module***. The module uses the Invoke-PolicyEvaluation cmdlet and then loads the output into a SQL Server table. In addition failed evaluations are optionally written to the even log.

### Getting Started

  1. Download SQLPSX 
  2. The functions obtains a list of servers from a [Central Management Server](http://msdn.microsoft.com/en-us/library/bb895144.aspx) (CMS) Group. Ensure you have a CMS server with group and servers registered. The examples that follow use a group named XA: 

 

![CMS](http://images.sev17.com/CMS_thumb.png)
  3. Create an empty SQL Server database (or use existing database). For example "MDW"
  4. Use SQL Server Management Studio to run the 2 table creation scripts dbo.PolicyEval.Table.sql and dbo.PolicyEvalError.Table.sql 
  5. Copy PBM PowerShell module folder to a directory listed in your $env:psmodulepath 
  6. Modify PBM.psm1 Script-level variables in PBM.psm1 to point to the server and database created in step 1 and set preference variables as desired: 
    
        $Script:EvaluationMode = "Check"
    $Script:PolicyServer = "Z003R2"
    $Script:PolicyDatabase = "MDW"
    $Script:CMS = "Z003SQLEXPRESS"
    $Script:WriteEventLog = $false
    $Script:LogName = "Application"
    $Script:LogSource = "PBMScript"
    $Script:EntryType = "Error"
    $Script:EventId = 34052

  7. Import Policies from **PBMPolicies** folder or use your existing policy 
  8. Ensure policies have a Category set. In SSMS >> Policy Management >> Policies >> <YOUR POLICY> >> Select Description Tab 

![PBM](http://images.sev17.com/PBM_thumb.png)

### Running Import-PolicyEvaluation function

  1. The module should be used in the sqlps mini-shell. Alternatively you can use one of the techniques document in [this blog post](/2010/07/making-a-sqlps-module/) to load the full sqlps providers and cmdlets in regular PowerShell host. Because import-module isn't supported in sqlps, source the functions: 
    
         . C:Usersu00DocumentsWindowsPowerShellModulesPBMPBM.psm1

  2. Run Import-PolicyEvaluation specifying a ConfigurationGroup (the CMS Server Registration Group) and PolicyCategoryFilter (as defined in step 6)   For example: 

 
    
        Import-PolicyEvaluation "XA" "EPM: Configuration"

  3. Optionally create SQL Agent job for each configuration group with the following PowerShell Job Step: 
    
    
    . C:PBM.psm1
    Import-PolicyEvaluation "XA" "EPM: Configuration"

## Evaluating Results

The results of each policy evaluation will be loaded to the PolicyEval table:

![PBM2](http://images.sev17.com/PBM2_thumb.png)

Errors in invoking the policy evaluation will be written to the PolicyEvalError table.

If you sent $Script:WriteEventLog to $true, an event log entry will be created for each policy evaluation failure with the specified event id ($Script:EventId = 34502). **NOTE: In release 2.3.1 this line in PBM.psm1 $log.WriteEntry($Message,$Script:EntryType,$Script:EventId) should be changed to $eventlog.WriteEntry($Message,$Script:EntryType,$Script:EventId)**  The nice thing about this is Ops Manager can pick up the Windows Event log entry and create an alert. In my environment we’ll take it a step further towards enterprise integration and even create incidents in our help desk application via connectivity between Ops Manager and CA Service Desk.

The PBM module includes two SSRS reports (a master and detail) which show overall success as well as drill-through to details: