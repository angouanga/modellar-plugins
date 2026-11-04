# Create a configuration element

Create one element of any type listed in `element-catalog.md`: a data type, an
interface, an operation, a mode declaration, a runnable, and so on. One run creates
one element. For a chain (an interface and its operations), repeat it per element.

Open `element-catalog.md` at the element type's row before you start. It gives the
type's parent, its fields, and which route works today.

## 1. Where am I?

URL: `https://<host>/designer/{modelId}/...`. Take `{modelId}` from it for every MCP
call. No `/designer/` in the URL: call `list_accessible_models`. Exactly one model:
use it and say which. Several: ask which one.

## 2. Gather and check

1. **The values.** The user names at least the type and the short name. The catalog
   row lists what else the type needs (a kind, a data type, a period, …). Ask for a
   missing required value; don't invent one. Short name rules: starts with a letter,
   then letters, digits and `_`, at most 128 characters, not a C/C++ keyword such as
   `int` or `class`.
2. **The parent** (child types only, see the catalog). Find it in two steps:
   1. `search_elements({ modelId, elementType: "<parent type>", search: "<parent's full short name>" })`,
      without element data. `search` is a part-of-a-word match ("CHR" also finds
      "Syn**chr**on…"), so use the whole name, or `qualifiedNamePrefix` with the
      package path when you know it.
   2. Once you have one or a few matches (at most 10), search again with
      `includeElementData: true` to read them. Only the first 10 results carry
      element data; a wide search with data can also be too big to read.

   Keep its `id`, `qualifiedName`, `batchId` and `elementData.sourceFile`. Several
   matches: ask which one, showing each `qualifiedName`. Not found: offer to create
   it first (another run of this task).

3. **Referenced elements** (a data type for a data element, a mode declaration group
   for a prototype, …): look each one up the same way and keep `id` and
   `qualifiedName`. The user named a kind, not an element ("an integer data
   type"): never pick one yourself. Show 3–5 candidates, plain ones first (e.g. an
   implementation type `uint8` / `uint16` before an application type with a
   specific meaning), and let the user pick.
4. **No duplicate.**
   `search_elements({ modelId, elementType: "<type>", search: "<short name>" })`.
   A result whose `qualifiedName` equals the new element's qualified name (catalog:
   **Qname**) already exists: stop, tell the user, offer `edit-element.md` instead.
5. **The batch.**
   - A **child** goes into its parent's batch: the parent's `batchId`. Don't ask.
   - A **top-level** element (catalog: "Top-level") goes into a batch the user
     chooses: `list_staging_batches({ modelId })`, show the batches (newest first,
     with status and element count), and let the user pick, **even when there is
     only one**: name it in the question so a yes is enough. Never pick one
     yourself; reuse their choice for the rest of the conversation. Any status
     works, **COMPLETED** included: a completed batch still takes new elements.
     Empty list: ask
     them to create one (**Model** menu → **ARXML Management** → **Process
     Batches** tab → **Add batch**).
6. **Source file.** A child copies its parent's `elementData.sourceFile`. A
   top-level element reuses the `sourceFile` of a related element of the same kind
   (search one, `includeElementData: true`), or the user names one. Nothing related
   and the user has no preference: `agent-generated`.

## 3. MCP route (default)

1. **Confirm.** Summarise: type, qualified name, parent, the values, the batch. Wait
   for go.
2. **Shape.** `describe_element_type({ elementType: "<type>" })` once per type in the
   conversation. Fill exactly the fields the catalog row names, with the schema's
   spelling of every enum value.
3. **Stage it.** `stage_element` with:
   - `modelId`, `batchId` (step 2.5), `elementType`;
   - `absoluteQualifiedName`: the catalog's **Qname** for the type;
   - `elementData`: the catalog's fields, plus `shortName`, `sourceFile` and
     `description` if given. Don't put `elementType` inside `elementData`;
   - `idempotencyKey`: e.g. `"<type>-<short name>-1"`.
4. **Validation issues** (`"Validation failed - nothing was staged."`): fix exactly
   the fields listed and call again with the same key.
5. **Next.** Offer to open it: `search_elements` returns a `pagePath` for the new
   element, or use **Go to…** with its name (skill `modellar-navigation`).

## 4. Visual route

For when the user asks to use the form. Check the catalog row first: some create
forms are broken today and say which other way to use.

### Open the form

Pick one:

- **The table's dialog:** menubar **Configuration** → the page (catalog: **Menu**)
  → the button **New …** (catalog: **Button**). A dialog opens (catalog: **Dialog**).
  Click **New …** only once the table shows rows: a click while the page is still
  loading does nothing. No dialog after a click: wait a moment and click once more.
- **The full page:** **Go to…** (or **Ctrl+K**), type `New <element>` (for example
  `New Interface`, `New Data Type`, singular) and press **Enter** once the entry is
  visible. The page shows the heading **Create …**.
