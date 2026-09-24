# Sources

Every factual claim in this repo traces to James Anderson's own terminal output
or his own words (REQUEST fragments) in his DeepSeek chat archive
(`~/workspace/deepseek-archive/conversations.json`, exported 2026-07-16).
Assistant responses in those chats are never treated as verified fact.

## Conversation IDs

- `bf67067c-6ab7-40d6-98c1-070bc3e15a3a` — 2025-08-18, "Analysis of I2C Bus Scan Results"
- `7ddf854f-8791-4ff6-a8f4-77240039f118` — 2025-08-20, "Diagnosing I2C Bus Error with Python Script"
- `5ad6dbc6-d588-41bf-8d8d-95c1e0094379` — 2025-08-20, "Debugging Device Data Extraction Commands"
- `70245911-cda7-4673-b68c-3cc70ddeec16` — 2025-08-20, "Hacking Apple Battery Case Firmware Guide"
- `3ece828d-c1fe-4bc1-8dd6-beb51a12f482` — 2025-08-20, "Apple Smart Battery Case Debugging and Hacking Guide"

## Claim → evidence

| Claim | Evidence |
|---|---|
| Device is an Apple Smart Battery Case (iPhone XS), model A2070 | `70245911` REQ: "its a apple iphone xs smart battery case"; `5ad6dbc6` terminal: `<PARAM MODEL A2070 OK>`; `bf67067c` REQ: "lol webcam? its a apple smart battery case" |
| Debug console banner `Quasar 1.8.4-QSEVT2 (DEBUG) Debug May 25 2019, 11:40:03` | `bf67067c`, `5ad6dbc6` terminal output |
| Serial access via `screen /dev/tty.usbserial-2 115200`; Apple DCSD cable; Lightning breakout board on the case's male Lightning port | `7ddf854f` REQ: "i use screen /dev/tty.usbserial-2 115200"; `3ece828d` REQ: "keep using this apple dcsd cable" + "i got a female lightning breakout board … connect it to the male lightning port of the apple smart battery case" |
| Full top-level `help` command list (24 commands) | `bf67067c` terminal (`help` output) |
| `sys help` subcommand list (HRST, RESET, STOP, LOCK, VERIFY, BOOT, BPAGE, CE, CAFFEINE, UDID, TEST) | `70245911` / `5ad6dbc6` terminal |
| `sys lock` → `<SYS LOCK rdp=2 dbg=0 OK>` (RDP level 2, debug off) | `70245911`, `5ad6dbc6` terminal |
| `sys boot` → `BLV=0013`; `sys bpage` → `0 0` | `70245911` / `5ad6dbc6` terminal |
| I2C bus scan results (bus 0/4 ERR; bus 1: 12 e0 e2 e8; bus 2: 34; bus 3: empty, then 72 with iPhone X in recovery) | `5ad6dbc6` terminal (`i2c scan 0-4`); `bf67067c` REQ: "i get a new register 72 on bus 3 after connecting my iphone x in recovery mode" |
| Bus 1 reg-0 reads (12 → `ff…`; e0/e2 → `19 ff…`; e8 → `18 ff…`) | `5ad6dbc6`, `7ddf854f` terminal |
| Bus 2 addr `0x34` 256-byte register map (xxd) | `5ad6dbc6` REQ: James's own `xxd` of the assembled binary |
| Bus 3 addr 72 register reads, 20-byte repeating pattern (regs 0–59) | `bf67067c` terminal (`i2c read 3 72 …`) |
| BMU telemetry: `STATUS 24 4319 0 74% 0000`, `SOC 74`, `CYCLE 0`, `UPTIME 677`, `REFRESH 2000`, `DFCS CE1B 7C47 0847` | `5ad6dbc6`, `3ece828d` terminal |
| BMU sealed: `bmu unseal` → `ERR`; `bmu page 0/1/2` → `ERR`; `bmu reg N` → `ERR` | `5ad6dbc6`, `3ece828d` terminal |
| I2C writes to `0x34` reg `0x00` return OK but don't stick | `3ece828d` terminal |
| `ind mem read` (0xF0000000 et al) → `AERR`; `ind enable 1` → OK first | `5ad6dbc6` terminal |
| `ind fulldl`/`ramdl` → `DLERR`; `ind burnotp` → `OTP DLERR`; `ind verotp` → `OTP VERR` | `5ad6dbc6` terminal |
| `auth check` → idle; `auth profile` → `0 0`; `auth run <nonce>` → `ERR` | `5ad6dbc6` terminal |
| `param colour` → `unknown(0)`; `param rac/cabc/casc/balance/proxystat/recover/ledcal/adccal` outputs | `5ad6dbc6` terminal |
| Console is not a Unix shell (no pipes/loops/vars); `REPEAT` parser is primitive | `bf67067c` terminal (repeated `unknown command` / arg-count errors) |

## Deliberately excluded

- **Pasted assistant "decodings"** of register meanings (e.g. claims about auth
  keys, voltage scalings, chip IDs in `70245911` REQ#1 and `bf67067c` REQ#1):
  these are chatbot speculation pasted by James, never confirmed by him, and are
  not reproduced anywhere in this repo.
- **Device identifiers**: `SYS UDID`, `PARAM FSN`, `PARAM MSN`, `BMU SERIAL` —
  redacted everywhere.
- **Session nonces** (`cbit nonce` outputs) — omitted; single-use random values
  with no research value.
- **James's local paths/hostname** in the `xxd` terminal prompt — redacted.
- **No firmware binaries** exist or are included; **no RDP/BMU/auth bypass
  instructions** are included — none were found, per `notes/limits.md`.
