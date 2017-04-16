title: Disk Alignment Partitioning: The Good, the Bad, the OK and the Not So Ugly
link: http://sev17.com/2009/02/28/disk-alignment-partitioning-the-good-the-bad-the-ok-and-the-not-so-ugly/
author: Chad Miller
description: 
post_id: 9945
created: 2009/02/28 11:34:00
created_gmt: 2009/02/28 15:34:00
comment_status: open
post_name: disk-alignment-partitioning-the-good-the-bad-the-ok-and-the-not-so-ugly
status: publish
post_type: post

# Disk Alignment Partitioning: The Good, the Bad, the OK and the Not So Ugly

Lately I've been reading about the [disk alignment partitioning](http://sqlblog.com/blogs/kevin_kline/archive/2008/10/08/how-to-improve-application-and-database-performance-up-to-40-in-one-easy-step.aspx) and performance degragaton that occurs in Windows 2003 when using the wrong partition NTFS partition offset. There is an I/O performance impact of 30 - 40% when disk partitions are not properly aligned. So, this starting me thinking, what is good offset anways and how can I write a Powershell script to verify disk are configurated correctly?

It turns out Windows 2003 uses a 64KB  or 128 sectors (64KB = 128 sectors) so you want to choose 64 or a multiple of 64. However different storage arrays use varying offsets (4, 64, 256, 512) and not only do you have to consider the Windows offset you should also take into account the storage array offsets to ensure they are aligned. Because of the need to consider the storage array offsets, Microsoft KB artcle [929491](http://support.microsoft.com/kb/929491) recommends a starting offset of 2,048 sectors (1 megabyte). The 2,048 offset cover most storage arrays including EMC Symmetrix and CLARIION which may use a setting other than 64KB depending on the configuration. An offset of 63 which is the detault offset for MBR, is a bad setting. For the boot partition 63 is OK,  but for a SQL Server data and log drive 63 is bad always.

With the question of a good offset answered; 64 might be OK, but 2,048 will cover almost any SAN configuration, I created a Powershell script to verify disk settings. I want to report the system name, logical disk name, partition name, blocksize starting offset, and finally the start sector. The start sector is calucated by dividing the StartingOffset by the value of BlockSize as described in the "More Information" section of KB article [929491](http://support.microsoft.com/kb/929491). The WMI classes Win32_LogicalDisk and Win32_DiskPartition provide the needed information, however we'll need to use the association class Win32_LogicalDiskToPartition to link logical disks with partitions. .

Save the script below as a ps1 file, **partalign.ps1**, and run by specifying the computer name as a parameter** ./partalign.ps1 Z002**

**param** ($computer) $partitions = **Get-WmiObject** -computerName $computer Win32_DiskPartition  $partitions | **foreach** { **Get-WmiObject** -computerName $computer -query "ASSOCIATORS OF {Win32_DiskPartition.DeviceID='$($_.DeviceID)'} WHERE AssocClass = Win32_LogicalDiskToPartition" | **add-member** -membertype noteproperty PartitionName $_.Name -passthru | **add-member** -membertype noteproperty Block $_.BlockSize -passthru | **add-member** -membertype noteproperty StartingOffset $_.StartingOffset -passthru | **add-member** -membertype noteproperty StartSector $($_.StartingOffset/$_.BlockSize) -passthru } | Select SystemName, Name, PartitionName, Block, StartingOffset, StartSector  Using the WMI assocation classes is something you won't have do very often, but its not too ugly. You could construct the Powershell WMI query as one statement, but I found it easier to understand to break up the statments in two parts (Partition, and LogicalDisk). The Win32_LogicalDiskToPartition class returns a Win32_LogicalDisk WMI object which I then add the partition information to using add-member. I have a few servers with mount points and use the Win32_Volume class for returning space usage information. I haven't figured out a way to associate mount points to partitions and I don't think the WMI association classes exist to do so. If anyone knows how to associate mount points with partitions using WMI please post a comment

## Comments

**[Chad Miller](#36 "2009-03-01 11:34:00"):** Scott thanks for the very detailed information on mount points and partitions. I'll give DiskExt a try. It's unfortunate WMI does have an association class for partitions to mount points, but at least you found a good workaround. I noticed Jimmy's whitepaper which was published after my blog post. This post is from last year on 2/28/2009. I would have liked to have seen WMI and Powershell based examples in the whitepaper, but good stuff nonetheless.  
  
I agree fragementation information is another area to look into. I've done some work with DefragAnalysis and Powershell and even posted a blog with a simple script (probably simplier than VBScript). See <http://sev17.com/2009/10/operations-manager-shell/>

**[Scott](#37 "2009-03-01 11:34:00"):** Chad,  
  
Thanks for your post. Glad that others see the value of mounted volumes (and some of the challenges dealing with them – so many functions and utilities frequently document drive letter examples, but mounted volume examples are less frequently or not at all).  
  
I encountered these same issues some time ago when accessing partition, logical disk, and their association WMI classes programmatically in VBScript. To my knowledge, there is no WMI association class for partitions to mounted volumes. It seems like a “missing link” that should be present, but isn’t.  
  
A WMI association class is like “junction” table in relational DBs, describing a many-to-many relationship between the two base entities / WMI classes (such as the existing partition and logical disk association class, or the desired but not yet existing partition to disk volume association class).  
  
For my solution, I developed a work-around by using the free DiskExt command line utility from Sysinternals (<http://technet.microsoft.com/en-us/sysinternals/bb896648.aspx>). Mark’s earlier version of DiskExt only worked with drive letter disk volumes. Following some e-mail conversations we had a few years ago, he upgraded DiskExt to support mounted volumes as well as drive letter volumes. It generates text output that can both associate the mounted volume name (drive letter or mount point file path) and the Windows OS disk number (Disk 0, etc.), and also displays the volume size (bytes) and the volume offset (bytes – disk alignment offset). So while it doesn’t directly associate the mounted volume to the partition, it gives enough key information (for my purposes) that I could get by without the missing WMI volume to partition association class (although it would be a useful association class to have). The associated partition can be implied from the disk ID / # when the disk (LUN) is configured as a single partition (as many disks are configured – this method doesn’t work for disks / LUNs configured with multiple partitions). This solution is not WMI-based or set-based – I had to integrate the information with WMI data programmatically by parsing the DiskExt text output and using loops (RBAR – row by agonizing row). I wish there was an easier way.  
  
The other problem that I have with WMI class enhancements in general (new classes, new fields within classes, etc.) is they are rarely retroactive to earlier OS product versions or maintenance levels. So for a comprehensive solution to collect desired information over a range of OS versions and SP levels, you need to sense the OS version / SP level and attempt a WMI access strategy that uses only the WMI data that is available for that OS version / SP level (to avoid generating errors and incomplete collected information that is available to collect – if the right access strategy or error trapping were used). Or have a solution that you know will only work on a certain OS version or later (but not on earlier OS versions). Even if the WMI class enhancements were retroactive to earlier OS versions, it would probably be delivered through SPs or hot fixes which may or may not be present on a given older system.  
  
In addition to the KB articles you quoted in your post, check out the SQLCAT article “Disk Partition Alignment Best Practices for SQL Server” (co-authored by Jimmy May – link: <http://sqlcat.com/whitepapers/archive/2009/05/11/disk-partition-alignment-best-practices-for-sql-server.aspx>) and Jimmy May’s posts on disk alignment and the benefits in choosing compatible disk alignment offsets based on disk stripe element size and NTFS allocation unit size (link: <http://blogs.msdn.com/jimmymay/archive/2009/05/08/disk-partition-alignment-sector-alignment-make-the-case-with-this-template.aspx> \- also has links to his earlier posts on disk alignment).  
  
Side note: One additional information source I gathered in my earlier mentioned solution with the WMI disk volume class info is to use the WMI DefragAnalysis method to return the current disk fragmentation state info for that volume (not to initiate actual defragmentation activity – only to report on the current disk fragmentation state for a given disk volume – like programmatically using the command line defrag –a <drive letter or mount point name>). Again, my solution used VBScript, but I suspect this method could be used from PowerShell for similar results (just haven’t done this yet in PowerShell myself). Not directly related to disk volume alignment, but is related to good disk volume provisioning and maintenance practices.  
  
Good luck with your efforts on mapping mounted volumes to their partition information. Let me know if the DiskExt command helps in your efforts, or any other solutions you may find.  
  
Scott R.

