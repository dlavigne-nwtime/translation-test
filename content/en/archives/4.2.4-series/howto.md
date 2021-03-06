---
title: "How to Write a Reference Clock Driver"
type: archives
---

![gif](/archives/pic/pogo4.gif)[from _Pogo_, Walt Kelly](/reflib/pictures)

You need a little magic.

Last update: 18:39 UTC Thursday, July 28, 2005

* * *

#### Table of Contents

*   [Description](/archives/4.2.4-series/howto/#description)
*   [Conventions, Fudge Factors and Flags](/archives/4.2.4-series/howto/#conventions-fudge-factors-and-flags)
*   [Files Which Need to be Changed](/archives/4.2.4-series/howto/#files-which-need-to-be-changed)
*   [Interface Routine Overview](/archives/4.2.4-series/howto/#interface-routine-overview)

* * *

#### Description

NTP reference clock support maintains the fiction that the clock is actually an ordinary peer in the NTP tradition, but operating at a synthetic stratum of zero. The entire suite of algorithms used to filter the received data, select the best clocks or peers and combine them to produce a system clock correction operate just like ordinary NTP peers. In this way, defective clocks can be detected and removed from the peer population. As no packets are exchanged with a reference clock; however, the transmit, receive and packet procedures are replaced with separate code to simulate them.

It is important to understand how the NTP clock driver interface works. The driver assumes three timescales: standard time maintained by a distant laboratory such as USNO or NIST, reference time maintained by the external radio and the system time maintained by NTP. The radio synchronizes reference time and frequency to standard time via radio, satellite or modem. As the transmission means may not always be reliable, most radios continue to provide clock updates for some time after signal loss using an internal reference oscillator. In such cases the radio may or may not reveal the time since last synchronized and/or the estimated time error.

All three timescales run _only_ in Coordinated Universal Time (UTC), 24-hour format, and are not adjusted for local timezone or standard/daylight time. The local timezone, standard/daylight indicator and year, if provided, are ignored. However, it is important to determine whether a leap second is to be inserted in the UTC timescale in the near future so NTP can insert it in the system timescale at the appropriate epoch.

The NTP clock driver synchronizes the system time and frequency to the radio via serial or parallel port, PPS signal or other means. The driver routinely checks the radio timecode string or status indicators to determine whether it is operating correctly or not. If it is, the driver decodes the radio timecode in days, hours, minutes, seconds and nanoseconds and provides these data with the NTP receive timestamp corresponding to the on-time epoch of the timecode. The driver interface computes the difference between the timecode time and NTP timestamp and saves the difference in a circular buffer for later processing. Once each poll interval, usually 64 s, the driver provides ancillary data including leap bits and last reference time to the interface. The interface processes the circular buffer using a median/trimmed mean algorithm to extract the best estimate and provides this and the ancillary data to the clock filter as with ordinary NTP peers.

The audio drivers are designed to look like a typical external radio in that the reference oscillator is derived from the audio codec oscillator and separate from the system clock oscillator. In the WWV and IRIG drivers, the codec oscillator is disciplined in frequency to the standard timescale via radio or local sources and can be assumed to have the same reliability and accuracy as an external radio. In these cases the driver continues to provide updates to the clock filter even if the WWV or IRIG signals are lost. However, the interface is provided the last reference time when the signals were received and increases the dispersion as expected with an ordinary peer.

The best way to understand how the clock drivers work is to study the <tt>ntp_refclock.c</tt> module and one of the drivers already implemented, such as <tt>refclock_wwvb.c</tt>. Routines <tt>refclock_transmit()</tt> and <tt>refclock_receive()</tt> maintain the peer variables in a state analogous to a network peer and pass received data on through the clock filters. Routines <tt>refclock_peer()</tt> and <tt>refclock_unpeer()</tt> initialize and terminate reference clock associations, should this ever be necessary. A set of utility routines is included to open serial devices, process sample data, edit input lines to extract embedded timestamps and to perform various debugging functions.

The main interface used by these routines is the <tt>refclockproc</tt> structure, which contains for most drivers the decimal equivalents of the year, day, month, hour, second and nanosecond decoded from the radio timecode. Additional information includes the receive timestamp, reference timestamp, exception reports, statistics tallies, etc. The support routines are passed a pointer to the <tt>peer</tt> structure, which is used for all peer-specific processing and contains a pointer to the <tt>refclockproc</tt> structure, which in turn contains a pointer to the unit structure, if used. For legacy purposes, a table <tt>typeunit[type][unit]</tt> contains the peer structure pointer for each configured clock type and unit. This structure should not be used for new implementations.

The reference clock interface supports auxiliary functions to support in-stream timestamping, pulse-per-second (PPS) interfacing and precision time kernel support. In most cases the drivers do not need to be aware of them, since they are detected at autoconfigure time and loaded automatically when the device is opened. These include the <tt>tty_clk</tt> STREAMS module and <tt>ppsapi</tt> PPS interface described in the [Line Disciplines and Streams Modules](/archives/4.2.4-series/ldisc) page. The <tt>tty_clk</tt> module reduces latency errors due to the operating system and serial port code in slower systems. The <tt>ppsapi</tt> PPS interface replaces the <tt>ppsclock</tt> STREAMS module and is expected to become the IETF standard cross-platform interface for PPS signals. In either case, the PPS signal can be connected via a level converter/pulse generator described in the [Pulse-per-second (PPS) Signal Interfacing](/archives/4.2.4-series/pps) page.

Radio and modem reference clocks by convention have addresses in the form <tt>127.127._t_._u_</tt>, where _t_ is the clock type and _u_ in the range 0-3 is used to distinguish multiple instances of clocks of the same type. Most clocks require a serial or parallel port or special bus peripheral. The particular device is normally specified by adding a soft link <tt>/dev/device_d_d</tt> to the particular hardware device involved, where <tt>_d_</tt> corresponds to the unit number.

By convention, reference clock drivers are named in the form <tt>refclock__xxxx_.c</tt>, where _xxxx_ is a unique string. Each driver is assigned a unique type number, long-form driver name, short-form driver name and device name. The existing assignments are in the [Reference Clock Drivers](/archives/4.2.4-series/refclock) page and its dependencies. All drivers supported by the particular hardware and operating system are automatically detected in the autoconfigure phase and conditionally compiled. They are configured when the daemon is started according to the configuration file, as described in the [Configuration Options](/archives/4.2.4-series/config) page.

The standard clock driver interface includes a set of common support routines some of which do such things as start and stop the device, open the serial port, and establish special functions such as PPS signal support. Other routines read and write data to the device and process time values. Most drivers need only a little customizing code to, for instance, transform idiosyncratic timecode formats to standard form, poll the device as necessary, and handle exception conditions. A standard interface is available for remote debugging and monitoring programs, such as <tt>ntpq</tt> and <tt>ntpdc</tt>, as well as the <tt>filegen</tt> facility, which can be used to record device status on a continuous basis.

The general organization of a typical clock driver includes a receive-interrupt routine to read a timecode from the I/O buffer and convert to internal format, generally in days, hours, minutes, seconds and fraction. Some timecode formats include provisions for leap-second warning and determine the clock hardware and software health. The interrupt routine then calls <tt>refclock_process()</tt> with these data and the timestamp captured at the on-time character of the timecode. This routine saves each sample as received in a circular buffer, which can store from a few up to 60 samples, in cases where the timecodes arrive one per second.

The <tt>refclock_transmit()</tt> routine in the interface is called by the system at intervals defined by the poll interval in the peer structure, generally 64 s. This routine in turn calls the transmit poll routine in the driver. In the intended design, the driver calls the <tt>refclock_receive()</tt> to process the offset samples that have accumulated since the last poll and produce the final offset and variance. The samples are processed by recursively discarding median outlyers until about 60 percent of samples remain, then averaging the surviving samples. When a reference clock must be explicitly polled to produce a timecode, the driver can reset the poll interval so that the poll routine is called a specified number of times at 1-s intervals.

The interface code and this documentation have been developed over some time and required not a little hard work converting old drivers, etc. Should you find success writing a driver for a new radio or modem service, please consider contributing it to the common good. Send the driver file itself and patches for the other files to Dave Mills (mills@udel.edu).

#### Conventions, Fudge Factors and Flags

Most drivers support manual or automatic calibration for systematic offset bias using values encoded in the <tt>fudge</tt> configuration command. By convention, the <tt>time1</tt> value defines the calibration offset in seconds. For those drivers that support statistics collection using the <tt>filegen</tt> utility and the <tt>clockstats</tt> file, the <tt>flag4</tt> switch enables the utility. When a PPS signal is available, a special automatic calibration facility is provided. If the <tt>flag1</tt> switch is set and the PPS signal is actively disciplining the system time, the calibration value is automatically adjusted to maintain a residual offset of zero. Should the PPS signal or the prefer peer fail, the adjustment is frozen and the remaining drivers continue to discipline the system clock with a minimum of residual error.

#### Files Which Need to be Changed

A new reference clock implementation needs to supply, in addition to the driver itself, several changes to existing files.

<dl>

<dt><tt>./include/ntp.h</tt></dt>

The reference clock type defines are used in many places. Each driver is assigned a unique type number. Unused numbers are clearly marked in the list. A unique <tt>REFCLK__xxxx_</tt> identification code should be recorded in the list opposite its assigned type number.

<dt><tt>./libntp/clocktypes.c</tt></dt>

The <tt>./libntp/clktype</tt> array is used by certain display functions. A unique short-form name of the driver should be entered together with its assigned identification code.

<dt><tt>./ntpd/ntp_control.c</tt></dt>

The <tt>clocktypes</tt> array is used for certain control message displays functions. It should be initialized with the reference clock class assigned to the driver, as per the NTP specification RFC-1305. See the <tt>./include/ntp_control.h</tt> header file for the assigned classes.

<dt><tt>./ntpd/refclock_conf.c</tt></dt>

This file contains a list of external structure definitions which are conditionally defined. A new set of entries should be installed similar to those already in the table. The <tt>refclock_conf</tt> array is a set of pointers to transfer vectors in the individual drivers. The external name of the transfer vector should be initialized in correspondence with the type number.

<dt><tt>./configure.in</tt></dt>

This is a configuration file used by the autoconfigure scheme. Add lines similar to the following:

<pre>  AC_MSG_CHECKING(FOO clock_description)
  AC_ARG_ENABLE(FOO,
      AC_HELP_STRING([--enable-FOO], [x clock_description]),
      [ntp_ok=$enableval], [ntp_ok=$ntp_eac])
  if test "$ntp_ok" = "yes"; then
      ntp_refclock=yes
      AC_DEFINE(CLOCK_FOO, 1, [Foo clock?])
  fi
  AC_MSG_RESULT($ntp_ok)
</pre>



(Note that <tt>$ntp_eac</tt> is the value from <tt>--{dis,en}able-all-clocks</tt> for non-PARSE clocks and <tt>$ntp_eacp</tt> is the value from <tt>--{dis,en}able-parse-clocks</tt> for PARSE clocks. See the documentation on the autoconf and automake tools from the GNU distributions.)

<dt><tt>./ntpd/Makefile.am</tt></dt>

This is the makefile prototype used by the autoconfigure scheme. Add the driver file name to the entries already in the <tt>ntpd_SOURCES</tt> list.

Do the following sequence of commands:

<pre>  autoreconf
  configure
</pre>



or simply run <tt>make</tt>, which will do this command sequence automatically.

</dl>

#### Interface Routine Overview

<dl>

<dt><tt>refclock_newpeer</tt> - initialize and start a reference clock</dt>

This routine allocates and initializes the interface structure which supports a reference clock in the form of an ordinary NTP peer. A driver-specific support routine completes the initialization, if used. Default peer variables which identify the clock and establish its reference ID and stratum are set here. It returns one if success and zero if the clock address is invalid or already running, insufficient resources are available or the driver declares a bum rap.

<dt><tt>refclock_unpeer</tt> - shut down a clock</dt>

This routine is used to shut down a clock and return its resources to the system.

<dt><tt>refclock_transmit</tt> - simulate the transmit procedure</dt>

This routine implements the NTP transmit procedure for a reference clock. This provides a mechanism to call the driver at the NTP poll interval, as well as provides a reachability mechanism to detect a broken radio or other madness.

<dt><tt>refclock_sample</tt> - process a pile of samples from the clock</dt>

This routine converts the timecode in the form days, hours, minutes, seconds, milliseconds/microseconds to internal timestamp format. It then calculates the difference from the receive timestamp and assembles the samples in a shift register. It implements a recursive median filter to suppress spikes in the data, as well as determine a rough dispersion estimate. A configuration constant time adjustment <tt>fudgetime1</tt> can be added to the final offset to compensate for various systematic errors. The routine returns one if success and zero if failure due to invalid timecode data or very noisy offsets.

Note that no provision is included for the year, as provided by some (but not all) radio clocks. Ordinarily, the year is implicit in the Unix file system and hardware/software clock support, so this is ordinarily not a problem. Nevertheless, the absence of the year should be considered more a bug than a feature and may be supported in future.

<dt><tt>refclock_receive</tt> - simulate the receive and packet procedures</dt>

This routine simulates the NTP receive and packet procedures for a reference clock. This provides a mechanism in which the ordinary NTP filter, selection and combining algorithms can be used to suppress misbehaving radios and to mitigate between them when more than one is available for backup.

<dt><tt>refclock_gtlin</tt> - groom next input line and extract timestamp</dt>

This routine processes the timecode received from the clock and removes the parity bit and control characters. If a timestamp is present in the timecode, as produced by the <tt>tty_clk</tt> line discipline/streams module, it returns that as the timestamp; otherwise, it returns the buffer timestamp. The routine return code is the number of characters in the line.

<dt><tt>refclock_open</tt> - open serial port for reference clock</dt>

This routine opens a serial port for I/O and sets default options. It returns the file descriptor if success and zero if failure.

<dt><tt>refclock_ioctl</tt> - set serial port control functions</dt>

This routine attempts to hide the internal, system-specific details of serial ports. It can handle POSIX (<tt>termios</tt>), SYSV (<tt>termio</tt>) and BSD (<tt>sgtty</tt>) interfaces with varying degrees of success. The routine sets up the <tt>tty_clk, chu_clk</tt> and <tt>ppsclock</tt> streams module/line discipline, if compiled in the daemon and requested in the call. The routine returns one if success and zero if failure.

<dt><tt>refclock_control</tt> - set and/or return clock values</dt>

This routine is used mainly for debugging. It returns designated values from the interface structure that can be displayed using ntpdc and the clockstat command. It can also be used to initialize configuration variables, such as <tt>fudgetimes, fudgevalues,</tt> reference ID and stratum.

<dt><tt>refclock_buginfo</tt> - return debugging info</dt>

This routine is used mainly for debugging. It returns designated values from the interface structure that can be displayed using <tt>ntpdc</tt> and the <tt>clkbug</tt> command.

* * *

![gif](/archives/pic/pogo1a.gif)
