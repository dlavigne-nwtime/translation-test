---
title: "Poll Process"
type: archives
---

Last update: 10-Mar-2014 05:17 UTC

* * *

The poll process sends NTP packets at intervals determined by the clock discipline algorithm. The process is designed to provide a sufficient update rate to maximize accuracy while minimizing network overhead. The process is designed to operate over a poll exponent range between 3 (8 s) and 17 (36 hr). The minimum and maximum poll exponent within this range can be set using the <tt>minpoll</tt> and <tt>maxpoll</tt> options of the [<tt>server</tt>](/archives/4.2.8-series/confopt/#server-command-options) command, with default 6 (64 s) and 10 (1024 s), respectively.

The poll interval is managed by a heuristic algorithm developed over several years of experimentation. It depends on an exponentially weighted average of clock offset differences, called _clock jitter_, and a jiggle counter, which is initially set to zero. When a clock update is received and the offset exceeds the clock jitter by a factor of 4, the jiggle counter is increased by the poll exponent; otherwise, it is decreased by twice the poll exponent. If the jiggle counter is greater than an arbitrary threshold of 30, it is reset to 0 and the poll exponent is increased by 1. If the jiggle counter is less than -30, it is set to 0 and the poll exponent decreased by 1. In effect, the algorithm has a relatively slow reaction to good news, but a relatively fast reaction to bad news.

As an option of the [<tt>server</tt>](/archives/4.2.8-series/confopt/#option) command, instead of a single packet, the poll process can send a burst of several packets at 2-s intervals. This is designed to reduce the time to synchronize the clock at initial startup (<tt>iburst</tt>) and/or to reduce the phase noise at the longer poll intervals (<tt>burst</tt>). The <tt>iburst</tt> option is effective only when the server is unreachable, while the <tt>burst</tt> option is effective only when the server is reachable. The two options are independent of each other and both can be enabled at the same time.

For the <tt>iburst</tt> option the number of packets in the burst is six, which is the number normally needed to synchronize the clock; for the <tt>burst</tt> option, the number of packets in the burst is determined by the difference between the current poll exponent and the minimum poll exponent as a power of 2. For instance, with the default minimum poll exponent of 6 (64 s), only one packet is sent for every poll, while the full number of eight packets is sent at poll exponents of 9 (512 s) or more. This insures that the average headway will never exceed the minimum headway.

The burst options can result in increased load on the network if not carefully designed. Both options are affected by the provisions described on the [Rate Management and the Kiss-o'-Death Packet](/archives/4.2.8-series/rate) page. In addition, when <tt>iburst</tt> or <tt>burst</tt> are enabled, the first packet of the burst is sent, but the remaining packets sent only when the reply to the fist packet is received. If no reply has been received after a timeout set by the <tt>minpoll</tt> option, the first packet is sent again. This means that, even if a server is unreachable, the network load is no more than at the minimum poll interval.

To further reduce the network load when a server is unreachable, an unreach timer is incremented by 1 at each poll interval, but is set to 0 as each packet is received. If the timer exceeds the _unreach threshold_ set at 10, the poll exponent is incremented by 1 and the unreach timer set to 0. This continues until the poll exponent reaches the maximum set by the <tt>maxpoll</tt> option.