- **From the parent's row** (child types, best when the parent is known): open the
  parent's table, its row menu **Actions for <parent>**, and the list item for the
  child (table below). A dialog lists that parent's children only; its create
  button opens the form with the parent already set and its picker hidden.

  | Parent's table                  | Row menu item                                   | Children                          | Create button                          |
  | ------------------------------- | ----------------------------------------------- | --------------------------------- | -------------------------------------- |
  | **SW Component Types** (atomic) | **Open Internal Behaviour**                     | Behavior                          | **New Behavior**                       |
  | **SW Component Types** (atomic) | **Open Runnables** / **Open Events**            | Runnable / RteEvent               | **New Runnable** / **New RTE Event**   |
  | **SW Component Types**          | **Open Ports**                                  | PortPrototype (skill `modellar-ports`) | **Add Port Prototype**            |
  | **SW Component Types** (atomic) | **Open Local VDP** / **Open Local Parameter Data** | the behavior's own data        | **New Variable Data Prototype** / **New Parameter Data Prototype** |
  | **SWC Internal Behaviors**      | **Open Runnables** / **Open Events**            | Runnable / RteEvent               | **New Runnable** / **New RTE Event**   |
  | **Runnable Entities**           | **Open Access Points**                          | AccessPoint                       | **Add Access Point**                   |
  | **Interfaces**                  | **Manage Variable Data Prototypes** (sender-receiver, NV data), **Manage Operations** (client-server), **Manage Triggers**, **Manage Parameter Data Prototypes**, **Manage Mode Declaration Group Prototypes** | the interface's one child kind | **New Variable Data Prototype**, **New Operation**, **New Trigger**, **New Parameter Data Prototype**, **New Prototype** |
  | **System Configuration**        | **Open ECU Prototypes** / **Open Connections**  | skill `modellar-communication`    | **New ECU Prototype** / **New Connection** |
  | **ECU Types**                   | **Open Ports**                                  | skill `modellar-communication`    | **Send / Receive a PDU**               |

  The dialog's title is "<Children> — <parent>" (e.g. **Runnables — MySwc**), or
  "Manage <parent> <children>" for an interface. From a component's row, a new
  runnable or RTE event still asks for the behavior when the component has several;
  with one, it is already chosen.

Never open a form on an address that holds the model's slug (see skill
`modellar-navigation`): it displays fine, but nothing it saves arrives.

### Fill it

1. **Staging Batch** (top-level types only), **before anything else**. The button
   may read **Select staging batch** at first and fill in by itself a moment
   later: an 8-character id, a status and "N elements". Wait a moment, then check
   it is the batch the user chose. To pick one, click it, type part of the id into
   **Search by batch or trigger run id...**, then click the entry (or press
   **Enter**). Child types have no batch field: they take the parent's.
2. **The parent picker first** (child types): Interface, Software Component,
   Internal Behaviour, Mode Declaration Group, …. Some forms show **Short Name**
   only after the parent is chosen.
3. **Searchable pickers** (any field whose button reads "Select a …"): click it,
   **type the name** into the search box, then click the row with the right name.
   They load 10 rows at a time; the description shows "(N total)". Typing beats
   scrolling. Opening a picker only highlights the first row: it isn't chosen until
   you click it or press **Enter**. A click closes the picker by itself. **Don't
   press Escape after a pick**: in a dialog it closes the whole dialog and loses
   what you filled in.
4. **Short Name**, then the other fields of the catalog row, by their labels. Then
   check that **Short Name** still holds what you typed: in one test, typing landed
   elsewhere around the batch pick.
5. **Source File** (required, red \*, on top-level types such as Interface, Data
   Type, Mode Declaration Group, SW Component Type, ECU Type, System): the value
   from step 2.6. Child forms mostly fill it from the parent; the catalog's
   **Traps** say where they don't.
6. **Fields the form fills in silently** (catalog: **Defaults**): tell the user
   which value will be saved when they leave such a field empty.
7. Summarise the values and wait for go. Then click the submit button (catalog:
   **Submit**) and wait until it stops reading "Creating...".

### After submit

- Success: the toast **"<Thing> created successfully"** (or "… staged
  successfully"), once. The dialog closes, or the full page returns to
  the list.
- **"Fix 1 field before saving"** / **"Fix N fields before saving"**: read the red
  message under each field, fix exactly those, and submit again.
- **Nothing happens** (no toast, the dialog stays open, the button is back to its
  label): the save failed without a message. Check with `search_elements`. If the
  element isn't there, switch to the MCP route and tell the user the form failed.

## 5. Report

The element's type and qualified name, its parent, the batch, and where to see it
(its edit page or table). Offer the next step of the chain, if any.

## Traps

- **Duplicates are created silently.** Always do step 2.4.
- After an error from `stage_element`, search before retrying: it may have been
  written anyway. Retry with the **same** idempotency key. Never retry after a
  warning that says not to.
- **"Staging batch not found"** (MCP), or the toast "The staging batch belongs to
  another user or no longer exists, so nothing was saved." (form), for a child: the
  parent sits in another user's batch. A child can only go into its parent's batch,
  so tell the user; don't try another batch.
- A leading slash in **Package Path** produces `//Pkg/Name`. Strip it.
- Operations have **no arguments** yet: neither the form nor the connector can add
  them. Say so if the user asks for arguments.
- Delete isn't available (no tool; the menu item says "Delete not available yet" or
  "Not supported yet"). Don't promise it.
