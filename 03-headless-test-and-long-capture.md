# Zigbee Sniffing: Headless Test and Long Capture

## Before you start

You need `whsniff` and `tshark` installed, a dongle that captures, and the channel of the network you want to watch.

**Install.** On NixOS, add both packages to the system packages in the flake, rebuild, and check:

```
environment.systemPackages = with pkgs; [
  whsniff
  wireshark-cli
];
```

```
which whsniff tshark
```

**Dongle.** Plug it in and check that `lsusb | rg -i 0451` lists `0451:16ae`. If it does not capture, see [CC2531 Dongle: Identify and Flash](02-identify-and-flash.md).

**Channel.** In Home Assistant: Settings → Devices & services → Zigbee Home Automation → Configure, in the network settings. On 10 November 2026 it was 20. It stays fixed unless you change it in ZHA.

**Unknown channel.** Scan all 16 channels for 15 seconds each (about 4 minutes) and look for the ones with packets:

```
mkdir -p ~/zigbee_captures && cd ~/zigbee_captures
for c in $(seq 11 26); do
  timeout 15 /run/wrappers/bin/sudo $(which whsniff) -c $c > scan_$c.pcap
  echo "channel $c: $(tshark -r scan_$c.pcap 2>/dev/null | wc -l) packets"
done
```

## Test capture

A 30-second run on a headless machine shows whether packets arrive on your channel.

```
mkdir -p ~/zigbee_captures && cd ~/zigbee_captures
timeout 30 /run/wrappers/bin/sudo $(which whsniff) -c 20 > test.pcap
tshark -r test.pcap | head -20
```

Replace `20` with your channel. `timeout 30` stops the capture after 30 seconds. The output looks like this, with `0x0000` as the coordinator:

```
    1   0.000000       0xc8bb → 0x0000       ZigBee 53 Data, Dst: 0x0000, Src: 0xc8bb
    2   0.002062              →              IEEE 802.15.4 5 Ack
```

A few `Bad FCS` or `Malformed Packet` lines are normal on a busy radio channel. When `tshark` runs as root it prints a warning (`Running as user "root"`), which is harmless. An empty result means the wrong channel or a quiet network.

## Long capture

Run the capture inside `tmux` so it keeps going after the ssh session closes, and let `whsniff` start a new file every hour.

```
tmux new -s zigbee
cd ~/zigbee_captures
/run/wrappers/bin/sudo $(which whsniff) -c 20 -f -h
```

Replace `20` with your channel. Add `tmux` to the system packages in the flake if it is missing.

| Option | Effect |
| --- | --- |
| `-f` | Name the output file by its start date and time |
| `-h` | With `-f`, start a new file every hour |
| `-d` | With `-f`, start a new file every day |
| `-k` | Keep the original FCS, which carries RSSI and LQI |

**Exactly one hour.** Stop after 3600 seconds and write one file:

```
timeout 3600 /run/wrappers/bin/sudo $(which whsniff) -c 20 -f
```

**Leave and come back.** Detach from `tmux` with Ctrl-b, then d. Reattach with `tmux attach -t zigbee`. Stop the capture with Ctrl-C.

**Check the files.** They are named by start time and written to the directory you ran the command in. Confirm with `ls`:

```
ls -lh ~/zigbee_captures
```

## Read the captures

`tshark` reads the files on the headless machine, and Wireshark reads them on any machine with a screen.

**On the headless machine:**

```
tshark -r FILE.pcap | head -20
tshark -r FILE.pcap | wc -l
tshark -r FILE.pcap -Y "wpan.src16==0x8f8a" -T fields -e frame.time_relative -e wpan.src16 -e wpan.dst16 -e frame.len
```

The first shows the first packets, the second counts them, and the third lists the packets from one device (`0x8f8a` as an example) with time, source, destination and length.

**In Wireshark.** Copy the file over and open it:

```
scp homeassistant.lan:zigbee_captures/FILE.pcap .
wireshark FILE.pcap
```

**Encryption.** The payloads are encrypted, so a capture shows addresses, timing and packet sizes, not their contents. Decoding needs the network key of that network.
