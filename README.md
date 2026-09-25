# Solar Monitor

Firmware releases for Solar Monitor, an ESP32 board that reads a Victron
SmartSolar charge controller over Bluetooth and shows its battery voltage,
charge current, solar power and history on a web page on your local network.

This repository only publishes firmware. The source is kept elsewhere.

## Updating a board

On the board's page, open **Setup -> Firmware** and press **Check for update**.
If a newer release is listed here, press **Install**. The board downloads it,
checks it, and restarts into it - about a minute. Nothing is installed unless
someone presses the button, and if new firmware fails to start, the board goes
back to the previous version by itself.

## What each release contains

- `solar-monitor.bin` - the firmware image the board installs
- `version.json` - its version number, download link, size and checksum, which
  the board reads to decide whether there is an update

The first install onto a new board is done over USB; after that, updates come
from here.
