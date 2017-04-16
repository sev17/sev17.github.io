title: Building A Remote Desktop Manager Cluster List
link: http://sev17.com/2010/06/30/building-a-remote-desktop-manager-cluster-list/
author: Chad Miller
description: 
post_id: 10363
created: 2010/06/30 16:37:29
created_gmt: 2010/06/30 20:37:29
comment_status: open
post_name: building-a-remote-desktop-manager-cluster-list
status: publish
post_type: post

# Building A Remote Desktop Manager Cluster List

The [Microsoft Clustering and High Availablity bloggers](http://blogs.msdn.com/b/clustering/archive/2010/06/23/10028855.aspx) have taken noticed of the [Remote Desktop Connection Manager (RDCMan)](http://www.microsoft.com/downloads/details.aspx?FamilyID=4603c621-6de7-4ccb-9f51-d53dc7e48047&displaylang=en) utility from a cluster management perspective. I've previously blogged about [Building A Remote Desktop Manager Connection List](/2010/06/building-a-remote-desktop-manager-connection-list/) from the SQL Server Central Management Server (CMS) tables. The RDCMan use case for cluster administration is very useful and it just so happens that I built a similar solution for cluster servers. So, as a follow up here's a way to auto generate an RDCMan file from a list of clusters, nodes and virtuals. First create and populate a cluster, cluster_node and cluster_virtual tables defined in the following [T-SQL script file](http://cid-ea42395138308430.office.live.com/self.aspx/Public/Blog/rdcman%5E_cluster%5E_dbobjs.sql):  **Note: You'll need to insert your cluster information into the three tables.** Next use the [following T-SQL XQuery to generate the RDCMan](http://cid-ea42395138308430.office.live.com/self.aspx/Public/Blog/rdcman%5E_cluster%5E_query.sql) XML and save the file as cluster.rdg.  Finally open the cluster.rdg file in Remote Desktop Connection Manager. RDCMan allows to you to have multiple rdg files open at one time which means our previous CMS-based list of SQL Server and our cluster-based list can both be used simultaneously. Enjoy!