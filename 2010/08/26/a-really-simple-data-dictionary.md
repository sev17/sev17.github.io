title: A Really Simple Data Dictionary
link: http://sev17.com/2010/08/26/a-really-simple-data-dictionary/
author: Chad Miller
description: 
post_id: 10438
created: 2010/08/26 10:49:44
created_gmt: 2010/08/26 14:49:44
comment_status: open
post_name: a-really-simple-data-dictionary
status: publish
post_type: post

# A Really Simple Data Dictionary

Last month I sat down with [Blain Barton](http://blogs.technet.com/b/blainbar/) and [TechNet Edge](http://edge.technet.com/) in an interview ([Blain Barton Interviews Raymond James Chad Miller on PowerShell](http://edge.technet.com/Media/Blain-Barton-Interviews-Raymond-James-Chad-Miller-on-PowerShell/)) and described a PowerShell-based solution for providing a classic centralized data dictionary of various data sources to our IT users. I'm pleased to announce I've started a CodePlex project called [Really Simple Data Dictionary](http://rsdd.codeplex.com/) or RSDD which demonstrates the coding techniques used to develop the solution. 

### What is RSDD?

[RSDD](http://rsdd.codeplex.com/) is a meta data collector for SQL Server and Oracle.  RSDD imports meta data into a central SQL Server repository from INFORMATION_SCHEMA or equivalent supported systems views. RSDD is packaged as PowerShell module, to get started you'll need PowerShell 2.0 and SQL Server instance to host the repository. Check out the [documentation](http://rsdd.codeplex.com/documentation) for setup instructions. Once you've collected the meta data, the data can easily be exposed via reports or PowerPivot ([see screenshots](http://rsdd.codeplex.com/documentation)). 

### Next Steps

  * Install RSDD (_As always: Test in non-production environment first!)_
  * Leave [feedback on the project site](http://rsdd.codeplex.com/discussions)

### Interested in Working on RSDD?

Contact [me](http://www.codeplex.com/site/users/view/cmille19) if you're interested in contributing to RSDD