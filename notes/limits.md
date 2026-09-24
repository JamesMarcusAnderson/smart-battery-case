# Where the investigation stopped

Every console-exposed path toward firmware or protected memory was tried.
Every one failed closed. This is the complete list — there is no hidden
step and no omitted success.

## Readout protection: RDP level 2

```
$ sys lock
<SYS LOCK rdp=2 dbg=0 OK>
```

The main MCU reports readout protection level 2 with debug disabled.
On this class of MCU, RDP level 2 permanently disables firmware readout —
via SWD/JTAG and via any on-chip path. This is stated plainly because it is
the single fact that bounds the whole project.

## Inductive-charger memory: access denied

```
$ IND MEM READ 0xF0000000
<IND MEM 00000000 AERR>
```

Reads at `0xF0000000`, `0xF0000004`, `0xF0000008`, `0xF0000800`,
`0xF0000804`, `0xF0000808`, `0xF000080C`, `0xF0000E04`, `0xF0000E08`,
`0xF0000E0C` all returned `AERR` (access error), including after
`IND ENABLE 1`.

## Firmware / OTP operations: rejected

```
$ IND FULLDL
<IND FULLDL 1 DLERR>
$ IND RAMDL
<IND RAMDL 1 DLERR>
$ IND BURNOTP
<IND BURNOTP 1 OTP DLERR>
$ IND VEROTP
<IND VEROTP 1 OTP VERR>
```

## BMU: sealed, auth handshake: rejected

- `bmu unseal` → `ERR` (all attempts, including after I2C writes to the
  chip's control register)
- `bmu page 0/1/2` → `ERR`
- `auth run <session-nonce>` → `ERR` (every attempt)

## Bottom line

What survived: the debug console itself, the full command reference, the
I2C bus/device inventory, raw register dumps, and live BMU telemetry —
all published in this repo. What did not: any firmware image, any
unsealed/protected memory contents, any completed authentication handshake.
No bypass for any of the above is documented here, because none was found.
