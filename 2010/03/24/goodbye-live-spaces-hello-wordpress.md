title: Goodbye Live Spaces, Hello WordPress
link: http://sev17.com/2010/03/24/goodbye-live-spaces-hello-wordpress/
author: Chad Miller
description: 
post_id: 10126
created: 2010/03/24 20:11:41
created_gmt: 2010/03/25 00:11:41
comment_status: open
post_name: goodbye-live-spaces-hello-wordpress
status: publish
post_type: post

# Goodbye Live Spaces, Hello WordPress

After blogging on Live Spaces for a 517 days, I’ve made the difficult decision to move to my own domain here to [Sev17.com](/). As an aside, the meaning of Sev17 is described in the [About section.](http://sev17.com/about/) All of the content and comments have been migrated from [my old blog](http://chadwickmiller.spaces.live.com/) to this site.  Moving to my own site has been surprisingly easy and I must say I'm really digging the WordPress platform. Having gone through the exercise of moving from Live Spaces and migrating the content I thought I would take this opporunity to describe the steps: 

### Exporting Live Spaces Content

Unfortunately Live Spaces does not provide a native way to export content and comments, however there is an automated Python script called [Live Space Mover](http://b2.broom9.com/?page_id=519) that worked very well for me. All 85 of my posts including comments were succussefully exported. The Python script works by parsing the HTML from Live Spaces to generate a WordPress export XML file. There are a few setup tasks to prepare for exporting your Live Space data as well as detailed instructions on the entire process described on the Live Space Mover site. I followed instructions and won't rehash them here.  A few notes on my experience are listed below : 

  * Although the instructions reference 2.5, I'm using Python 2.6 from [ActiveState](http://www.activestate.com/).
  * Don't worry I don't know how to write Python scripts either, but fortunately you don't need to.
  * One of the components used in Live Space Mover is another Python script called [Beautiful Soup](http://www.crummy.com/software/BeautifulSoup/), I made the mistake of grabbing a later version than referenced in the Live Space Mover instructions and had issues generating the XML document. So, I then grabbed the 3.06 version which worked correctly.
  * I hit the date format issue described in the Live Space Mover instructions, but after specifying the Python script date parameters it worked fine
  * Per the instructions I changed my Live Spaces theme to "Journey"
  * The Python 2.5 bug referenced in the Live Spacer Mover instructions has been fixed in 2.6.
And with that a WordPress eXtended RSS (WXR) file is generated which includes all posts, comments,and categories from my Live Spaces blog. 

### Importing and Tidying Up

Importing to my WordPress blog was simple, I just selected import from wp-admin site. After I imported I then spent a couple of hours verifying each blog post and updating any post links with links to previous posts to now point to sev17. As side effect of going through each post [pingbacks](http://en.wikipedia.org/wiki/Pingback) were created for any blogs I referenced. This was a pleasant surprise as Live Spaces doesn't do pingbacks and even when I tried to do them manually the ugly URLs Live Spaces creates (<http://chadwickmiller.spaces.live.com/blog/cns!EA42395138308430!201.entry>) were often grabbed by other blogging platforms. 

### What's left?

I've moved over everything, the only issue I have is with images that still point to Live Spaces. It would be nice to move these to Amazon S3 storage, but I'm not going to worry about them for now. 

### Credits

I owe a big thanks to [Brent Ozar](http://www.brentozar.com/) ([@BrentO](http://twitter.com/BrentO)) for rescuing me from Live Spaces and helping setup this blog. Brent also patiently answered my questions on WordPress. Brent has a [great two part blog series on getting started with blogging](http://www.brentozar.com/archive/2008/12/how-to-start-a-technical-blog-part-2-wordpress/) that I wish I would have seen before creating a Live Space blog. If you're just getting started with blogging, don't make the same mistake I did--Read Brent's posts and if you have questions, ask. I love it when you can find a script that someone has taken the time to share with community and provide detailed instructions. My thanks to Wei Wei, the creator of Live Spaces Mover. It saved me hours if not days of time. BTW-- I did make  a small donation to the Live Spaces Mover project. The  only reason I bring this topic up is that many people may not be aware of  what is the right thing to do with open source software. My feeling is that if I get some good use out of it and they are seeking funding, then I'll put a few bucks in via Paypal. This reminds me there are a few other open source software packages I need to support...

## Comments

**[Hrvoje Piasevoli](#120 "2010-03-26 06:24:34"):** Congratulations for the move. I'm trying to convince my brother to do the same, so I'm referring him to this article. Good luck and looking forward to reading on the new site. Oh, one other note - maybe including one of those iphone-friendy wordpress themes would make your new site even better. Cheers, Hrvoje

**[Chad Miller](#121 "2010-03-26 15:21:21"):** Thanks -- I just wish I would have moved sooner. Per your advise I added the WPTouch plugin.

**[Volcano](#122 "2010-06-26 15:11:07"):** I too imported from live spaces to wordpress. All posts were successful but none of the comments are imported (though I followed every step). Can't figure out why :( :(

**[Tomislav Piasevoli](#123 "2010-07-07 18:27:44"):** My brother's persistence payed off in the end. I moved in one afternoon! The script was outdated so I had to fix it. Don't worry, I've never coded in Python either so you can fix it too if necessary, to paraphrase Chad and correct him. I've sent it also to the author and he thanked me but hasn't uploaded it on his site at the time of writing this. If you're interested, my version extracts dates, comments, persons, ... well, pretty much everything.

**[Heather](#124 "2010-08-11 13:06:55"):** I have been desperatly trying to find a way to move my blog for months. Exhausted entire weekends. Only to come up with no dice. Thing is...I know NOTHING about programming or computers at all. I have been following your instructions and I have dowloaded all the programs, but now what?? How do I get them to start extracting?? Any help would be very appreciated!!

**[Heather](#125 "2010-08-11 13:55:29"):** Also, do you think 5 years of content will be too much to import?? Thanks, again :)

**[Chad Miller](#126 "2010-08-11 20:59:08"):** @Heather I can't say I'm an expert on moving blog as I've only done so once. I simply followed the instructions listed on http://b2.broom9.com/?page_id=519 with a couple of minor modifications as noted in this blog post. I would say the volume of content you have may be an issue. I did not have near as many posts on Live Spaces as you. Since I already have the Live Spaces Mover scripts setup on my machine I ran a quick test to see if I could generate the XML file using this command: C:Usersu00binspaces>python live-space-mover.py -s http://cheersrayne.spaces.live.com/ The script bombed on blog entry "Comfortable Corner - Write A Thon." This is probably not a good sign as it will take someone more skilled than me to make the script work against your blog. You've probably already considered this, but one other option many bloggers take is to start anew and not convert their old posts to their new site. Having struggled with Live Spaces for a couple of years I sympathize with your problem. Sorry I couldn't be of more help and good luck moving to WordPress.

