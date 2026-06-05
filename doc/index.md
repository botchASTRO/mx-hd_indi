# MX-HD Telescope

## Overview

`indi_mxhd_telescope` is an INDI telescope driver for the MX-HD equatorial mount.

The driver is implemented on top of `INDI::Telescope` and speaks the MX-HD command set directly over a serial link. It supports both USB serial adapters such as `/dev/ttyUSB0` and Bluetooth RFCOMM serial devices such as `/dev/rfcomm0` when the operating system exposes the mount as a normal serial port.

## Supported Features

- Connect and disconnect over serial
- Read current RA/Dec
- Slew/Goto
- Sync
- Abort
- Home (`MXHD_HOME`)
- Park / Unpark
- Tracking enable / disable
- Standard sidereal / solar / lunar `TELESCOPE_TRACK_MODE`
- Timed pulse guiding for guider clients such as PHD2
- Site and time updates from INDI clients
- Pier-side reporting for GEM operation using hour-angle based simulation

## Connection

The driver expects the mount to be reachable through a serial device file.

Typical examples:

- `/dev/ttyUSB0`
- `/dev/rfcomm0`

The baud rate should match the MX-HD connection being used. The driver has been tested with `9600` baud over USB serial and with Bluetooth RFCOMM presented as a serial device by the operating system.

## Site, Time, and Time Zone

The driver accepts `TIME_UTC` and `GEOGRAPHIC_COORD` from INDI clients and forwards them to the mount.

Important details:

- INDI longitude is interpreted as degrees east positive
- MX-HD receives longitude converted to its west-positive format
- INDI UTC offset is east-positive
- MX-HD receives the reversed-sign offset required by its `:SG` command
- Local date and time sent with `:SC` and `:SL` are derived from UTC plus the INDI-provided offset

The driver intentionally uses INDI-provided site and time values. It does not read the mount's stored site information and feed it back into the control flow.

## Home and Unpark Behavior

MX-HD treats Home and Park differently.

- Home is a fixed absolute pose and is exposed through the driver-specific `MXHD_HOME` action
- Park is a stored park position

For MX-HD, `Unpark` is intentionally implemented as a HOME return sequence before the driver clears the parked state. This is done to restore a known mount attitude and position solution before remote operation resumes.

## Pier Side

MX-HD does not report pier side directly. The driver derives `TELESCOPE_PIER_SIDE` from the current hour angle.

Implementation notes:

- The hour angle is computed using the current UTC at each status update
- The calculation uses the current RA and the INDI-provided east-positive longitude
- `SIMULATE_PIER_SIDE` is enabled because pier side is inferred rather than reported directly by the mount

This approach is suitable for MX-HD because the mount always slews into a pose that remains consistent with hour-angle based pier-side determination.

## Polling

Mount status is polled with `@ST#`.

- The driver follows the INDI `POLLING_PERIOD` property
- The default polling period is `4000` ms

## Notes

- Serial I/O is serialized with a mutex to avoid interleaved reads and writes
- The RX buffer is flushed before each command/query to reduce stale-byte desynchronization
- Tracking mode is exposed through the standard INDI `TELESCOPE_TRACK_MODE` property
- Timed guiding is implemented with MX-HD guide-move commands and a timed stop sequence

## Packaging Notes

- The driver source lives in the `indi-mxhd` project directory and is intended to be packaged as `indi_mxhd_telescope`.
- Local backup artifacts are kept under `backup/` for development only and should not be included in public release packaging.
