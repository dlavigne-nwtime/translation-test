---
title: "ntpdc - special NTP query program"
type: archives
---

![gif](/archives/pic/alice31.gif)[from _Alice's Adventures in Wonderland_, Lewis Carroll](/reflib/pictures)

This program is a big, deprecated puppy.

Last update: 31-Jan-2014 06:54 UTC

* * *

#### Table of Contents

*   [Synopsis](/archives/4.2.8-series/ntpdc/#synopsis)
*   [Description](/archives/4.2.8-series/ntpdc/#description)
*   [Command Line Options](/archives/4.2.8-series/ntpdc/#command-line-options)
*   [Interactive Commands](/archives/4.2.8-series/ntpdc/#interactive-commands)
*   [Control Message Commands](/archives/4.2.8-series/ntpdc/#control-message-commands)
*   [Runtime Configuration Requests](/archives/4.2.8-series/ntpdc/#runtime-configuration-requests)
*   [Bugs](/archives/4.2.8-series/ntpdc/#bugs)

* * *

#### Synopsis

<tt>ntpdc [ -46dilnps ] [ -c _command_ ] [ _host_ ] [ ... ]</tt>

* * *

#### Description

{{% alert color="warning" %}}
<tt>ntpdc</tt> is deprecated - please use <tt>ntpq</tt> now, as it uses a more sane interface and can provide all of the information that <tt>ntpdc</tt> used to provide.
{{% /alert %}}

<tt>ntpdc</tt> is used to query the <tt>ntpd</tt> daemon about its current state and to request changes in that state. The program may be run either in interactive mode or controlled using command line arguments. Extensive state and statistics information is available through the <tt>ntpdc</tt> interface. In addition, nearly all the configuration options which can be specified at startup using ntpd's configuration file may also be specified at run time using <tt>ntpdc</tt>.

If one or more request options are included on the command line when <tt>ntpdc</tt> is executed, each of the requests will be sent to the NTP servers running on each of the hosts given as command line arguments, or on localhost by default. If no request options are given, <tt>ntpdc</tt> will attempt to read commands from the standard input and execute these on the NTP server running on the first host given on the command line, again defaulting to localhost when no other host is specified. <tt>ntpdc</tt> will prompt for commands if the standard input is a terminal device.

<tt>ntpdc</tt> uses NTP mode 7 packets to communicate with the NTP server, and hence can be used to query any compatible server on the network which permits it. Note that since NTP is a UDP protocol this communication will be somewhat unreliable, especially over large distances in terms of network topology. <tt>ntpdc</tt> makes no attempt to retransmit requests, and will time requests out if the remote host is not heard from within a suitable timeout time.

The operation of <tt>ntpdc</tt> are specific to the particular implementation of the <tt>ntpd</tt> daemon and can be expected to work only with this and maybe some previous versions of the daemon. Requests from a remote <tt>ntpdc</tt> program which affect the state of the local server must be authenticated, which requires both the remote program and local server share a common key and key identifier.

Note that in contexts where a host name is expected, a <tt>-4</tt> qualifier preceding the host name forces DNS resolution to the IPv4 namespace, while a <tt>-6</tt> qualifier forces DNS resolution to the IPv6 namespace.

* * *

#### Command Line Options

Specifying a command line option other than <tt>-i</tt> or <tt>-n</tt> will cause the specified query (queries) to be sent to the indicated host(s) immediately. Otherwise, <tt>ntpdc</tt> will attempt to read interactive format commands from the standard input.

<dl>

<dt><tt>-4</tt></dt>

Force DNS resolution of following host names on the command line to the IPv4 namespace.

<dt><tt>-6</tt></dt>

Force DNS resolution of following host names on the command line to the IPv6 namespace.

<dt><tt>-c _command_</tt></dt>

The following argument is interpreted as an interactive format command and is added to the list of commands to be executed on the specified host(s). Multiple -c options may be given.

<dt><tt>-d</tt></dt>

Turn on debugging mode.

<dt><tt>-i</tt></dt>

Force <tt>ntpdc</tt> to operate in interactive mode. Prompts will be written to the standard output and commands read from the standard input.

<dt><tt>-l</tt></dt>

Obtain a list of peers which are known to the server(s). This switch is equivalent to <tt>-c listpeers</tt>.

<dt><tt>-n</tt></dt>

Output all host addresses in dotted-quad numeric format rather than converting to the canonical host names.

<dt><tt>-p</tt></dt>

Print a list of the peers known to the server as well as a summary of their state. This is equivalent to <tt>-c peers</tt>.

<dt><tt>-s</tt></dt>

Print a list of the peers known to the server as well as a summary of their state, but in a slightly different format than the -p switch. This is equivalent to <tt>-c dmpeers</tt>.

</dl>

* * *

#### Interactive Commands

Interactive format commands consist of a keyword followed by zero to four arguments. Only enough characters of the full keyword to uniquely identify the command need be typed. The output of a command is normally sent to the standard output, but optionally the output of individual commands may be sent to a file by appending a <tt>\<</tt>, followed by a file name, to the command line.

A number of interactive format commands are executed entirely within the <tt>ntpdc</tt> program itself and do not result in NTP mode 7 requests being sent to a server. These are described following.

<dl>

<dt><tt>? [ _command_keyword_ ]</tt>  
<tt>help [ _command_keyword_ ]</tt></dt>

A <tt>?</tt> by itself will print a list of all the command keywords known to this incarnation of <tt>ntpq</tt>. A <tt>?</tt> followed by a command keyword will print function and usage information about the command. This command is probably a better source of information about <tt>ntpq</tt> than this manual page.

<dt><tt>delay _milliseconds_</tt></dt>

Specify a time interval to be added to timestamps included in requests which require authentication. This is used to enable (unreliable) server reconfiguration over long delay network paths or between machines whose clocks are unsynchronized. Actually the server does not now require timestamps in authenticated requests, so this command may be obsolete.

<dt><tt>host _hostname_</tt></dt>

Set the host to which future queries will be sent. Hostname may be either a host name or a numeric address.

<dt><tt>hostnames [ yes | no ]</tt></dt>

If <tt>yes</tt> is specified, host names are printed in information displays. If <tt>no</tt> is specified, numeric addresses are printed instead. The default is <tt>yes</tt>, unless modified using the command line <tt>-n</tt> switch.

<dt><tt>keyid _keyid_</tt></dt>

This command allows the specification of a key number to be used to authenticate configuration requests from ntpdc to the host(s). This must correspond to a key number which the host/server has been configured to use for this purpose (server options: <tt>trustedkey</tt>, and <tt>requestkey</tt>). If authentication is not enabled on the host(s) for ntpdc commands, the command <tt>"keyid 0"</tt> should be given; otherwise the _keyid_ of the next subsequent <tt>addpeer/addserver/broadcast</tt> command will be used.

<dt><tt>quit</tt></dt>

Exit <tt>ntpdc</tt>.

<dt><tt>passwd</tt></dt>

This command prompts you to type in a password (which will not be echoed) which will be used to authenticate configuration requests. The password must correspond to the key configured for use by the NTP server for this purpose if such requests are to be successful.

<dt><tt>timeout _milliseconds_</tt></dt>

Specify a timeout period for responses to server queries. The default is about 8000 milliseconds. Note that since <tt>ntpdc</tt> retries each query once after a timeout, the total waiting time for a timeout will be twice the timeout value set.

</dl>

* * *

#### Control Message Commands

Query commands result in NTP mode 7 packets containing requests for information being sent to the server. These are read-only commands in that they make no modification of the server configuration state.

<dl>

<dt><tt>listpeers</tt></dt>

Obtains and prints a brief list of the peers for which the server is maintaining state. These should include all configured peer associations as well as those peers whose stratum is such that they are considered by the server to be possible future synchronization candidates.

<dt><tt>peers</tt></dt>

Obtains a list of peers for which the server is maintaining state, along with a summary of that state. Summary information includes the address of the remote peer, the local interface address (0.0.0.0 if a local address has yet to be determined), the stratum of the remote peer (a stratum of 16 indicates the remote peer is unsynchronized), the polling interval, in seconds, the reachability register, in octal, and the current estimated delay, offset and dispersion of the peer, all in seconds.

The character in the left margin indicates the mode this peer entry is operating in. A <tt>+</tt> denotes symmetric active, a <tt>-</tt> indicates symmetric passive, a <tt>=</tt> means the remote server is being polled in client mode, a <tt>^</tt> indicates that the server is broadcasting to this address, a <tt>~</tt> denotes that the remote peer is sending broadcasts and a <tt>*</tt> marks the peer the server is currently synchronizing to.

The contents of the host field may be one of four forms. It may be a host name, an IP address, a reference clock implementation name with its parameter or <tt>REFCLK(_implementation number_, _parameter_)</tt>. On <tt>hostnames no</tt> only IP-addresses will be displayed.

<dt><tt>dmpeers</tt></dt>

A slightly different peer summary list. Identical to the output of the <tt>peers</tt> command, except for the character in the leftmost column. Characters only appear beside peers which were included in the final stage of the clock selection algorithm. A <tt>.</tt> indicates that this peer was cast off in the falseticker detection, while a <tt>+</tt> indicates that the peer made it through. A <tt>*</tt> denotes the peer the server is currently synchronizing with.

<dt><tt>showpeer _peer_address_ [...]</tt></dt>

Shows a detailed display of the current peer variables for one or more peers. Most of these values are described in the NTP Version 2 specification.

<dt><tt>pstats _peer_address_ [...]</tt></dt>

Show per-peer statistic counters associated with the specified peer(s).

<dt><tt>clockstat _clock_peer_address_ [...]</tt></dt>

Obtain and print information concerning a peer clock. The values obtained provide information on the setting of fudge factors and other clock performance information.

<dt><tt>kerninfo</tt></dt>

Obtain and print kernel phase-lock loop operating parameters. This information is available only if the kernel has been specially modified for a precision timekeeping function.

<dt><tt>loopinfo [ oneline | multiline ]</tt></dt>

Print the values of selected loop filter variables. The loop filter is the part of NTP which deals with adjusting the local system clock. The <tt>offset</tt> is the last offset given to the loop filter by the packet processing code. The <tt>frequency</tt> is the frequency error of the local clock in parts-per-million (ppm). The <tt>time_const</tt> controls the stiffness of the phase-lock loop and thus the speed at which it can adapt to oscillator drift. The <tt>watchdog timer</tt> value is the number of seconds which have elapsed since the last sample offset was given to the loop filter. The <tt>oneline</tt> and <tt>multiline</tt> options specify the format in which this information is to be printed, with <tt>multiline</tt> as the default.

<dt><tt>sysinfo</tt></dt>

Print a variety of system state variables, i.e., state related to the local server. All except the last four lines are described in the NTP Version 3 specification, RFC-1305.

The <tt>system flags</tt> show various system flags, some of which can be set and cleared by the <tt>enable</tt> and <tt>disable</tt> configuration commands, respectively. These are the <tt>auth</tt>, <tt>bclient</tt>, <tt>monitor</tt>, <tt>pll</tt>, <tt>pps</tt> and <tt>stats</tt> flags. See the <tt>ntpd</tt> documentation for the meaning of these flags. There are two additional flags which are read only, the <tt>kernel_pll</tt> and <tt>kernel_pps</tt>. These flags indicate the synchronization status when the precision time kernel modifications are in use. The <tt>kernel_pll</tt> indicates that the local clock is being disciplined by the kernel, while the kernel_pps indicates the kernel discipline is provided by the PPS signal.

Note that some directives, like <tt>enable pps</tt>, are only supported on certain versions of <tt>ntpd</tt>.

The <tt>stability</tt> is the residual frequency error remaining after the system frequency correction is applied and is intended for maintenance and debugging. In most architectures, this value will initially decrease from as high as 500 ppm to a nominal value in the range .01 to 0.1 ppm. If it remains high for some time after starting the daemon, something may be wrong with the local clock, or the value of the kernel variable <tt>tick</tt> may be incorrect.

The <tt>broadcastdelay</tt> shows the default broadcast delay, as set by the <tt>broadcastdelay</tt> configuration command.

The <tt>authdelay</tt> shows the default authentication delay, as set by the <tt>authdelay</tt> configuration command.

<dt><tt>sysstats</tt></dt>

Print statistics counters maintained in the protocol module.

<dt><tt>memstats</tt></dt>

Print statistics counters related to memory allocation code.

<dt><tt>iostats</tt></dt>

Print statistics counters maintained in the input-output module.

<dt><tt>timerstats</tt></dt>

Print statistics counters maintained in the timer/event queue support code.

<dt><tt>reslist</tt></dt>

Obtain and print the server's restriction list. This list is (usually) printed in sorted order and may help to understand how the restrictions are applied.

<dt><tt>ifstats</tt></dt>

List interface statistics for interfaces used by ntpd for network communication.

<dt><tt>ifreload</tt></dt>

Force rescan of current system interfaces. Outputs interface statistics for interfaces that could possibly change. Marks unchanged interfaces with `.`, added interfaces with `+` and deleted interfaces with `-`.
<dt><tt>monlist [ _version_ ]</tt></dt>

Obtain and print traffic counts collected and maintained by the monitor facility. The version number should not normally need to be specified. At most, 600 entries are displayed by <tt>monlist</tt>. To display the entire MRU list, use the <tt>ntpq</tt> program's [<tt>mrulist</tt>](/archives/4.2.8-series/ntpq/#mrulist) command.

<dt><tt>clkbug _clock_peer_address_ [...]</tt></dt>

Obtain debugging information for a reference clock driver. This information is provided only by some clock drivers and is mostly undecodable without a copy of the driver source in hand.

</dl>

* * *

#### Runtime Configuration Requests

All requests which cause state changes in the server are authenticated by the server using a configured NTP key (the facility can also be disabled by the server by not configuring a key). The key number and the corresponding key must also be made known to <tt>ntpdc</tt>. This can be done using the keyid and passwd commands, the latter of which will prompt at the terminal for a password to use as the encryption key. You will also be prompted automatically for both the key number and password the first time a command which would result in an authenticated request to the server is given. Authentication not only provides verification that the requester has permission to make such changes, but also gives an extra degree of protection again transmission errors.

Authenticated requests always include a timestamp in the packet data, which is included in the computation of the authentication code. This timestamp is compared by the server to its receive time stamp. If they differ by more than a small amount the request is rejected. This is done for two reasons. First, it makes simple replay attacks on the server, by someone who might be able to overhear traffic on your LAN, much more difficult. Second, it makes it more difficult to request configuration changes to your server from topologically remote hosts. While the reconfiguration facility will work well with a server on the local host, and may work adequately between time-synchronized hosts on the same LAN, it will work very poorly for more distant hosts. As such, if reasonable passwords are chosen, care is taken in the distribution and protection of keys and appropriate source address restrictions are applied, the run time reconfiguration facility should provide an adequate level of security.

The following commands all make authenticated requests.

<dl>

<dt><tt>addpeer _peer_address_ [ _keyid_ ] [ _version_ ] [ <tt>minpoll# | prefer | minpoll _N_ | <tt>maxpoll</tt> _N_ [...] ]</tt></tt></dt>

<dt><tt>addpeer _peer_address_ [ <tt>prefer | minpoll _N_ | <tt>maxpoll</tt> _N_ | <tt>keyid</tt> _N_ | <tt>version</tt> _N_ [...] ]</tt></tt></dt>

Add a configured peer association at the given address and operating in symmetric active mode. Note that an existing association with the same peer may be deleted when this command is executed, or may simply be converted to conform to the new configuration, as appropriate. If the <tt>keyid</tt> is nonzero, all outgoing packets to the remote server will have an authentication field attached encrypted with this key. If the value is 0 (or not given) no authentication will be done. If ntpdc's key number has not yet been set (_e.g.,_ by the keyid command), it will be set to this value. The <tt>version#</tt> can be 1 through 4 and defaults to 3. The remaining options are either a numeric value for <tt>minpoll</tt> or literals <tt>prefer</tt>, <tt>burst</tt>, <tt>minpoll</tt> _N_, <tt>keyid</tt> _N_, <tt>version</tt> _N_, or <tt>maxpoll</tt> _N_ (where _N_ is a numeric value), and have the action as specified in the <tt>peer</tt> configuration file command of ntpd. See the [Server Options](/archives/4.2.8-series/confopt) page for further information. Each flag (or its absence) replaces the previous setting. The <tt>prefer</tt> keyword indicates a preferred peer (and thus will be used primarily for clock synchronisation if possible). The preferred peer also determines the validity of the PPS signal - if the preferred peer is suitable for synchronisation so is the PPS signal. The <tt>dynamic</tt> keyword allows association configuration even when no suitable network interface is found at configuration time. The dynamic interface update mechanism may complete the configuration when new interfaces appear (e.g. WLAN/PPP interfaces) at a later time and thus render the association operable.

<dt><tt>addserver _peer_address_ [ _address_ [ _keyid_ ] [ _version_ ] [ minpoll | prefer | iburst | burst | minpoll _N_ | maxpoll _N_ [...] ] prefer | iburst | burst | minpoll _N_ | maxpoll _N_ | keyid _N_ | version _N_ [...] ]</tt></dt>

Identical to the addpeer command, except that the operating mode is client.

<dt><tt>broadcast _peer_address_ [ _keyid_ ] [ _version_ ] [ _prefer_ ]</tt></dt>

Identical to the addpeer command, except that the operating mode is broadcast. In this case a valid non-zero key identifier and key are required. The <tt>peer_address</tt> parameter can be the broadcast address of the local network or a multicast group address assigned to NTP. If a multicast address, a multicast-capable kernel is required.

<dt><tt>unconfig _peer_address_ [...]</tt></dt>

This command causes the configured bit to be removed from the specified peer(s). In many cases this will cause the peer association to be deleted. When appropriate, however, the association may persist in an unconfigured mode if the remote peer is willing to continue on in this fashion.

<dt><tt>fudge _peer_address_ [ _time1_ ] [ _time2_ ] [ _stratum_ ] [ _refid_ ]</tt></dt>

This command provides a way to set certain data for a reference clock. See the source listing for further information.

<dt><tt>enable [ auth | bclient | calibrate | kernel | monitor | ntp | pps | stats]</tt>  
<tt>disable [ auth | bclient | calibrate | kernel | monitor | ntp | pps | stats]</tt></dt>

These commands operate in the same way as the <tt>enable</tt> and <tt>disable</tt> configuration file commands of <tt>ntpd</tt>. See the [Miscellaneous Options](/archives/4.2.8-series/miscopt) page for further information.

<dt><tt>restrict _address mask flag_ [ _flag_ ]</tt></dt>

This command operates in the same way as the <tt>restrict</tt> configuration file commands of <tt>ntpd</tt>.

<dt><tt>unrestrict _address mask flag_ [ _flag_ ]</tt></dt>

Unrestrict the matching entry from the restrict list.

<dt><tt>delrestrict _address mask [ ntpport ]_</tt></dt>

Delete the matching entry from the restrict list.

<dt><tt>readkeys</tt></dt>

Causes the current set of authentication keys to be purged and a new set to be obtained by rereading the keys file (which must have been specified in the <tt>ntpd</tt> configuration file). This allows encryption keys to be changed without restarting the server.

<dt><tt>trustedkey _keyid_ [...]</tt></dt>

<dt><tt>untrustedkey _keyid_ [...]</tt></dt>

These commands operate in the same way as the <tt>trustedkey</tt> and <tt>untrustedkey</tt> configuration file commands of <tt>ntpd</tt>.

<dt><tt>authinfo</tt></dt>

Returns information concerning the authentication module, including known keys and counts of encryptions and decryptions which have been done.

<dt><tt>traps</tt></dt>

Display the traps set in the server. See the source listing for further information.

<dt><tt>addtrap [ _address_ [ _port_ ] [ _interface_ ]</tt></dt>

Set a trap for asynchronous messages. See the source listing for further information.

<dt><tt>clrtrap [ _address_ [ _port_ ] [ _interface_]</tt></dt>

Clear a trap for asynchronous messages. See the source listing for further information.

<dt><tt>reset</tt></dt>

Clear the statistics counters in various modules of the server. See the source listing for further information.

</dl>

* * *

#### Bugs

<tt>ntpdc</tt> is a crude hack. Much of the information it shows is deadly boring and could only be loved by its implementer. The program was designed so that new (and temporary) features were easy to hack in, at great expense to the program's ease of use. Despite this, the program is occasionally useful.
