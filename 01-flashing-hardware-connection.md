# CC2531 Flashing: Hardware Connection

## What you need

Flashing needs the CC Debugger, its downloader cable, and a USB port for the Debugger and another for the dongle.

- **CC Debugger:** the black TI programmer, with its USB cable.
- **Downloader cable:** the ribbon and the small adapter board from the kit.
- **The dongle** to flash.
- **A machine with Nix**, to run `cc-tool`, and the firmware file. Both are covered in [CC2531 Dongle: Identify and Flash](02-identify-and-flash.md).

## Connect

The Debugger connects to the dongle's DEBUG header through the ribbon and adapter board, with the ribbon's marked edge toward the antenna.

1. Plug the ribbon into the CC Debugger's 10-pin socket. The socket is keyed, so it fits one way.
2. Connect the other end of the chain to the dongle's DEBUG header through the adapter board from the kit.
3. Keep the marked edge on the same side at every connector: the purple edge of the ribbon (red on some cables) must line up with the marked wire of the cable it joins.
4. Pin 1 is the marked edge. On the dongle, the silkscreen puts `1` at the antenna end of the DEBUG header and `10` at the USB end, so the marked edge goes toward the antenna.
5. Plug the dongle and the CC Debugger into USB on the same machine.
6. If the Debugger's LED is red, press its Reset button until the LED turns green.

**Warning.** Some connectors in the kit have 6 pins and are connected to a cable with more wires. Check the kit before connecting anything, and do not force a connector that does not fit.

## Check the connection

A green LED and the detect command show that the Debugger sees the dongle.

```
nix-shell -p cc-tool --run '/run/wrappers/bin/sudo $(which cc-tool) -n CC2531'
```

Expected output:

```
Programmer: CC Debugger
Target: CC2531
No actions specified
```

This only detects the dongle and changes nothing on it. To flash, continue with [CC2531 Dongle: Identify and Flash](02-identify-and-flash.md).

## Troubleshooting

Most problems come from the USB connection or a ribbon that is reversed or loose. Unplug both USB connections before reseating or reversing the ribbon.

| Symptom | What to try |
| --- | --- |
| `CC Debugger device not found` | Check that `lsusb` lists `0451:16a2` (the Debugger), then try another USB cable or port |
| Debugger LED stays red | Press its Reset button, then reseat the ribbon |
| Target not detected | Reseat the ribbon with its marked edge toward the antenna |
| Detected, but flashing fails | Reseat the dongle and the ribbon, press Reset until the LED is green, and run the check again |
