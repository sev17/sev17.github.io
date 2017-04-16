title: Powershell Charting with MS Chart Controls
link: http://sev17.com/2009/07/08/powershell-charting-with-ms-chart-controls/
author: Chad Miller
description: 
post_id: 9967
created: 2009/07/08 22:06:00
created_gmt: 2009/07/09 02:06:00
comment_status: open
post_name: powershell-charting-with-ms-chart-controls
status: publish
post_type: post

# Powershell Charting with MS Chart Controls

[Richard MacDonald's ](http://blogs.technet.com/richard_macdonald/default.aspx)demonstrates using Microsoft Chart Controls with Powershell in his post, [Charting with Powershell](http://blogs.technet.com/richard_macdonald/archive/2009/04/28/3231887.aspx). The chart controls are free and work with standard Windows forms. Another nice thing is the ability data bind to any object that implements IEnumerable (arrays, hashes, data tables, etc.). This makes working with the charts particularly easy, just create a hashtable and bind it to the chart data series. Jeffery Snover provides us with a [useful bit of code called ConvertTo-Hashtable ](http://blogs.msdn.com/powershell/archive/2008/11/23/poshboard-and-convertto-hashtable.aspx)which does what the name implies. Armed with this information, I thought it would interesting to take the concept of Powershell charting a few steps further and create a reusable charting library, called [LibraryChart](http://poshcode.org/1330) *****Updated 9/20/09***** available on [Poshcode](http://poshcode.org/). The library implements several features including:

  * Pipe the output of a Powershell command to automatically create a chart.
  * Display the chart in a Windows Form
  * Save the chart to an image file
  * Specify either bar, column, line or pie chart types
  * Display real-time automatic updating charts by passing scriptblock to the function. The scriptblock will execute at the specified interval and display chart updates (think Perfmon)
  * Works with Powershell V1
**To use the Library **_(Note: Requires .NET 3.5 framework):_

  * Download and install [Microsoft Chart Controls for Microsoft .NET Framework 3.5](http://www.microsoft.com/downloads/details.aspx?familyid=130F7986-BF49-4FE5-9CA8-910AE6EA442C&displaylang=en)
  * Download [LibraryChart](http://poshcode.org/1205)
  * If you want to extend/use MS Chart Control grab the [document](http://www.microsoft.com/downloads/details.aspx?familyid=EE8F6F35-B087-4324-9DBA-6DD5E844FD9F&displaylang=en) and [samples](http://code.msdn.microsoft.com/mschart). I referred to the C# WinForm samples frequently.
Here are a few examples: Create a column chart of process workingset information: **Get-Process** | **Sort-Object** -Property WS | **Select-Object** Name,WS -Last 5 | **out-chart** -xField 'name' -yField 'WS' ![](http://images.sev17.com/MSChartCol.jpg) Save the chart to a file instead of displaying: **Get-Process** | **Sort-Object** -Property WS | **Select-Object** Name,WS -Last 5 | **out-chart** -xField 'name' -yField 'WS' -filename 'c:usersu00documentsprocess.png' **Get-Process** | **Sort-Object** -Property WS | **Select-Object** Name,WS -Last 5 | **out-chart** -xField 'name' -yField 'WS' -chartType 'Pie' ![](http://images.sev17.com/MSChartPie.jpg) Produce a real-time line chart of process working set by passing a scriptblock i.e. the Powershell command between the two curly brackets. (Image note shown): **out-chart** -xField 'name' -yField 'WS' -scriptBlock {**Get-Process** | **Sort-Object** -Property WS | **Select-Object** Name,WS -Last 1} -chartType 'line' I'm not entirely happy with the script (uses global variable, hash generation code repeated, pie and line chart appearance could be improved), so if anyone would like to take the charting library even further go for it!

## Comments

**[Chad Miller](#79 "2009-09-20 22:06:00"):** Thanks Ben, -- I went ahead and updated the chart library with the fixes noted below fixes and on your blog <http://xcud.com/post/192277838/mschart-in-psh>. Feel free to update the code PoshCode directly if you find anything else or would like extend the library.

**[Ben](#80 "2009-09-18 22:06:00"):** On line 56 of the LibraryChart.ps1 lib file change this "[void]$global:Chart.Titles.Add($chartTitle)" to this "[void]$global:Chart.Titles.Add([string]$chartTitle)" then re-import the lib and run again. This fixes the ambiguous overload error on my machine.

**[Hal](#81 "2009-07-15 22:06:00"):** FYI, I got this err when trying to create a chart, although it did work:  
  
Multiple ambiguous overloads found for "Add" and the argument count: "1".  
At C:Documents and Settingshxr8765My DocumentsWindowsPowerShellliblib-LibraryChart.ps1:  
56 char:35  
\+ [void]$global:Chart.Titles.Add <<<< ($chartTitle)  
\+ CategoryInfo : NotSpecified: (:) [], MethodException  
\+ FullyQualifiedErrorId : MethodCountCouldNotFindBest

