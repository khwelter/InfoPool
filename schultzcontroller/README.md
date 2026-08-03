# SchultzController and OpenPnP Configuration

## What SchultzController is

SchultzController is a G-code controller for Siemens S feeders, specifically the Siemens Siplace Schultz feeder path documented by OpenPnP.

Upstream repo:

- https://github.com/bilsef/SchultzController

The upstream README describes it as:

- a controller board that distributes power to the feeders
- a communication interface for 20 feeder units / 40 lanes
- available in a 4-pin and a 96-pin hardware variant
- based on an Arduino Nano Every

## Why it fits OpenPnP well

It fits OpenPnP's native architecture almost exactly:

- OpenPnP already has `SchultzFeeder` and `SlotSchultzFeeder`
- OpenPnP uses actuators as the interface contract
- SchultzController exposes feeder operations as G-code M-codes
- GcodeDriver maps those M-codes to actuators

So the integration does not require a custom Java feeder plugin.

## The most important OpenPnP design decision

Use `SlotSchultzFeeder` unless you have a strong reason not to.

That is the configuration model the OpenPnP wiki focuses on, and it is the best fit for removable Siemens feeders because it separates:

- slot geometry on the machine
- feeder identity stored in EEPROM
- part assignment and feeder offsets

## Required actuators

The dedicated OpenPnP wiki page lists nine required actuator roles shared by `SchultzFeeder` and `SlotSchultzFeeder`:

1. `GetID`
2. `Pre-Pick`
3. `Post-Pick`
4. `AdvanceIgnoreError`
5. `GetCount`
6. `ClearCount`
7. `GetPitch`
8. `TogglePitch`
9. `GetStatus`

If you use more than one controller, each controller needs its own uniquely named set of actuators.

## What OpenPnP binds in the Schultz feeder wizard

The source-backed configuration fields are:

- feeder number: `actuatorValue`
- pre-pick actuator: `actuatorName`
- post-pick actuator: `postPickActuatorName`
- get feed count actuator: `feedCountActuatorName`
- clear count actuator: `clearCountActuatorName`
- get pitch actuator: `pitchActuatorName`
- toggle pitch actuator: `togglePitchActuatorName`
- get status actuator: `statusActuatorName`
- get ID actuator: `idActuatorName`

For `SlotSchultzFeeder`, OpenPnP also binds:

- slot base location
- fiducial part
- feed retry count
- pick retry count
- bank
- removable feeder selection
- feeder offsets X/Y/Z/rotation
- removable feeder part assignment

## What each operation does in practice

### `GetID`

Reads feeder EEPROM through the configured actuator and fills the ID field in the configuration wizard.

### `Pre-Pick`

Used before pickup. The OpenPnP wiki describes this as opening the shutter.

### `Post-Pick`

Used after pickup. The OpenPnP wiki describes this as closing the shutter and advancing the tape.

### `AdvanceIgnoreError`

Alternate advance path for testing or recovery when the feeder reports an error condition.

### `GetCount` and `ClearCount`

Read and reset feed count.

### `GetPitch` and `TogglePitch`

Read the current pitch and toggle between 2 mm and 4 mm.

### `GetStatus`

Reads feeder state for diagnostics.

## SchultzController firmware command map

From the firmware source:

- `M115`: controller firmware info
- `M600 N...`: pre-pick
- `M601 N...`: advance/post-pick
- `M601 N... X1`: advance ignoring error
- `M602 N...`: feeder status
- `M603 N...`: feed count
- `M608 N...`: get pitch
- `M610 N...`: get feeder ID
- `M623 N...`: clear feed count
- `M628 N...`: toggle pitch
- `M630 N...`: read EEPROM
- `M640 N... X...`: set feeder ID
- `M615 N...`: feeder firmware info
- `M650 N...`: start self-test
- `M651 N...`: stop self-test

The OpenPnP wiki uses the subset needed for normal feeder operation.

## Firmware-grounded controller behavior

Important controller facts from the SchultzController firmware:

- controller serial baud is `115200`
- feeder-side serial is initialized at `9600`
- firmware announces itself with `FIRMWARE_NAME: Schultz Feeder Controller, FIRMWARE_VERSION: 2.0`
- success responses start with `ok`
- failure responses start with `error`
- feeder numbers are validated against `0..39`

## OpenPnP GcodeDriver configuration pattern

The documented OpenPnP setup uses a dedicated GcodeDriver for the Schultz controller with:

- `COMMAND_CONFIRM_REGEX`: `^ok.*`
- `COMMAND_ERROR_REGEX`: `^error.*`

And actuator command mappings like:

```xml
<command head-mountable-id="actSchultzGetID" type="ACTUATOR_READ_COMMAND">
   <text><![CDATA[M610N{IntegerValue}]]></text>
</command>
<command head-mountable-id="actSchultzGetID" type="ACTUATOR_READ_REGEX">
   <text><![CDATA[^ok.*ID: (?<Value>.+)]]></text>
</command>

<command head-mountable-id="actSchultzPrePick" type="ACTUATE_DOUBLE_COMMAND">
   <text><![CDATA[M600N{IntegerValue}]]></text>
</command>

<command head-mountable-id="actSchultzPostPick" type="ACTUATE_DOUBLE_COMMAND">
   <text><![CDATA[M601N{IntegerValue}]]></text>
</command>

<command head-mountable-id="actSchultzAdvIgnorErr" type="ACTUATE_DOUBLE_COMMAND">
   <text><![CDATA[M601N{IntegerValue}X1]]></text>
</command>
```

