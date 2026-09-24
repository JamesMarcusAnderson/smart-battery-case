# Debug console command reference

Transcribed from the case's own `help` output over the serial debug console.
The console is not a Unix shell: it has no pipes, no loops, no variable
expansion — only the commands listed here, each taking at most a few arguments.

```
Quasar 1.8.4-QSEVT2 (DEBUG) Debug May 25 2019, 11:40:03
```

## Top-level commands (`help`)

```
HELP - help
VERSION - print board and firmware revision
FLAGS - execution flags
LOG - debug log flags
IO - read/write GPIO
IOM - read/write GPIO mode
IOB - read/write GPIO port
LED - enable LED
DLED - enable debug LED
DLINT - debug LED flash interval
I2C - I2C operations
CBIT - Factory Control Bit operations
ADC - read ADC
SYS - system commands
PARAM - stored parameter commands
RP - rear port operations
FP - front port operations
CHARGER - charger operations
BMU - BMU operations
TMP - Temp Sensor operations
IND - inductive power operations
PM - power manager operations
Auth - Device authentication operations
TUNNEL - set up tunnel to application
DELAY - inline delay
REPEAT - repeat commands
```

## `SYS` subsystem (`sys help`)

```
HELP - help
HRST - hard reset
RESET - reset
STOP - stop
LOCK - set Readout Protection level
VERIFY - verify App CRC
BOOT - verify Bootloader info
BPAGE - show/set boot page
CE - show/clear critical errors
CAFFEINE - show/clear keep awake
UDID - get UDID
TEST - show
```

Observed `SYS` responses (identifiers redacted):

```
$ sys udid
<SYS UDID <redacted> OK>
$ sys boot
<SYS BOOT BLV=0013 OK>
$ sys bpage
<SYS BPAGE 0 0 OK>
$ sys lock
<SYS LOCK rdp=2 dbg=0 OK>
```

`rdp=2` = readout protection level 2 (permanent); `dbg=0` = debug disabled.
This single response is what ended the firmware-extraction effort (see
[limits.md](limits.md)).

## `I2C` subsystem

```
$ i2c help
HELP - help
SCAN - scan bus for devices
READ - read registers
WRITE - write registers
RNB - read N bytes
```

Notes from the session:
- `i2c scan <bus>` probes buses 0–4. Buses 0 and 4 return `ERR`; the rest
  report detected addresses (see [i2c-bus-map.md](i2c-bus-map.md)).
- `i2c read <bus> <addr> <reg> [count]` — address and register accept decimal
  or `0x`-hex; the console echoes values as `xx=yy`.
- `REPEAT <n> <cmd…>` exists but its parser is primitive (no quoting, no
  substitution); multi-register sweeps were done one command at a time.

## `PARAM` (stored parameters, selected)

```
$ param model
<PARAM MODEL A2070 OK>
$ param colour
<PARAM COLOUR unknown(0) OK>
$ param rac
<PARAM RAC 0 OK>
$ param cabc
<PARAM CABC 0x00000000 0x00000000 0x00000000 0x00000000 OK>
$ param casc
<PARAM CASC 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000 OK>
$ param balance
<PARAM BALANCE 0 677 OK>
$ param proxystat
<PARAM PROXYSTAT Proxy Status Workaround isDisabled: 0 OK>
$ param recover
<PARAM RECOVER 0 OK>
$ param ledcal 0
<PARAM LEDCAL Green(0) 0 OK>
$ param adccal 0
<PARAM ADCCAL 0 0 0 OK>
```

Factory and manufacturing serials (`param fsn`, `param msn`) are redacted
from this repo.
