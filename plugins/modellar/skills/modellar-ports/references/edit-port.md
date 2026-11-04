# Edit a port

Change an existing port: its name, direction, port interface or description. The
change applies to the SWC type, so every component of that type sees it.

## 1. Where am I?

URL: `https://<host>/designer/{modelId}/...`. Take `{modelId}` from it.

## 2. Look up

1. `search_elements({ modelId, elementType: "PortPrototype", search: "<port name>", includeElementData: true })`.
   Keep the one whose `qualifiedName` is `<SWC qualifiedName>/<port name>`. Several
   SWCs have a port of that name? Ask which SWC.
2. A new interface? Look it up as in `add-port.md`, step 2.3.
3. A new name? Check that the SWC has no port with that name yet (search again).

## 3. MCP route (default)

1. **Confirm.** Show the port's `qualifiedName`, each field old → new. Wait for go.
2. **Update.** `update_staged_element` with the port's `id`, the
   `updateElementType` and `elementData` that `search_elements` returned (with
   `includeElementData: true` the data already comes in the shape the update
   takes), and `expectedUpdatedAt` = its `updatedAt`. If the result also has
   `updateIssues`, supply the fields it names (`swCaqname` / `swCId` are the
   owning SWC's `qualifiedName` / `id`).

   In that elementData, change only the fields asked for:
   - name: `shortName`;
   - direction: `direction` (`Provider`, `Receiver`, `ProviderReceiver`);
   - interface: `interfaceRef`, `interfaceId` and `interfaceRefDest` together (see
     `add-port.md`, step 3.3, for the tag);
   - text: `description`.

3. The open diagram and its properties panel show the change within about 10
   seconds; no reload needed.

## 4. Visual route

1. Open the SWC's own diagram, select its frame, properties panel, tab **Ports**
   (skill `modellar-diagrams`, `properties-panel.md`).
2. On the port's card, click **Edit port <name>**. The dialog **Edit Port
   Prototype** opens, already filled in.
3. Change only the fields asked for: **Short Name**, **Direction**, **Port
   Interface** (same combobox as in `add-port.md`, step 4.7), **Description**.
4. Summarise old → new and wait for go. Click **Update**; wait until it stops reading
   "Updating...".

Success: **"Port prototype updated successfully"**. Errors: **"Update failed"** (report the text), or **"Cannot update
this port"** (its stored data is broken: tell the user it must be repaired or
re-created).

## 5. Report

The port, what changed (old → new), and on which SWC.

## Traps

- **Changing the direction or interface can break its connectors.** A connector
  joins a provider to a receiver of the same interface. If the port has
  connections (count badges **Assemblies** / **Delegations** on its card above 0),
  tell the user before you change it; the app does not check.
- **Renaming** changes the port's path. Mention its connections, if any, so the
  user can check them afterwards.
- Edit the port on the **type**'s diagram. A component box's Ports tab is view only:
  no **Edit** button there.
- Delete: the button **Delete port <name>** exists, but deleting isn't available
  yet ("Delete not available yet"). Don't promise it.
