title: Counting Licenses for VMs
link: http://sev17.com/2011/06/27/counting-licenses-for-vms/
author: Chad Miller
description: 
post_id: 10682
created: 2011/06/27 21:12:57
created_gmt: 2011/06/28 01:12:57
comment_status: open
post_name: counting-licenses-for-vms
status: publish
post_type: post

# Counting Licenses for VMs

This is old news, but I’ve had to explain SQL Server VM licensing to enough people recently that I thought I’d put together a post. In June 2009, Microsoft released an update to SQL Server licensing which can significantly lower TCO for SQL Server instances running under VMs for  not only Hyper-V, but also VMWare. <http://download.microsoft.com/download/6/F/8/6F84A9FE-1E5C-44CC-87BB-C236BFCBA4DF/SQLServer2008_LicensingGuide.pdf> is a good guide for helping here. Microsoft S/W licensing like all software vendors is complex, so SQL Server virtualization licensing is best illustrated by looking at several examples: Each scenario was calculated using the formulas in the referenced document ( starting on page 25) 

### In scenario #1 -- 1 processor needed.

**A. Number of virtual processors supporting the VM - _Answer here is 2_** **B. Number of cores per physical processor (if hyper-threading is off) OR number of threads per physical processor (if** **hyper-threading is on) - _Answer here is 4 _** **C. Number of physical processors – _Answer here is 2_** ** ** **So this plugs into the formula ** ** ** **A / B = Number of licenses ( rounded up to whole number)** ** ** **2 / 4 = 1 processor license needed.**

### Scenario # 2 -- 1 processor license needed

**A. Number of virtual processors supporting the VM - _Answer here is 4_** **B. Number of cores per physical processor (if hyper-threading is off) OR number of threads per physical processor (if** **hyper-threading is on) - _Answer here is 8_** **C. Number of physical processors – _Answer here is 4_** ** ** **So this plugs into the formula ** ** ** **A / B = Number of licenses ( rounded up to whole number)** ** ** **4 / 8 = 1 processor license needed.** ** **

### Scenario # 3 --1 processor license needed

**A. Number of virtual processors supporting the VM - _Answer here is 8_** **B. Number of cores per physical processor (if hyper-threading is off) OR number of threads per physical processor (if** **hyper-threading is on) - _Answer here is 8_** **C. Number of physical processors – _Answer here is 4_** ** ** **So this plugs into the formula ** ** ** **A / B = Number of licenses ( rounded up to whole number)** ** ** **8 / 8 = 1 processor license needed.** Microsoft has provided generous virtualization licensing which means a VCPU isn’t equal to a processor and in fact is significantly less. Looking at scenario #3 allocating 8 VCPU ‘s is equal one processor license! 

## How Does SQL Server VM licensing compare to Oracle and IBM?

I think it’s important to point out I am in no way a Microsoft zealot nor I am simply bashing non-Microsoft products. I consider myself a customer of each vendor and virtualization licensing is just one of many factors which should be considered in selecting a vendor along with product capabilities, cost, and skill set of people working on it. What follows is my summary of  how Oracle and IBM address virtualization licensing provided for comparison purposes to Microsoft. These are my opinions only and you should seek the advice of  licensing experts whenever choosing a licensing strategy with any vendor. 

## Oracle

Oracle does not recognize virtualization—or at least most forms of virtualization not their own. Oracle has taken a tough stance on virtualization when compared to Microsoft and IBM as described in their own licensing documents. Oracle differentiates between what they call soft partitioning and hard partitioning. Here’s an excerpt from Oracle licensing doc <http://www.oracle.com/us/corporate/pricing/specialty-topics/index.html> in regard to virtualization. Oracle Virtual Machine (OVM) and Solaris Containers are recognized as “hard partitioning” while VMWare is consider “soft partitioning” and is not. 

> Approved hard partitioning technologies include: Dynamic System Domains (DSD) -- enabled by Dynamic Reconfiguration (DR), Solaris 10 Containers (capped Containers only), LPAR (adds DLPAR with AIX 5.2), Micro-Partitions (capped partitions only), vPar, nPar, Integrity Virtual Machine (capped partitions only), Secure Resource Partitions (capped partitions only), Static Hard Partitioning, Fujitsu’s PPAR. Oracle VM can also be used as hard partitioning technology only as described in the following document: <http://www.oracle.com/technology/tech/virtualization/pdf/ovm-hardpart.pdf>.

This is why you aren’t seeing a lot of virtualization in Oracle. As an example you can use VMWare (there may be some support issues to consider), but because VMWare is a soft partition in Oralce’s view you’d need to license the entire VMWare cluster in order to virtualize a single Oracle server or use one of the hard partitioning technologies described above. One thing I find interesting is that OVM is a Xen based hypervisor which is similar to VMWare yet one’s considered hard partitioning and the other soft partitioning. Needless to say as an Oracle customer I’m not happy with their virtualization licensing stance. As more and more Oracle customers move to virtualization I don’t think this is a sustainable strategy. It  is my belief due to customer demand at some point Oracle will need to change its stance on virtualization. 

## IBM

IBM recognizes virtualization capacity licensing, now IBM licenses on a per core basis instead of a per socket basis like SQL Server. Generally a VCPU is equal to one core, but may not be. See the licensing documentation for specific virtualization technology guidance: [IBM Virtualization Capacity License Counting Rules](http://www-01.ibm.com/software/lotus/passportadvantage/Counting_Software_licenses_using_specific_virtualization_technologies.html) Keep in mind IBM uses something called Processor Value Units or PVU’s when calculating licensing. different processor chips may have different PVU multipliers. As an example there’s a higher PVU for Nehalem or higher processors than pre-Nehalem on the Intel platform. The PVU calculations are used in determining virtualization licensing. See <http://www-01.ibm.com/software/lotus/passportadvantage/pvu_licensing_for_customers.html> for further information. 

## Summary