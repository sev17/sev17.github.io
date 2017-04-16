title: Hello SMO (F#) World!
link: http://sev17.com/2010/01/13/hello-smo-f-world/
author: Chad Miller
description: 
post_id: 9991
created: 2010/01/13 20:47:00
created_gmt: 2010/01/14 00:47:00
comment_status: open
post_name: hello-smo-f-world
status: publish
post_type: post

# Hello SMO (F#) World!

Reading [The F# Survival Guide](http://www.ctocorner.com/fsharp/book/) has motivated me to write my version of an F# "Hello World!" utility. What I mean by that is to write something simple that I've written in other programming languages as a learning exercise. In my world of databases I use [SMO](http://msdn.microsoft.com/en-us/library/ms162169.aspx) (**pronounced smoh or S-M-O**). One of the easiest things I can do is  write some code to script out SQL Servers tables.

I'm going to use the [F#](http://msdn.microsoft.com/en-us/fsharp/default.aspx) command-style interactive console, fsi.exe that ships with F#. The only installation needed is F# and SMO version 10 that is included with SQL Server 2008 Management Studio. On my machine using the Oct 2009 CTP version the path to fsi.exe is C:Program FilesFSharp-1.9.7.8bin. To run the interactive console open a command windows and navigate to the bin directory and run fsi.exe. Once in the interactive console you can either type or paste the F# code to run. Let's take a look at the code and then I'll provide a short explaination:

#I @"C:Program FilesMicrosoft SQL Server100SDKAssemblies";; #r "Microsoft.SqlServer.Smo.dll";; #r "Microsoft.SqlServer.ConnectionInfo.dll";; open Microsoft.SqlServer.Management.Smo open Microsoft.SqlServer.Management.Common let svr = Server(@"Z002SQL2K8") let db = svr.Databases.["pubs"] for t in db.Tables do for s in t.Script() do printfn "%s" s;;

### Notes

  * The first three lines are not comments, they are used to resovle the assembly path and reference the SMO assemblies. These lines are specific to the interactive console if you're using Visual Studio you would add references as you would normally.
  * F# is case sensitive
  * Whitespace is important
  * You use a dot before brackets to access an element, which is different than other languages
  * Double semi-colons terminate a command in the interactive console
  * The @ sign is used for verbatim strings (here-strings) -- used to escape special characters.
  * The above example isn't very F#-like which favors functions and recursion over imperative looping, but this just a simple example
  * Although it may not look like it, F# is strongly typed. It uses type inference to determine type. You can explicitly type items
**EDIT Jan 24, 2010:** [Tony Davis](http://www.simple-talk.com/community/blogs/tony_davis/default.aspx) blogged about this post in his article [Life at the F# end](http://www.simple-talk.com/community/blogs/tony_davis/archive/2010/01/21/87875.aspx) providing a revised solution that is more F#-like as follows: db.Tables |> Seq.cast |> Seq.collect (fun (t:Table) -> t.Script() |> Seq.cast) |> Seq.iter (fun s -> printfn "%s" s);; You'll need to read the article for an explanation of the F# code. Tony also suggests F# as a common scripting language for both developers and administrators. My thought on the subject is that Powershell is the common scripting language for administrators, but perhaps F# may have a niche use case for administrators needing better scale--I would love to see more practical examples of F# administration scripts. Be sure to read the comments section in which I respond with my reasons for exploring F# out of a need to achieve some concurrency missing  from Powershell. Oh, and I also appologize for making someones' teeth itch with my use of imperative looping in F# ![Open-mouthed](http://shared.live.com/rzvDQW1qjIikH13dsbM42g/emoticons/smile_teeth.gif)

## Comments

**[Mike Shepard](#103 "2010-03-24 00:10:54"):** I'm always interested in looking at a new language. I hadn't ever looked at F#, but after seeing this I started reading some F# books and watched a channel 9 video and it looks very interesting. I like the idea of a .net language which has pipelines. It might be that moving from scripting (which I agree will more naturally be in powershell) to a compiled .net language might be just a bit easier in F# due to pipelines. Also...I like the new site. Never was a big fan of livespaces, and I use wordpress myself. Keep up the good work.

**[Chad Miller](#104 "2010-03-24 07:05:13"):** Thanks -- If get some time I would like to revisit the original problem I was attempting to solve with F# as described in the comment I made on Tony's blog http://www.simple-talk.com/community/blogs/tony_davis/archive/2010/01/21/87875.aspx

