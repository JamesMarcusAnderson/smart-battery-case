# Smart Battery Case — opened the debug console, mapped the whole surface

Apple's Smart Battery Case (iPhone XS, model A2070) hides a full interactive
debug shell behind its Lightning port. I opened it over serial, enumerated
every command it offers, scanned all five I2C buses, and pulled live battery
telemetry from the management unit — then documented exactly where the
investigation provably stopped: **readout protection is at RDP level 2, so
firmware extraction was not possible.**

This is hardware diagnostics and telemetry analysis: enumerate the surface,
measure what's measurable, and write down the wall you hit as carefully as
the ground you covered.

**Scope:** all work on my own personally-owned case in an isolated bench
setup. Read-only in outcome — the one attempted register write did not
persist (documented in [notes/bmu-telemetry.md](notes/bmu-telemetry.md)).
No third-party systems, no bypass material.

## What was done

- Opened the case's debug console over a serial connection (`screen /dev/tty.usbserial-2 115200`)
  via an Apple DCSD cable. James's documented setup also included a female
  Lightning breakout board mated to the case's male Lightning port so the
  console stayed reachable during the session.
- The console identifies itself on boot:

```
Quasar 1.8.4-QSEVT2 (DEBUG) Debug May 25 2019, 11:40:03
```

- Enumerated the complete debug command set (`help`), the `SYS` subsystem
  (`sys help`), and the parameter store (`PARAM`). Full listing in
  [notes/debug-console.md](notes/debug-console.md).
- Scanned all five I2C buses and dumped the registers of every responding device.
  Bus map and register dumps in [notes/i2c-bus-map.md](notes/i2c-bus-map.md) and
  [dumps/](dumps/).
- Queried the battery management unit (BMU) over its console interface: live
  telemetry (state of charge, voltage, temperature, cycle count, uptime) in
  [notes/bmu-telemetry.md](notes/bmu-telemetry.md).
- Attempted every console-exposed path to firmware or protected memory. All failed
  closed. Documented in [notes/limits.md](notes/limits.md).

## Key findings

| Finding | Evidence |
|---|---|
| Console banner: `Quasar 1.8.4-QSEVT2 (DEBUG)`, build May 25 2019 | boot log |
| `PARAM MODEL` → `A2070` | `param model` |
| `SYS LOCK` → `rdp=2 dbg=0`: RDP level 2, debug disabled | `sys lock` |
| I2C bus 2, addr `0x34`: BMU / fuel-gauge IC, full 256-byte register map dumped | `i2c scan 2`, `I2C READ 2 0x34 …` |
| I2C bus 3, addr `72`: device appears only while an iPhone X is attached in recovery mode; 20-byte repeating register pattern | `i2c scan 3` before/after |
| BMU live telemetry: 24 °C, 4319 mV, 0 mA, 74 % SoC, 0 cycles, 677 s uptime | `bmu status`, `bmu soc`, `bmu cycle`, `bmu uptime` |
| BMU sealed: `bmu unseal` → `ERR`; `bmu page` → `ERR` | console |
| Inductive-charger memory reads → `AERR`; firmware download/OTP ops → `DLERR`/`VERR` | `ind mem read`, `ind fulldl`, `ind burnotp` |
| Device auth handshake attempts → `ERR` | `auth run …` |

## Dead ends

- **The write that didn't stick:** an I2C write to the BMU's register `0x00`
  returned `OK` — then readback showed the register unchanged. The chip
  accepted the command and ignored it. Documented in
  [notes/bmu-telemetry.md](notes/bmu-telemetry.md); kept because the
  behavior itself is data about the chip's access control.
- **Every extraction path failed closed:** `ind mem read` → `AERR`,
  `ind fulldl` → `DLERR`, `bmu unseal` → `ERR`, `auth run` → `ERR`.
  The complete failure catalog is in [notes/limits.md](notes/limits.md) —
  every console-exposed path was tried, every one failed closed, and none
  was omitted.

## What this is NOT

- **No firmware was extracted.** RDP level 2 locks readout permanently; the console
  reports it directly and every memory-read path returns an access error.
- **No BMU unseal.** The fuel gauge stayed sealed for the entire session.
- **No register meanings are claimed.** The dumps are published as raw observed
  bytes. Any "decoded" interpretations floating around in chat logs are unverified
  speculation and are deliberately not reproduced here.
- **No bypass instructions.** This repo contains telemetry analysis only — console
  transcripts, bus scans, and register dumps. Nothing here defeats readout
  protection, unseals the BMU, or bypasses device authentication.

## Layout

```
README.md                  this file
SOURCES.md                 every claim traced to its source conversation
notes/debug-console.md     full debug command reference (from `help` / `sys help`)
notes/i2c-bus-map.md       bus scan results and device inventory
notes/bmu-telemetry.md     BMU console session (telemetry + sealed-state evidence)
notes/limits.md            every extraction path tried, and how each failed
dumps/bus2-addr34-regmap.xxd   256-byte register map, I2C bus 2 addr 0x34 (xxd)
dumps/bus3-addr72-registers.txt  register reads, I2C bus 3 addr 72
```

Identifiers (UDID, factory/manufacturing serials, BMU serial) seen in the raw
transcripts are redacted in this repo.
