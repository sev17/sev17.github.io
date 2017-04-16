title: SQLPSX Release 1.3
link: http://sev17.com/2008/12/14/sqlpsx-release-1-3/
author: Chad Miller
description: 
post_id: 9929
created: 2008/12/14 12:46:00
created_gmt: 2008/12/14 16:46:00
comment_status: open
post_name: sqlpsx-release-1-3
status: publish
post_type: post

# SQLPSX Release 1.3

I completed [Release 1.3 of SQLPSX](http://www.codeplex.com/SQLPSX/Release/ProjectReleases.aspx?ReleaseId=20521) which adds 21 new functions for working with SQL Server Replication via RMO. With this release there are now 59 total functions and 10 scripts around SMO, Agent, RMO.

Here's a few example of working with SQL Server Replication through RMO: 

Get a reference to a Replication Server. Note: server may or may not participate in replication several properties will indicate: **$replServer = Get-Replserver 'Z002Sql1'** Get all of  Subscriptions on a subscriber server. Note: Unlike the other functions that are executed on the publisher, this is the only function which should be executed on the subscriber: **Get-ReplSubscriberSubscription $replServer** Get all Publications: **Get-ReplPublication $replServer** Get all registered Subscriptions of all Publications **Get-ReplPublication $replServer | Get-ReplSubscription** Get all Articles of all Publications: **Get-ReplPublication $replServer | Get-ReplArticle** Monitoring examples that show the same type of information available in Microsoft's GUI Replication Monitor utility **$replMon = Get-ReplMonitor 'Z002Sql1' $publisherMon = Get-ReplPublisherMonitor 'Z002Sqlqa1 $pubMon = Get-ReplPublicationMonitor 'Z002Sql1' $pubMon | Get-ReplTransPendingCommandInfo $publisherMon | Get-ReplEnumPublications $publisherMon | Get-ReplEnumPublications2 $pubMon | Get-ReplEnumSubscriptions $pubMon | Get-ReplEnumSubscriptions2 $pubMon | Get-ReplenumlogReaderAgent $pubMon | Get-ReplenumSnapshotAgent** Script out Replication Server, Publication, Subscription and Articles: **$replServer | Get-Replscript Get-Replpublication 'Z002Sql1' | Get-ReplScript Get-Replpublication 'Z002Sql1' | Get-ReplSubscription | Get-ReplScript Get-Replpublication 'Z002Sql1' | Get-ReplArticle | Get-ReplScript**

The complete list of new functions added in the 1.3 Release:

**Get-SqlConnection** Returns a ServerConnection object **Get-ReplServer** Returns an RMO.ReplicationServer **Get-ReplLightPublication ** Returns an RMO.LightPublication **New-ReplTransPublication ** Constructor for RMO.TransPublication **New-ReplMergePublication** Constructor for RMO.MergePublication **Get-ReplSubscriberSubscription** Returns an RMO.SubscriberSubscription. Note: this is the only function executed on a subscriber **Get-ReplPublication** Returns either an RMO.TransPublication or RMO.MergePublication object **Get-ReplSubscription** Returns an RMO.TransSubscription or RMO.MergeSubscription object from a Publication **Get-ReplArticle ** Returns an RMO.TransArticle or RMO.MergeArticle object from a Publication **Get-ReplMonitor ** Returns an RMO.ReplicationMonitor **Get-ReplPublisherMonitor ** Returns an RMO.PublisherMonitor **Get-ReplPublicationMonitor ** Returns an RMO.PublicationMonitor **Get-ReplEnumPublications** Calls the EnumPublications method on a PublisherMonitor object **Get-ReplEnumPublications2** Calls the EnumPublications method on a PublisherMonitor object **Get-ReplEnumSubscriptions** Calls the EnumSubscriptions method on a PublicationMonitor object **Get-ReplEnumSubscriptions2** Calls the EnumSubscriptions2 method on a PublicationMonitor object **Get-ReplTransPendingCommandInfo ** Calls the TransPendingCommandInfo method on a PublicationMonitor object **Get-ReplEnumLogReaderAgent ** Calls the EnumLogReaderReader method on a PublicationMonitor object **Get-ReplEnumSnapshotAgent ** Calls the EnumSnapshotAgent method on a PublicationMonitor object **Set-ReplScriptOptions ** Sets the Enum ScriptOptions for scripting RMO objects. Unlike SMO which has a default script options RMO at at a minimum CREATION enum must be specified. **Get-ReplScript** Calls Script Method on RMO objects include ReplicationServer, Publication, Subscription and Articles

Just as in the 1.2 released focused on SQL Agent I choose to put the Replication related functions into a separate Library file, LibraryRMO.ps1. I did this because the RMO related objects are in the Management.SqlServer.Replication namespace instead of the Smo namespace, so it made sense to use separate Library file. You'll need to source the additional Library file to load function definitions. With Release 1.3 complete, I'm starting work on the 1.4 Release which will add functions for working with Integration Services.

I noted in the [post on the 1.2 Release](/2008/12/sqlpsx-release-1-3/) my goal is to to have two more releases completed by the end of this calendar year (2008). Other priorites have meant I've spent less time working on SQLPSX and given that December is half over, February or March 2009 is more likely.

## Comments

**[sreenath](#320 "2013-06-21 07:23:57"):** Using the module, how do I sync subscription server with publication server on one time basis ? The requirement for us is we have taken backup of publication database and we need to sync with subscription database after all mundane configuration. We want this to be automated as the sync process normally takes about 2-3 hours to complete. Any help regarding this would be appreciated. Thanks for making this great module.

**[Chad Miller](#323 "2013-06-21 19:15:42"):** The replication module doesn't have any functions for creating or updating replication. You may want to ask the question in a SQL Server forum.

