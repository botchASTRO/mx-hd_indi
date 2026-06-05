# MX-HD Telescope INDI Driver

This folder contains an INDI telescope driver for the MX-HD equatorial mount.

## Supported (initial)
- Serial connection via USB-serial (`/dev/ttyUSB0` etc.) and Bluetooth RFCOMM serial devices (`/dev/rfcomm0` etc.)
- RA/Dec read: `:GR#` / `:GD#` (parsing is tolerant to separators)
- Slew/Goto: `:Sr ...#`, `:Sd ...#`, `:MS#` (expects `0`)
- Sync: `:CM#`
- Tracking on/off: `@FD1#` / `@FD0#`
- Standard `TELESCOPE_TRACK_MODE`:
  - Sidereal: `@CE0#`
  - Solar: `@LP1#`
  - Lunar: `@LP4#`
- Home (`MXHD_HOME`): `@OG#`
- Park: `@Hm#`
- Unpark: `@OG#` (returns to Home before clearing parked state)
- Status: `@ST#` reads 3 binary bytes (no terminator)
- Pulse guide: `@mN#/@mS#/@mE#/@mW#` with timed stop via `:Q#`
- Pier side: HA-based `TELESCOPE_PIER_SIDE` for GEM operation

## Build (Raspberry Pi / Ubuntu)
Install prerequisites:

```bash
sudo apt update
sudo apt install -y build-essential cmake libindi-dev libnova-dev
```

Build:

```bash
cd indi-mxhd
mkdir -p build
cd build
cmake ..
make -j
```

Install:

```bash
sudo make install
```

If `INDI_DATA_DIR` is available from `libindi`, the build also installs
`indi_mxhd_telescope.xml` so the driver appears in INDI driver lists.

## Run
Typically under indiserver:

```bash
indiserver -v indi_mxhd_telescope
```

## Notes
- `@ST#` polling follows the INDI `POLLING_PERIOD` setting. The default is 4 seconds.
- Bluetooth is supported when the operating system exposes the MX-HD link as a serial RFCOMM device such as `/dev/rfcomm0`.
- Serial RX buffer is flushed before each command/query to reduce stale-byte desync.
- All mount I/O is serialized with a mutex to avoid read/write interleaving.
- `updateLocation()` and `updateTime()` now push INDI site/time updates to the mount:
  - INDI longitude is treated as degrees east positive.
  - MX-HD receives the converted west-positive `:Sg` value.
  - INDI UTC offset is east-positive; the mount gets the reversed-sign `:SG` value.
- `:SC` / `:SL` are sent using local date/time derived from UTC plus the reported offset.
- `TELESCOPE_TRACK_MODE` is implemented using the standard INDI sidereal / solar / lunar modes rather than a driver-specific tracking-rate property.
- `Unpark` is intentionally implemented as a HOME return sequence before `SetParked(false)`.
- This is specific to MX-HD: returning to the fixed HOME pose restores a known mount attitude/position solution for reliable remote operation.
- Home is exposed as a driver-specific `MXHD_HOME` action. This avoids client-side issues with standard `TELESCOPE_HOME` handling while preserving MX-HD's fixed HOME recovery behavior.
- `TELESCOPE_PIER_SIDE` is derived from hour angle using the current UTC at each status update.
- The driver enables `SIMULATE_PIER_SIDE` because MX-HD does not report pier side directly; the mount's guaranteed HA-consistent pointing model makes this a valid approach.
- Home/Park/Abort behavior is implemented to match MX-HD operational semantics, including HOME-based unpark recovery.
