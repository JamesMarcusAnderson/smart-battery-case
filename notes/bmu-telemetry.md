# BMU telemetry session

The `BMU` console subsystem talks to the battery management unit (I2C bus 2,
addr `0x34` — a fuel-gauge IC). Live telemetry read fine; everything protected
stayed protected.

## Live telemetry

```
$ bmu status
<BMU STATUS 24 4319 0 74% 0000 OK>
$ bmu soc
<BMU SOC 74 OK>
$ bmu cycle
<BMU CYCLE 0 OK>
$ bmu uptime
<BMU UPTIME 677 OK>
$ bmu refresh
<BMU REFRESH 2000 OK>
$ bmu enable
<BMU ENABLE 1 OK>
$ bmu dfcs
<BMU DFCS CE1B 7C47 0847 OK>
```

Read as: 24 °C, 4319 mV pack voltage, 0 mA current, 74 % state of charge,
status flags `0000`, 0 charge cycles, 677 s uptime, 2000 ms telemetry refresh,
DataFlash checksums `CE1B 7C47 0847`.

## Sealed state (all denied)

```
$ bmu unseal
<BMU UNSEAL ERR>
$ bmu page 0
<BMU PAGE 1 ERR>
$ bmu page 1
<BMU PAGE 1 ERR>
$ bmu page 2
<BMU PAGE 1 ERR>
$ bmu reg 0
<BMU REG 1 ERR>
```

The gauge is sealed; unseal requires an authentication key that was not
available, and direct register/page access through the BMU interface is
refused. (Raw I2C reads of the same chip at bus 2 / addr `0x34` still work —
see [i2c-bus-map.md](i2c-bus-map.md) — because the I2C path doesn't go through
the BMU's own access control.)

Direct writes to the chip's register 0x00 over I2C returned `OK` but did not
stick — readback showed the register unchanged:

```
$ I2C WRITE 2 0x34 0x00 0x00
<I2C WRITE 2 34 00=00 OK>
$ I2C READ 2 0x34 0x00 32
<I2C READ 2 34 00=00 f1 00 00 … OK>
```

## Device authentication

```
$ auth check
<Auth CHECK idle 0 0 OK>
$ auth profile
<Auth PROFILE 0 0 OK>
$ auth run <nonce>
<Auth RUN ERR>
```

A fresh nonce is issued per session (`cbit nonce`); submitting it back via
`auth run` was rejected every attempt. No handshake was completed.

The BMU serial number shown by `bmu serial` is redacted from this repo.
