title: Installing Visual Studio Team System 2008 Database Edition
link: http://sev17.com/2008/12/21/installing-visual-studio-team-system-2008-database-edition/
author: Chad Miller
description: 
post_id: 9933
created: 2008/12/21 20:36:00
created_gmt: 2008/12/22 00:36:00
comment_status: open
post_name: installing-visual-studio-team-system-2008-database-edition
status: publish
post_type: post

# Installing Visual Studio Team System 2008 Database Edition

I've been following Visual Studio Database Edition (VSDB) aka Data Dude, since the initial release over two years. The best overview of VSDB I've seen recently is a [presentation](http://channel9.msdn.com/pdc2008/TL45/) given by Gert Drapers at PDC2008. Until recently I've only been able to look at the product from a distance attending several presentations at various SQL Server conferences, but never able to install it because of the steep licensing costs. In October, 2008, [Microsoft announced a change in licensing terms ](http://blogs.msdn.com/gertd/archive/2008/10/02/more-team-developer-data-edition-merge-information.aspx)providing VSDB with certain MSDN licenses including my own, so I proceeded to download and install VSDB.

Since I already had Visual Studio 2008 Professional Edition installed I grabbed the "Visual Studio Team System 2008 Database Edition" from my MSDN and proceeded to install and did not encounter any issues. I then fired up Visual Studio 2008 to create a Database Project and quickly noticed the options available did not look the same as the [PDC2008 presentation](http://channel9.msdn.com/pdc2008/TL45/):

![](http://images.sev17.com/preGDR.jpg)

After reading the installation instructions (something I should have done in the first place) I noticed two things you must have [Visual Studio 2008 Service Pack 1](http://www.microsoft.com/downloads/details.aspx?FamilyID=27673c47-b3b5-4c67-bd99-84e525b5ce61&displaylang=en) installed and then you must install [Visual Studio Team System 2008 Database Edition GDR](http://www.microsoft.com/downloads/details.aspx?FamilyID=bb3ad767-5f69-4db9-b1c9-8f55759846ed&displaylang=en). I thought I had Visual Studio SP1 installed, so I tried to install the GDR and received the following error message:

**"Visual Studio Team System 2008 Database Edition GDR does not apply, or is blocked by another condition on your system. Please click the link below for more details."**

So, maybe I didn't have VS SP1 installed afterall. I then installed [Visual Studio 2008 SP1](http://www.microsoft.com/downloads/details.aspx?FamilyID=27673c47-b3b5-4c67-bd99-84e525b5ce61&displaylang=en) and did not encounter any problems. The installation however did ask for the media for VSDB, but not for VS Pro. With SP1 installed I was able to install VSDB GDR and my project options looked correct:

![](http://images.sev17.com/postGDR.jpg)

In summary to install VSDB 2008 you must:

  1. Install Visual Studio 2008 Database Edition
  2. Install [Visual Studio 2008 Service Pack 1](http://www.microsoft.com/downloads/details.aspx?FamilyID=27673c47-b3b5-4c67-bd99-84e525b5ce61&displaylang=en)
  3. Install [Visual Studio 2008 Database Edition GDR](http://www.microsoft.com/downloads/details.aspx?FamilyID=bb3ad767-5f69-4db9-b1c9-8f55759846ed&displaylang=en)
_Note: Step #1 and #2 are intechangable, but this is not how I installed VSDB. _ **Lesson Learned: Although the installation of VSDB is not intuitive, carefully reading the installation instructions would have saved me several hours of frustration! ** [This post](http://nayyeri.net/blog/visual-studio-team-system-2008-database-edition-gdr-ctp16/) helped me find a resolution.

## Comments

**[Chad Miller](#17 "2008-12-23 20:36:00"):** A colleage of mine was still having trouble installing VSDB, but then ran the Visual Studio SP1 Preperation tool which resolved the issue he was having:  
[http://www.microsoft.com/downloads/details.aspx?familyid=A494B0E0-EB07-4FF1-A21C-A4663E456D9D&displaylang=en](http://www.microsoft.com/downloads/details.aspx?familyid=A494B0E0-EB07-4FF1-A21C-A4663E456D9D&displaylang=en)

