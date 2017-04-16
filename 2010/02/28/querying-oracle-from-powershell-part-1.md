title: Querying Oracle from Powershell Part 1
link: http://sev17.com/2010/02/28/querying-oracle-from-powershell-part-1/
author: Chad Miller
description: 
post_id: 9998
created: 2010/02/28 22:02:00
created_gmt: 2010/03/01 02:02:00
comment_status: open
post_name: querying-oracle-from-powershell-part-1
status: publish
post_type: post

# Querying Oracle from Powershell Part 1

In this two part blog post we will demonstrate how to query an Oracle database from Powershell. Before we can run queries against Oracle we need to install the Oracle client on our Windows machine. Unlike SQL Server, the drivers for connecting to Oracle are not included with the operating systems. The drivers are however freely available from Oracle. You can find the client software on the [Oracle Database Software Downloads Page](http://www.oracle.com/technology/software/products/database/index.html). 

### Downloading the Oracle Client

You’ll notice several versions of Oracle software on the download page. The software you choose will varying depending on your operating system. Generally when with dealing Oracle client software it is safe to choose the latest client version even if the Oracle database you will be connecting to is a lower version. At the time of this blog post the following versions were the latest available: 

  * **11.1.0.7.0** Windows 2008 and Windows 2008 R2
  * **11.1.0.6.0** Windows 2003
However, check [the download page](http://www.oracle.com/technology/software/products/database/index.html) and choose a later version if listed. I’ve installed both the Windows 2008 and 2003 x64 versions, but for this blog series I’m using the Windows 2003 x64 version. To complete the download 

  * Select **See All**
  * Select **Oracle Database 11g Release 1 Client (11.1.0.6.0) for Microsoft Windows (x64).** **Note: Be sure you select the Client download and not the full Oracle database software!**
_Note: When you attempt to download Oracle software you will be prompted to login to the Oracle Technology Network (OTN). If you don’t have an account you’ll need to create one—It’s free._ We’re now ready to install and configure the Oracle client software.

### Installing the Oracle Client

Many of the components included with the Oracle client are not needed. The following steps are used to perform a minimal Oracle client installation. Run setup.exe ![oracleClient1](http://images.sev17.com/oracleClient1_thumb.jpg) Click next on the Install Welcome Screen. ![oracleClient2](http://images.sev17.com/oracleClient2_thumb.jpg) Select Custom installation type and click next. ![oracleClient3](http://images.sev17.com/oracleClient3_thumb.jpg) The Oracle base directory should be off of a root drive of your choosing. I’m using C:Oracle. Change the path and ensure the name field is auto populated correctly and then click next. ![oracleClient4](http://images.sev17.com/oracleClient4_thumb.jpg) Ensure all the requirement checks succeed and click next (Note: you may receive warnings on Windows 2008 R2 when using the Windows 2008 installation software. The install will still succeed even with these warnings). ![oracleClient5](http://images.sev17.com/oracleClient5_thumb.jpg) Select SQL Plus and scroll down to select more components. ![oracleClient6](http://images.sev17.com/oracleClient6_thumb.jpg) Select Oracle Windows Interfaces and ensure the first three components are **NOT** selected. Ensure all other Windows Interface **ARE** checked and scroll down to select additional components. ![oracleClient7](http://images.sev17.com/oracleClient7_thumb.jpg) Select the Oracle Net component and click next. ![oracleClient8](http://images.sev17.com/oracleClient8_thumb.jpg) Select Install. ![oracleClient9](http://images.sev17.com/oracleClient9_thumb.jpg) Once the installation is complete the configuration utility will be launched by the installer. 

### Configuring the Oracle Client

Select next from the Oracle Net Configuration Assistant Welcome screen. ![oracleClient10](http://images.sev17.com/oracleClient10_thumb.jpg) Select Next. ![oracleClient11](http://images.sev17.com/oracleClient11_thumb.jpg) Enter the Oracle database service name. _Note: I’m using Oracle Express on Ubuntu Linux. The service name is XE, your service name may differ._ ![oracleClient12](http://images.sev17.com/oracleClient12_thumb.jpg) Select Next. ![oracleClient13](http://images.sev17.com/oracleClient13_thumb.jpg) Enter the Oracle database server host name or IP address. ![oracleClient14](http://images.sev17.com/oracleClient14_thumb.jpg) Select Next to test connectivity. ![oracleClient15](http://images.sev17.com/oracleClient15_thumb.jpg) The test will fail, you’ll need to change the login and password by selecting Change Login ![oracleClient16](http://images.sev17.com/oracleClient16_thumb.jpg) The test should succeed and if not use the error message to troubleshoot. ![oracleClient17](http://images.sev17.com/oracleClient17_thumb.jpg)

## Comments

**[Chad Miller](#109 "2010-03-20 15:43:59"):** Just how I had been doing it at work. By using OLEDB I'm able to use the same code base to connect to SQL Server, Oracle, Informix and Tandem. Most of the time I'm connecting SQL Server and use System.Data.SqlClient. I don't use Oracle very often .Is ODAC easier to install/configure?

**[Mike Schmidt](#110 "2010-03-20 13:33:59"):** Thanks Chad! Any reason you chose the Oracle Client and OLEDB provider over the Oracle Data Access Components (ODAC) which is also free. ODAC seems to give a more powerful .NET-centric interface to accessing back-end Oracle databases.

**[Mike Schmidt](#112 "2010-03-25 23:32:09"):** Welcome to your new site. > Is ODAC easier to install/configure? About the same task to install and configure. -Mike

