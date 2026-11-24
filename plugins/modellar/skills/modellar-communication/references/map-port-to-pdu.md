# Map a component port to a PDU signal

Link a data element of a port to a signal of a PDU that the ECU sends or
receives: a **signal mapping**. On the ECU diagram it is a "Mapping" line between
the port and the PDU on the ECU frame. Two kinds of port can be mapped:

- **(a) a component's port**: a component (prototype) inside the ECU's
  composition. It needs an access point in a runnable of the component, or a
  runnable to create one in.
- **(b) the ECU composition's own port**: a port on the composition frame. No
  access point, no runnable, and no component needs to be connected yet:
  components feed the port later through delegation connectors drawn on the
  composition diagram (skill `modellar-connectors`, `add-connector.md`).

Saving creates a signal-mapping ComSpec and, when needed, the access point (a
only) and the system's root composition `<composition>_Root`.

## What has to be true first

1. The ECU sends or receives the PDU: it has a port for it (`get_pdu`,
   `transmitters` / `receivers`). If not: task `send-or-receive-a-pdu.md`.
2. So the PDU has a system, and the ECU is placed in it (task
   `place-ecu-in-system.md`).
3. The PDU has signals (task `create-pdu-and-signals.md`).
4. The port has a **sender-receiver** interface with data elements.

## 1. Where am I?

- The ECU diagram: the URL contains `/diagram/ecu/<diagramId>`. The ECU frame
  carries one handle per PDU the ECU sends or receives; the composition frame
  inside it carries the composition's own ports, and the component boxes carry
  theirs.
- Open with the dialog **Map port to PDU** ("Map port <port> of <component> to a
  signal of PDU <PDU>.", or "Map port <port> of the ECU composition <composition>
  to a signal of PDU <PDU>."): the user already dragged; continue at visual route
  step 4.
- The ECU frame's tab **Mapping** (skill `modellar-diagrams`, `properties-panel.md`)
  lists the mappings already made, one card per PDU and port, named
  `<PDU>_XIX_<component>_<port>`. A card's port badge reads **Context Port** for a
  component's port and **Composition Port** for the ECU composition's own port.

## 2. MCP route (default)

1. **The ECU**: `list_ecus({ modelId, search: "<ECU name>" })`, its `id`.
2. **The PDU**: `list_pdus({ modelId, ecuId, direction: "tx" })` (or `"rx"`), then
   `get_pdu({ modelId, pduId })`. Keep:
   - the entry for this ECU in `transmitters` (the ECU sends it) or `receivers` (it
     receives it): its `portId` is the `pduPortId`, its `portQualifiedName` is
     used in step 7. No entry: the ECU doesn't send or receive the PDU yet; stop
     and offer `send-or-receive-a-pdu.md`.
   - `signals`: the `id` of the signal to map (`busDataId`).
3. **The ECU's composition** is the ECU type's `softwareComponentRef`
   (`search_elements` with `elementType: "EcuInstanceType"`,
   `includeElementData: true`). Which kind of port did the user name?
   - **(b) a port of that composition itself**:
     `search_elements({ modelId, elementType: "PortPrototype", qualifiedNamePrefix: "<composition qualifiedName>/", includeElementData: true })`,
     its `id` is `compositionPortId`. Check direction and interface as in step 4,
     then step 5 (data element), skip step 6, go on at step 7.
   - **(a) a component's port**: find the component (prototype) with
     `search_elements({ modelId, elementType: "SwComponentPrototype", search: "<name>", includeElementData: true })`,
     and keep the one whose `qualifiedName` starts with the composition's
     `qualifiedName` + `/`: its `id` is `prototypeId`, its type is
     `elementData.softwareComponentRef`.
4. **The port** (a): the ports are the type's.
   `search_elements({ modelId, elementType: "PortPrototype", qualifiedNamePrefix: "<type qualifiedName>/", includeElementData: true })`.
   Keep its `id` (`portId`), `elementData.direction` and `elementData.interfaceRef`.
   - The ECU **sends** the PDU: the port must be `Provider` (or
     `ProviderReceiver`). It **receives** it: `Receiver` (or `ProviderReceiver`).
   - The interface must be a `SenderReceiverInterface`.
5. **The data element**:
   `search_elements({ modelId, elementType: "VariableDataPrototype", qualifiedNamePrefix: "<interface qualifiedName>/" })`,
   its `id` (`variableDataPrototypeId`). A signal and a data element of the same
   name usually belong together; when the names differ, ask.
6. **The access point** (a only; the runnable side):
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
   (start bit, bit length), component.port or "composition port <port> of
   <composition>", data element, the access point or "a new DataSendPoint /
   DataReceivePointByArgument in runnable R" (a only), and that the system's root
   composition will be created if it has none. No batch to ask for: each element
   goes into its parent's batch. Wait for go.
9. The call:
   - (a) `map_port_to_pdu({ modelId, ecuId, pduPortId, prototypeId, portId, variableDataPrototypeId, busDataId, accessPointId })`,
     or with `createAccessPointInRunnableId` instead of `accessPointId` (never
     both).
   - (b) `map_port_to_pdu({ modelId, ecuId, pduPortId, compositionPortId, variableDataPrototypeId, busDataId })`,
     with **no** `prototypeId`, `portId`, `accessPointId` or
     `createAccessPointInRunnableId`: the call is refused otherwise.

   The answer has `comSpecId`, `accessPointId` (null for b), `rootCompositionId`,
   `created` (each new element: the ComSpec, and the access point and root
   composition when they were made) and `warnings`.
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
2. **Show both ends.** The PDU's handle on the ECU frame and the port's handle (on
   the component box, or on the composition frame for its own port) must be
   visible (skill `modellar-diagrams`, `show-or-hide-all.md` and
   `properties-panel.md`, tab **Ports**). A component box must be on the diagram.
3. **Drag** from the port's handle onto the PDU's handle on the ECU frame, or the
   reverse. Handles have no name: they can only be hit by position.
   - Dialog **Map port to PDU** opens: go on.
   - Toast **No connection created** with a reason: nothing was made. Reasons:
     "Drag a component port onto a PDU of the ECU."; "Drag a port of this ECU's
     composition onto one of its PDUs." (a composition frame that isn't this
     ECU's); "Draw connectors between components on the composition diagram. On
     the ECU diagram, drag a component port onto a PDU of the ECU."; "The port
     behind the component handle was not found." / "The port behind the
     composition handle was not found."
4. The dialog first shows "Loading the PDU signals and the port's data
   elements…". A red box instead means the mapping can't be made; it gives the
   reason (Traps) and **Map port** stays disabled: tell the user and **Cancel**.
5. Read the top lines: **PDU** "<PDU> (the ECU sends it)" or "(the ECU receives
   it)", **System**, and **Inner port** when the port leads into a composition. A
   blue note "System <system> has no root composition yet: saving also creates
   <name> for it." means the root composition will be created. For the ECU
   composition's own port, the dialog has no **Access point** or **Runnable for a
   new access point** field (skip step 8).
