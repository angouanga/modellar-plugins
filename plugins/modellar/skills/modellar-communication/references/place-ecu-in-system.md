# Place an ECU in a system

Make an ECU type part of a system (an ECU prototype), so it can send and receive
that system's PDUs. This is why an ECU is missing from **Add transmitting ECU**.

## 1. Where am I?

- From the System Description editor: the assign dialog offers it for the diagram's
  own ECU (task `send-or-receive-a-pdu.md`, visual route step 2).
- On a **System diagram**: select the system frame; tab **ECU Prototypes** lists the
  ECUs placed in it (heading **ECU Instance Prototypes**).
- In **Configuration** → **System Configuration**: the system's row menu (**Actions
  for <system>**) → **Open ECU Prototypes** lists them in a dialog **ECU Prototypes —
  <system>**.

## 2. MCP route (default)

1. `list_ecus({ modelId, search: "<ECU name>" })`: its `id` and `placedIn`. Already
   placed in the system: nothing to do.
2. `list_systems({ modelId })`: the system's `id` (pick by `sourceFile` when names
   repeat). For a PDU, use the system `get_pdu` returns.
3. Summarise ("place ECU X in system S, file F") and wait for go. No batch to ask
   for: the prototype goes into the system's batch.
4. `place_ecu_in_system({ modelId, ecuId, systemId })`. It fills every
   reference itself. When the ECU is already there it answers `alreadyPlaced: true`
   and stages nothing. When another ECU of the same name is already placed in the
   system, the new placement is named `<ECU>_1` (then `_2` …): its `qualifiedName`
   in the answer says which.

## 3. Visual route

Open the dialog **Create ECU Prototype** one of three ways:

- **System diagram:** open the System diagram of that system (skill
  `modellar-diagrams`), select the system frame, tab **ECU Prototypes**, click **Add
  ECU Prototype** (or **Add your first ECU prototype** when the list is empty).
- **The system's row:** menubar **Configuration** → **System Configuration**, the
  system's row menu **Actions for <system>** → **Open ECU Prototypes** → **New ECU
  Prototype**.
- **The ECU prototypes page:** **Configuration** → **System Configuration** → **ECU
  Prototype** → **New ECU Prototype**. A first dialog **New ECU Prototype** asks for
  the system ("Choose the system to deploy the ECU in."): pick it, **Continue**.

Then:

1. **Short Name** (usually the ECU's own name), **ECU Instance Type** (the ECU),
   optional **Description**. There is no batch field: the line under the form says
   "Deployed in <system path>, staged in its batch."
2. Summarise and wait for go. Click **Create Prototype**. Toast **ECU prototype
   created successfully**.

## 4. Report

The ECU, the system and its file. Offer the next step: making it send or receive the
PDU (`send-or-receive-a-pdu.md`).

## Traps

- Several systems are called "CanSystem": place the ECU in the PDU's own system (its
  file), or it still can't be picked. `list_ecus` gives each placement's
  `systemSourceFile`.
- `stage_element` with `EcuInstancePrototype` needs seven reference fields, two of
  them duplicates: use `place_ecu_in_system` instead.
- **Refused because the system's batch belongs to another user** (an import someone
  else did): nothing was staged. Tell the user; the system's owner has to place it.
