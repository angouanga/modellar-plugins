# The properties panel

Every diagram has a **properties panel** on the right of the canvas. It shows the
selected box or connection, in tabs. The tabs are where you edit an element, and
where you list **all** of a frame's children (components, ports, connectors, ECU
prototypes) and show or hide them on the diagram.

This file is a map. Other tasks point here for "open the panel on X, tab Y".

## Open it

1. **Select** what you want to see: click the box (a frame: click its header) or
   the connection on the canvas. A selected box shows a pink resize outline.
   - **The click didn't select it** (no outline; with the panel open it still
     reads "Select a node or edge to view its properties"): use the keyboard.
     In the accessibility tree each box is a group named `"<name> – <kind>"`,
     for example `"Kessy – Composition"`. Focus it and press **Enter**. Don't
     click at screen positions instead: after a resize or zoom they miss.
2. Click the button **Properties panel**: a small chevron on the right edge of the
   canvas, at mid-height. Selecting alone does **not** open the panel.
3. The button has the same name open or closed: read its `expanded` state.
   `expanded=true` means it is already open; don't click it again, that closes it.

The panel shows the heading **Properties** and a badge with the selection's kind
(for example `Node: compositionFrame`, `Edge: connectorEdge`, or `None`). With
nothing selected it reads "Select a node or edge to view its properties".

Once open, the panel follows the selection: click another box and the panel
switches to it. Tabs switch on click (or Enter), not with arrow keys.

## Tabs, by diagram kind and selection

| Diagram     | Select …                      | Tabs                                                |
| ----------- | ----------------------------- | --------------------------------------------------- |
| Composition | the composition frame         | Properties · Ports · Components · Connections       |
| Composition | a component (prototype) box   | Properties · Ports (view only) · Connections        |
| Composition | a connection                  | no tabs: one connector card                         |
| Atomic SWC  | the SWC frame                 | Properties · Ports · Behaviors                      |
| Atomic SWC  | the internal behaviour frame  | Properties · Runnables · RTE Events                 |
| Atomic SWC  | a runnable box                | Properties · Access Points · RTE Events             |
| Atomic SWC  | an RTE event box              | no tabs: the event's form                           |
| ECU         | the ECU frame                 | Properties · Ports · Mapping                        |
| ECU         | a component (prototype) box   | Properties · Ports (view only) · Mappings           |
| ECU         | a composition frame inside it | Properties · Ports · Components · Connections       |
| ECU         | a mapping connection          | no tabs: one connection card                        |
| System      | the system frame              | Properties · ECU Prototypes · Connections           |
| System      | an ECU prototype box          | Properties · Ports (view only) · Connections        |
| System      | a connection                  | no tabs: one connector card                         |
| Workspace   | a diagram box                 | no tabs: read-only Name, Description, Diagram type… |

Ports on the canvas can't be selected: manage them from the frame's **Ports** tab.

## What each tab does

- **Properties**: the element's form, already filled in. The submit button saves
  the element at once (**Update Component**, **Update Prototype**, **Update**,
  **Update Event**, **Update System**). It is a write: summary and go first.
- **List tabs** (Ports, Components, Connections, ECU Prototypes, Mapping,
  Runnables, RTE Events, Behaviors, Access Points): one card per child, with:
  - a toolbar: **Show All** / **Hide All** (not on every tab), **Filter**, and an
    add button (**Add Component**, **Add Port**, **Add Connector**, **Add ECU
    Prototype**, **Add Runnable**, **Add RTE Event**, **Add Behavior**, **Add
    Access Point**) that opens a create dialog;
  - per card: a visibility button, **Edit …** and **Delete …** (named after the
    item, for example "Edit component prototype FoDMaster"). The ECU's
    **Mapping** cards, and a mapping line's card, have no **Edit**: a signal
    mapping can't be edited (skill `modellar-communication`,
    `map-port-to-pdu.md`).
  - Long lists show 10 at a time: "Showing X of Y …" and **Load More**.

### The visibility button on a card

| Card button | Meaning                                    | Click does                    |
| ----------- | ------------------------------------------ | ----------------------------- |
| **Add**     | not on this diagram yet ("Not in diagram") | creates the box or edge       |
| **Show**    | on the diagram but hidden                  | shows it                      |
| **Hide**    | shown                                      | hides it (nothing is deleted) |

Port cards use an eye button named "Show port <name>" / "Hide port <name>"
instead. Runnable cards use an icon-only eye with no name (see Traps).

### Filter popover

**Filter** opens a popover: **Search by name**, then selects such as **Staging
Status**, **Visibility** (All / Visible / Hidden), **Direction**, **Interface
Type** or **Connector Type**, depending on the tab. Close it with **OK**. A badge
on **Filter** counts the active filters. Remove them with **Clear filters** (in
the Connections tabs: **Reset all filters**), or **Reset filters** in the empty
list.

**Visibility: Hidden** lists both hidden items and items not on the diagram yet.

## What is saved, and when

The rule: **a new box is saved by itself; everything else on the canvas needs
Save Diagram.**

| Action                                                                      | Saved                    |
| --------------------------------------------------------------------------- | ------------------------ |
| Form submit, create dialog, edit dialog (the model element)                 | at once                  |
| A new box on the diagram: **Add** on a card, the boxes **Show All** creates | at once                  |
| Showing or hiding a box that is already on the diagram                      | only on **Save Diagram** |
| Ports (handles): show, hide                                                 | only on **Save Diagram** |
| Connection lines (edges): add, show, hide                                   | only on **Save Diagram** |
| Box position and size                                                       | only on **Save Diagram** |

**Save Diagram** is in the floating dock. The dock's unsaved-changes dot lights up
for moved boxes only, **not** for visibility changes: after any Show or Hide, click
**Save Diagram** yourself and wait for the toast **"Diagram Saved"**. Without it,
a reload brings the old state back.

A port's side can't be changed for good in the UI (the handle menu's **Change
Position** is unfinished); the Modellar tool `set_port_placement` does it (skill
`modellar-ports`).

Deleting isn't available yet. The **Delete …** buttons delete the **model
element**, not the drawing: they open a dialog that says deleting isn't available,
lists what a delete would affect, and offers only **Delete (not available yet)**,
disabled. Tell the user; don't look for another way.

The panel has no "remove from diagram". Taking a box off a diagram is the box's ⋮
menu **Remove from Diagram**, or the Modellar connector: task
`remove-from-diagram.md`.

## Traps

- **The toolbar's Filter Components is not this.** The **Filter Components** button
  at the top of the canvas lists only boxes already on the canvas. Components never
  placed on the diagram are missing from it, so it can't show "all components".
  Use the frame's list tab here.
- The frame's **"Visible N"** badge on the canvas counts its visible **ports**, not
  its visible children.
- Component cards name their button after the component ("Add <name> to the
  diagram", "Show <name> on the diagram"). It is disabled while it works: click it
  **once** and wait for the toast. Other cards' buttons are named just **Add**,
  **Show** or **Hide**: go by the card they sit in (the card's title is the item's
  name), never by position in the list.
- Runnable cards' buttons (add/show/hide, edit, delete) have no accessible name,
  only a tooltip. Hover to read it before clicking.
- The selects in the Filter popover have no name either: find them by the label
  text just above each.
- Workspace diagram boxes are opened with their ⋮ menu **Go to diagram**, or by
  double-clicking. The panel has no link.
