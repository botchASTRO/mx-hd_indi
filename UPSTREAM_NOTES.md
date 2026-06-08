# MX-HD INDI Driver Upstream Notes

## Summary

- Implemented an INDI telescope driver for the MX-HD equatorial mount.
- Submitted the driver to INDI core and submitted the required driver documentation.
- Both upstream pull requests were merged.

## Confirmed Features

- USB serial connection
- Bluetooth RFCOMM serial connection (when exposed by the OS as a serial device)
- Goto / Sync / Abort
- Park / Unpark
- Driver-specific `MXHD_HOME`
- Standard `TELESCOPE_TRACK_MODE`
- Standard `TELESCOPE_SLEW_RATE`
- Timed pulse guiding
- INDI site / time synchronization
- HA-based `TELESCOPE_PIER_SIDE`

## Implementation Notes

- `Unpark` intentionally performs a HOME-return sequence before clearing the parked state, in order to restore a valid mount attitude / position solution.
- `TELESCOPE_HOME` was not used in the final version due to client compatibility issues observed during testing. HOME is exposed through `MXHD_HOME`.
- The serial layer was refactored to use standard `indicom` `tty_*` helpers.
- The PR branch was built on Raspberry Pi 5 and re-tested on real MX-HD hardware.

## Upstream Pull Requests

- INDI core: `indilib/indi#2409`
- Driver documentation: `indilib/drivers-docs#26`

## Related Repositories

- Standalone driver repository: https://github.com/botchASTRO/mx-hd_indi
- INDI fork used for submission: https://github.com/botchASTRO/indi
- drivers-docs fork used for submission: https://github.com/botchASTRO/drivers-docs
