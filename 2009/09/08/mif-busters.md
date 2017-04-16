title: MIF Busters
link: http://sev17.com/2009/09/08/mif-busters/
author: Chad Miller
description: 
post_id: 9979
created: 2009/09/08 11:32:00
created_gmt: 2009/09/08 15:32:00
comment_status: open
post_name: mif-busters
status: publish
post_type: post

# MIF Busters

[Management Information Format (MIF) files](http://technet.microsoft.com/en-us/library/cc180618.aspx) are formatted text files containing additional information about hardware and software components. The MIF format originated out of the [Desktop Management Interface Standards](http://www.dmtf.org/standards/dmi/) in 1996, but has since been displaced by newer technologies including [CIM](http://www.dmtf.org/standards/cim/). As an aside Microsoft's implementation of CIM and WBEM standards is WMI.

Why am I explaining MIF files? Even though the [DMI standard was end of life in 2005,](http://www.dmtf.org/standards/dmi/self_cert) Microsoft System Management Server (SMS)/System Center Configuration Manager (SCCM) use MIF files to extend the information collected on managed devices. If you use SMS/SCCM, MIF files are still relevant.

The format of a MIF files looks something like this:

Start Component Name = "Acme Server Location" Start Group Name    = "Acme Server Location" ID      = 1 Class   = "Acme Server Location" Start Attribute Name    = "Admin Contact" ID      = 4 Type    = String(50) Value   = "Chad Miller" End Attribute Start Attribute Name    = "Admin Phone" ID      = 5 Type    = String(40) Value   = "55500"

End Attribute End Group End Component 

The above MIF file is used to assign contact information for a server and may contain additional information including location, asset tag, or server type. If SMS/SCCM has been configured to collect MIF files as part of its inventorying process, this additional information will be added to SMS/SCCM. Depending on your organization's use of MIF files there could be alot of information contained in these files. So, I thought I'd write a PowerShell script to parse MIF files.

One of the things that struck me about the MIF file format is its resemblance to XML. Instead of closing and ending tags, there are "Start" and "End" sections. Its almost as if a MIF file is an XML file stuck in a text file body.  Converting a MIF into XML seems like a logically approach in extracting the data in a useable format. The script below uses a series of replace and regex to transform a MIF file into a XML document. Once we have an XML document, we can select the properties and obtain the parent group and component attributes:

**param** ($fileName, $computerName=$env:ComputerName) ####################### **function** ConvertTo-MIFXml { **param** ($mifFile) $mifText = gc $mifFile | #Remove illegal XML characters % { $_ -**replace** "&", "&amp;" } | % { $_ -**replace**"'", "&apos;" } | % { $_ -**replace** "<", "&lt;" } | % { $_ -**replace** ">", "&gt;" } | #Create Component attribute % { $_ -**replace** 'Start Component','<Component' } | #Create Group attribute % { $_ -**replace** 'Start Group','><Group' } | #Create Attribute attribute % { $_ -**replace** 'Start Attribute','><Attribute' } | #Create closing tags % { $_ -**replace** 'End Attribute','></Attribute>' } | % { $_ -**replace** 'End Group','</Group>' } | % { $_ -**replace** 'End Component','</Component>'} | #Remove all quotes % { $_ -**replace** '"' } | #Remove MIF comments. MIF Comments start with // % { $_ -**replace** "(s*//s*.*)" } | #Extract name/value and quote value % { $_ -**replace** "s*([^s]+)s*=s*(.+$)",'$1="$2"' } | #Replace tabs with spaces % { $_ -**replace** "t", " " } | #Replace 2 spaces with 1 % { $_ -**replace** "s{2,}", " " } #Join the array, cleanup some spacing and extra > signs **[xml]**$mifXml = **[string]**::Join(" ", $mifText) -**replace** ">s*>",">" -**replace** "s+>",">" **return** $mifXml } #ConvertTo-MIFXml #######################