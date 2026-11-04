# Add a port to an SWC

Create a port (a port prototype) on an SWC type, atomic or composition. This
creates the port **in the model only**: it is hidden on every diagram until it is
shown (task `show-hide-or-move-ports.md`).

The user names:

- **the SWC** that gets the port (e.g. "SpeedSensor", or the composition "Zuko");
- **the port name** (short name: starts with a letter, then letters, digits and
  `_`; not a C/C++ keyword such as `int` or `class`);
- **the direction**: provider (sends, serves), receiver (reads, calls) or both
  (`ProviderReceiver`). Ask if it isn't clear;
- **the port interface** it uses (e.g. "If_VehicleSpeed").

## 1. Where am I?

URL: `https://<host>/designer/{modelId}/...`. Take `{modelId}` from it.

## 2. Look up and check

1. **The SWC.** `search_elements({ modelId, elementType: "SwComponent", search: "<SWC name>", includeElementData: true })`.
   Keep its `id`, `qualifiedName`, `batchId` and `elementData.sourceFile`. Several
   matches: ask which one, showing each `qualifiedName`.
   - The user named a **component in a composition** (a prototype)? A port belongs
     to the prototype's **type**: look up the prototype
     (`elementType: "SwComponentPrototype"`, `includeElementData: true`), take its
     type from `softwareComponentRef`, tell the user the port goes on that type
     (every instance of it gets the port), and continue with the type.
2. **No duplicate.** `search_elements({ modelId, elementType: "PortPrototype", search: "<port name>" })`.
   A match whose `qualifiedName` is `<SWC qualifiedName>/<port name>` already
   exists: stop and tell the user.
3. **The interface.** `search_elements` with the interface's kind as
   `elementType`: `SenderReceiverInterface`, `ClientServerInterface`,
   `ParameterInterface`, `ModeSwitchInterface`, `TriggerInterface` or
   `NvDataInterface`. Unsure of the kind? Search without `elementType` and keep the
   results whose `elementType` ends in `Interface`. Keep its `id`,
   `qualifiedName` and `elementType`. Not found: it must be created
   first (skill `modellar-configuration`, task `create-element.md`).

## 3. MCP route (default)

1. **The batch: the SWC's.** A port goes into its SWC's staging batch: the
   `batchId` from step 2.1. Don't ask the user. No `batchId` returned (an older
   connector)? Use the visual route.
2. **Confirm.** Summarise: SWC (`qualifiedName`), port name, direction, interface
   (`qualifiedName`). Wait for go.
3. **Stage it.** `describe_element_type({ elementType: "PortPrototype" })` if you
   haven't yet, then `stage_element` with:
   - `elementType`: `"PortPrototype"`
   - `absoluteQualifiedName`: `<SWC qualifiedName>/<port name>`
   - `elementData`: `shortName`; `direction` = `Provider`, `Receiver` or
     `ProviderReceiver`; `swCaqname` and `swCId` = the SWC's `qualifiedName` and
     `id`; `interfaceRef` and `interfaceId` = the interface's `qualifiedName` and
     `id`; `interfaceRefDest` = the interface's kind as an AUTOSAR tag
     (`SenderReceiverInterface` → `SENDER-RECEIVER-INTERFACE`,
     `ClientServerInterface` → `CLIENT-SERVER-INTERFACE`, and so on: split the
     words, upper case, hyphens); `sourceFile` = the SWC's `elementData.sourceFile`;
     `description` if given. Leave `context` out.
   - `idempotencyKey`: e.g. `"port-<SWC name>-<port name>-1"`.
4. **Next.** The port exists but is hidden on every diagram. If the user wants to
   see it, continue with `show-hide-or-move-ports.md`.

## 4. Visual route

For when the user asks to do it in the UI, or the MCP route isn't available.

1. Open the **SWC's own diagram** (skill `modellar-diagrams`, task
   `open-or-create-swc-diagram.md`). A component box on a composition diagram can't
   get ports: its Ports tab is "(View Only)".
2. Select the SWC's frame (click its header) and open the properties panel, tab
   **Ports** (skill `modellar-diagrams`, `properties-panel.md`). The heading reads
   **Frame Ports**.
3. **Check for duplicates.** If the port name is already a card in the list, stop.
   Clear any filter first (badge on **Filter**).
4. Click **Add Port** (empty list: **Add your first port**). The dialog **Create
   Port Prototype** opens. If nothing opens, see Traps.
5. **Short Name**: the port name.
6. **Direction**: the select, default **Provider**. Options **Provider**,
   **Receiver**, **ProviderReceiver**.
7. **Port Interface**: click the combobox **Select a port interface...** (older
   versions: **Select a SWC interface...**). In the
   popover, type the interface name into **Search by name, short name, or
   description...** (the select **Filter by type** narrows by kind, for example
   **S/R Interface**; each row shows its kind the same way). Pick the row with the right name. The list grows as
   you scroll; "No port interfaces found." (or "No SWC interfaces found.") means no
   match.
8. **Description**: optional.
9. Summarise the values and wait for go. Then click **Create** and wait until it
   stops reading "Creating...".

Success shows the toast **"Port prototype created successfully"**. The new card appears in the list, dimmed: it isn't drawn.
To draw it, continue with `show-hide-or-move-ports.md`.

**Without a diagram:** menubar **Configuration** → **SW Component Types**, the SWC's
row menu **Actions for <SWC>** → **Open Ports** (dialog **Ports — <SWC>**) → **Add
Port Prototype**. Dialog **Create Port Prototype** with the SWC already set; fill
steps 5–9 the same way. The port is in the SWC's batch and not on any diagram.

Errors: **"Fix 1 field before saving"** (a field is red: read its message under
it, such as "Please select an interface" or "Short name must begin with a letter
and contain only letters, numbers and underscores"); **"Creation failed"** (the
server refused it: report the text).

## 5. Report

The port's name, direction and interface, the SWC it was added to, and that it is
not shown on the diagram yet. Offer to show it.

## Traps

- **Duplicates are created silently.** Always check step 2.2 / 4.3 first.
- **Add Port does nothing** when the SWC has no staging batch. No message appears.
  Use the MCP route, or tell the user.
- The port goes on the **type**: every component of that type gets it.
- After an error from `stage_element`, search for the port before retrying. It may
  have been written anyway. Retry with the **same** idempotency key.
- **"Staging batch not found"** (MCP), or the toast "The staging batch belongs to
  another user or no longer exists, so nothing was saved." (form), means the SWC
  sits in another user's batch. Tell the user; don't try another batch.
- The dock lists no **Add Port** button: use the Ports tab.
