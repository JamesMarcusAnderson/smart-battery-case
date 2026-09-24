# I2C bus map

From `i2c scan <bus>` on the debug console. Five buses exist; 0 and 4 error out.

```
$ i2c scan 0
<I2C SCAN ERR>
$ i2c scan 1
detect at 12 e0 e2 e8
<I2C SCAN 1: 12 e0 e2 e8 OK>
$ i2c scan 2
detect at 34
<I2C SCAN 2: 34 OK>
$ i2c scan 3
detect at
<I2C SCAN 3: OK>
$ i2c scan 4
<I2C SCAN ERR>
```

## Bus 1 — devices at 12, e0, e2, e8

Register 0x00, 32-byte reads. Mostly unresponsive (`0xFF` fill):

```
$ I2C READ 1 0x12 0x00 32
<I2C READ 1 12 00=ff ff … (32 bytes) OK>
$ I2C READ 1 0xE0 0x00 32
<I2C READ 1 e0 00=19 ff ff … (32 bytes) OK>
$ I2C READ 1 0xE2 0x00 32
<I2C READ 1 e2 00=19 ff ff … (32 bytes) OK>
$ I2C READ 1 0xE8 0x00 32
<I2C READ 1 e8 00=18 ff ff … (32 bytes) OK>
```

Devices `e0`/`e2` return a `0x19` ID byte, `e8` returns `0x18`, then `0xFF`.
No meaning is claimed for these bytes — recorded as observed.

## Bus 2 — device at 0x34 (BMU / fuel gauge)

The battery management unit. Full 256-byte register map dumped in 32-byte
pages (`I2C READ 2 0x34 <offset> 32`, offsets `0x00`–`0xFA`); the assembled
binary is in [../dumps/bus2-addr34-regmap.xxd](../dumps/bus2-addr34-regmap.xxd).

Shape of the map: populated header region (`0x00`–`0x2F`), a second populated
window around `0x50`–`0x7F` (repeating the header pattern), fill regions
(`0x90`: `0x0A` × 48, `0xA0`: `0x75` × 48), and zeros elsewhere. Raw bytes
only — no register semantics are claimed.

## Bus 3 — device at 72 (appears with iPhone attached)

Bus 3 scans empty on its own. After connecting an iPhone X in recovery mode,
a device appears at address 72:

```
$ i2c scan 3
detect at 72
<I2C SCAN 3: 72 OK>
```

Single-byte register reads show a 20-byte repeating pattern (registers
0–59 dumped; values repeat every 20 registers):

```
reg: 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f 10 11 12 13
val: 04 fd 40 c0 00 7c 76 10 00 00 05 93 55 01 01 01 00 ff 00 00
```

Full read log in [../dumps/bus3-addr72-registers.txt](../dumps/bus3-addr72-registers.txt).
The pattern repeats identically at registers 20–39 and 40–59, so only the
first 20 unique bytes are tabulated here.
