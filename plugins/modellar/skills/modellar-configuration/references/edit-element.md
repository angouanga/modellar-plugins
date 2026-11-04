# Edit a configuration element

Rename an element or change its fields: a data type's category, an interface's
description, a runnable's minimum start interval, an RTE event's period, and so on.
Open `element-catalog.md` at the type's row first.

## 1. Where am I?

URL: `https://<host>/designer/{modelId}/...`. Take `{modelId}` from it. On an edit
page (`…/configuration/<section>/<elementId>/edit`), `<elementId>` is the element
being edited.

## 2. Look up

1. `search_elements({ modelId, elementType: "<type>", search: "<full short name>", includeElementData: true })`.
   Use the whole name: a short text matches inside other names too, and only the
   first 10 results carry `elementData`. Keep the element's `id`,
   `qualifiedName`, `updatedAt` and `elementData`.
   Several matches: ask which one, showing each `qualifiedName`.
2. A **new reference** (another data type, interface, mode group …): look it up
   too, and keep its `id` and `qualifiedName`.
3. A **new name**: check that no sibling has it yet (search the new name; compare
   `qualifiedName`s).

## 3. MCP route (default)

1. **Confirm.** Show the `qualifiedName`, and each field old → new. Wait for go.
2. **Build the complete field set.** `update_staged_element` re-checks the element
   as if it were new, so it needs **every required field**, not only the changed
   one. And `search_elements` returns the **stored** field names, which are not
   always the names the tool accepts. Build `elementData` like this:
   1. Take the catalog row's **fields** list (the accepted names).
   2. Fill each from the stored `elementData`, using the row's **Stored as**
      mapping where the stored name differs. One rule covers every type: the
      stored **`parentId`** holds the parent's qualified name (a child's
      `…Qname` / `parentQname` field) or the Package Path (a top-level element's
      `packagePath`). Trust it over other stored reference fields.
   3. Change only the fields the user asked for. Send references (`…Ref` fields)
      without a leading `/`, even when the stored one still has it.
   4. Leave out `elementType`.
3. **Update.** `update_staged_element` with `stagedElementId` = the `id`,
   `elementType`, the `elementData` from step 2, and `expectedUpdatedAt` = the
   `updatedAt` from section 2.
4. **"This element changed after you read it"**: someone saved it in between. Read
   it again (section 2), re-apply only the user's change, confirm again if the other
   change touched the same field, and update.
5. **Validation issues** (`"Validation failed - nothing was updated."`): fix exactly
   those fields and call again.

A changed element that is already drawn on an open diagram updates by itself within
about 10 seconds.

## 4. Visual route

### Open the form

Pick one:

- **The edit page:** **Go to…** (or **Ctrl+K**), type the element's name, wait for
  it under **Elements**, and press **Enter**. The page shows **Edit <Thing>**.
- **The table's dialog:** open the table (catalog: **Menu**), find the row (use the
  search box, see `find-in-tables.md`), click its **⋯** button (named "Actions for
  <name>"), then **Edit**. The dialog **Edit <Thing>** opens.

### Change it

1. Wait until the form is filled in (the Short Name shows the current name). The
   parent picker is hidden in edit mode: the parent can't be changed here.
2. Change only the fields asked for. Pickers work as in `create-element.md`,
   section 4.
3. Summarise old → new and wait for go. Click the update button (**Update …** or
   **Update**) and wait until it stops reading "Updating...".

### After submit

- Success: **"<Thing> updated successfully"**. Edit pages return to the list.
- **Nothing happens**: the save failed without a message. Check with
  `search_elements`: compare the field and `updatedAt`. If unchanged, use the MCP
  route and tell the user the form failed.

## 5. Report

The element's qualified name, and each field old → new.

## Traps

- **Renaming doesn't change the qualified name.** The stored path keeps the old
  name, and so do its children's paths. Tell the user if they rely on the path.
- **Changing a kind or a reference can break what uses it**: an interface's type
  for its ports and connectors, a data type for its data elements, a direction for
  connectors. The app doesn't check. Say what uses it (search for elements that
  reference its `qualifiedName`) before you change it.
- **An interface's kind** can be changed in the form (**Interface Type**), but its
  operations, data elements or triggers don't follow. Prefer creating a new
  interface.
- Sending only the changed field fails validation. Always send the complete set.
