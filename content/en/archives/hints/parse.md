---
title: "parse"
type: archives
---

<pre>Compilation:
	Usual thing: rm -f Config.local ; make for vanilla
			make refconf for reference clock (e. g. DCF77)

Directory contents:

	hints/PARSE	- this file

	xntpd/refclock_parse.c
			- reference clock support for DCF77/GPS in xntp
	parse/parse.c
			- Reference clock data parser framework
	parse/parse_conf.c
			- parser configuration (clock types)
	parse/clk_meinberg.c
			- Meinberg clock formats (DCF U/A 31, PZF 535, GPS166)
	parse/clk_schmid.c
			- Schmid receiver (DCF77)
	parse/clk_rawdcf.c
			- 100/200ms pulses via 50 Baud line (DCF77)
	parse/clk_dcf7000.c
			- ELV DCF7000 (DCF77)
	parse/clk_trimble.c
			- Trimble SV6 GPS receiver

			  If you want to add new clock types please check
			  with kardel@informatik.uni-erlangen.de. These files
			  implement the conversion of RS232 data streams into
			  timing information used by refclock_parse.c which is
			  mostly generic except for NTP configuration constants.

	parse/Makefile.kernel
			- *SIMPLE* makefile to build a loadable STREAMS
			  module for SunOS 4.x / SunOS 5.x systems

	parse/parsestreams.c
			- SUN Streams module (loadable) for radio clocks
			  This streams module is designed for SunOS 4.1.X.

	parse/parsesolaris.c
			- SUN Streams module (loadable) for radio clocks.
			  This streams module is designed for SunOS 5.x
			  Beware this is still new - so it might crash
			  your machine (we have seen it working, though).

	parse/parsetest.c
			- simple test program for STREAMS module. Its so simple,
			  that it doesn't even set TTY-modes, thus they got to
			  be correct on startup - works for Meinberg receivers

	parse/testdcf.c
			- test program for raw DCF77 (100/200ms pulses)
			  receivers

        include/parse.h - interface to "parse" module and more
        include/parse_conf.h
			- interface to "parse" configuration

	include/sys/parsestreams.h
			- STREAMS specific definitions

	scripts/support
			- scripts (perl & sh) for statistics and rc startup
			  the startup scripts are used in Erlangen for
			  starting the daemon on a variety of Suns and HPs
			  and for Reference Clock startup on Suns
			  These scripts may or may not be helpful to you.

Supported clocks:
	Meinberg DCF U/A 31
	Meinberg PZF535/TCXO	(Software revision PZFUERL 4.6)
	Meinberg PZF535/OCXO	(Software revision PZFUERL 4.6)
	Meinberg GPS166		(Software version for Uni-Erlangen)
	ELV DCF7000		(not recommended - casual/emergency use only)
	Conrad DCF77 receiver	(email: time@informatik.uni-erlangen.de)
	  + level converter
	TimeBrick		(email: time@informatik.uni-erlangen.de)
	Schmid Receiver Kit
	Trimble SV6 GPS receiver

Addresses:
  Meinberg Funkuhren
  Auf der Landwehr 22
  31812 Bad Pyrmont
  Germany
  Tel.: 05281/20 18
  FAX:  05281/60 81 80

  ELV Kundenservice
  Postfach 1000
  26787 Leer
  Germany
  Tel.: 0491/60 08 88

  Walter Schmidt
  Eichwisrain 14
  8634 Hombrechtikon
  Switzerland

If you have problems mail to:

	time@informatik.uni-erlangen.de

We'll help (conditions permitting)

</pre>
