---
title: "ntpd - Network Time Protocol (NTP) daemon"
type: archives
---

![gif](/archives/pic/alice47.gif)[from _The Wizard of Oz_, L. Frank Baum](/reflib/pictures)

The mushroom knows all the command line options.

* * *

#### Table of Contents

*   [Synopsis](/archives/4.1.2/ntpd/#synopsis)
*   [Description](/archives/4.1.2/ntpd/#description)
*   [How NTP Operates](/archives/4.1.2/ntpd/#how-ntp-operates)
*   [Frequency Discipline](/archives/4.1.2/ntpd/#frequency-discipline)
*   [Operating Modes](/archives/4.1.2/ntpd/#operating-modes)
*   [Poll Interval Control](/archives/4.1.2/ntpd/#poll-interval-control)
*   [The huff-n'-puff Filter](/archives/4.1.2/ntpd/#the-huff-n-puff-filter)
*   [Notes](/archives/4.1.2/ntpd/#notes)
*   [Command Line Options](/archives/4.1.2/ntpd/#command-line-options)
*   [The Configuration File](/archives/4.1.2/ntpd/#the-configuration-file)
*   [Files](/archives/4.1.2/ntpd/#files)
*   [Bugs](/archives/4.1.2/ntpd/#bugs)

* * *

#### Synopsis

<tt>ntpd [ -aAbdgLmNPqx ] [ -c _conffile_ ] [ -f _driftfile_ ] [ -g ] [ -k _keyfile_ ] [ -l _logfile_ ] [ -p _pidfile_ ] [ -r _broadcastdelay_ ] [ -s _statsdir_ ] [ -t _key_ ] [ -v _variable_ ] [ -V _variable_ ] [ -x ]</tt>

* * *  

#### Description

The <tt>ntpd</tt> program is an operating system daemon which sets and maintains the system time of day in synchronism with Internet standard time servers. It is a complete implementation of the Network Time Protocol (NTP) version 4, but also retains compatibility with version 3, as defined by RFC-1305, and version 1 and 2, as defined by RFC-1059 and RFC-1119, respectively. <tt>ntpd</tt> does most computations in 64-bit floating point arithmetic and does relatively clumsy 64-bit fixed point operations only when necessary to preserve the ultimate precision, about 232 picoseconds. While the ultimate precision is not achievable with ordinary workstations and networks of today, it may be required with future gigahertz CPU clocks and gigabit LANs.

* * *

#### How NTP Operates

The <tt>ntpd</tt> program operates by exchanging messages with one or more configured servers at designated poll intervals. When started, whether for the first or subsequent times, the program requires several exchanges from the majority of these servers so the signal processing and mitigation algorithms can accumulate and groom the data and set the clock. In order to protect the network from bursts, the initial poll interval for each server is delayed an interval randomized over a few seconds. At the default initial poll interval of 64s, several minutes can elapse before the clock is set. The initial delay to set the clock can be reduced using the <tt>iburst</tt> keyword with the <tt>server</tt> configuration command, as described on the [Configuration Options](/archives/4.1.2/confopt) page.

Most operating systems and hardware of today incorporate a time-of-year (TOY) chip to maintain the time during periods when the power is off. When the machine is booted, the chip is used to initialize the operating system time. After the machine has synchronized to a NTP server, the operating system corrects the chip from time to time. In case there is no TOY chip or for some reason its time is more than 1000s from the server time, <tt>ntpd</tt> assumes something must be terribly wrong and the only reliable action is for the operator to intervene and set the clock by hand. This causes <tt>ntpd</tt> to exit with a panic message to the system log. The <tt>-g</tt> option overrides this check and the clock will be set to the server time regardless of the chip time. However, and to protect against broken hardware, such as when the CMOS battery fails or the clock counter becomes defective, once the clock has been set, an error greater than 1000s will cause <tt>ntpd</tt> to exit anyway.

Under ordinary conditions, <tt>ntpd</tt> adjusts the clock in small steps so that the timescale is effectively continuous and without discontinuities. Under conditions of extreme network congestion, the roundtrip delay jitter can exceed three seconds and the synchronization distance, which is equal to one-half the roundtrip delay plus error budget terms, can become very large. The <tt>ntpd</tt> algorithms discard sample offsets exceeding 128 ms, unless the interval during which no sample offset is less than 128 ms exceeds 900s. The first sample after that, no matter what the offset, steps the clock to the indicated time. In practice this reduces the false alarm rate where the clock is stepped in error to a vanishingly low incidence.

As the result of this behavior, once the clock has been set, it very rarely strays more than 128 ms, even under extreme cases of network path congestion and jitter. Sometimes, in particular when <tt>ntpd</tt> is first started, the error might exceed 128 ms. This may on occasion cause the clock to be set backwards if the local clock time is more than 128 s in the future relative to the server. In some applications, this behavior may be unacceptable. If the <tt>-x</tt> option is included on the command line, the clock will never be stepped and only slew corrections will be used.

The issues should be carefully explored before deciding to use the <tt>-x</tt> option. The maximum slew rate possible is limited to 500 parts-per-million (PPM) as a consequence of the correctness principles on which the NTP protocol and algorithm design are based. As a result, the local clock can take a long time to converge to an acceptable offset, about 2,000 s for each second the clock is outside the acceptable range. During this interval the local clock will not be consistent with any other network clock and the system cannot be used for distributed applications that require correctly synchronized network time.

In spite of the above precautions, sometimes when large frequency errors are present the resulting time offsets stray outside the 128-ms range and an eventual step or slew time correction is required. If following such a correction the frequency error is so large that the first sample is outside the acceptable range, <tt>ntpd</tt> enters the same state as when the <tt>ntp.drift</tt> file is not present. The intent of this behavior is to quickly correct the frequency and restore operation to the normal tracking mode. In the most extreme cases (<tt>time.ien.it</tt> comes to mind), there may be occasional step/slew corrections and subsequent frequency corrections. It helps in these cases to use the <tt>burst</tt> keyword when configuring the server.

* * *

#### Frequency Discipline

The <tt>ntpd</tt> behavior at startup depends on whether the frequency file, usually <tt>ntp.drift</tt>, exists. This file contains the latest estimate of clock frequency error. When the <tt>ntpd</tt> is started and the file does not exist, the <tt>ntpd</tt> enters a special mode designed to quickly adapt to the particular system clock oscillator time and frequency error. This takes approximately 15 minutes, after which the time and frequency are set to nominal values and the <tt>ntpd</tt> enters normal mode, where the time and frequency are continuously tracked relative to the server. After one hour the frequency file is created and the current frequency offset written to it. When the <tt>ntpd</tt> is started and the file does exist, the <tt>ntpd</tt> frequency is initialized from the file and enters normal mode immediately. After that the current frequency offset is written to the file at hourly intervals.

* * *

#### Operating Modes

<tt>ntpd</tt> can operate in any of several modes, including symmetric active/passive, client/server broadcast/multicast and manycast, as described in the [Association Management](/archives/4.1.2/assoc) page. It normally operates continuously while monitoring for small changes in frequency and trimming the clock for the ultimate precision. However, it can operate in a one-time mode where the time is set from an external server and frequency is set from a previously recorded frequency file. A broadcast/multicast or manycast client can discover remote servers, compute server-client propagation delay correction factors and configure itself automatically. This makes it possible to deploy a fleet of workstations without specifying configuration details specific to the local environment.

By default, <tt>ntpd</tt> runs in continuous mode where each of possibly several external servers is polled at intervals determined by an intricate state machine. The state machine measures the incidental roundtrip delay jitter and oscillator frequency wander and determines the best poll interval using a heuristic algorithm. Ordinarily, and in most operating environments, the state machine will start with 64s intervals and eventually increase in steps to 1024s. A small amount of random variation is introduced in order to avoid bunching at the servers. In addition, should a server become unreachable for some time, the poll interval is increased in steps to 1024s in order to reduce network overhead.

In some cases it may not be practical for <tt>ntpd</tt> to run continuously. A common workaround has been to run the <tt>ntpdate</tt> program from a <tt>cron</tt> job at designated times. However, this program does not have the crafted signal processing, error checking and mitigation algorithms of <tt>ntpd</tt>. The <tt>-q</tt> option is intended for this purpose. Setting this option will cause <tt>ntpd</tt> to exit just after setting the clock for the first time. The procedure for initially setting the clock is the same as in continuous mode; most applications will probably want to specify the <tt>iburst</tt> keyword with the <tt>server</tt> configuration command. With this keyword a volley of messages are exchanged to groom the data and the clock is set in about 10 s. If nothing is heard after a couple of minutes, the daemon times out and exits. After a suitable period of mourning, the <tt>ntpdate</tt> program may be retired.

When kernel support is available to discipline the clock frequency, which is the case for stock Solaris, Tru64, Linux and FreeBSD, a useful feature is available to discipline the clock frequency. First, <tt>ntpd</tt> is run in continuous mode with selected servers in order to measure and record the intrinsic clock frequency offset in the frequency file. It may take some hours for the frequency and offset to settle down. Then the <tt>ntpd</tt> is stopped and run in one-time mode as required. At each startup, the frequency is read from the file and initializes the kernel frequency.

* * *

#### Poll Interval Control

This version of NTP includes an intricate state machine to reduce the network load while maintaining a quality of synchronization consistent with the observed jitter and wander. There are a number of ways to tailor the operation in order enhance accuracy by reducing the interval or to reduce network overhead by increasing it. However, the user is advised to carefully consider the consequences of changing the poll adjustment range from the default minimum of 64 s to the default maximum of 1,024 s. The default minimum can be changed with the <tt>tinker minpoll</tt> command to a value not less than 16 s. This value is used for all configured associations, unless overridden by the <tt>minpoll</tt> option on the configuration command. Note that most device drivers will not operate properly if the poll interval is less than 64 s and that the broadcast server and manycast client associations will also use the default, unless overridden.

In some cases involving dial up or toll services, it may be useful to increase the minimum interval to a few tens of minutes and maximum interval to a day or so. Under normal operation conditions, once the clock discipline loop has stabilized the interval will be increased in steps from the minimum to the maximum. However, this assumes the intrinsic clock frequency error is small enough for the discipline loop correct it. The capture range of the loop is 500 PPM at an interval of 64s decreasing by a factor of two for each doubling of interval. At a minimum of 1,024 s, for example, the capture range is only 31 PPM. If the intrinsic error is greater than this, the drift file <tt>ntp.drift</tt> will have to be specially tailored to reduce the residual error below this limit. Once this is done, the drift file is automatically updated once per hour and is available to initialize the frequency on subsequent daemon restarts.

* * *

#### The huff-n'-puff Filter

In scenarios where a considerable amount of data are to be downloaded or uploaded over telephone modems, timekeeping quality can be seriously degraded. This occurs because the differential delays on the two directions of transmission can be quite large. In many cases the apparent time errors are so large as to exceed the step threshold and a step correction can occur during and after the data transfer is in progress.

The huff-n'-puff filter is designed to correct the apparent time offset in these cases. It depends on knowledge of the propagation delay when no other traffic is present. In common scenarios this occurs during other than work hours. The filter maintains a shift register that remembers the minimum delay over the most recent interval measured usually in hours. Under conditions of severe delay, the filter corrects the apparent offset using the sign of the offset and the difference between the apparent delay and minimum delay. The name of the filter reflects the negative (huff) and positive (puff) correction, which depends on the sign of the offset.

The filter is activated by the <tt>tinker</tt> command and <tt>huffpuff</tt> keyword, as described in the [Miscellaneous Options](/archives/4.1.2/miscopt) page.

* * *

#### Notes

If NetInfo support is built into <tt>ntpd</tt>, then <tt>ntpd</tt> will attempt to read its configuration from the NetInfo if the default ntp.conf file cannot be read and no file is specified by the <tt>-c</tt> option.

Various internal <tt>ntpd</tt> variables can be displayed and configuration options altered while the <tt>ntpd</tt> is running using the <tt>[ntpq](/archives/4.1.2/ntpq)</tt> and <tt>[ntpdc](/archives/4.1.2/ntpdc)</tt> utility programs.

When <tt>ntpd</tt> starts it looks at the value of <tt>umask</tt>, and if zero <tt>ntpd</tt> will set the <tt>umask</tt> to <tt>022</tt>.

* * *

#### Command Line Options

<dt><tt>-a</tt></dt>

Enable authentication mode (default).

<dt><tt>-A</tt></dt>

Disable authentication mode.

<dt><tt>-b</tt></dt>

Synchronize using NTP broadcast messages.

<dt><tt>-c _conffile_</tt></dt>

Specify the name and path of the configuration file. (Disable netinfo?)

<dt><tt>-d</tt></dt>

Specify debugging mode. This flag may occur multiple times, with each occurrence indicating greater detail of display.

<dt><tt>-D _level_</tt></dt>

Specify debugging level directly.

<dt><tt>-f _driftfile_</tt></dt>

Specify the name and path of the drift file.

<dt><tt>-g</tt></dt>

Normally, <tt>ntpd</tt> exits if the offset exceeds the sanity limit, which is 1000 s by default. If the sanity limit is set to zero, no sanity checking is performed and any offset is acceptable. This option overrides the limit and allows the time to be set to any value without restriction; however, this can happen only once. After that, <tt>ntpd</tt> will exit if the limit is exceeded. This option can be used with the <tt>-q</tt> option. 

<dt><tt>-k _keyfile_</tt></dt>

Specify the name and path of the file containing the NTP authentication keys.

<dt><tt>-l _logfile_</tt></dt>

Specify the name and path of the log file. The default is the system log facility.

<dt><tt>-L</tt></dt>

Listen to virtual IPs. 

<dt><tt>-m</dt></tt>
    
Synchronize using NTP multicast messages on the IP multicast group address 224.0.1.1 (requires multicast kernel).

<dt><tt>-n</tt></dt>

Don't fork.

<dt><tt>-N</tt></dt>

To the extent permitted by the operating system, run the <tt>ntpd</tt> at a high priority.

<dt><tt>-p _pidfile_</tt></dt>

Specify the name and path to record the <tt>ntpd</tt>'s process ID. 

<dt><tt>-P _priority_</tt></dt>

Override the priority limit set by the operating system. Not recommended for sissies.

<dt><tt>-q</tt></dt>

Exit the <tt>ntpd</tt> just after the first time the clock is set. This behavior mimics that of the <tt>ntpdate</tt> program, which is to be retired. The <tt>-g</tt> and <tt>-x</tt> options can be used with this option. 

<dt><tt>-r _broadcastdelay_</tt></dt>

Specify the default propagation delay from the broadcast/multicast server abd this computer. This is necessary only if the delay cannot be computed automatically by the protocol.

<dt><tt>-s _statsdir_</tt></dt>

Specify the directory path for files created by the statistics facility. 

<dt><tt>-t _key_</tt></dt>

Add a key number to the trusted key list. 

<dt><tt>-v _variable_</tt></dt>

<dt><tt>-V _variable_</tt></dt>

Add a system variable listed by default.

<dt><tt>-x</tt></dt>

Normally, the time is slewed if the offset is less than the step threshold, which is 128 ms by default, and stepped if above the threshold. This option forces the time to be slewed in all cases. If the step threshold is set to zero, all offsets are stepped, regardless of value and regardless of the <tt>-x</tt> option. In general, this is not a good idea, as it bypasses the clock state machine which is designed to cope with large time and frequency errors Note: Since the slew rate is limited to 0.5 ms/s, each second of adjustment requires an amortization interval of 2000 s. Thus, an adjustment of many seconds can take hours or days to amortize. This option can be used with the <tt>-q</tt> option.

* * *

#### The Configuration File

Ordinarily, <tt>ntpd</tt> reads the <tt>ntp.conf</tt> configuration file at startup time in order to determine the synchronization sources and operating modes. It is also possible to specify a working, although limited, configuration entirely on the command line, obviating the need for a configuration file. This may be particularly useful when the local host is to be configured as a broadcast/multicast client, with all peers being determined by listening to broadcasts at run time.

Usually, the configuration file is installed in the <tt>/etc</tt> directory, but could be installed elsewhere (see the <tt>-c _conffile_</tt> command line option). The file format is similar to other Unix configuration files - comments begin with a <tt>#</tt> character and extend to the end of the line; blank lines are ignored.

Configuration commands consist of an initial keyword followed by a list of arguments, some of which may be optional, separated by whitespace. Commands may not be continued over multiple lines. Arguments may be host names, host addresses written in numeric, dotted-quad form, integers, floating point numbers (when specifying times in seconds) and text strings. Optional arguments are delimited by <tt>[ ]</tt> in the following descriptions, while alternatives are separated by <tt>|</tt>. The notation <tt>[ ... ]</tt> means an optional, indefinite repetition of the last item before the <tt>[ ... ]</tt>.

[Configuration Options](/archives/4.1.2/confopt)  
[Authentication Options](/archives/4.1.2/authopt)  
[Monitoring Options](/archives/4.1.2/monopt)  
[Access Control Options](/archives/4.1.2/accopt)   
[Reference Clock Options](/archives/4.1.2/clockopt)  
[Miscellaneous Options](/archives/4.1.2/miscopt)

* * *

#### Files

<tt>/etc/ntp.conf</tt> - the default name of the configuration file

<tt>/etc/ntp.drift</tt> - the default name of the drift file 

<tt>/etc/ntp.keys</tt> - the default name of the key file 

* * *

#### Bugs

<tt>ntpd</tt> has gotten rather fat. While not huge, it has gotten larger than might be desirable for an elevated-priority <tt>ntpd</tt> running on a workstation, particularly since many of the fancy features which consume the space were designed more with a busy primary server, rather than a high stratum workstation in mind. 