# Create a PDU and its signals

Create a new PDU (with its interface, and optionally its frame, its system and its
timing), add signals to a PDU, or change a PDU (its timing too) or a signal.

## 1. Where am I?

The System Description editor (skill `modellar-communication`, task
`find-pdus-and-signals.md`, section 1). On a diagram's dock panel, press **Edit**
(top right) first: it opens view only.

## 2. MCP route (default)

**New PDU**

1. Look first: `list_pdus({ modelId, search: "<name>" })`. A PDU with that name
   already made in the app makes `create_pdu` refuse.
2. Find the system: `list_systems({ modelId })` (pick by `sourceFile` when names
   repeat). **Pick the system now**: without one, no ECU can send or receive the
   PDU. A frame is optional: `search_elements({ modelId, elementType: "Frame",
   search: "<frame name>" })`.
3. The batch: `list_staging_batches({ modelId })`, the user picks.
4. Summarise (name, length in bytes, PDU type, bus, system, frame, timing in ms)
   and wait for go.
5. `create_pdu({ modelId, batchId, name, lengthBytes, controllerType, pduType?,
   systemId?, frameId?, timing? })`. `controllerType` is the bus: `CAN`, `CANFD`,
   `LIN`, `FlexRay`, `Ethernet` or `MOST`.

   `timing` is optional, every time in **milliseconds**. Leave it out for none:
   - `transmissionMode`: `CYCLIC` (sent every period), `EVENT` (sent when the data
     changes, repeated) or `MIXED` (both);
   - `cyclic: { periodMs, offsetMs? }`: required for `CYCLIC` and `MIXED`, left
     out for `EVENT`. `periodMs` > 0, `offsetMs` ≥ 0;
   - `event: { numberOfRepetitions, repetitionPeriodMs? }`: required for `EVENT`
     and `MIXED`, left out for `CYCLIC`. `numberOfRepetitions` is a whole number
     ≥ 0, `repetitionPeriodMs` > 0;
   - `minimumDelayMs?`: ≥ 0, the least time between two sends.

   Only these names are accepted: a misspelled or extra key (`period`, `offset`,
   `repetitionPeriod`) is refused, and a wrong timing creates nothing. Example, a
   100 ms cycle with a 10 ms minimum delay:

   ```
   timing: { transmissionMode: "CYCLIC", cyclic: { periodMs: 100 }, minimumDelayMs: 10 }
   ```

   Event-driven, 3 repetitions 20 ms apart:
   `{ transmissionMode: "EVENT", event: { numberOfRepetitions: 3, repetitionPeriodMs: 20 } }`.

**New signal**

1. `get_pdu({ modelId, pduId })`: its signals and free bits.
2. Summarise (name, start bit, bit length, all in **bits**) and wait for go.
3. `add_signal({ modelId, batchId, pduId, name, bitLength, startBit? })`. Without
   `startBit` it goes right after the last signal; the answer says where. It is
   refused when it overlaps a signal, doesn't fit the PDU's length, or the name is
   taken in that PDU.

**Change a PDU or a signal**: no dedicated tool. Use `update_staged_element`
(general rules apply), or the visual route below.

**Change a PDU's timing** (period, cycle time, repetitions, minimum delay):

1. `get_pdu({ modelId, pduId })`: its `timing` (ms) and `timingSource`.
2. Summarise old and new timing, in ms, and wait for go.
3. `update_staged_element` on the PDU (elementType `BusDatagram`) with `timing`
   in **seconds**, the stored form, with these names: `{ transmissionMode,
   cyclic: { period, offset? }, event: { numberOfRepetitions, repetitionPeriod? },
   minimumDelay? }`. Same mode rules as `create_pdu`. A 100 ms cycle is
   `{ transmissionMode: "CYCLIC", cyclic: { period: 0.1 } }`. `timing: null`
   clears it; leaving `timing` out keeps the stored one.
4. Read it back with `get_pdu`: `timing` in ms again, `timingSource` `stored`.

A failed write lists what it already staged before the failure: don't stage those
again. Afterwards, send the user to the System Description page and have them press
**Refresh**.

## 3. Visual route

**New PDU**

1. Click **New PDU**. Dialog **New PDU**.
2. **Name**, **Length (bytes)**, **PDU type**, **Controller**.
3. **Frame (optional)** and **System (optional)**: searchable lists. Type part of a
   name, or of the second line under it (a frame's path, a system's file name),
   then click the row. The chosen row shows that second line too.
