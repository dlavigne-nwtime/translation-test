---
title: "Debugging Hints for Reference Clock Drivers"
type: archives
---

The [<tt>ntpq</tt>](/archives/3-5.93e/ntpq) and [<tt>xntpdc</tt>](/archives/3-5.93e/xntpdc) utility programs can be used to debug reference clocks, either on the server itself or from another machine elsewhere in the network. The server is compiled, installed and started using the configuration file described in the [<tt>xntpd</tt>](/archives/3-5.93e/xntpd) page. The first thing to look for are error messages on the system log. If none occur, the daemon has started, opened the devices specified and waiting for peers and radios to come up. 

The next step is to be sure the RS232 messages, if used, are getting to and from the clock. The most reliable way to do this is with an RS232 tester and to look for data flashes as the driver polls the clock and/or as data arrive from the clock. Our experience is that the overwhelming fraction of problems occurring during installation are due to problems such as miswired connectors or improperly configured device links at this stage. 

If RS232 messages are getting to and from the clock, the variables of interest can be inspected using the <tt>ntpq</tt> program and various commands described on the documentation page. First, use the <tt>pe</tt> and <tt>as</tt> commands to display billboards showing the peer configuration and association IDs for all peers, including the radio clock peers. The assigned clock address should appear in the <tt>pe</tt> billboard and the association ID for it at the same relative line position in the <tt>as</tt> billboard. If things are operating correctly, after a minute or two samples should show up in the <tt>pe</tt> display line for the clock. 

Additional information is available with the <tt>rv</tt> and <tt>clockvar</tt> commands, which take as argument the association ID shown in the <tt>as</tt> billboard. The <tt>rv</tt> command with no argument shows the system variables, while the <tt>rv</tt> command with association ID argument shows the peer variables for the clock, as well as other peers of interest. The <tt>clockvar</tt> command with argument shows the peer variables specific to reference clock peers, including the clock status, device name, last received timecode (if relevant), and various event counters. In addition, a subset of the <tt>fudge</tt> parameters is included.

The <tt>xntpdc</tt> utility program can be used for detailed inspection of the clock driver status. The most useful are the <tt>clockstat</tt> and <tt>clkbug</tt> commands described in the document page. While these commands permit getting quite personal with the particular driver involved, their use is seldom necessary, unless an implementation bug shows up.

Most drivers write a message to the <tt>clockstats</tt> file as each timecode or surrogate is received from the radio clock. By convention, this is the last ASCII timecode (or ASCII gloss of a binary-coded one) received from the radio clock. This file is managed by the <tt>filegen</tt> facility described in the <tt>xntpd</tt> page and requires specific commands in the configuration file. This forms a highly useful record to discover anomalies during regular operation of the clock. The scripts included in the <tt>./scripts/stats</tt> directory can be run from a <tt>cron</tt> job to collect and summarize these data on a daily or weekly basis. The summary files have proven invaluable to detect infrequent misbehavior due to clock implementation bugs in some radios.