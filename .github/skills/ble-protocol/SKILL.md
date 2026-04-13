---
name: ble-protocol
description: "Genial-T31 BLE protocol reference — GATT services, packet format, commands, temperature decoding. Use when implementing or reviewing BLE communication code."
---

# Genial-T31 BLE Protocol Skill

Quick reference for the reverse-engineered Genial-T31 BLE protocol. All hex values are verified against [Docs/FINDINGS.md](../../../Docs/FINDINGS.md).

**Canonical source:** Always cross-reference `Docs/FINDINGS.md` and `Docs/PROJECT_SPEC.md` for the full protocol details. This skill is a quick-access summary — if there's a discrepancy, the docs are authoritative.

## Device Identity

| Property | Value |
|---|---|
| Device name | `Genial-T31` |
| Default MAC | `C1:80:10:00:0A:EE` (configurable in settings) |
| BLE chip | TI CC2541 |

## GATT Service Map

| Service UUID | Description | Used? |
|---|---|---|
| `00001809-0000-1000-8000-00805f9b34fb` | Health Thermometer (main) | **YES** |
| `0000180f-0000-1000-8000-00805f9b34fb` | Battery Service | Optional |
| `0000180a-0000-1000-8000-00805f9b34fb` | Device Information | Optional |
| `f000ffc0-0451-4000-b000-000000000000` | TI OAD (firmware) | No |

## Key Characteristics (under service `0x1809`)

| Characteristic UUID | Direction | Purpose |
|---|---|---|
| `0000FFF2-0000-1000-8000-00805f9b34fb` | **Write** | Send commands to device |
| `0000FFF1-0000-1000-8000-00805f9b34fb` | **Notify** | Receive responses & temperature data |

## Notification Setup

Enable notifications on FFF1 by writing to CCCD descriptor:
```
Descriptor: 00002902-0000-1000-8000-00805f9b34fb
Value: 0x01 0x00
```

## Packet Format

All packets (TX and RX):
```
A6 [LEN] [CMD] [PAYLOAD...] [CHECKSUM] 6A
```

| Field | Size | Description |
|---|---|---|
| `0xA6` | 1 byte | Start marker (fixed) |
| LEN | 1 byte | Byte count of CMD + PAYLOAD |
| CMD | 1 byte | Command identifier |
| PAYLOAD | 0+ bytes | Command-specific data |
| CHECKSUM | 1 byte | `(LEN + CMD + sum(PAYLOAD)) & 0xFF` |
| `0x6A` | 1 byte | End marker (fixed) |

### Packet Builder (Kotlin)

```kotlin
fun buildPacket(cmd: Byte, payload: ByteArray = byteArrayOf()): ByteArray {
    val len = (1 + payload.size).toByte()
    val checksum = ((len + cmd + payload.sum()) and 0xFF).toByte()
    return byteArrayOf(0xA6.toByte(), len, cmd, *payload, checksum, 0x6A)
}
```

## Commands Reference

### Poll Temperature — CMD `0x01` (Primary Command)

```
TX: A6 02 01 00 03 6A
```
Send every ~4-5 seconds for continuous readings.

**Response format (notify on FFF1):**
```
A6 09 01 [T1_HI] [T1_LO] [T2_HI] [T2_LO] [05] [HH] [MM] [SS] [CHK] 6A
```

**Temperature decoding:**
```kotlin
val temp1 = ((data[3].toInt() and 0xFF) shl 8 or (data[4].toInt() and 0xFF)) / 100.0
val temp2 = ((data[5].toInt() and 0xFF) shl 8 or (data[6].toInt() and 0xFF)) / 100.0
```
- T1 and T2 are two probe channels (identical when single probe connected)
- `[05] [HH] [MM] [SS]` is the device-side timestamp
- Unit: **°C**

### Get Status/Battery — CMD `0xA5`

```
TX: A6 02 A5 00 A7 6A
RX: A6 03 5A 00 [battery%] [chk] 6A
```
**Note:** Response CMD is `0x5A` (not `0xA5`). Third payload byte is battery percentage.

### Get Device Info — CMD `0xB1`

```
TX: A6 02 B1 00 B3 6A
RX: A6 05 B1 [seq] [b1] [b2] [b3] [chk] 6A
```

### Get Config — CMD `0x1D`

```
TX: A6 02 1D 00 1F 6A
RX: A6 08 1D 01 6D 01 00 00 00 00 [chk] 6A
```

### Set Time — CMD `0x37`

```
TX: A6 05 37 [05] [HH] [MM] [SS] [chk] 6A
RX: A6 02 37 00 39 6A  (ACK)
    A6 02 38 01 3B 6A  (success confirmation)
```

## Verified Temperature Readings

| Raw hex | T1 | T2 |
|---|---|---|
| `A609010A440A440514050BCF6A` | 26.28°C | 26.28°C |
| `A609010A530A5305140B02EA6A` | 26.43°C | 26.43°C |
| `A609010A500A5005140B04E66A` | 26.40°C | 26.40°C |
| `A609010A4D0A4D05140B08E46A` | 26.37°C | 26.37°C |

## Full Connection Flow

```
1.  Scan for BLE device by name "Genial-T31" or configured MAC
2.  Connect GATT (autoConnect = false)
3.  Discover services
4.  Find service 0x1809, chars FFF1 (notify) and FFF2 (write)
5.  Enable notifications on FFF1 (write 0x0100 to CCCD 0x2902)
6.  Write FFF2: A6 02 A5 00 A7 6A          → get battery/status
7.  Write FFF2: A6 02 B1 00 B3 6A          → get device info
8.  Parse responses on FFF1 (CMD 0x5A, 0xB1)
9.  Write FFF2: A6 02 1D 00 1F 6A          → get config
10. Write FFF2: A6 05 37 05 HH MM SS CHK 6A → sync time
11. LOOP (2-3 iterations, ~4s apart):
      Write FFF2: A6 02 01 00 03 6A         → poll temperature
      Parse FFF1 notify → CMD 0x01 → extract T1, T2
      Forward reading to HA + Health Connect
12. Disconnect GATT cleanly
```

**Total session time: ~10-15 seconds**

## BLE Coexistence Rules

- **Never hold connections longer than necessary.** Connect → poll → disconnect.
- **If connection fails** (device busy with GenialCloud), silently retry on next WorkManager cycle.
- **No aggressive retry loops** — GenialCloud only holds the connection for ~10-16 seconds per session.
- **autoConnect = false** for faster connection when device is available.
