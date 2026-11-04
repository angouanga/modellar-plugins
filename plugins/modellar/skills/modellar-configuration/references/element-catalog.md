# Element catalog

One block per element type. Read only the block you need.

How to read a block:

- **Parent**: "Top-level" means no parent; the user chooses the batch. Otherwise the
  parent type: the element goes into the parent's batch and copies its
  `sourceFile`.
- **Qname**: the `absoluteQualifiedName` to send to `stage_element`. `<pkg>` is the
  Package Path without a leading slash; `<parent>` is the parent's `qualifiedName`.
- **Fields**: the `elementData` names `stage_element` and `update_staged_element`
  accept, besides `shortName`, `sourceFile` and `description`, which every type
  takes (except where noted). Required fields are **bold**. `X / XId` means the
  referenced element's `qualifiedName` and `id`.
- **Stored as**: names that `search_elements` returns differently. Map them back
  before an update (`edit-element.md`, step 3.2). Always: the stored `parentId` is
  the parent's qualified name, or the Package Path for a top-level element.
  A stored reference (a `…Ref` field) may start with `/` (imported from ARXML) or
  not (created in the app). Both mean the same element: compare names ignoring a
  leading `/`, and in an update send a reference back in the form it is stored in.
- **Defaults**: values saved when a field is left empty, though the form doesn't
  show them. Tell the user before saving.
- **Menu**: the path under the menubar **Configuration**. The menubar is the row of
  tabs **Diagram**, **Model**, **Configuration** at the top of the designer: click
  **Configuration**, then hover a sub-menu's label to open it. **Button** /
  **Dialog** / **Submit**: the table's create button, the dialog title, and the
  submit button.
- **Route today**: which routes work. "Form broken" means use the MCP route and tell
  the user why if they asked for the form.

