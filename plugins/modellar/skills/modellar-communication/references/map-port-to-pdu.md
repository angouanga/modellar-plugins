# Map a component port to a PDU signal

Link a data element of a software component's port to a signal of a PDU that the
ECU sends or receives: a **signal mapping**. On the ECU diagram it is a "Mapping"
line between the component's port and the PDU on the ECU frame. Saving one creates
a signal-mapping ComSpec and, when needed, an access point in a runnable of the
component and the system's root composition `<composition>_Root`.

## What has to be true first

1. The ECU sends or receives the PDU: it has a port for it (`get_pdu`,
   `transmitters` / `receivers`). If not: task `send-or-receive-a-pdu.md`.
2. So the PDU has a system, and the ECU is placed in it (task
   `place-ecu-in-system.md`).
3. The PDU has signals (task `create-pdu-and-signals.md`).
4. The port has a **sender-receiver** interface with data elements.

## 1. Where am I?

- The ECU diagram: the URL contains `/diagram/ecu/<diagramId>`. The ECU frame
  carries one handle per PDU the ECU sends or receives; the component boxes inside
  it carry their ports.
- Open with the dialog **Map port to PDU** ("Map port <port> of <component> to a
  signal of PDU <PDU>."): the user already dragged; continue at visual route step 4.
- The ECU frame's tab **Mapping** (skill `modellar-diagrams`, `properties-panel.md`)
  lists the mappings already made, one card per PDU and port, named
  `<PDU>_XIX_<component>_<port>`.

## 2. MCP route (default)

1. **The ECU**: `list_ecus({ modelId, search: "<ECU name>" })`, its `id`.
2. **The PDU**: `list_pdus({ modelId, ecuId, direction: "tx" })` (or `"rx"`), then
   `get_pdu({ modelId, pduId })`. Keep:
   - the entry for this ECU in `transmitters` (the ECU sends it) or `receivers` (it
     receives it): its `portId` is the `pduPortId`, its `portQualifiedName` is
     used in step 7. No entry: the ECU doesn't send or receive the PDU yet; stop
     and offer `send-or-receive-a-pdu.md`.
   - `signals`: the `id` of the signal to map (`busDataId`).
3. **The component** (prototype): the ECU's composition is the ECU type's
   `softwareComponentRef` (`search_elements` with `elementType: "EcuInstanceType"`,
   `includeElementData: true`). Then
   `search_elements({ modelId, elementType: "SwComponentPrototype", search: "<name>", includeElementData: true })`,
   keep the one whose `qualifiedName` starts with the composition's
   `qualifiedName` + `/`: its `id` is `prototypeId`, its type is
   `elementData.softwareComponentRef`.
4. **The port**: the ports are the type's.
   `search_elements({ modelId, elementType: "PortPrototype", qualifiedNamePrefix: "<type qualifiedName>/", includeElementData: true })`.
   Keep its `id` (`portId`), `elementData.direction` and `elementData.interfaceRef`.
   - The ECU **sends** the PDU: the port must be `Provider` (or
     `ProviderReceiver`). It **receives** it: `Receiver` (or `ProviderReceiver`).
   - The interface must be a `SenderReceiverInterface`.
5. **The data element**:
   `search_elements({ modelId, elementType: "VariableDataPrototype", qualifiedNamePrefix: "<interface qualifiedName>/" })`,
   its `id` (`variableDataPrototypeId`). A signal and a data element of the same
   name usually belong together; when the names differ, ask.
6. **The access point** (the runnable side):
   `search_elements({ modelId, elementType: "AccessPoint", qualifiedNamePrefix: "<type qualifiedName>/", includeElementData: true })`.
   Keep those whose `portId` is the port and whose `variableDataPrototypeId` is the
   data element, with an access type of the PDU's direction: `DataWriteAccess` or
   `DataSendPoint` when the ECU sends; `DataReadAccess`,
   `DataReceivePointByValue` or `DataReceivePointByArgument` when it receives.
   - One or more: the user picks one (`accessPointId`).
   - None: a new one is created. The user picks the runnable:
     `search_elements({ modelId, elementType: "Runnable", qualifiedNamePrefix: "<type qualifiedName>/" })`
     (`createAccessPointInRunnableId`). No runnable: stop; one has to be created
     first (skill `modellar-configuration`, `create-element.md`).
   - For a **composition** component, the tool follows the port's delegation
     connector to the inner component's port: look up access points and runnables
     on that inner component instead.
7. **Not mapped yet**:
   `search_elements({ modelId, elementType: "ComSpec", qualifiedNamePrefix: "<PDU portQualifiedName>/" })`
   lists the mappings on this PDU port; each name contains its signal's path.
   The ECU sends the PDU and the signal already has one: stop (see Traps).
8. **Summary and go**: ECU, PDU and whether the ECU sends or receives it, signal
   (start bit, bit length), component.port, data element, the access point or "a
   new DataSendPoint / DataReceivePointByArgument in runnable R", and that the
   system's root composition will be created if it has none. No batch to ask for:
   each element goes into its parent's batch. Wait for go.
9. `map_port_to_pdu({ modelId, ecuId, pduPortId, prototypeId, portId, variableDataPrototypeId, busDataId, accessPointId })`,
   or with `createAccessPointInRunnableId` instead of `accessPointId` (never both).
   The answer has `comSpecId`, `accessPointId`, `rootCompositionId`, `created`
   (each new element: the ComSpec, and the access point and root composition when
   they were made) and `warnings`.
10. Refused: the answer says why and "Nothing was staged." A failure partway lists
    what was "Already staged before the failure (don't stage these again)": the
    same call again reuses that access point; don't create one by hand.

**Then show it** on the ECU diagram (skill `modellar-navigation`):
`list_diagrams({ modelId, stagedElementId: "<ECU id>" })` → `/designer/{modelId}/diagram/ecu/{diagramId}`.
The tool doesn't draw the line. On a diagram already open, **Reload Diagram**
first (skill `modellar-diagrams`, `refresh-diagram.md`). Then the ECU frame's tab
**Mapping**, the card `<PDU>_XIX_<component>_<port>`, its button **Show
connection**, then **Save Diagram**.

## 3. Visual route

1. **Open the ECU's diagram.** By address (MCP route, "Then show it"), or menubar
   **Diagram** → **Manage ECU** → dialog **Manage ECU Diagrams** → type the ECU's
   name in **Search existing diagrams...** → **Open <name>**. Never the trash icon
   next to it ("Delete diagram <name>"). No ECU diagram yet: ask the user before
   **Create Diagram**.
2. **Show both ends.** The PDU's handle on the ECU frame and the port's handle on
   the component box must be visible (skill `modellar-diagrams`,
   `show-or-hide-all.md` and `properties-panel.md`, tab **Ports**). The component
   box must be on the diagram.
3. **Drag** from the component's port handle onto the PDU's handle on the ECU frame,
   or the reverse. Handles have no name: they can only be hit by position.
   - Dialog **Map port to PDU** opens: go on.
   - Toast **No connection created** with a reason: nothing was made. Reasons:
     "Drag a component port onto a PDU of the ECU."; "A PDU is mapped to a port of
     a component inside the composition, not to the composition's own port.";
     "Draw connectors between components on the composition diagram. On the ECU
     diagram, drag a component port onto a PDU of the ECU."
4. The dialog first shows "Loading the PDU signals and the port's data
   elements…". A red box instead means the mapping can't be made; it gives the
   reason (Traps) and **Map port** stays disabled: tell the user and **Cancel**.
5. Read the top lines: **PDU** "<PDU> (the ECU sends it)" or "(the ECU receives
   it)", **System**, and **Inner port** when the port leads into a composition. A
   blue note "System <system> has no root composition yet: saving also creates
   <name> for it." means the root composition will be created.
