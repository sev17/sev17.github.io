title: Windows Search Enable Your Script Files
link: http://sev17.com/2010/07/10/windows-search-enable-your-script-files/
author: Chad Miller
description: 
post_id: 10388
created: 2010/07/10 11:32:49
created_gmt: 2010/07/10 15:32:49
comment_status: open
post_name: windows-search-enable-your-script-files
status: publish
post_type: post

# Windows Search Enable Your Script Files

At the July 8th [Tampa PowerShell User Group](http://powershellgroup.org/tampa.fl) meeting, Ed Wilson ([blog](http://www.scriptingguys.com/)|[twitter](http://twitter.com/scriptingguys/)) gave a presentation on “PowerShell Best Practices.” One of the tips Ed mentioned is enabling Windows Search to index the content of PowerShell ps1 files. Although I’ve used grep or in PowerShell, [Select-String](http://technet.microsoft.com/en-us/library/ee176956.aspx) to find strings within script files there are advantages to being able to search  in Windows Explorer and yes its OK to use the GUI sometimes. If like me you haven’t used Windows Search features in years, you’re probably wondering how to you set the indexing options. So I’ve provided a step-by-step guide… 

## Changing Indexing Options

In Windows 7 or Vista on the Start Menu in the Search Programs type Indexing options Select **Indexing Options** located under Control Panel ![IndexingOptions1](http://images.sev17.com/IndexingOptions1_thumb.jpg) Under Indexing Options select **Advanced** ![IndexingOptions2](http://images.sev17.com/IndexingOptions2_thumb.jpg) Next select the **File Types** tab Select the ps1 file extension and with the file extension selected, choose **Index Properties and File Content** Click **OK** and **Close** ![IndexingOptions3](http://images.sev17.com/IndexingOptions3_thumb.jpg) Window will start indexing your ps1 file immediately. You will then be able to search the content of PowerShell scripts within Windows Explorer. Here’s an example searching for the string invoke-coalesce across all PowerShell scripts: ![IndexingOptions4](http://images.sev17.com/IndexingOptions4_thumb.jpg) Although grep and select-string are more full-featured (for example the matching line and line numbers can be returned), Windows Search is fine for simple searches. You may want to enable content search for other types of script files such as .sql files also