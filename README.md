> [!WARNING]
> **All contant of this InfoPool is AI generated and AI researched.**
> ***There will be errors in the pages, which will be fixed in due time once known.***

# OpenPnP Info Pool

This repository collects source-grounded notes about OpenPnP with an emphasis on Siemens Siplace Schultz electric feeders and the SchultzController ecosystem.

## Documents

- [OpenPnP overview](openpnp/README.md)
- [Siemens S-feeders and OpenPnP support](siemens-s-feeders/README.md)
- [SchultzController and OpenPnP configuration](schultzcontroller/README.md)
- [Juki nozzle vision notes](juki-nozzles/README.md)
- [Source index](sources/README.md)

## Focus

The main focus of this info pool is practical configuration of SchultzController-backed Schultz feeders in OpenPnP:

- what OpenPnP supports natively
- which actuators must exist
- how the GcodeDriver is typically configured
- how SlotSchultzFeeder differs from SchultzFeeder
- what is documented vs. what still needs verification on a specific machine

## Scope note

OpenPnP's official feeder documentation explicitly names `SchultzFeeder` and `SlotSchultzFeeder` as support for commercial Siemens Siplace Schultz electric feeders. I did not find separate official OpenPnP documentation for a broader Siemens S-feeder family beyond that Schultz/Siplace feeder path, so this repo treats that as the primary supported Siemens feeder integration.