6. **Data element** (**Select a data element**, search **Search data
   elements...**). Filled in when the interface has only one.
7. **PDU signal** (**Select a signal**, search **Search signals...**); each row
   shows "bit <start>, <length> bits", or "no position". The signal of the same
   name as the data element is filled in: check it.
8. Then one of:
   - **Access point** (**Select an access point**): the access points that write
     (sent PDU) or read (received PDU) the data element on that port. Filled in
     when there is one.
   - **Runnable for a new access point** (**Select a runnable**), when there is
     none: "No access point writes <data element> on port <port>. Saving creates a
     DataSendPoint in this runnable." (or "reads" … DataReceivePointByArgument). An
     empty list reads "<component> has no runnables. Create one first."
9. Summarise and wait for go. Click **Map port** (it reads **Mapping…** while it
   works). Red messages under a field stop it: "Choose a data element", "Choose a
   signal", "Choose an access point", "Choose the runnable to create the access
   point in", "The ECU already sends this signal from a mapped port; a sent signal
   has one source", "This signal is already mapped to this data element".
10. Success: toast **Port mapped to PDU signal**, its description naming what else
    was made ("root composition created, access point created."). The line is
    drawn and saved by itself. Failure: toast **Port not mapped** with the reason,
    and "Already staged: …" when part of it was saved.

