# CC2531 Dongle: Identify and Flash

## Identify the dongle

A CC2531 dongle shows up as `0451:16ae` Texas Instruments in `lsusb`.

```
lsusb | rg -i 0451
```

Example output:

```
Bus 001 Device 008: ID 0451:16ae Texas Instruments, Inc. CC2531 Dongle
```

| What `lsusb` shows | Meaning |
| --- | --- |
| `0451:16ae` CC2531 Dongle | A CC2531 dongle, plugged in and detected |
| `0451:16a2` | The CC Debugger, used only for flashing |
| Two `16ae` lines | Two dongles are plugged into this machine |
| Nothing with `0451` | Not detected: try another port or cable, then look at the end of `dmesg` |

The ID shows the chip, not which firmware is on it. Two dongles can look identical, so mark a flashed one, for example with tape. The test capture below shows whether a dongle works.

## Test it

A 30-second capture on the channel of your network shows whether the dongle already works.

```
mkdir -p ~/zigbee_captures && cd ~/zigbee_captures
timeout 30 /run/wrappers/bin/sudo $(which whsniff) -c 20 > test.pcap
tshark -r test.pcap | head -20
```

Replace `20` with your ZHA channel. This needs `whsniff` and `tshark`, which [Zigbee Sniffing: Headless Test and Long Capture](03-headless-test-and-long-capture.md) sets up.

If lines like these appear, the dongle works and needs no flashing:

```
ZigBee 53 Data, Dst: 0x0000, Src: 0x7b5f
IEEE 802.15.4 5 Ack
```

If the output is empty, first confirm the channel with the all-channel scan in the sniffing document. Flash only if the channel is right and the dongle still gives no packets.

## Flash it

Flash only when the channel is confirmed and the dongle still gives no packets, or when you want different firmware on it. Flashing replaces whatever is on the dongle.

**1. Connect.** Connect the Debugger and the dongle as described in [CC2531 Flashing: Hardware Connection](01-flashing-hardware-connection.md), and confirm the Debugger's LED is green.

**2. Check.** Confirm that the Debugger sees the dongle. Expected output is `Programmer: CC Debugger`, `Target: CC2531` and `No actions specified`.

```
nix-shell -p cc-tool --run '/run/wrappers/bin/sudo $(which cc-tool) -n CC2531'
```

**3. Flash.** Erase, write and verify. To flash other firmware, change the `.hex` path.

```
nix-shell -p cc-tool --run '/run/wrappers/bin/sudo $(which cc-tool) -n CC2531 -e -w ~/Admin/ZigbeeMonitoring/WorkDir/sniffer_fw_cc2531.hex -v'
```

Expected output:

```
Programmer: CC Debugger
Target: CC2531
Erasing flash...
Completed
Writing flash (8 KB)...
Completed (0.68 s.)
Verifying flash...
Completed (0.39 s.)
```

**Firmware file.** `sniffer_fw_cc2531.hex` is 23 KB and lives in `~/Admin/ZigbeeMonitoring/WorkDir`. If it is lost, download PACKET-SNIFFER (not PACKET-SNIFFER-2) from ti.com/tool/PACKET-SNIFFER, which gives `swrc045z.zip`, and extract the file:

```
nix-shell -p p7zip --run '7z x swrc045z.zip -oPACKET-SNIFFER && 7z e PACKET-SNIFFER/Setup_SmartRF_Packet_Sniffer_2.18.0.exe bin/general/firmware/sniffer_fw_cc2531.hex'
```

Router or coordinator firmware is a different file. Which one to use is not covered here.

## Verify

After flashing, unplug the CC Debugger and the ribbon, then unplug the dongle and plug it back in alone. Check that it is detected:

```
lsusb | rg -i 0451
```

It should show `0451:16ae`. Then repeat the 30-second test capture above. If packets appear, the dongle works.
