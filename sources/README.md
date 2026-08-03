# Source Index

## Primary sources used

### OpenPnP project

- Website: https://openpnp.org/
- Repository: https://github.com/openpnp/openpnp
- Wiki home: https://github.com/openpnp/openpnp/wiki

### OpenPnP wiki pages

- Quick start: https://github.com/openpnp/openpnp/wiki/Quick-Start
- User manual: https://github.com/openpnp/openpnp/wiki/User-Manual
- Feeders setup: https://github.com/openpnp/openpnp/wiki/Setup-and-Calibration_Feeders
- Actuators setup: https://github.com/openpnp/openpnp/wiki/Setup-and-Calibration_Actuators
- GcodeDriver: https://github.com/openpnp/openpnp/wiki/GcodeDriver
- Schultz feeders: https://github.com/openpnp/openpnp/wiki/SchultzFeeder-and-SlotSchultzFeeder

### OpenPnP source files referenced conceptually

- `src/main/java/org/openpnp/machine/reference/feeder/SchultzFeeder.java`
- `src/main/java/org/openpnp/machine/reference/feeder/SlotSchultzFeeder.java`
- `src/main/java/org/openpnp/machine/reference/feeder/wizards/SchultzFeederConfigurationWizard.java`
- `src/main/java/org/openpnp/machine/reference/feeder/wizards/SlotSchultzFeederConfigurationWizard.java`
- `src/main/java/org/openpnp/machine/reference/ReferenceMachine.java`

### SchultzController project

- Repository: https://github.com/bilsef/SchultzController
- Root README: https://github.com/bilsef/SchultzController/blob/master/README.md
- Scripts README: https://github.com/bilsef/SchultzController/blob/master/Scripts/README.md

### SchultzController source files referenced conceptually

- `Firmware/SchultzController/SchultzController.ino`
- `Firmware/SchultzController/gcode.ino`
- `Firmware/SchultzController/config.h`
- `Firmware/SchultzController/Feeder.cpp`
- `Firmware/SchultzController/Feeder.h`
- `Scripts/SchultzFeeders/CreateSchultzFeeders.js`
- `Scripts/SchultzFeeders/AlignFeeders.js`
- `Scripts/SchultzFeeders/LoadFeederSlots.js`
- `Scripts/SchultzFeeders/UnloadAllSlots.js`

## Important evidence points

- OpenPnP officially documents `SchultzFeeder` and `SlotSchultzFeeder` as support for Siemens Siplace Schultz electric feeders.
- The dedicated OpenPnP Schultz feeder wiki page documents the required actuators and a sample GcodeDriver mapping.
- SchultzController firmware exposes matching M-code commands for feeder control and status reads.
- The OpenPnP source confirms that these feeder types and configuration wizards are part of the main application, not an external plugin.

## Gaps still worth validating on hardware

- exact physical wiring for a specific controller build
- exact slot numbering scheme on a given machine
- whether local firmware revisions differ from the public repo
- whether any non-Schultz Siemens feeder variant is actually Schultz-protocol compatible