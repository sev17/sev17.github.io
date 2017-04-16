title: Powershell + SMO Error Handling Tips
link: http://sev17.com/2009/06/24/powershell-smo-error-handling-tips/
author: Chad Miller
description: 
post_id: 9964
created: 2009/06/24 21:19:00
created_gmt: 2009/06/25 01:19:00
comment_status: open
post_name: powershell-smo-error-handling-tips
status: publish
post_type: post

# Powershell + SMO Error Handling Tips

[Allen White](http://sqlblog.com/blogs/allen_white/default.aspx) posted a helpful post on SMO error handling with Powershell. Actually the same concept equally applies to SMO coded in any other .NET language. SMO uses an error object's InnerException which can be several layers deep and you must traverse the nested InnerExceptions to get to the detailed error message. Related to Allen's post, [Michiel Worries](http://blogs.msdn.com/mwories/default.aspx) gives us another useful technique to get back full error messages. If you find your Powershell scripts are throwing too generic of a error message, you may find some useful messages in InnerException.

**Links:**

[MSDN: Handling SMO Exceptions](http://msdn.microsoft.com/en-us/library/ms162127.aspx)

[Allen White: Handling Errors in PowerShell](http://sqlblog.com/blogs/allen_white/archive/2009/06/08/handling-errors-in-powershell.aspx)

[PowerShell Tips & Tricks: Getting more detailed error information from PowerShell](http://blogs.msdn.com/mwories/archive/2009/06/08/powershell-tips-tricks-getting-more-detailed-error-information-from-powershell.aspx)