## 4. Report

The mapping: ECU, PDU (sent or received), signal, component.port, data element, the
access point used or created (and its runnable), and whether the system's root
composition `<composition>_Root` was created. Say where to see it: the ECU
diagram's line, or the ECU frame's tab **Mapping**.

## Traps

- **A mapping can't be changed, and can't be deleted yet.** The **Edit** button is
  gone from the mapping cards and the line's properties. The ComSpec form opens a
  signal-mapping ComSpec read only, and `update_staged_element` refuses it. The
  app says "delete it and map the port again", but **Delete SignalMapping <name>**
  only opens **Delete SignalMapping** with **Delete (not available yet)**,
  disabled. Get the summary right before go.
- **The ECU must be placed in the PDU's system** and send or receive the PDU.
  Refusals: "The ECU is not placed in system <system>, the PDU's system. …";
  "PDU <PDU> has no system. …".
- **Direction**: a sent PDU (the ECU transmits) needs a **provider** port, a
  received PDU a **receiver** port. Otherwise: "Port <port> is a receiver port,
  but the ECU sends this PDU: map a provider port to it." (or the reverse). Only a
  sender-receiver interface can be mapped.
- **A sent signal has one source**: when the ECU sends the PDU, a signal already
  mapped to any port is refused ("Signal <signal> already has a source …"). A
  **received** signal can be mapped to several ports.
- **The exact same mapping is refused**: "Signal <signal> is already mapped to
  <data element> on port <port>."
- **No access point** for the data element on that port: the user picks a runnable
  and a `DataSendPoint` (sent) or `DataReceivePointByArgument` (received) named
  `DSP_<port>_<data element>` / `DRPA_…` is created in it. A component with no
  runnable can't be mapped until one is created.
- **The root composition** `<ECU's composition>_Root` is created for a system that
  has none. When the ECU type has no composition: "System <system> has no root
  composition, and the ECU has no composition to create one from." The component
  must sit inside the system's root composition ("The component is not inside the
  system's root composition …").
- **A composition component's port** is followed through its delegation connector
  to the inner port. Refused when it has no delegation ("Connect it on the
  composition diagram first.") or several ("Map the inner component port on its
  own diagram instead.").
- **Refused because a batch belongs to another user** (the PDU port's, the
  runnable's or the system's): "<element>'s staging batch belongs to another user,
  so nothing can be added to it." Every batch is checked before anything is saved.
- **"PDU <PDU> has no signals yet."**: add one first (`create-pdu-and-signals.md`).
- `stage_element` refuses a ComSpec that carries `signalMapping`: always use
  `map_port_to_pdu`.