6. **PDU signal** (**Select a signal**, search **Search signals...**); each row
   shows "bit <start>, <length> bits", or "no position". The signal decides the
   data element: the one of the same name, else the interface's only one.
7. **Data element** (**Select a data element**, search **Search data
   elements...**) appears only when the chosen signal has no data element of its
   name and the interface has several: pick the one the signal carries.
8. A component's port only, then one of:
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

The mapping: ECU, PDU (sent or received), signal, component.port (or the
composition's own port), data element, the access point used or created (and its
runnable), and whether the system's root composition `<composition>_Root` was
created. Say where to see it: the ECU diagram's line, or the ECU frame's tab
**Mapping**. For a composition port, offer the next step: delegation connectors
from the components that feed it (skill `modellar-connectors`, `add-connector.md`).

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
  mapped to any port, component or composition, is refused ("Signal <signal> already has a source …"). A
  **received** signal can be mapped to several ports.
- **The exact same mapping is refused**: "Signal <signal> is already mapped to
  <data element> on port <port>."
- **The ECU composition's own port** is mapped without an access point: an
  `accessPointId` or `createAccessPointInRunnableId` with `compositionPortId` is
  refused ("A port of the ECU composition is mapped without an access point: leave
  accessPointId and createAccessPoint out."), and so is `compositionPortId`
  together with `prototypeId` / `portId`. Other refusals: "Port <port> is not a
  port of <composition>, the ECU's composition."; "The ECU has no composition, so
  it has no composition port."
- **Refused when the system's root composition is typed by another composition**
  than the ECU's (composition port only): "The root composition <root> of system
  <system> is typed by another composition than <composition>, so the port of
  <composition> has no context in that system. Map a port of a component inside
  the composition instead, or change the root composition's type."
- **No access point** for the data element on a component's port: the user picks a runnable
  and a `DataSendPoint` (sent) or `DataReceivePointByArgument` (received) named
  `DSP_<port>_<data element>` / `DRPA_…` is created in it. A component with no
  runnable can't be mapped until one is created.
- **The root composition** `<ECU's composition>_Root` is created for a system that
  has none. When the ECU type has no composition: "System <system> has no root
  composition, and the ECU has no composition to create one from." The component
  must sit inside the system's root composition ("The component is not inside the
  system's root composition …").
- **A port of a composition component** (a box inside the ECU's composition) is followed through its delegation connector
  to the inner port. Refused when it has no delegation ("Connect it on the
  composition diagram first.") or several ("Map the inner component port on its
  own diagram instead.").
- **Refused because a batch belongs to another user** (the PDU port's, the
  runnable's or the system's): "<element>'s staging batch belongs to another user,
  so nothing can be added to it." Every batch is checked before anything is saved.
- **"PDU <PDU> has no signals yet."**: add one first (`create-pdu-and-signals.md`).
- `stage_element` refuses a ComSpec that carries `signalMapping`: always use
  `map_port_to_pdu`.
