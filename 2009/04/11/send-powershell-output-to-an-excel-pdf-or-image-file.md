title: Send Powershell output to an Excel, PDF, or Image file
link: http://sev17.com/2009/04/11/send-powershell-output-to-an-excel-pdf-or-image-file/
author: Chad Miller
description: 
post_id: 9952
created: 2009/04/11 14:18:00
created_gmt: 2009/04/11 18:18:00
comment_status: open
post_name: send-powershell-output-to-an-excel-pdf-or-image-file
status: publish
post_type: post

# Send Powershell output to an Excel, PDF, or Image file

While reading through Wrox book, [Professional SQL Server 2008 Reporting Services](http://www.amazon.com/gp/product/0470242019) (SSRS) I noticed a chapter on "Integrating Reports Into Custom Applicaiton" where the authors make use of the ReportViewer control to embed an SSRS report in a WinForm application without needing an SSRS server. Since SSRS reports can use DataTables as a data sources I thought this would make a nice way to export the output of any Powershell command/script to a number of different formats natively supported by SSRS including Excel, PDF, and various image formats: BMP,EMF,GIF,JPEG,PNG or TIFF.

In Searching for examples of ReportViewer WinForm implementations I found a [very nice set of classes called ReportExporters](http://www.codeproject.com/KB/reporting-services/ReportExporters_WinForms.aspx) by Andriy Protskiv which greatly simplify working with ReportViewer. Using ReportExporters I created an [Out-Report script](http://poshcode.org/1015) available on [Poshcode](http://poshcode.org/). To use you'll need to install the Microsoft Report Viewer Redistributable, either the 2005 or 2008 version will work. I think Visual Studio and/or SSRS server includes the ReportViewer. I have both installed on my machine and did not need to install the Redistributable. You will also need to download or compile the ReportExporters dlls. You can grab the compiled version from [demo code](http://www.codeproject.com/KB/reporting-services/ReportExporters_WinForms/ReportExportersDemo.zip) in the article.

Here are few examples of what you can do with the script:

**get-alias** | ./**out-report**.ps1 "c:usersu00documentsaliases.xls" xls **get-alias** | ./**out-report**.ps1 "c:usersu00documentsaliases.pdf" pdf **get-alias** | ./**out-report**.ps1 "c:usersu00documentsaliases.jpeg" -filetype image -imagetype JPEG -height 22 -width 11 

The script makes use of a [DataTable routine](http://mow001.blogspot.com/2006/05/powershell-out-datagrid-update-and.html) by [Marc van Orsouw (//o//)](http://thepowershellguy.com/blogs/posh/default.aspx) . I often use variations of the DataTable code in many of my scripts, especially for [importing data into SQL Server tables](http://chadwickmiller.spaces.live.com/blog/cns!EA42395138308430!236.entry).

Andriy points out, not only can you generate Excel, PDF and images from the ReportViewer control, but you can also [extended SSRS to support DOC, RTF, WordprocessingML, and OOXML](http://www.codeproject.com/KB/reporting-services/report-viewer-hack.aspx). A little hacky, but interesting.

## Comments

**[John Kavanagh](#49 "2009-04-11 14:18:00"):** Excellent

**[Tim Green](#50 "2011-07-20 16:26:53"):** I've tried to use this but receive the following error: Exception calling "ExportToPdf" with "0" argument(s): "Could not load file or assembly 'Microsoft.ReportViewer.WinForms, Version=8.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies. The system cannot find the file specified." At C:Documents and SettingsgreenwtMy DocumentspsOut-Report.ps1:74 char:53 \+ 'PDF' { $content = $reportExporter.ExportToPdf <<<< () } \+ CategoryInfo : NotSpecified: (:) [], MethodInvocationException \+ FullyQualifiedErrorId : DotNetMethodException You cannot call a method on a null-valued expression. I've noticed that when I load the Winforms assembly, I'm actually loading version 9.0.0.0 - same public key value though. So is my problem that version 8.0.0.0 is hard-coded in the .dlls? Any idea how to fix it, or do I need to change the source and compile the dlls myself? [reflection.assembly]::LoadWithPartialName("Microsoft.ReportViewer.WinForms") GAC Version Location \--- ------- -------- True v2.0.50727 C:WINDOWSassemblyGAC_MSILMicrosoft.ReportViewer.WinForms9.0.0.0__b03f5f7f11d50a3aMicrosoft.ReportViewer.WinForms.dll

**[Chad Miller](#51 "2011-07-20 16:44:23"):** Funny I started re-visiting this old script for a presentation I'm giving on Saturday 7/23 and ran into the same issue. The CodeProject ReportExporters on which this scripts relies is compiled to use an older version of the Microsoft ReportViewer. I've downloaded the source and re-compiled to use 2010 SP1 ReportViewer. I'm working on turning the whole thing into a module so I then re-distribute all of the needed assemblies and functions as one package. I'll post a link soon. but if you can't wait--yes recompiling solves the issue. You'll need to re-compile reportexporters common and reportexporters WinForms.

**[Tim Green](#52 "2011-07-20 16:50:01"):** My apologies... the code actually works in regular PowerShell. I was trying to run it from the PowerGui PS command prompt which I'm guessing must be loading a later version of WinForms thus interfering. Any ideas how to fix that?!? ;) Thanks for sharing... very useful and nicely (and concisely) written.

**[Tim Green](#53 "2011-07-20 16:51:16"):** Ah... I'm looking forward to that. Perhaps that'll solve the problem with PowerGui too.

**[Chad Miller](#54 "2011-07-20 19:17:20"):** I re-compiled ReportExporters and turned it into a module here[ https://skydrive.live.com/?cid=ea42395138308430&sc=documents&uc=1&id=EA42395138308430%21113#](https://skydrive.live.com/?cid=ea42395138308430&sc=documents&uc=1&id=EA42395138308430%21113#) Unfortunately I couldn't figure how to get it to work without installing Microsoft ReportViewer. I tried installing the MS assemblies into the GAC, still no luck. I compiled ReportExporters against the [Microsoft Report Viewer 2010 SP1 Redistributable Package](http://www.microsoft.com/download/en/details.aspx?id=6610)

**[Tim Green](#55 "2011-07-21 08:27:46"):** I don't see it there Chad. Are you sure you made it public? The most recent file is dated July 4th.

**[Chad Miller](#56 "2011-07-21 09:15:05"):** hmm, I thought it was public. Unfortunately Skydrive is blocked at my work. I'll take a look at it tonight.

**[Tim Green](#57 "2011-07-21 10:24:18"):** I'm not sure if some magic occurred or if I was just blind, but I see it now. Thanks!

