title: Providing Online Help for Powershell Modules
link: http://sev17.com/2010/02/04/providing-online-help-for-powershell-modules/
author: Chad Miller
description: 
post_id: 9995
created: 2010/02/04 09:00:00
created_gmt: 2010/02/04 13:00:00
comment_status: open
post_name: providing-online-help-for-powershell-modules
status: publish
post_type: post

# Providing Online Help for Powershell Modules

As a finishing touch for the [SQL Server Powershell Extensions](http://sqlpsx.codeplex.com/) 2.0 Release I wanted to provide an online version of the help documentation I created from both comment-based and MAML formats. I had two requirements I need to be able to automatically convert comment-based and MAML-based help into static HTML pages and I need a free place to host the pages. The reason for the latter requirement is that I’m kind of lazy about web hosting. I don’t have my own server and I really don’t have a desire to have my own site--that’s part of the reason I blog at http://chadwickmiller.spaces.live.com. So I need to find a free static web hosting service. But, my first task is to automatically create HTML pages… 

### Generating Help HTML Pages

My favorite Powershell scripts are the ones I don’t have to write and a great place to find ready-to-use Powershell scripts is [PoshCode](http://poshcode.org/) which hosts a repository of over 1,500 scripts. A quick search of PoshCode turned up a script called [Out-Html by Vegard Hamar](http://poshcode.org/587) (whose script in turn is based on a script called Out-Wiki by Dimitry Sotnikov). The script converts help from pssnapin’s to HTML. I need to convert help for function and cmdlets within modules, so I performed [a minor edit of Out-Html](http://poshcode.org/1612) PoshCode creates a new version of a Powershell script if you modify an existing one. If you do make a useful modification, please consider sharing. This goes for original scripts also.. To use Out-Html I need to import my modules: sqlserver, repl, Agent, SQLParser and ShowMbrs. Next modify the last line of the Out-html script to filter for these modules: 
    
    
    Out-HTML ( get-command | where {($_.modulename -eq 'sqlserver' -or $_.modulename -eq 'repl' -or `
    
    
    $_.modulename -eq 'Agent' -or $_.modulename -eq 'SQLParser' -or $_.modulename -eq 'SSIS' -or $_.modulename -eq 'ShowMbrs') -and $_.commandtype -ne 'Alias'}) $outputDir

Finally, run Out-Html: ./Out-Html And viola, 126 html files are produced in a folder named help under the current directory. The HTML files are pretty clean, but do contain a stray bracket, question mark and require some manual editing. Rather tweak the Out-Html script or mess around with Powershell I can easily fix all HTML documents using my favorite text editor, Vim:

  * Select all htm files in Explorer and select edit with single Vim
  * In command mode 

args *.htm 

argdo %s/”<div>/<div>/ge | update

argdo %s/</table>}/</table>/ge | update

If only Powershell ISE could do stuff like this, I might actually use it ![Open-mouthed](http://shared.live.com/rzvDQW1qjIikH13dsbM42g/emoticons/smile_teeth.gif). One other minor edit which I’ll explain in the next section, I need to rename default.htm to index.htm and index.html to default.htm. In addition, change the new index.htm line frame src="./default.htm". Having generated 126 HTML pages, I now need to find a place to host them...

### Hosting Help HTML Pages

While searching for a place to plunk down my static we pages I found a blog post by [Charles Engelke](http://engelke.com/) that describes how to use [Google AppEngine for web hosting](http://blog.engelke.com/2008/07/30/google-appengine-for-web-hosting/) of static web pages--exactly what I’m looking for…

#### Setting Up a Google Application

To get started I had to perform a few setup tasks: 

  * [Sign up for a Google App Engine account](http://code.google.com/appengine/). As part of the sign up I picked **sqlpsx** to be the application name.
  * Install the [latest version of Python from ActiveState](http://www.activestate.com/activepython/downloads/). In order to use the static web page hosting method Python is required. Now, I don’t know Python, but fortunately you don’t need to know how to write a single line of Python code to setup static web pages.
  * Download and install [Google App Engine SDK for Python](http://code.google.com/appengine/downloads.html). The SDK provides a GUI based utility for testing and deploying  a Google application.

#### Testing Google Application

To setup a test/deployment environment on my machine. First I created a directory **C:sqlpsx** and subdirectory static i.e. **C:sqlpsxstatic.** I then moved all 126 htm files to the static directory.  Following Charles’ instructions I created an app.yaml file with the contents below and saved the file to **C:sqlpsx** application: sqlpsx version: 1 runtime: python api_version: 1 handlers: **-** url: (.***)/**