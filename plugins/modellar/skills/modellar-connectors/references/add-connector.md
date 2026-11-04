# Add a connector

Wire two ports inside a composition:

- **Assembly**: component A's provider port → component B's receiver port, for
  example "connect SpeedSensor's VehicleSpeed to Dashboard's VehicleSpeed".
- **Delegation**: a component's port ↔ the composition's own port, for example
  "route Zuko's outer port Speed_Out to SpeedSensor's VehicleSpeed".

This creates the connector **in the model only**. It is not drawn on the diagram
yet: task `show-or-hide-connections.md` does that.

## 1. Where am I?

URL: `https://<host>/designer/{modelId}/...`. Take `{modelId}` from it.

## 2. Look up and check

1. **The composition.** `search_elements({ modelId, elementType: "SwComponent", search: "<name>", includeElementData: true })`;
   its `elementData.type` must be `Composition` (or `RootComposition`). Keep `id`,
   `qualifiedName`, `batchId`, `elementData.sourceFile`. The user named only the
   components? Find their composition from a component's `qualifiedName` (its
   parent path).
2. **The components** (prototypes) inside it:
   `search_elements({ modelId, elementType: "SwComponentPrototype", search: "<name>", includeElementData: true })`.
   Keep the ones whose `qualifiedName` starts with the composition's
   `qualifiedName` + `/`. Keep `id`, `qualifiedName` and the type from
   `elementData.softwareComponentRef`. Not in the composition: stop, offer skill
   `modellar-components`, `add-component-to-composition.md`.
3. **The ports.** A component's ports are its **type**'s ports:
   `search_elements({ modelId, elementType: "PortPrototype", search: "<type qualifiedName>/", includeElementData: true })`,
   keep those whose `qualifiedName` is `<type qualifiedName>/<port name>`. For a
   delegation, the outer port is the composition's own port
   (`<composition qualifiedName>/<port name>`). Keep each port's `id`,
   `qualifiedName`, `elementData.direction`, `elementData.interfaceRef`. The user
   didn't name the ports? List the candidates (step 4) and ask.
4. **Do they match?** The app doesn't check this; you do. If not, tell the user
   and stop:
   - **Same interface**: both ports' `interfaceRef` are equal.
   - **Assembly**: the providing port is `Provider` or `ProviderReceiver`; the
     requiring port is `Receiver` or `ProviderReceiver`; two different components.
   - **Delegation**: inner and outer port have the same `direction`.
5. **Not connected yet.** Connectors are **stored** as `elementType: "SwConnector"`
   (assembly and delegation alike), whatever type staged them. Search with
   `search_elements({ modelId, elementType: "SwConnector", search: "<port name>", includeElementData: true, limit: 50 })`:
   connector names usually contain the ports' names. The stored ends are
   `providerPortRef` / `requesterPortRef` (assembly) or `innerPortRef` /
   `outerPortRef` (delegation). If one already joins the same two ports, stop
   and tell the user. Also no connector of the chosen name. A search returns at
   most 50: if it does, narrow it rather than conclude "none".
6. **Name.** If the user gave none, propose the one the diagram would make:
   - assembly `Asc_<ProvComponent><ProvPort>_<ReqComponent><ReqPort>`;
   - delegation `Dc_<InnerComponent><InnerPort>_<Composition><OuterPort>`;
     letters, digits and `_` only.

## 3. MCP route (default)

1. **The batch: the composition's** (`batchId` from step 2.1). Don't ask the user.
   None returned? Use the visual route.
2. **Confirm.** Summarise kind, name, both ends (component.port) and the shared
   interface. Wait for go.