4. **Timing** (the section at the bottom of the dialog, optional): see **Timing
   section** below. Leave **Transmission** on **None** for no timing.
5. Summarise and wait for go. Click **Create** (disabled while a Timing field is
   wrong). Toast **PDU created**; the new PDU's row opens, its header showing the
   timing badge.

**Timing section** (New PDU and Edit PDU, every time in **ms**)

1. **Transmission**: **None**, **Cyclic**, **Event** or **Cyclic + Event**. Only
   the fields that choice needs appear:
   - **Cyclic**: **Period (ms)**, **Offset (ms)**, **Minimum delay (ms)**;
   - **Event**: **Repetitions**, **Repetition period (ms)**, **Minimum delay (ms)**;
   - **Cyclic + Event**: all five.
2. **Period (ms)** and **Repetitions** are required (placeholder "required"); the
   others are optional (placeholder "optional"). **Repetitions** is a whole number.
3. Under a wrong field: **Required.**, **Enter a number.**, **Enter a whole
   number.**, **Must be more than 0.** (Period, Repetition period) or **Must be 0
   or more.** (Offset, Repetitions, Minimum delay).
4. Choosing **None** in Edit PDU removes the PDU's timing on **Save**.

**New signal**

1. In the PDU's opened row click **+ Signal** (named "Add signal to <PDU>"). Dialog
   **New Signal**, with **Parent PDU** set and **Start bit** at the first free bit.
   (The toolbar's **New Signal** does the same for the one open PDU, or the PDU just
   created; otherwise pick **Parent PDU** from its searchable list.)
2. **Name**, **Bit length** (bits). **End bit** is computed. Optional **Data type
   reference** and **Description**.
3. Summarise and wait for go. Click **Create**. Toast **Signal created**; the signal
   appears in the opened row. A refused signal shows the reason in the toast.

**Edit**

- **Edit PDU**: the pencil in the opened row's header (named "Edit PDU <PDU>").
  Name, type, length, **System**, **Frame**, then the **Timing** section at the
  bottom (see **Timing section** above); **Save** (disabled while a Timing field
  is wrong). Toast **PDU updated**. Choosing **No system** removes the PDU's
  system link. An imported PDU's timing shows the note "Imported from the ARXML
  file. Saved on this PDU only when you change it." Don't rename: see Traps.
- **Edit a signal**: the pencil on the signal's line (named "Edit <signal>"). Dialog
  **Edit signal**; **Save**.

## 4. Report

What was created (PDU qualified name, its system, frame, timing in ms) or the
signal (start bit, bit length). Offer the next step: signals, then who sends and receives it
(`send-or-receive-a-pdu.md`).

## Traps

- Length units: the PDU in **bytes**, signals in **bits**. 8 bytes = bits 0–63.
- **Timing units differ by route.** **Milliseconds** in the dialogs and in
  `create_pdu`, `get_pdu` and `list_pdus` (fields named `…Ms`); **seconds** in
  `update_staged_element` (fields `period`, `offset`, `repetitionPeriod`,
  `minimumDelay`). Convert: 100 ms = `0.1`. Sending ms to `update_staged_element`
  stores a period 1000 times too long, without an error.
- **An imported PDU shows its imported timing until it is changed**
  (`timingSource` `imported`): nothing is saved on the PDU until the timing is
  changed and saved. In a model imported before timing existed, imported PDUs
  show only a cyclic period and minimum delay, until the ARXML is imported again.
  A renamed imported PDU shows no imported timing at all.
- **"Required." shows only after leaving a field.** Choosing a Transmission shows
  empty required fields with no message, yet **Save** / **Create** stays disabled.
  If it is greyed out, fill **Period (ms)** or **Repetitions**.
- **Renaming a PDU loses its links** (known, not fixed yet): its system, frame
  and ECU ports find the PDU by its old name, so after a rename it has no system
  and its transmitters and receivers vanish from the table. The Edit PDU dialog
  warns when the name changes. Don't rename a PDU unless the user insists after
  hearing this; `update_staged_element` on a PDU's `shortName` does the same
  damage. If it happened: choose the system and the frame again in Edit PDU,
  then assign the ECUs again.
- Escape or a click outside does **not** close a dialog that holds input: use
  **Cancel**.
- Deleting a PDU or signal isn't possible yet. **Delete <PDU>** (trash icon) only
  shows what a delete would affect; `get_deletion_impact` gives the same list.
  Neither lists the PDU's interfaces, system link, frame mapping, ports or
  connections (they refer to it by name); the dialog says so under the list.
