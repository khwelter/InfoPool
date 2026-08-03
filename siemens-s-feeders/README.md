# Siemens S-Feeders and OpenPnP Support

## Short answer

OpenPnP has explicit built-in support for Siemens Siplace Schultz electric feeders through:

- `SchultzFeeder`
- `SlotSchultzFeeder`

The official feeder documentation calls these feeders a slot system for commercial Siemens Siplace Schultz electric feeders.

## What I could verify

From official OpenPnP sources, the supported Siemens path is specifically the Schultz/Siplace feeder family.

Verified sources:

- OpenPnP feeder guide names `SchultzFeeder and SlotSchultzFeeder`
- OpenPnP source tree contains feeder classes and configuration wizards for both
- OpenPnP wiki has a dedicated page: `SchultzFeeder and SlotSchultzFeeder`

## What I did not verify

I did not find separate official OpenPnP documentation for other Siemens feeder families under a generic `S-feeder` label.

So the safe conclusion is:

- OpenPnP officially documents Siemens Schultz/Siplace electric feeder support
- broader Siemens feeder compatibility should be treated as unverified unless it is electrically and protocol-compatible with the Schultz feeder path

## Core OpenPnP support surface

### SchultzFeeder

This is the direct feeder abstraction. It stores:

- feeder number as `actuatorValue`
- actuator names for pre-pick, post-pick, status, ID, count, pitch, and pitch toggle
- fiducial part information

It feeds by:

1. resolving the configured actuator
2. moving the nozzle to the feeder pick location at safe Z
3. actuating the configured feeder command

It can also execute an optional post-pick actuator after pickup.

### SlotSchultzFeeder

This extends `SchultzFeeder` and adds slot logic:

- a slot has a physical location
- a bank groups removable feeders
- a feeder identity can be loaded into or unloaded from a slot
- the effective pick location is `slot location + feeder offsets`

This is the better model for a machine where feeders are removable and may move between slots.

## OpenPnP slot concepts specific to Schultz

SlotSchultzFeeder keeps a persistent bank database in machine properties under:

- `SchultzFeederSlot.banks`

Each bank contains feeder records with:

- feeder ID / name
- part assignment
- offsets relative to the slot base location

This lets the machine keep stable slot positions while tracking which removable feeder is currently installed.

## Official setup flow in OpenPnP docs

The dedicated wiki page describes this pattern:

1. Add a `SlotSchultzFeeder`.
2. Name the slot.
3. Physically mount the feeder.
4. Assign the feeder number.
5. Select actuators.
6. Test the actuators.
7. Read the feeder ID and create/load the removable feeder record.
8. Use the feeder fiducial for precise location refinement.
9. Assign the part.
10. Set feeder offsets by centering on the exposed part.

## Practical distinction: direct vs slot mode

Use `SchultzFeeder` when:

- the feeder behaves like a fixed feeder position
- you do not need removable-slot bookkeeping

Use `SlotSchultzFeeder` when:

- feeders are swapped between slots
- you want feeder identity from EEPROM to drive automatic loading
- you want reusable per-feeder offsets and part assignments

## Important caution on feeder numbering

There is a small documentation mismatch worth noting:

- the OpenPnP wiki says to assign feeder number `0 to 40`
- the SchultzController firmware defines `NUMBER_OF_FEEDERS` as `40`, which makes the valid feeder-number range `0` through `39`

Treat `0-39` as the firmware-grounded range unless your controller firmware has been modified.

## What this means for "Siemens S-feeder" support

If by `Siemens S-feeders` you mean Siemens Siplace Schultz electric feeders, OpenPnP support is real, built-in, and documented.

If you mean another Siemens feeder family, you still need to verify:

- electrical interface compatibility
- command protocol compatibility
- whether SchultzController or an equivalent controller can drive it
- whether OpenPnP can treat it as Schultz-compatible through the same actuator contract