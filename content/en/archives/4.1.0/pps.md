---
title: "Pulse-Per-Second (PPS) Signal Interfacing"
type: archives
---

![gif](/archives/pic/alice32.gif)[from _Alice's Adventures in Wonderland_, Lewis Carroll](/reflib/pictures)

Alice is trying to find the PPS signal connector.

* * *

Some radio clocks and related timekeeping gear have a pulse-per-second (PPS) signal that can be used to discipline the local clock oscillator to a high degree of precision, typically to the order less than 10 <font face="Symbol">m</font>s in time and 0.01 parts-per-million (PPM) in frequency. The PPS signal can be connected in either of two ways: via the data carrier detector (DCD) pin of a serial port or via the acknowledge (ACK) pin of a parallel port, depending on the hardware and operating system. Connection via a serial port may require signal conversion and regeneration to RS232 levels, which can be done using a circuit such as described in the [Gadget Box PPS Level Converter and CHU Modem](/archives/4.1.0/gadget) page. Note that NTP no longer supports connection via the data leads of a serial port.

Both the serial and parallel port connection require operating system support, which is available in only a few operating systems, including Linux, FreeBSD and latest Solaris beginning with 2.7. Support on an experimental basis is available for several older systems, including SunOS, Digital Ultrix and HP-UX, and in current Digital Tru64 (Alpha). The PPS application program interface defined in RFC-2783 (PPSAPI) is the only interface currently supported. Older PPS interfaces based on the <tt>ppsclock</tt> and <tt>tty_clk</tt> streams modules are no longer supported. As the PPSAPI is expected to become an IETF cross-platform standard, it should be used by new applications.

The PPSAPI inerface requires a <tt>/usr/include/sys/ppstime.h</tt> header file. This file is included in Linux and FreeBSD distributions, but not in other distributions or standard workstation products at this time. Header files for other systems, including Solaris, can be found in the <tt>nanokernel.tar.gz</tt> distribution, which can be found via the Collaboration Resources link at www.ntp.org. The top level directory contains a number of subdirectories for each architecture, including Solaris. The <tt>ppstime.h</tt> file for each architecture can be found in the subdirectory of the same name.

In the preferred mode of operation, PPS signals are processed by the [PPS Clock Discipline](/archives/drivers/driver22) driver and other clock drivers which might be involved need not know or care about them. In some cases where there is no other driver, time might be obtained from remote NTP servers via the network and local PPS signals, for instance from a calibrated cesium oscillator, used to stabilize the frequency and remove network jitter. Note that the <tt>pps</tt> configuration command has been obsoleted by this driver.

The PPS driver operates in conjunction with a preferred peer, as described in the [Mitigation Rules and the <tt>prefer</tt> Keyword](/archives/4.1.0/prefer) page. One of the drivers described in the [Reference Clock Drivers](/archives/4.1.0/refclock) page or another NTP server furnishes the coarse timing and disambiguates the seconds numbering of the PPS signal itself. The NTP daemon mitigates between the clock driver or NTP server and the PPS driver as described in that page in order to provide the most accurate time, while respecting the various types of equipment failures that could happen.

Some Unix system kernels support a PPS signal directly, as described in the [A Kernel Model for Precision Timekeeping](/archives/4.1.0/kern) page. Specifically, the PPS driver can be used to direct the PPS signal to the kernel for use as a discipline source for both time and frequency. The presence of the kernel support is automatically detected during the NTP build process and supporting code automatically compiled. Note that the PPS driver does not normally enable the PPS kernel code, since performance is generally better without it. However, this code can be enabled by a driver fudge flag if necessary.

Some configurations may include multiple radio clocks with individual PPS outputs. In some PPSAPI designs multiple PPS signals can be connected to multiple instances of the PPS driver. In such cases the NTP mitigation and grooming algorithms operate with all the radio timecodes and PPS signals to develop the highest degree of redundancy and survivability.