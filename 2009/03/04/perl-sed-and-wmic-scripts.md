title: Perl, sed and WMIC Scripts
link: http://sev17.com/2009/03/04/perl-sed-and-wmic-scripts/
author: Chad Miller
description: 
post_id: 9946
created: 2009/03/04 16:20:00
created_gmt: 2009/03/04 20:20:00
comment_status: closed
post_name: perl-sed-and-wmic-scripts
status: publish
post_type: post

# Perl, sed and WMIC Scripts

Last week I received a notification from my internet service provider they were migrating their member web pages to a new location. I had some old static web pages on my site that I hadn't updated in years. The notification requested I manually move the pages I wanted to keep to their new site. Since I hadn't hadn't updated the pages in some time and the information wasn't something I use, I figured its not worth the effort to move, but instead zipped them up and placed them [here](http://cid-ea42395138308430.skydrive.live.com/self.aspx/Public/oldHomePage.zip).

The zip file contains a few Perl, sed and WMIC scripts I used before adopting Powershell as my primary administrative scripting language in September 2007. The only scripts I still ocassionaly use are the WMIC ones.  The rest have either been replaced by Powershell scripts or I haven't had a need to do the tasks in my current job function. WMIC has some built-in format files, but you can create your own, so one of the interesting exercises I went through with WMIC was creating an XSL transformation of the XML output. I literally spent a few days figuring out XSL and applying it to the raw XML WMIC generates. Looking at the XSL now, I'm struck with how easy it is to accomplish the same thing in Powershell with no need to fiddle with XSL formatting.

The other thing I noticed is that with some of my Perl scripts and Windows bat files I would implement some basic help. For Perl it would look something like this:
    
    
    if (!defined $opts{S} or !defined $opts{m} or !defined $opts{r})
    {
       printUsage();
    }
    … Rest of Perl script …
    sub printUsage {
       print <<__Usage__;
    usage:
       cmd>perl $0 -S  -m   -r 
    __Usage__
       exit;
    

And for Windows bat files I would do something like this: 
    
    
    @echo off
    @if "%1"=="?" goto Syntax
    @if "%1"=="" goto Local
    
    ….Rest of bat file …
    
    :Syntax
    @echo Syntax: cpu [machine1 machine2 machine3 ...]
    goto :EXIT
    

For Powershell I’ll throw an error, like this: 
    
    
    param([string]$sqlserver=$(throw ‘Get-SqlServer:`$sqlserver is required.’)
    

Powershel V2 allows you to implement a more advanced help feature equivalent to the help that comes with most cmdlets. However, with V1 in looking at my old Perl and bat scripts I wonder if a usage based throw like below should be used? 
    
    
    param([string]$sqlserver=$(throw ‘Usage:Get-SqlServer -SqlServer Z002′)
    

Which throw do you prefer?