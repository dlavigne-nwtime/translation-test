---
title: "hopf GPS/DCF77 6021/komp for Serial Line"
type: archives
---

Last update: 21-Oct-2010 23:44 UTC

* * *

#### Table of Contents

*   [Synopsis](/archives/drivers/driver38/#synopsis)
*   [Description](/archives/drivers/driver38/#description)
*   [Operating System Compatibility](/archives/drivers/driver38/#operating-system-compatibility)
*   [O/S Serial Port Configuration](/archives/drivers/driver38/#os-serial-port-configuration)
*   [Fudge Factors](/archives/drivers/driver38/#fudge-factors)
*   [Data Format](/archives/drivers/driver38/#data-format)
*   [Questions or Comments](/archives/drivers/driver38/#questions-or-comments)

* * *

#### Synopsis

<tt>Address:  </tt> `127.127.38._X_`

<tt>Reference ID:  </tt> `.hopf.` (default), `GPS, DCF`

<tt>Driver ID:  </tt> `HOPF_S`

<tt>Serial Port:  </tt> **`/dev/hopfclock_X_`**

<tt>Serial I/O:  </tt> `9600 baud, 8-bits, 1-stop, no parity`

![gif](/archives/pic/fg6021.gif)

* * *

#### Description

The **refclock_hopf_serial** driver supports [hopf electronic receivers](http://www.hopf.com) with serial Interface kompatibel 6021.  

Additional software and information about the software drivers is available from: [http://www.ATLSoft.de/ntp](http://www.ATLSoft.de/ntp).  

Latest NTP driver source, executables and documentation is maintained at: [http://www.ATLSoft.de/ntp](http://www.ATLSoft.de/ntp).

* * *

#### Operating System Compatibility

The hopf clock driver has been tested on the following software and hardware platforms:  

| Platform | Operating System |
| ----- | ----- |
| i386 (PC) | Linux |
| i386 (PC) | Windows NT |
| i386 (PC) | Windows 2000 |

* * *

#### O/S Serial Port Configuration

The driver attempts to open the device `/dev/hopfclock_X_` where `_X_` is the NTP refclock unit number as defined by the LSB of the refclock address.  Valid refclock unit numbers are 0 - 3.

The user is expected to provide a symbolic link to an available serial port device.  This is typically performed by a command such as:

\> <tt>ln -s /dev/ttyS0 /dev/hopfclock0</tt>

Windows NT does not support symbolic links to device files.   
**COMx**: is used by the driver, based on the refclock unit number, where **unit 1** corresponds to **COM1**: and **unit 3** corresponds to **COM3**.  

* * *

## Fudge Factors

**<tt>time1 _time_</tt>**

Specifies the time offset calibration factor, in seconds and fraction, with default 0.0. Should be set to 20 milliseconds to correct serial line and operating system delays incurred in capturing time stamps from the synchronous packets.

**<tt>refid _string_</tt>**

Specifies the driver reference identifier, **GPS** _or_ **DCF**.

**<tt>flag1 0 | 1</tt>**

When set to 1, driver syncs even if only crystal driven.

* * *

#### Data Format

As specified in clock manual under pt. "Data String for NTP (Network Time Protocol)".

* * *

#### Questions or Comments

[Bernd Altmeier](mailto:altmeier@atlsoft.de)

[Ing.-B??ro f??r Software www.ATLSoft.de](http://www.ATLSoft.de)