Every required enum value must use the spelling shown here (the schema's).

---

## DataType

- **Parent:** Top-level. **Qname:** `<pkg>/<shortName>`.
- **Fields:** **`packagePath`**, **`type`** (`ApplicationPrimitive`,
  `ApplicationArray`, `ApplicationRecord`, `Implementation`, `ApplicationUnion`,
  `ApplicationReference`, `ApplicationComposite`; `SwBaseType` for a base type),
  **`category`** (`Value`, `Array`, `Structure`, `FixedLength`, `Primitive`,
  `TypeReference`, `DataReference`), `implementationType` (e.g. `uint8`),
  `calibrationAccess` (e.g. `ReadOnly`).
- **Stored as:** `parentId` → `packagePath`.
- **Defaults:** no `implementationType` (an empty **Implementation Type**) is saved
  as **`uint32`**, through the form and through MCP alike. The form's placeholder
  says "e.g., uint8". Ask for the implementation type, or tell the user `uint32`
  will be stored.
- **Menu:** **Types** → **Data Types** (base types: **Types** → **Base Types**).
  **Button:** **New Data Type** (**New Base Type**). **Dialog:** **Create New Data
  Type**. **Submit:** **Create Data Type**.
- **Form fields:** Staging Batch, Short Name, Type (default ApplicationPrimitive),
  Category (default Value), Package Path, Source File, Implementation Type,
  Calibration Access, Description.
- **Route today:** MCP and form.
- **Traps:** **New Base Type** doesn't preset Type: set **SwBaseType** yourself, or
  it becomes an ordinary data type. Base types appear only on the Base Types page.

## Port interfaces

`SenderReceiverInterface`, `ClientServerInterface`, `ModeSwitchInterface`,
`NvDataInterface`, `ParameterInterface`, `TriggerInterface`.

- **Parent:** Top-level. **Qname:** `<pkg>/<shortName>`.
- **Fields:** **`packagePath`**. `ParameterInterface` and `TriggerInterface` take
  **no** `description` yet. The form still shows **Description** for them, but the
  text is dropped: tell the user if they type one.
- **Stored as:** `parentId` → `packagePath`.
- **Menu:** **Interfaces**. **Button:** **New Interface**. **Dialog:** **Create New
  Interface**. **Submit:** **Create Interface**.
- **Form fields:** Staging Batch, Short Name, Interface Type (**Client-Server**,
  **Sender-Receiver** (default), **Mode Switch**, **Parameter**, **Trigger**, **NV
  Data**), Package Path, Source File, Description.
- **Route today:** MCP and form.
- **Content of each kind** (created separately, below): Sender-Receiver and NV Data →
  VariableDataPrototype; Client-Server → Operation; Parameter →
  ParameterDataPrototype; Trigger → Trigger; Mode Switch →
  ModeDeclarationGroupPrototype.

## VariableDataPrototype (data element, inter-runnable variable)

- **Parent:** a Sender-Receiver or NV Data interface (scope `INTERFACE`), or a
  Behavior (scope `SWC`). **Qname:** `<parent>/<shortName>`.
- **Fields:** **`scope`** (`INTERFACE`, `SWC`), **`container`** (`DATA_ELEMENT` for
  `INTERFACE`; `IMPLICIT_INTER_RUNNABLE_VARIABLE` or
  `EXPLICIT_INTER_RUNNABLE_VARIABLE` for `SWC`), **`parentQname`** (= the parent's
  `qualifiedName`), **`dataTypeRef / dataTypeId`**. Interface scope:
  `interfaceRef / interfaceId`, `interfaceType` (the interface's elementType without
  "Interface": `SenderReceiver`, `NvData`). SWC scope: `behaviorRef / behaviorId`.
  Optional: `isQueued` (false), `swImplPolicy`, `swCalibrationAccess`.
- **Stored as:** no `parentQname` stored: rebuild it from `interfaceRef` or
  `behaviorRef`.
- **Menu:** **Interfaces** → **Variable Data Prototypes**. **Button:** **New Variable
  Data Prototype**. **Dialog:** **Create Variable Data Prototype**. **Submit:**
  **Create Variable Data Prototype**.
- **Form fields:** Scope (**Interface** (default) or **SWC / Behaviour**); then
  Interface, or Software Component then Internal Behaviour; then Short Name,
  Container (read-only "Data Element" for interfaces; a required choice **Implicit
  Inter-Runnable Variable** / **Explicit Inter-Runnable Variable** otherwise), Data
  Type, Is Queued, SW Implementation Policy, SW Calibration Access, Description.
- **Route today:** MCP and form.

## ParameterDataPrototype

- **Parent:** a Parameter interface (scope `INTERFACE`) or a Behavior (scope `SWC`).
  **Qname:** `<parent>/<shortName>`.
- **Fields:** **`scope`** (`INTERFACE`, `SWC`), **`parentQname`**,
  `dataTypeRef / dataTypeId`; interface scope: `interfaceRef / interfaceId`,
  `interfaceName` (the interface's short name); SWC scope: `behaviorRef /
behaviorId`.
- **Menu:** **Interfaces** → **Parameter Data Prototypes**. **Button:** **New
  Parameter Data Prototype**. **Submit:** **Create Parameter Data Prototype**.
- **Form fields:** Scope, Interface (or Software Component + Internal Behaviour),
  Short Name, Data Type, Description.
- **Route today:** MCP and form.

## Operation

- **Parent:** a Client-Server interface. **Qname:** `<parent>/<shortName>`.
- **Fields:** **`interfaceQname / interfaceId`**.
- **Stored as:** `parentId` → `interfaceQname`. Not `interfaceRef`: on operations
  created by older versions of the app it holds the interface's **id**, and the
  Operations table shows that id in the Interface column. The operation itself is
  fine, and its first update repairs the column.
- **Menu:** **Interfaces** → **Operations**. **Button:** **New Operation**.
  **Dialog:** **Create Operation**. **Submit:** **Create Operation**.
- **Form fields:** Interface first; Short Name and Description appear after it.
  Submit stays disabled until an interface is chosen.
- **Route today:** MCP and form.
- **Traps:** no arguments (the edit form's **Manage Argument Data Prototype** button
  is a disabled placeholder).

## Trigger

- **Parent:** a Trigger interface. **Qname:** `<parent>/<shortName>`.
- **Fields:** **`interfaceRef / interfaceId`**, **`triggerInterfaceName`** (the
  interface's short name). No `description`.
- **Menu:** **Interfaces** → **Triggers**. **Button:** **New Trigger**. **Submit:**
  **Create Trigger**.
- **Form fields:** Interface first, then Short Name.
- **Route today:** MCP and form.

## ModeDeclarationGroup

- **Parent:** Top-level. **Qname:** `<pkg>/<shortName>`.
- **Fields:** **`packagePath`**, `category` (e.g. `ALPHABETIC_ORDER`),
  `onTransitionValue` (number), `initialModeRef` (a ModeDeclaration's
  `qualifiedName`: set it in an update, after its modes exist).
- **Stored as:** `parentId` → `packagePath`.
- **Menu:** **Types** → **Mode Declaration Groups**. **Button:** **New Mode
  Declaration Group**. **Submit:** **Create Mode Declaration Group**.
- **Form fields:** Staging Batch, Short Name, Package Path, Initial Mode Reference
  (disabled until the group exists), Category, On Transition Value, Source File,
  Description.
- **Route today:** MCP and form.

## ModeDeclaration (a mode)

- **Parent:** a ModeDeclarationGroup. **Qname:** `<parent>/<shortName>`.
- **Fields:** **`modeDeclarationGroupQname / modeDeclarationGroupId`**, `value`
  (integer: the evaluation order).
- **Stored as:** `parentModeDeclarationGroup` → `modeDeclarationGroupQname`.
- **Menu:** **Types** → **Mode Declaration Groups** → **Mode Declarations**.
  **Button:** **New Mode Declaration**. **Submit:** **Create Mode Declaration**.
- **Form fields:** Staging Batch, Mode Declaration Group, Short Name, Value, Source
  File, Description.
- **Route today:** MCP and form.
- **Traps:** the form doesn't copy Source File from the group: type the group's
  source file yourself.

## ModeDeclarationGroupPrototype

- **Parent:** a Mode Switch interface. **Qname:** `<parent>/<shortName>`.
- **Fields:** **`parentInterface / interfaceId`** (the interface),
  **`typeRef`** and **`modeDeclarationGroupRef`** (both the group's
  `qualifiedName`), **`modeDeclarationGroupId`**, `typeRefDest`
  (`MODE-DECLARATION-GROUP`).
- **Stored as:** `ModeDeclarationGroupRef` → `modeDeclarationGroupRef`,
  `ModeDeclarationGroupId` → `modeDeclarationGroupId` (capital M).
- **Menu:** **Interfaces** → **Mode Declaration Group Prototypes**. **Button:** **New
  Prototype**. **Submit:** **Create Prototype**.
- **Form fields:** Interface first; then Short Name and Mode Declaration Group.
- **Route today:** MCP and form.
- **Traps:** one prototype per mode switch interface. Check before creating a
  second.

## Behavior (SWC internal behavior)

- **Parent:** an atomic SwComponent. **Qname:** `<parent>/<shortName>`.
- **Fields:** **`componentQname / componentId`**.
- **Stored as:** `componentRef` → `componentQname`.
- **Menu:** none: **Go to…** `SWC Internal Behaviors`. **Button:** **New Behavior**.
  **Dialog:** **Add Behavior**. **Submit:** **Create**. From the SWC's row: **Open
  Internal Behaviour** (`create-element.md`, "From the parent's row").
- **Route today:** MCP and form. The form takes the SWC's batch and source file.
- **Traps:** an SWC normally has one behavior. Search for an existing one first.

## Runnable (runnable entity)

- **Parent:** a Behavior. **Qname:** `<parent>/<shortName>`.
- **Fields:** **`symbol`** (the C function name; usually the short name),
  **`behaviorQname / behaviorId`**, **`componentRef`** (the SWC's
  `qualifiedName`), `minimumStartInterval` (integer, ms),
  `canBeInvokedConcurrently` (false).
- **Stored as:** `behaviorRef` → `behaviorQname`.
- **Menu:** **SW Component Types** → **Runnable Entities**. **Button:** **New
  Runnable**. **Dialog:** **Create Runnable**. **Submit:** **Create**. From a
  parent's row: **Open Runnables** on the SWC or the behavior.
- **Route today:** MCP and form. The form takes the behavior's batch and source
  file.

## RteEvent

- **Parent:** a Behavior. **Qname:** `<parent>/<shortName>`.
- **Fields:** **`eventType`** (`TimingEvent`, `InitEvent`, `BackgroundEvent`,
  `DataReceivedEvent`, `DataReceiveErrorEvent`, `DataSendCompletedEvent`,
  `DataWriteCompletedEvent`, `OperationInvokedEvent`,
  `AsynchronousServerCallReturnsEvent`, `SwcModeSwitchEvent`, … see
  `describe_element_type`), **`behaviorQname / behaviorId`**, **`componentRef`**,
  **`runnableRef / runnableId`** (the runnable it starts). `TimingEvent`: `period`
  (seconds, e.g. `0.01` for 10 ms), `offset`. Data events: `portRef / portId`,
  `dataElementRef / variableDataPrototypeId`. `OperationInvokedEvent`:
  `portRef / portId`, `operationRef / operationId`.
- **Stored as:** `behaviorRef` → `behaviorQname`.
- **Menu:** **SW Component Types** → **RTE Events**. **Button:** **New RTE Event**.
  **Submit:** **Create Event**.
- **Route today:** MCP and form. The form takes the behavior's batch and source
  file, and requires **Runnable Entity (startOnEvent)**. From a parent's row:
  **Open Events** on the SWC or the behavior. Through MCP, use real ids for
  `behaviorId` and `runnableId`.
- **Traps:** `period` and `offset` are stored in **seconds**. The table shows them
  in ms ("10 ms"); older versions of the app show the seconds value with "ms"
  after it ("0.01ms"), which also means 10 ms. Don't "correct" either.

## AccessPoint

- **Parent:** a Runnable. **Qname:** `<parent>/<shortName>`.
- **Fields:** **`accessType`** (`DataReadAccess`, `DataWriteAccess`,
  `DataReceivePointByValue`, `DataReceivePointByArgument`, `DataSendPoint`,
  `ParameterAccess`, `ModeAccessPoint`, `ModeSwitchPoint`,
  `SynchronousServerCallPoint`, `AsynchronousServerCallPoint`,
  `ExternalTriggerPoint`, …), **`runnableQname / runnableId`**, **`behaviorRef`**,
  **`componentRef`**, `portRef / portId`, `portRefDest` (`R-PORT-PROTOTYPE` for a
  receiver port, `P-PORT-PROTOTYPE` for a provider), `interfaceRef`. Data access:
  `dataElementRef / variableDataPrototypeId`, `dataElementRefDest`
  (`VARIABLE-DATA-PROTOTYPE`). Server call: `operationRef / operationId`,
  `operationRefDest` (`CLIENT-SERVER-OPERATION`), `timeout`.
- **Stored as:** `runnableRef` → `runnableQname`.
- **Menu:** **SW Component Types** → **Runnable Entities** → **Access Points**.
  **Button:** **Add Access Point**. **Dialog:** **Add Access Point**. **Submit:**
  **Create Access Point**.
- **Form fields:** Software Component, SWC Internal Behavior, Runnable Entity (when
  not opened from a runnable); Short Name; Port; Access Type (enabled after the
  port; options depend on the port); then Variable Data Prototype, Operation +
  Timeout (ms), parameter, trigger or mode group, depending on the interface.
- **Route today:** MCP and the table's dialog. The full create page needs a
  runnable: it shows "Runnable context is required …" when opened from **Go to…**.
  From the runnable's row (**Open Access Points**) the component, behavior and
  runnable are already set and the **Select Component Context** picker is hidden.

## ComSpec

- **Parent:** a PortPrototype. **Qname:** `<parent>/<shortName>`.
- **Fields:** **`type`** and **`comSpecType`** (the same value:
  `NonqueuedSenderComSpec`, `QueuedSenderComSpec`, `NonqueuedReceiverComSpec`,
  `QueuedReceiverComSpec`, `ServerComSpec`, `ClientComSpec`, …),
  **`portDirection`** (`Provider`, `Receiver`), **`portRef / portId`**;
  `dataElementRef / dataElementId` or `operationRef / operationId`, `initValue`,
  `queueLength`, `aliveTimeout`.
- **Stored as:** `portPrototypeRef` / `portPrototypeId` → `portRef` / `portId`;
  `variableDataPrototypeId` → `dataElementId`. The stored `comSpecType` may be
  spelled like `NONQUEUED_RECEIVER` or `CLIENT-COM-SPEC`: convert it to the
  schema's spelling before an update.
- **Menu:** **Types** → **ComSpecs**. **Button:** **New ComSpec**. **Submit:**
  **Create**.
- **Route today:** **MCP only for create.** The form picks a batch on its own and
  saves without a source file. **ComSpec Type** starts empty and must be picked;
  "Unknown" shows only on an imported ComSpec whose type wasn't read: change it.

## EcuInstanceType (ECU type)

- **Parent:** Top-level. **Qname:** `<pkg>/<shortName>`.
- **Fields:** **`packagePath`**, `softwareComponentRef / softwareComponentId` (the
  root composition), `isSystemRelevant` (true), `hasConnectors` (false),
  `hasCommControllers` (false), `referenceOnly` (false).
- **Stored as:** `parentId` → `packagePath`.
- **Menu:** **ECU Types**. **Button:** **New ECU Type**. **Dialog:** **Create ECU
  Type**. **Submit:** **Create**.
- **Route today:** MCP and form.

## SystemType (system)

- **Parent:** Top-level. **Qname:** `<pkg>/<shortName>`.
- **Fields:** **`packagePath`**, `isSystemRelevant` (true), `hasSwComposition`,
  `hasSystemMappings`, `hasValidationElements` (all false).
- **Stored as:** `parentId` → `packagePath`.
- **Menu:** **System Configuration**. **Button:** **New System**: opens the page
  **Create System** (no dialog). **Submit:** **Create System**.
- **Route today:** MCP and form.
