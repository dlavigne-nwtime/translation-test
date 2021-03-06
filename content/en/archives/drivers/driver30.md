---
title: "Motorola Oncore GPS Receiver"
type: archives
---

Last update: 21-Oct-2010 23:44 UTC

* * *

#### Table of Contents

*   [Synopsis](/archives/drivers/driver30/#synopsis)
*   [Description](/archives/drivers/driver30/#description)
*   [Monitor Data](/archives/drivers/driver30/#monitor-data)
*   [Fudge Factors](/archives/drivers/driver30/#fudge-factors)
*   [Additional Information](/archives/drivers/driver30/#additional-information)

* * *

#### Synopsis

Address: 127.127.30._u_  
Reference ID: <tt>GPS</tt>  
Driver ID: ONCORE  
Serial Port: <tt>/dev/oncore.serial.</tt>_u_;  9600 baud, 8-bits, no parity.  
PPS Port: <tt>/dev/oncore.pps.</tt>_u_;  <tt>PPS_CAPTUREASSERT</tt> required,  <tt>PPS_OFFSETASSERT</tt> supported.  
Configuration File: <tt>ntp.oncore</tt>, or <tt>ntp.oncore.</tt>_u_, or <tt>ntp.oncore</tt>_u_, in <tt>/etc/ntp</tt> or <tt>/etc</tt>.

* * *

#### Description

This driver supports most models of the Motorola Oncore GPS receivers (Basic, PVT6, VP, UT, UT+, GT, GT+, SL, M12, M12+T), as long as they support the _Motorola Binary Protocol_.

The interesting versions of the Oncore are the VP, the UT+, the "Remote" which is a prepackaged UT+, and the M12 Timing. The VP is no longer available new, and the UT, GT, and SL are at end-of-life. The Motorola evaluation kit can be recommended. It interfaces to a PC straightaway, using the serial (DCD) or parallel port for PPS input and packs the receiver in a nice and sturdy box. Less expensive interface kits are available from [TAPR](http://www.tapr.org) and [Synergy](http://www.synergy-gps.com).  

<center>UT+ oncore</center>

![gif](/archives/pic/oncore_utplusbig.gif)

<center>Evaluation kit</center>

![gif](/archives/pic/oncore_evalbig.gif)

<center>Oncore Remote</center>

![gif](/archives/pic/oncore_remoteant.jpg)

The driver requires a standard <tt>PPS</tt> interface for the pulse-per-second output from the receiver. The serial data stream alone does not provide precision time stamps (0-50msec variance, according to the manual), whereas the PPS output is precise down to 50 nsec (1 sigma) for the VP/UT models and 25 nsec for the M12 Timing. If you do not have the PPS signal available, then you should probably be using the NMEA driver rather than the Oncore driver. Most of these are available on-line

The driver will use the "position hold" mode with user provided coordinates, the receivers built-in site-survey, or a similar algorithm implemented in this driver to determine the antenna position.

* * *

#### Monitor Data

The driver always puts a lot of useful information on the clockstats file, and when run with debugging can be quite chatty on stdout. When first starting to use the driver you should definitely review the information written to the clockstats file to verify that the driver is running correctly.

In addition, on platforms supporting Shared Memory, all of the messages received from the Oncore receiver are made available in shared memory for use by other programs. See the [Oncore-SHMEM](/archives/drivers/oncore-shmem) manual page for information on how to use this option. For either debugging or using the SHMEM option, an Oncore Reference Manual for the specific receiver in use will be required.

* * *

#### Fudge Factors

<dl>

<dt><tt>time1 _time_</tt></dt>

<dd>Specifies the time offset calibration factor, in seconds and fraction, with default 0.0.</dd>

<dt><tt>time2 _time_</tt></dt>

<dd>Not used by this driver.</dd>

<dt><tt>stratum _number_</tt></dt>

<dd>Specifies the driver stratum, in decimal from 0 to 15, with default 0.</dd>

<dt><tt>refid _string_</tt></dt>

<dd>Specifies the driver reference identifier, an ASCII string from one to four characters, with default <tt>GPS</tt>.</dd>

<dt><tt>flag1 0 | 1</tt></dt>

<dd>Not used by this driver.</dd>

<dt><tt>flag2 0 | 1</tt></dt>

<dd>Not used by this driver.</dd>

<dt><tt>flag3 0 | 1</tt></dt>

<dd>Not used by this driver.</dd>

<dt><tt>flag4 0 | 1</tt></dt>

<dd>Not used by this driver.</dd>

</dl>

* * *

#### Additional Information

The driver was initially developed on FreeBSD, and has since been tested on Linux, SunOS, and Solaris.

**Configuration**

There is a driver specific configuration file <tt>ntp.oncore</tt> (or <tt>ntp.oncore.</tt>_u_ or <tt>ntp.oncore</tt>_u_ if you must distinguish between more than one Oncore receiver) that contains information on the startup mode, the location of the GPS receiver, an offset of the PPS signal from zero, and the cable delay. The offset shifts the PPS signal to avoid interrupt pileups `on' the second, and adjusts the timestamp accordingly. See the driver source for information on this file. The default with no file is: no delay, no offset, and a site survey is done to get the location of the gps receiver.

The following three options can be set in the driver specific configuration file only if the driver is using the PPSAPI. The edge of the PPS signal that is `on-time' can be set with the keywords [ASSERT/CLEAR] and the word HARDPPS will cause the PPS signal to control the kernel PLL.

**Performance**

Really good. With the VP/UT+, the generated PPS pulse is referenced to UTC(GPS) with better than 50 nsec (1 sigma) accuracy. The limiting factor will be the timebase of the computer and the precision with which you can timestamp the rising flank of the PPS signal. Using FreeBSD, a FPGA based Timecounter/PPS interface, and an ovenized quartz oscillator, that performance has been reproduced. 

[//]: # (24/5/2021 DL: goes to apology saying contents lost in disk crash)
[//]: # (For more details on this aspect: Sub-Microsecond timekeeping under FreeBSD http://phk.freebsd.dk/rover.html )
