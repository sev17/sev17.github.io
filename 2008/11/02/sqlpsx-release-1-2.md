title: SQLPSX Release 1.2
link: http://sev17.com/2008/11/02/sqlpsx-release-1-2/
author: Chad Miller
description: 
post_id: 9923
created: 2008/11/02 18:00:00
created_gmt: 2008/11/02 22:00:00
comment_status: open
post_name: sqlpsx-release-1-2
status: publish
post_type: post

# SQLPSX Release 1.2

I completed [Release 1.2 of SQLPSX](http://www.codeplex.com/SQLPSX/Release/ProjectReleases.aspx?ReleaseId=18997) which adds 14 new functions for working with SQL Server Agent. Here's an example scripting out all SQL Server Agent Jobs: **Get-AgentJob 'Z002' | Get-SQLScripter** And this command will display the job history for a job named backup_all_dbs between 10/15/2008 and 10/31/2008: **$filter = Set-AgentJobHistoryFilter -name 'backup_all_dbs' -startDate '10/15/2008' -endDate '10/31/2008' Get-AgentJobHistory 'Z002' $filter**

** **

The complete list of new functions added in the 1.2 Release:

**Get-AgentJobServer** Returns a Microsoft.SqlServer.Management.Smo.Agent.JobServer Object. This is the top level object for Agent.Smo **Get-AgentAlertCategory** Returns an SMO.Agent AlertCategory object or collection of AlertCategory objects **Get-AgentAlert** Returns an SMO.Agent Alert object or collection of Alert objects **Get-AgentJob** Returns an SMO.Agent Job object or collection of Job objects **Get-AgentJobSchedule** Returns an SMO.Agent JobSchedule object or collection of JobSchedule objects for Job Objects **Get-AgentJobStep** Returns an SMO.Agent JobStep object or collection of JobStep objects **Get-AgentOperator** Returns an SMO.Agent Operator object or collection of Operator objects **Get-AgentOperatorCategory** Returns an SMO.Agent OperatorCategory object or collection of OperatorCategory objects **Get-AgentProxyAccount** Returns an SMO.Agent ProxyAccount object or collection of ProxyAccount objects **Get-AgentSchedule** Returns an SMO.Agent JobSchedule object or collection of JobSchedule objects for JobServer Shared Schedules **Get-AgentTargetServerGroup** Returns an SMO.Agent TargetServerGroup object or collection of TargetServerGroup objects **Get-AgentTargetServer** Returns an SMO.Agent TargetServer object or collection of TargetServer objects **Set-AgentJobHistoryFilter** Sets filtering option used in Get-AgentJobHistory function **Get-AgentJobHistory** Returns an DataTable of job history, filtering can be applied by using the Set-AgentJobHistoryFilter functio

****

I choose to put the Agent related functions into a separate Library file, LibraryAgent.ps1. I did this because the Agent related objects are in the Smo.Agent namespace instead of the base Smo namespace, so it made sense to use separate Library file. You'll need to source the additional Library file to load function definitions. With Release 1.2 complete, I'm planning the next three releases as follows:

  * Release -- 1.3 will add functions for working with replication using RMO
  * Release -- 1.4 will add functions for working with Integration Services
  * Release -- 1.5 will add Remove, Add, and Update functions where appropriate (prior releases have focused soley on Get functions)
My goal is to have the releases completed by the end of this calendar year (2008).