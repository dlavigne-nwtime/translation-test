---
title: "hopf PCI-Bus Receiver (6039 GPS/DCF77)"
type: archives
---

Last update: 21-Oct-2010 23:44 UTC

* * *

#### Table of Contents

*   [Synopsis](/archives/drivers/driver39/#synopsis)
*   [Description](/archives/drivers/driver39/#description)
*   [Operating System Compatibility](/archives/drivers/driver39/#operating-system-compatibility)
*   [O/S System Configuration](/archives/drivers/driver39/#os-system-configuration)
*   [Fudge Factors](/archives/drivers/driver39/#fudge-factors)
*   [Questions or Comments](/archives/drivers/driver39/#questions-or-comments)

* * *

#### Synopsis

<tt>Address:  </tt> `127.127.39._X_`

<tt>Reference ID:  </tt> `.hopf.` (default), `GPS`, `DCF`

<tt>Driver ID:  </tt> `HOPF_P`

![gif](/archives/pic/fg6039.jpg)

* * *

#### Description

The **refclock_hopf_pci** driver supports the [hopf](http://www.hopf.com) PCI-bus interface 6039 GPS/DCF77.  
Additional software and information about the software drivers as well as the latest NTP driver source, executables, and documentation is maintained at [http://www.ATLSoft.de/ntp](http://www.ATLSoft.de/ntp).

* * *

#### Operating System Compatibility

The hopf clock driver has been tested on the following software and hardware platforms:  

| Platform | Operating System |
| ----- | ----- |
| i386 (PC) | Linux |
| i386 (PC) | Windows NT |
| i386 (PC) | Windows 2000 |

* * *

#### O/S System Configuration

**UNIX**

The driver attempts to open the device `/dev/hopf6039 REFID`. The device entry will be made by the installation process of the kernel module for the PCI-bus board. The driver sources belongs to the delivery equipment of the PCI-board.

**Windows NT/2000**

The driver attempts to open the device by calling the function `OpenHopfDevice()`. This function will be installed by the Device Driver for the PCI-bus board. The driver belongs to the delivery equipment of the PCI-board.

* * *

#### Fudge Factors

<tt>**refid _string_**</tt>

Specifies the driver reference identifier, **GPS** or **DCF**.

<tt>**flag1 0 | 1**</tt>

When set to 1, driver syncs even if only crystal driven.

* * *

#### Questions or Comments

[Bernd Altmeier](mailto:altmeier@atlsoft.de)[  
Ing.-B??ro f??r Software www.ATLSoft.de](http://www.ATLSoft.de)
