---
title: "Austron 2200A/2201A GPS Receivers"
type: archives
---

Author: David L. Mills (mills@udel.edu)  
Last update: 11-Sep-2010 05:56 UTC

* * *

#### Table of Contents

*   [Synopsis](/archives/drivers/driver10/#synopsis)
*   [Description](/archives/drivers/driver10/#description)
*   [Monitor Data](/archives/drivers/driver10/#monitor-data)
*   [Fudge Factors](/archives/drivers/driver10/#fudge-factors)
*   [Additional Information](/archives/drivers/driver10/#additional-information)

* * *

#### Synopsis

Address: 127.127.10._u_  
Reference ID: <tt>GPS</tt>  
Driver ID: <tt>GPS_AS2201</tt>  
Serial Port: <tt>/dev/gps_u_</tt>; 9600 baud, 8-bits, no parity  
Features: <tt>tty_clk</tt>

* * *

#### Description

This driver supports the Austron 2200A/2201A GPS/LORAN Synchronized Clock and Timing Receiver connected via a serial port. It supports several special features of the clock, including the Input Buffer Module, Output Buffer Module, IRIG-B Interface Module and LORAN Assist Module. It requires the RS232 Buffered Serial Interface module for communication with the driver.

For use with a single computer, the receiver can be connected directly to the receiver. For use with multiple computers, one of them is connected directly to the receiver and generates the polling messages. The other computers just listen to the receiver output directly or through a buffer amplifier. For computers that just listen, <tt>fudge flag2</tt> must be set and the <tt>ppsclock</tt> streams module configured on each of them.

This receiver is capable of a comprehensive and large volume of statistics and operational data. The specific data collection commands and attributes are embedded in the driver source code; however, the collection process can be enabled or disabled using the flag4 flag. If set, collection is enabled; if not, which is the default, it is disabled. A comprehensive suite of data reduction and summary scripts is in the ./scripts/stats directory of the ntp3 distribution.

* * *

#### Monitor Data

When enabled by the <tt>flag4</tt> fudge flag, every received timecode is written as-is to the <tt>clockstats</tt> file.

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

<dd>Set for computers that listen-only.</dd>

<dt><tt>flag3 0 | 1</tt></dt>

<dd>Not used by this driver.</dd>

<dt><tt>flag4 0 | 1</tt></dt>

<dd>Enable verbose <tt>clockstats</tt> recording if set.</dd>

</dl>

* * *

#### Additional Information

[Reference Clock Drivers](/archives/4.2.8-series/refclock)
