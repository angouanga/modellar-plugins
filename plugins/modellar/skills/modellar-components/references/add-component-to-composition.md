# Add a component to a composition

Add a component (an SWC prototype: an instance of an atomic or composition type) to
a composition. This creates the component **in the model only**. It is hidden on
the composition's diagram until it is shown there (skill `modellar-diagrams`, task
`show-component-on-diagram.md`).

**Already in the composition, just not drawn?** Then there is nothing to add:
don't use this task. Go straight to skill `modellar-diagrams`, task
`show-component-on-diagram.md`. A composition often holds components that its
diagram doesn't show yet.

The user names two things:

- **the composition** (e.g. "Zuko"): the parent that gets the new component;
- **the component type** (e.g. "SpeedSensor"): what gets instantiated.

Optionally they also give the **instance name** (the prototype's short name). If
they didn't, propose `<TypeName>_Proto` or ask. It must start with a letter and use
only letters, digits and `_`.

## 1. Where am I?

URL: `https://<host>/designer/{modelId}/...`. Take `{modelId}` from it.

## 2. Look up and check

1. Call `search_elements({ modelId, elementType: "SwComponent", search: "<name>", includeElementData: true })`
   once for the composition and once for the component type.
   - The composition's `elementData.type` must be `Composition` (or
     `RootComposition`). If it's anything else, tell the user.
   - If the component type doesn't exist, offer to create it first (task
     `create-swc-type.md` in this skill).
   - Several matches: ask which one, showing each `qualifiedName`.
2. **No duplicate instance.** Call
   `search_elements({ modelId, elementType: "SwComponentPrototype", search: "<instance name>" })`.
   If a match's `qualifiedName` is `<the composition's qualifiedName>/<instance name>`,
   stop and tell the user.

## 3. MCP route (default)

Use it when `search_elements` returned a `batchId` for the composition. If it
didn't (an older connector), use the visual route.

1. **The batch: the composition's.** A component inside a composition goes into the
   composition's staging batch, as the diagram does. Take the `batchId` that
   `search_elements` returned for the composition in section 2. Don't ask the user.
2. **Confirm.** Summarise the instance name, the type's `qualifiedName` and the
   composition, and wait for the user's go.
3. **Stage it.** `describe_element_type({ elementType: "SwComponentPrototype" })` if
   you haven't yet, then `stage_element` with:
   - `elementType`: `"SwComponentPrototype"`
   - `absoluteQualifiedName`: `<the composition's qualifiedName>/<instance name>`
   - `elementData`: `shortName` = the instance name; `softwareComponentRef` and
     `softwareComponentId` = the type's `qualifiedName` and `id`;
     `parentCompositionRef` and `parentCompositionId` = the composition's
     `qualifiedName` and `id`; `sourceFile` = the composition's
     `elementData.sourceFile`; `description` if the user gave one.
   - `idempotencyKey`: e.g. `"proto-<instance name>-1"`.
4. **Next.** The component exists but isn't on the diagram yet. If the user wants
   to see it, continue with skill `modellar-diagrams`, task
   `show-component-on-diagram.md`.

## 4. Visual route

For when the user asks to do it in the UI, or the MCP route isn't available.

1. Get onto the composition's diagram (skill `modellar-diagrams`, task
   `open-or-create-swc-diagram.md`).
2. Open the **Components** panel as described in task
   `show-component-on-diagram.md`, section 3.
3. **Check for duplicates.** If the instance name is already in the list, stop and
   tell the user.
4. Click **Add Component**. If the list is empty, the button reads **Add your
   first component** instead.
5. The dialog **Add Component to <Composition>** opens.
6. **Short Name \*.** The instance name.
7. **Software Component Type \*.** Click the combobox **Select a component type...**,
   type the type's short name into the search box, and pick the row whose grey path
   equals its `qualifiedName`. Atomic and composition types are both listed; the
   type chip narrows the list. Root compositions are never offered.
8. **Description.** Optional.
9. Summarise the values and wait for go. Then click **Create Prototype** and wait
   until it stops reading "Creating...".

Success shows the toast **"SW component prototype staged successfully"**. The new card
has the footer button **Add** (its grey status icon's tooltip reads "Not in diagram").
To show it, continue with task
`show-component-on-diagram.md`, section 5.

## 5. Report

Tell the user the instance name, its type (qualified name) and the composition, and
whether it is shown on the diagram yet.

## Traps

- If `stage_element` returns validation issues, nothing was written. Fix exactly
  those fields and call again with the **same** idempotency key.
- After an error from `stage_element`, search for the instance before retrying. It
  may have been written anyway.
- **"Staging batch not found"** means the composition sits in another user's batch,
  which only its owner can write into. Tell the user; don't try another batch.
