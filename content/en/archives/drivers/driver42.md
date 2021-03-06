---
title: "Zyfer GPStarplus Receiver"
type: archives
---

Last update: 21-Oct-2010 23:44 UTC

* * *

#### Table of Contents

*   [Synopsis](/archives/drivers/driver42/#synopsis)
*   [Description](/archives/drivers/driver42/#description)
*   [Additional Information](/archives/drivers/driver42/#additional-information)

* * *

#### Synopsis

Address: `127.127.42._u_`  
Reference ID: <tt>GPS</tt>  
Driver ID: <tt>Zyfer GPStarplus</tt>  
Serial Port: <tt>/dev/zyfer_u_</tt>; 9600 baud, 8-bits, no parity  
Features: <tt>(none)</tt>

* * *

#### Description

This driver supports the [Zyfer GPStarplus](http://www.zyfer.com/) receiver.

The receiver has a DB15 port on the back which has input TxD and RxD lines for configuration and control, and a separate TxD line for the once-per-second timestamp.

Additionally, there are BNC connectors on the back for things like PPS and IRIG output.

* * *

#### Additional Information

[Reference Clock Drivers](/archives/4.2.8-series/refclock)
