# List, add or edit a system's connections

A system **connection** links a port of one ECU prototype to a port of another, in
one system (an ECU-to-ECU connector). Most are made for you: when an ECU sends or
receives a PDU, the app connects it to every ECU on the other side (task
`send-or-receive-a-pdu.md`). Use this task to see them, to fix one, or to add one
by hand.

## 1. Where am I?

- **Configuration** → **System Configuration**: the system's row menu **Actions for
  <system>** → **Open Connections** opens **Connections — <system>**, only that
  system's connections.
- **Configuration** → **System Configuration** → **Connections**: every system's
  connections, one table.
- On a **System diagram**: the system frame's tab **Connections** (skill
  `modellar-diagrams`, `properties-panel.md`).

## 2. MCP route (default)

- **List:** `search_elements({ modelId, elementType: "SwConnector", qualifiedNamePrefix: "<system path>/" })`.
  A system connection's qualified name starts with its system's. Add
  `includeElementData: true` (at most 10 per call) for its ends:
  `providingEcuInstanceProtoRef` / `providingPortRef` and
  `requiringEcuInstanceProtoRef` / `requiringPortRef`.
- **Add:** prefer `assign_pdu_to_ecu` (task `send-or-receive-a-pdu.md`): it makes
  the port and every connection the PDU needs. There is no tool for a single bare
  connection; use the visual route.
- **Edit:** `update_staged_element` as for any element (skill
  `modellar-configuration`, `edit-element.md`).

## 3. Visual route

### Add

1. **Open Connections** on the system's row → **New Connection**. From the
   **Connections** page, **New Connection** first asks for the system (dialog **New
   Connection**, "Choose the system the connection belongs to."): pick it,
   **Continue**.
2. Dialog **Create System Connector** ("Connect two ECU instances deployed in this
   system."). **Short Name**, **Providing ECU Prototype** and its **Providing
   Port**, **Requiring ECU Prototype** and its **Requiring Port**. Only that system's
   ECU prototypes are offered; picking an ECU clears its port. The line under the
   form reads "Owned by <system path>". No batch field: it goes into the system's
   batch.
3. Summarise and wait for go. **Create Connector**. Toast **System connector
   created successfully**.

### Edit

The connection's row menu → **Edit**. Dialog **Edit System Connector**, with both
ends filled in. Change, summarise, wait for go, **Update Connector**. Toast **System
connector updated successfully**.

## 4. Report

The connection's name, its two ends (ECU and port each), and its system.

## Traps

- **"Cannot open the connection form"** (with a reason): the system or its ECU
  prototypes couldn't be loaded. Try again; report the reason if it stays.
- **"Cannot edit this connection"** / "It is not linked to a system.": an imported
  connection without a system. It can't be edited here.
- **"The system has no source file."** on create: a system needs one. Tell the
  user; nothing was saved.
- **Refused because the system's batch belongs to another user**: nothing was
  saved. Tell the user.
- Deleting a connection isn't available.