3. **Stage it.** `describe_element_type` for the kind if you haven't yet, then
   `stage_element` with `absoluteQualifiedName` = `<composition qualifiedName>/<name>`,
   `idempotencyKey` e.g. `"conn-<name>-1"`, and:
   - **Assembly**, `elementType: "AssemblySwConnector"`, `elementData`:
     `shortName`; `parentQname` = composition `qualifiedName`;
     `providerComponentRef` / `requesterComponentRef` = the two **prototypes'**
     `qualifiedName`; `providingSwcProtoId` / `requiringSwcProtoId` = their `id`;
     `providerPortRef` / `requesterPortRef` = the two **ports'** `qualifiedName`;
     `providerPortId` and `providingPortId` = the provider port's `id`;
     `requesterPortId` and `requiringPortId` = the receiver port's `id`;
     `sourceFile` = the composition's.
   - **Delegation**, `elementType: "DelegationSwConnector"`, `elementData`:
     `shortName`; `parentQname`; `componentRef` = the inner prototype's
     `qualifiedName`; `innerPortRef` / `innerPortId` = the inner port;
     `outerPortRef` / `outerPortId` = the composition's port;
     `innerPortDirection` = the inner port's direction; the inner prototype's `id`
     in `providingSwcProtoId` if that direction is `Provider`, otherwise in
     `requiringSwcProtoId`; `sourceFile`.
4. **Next.** It is not drawn. If the user wants to see it, continue with
   `show-or-hide-connections.md`.

## 4. Visual route

1. Open the composition's diagram (skill `modellar-diagrams`,
   `open-or-create-swc-diagram.md`). Select its frame (click the header) and open
   the properties panel, tab **Connections** (`properties-panel.md`). Heading **SW
   Connectors**.
2. **Check for duplicates** in the list (filter **Connector Type**, or **Search by
   name**). Clear the filter afterwards.
3. Click **Add Connector** (empty list: **Add your first connector**). The dialog
   **Create Connector in <composition>** opens.
4. **Short Name \***: the name.
5. **Parent Composition** is shown, read only. Check it is the right one.
6. **Connection Type \***: the radio **Assembly Connector - Connect two component
   prototypes (SWC-to-SWC)** (default) or **Delegation Connector - Connect inner
   port to outer composition boundary**. Pick it **before** the ends: switching
   clears them.
7. The ends. Each is a combobox: click it, type into the search box (**Search
   prototypes...** / **Search ports...**), pick the row. A port combobox appears
   only after its component is picked, and shows each port's direction and
   interface.
   - **Assembly**: **Providing Component**, **Providing Port**, then **Requiring
     Component**, **Requiring Port**.
   - **Delegation**: **Outer Port** (the composition's port), then **Inner
     Component**, **Inner Port**.
     After a port is picked, its interface shows under **Port Interface:**. Check
     both ends show the same interface (step 2.4).
8. Summarise and wait for go. Click **Create Connector**; wait until it stops
   reading "Creating...".

Success: **"Assembly connector staged successfully"** (or Delegation). The new
card's button reads **Add**
("Not in diagram"). Errors: **"Fix 1 field before saving"** (read the red message,
such as "Requiring port is required for assembly connectors"), or **"Error"** with
the server's text.

**Drawing instead.** Dragging from one component's port dot to another's on the
canvas opens the same dialog already filled in (type locked, name proposed), and
draws the line after **Create Connector**. Port dots can only be hit by position,
so prefer the panel. If the drag doesn't open the dialog, a bare line was drawn
that is not a connector: reload the diagram to drop it.

## 5. Report

Kind, name, both ends, the shared interface, and that it isn't drawn yet (unless it
was dragged). Offer to show it.

## Traps

- **No interface check in the app**: a connector between different interfaces is
  accepted and is wrong. Step 2.4 is yours.
- **Duplicates are created silently**: step 2.5.
- A line drawn by dragging, or with the card's **Add**, is **not saved** until
  **Save Diagram**. The connector element itself is saved at once.
- After an error from `stage_element`, search before retrying, then retry with the
  **same** idempotency key.
- **"Staging batch not found"**: the composition is in another user's batch. Tell
  the user.
