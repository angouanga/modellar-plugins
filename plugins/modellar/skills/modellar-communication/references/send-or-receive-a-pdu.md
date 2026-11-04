# Make an ECU send or receive a PDU

Set an ECU as a transmitter (Tx) or a receiver (Rx) of a PDU. The app then gives the
ECU a communication port for the PDU and connects it, in the PDU's system, to every
ECU on the other side.

## What has to be true first

1. The PDU has a **system**.
2. The ECU is **placed in that system**. If not: task `place-ecu-in-system.md`.

## 1. Where am I?

- The System Description editor (task `find-pdus-and-signals.md`, section 1), in
  edit mode, works from the PDU. Opened from an **ECU diagram's** dock, the editor
  knows that diagram's ECU (see the visual route).
- **Configuration** → **ECU Types** works from the ECU: its row menu → **Open
  Ports** (dialog **Ports — <ECU>**), or the **ECU Ports** page, lists ECU ports;
  their button **Send / Receive a PDU** is this task. An ECU port is never created
  on its own: it exists because the ECU sends or receives a PDU.

## 2. MCP route (default)

1. `get_pdu({ modelId, pduId })`: its `system`, `transmitters`, `receivers`.
   No system: set one first (task `create-pdu-and-signals.md`, Edit PDU).
2. `list_ecus({ modelId, search: "<ECU name>" })`: the ECU's `id` and `placedIn`.
   Not placed in the PDU's system: task `place-ecu-in-system.md` first.
3. The batch: `list_staging_batches({ modelId })`, the user picks. It only takes
   the PDU's interface when one has to be created; the port goes into the ECU
   type's batch, the connections into the system's.
4. Summarise ("ECU X will transmit PDU P in system S") and wait for go.
5. `assign_pdu_to_ecu({ modelId, batchId, pduId, ecuId, direction: "tx" })` (or
   `"rx"`). The answer has the new `portId` and `connectorIds`: one connection per
   ECU on the other side. `notConnected` names ECUs on the other side that got no
   connection, and why.
6. Refused answers say why: no system, ECU not placed, the ECU already has that
   role, or the ECU type's or system's batch belongs to another user. Nothing was
   staged then.

## 3. Visual route

### From the PDU (System Description)

1. In the PDU's row, click **Add transmitting ECU** (column **Transmitting ECUs**) or
   **Add receiving ECU** (column **Receiving ECUs**). Dialog **Add transmitting ECU**
   (or receiving).
2. **ECU**: searchable list of the ECUs placed in the PDU's system. Opened from an
   ECU diagram, that diagram's ECU is already chosen when it is placed there. When
   it isn't, the dialog says "<ECU>, this diagram's ECU, is not placed in <system>
   (<file>)." with a button **Place <ECU> in <system> (<file>)**: summarise, wait
   for go, click it. Toast **ECU placed in the system**, then the ECU is chosen.
   Two ECUs can share a name ("BCM"): each row shows the ECU's path under its
   name, and so does the chosen one. Check the path before **Assign**.
3. An empty list says why: the PDU has no system, or no ECU is placed in it.
4. Summarise and wait for go. Click **Assign**. It takes a few seconds (the button
   spins). Toast **Assignment added**, or
   **Assignment added — connection created** (one connection), or **Assignment added
   — N connections created**. The ECU appears in the column. A toast description
   names ECUs on the other side that weren't connected.

### From the ECU (Configuration)

1. Menubar **Configuration** → **ECU Types**. Either the ECU's row menu **Actions
   for <ECU>** → **Open Ports** → **Send / Receive a PDU**; or **ECU Types** → **ECU
   Ports** → **Send / Receive a PDU**, which first asks for the ECU (dialog **Send /
   Receive a PDU**, "Choose the ECU type that gets the port.", field **ECU Type**,
   **Continue**).
2. Dialog **Send / Receive a PDU** ("<ECU> gets a port for the PDU, connected to the
   ECUs on the other side. Staged in the ECU type's batch."). **Direction**: **Send
   (Tx)** or **Receive (Rx)**. Changing it clears the PDU.
3. **PDU**: searchable list (**Search PDUs...**) of the PDUs this ECU can take in
   that direction, each with its path under the name. The others are under
   **N PDUs can't be chosen, and why** (click to open), grouped by reason: "This ECU
   already sends it." / "This ECU already receives it.", "This ECU is not placed in
   <system>, the PDU's system." (task `place-ecu-in-system.md`), "The PDU has no
   system. …".
4. Summarise and wait for go. Click **Send** (or **Receive**). Toast **Port
   created**, **Port created — connection created** or **Port created — N
   connections created**; the port appears in the list. On failure: **Could not
   send or receive the PDU** with the reason. Editing a port later (row menu
   **Edit**, dialog **Edit ECU Port**) changes its interface or description only.

## 4. Report

Which ECU now transmits or receives the PDU, in which system, and the connections
made (to which ECUs). Say what is still missing (a receiver, a transmitter).

## Traps

- A PDU may have several transmitters; every transmitter gets a connection to every
  receiver.
- **Remove** (✕ on a badge, named "Remove <ECU> rx" or "… tx") deletes the ECU's
  port for that PDU at once, with no confirmation: ask before clicking it. Its
  system connections stay behind; tell the user.
- On an open System diagram the new port and connections appear by themselves; if
  they don't, **Reload Diagram** (skill `modellar-diagrams`, `refresh-diagram.md`).
