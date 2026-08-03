# OpenPnP Overview

## What OpenPnP is

OpenPnP is an open source SMT pick-and-place system. It combines:

- machine control
- vision and fiducial alignment
- feeder management
- job, board, panel, part, and package data management

The project is positioned for both hobbyist and low-volume production use. The official site and upstream repository describe it as software that can run custom-built machines as well as retrofit or extend existing commercial machines.

Primary upstream entry points:

- Website: https://openpnp.org/
- Code: https://github.com/openpnp/openpnp
- Wiki: https://github.com/openpnp/openpnp/wiki

## What matters here for feeder integration

For feeder work, four OpenPnP concepts matter most:

1. Drivers: These translate OpenPnP actions into machine-controller commands.
2. Actuators: These are generic devices OpenPnP can actuate or read, including feeder triggers and sensors.
3. Feeders: These provide pick locations and feed behavior for parts.
4. Machine configuration: Most integration lives in `machine.xml` and related OpenPnP configuration.

The official GcodeDriver documentation is especially important because it is the standard way to integrate custom feeder controllers without writing a new Java driver.

## Relevant OpenPnP documentation areas

- Quick start and user workflow: https://github.com/openpnp/openpnp/wiki/Quick-Start
- User manual: https://github.com/openpnp/openpnp/wiki/User-Manual
- Feeders setup: https://github.com/openpnp/openpnp/wiki/Setup-and-Calibration_Feeders
- Actuators setup: https://github.com/openpnp/openpnp/wiki/Setup-and-Calibration_Actuators
- GcodeDriver: https://github.com/openpnp/openpnp/wiki/GcodeDriver

## Feeder support model

OpenPnP supports both simple and advanced feeder models:

- virtual feeders such as strip, tray, and tube feeders
- mechanically actuated feeders such as drag or push/pull feeders
- auto feeders driven through actuators
- slot-based feeder systems that separate a physical slot from a removable feeder identity

That last category is the important one for Siemens Schultz hardware. OpenPnP includes built-in feeder classes for it.

## Built-in Schultz support in core OpenPnP

The OpenPnP source includes these feeder types:

- `SchultzFeeder`
- `SlotSchultzFeeder`

The upstream feeder guide describes them as:

> A feeder slot system similar to ReferenceSlotAutoFeeder for commercial Siemens Siplace Schultz electric feeders.

That means Schultz/Siplace support is not an external plugin. It is part of core OpenPnP.

## Why GcodeDriver matters for SchultzController

OpenPnP's GcodeDriver is documented as a universal driver that can configure complex machines and add-on hardware such as feeders without custom driver code. That is exactly the integration path used by the documented SchultzController setup:

- OpenPnP defines actuators for feeder operations
- the GcodeDriver maps those actuators to controller commands
- SchultzFeeder or SlotSchultzFeeder calls those actuators during feed and status operations

## Daily workflow implications

For normal users, the practical sequence is:

1. Configure the machine and driver.
2. Add and test actuators.
3. Add Schultz feeder slots or direct feeders.
4. Assign part, location, offsets, and actuator mappings.
5. Test `Get ID`, pre-pick, post-pick, feed count, pitch, and status before production use.

For slot-based systems, once the slot geometry and actuator mapping are stable, daily operation shifts toward swapping feeder identities rather than rebuilding feeder configuration each time.