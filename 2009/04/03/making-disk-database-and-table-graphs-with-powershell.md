title: Making Disk, Database and Table Graphs with Powershell
link: http://sev17.com/2009/04/03/making-disk-database-and-table-graphs-with-powershell/
author: Chad Miller
description: 
post_id: 9950
created: 2009/04/03 11:18:00
created_gmt: 2009/04/03 15:18:00
comment_status: open
post_name: making-disk-database-and-table-graphs-with-powershell
status: publish
post_type: post

# Making Disk, Database and Table Graphs with Powershell

Sometimes seeing data visually rather than in text format makes things easier to understand. For instance, I prefer to use [WinDirStat](http://windirstat.info/) to see a [Treemap](http://en.wikipedia.org/wiki/Treemap) of disk space usage by file and type rather than a sorted text list of large files. Seeing the files visually I can immediately determine which files are taking up the most space. I also would rather view database file space usage in the TaskPad view of SQL Server 2000 Enterprise Manager instead of running a query. Outside of pre-built tools I'll often use Excel pivot charts to quickly create graphs. In Powershell creating your own graphs is fairly simple thanks to [Joel Bennet's (Jaykul)](http://huddledmasses.org/), [Powerboots](http://huddledmasses.org/powerboots/) and [Visifire SiliverLight and WPF charts](http://visifire.com/).

**Getting Started**

  1. Install Powerboots
  2. Install [SQL Server Powershell Extensions (SQLPSX) ](http://sqlpsx.codeplex.com/)
  3. Download the open source [Visifire components](http://visifire.com/download_silverlight_charts.php)

*_Note: I'm using Powershell V1 and I've placed Powerboots, SQLPSX, and Visifire in my ...DocumentsWindowsPowerShellLibraries directory. In addition since Powerboots and Visifire use WPF, the .NET 3.5 Framework is required._

Once you've completed the install you can use this script to create a test chart:

$libraryDir = **Convert-Path** (**Resolve-Path** "$ProfileDirLibraries") **[Void][Reflection.Assembly]**::LoadFrom( (**Convert-Path** (**Resolve-Path** "$libraryDirWPFVisifire.Charts.dll")) ) **if** (!(**Get-PSSnapin** | ?{$_.name -**eq** 'PoshWpf'})) { **Add-PsSnapin** PoshWpf } **New-BootsWindow** -Async { $chart = **New-Object** Visifire.Charts.Chart $chart.Height    = 300 $chart.Width     = 600 $chart.watermark = $false $ds = **New-Object** Visifire.Charts.DataSeries $ds.RenderAs = **[Visifire.Charts.RenderAs]**"Bar" $chart.Series.Add($ds) **for**($i=0; $i -**lt** 5; $i++) { $dp = **new-object** Visifire.Charts.DataPoint $dp.YValue = (**get-random** -min 1 -max 20) $ds.DataPoints.Add($dp) } $chart } -Title "Test"

Here are a few examples I've posted on [Poshcode](http://poshcode.org/) for disk, database and table space graphs.

[WPFDiskSpace](http://poshcode.org/992) \-- Uses Powerboots and Visifire to display a WPF graph of disk space including percent used and free

**Get-WmiObject** Win32_LogicalDisk -**filter** "DriveType=3" | ./WPFDiskSpace.ps1 ![](http://images.sev17.com/wpfdisk.jpg)

[WPFDbSpace](http://poshcode.org/993) \-- Uses Powerboots, Visifire and SQLPSX to Display a WPF graph of SQL Server database data and log file space:

**Get-SqlDatabase** 'Z002Sql2k8' | **where** {$_.name -**like** "pubs*"} | ./WPFDbSpace.ps1 

![](http://images.sev17.com/wpfdb.jpg)

[WPFTableSpace](http://poshcode.org/994) \-- Uses Powerboots, Visifire and SQLPSX to display a WPF graph of SQL Server table data and index space usage:

./WPFTableSpace.ps1 'Z002SqlExpress' AdventureWorks 

![](http://images.sev17.com/wpftable.jpg)

The graphs I've created so far have been simple/static graphs, but you can do a lot more with Powerboots. For example Joel has a [really cool script](http://huddledmasses.org/pingmonitor-interactive-wpf-uis-from-powershell-10-or-20/) showing a real-time graphical ping monitor. And not only can you make charts, you can use WPF to create some very slick looking screens. In short, Powerboots is an impressive toolkit which I will definitely find additional uses for.