The same pattern is used for count, pitch, and status reads.

## Minimal configuration checklist in OpenPnP

### 1. Add the controller driver

Create a dedicated GcodeDriver for SchultzController rather than mixing these commands into your main motion controller unless you have a good reason to.

Why:

- the OpenPnP docs already frame this as a separate controller
- actuator traffic is logically separate from motion traffic
- multi-controller setups are explicitly supported by OpenPnP

### 2. Add the actuators first

In `Machine Setup`, create the actuator set before you configure the feeders.

Set each actuator to a value type compatible with the documented examples. The sample OpenPnP config uses `Double` and then passes the feeder number as the numeric value.

### 3. Bind driver commands to actuator roles

For each actuator:

- assign the SchultzController driver
- add the matching `ACTUATE_DOUBLE_COMMAND` or `ACTUATOR_READ_COMMAND`
- add a matching regex for read commands

### 4. Verify controller handshake first

Before touching feeder slots, verify:

- driver connects cleanly
- `^ok.*` and `^error.*` actually match your controller responses
- `M115` returns the expected firmware identity

If this step fails, stop there. The feeder layer will only hide the real problem.

### 5. Add `SlotSchultzFeeder` entries

For each slot:

- set slot name
- set slot base location
- set feeder number in `Feeder Number`
- assign all relevant actuators

### 6. Test in this order

Use the built-in wizard buttons in roughly this order:

1. `Get ID`
2. `Get Status`
3. `Get Pitch`
4. `Test pre pick`
5. `Test post pick`
6. `Get feed count`
7. `Clear feed count`

That order isolates communication and EEPROM reads before mechanical movement.

### 7. Create or load the removable feeder record

Once `Get ID` works:

- load the feeder into the slot if it already exists in the bank
- otherwise create it from the ID and assign part plus offsets

### 8. Set location and offsets correctly

For `SlotSchultzFeeder`, there are two location concepts:

- slot location: where the slot base is on the machine
- feeder offsets: where the exposed pick point is relative to the slot base

The official docs recommend using the feeder fiducial dot as the slot location reference, then capturing the exposed-part location into feeder offsets.

## Source-grounded operational details

### Automatic data refresh in the wizard

If the machine is enabled, the Schultz feeder wizards automatically try to read:

- ID
- feed count
- pitch
- status

That means a broken actuator or regex will show up immediately when the feeder config opens.

### Slot defaults when a new feeder identity is created

When a feeder ID is read and no feeder record exists in the selected bank, the slot wizard creates one and seeds default offsets:

- X offset: `-5`
- Y offset: `-30`

Treat these as placeholders, not calibrated values.

### Effective pick location formula

For `SlotSchultzFeeder`, OpenPnP computes the pick location as:

- slot base location offset by the removable feeder's stored offsets

This is the central reason slot mode scales better than manually duplicating direct feeders.

### Enablement logic

`SlotSchultzFeeder` is only effectively usable when:

- the parent feeder is enabled
- a removable feeder is loaded into the slot
- that removable feeder has a part assigned

If one of those is missing, the feeder should be treated as incomplete.

## Recommended practical commissioning sequence

1. Flash SchultzController and confirm the board variant in `config.h`.
2. Verify controller serial link and firmware response.
3. Create the nine actuators.
4. Add GcodeDriver command and regex mappings.
5. Add one SlotSchultzFeeder only.
6. Prove `Get ID`, `Status`, `Pitch`, `Pre-Pick`, and `Post-Pick` on that one slot.
7. Calibrate slot location and feeder offsets.
8. Only then scale out to the rest of the slots, ideally via the helper scripts.

That sequence minimizes ambiguity. If all 40 slots are created before one slot is proven, debugging gets slower than it needs to be.

## Helper scripts worth knowing

The SchultzController repo includes OpenPnP scripts for slot-heavy setups:

- `CreateSchultzFeeders.js`: bulk-creates SlotSchultzFeeder slots
- `AlignFeeders.js`: re-aligns slot locations from feeder fiducials
- `LoadFeederSlots.js`: reads feeder IDs and auto-loads matching feeders into slots
- `UnloadAllSlots.js`: clears loaded slot assignments

These scripts are intended to go into the OpenPnP scripts directory.

## Risks and verification points

Items to verify on a real machine before trusting production:

- feeder-number to physical-port mapping
- exact regex matches for your firmware build
- whether your setup needs `PostPick` or `AdvanceIgnoreError` during early tuning
- shutter behavior and cover-tape tension
- slot pitch, origin, and rotation assumptions used by any helper scripts
- whether your controller build is 4-pin or 96-pin and wired consistently with the selected firmware board mode

## Bottom line

The OpenPnP side of SchultzController integration is not mysterious. It is a standard actuator-plus-GcodeDriver setup wrapped by built-in feeder classes. The hard parts are usually:

- wiring
- feeder numbering
- response regex correctness
- slot geometry calibration

If those four are right, the rest of the integration follows the existing OpenPnP model cleanly.