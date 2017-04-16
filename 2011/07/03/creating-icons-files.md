title: Creating Icons Files
link: http://sev17.com/2011/07/03/creating-icons-files/
author: Chad Miller
description: 
post_id: 10692
created: 2011/07/03 16:57:03
created_gmt: 2011/07/03 20:57:03
comment_status: open
post_name: creating-icons-files
status: publish
post_type: post

# Creating Icons Files

While working on a PowerPack for [PowerGUI](http://powergui.org/index.jspa) I needed to create a bunch of icon files from bitmaps files so I started with a quick web search. I didn’t find any PowerShell scripts suited to the task, but did find an excellent  C# WinForm by Haresh Ambaliya: <http://code.msdn.microsoft.com/Convert-Image-file-to-Icon-c927d9f7> Although the C# app is useful, it operates on a single file rather than whole bunch of files, so I quickly turned out a PowerShell called ConvertTo-Icon which I also posted on [PoshCode](http://poshcode.org/2765): 
    
    
    function ConvertTo-Icon
    {
        [cmdletbinding()]
        param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] $Path)
    
        process{
            if ($Path -is [string])
            { $Path = get-childitem $Path }
    
            $Path | foreach {
                $image = [System.Drawing.Image]::FromFile($($_.FullName))
    
                $FilePath =  "{0}{1}.ico" -f $($_.DirectoryName), $($_.BaseName)
                $stream = [System.IO.File]::OpenWrite($FilePath)
    
                $bitmap = new-object System.Drawing.Bitmap $image
                $bitmap.SetResolution(72,72)
                $icon = [System.Drawing.Icon]::FromHandle($bitmap.GetHicon())
                $icon.Save($stream)
                $stream.Close()
            }
        }
    
     }

Using the Convertto-Icon  function against my directory of bitmap files I was to create my icon files: 
    
    
    PS D:Icons> Get-ChildItem *.bmp | ConvertTo-Icon

This is often the pattern I follow when I need to create a script, first look to see if anyone else has already done it and if not look for C# examples which easily be translated into Powershell.