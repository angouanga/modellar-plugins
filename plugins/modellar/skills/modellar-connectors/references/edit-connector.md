# Edit a connector

Rename a connector, or move one of its ends to another component or port. Its kind
(assembly or delegation) can't change: to change the kind, a new connector is
needed (task `add-connector.md`).

## 1. Where am I?

URL: `https://<host>/designer/{modelId}/...`. Take `{modelId}` from it.

## 2. Look up and check

1. `search_elements({ modelId, elementType: "SwConnector", search: "<name>", includeElementData: true })`.
   Connectors are stored as `SwConnector`; the result's `updateElementType`
   (`AssemblySwConnector` or `DelegationSwConnector`) says the kind. Keep the one
   in the right composition.
2. A new end? Look up the component and port as in `add-connector.md`, steps
   2.2–2.3, and check the match (step 2.4) against the end that stays.
3. A new name? Check no other connector of the composition has it.

## 3. MCP route (default)

1. **Confirm.** The connector's `qualifiedName`, each field old → new. Wait for go.
2. `update_staged_element` with its `id`, the `updateElementType`
   (`AssemblySwConnector` or `DelegationSwConnector`) and the `elementData` that
   `search_elements` returned (already in the shape the update takes), with only
   the asked fields changed, and `expectedUpdatedAt` = its `updatedAt`. A moved
   end changes its ref **and** id fields together (see `add-connector.md`, step
   3.3). For a delegation whose inner port changes direction, move the inner
   prototype id to the matching field. If a required field is missing, the
   update says which (or the search result already lists them in
   `updateIssues`): `parentQname` and `sourceFile` are the composition's
   `qualifiedName` and `sourceFile`.
3. A line already drawn for it keeps its old ends until the diagram is refreshed
   (skill `modellar-diagrams`, `refresh-diagram.md`).

## 4. Visual route

1. Composition diagram → select its frame → properties panel, tab
   **Connections** (skill `modellar-diagrams`, `properties-panel.md`).
2. On the connector's card, click **Edit connector <name>**. The dialog **Edit
   Connector in <composition>** opens, filled in. **Connection Type** is locked
   ("Connection type is pre-selected and cannot be changed").
3. Change **Short Name** or the end comboboxes (as in `add-connector.md`, step
   4.7). Check the interface of a new port matches.
4. Summarise old → new and wait for go. Click **Update Connector**; wait until it
   stops reading "Updating...".

Success: **"Connector updated successfully"**. If an end moved and the line is drawn, refresh the diagram to see
it redrawn.

## 5. Report

The connector, what changed (old → new), and whether a refresh is needed to see it.

## Traps

- **The edit form doesn't require the ends.** It lets you save with an end
  emptied. Check every end is still set before you click **Update Connector**.
- No interface check in the app: yours.
- Delete: **Delete connector <name>** exists, but deleting isn't available yet
  ("Delete not available yet").
