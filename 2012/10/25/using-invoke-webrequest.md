title: Using Invoke-WebRequest
link: http://sev17.com/2012/10/25/using-invoke-webrequest/
author: Chad Miller
description: 
post_id: 10948
created: 2012/10/25 21:23:03
created_gmt: 2012/10/26 01:23:03
comment_status: open
post_name: using-invoke-webrequest
status: publish
post_type: post

# Using Invoke-WebRequest

The website PowershellCommunity.org is moving to a new site poweshell.org and I wasn't sure the old forum posts will be brought over. Well, I kind like some of the answers I provided in the SQL Server forum and wanted to download them into a text file in case I ever need to refer to them. It looks like I finally found a use case for the Powershell V3 cmdlet, Invoke-WebRequest which sends web requests and allows you to parse the response. The following script is purpose built. Of course any parsing of HTML is going very specific to the problem at hand. Powershell and the invoke-webrequest cmdlet provides an excellent way to explore the details of a webpage so you can quickly craft a custom web-scraping script. Here's the script I came up with, again it won't be applicable to your particular task, but it shows an example of using invoke-webrequest to parse links and download specific text from various pages. 
    
    
    $outputfile = 'C:UsersPublicbinpscommunitysql.txt'
    
    1..14 | foreach {
                invoke-webrequest -Uri http://www.powershellcommunity.org/Forums/tabid/54/aff/24/afpg/$_/Default.aspx |
                    where { $_.Links | where { $_.title -and $_.title -notlike "*Page" -and $_.title -notlike "PowerShellCommunity*" } |
                        foreach { invoke-webrequest -Uri $_.href } |
                            where { $_.Links | where { $_.href -like "*printmode*" } |
                                foreach { invoke-webrequest -Uri $_.href } |
                                    foreach { $_.ParsedHtml.forms |
                                        foreach { "####################`n$($_.innerText)" | out-file $outputfile -append -encoding 'utf8' }
                                    }
                            }
                    }
            }

### Explanation

  1. On line 3, I know there's 14 pages of forums questions which I determined by looking at SQL Server forum in the browser with roughly 30 questions per page, so I'll loop through 1 to 14
  2. On lines 4-5, I loop through each page then grab the link for each question by filtering out links which don't apply. This was something I determined through trial and error looking at the object invoke-webrequest returned in the previous line
  3. On lines 6-7 I'll get the web page for each question and since a question can have multiple page answers, I'll call invoke-webrequest on the questions's print preview mode which shows the question as a single page. This was something figured out by looking at links being returned by the invoke-webrequest call and noticing a little printer icon when viewing the same page in the browser
  4. Finally on lines 8 - 10, I'll use invoke-webrequest to get the print preview page for the question, get the parsed forms data and then the inner text for the form which is appended to a text file.