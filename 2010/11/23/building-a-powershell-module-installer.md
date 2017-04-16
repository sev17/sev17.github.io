title: Building A PowerShell Module Installer
link: http://sev17.com/2010/11/23/building-a-powershell-module-installer/
author: Chad Miller
description: 
post_id: 10470
created: 2010/11/23 07:06:43
created_gmt: 2010/11/23 12:06:43
comment_status: open
post_name: building-a-powershell-module-installer
status: publish
post_type: post

# Building A PowerShell Module Installer

As part of the 2.3 build of SQLPSX I built an MSI based installer to package all 10 [SQLPSX](http://sqlpsx.codeplex.com/) modules. The installer was created with [Windows XML Installer](http://wix.sourceforge.net/) (Wix). To build an MSI using Wix you manually edit XML files and run several command-line tool to generate the necessary files. Note if you’re a Visual Studio user there’s also a plug-ins to make some of the tasks easier. Let’s take a look at the solution built without Visual Studio... 

## Getting Started

  * Download and install [Windows XML Installer ](http://wix.sourceforge.net/)
  * Wix has a learning curve, so work through the [tutorial](http://www.tramontana.co.hu/wix/index.php) to understand the concepts

## Requirements

  * Do no block scripts for execution
  * Invoke UAC
  * Remove Previous versions of the module
  * Install to the default user module directory (C:Users<myUserID>DocumentsWindowsPowerShellModules), but allow the user to select an alternative directory including system module location (C:Windowssystem32WindowsPowerShellv1.0Modules)

## Solution

Unlike a zip file in which all of the contained files inherit the blocked settings when extracted, the files contained in MSI based installer will not be blocked. So, simply by using an MSI based installer over a zip file for module distribution I’ve eliminated a support issue where new PowerShell users forget to unblock the zip file downloaded from CodePlex. The remaining requirements are solved by defining a Wix file with the proper attributes. The Wix XML file below which we'll call **modules.wxs** provides a template: 
    
    
    <?xml version="1.0" encoding="utf-8"?>
    <?include $(sys.CURRENTDIR)Config.wxi?>
    <!--
          NEVER change the UPGRADE code. ALWAYS change the Id.
          Version 2.2.3.1       Product Id was: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
          Version 2.2.3.2       Product Id was: YYYYYYYY-YYYYYYYYY-YYYY-YYYYYYYYYYYY
          Version 2.3.0.0       Product Id was: ZZZZZZZZ-ZZZZ-ZZZZ-ZZZZ-ZZZZZZZZZZZZ
    -->
    <?XML:NAMESPACE PREFIX = [default] http://schemas.microsoft.com/wix/2006/wi NS = "http://schemas.microsoft.com/wix/2006/wi" /><?XML:NAMESPACE PREFIX = [default] http://schemas.microsoft.com/wix/2006/wi NS = "http://schemas.microsoft.com/wix/2006/wi" /><?XML:NAMESPACE PREFIX = [default] http://schemas.microsoft.com/wix/2006/wi NS = "http://schemas.microsoft.com/wix/2006/wi" /><?XML:NAMESPACE PREFIX = [default] http://schemas.microsoft.com/wix/2006/wi NS = "http://schemas.microsoft.com/wix/2006/wi" /><?XML:NAMESPACE PREFIX = [default] http://schemas.microsoft.com/wix/2006/wi NS = "http://schemas.microsoft.com/wix/2006/wi" /><wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
        <product id="ZZZZZZZZ-ZZZZ-ZZZZ-ZZZZ-ZZZZZZZZZZZZ" language="1033" name="SQLPSX" version="$(var.MajorVersion).$(var.MinorVersion).$(var.MicroVersion).$(var.BuildVersion)" manufacturer="MyApp" upgradecode="AAAAAAAA-AAAA-AAAAAAAAA-AAAAAAAAAAAA">
            <package description="MyApp Installer" installprivileges="elevated" comments="MyApp Installer" installerversion="200" compressed="yes"></package>
            <upgrade id="AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA">
              <upgradeversion onlydetect="no" property="PREVIOUSFOUND" minimum="1.0.0" includeminimum="yes" maximum="$(var.MajorVersion).$(var.MinorVersion).$(var.MicroVersion).$(var.BuildVersion)" includemaximum="no"></upgradeversion>
            </upgrade>
            <installexecutesequence>
                <removeexistingproducts after="InstallInitialize"></removeexistingproducts>
            </installexecutesequence>
            <media id="1" cabinet="SQLPSXInstaller.cab" embedcab="yes"></media>
            <wixvariable id="WixUILicenseRtf" value="License.rtf"></wixvariable>
            <directory id="TARGETDIR" name="SourceDir">
                <directory id="PersonalFolder" name="PersonalFolder">
                    <directory id="WindowsPowerShell" name="WindowsPowerShell">
                        <directory id="INSTALLDIR" name="Modules">
                            <directory id="MyModule1" name="MyModule1">
                            </directory>
                            <directory id="MyModule2" name="MyMode2">
                            </directory>
                        </directory>
                    </directory>
                </directory>
            </directory>
            <property id="ARPHELPLINK" value="http://myapp.com/support"></property>
            <property id="ARPURLINFOABOUT" value="http://myapp.codeplex.com"></property>
            <feature id="Module" title="MyApp" level="1" configurabledirectory="INSTALLDIR">
                <componentgroupref id="MyModule1"></componentgroupref>
                <componentgroupref id="MyModule2"></componentgroupref>
            </feature>
            <ui></ui>
            <uiref id="WixUI_InstallDir"></uiref>
            <property id="WIXUI_INSTALLDIR" value="INSTALLDIR"></property>
        </product>
    </wix>

  In line 2 we see the use of a include file (wxi), which we'll call **Config.wxi**. The include file allows you to externalize things like variable names. Config.wxi is located in the same directory as the wxs file and looks like this: 
    
    
    <?xml version="1.0" encoding="utf-8"?>
    
        <?define MyModule1="MyModule1" ?>
        <?define MyModule2="MyModule2" ?>
        <?define MajorVersion="2" ?>
        <?define MinorVersion="3" ?>
        <?define MicroVersion="0" ?>
        <?define BuildVersion="0" ?>

In lines 4-7, I like to include comments for the product ID. This ID is a GUID which you’ll need to generate. Using PowerShell you run the following command to create a GUID: 
    
    
    ([System.Guid]::NewGuid().toString()).ToUpper()

Most of the options are self-explanatory see the [Wix manual](http://wix.sourceforge.net/manual-wix3/main.htm) for details. Of note on line 11,  InstallPrivileges="elevated"  will invoke UAC. To handle upgrades I define the product version and specify to remove previous versions on initialization (lines 12-19). A custom license agreement can be used by creating an RTF file (preferably in Wordpad as the Wordpad RTFs are smaller in size then Word), placing the file in the same directory as our wxs file, and finally setting WixUILicenseRtf to the RTF file as in line 21. Next I define the define the directory structure in lines 22 through 33. The important thing to keep in mind is the location where the files will be installed is configurable by the user. In line 22 the Wix convention is to define the outer most directory using the syntax rectory Id="TARGETDIR" Name="SourceDir". Here again, there are built-in proprties in Wix. For a complete listing of the available properties see the [Wix Property Reference](http://msdn.microsoft.com/en-us/library/aa370905%28v=VS.85%29.aspx).  In line 23 the Document or My Documents folder is specified using the builtin Wix variable PersonalFolder: Directory Id="PersonalFolder" Name="PersonalFolder" In lines 24 and 25 I create the WindowsPowerShell and Modules directory if they don’t already exist. Finally I’ll define the root directories for each module in lines 25-29. In the example wxs file I only have two modules MyModule1 and MyModule2. Lines 34 and 35 add help and about links to the installer.

## Comments

**[Mike Shepard](#191 "2010-11-26 15:04:07"):** Nice write-up.

