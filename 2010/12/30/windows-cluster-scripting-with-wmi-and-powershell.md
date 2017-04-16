title: Windows Cluster Scripting with WMI and PowerShell
link: http://sev17.com/2010/12/30/windows-cluster-scripting-with-wmi-and-powershell/
author: Chad Miller
description: 
post_id: 10546
created: 2010/12/30 22:52:59
created_gmt: 2010/12/31 03:52:59
comment_status: open
post_name: windows-cluster-scripting-with-wmi-and-powershell
status: publish
post_type: post

# Windows Cluster Scripting with WMI and PowerShell

Windows 2008 R2 includes a failoverclusters PowerShell module, but you’ll probably find normal usage limiting for several reasons:

  * Only works on Windows 2008 R2 clusters 
  * Requires the cluster service which in effect makes it difficult to run remotely except via PSRemoting 
  * Implements specialized data types instead of using WMI 

Although I like seeing PowerShell coverage of products, the failoverclusters module seems like a step backwards from Windows 2003 cluster.exe. In Windows 2003 every server installation had a cluster management MMC as well as the command-line utility cluster.exe. The cool thing is you could go to any Windows 2003 machine or Windows XP machine and as long as the admin tools were installed you could manage a cluster remotely from the command-line.

If you’re managing a mixed environment which includes Windows 2003 or you want more flexibility in managing Windows 2008 R2 clusters there’s good coverage in the [Failover Clusters WMI provider](http://msdn.microsoft.com/en-us/library/aa372876%28v=vs.85%29.aspx). All that is needed is to create a few simple functions are the WMI classes. The script  [LibraryMSCS.ps1](http://poshcode.org/2426) posted on [PoshCode](http://poshcode.org/) provides some core functions for retrieving or modifying cluster configurations. Here’s some sample output below…
    
    
    PS C:userspublicbin> . .LibraryMSCS.ps1
    PS C:userspublicbin> get-cluster CLUMPXM
    
    Name                            : CLUMPXM
    
    PS C:userspublicbin> get-clustername CLUMPXM
    CLUMPXM
    PS C:userspublicbin> get-clusternode CLUMPXM
    
    Cluster                     : CLUMPXM
    ...
    Name                        : CLUMP2XP
    
    Cluster                     : CLUMPXM
    ...
    Name                        : CLUMP1XP
    
    PS C:userspublicbin> get-clusternode CLUMPXM | select cluster,Name
    
    Cluster                                                     Name                                                       
    -------                                                     ----                                                       
    CLUMPXM                                                CLUMP2XP                                              
    CLUMPXM                                                CLUMP1XP                                              
    
    PS C:userspublicbin> get-clustersqlvirtual CLUMPXM 
    
    Cluster           : CLUMPXM
    Name              : SQL Server (SQLPROD2)
    State             : 2
    VirtualServerName : CANARY
    InstanceName      : SQLPROD2
    ServerInstance    : CANARYSQLPROD2
    Node              : CLUMP2XP
    
    PS C:userspublicbin> get-clusternetworkname CLUMPXM 
    
    Cluster     : CLUMPXM
    Name        : SQL Network Name (CANARY)
    State       : 2
    NetworkName : CANARY
    Node        : CLUMP2XP
    
    Cluster     : CLUMPXM
    Name        : Cluster Name
    State       : 2
    NetworkName : CLUMPXM
    Node        : CLUMP2XP
    
    Cluster     : CLUMPXM
    Name        : DOVE
    State       : 2
    NetworkName : DOVE
    Node        : CLUMP2XP
    
    PS C:userspublicbin> get-clusterGroup CLUMPXM 
    
    Cluster             : CLUMPXM
    ...
    Node                : CLUMP2XP
    PreferredNodes      : CLUMP1XP
    ...
    Name                : SQL CANARY
    ..
    State               : 0
    Status              : 
    
    Cluster             : CLUMPXM
    Node                : CLUMP2XP
    PreferredNodes      : {}
    ...
    Name                : New Drives
    ...
    State               : 0
    
    Cluster             : CLUMPXM
    Node                : CLUMP2XP
    PreferredNodes      : {}
    ...
    Name                : Cluster Group
    ...
    State               : 0
    Status              : 
    
    Cluster             : CLUMPXM
    Node                : CLUMP2XP
    PreferredNodes      : {}
    ...
    Name                : MSDTC
    ...
    State               : 0
    
    PS C:userspublicbin> get-clusterresource CLUMPXM  | where {$_.Name -like "*SQL*CANARY*"}
    
    Cluster                  : CLUMPXM
    Node                     : CLUMP2XP
    Group                    : SQL CANARY
    ...
    Name                     : SQL Network Name (CANARY)
    ...
    Type                     : Network Name
    
    Cluster                  : CLUMPXM
    Node                     : CLUMP2XP
    Group                    : SQL CANARY
    ...
    Name                     : SQL IP Address 1 (CANARY)
    ...
    Type                     : IP Address