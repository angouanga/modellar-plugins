# Create a software component type

Create a new SWC type: atomic (Application, Service, Sensor Actuator, …) or a
composition. The Modellar MCP connector looks up what is needed and creates it.
The UI does it when the user asks, or when the MCP route isn't available.

## 1. Where am I?

Read the current URL: `https://<host>/designer/{modelId}/...`.

- `{modelId}` (an id or a slug) is the model to use with every MCP call.
- If the URL has no `/designer/`, ask the user which model to work in. Use
  `list_accessible_models` to offer the choices.

## 2. Gather and check (MCP)

| Field        | Rule                                                                           |
| ------------ | ------------------------------------------------------------------------------ |
| Short Name   | Starts with a letter, then only letters, digits and `_`. No spaces.            |
| Kind         | Atomic or composition (see the Type values below). Ask if the user didn't say. |
| Package Path | Like `Compositions/Zuko` or `App_IL/SWCTypes`. **No leading slash.**           |
| Source File  | The ARXML file it belongs to (see below).                                      |
| Description  | Optional.                                                                      |

Type values:

- **Atomic:** Application (default), Service, NV Block, Complex Device Driver,
  Service Proxy, ECU Abstraction, Sensor Actuator, Parameter.
- **Composition:** Composition (default), Root Composition. Use Root Composition
  only if the user asks for it explicitly.

1. **Source File and Package Path.** Find a related component with
   `search_elements({ modelId, elementType: "SwComponent", search: "<a sibling or parent name>", includeElementData: true })`.
   Reuse its `elementData.sourceFile`. If the user named a parent composition
   ("under Zuko"), use that composition's `elementData.packagePath`. If nothing fits,
   ask the user.
2. **It must not exist yet.**
   `search_elements({ modelId, elementType: "SwComponent", search: "<ShortName>" })`.
   If a result's `qualifiedName` equals `<Package Path>/<ShortName>` (no leading slash), **stop**. Tell
   the user it already exists and offer to open its diagram instead (skill
   `modellar-diagrams`, task `open-or-create-swc-diagram.md`).

## 3. MCP route (default)

Use it when the Modellar tools include `list_staging_batches`. If they don't, use
the visual route.

1. **The batch: the user chooses.** A new SWC type has no parent, so it goes into
   a batch the user picks. Call `list_staging_batches({ modelId })` and show the
   batches (newest first, with status and element count), and let the user choose.
   Never pick one yourself. Keep their choice for the rest of the conversation
   unless they say otherwise. Empty list: ask the user to create a batch in the app
   (**Model** menu → **ARXML Management** → **Process Batches** tab → **Add batch**).
2. **Confirm.** Summarise the values from section 2 and wait for the user's go.
3. **Stage it.** `describe_element_type({ elementType: "SwComponent" })` if you
   haven't yet, then `stage_element` with:
   - `elementType`: `"SwComponent"`
   - `absoluteQualifiedName`: `<Package Path>/<ShortName>`
   - `elementData`: `shortName`, `type`, `packagePath` (no leading slash),
     `sourceFile`, and `description` if the user gave one. `type` takes the
     schema's spelling: `Application`, `Service`, `NVBlock`, `ComplexDeviceDriver`,
     `ServiceProxy`, `EcuAbstraction`, `SensorActuator`, `Parameter`,
     `Composition`, `RootComposition`.
   - `idempotencyKey`: e.g. `"swc-<ShortName>-1"`.
4. **Next.** A new type has no diagram. If the user wants one, or wants to add
   components next, continue with skill `modellar-diagrams`, task
   `open-or-create-swc-diagram.md` (it creates the diagram in the UI).

## 4. Visual route

### Open the create form

1. In the top menubar, click **Diagram**.
2. Click the item that matches the kind:
   - **Manage Composition SW Component Types** → in the dialog footer click **Create New CSwC Type**.
   - **Manage Atomic Sw Component Types** → in the dialog footer click **Create New ASwC Type**.
   - Do not click the footer's **Create Diagram**. It makes a diagram for an existing component, not a new type.

If that path fails, use the fallback: menubar **Configuration** → **SW Component
Types** (page "Software Components Types Manager") → **New Software Component** →
**Atomic Component** or **Composition Component**.

The dialog opens with the title **Create Composition SWC Type** or **Create Atomic SWC Type**.

### Fill the form

1. **Staging Batch.** Already selected. Keep it unless the user named a batch.
2. **Short Name \***
3. **Type \*.** A dropdown with placeholder "Select component type". Pick the value.
4. **Package Path \*.** No leading slash.
5. **Source File \***
6. **Description.** Optional.

Summarise the values and ask for go. Then click
**Create Component** and wait until the button stops reading "Creating...".

Success shows the toast **"Software component created successfully"**. A red message under a field means it wasn't saved. Fix
exactly that field and submit again.

### The optional diagram step

The same dialog then shows **Create Diagram for New SWC** (badge "Optional").

- If the user asked for a diagram, or wants to add components next: leave
  "Diagram Name" as it is, confirm, then click **Create Diagram**. The app opens
  the new diagram.
- Otherwise, click **Skip Diagram**.

## 5. Report

Tell the user the type, the qualified name `<Package Path>/<ShortName>`, and
whether a diagram was created.

## Traps

- A leading slash in Package Path produces `//Pkg/Name`. Strip it.
- The name must be unique in its package. Always do the check in section 2.
- Root Composition is for the system's top-level composition only. Never pick it
  as a default.
