---
name: modellar-navigation
description: Go to a ModellAR Designer page - a model, a diagram (composition, atomic SWC, ECU, system, workspace), a configuration list, create or edit page for an element, or an app page such as Projects or Profile - the fast way, without reloading the app. Use when the user, in ModellAR (URL contains /designer/ or another ModellAR page), asks to go to, open, jump to or show a model, diagram, element, page or configuration page, or after a Modellar MCP change to take the user to what changed.
---

# ModellAR: going to a page

## How to get there

Pick the first way that applies:

1. **Inside a model** (the URL contains `/designer/`): use **Go to…**, the button in
   the header just left of **Ask Claude**, or press **Ctrl+K** (**Cmd+K** on a Mac).
   Type a name: a diagram, an element or a page. The results come in groups:
   **Elements** (opens the element's edit page), **Diagrams**, and **Pages**
   (configuration lists, "New …" create pages, ARXML Management, Analytics,
   Validation). Choose one with the arrow keys and press **Enter**.
   Element results appear after a short pause and need at least 2 characters.
2. **Outside a model:** for an app page, click the sidebar link: **Dashboard**,
   **Projects**, **Users**, **Profile**, **API reference**, **Element Workflow**.
   **Into a model** (the Dashboard has no model links, and the designer has no
   model switcher):
   - **When a Modellar tool already gave you the page** (the `pagePath` of
     `list_diagrams` or `search_elements`), open that address directly. It loads
     the app once, which is quicker than the four clicks below, and lands on the
     exact diagram or element.
   - Otherwise: sidebar **Projects** → click the project's name on its card (the
     `projectName` from `list_accessible_models` says which card) → the tab
     **Models** (table **AUTOSAR Models**) → click the model's name. It opens the
     model's workspace diagram, by id.
3. **By address** (the addresses below): when a tool gave you a `pagePath` (see 2),
   when there is no **Go to…** button yet, or when the user isn't in ModellAR at all.

**Why not always the address bar?** Typing an address reloads the whole app: every
list and diagram is fetched again, which is slow. Go to… and the sidebar change the
page without a reload. Inside a model that saving wins; from outside one, a single
address beats several page changes.

**Leaving a diagram loses unsaved layout**, whichever way you leave. On every kind of
diagram, moved or resized boxes are saved only by **Save Diagram** (toast **"Diagram
Saved"**). **Save Diagram** shows a dot, and the description "Unsaved layout
changes: …", while there is something to save.

## Ground rules

- Use the host of the user's current tab. Never switch between the live site and a
  local one.
- Build addresses only from ids the tools returned. Never guess an id.
- Opening a page needs no confirmation.
- After opening, check the page: the top-left button shows the open diagram's name,
  and a configuration page shows its title. A page that is still loading can show
  the previous page for a moment; wait briefly and look again.
- **No Modellar tools?** Ask the user to connect the Modellar connector in Claude's
  connector settings, using an API key from their ModellAR **Profile** page
  (**API keys** → **New key**). Go to… works without them.

## Addresses

Go to… builds these for you. You need them only for way 3, and to recognise where
you are from the URL.

### The model

`/designer/{modelId}`. `{modelId}` is the model's **`id`** from
`list_accessible_models`, or the segment after `/designer/` in the current URL when
that is already an id. It opens the most recent workspace diagram.

**Never put the model's `slug` in an address.** A page opened by slug (for example
`/designer/my-model/...`) displays normally, but nothing saved from it reaches the
model: creates fail, and the page shows no error. If the user's current URL holds a
slug, open the same page with the model's `id` before you fill in any form: take the
current address, replace only the segment after `/designer/` with the `id` from
`list_accessible_models`, and open that. The MCP tools themselves accept either.

### Diagrams

Get `{diagramId}` from `list_diagrams({ modelId, stagedElementId })`. It is the
diagram's own `id`, **not** the id of the element it shows.

| `diagramType` from `list_diagrams`                                    | Address                                                                 |
| --------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `COMPOSITION_SW_COMPONENT_TYPE`, `ROOT_COMPOSITION_SW_COMPONENT_TYPE` | `/designer/{modelId}/diagram/composition-sw-component-type/{diagramId}` |
| `ATOMIC_SW_COMPONENT_TYPE`                                            | `/designer/{modelId}/diagram/atomic-sw-component-type/{diagramId}`      |
| `ECU_MANAGEMENT`                                                      | `/designer/{modelId}/diagram/ecu/{diagramId}`                           |
| `SYSTEM_MANAGEMENT`                                                   | `/designer/{modelId}/diagram/system/{diagramId}`                        |
| `WORKSPACE`                                                           | `/designer/{modelId}/diagram/workspace/{diagramId}`                     |

Without `{diagramId}` the address opens the most recent diagram of that kind.

There is no address that selects or centres one node. Once the diagram is open,
find the node by its name on the canvas.

### Configuration pages

`/designer/{modelId}/configuration/{section}` lists the elements of one kind.
Most sections also have:

- `/designer/{modelId}/configuration/{section}/create`: a new element;
- `/designer/{modelId}/configuration/{section}/{elementId}/edit`: edit one element.
  `{elementId}` is the element's `id` from `search_elements`.

Sections with create and edit pages:
`sw-component-types`, `sw-component-prototypes`, `port-prototypes`, `interfaces`,
`datatypes`, `operations`, `access-points`, `runnable-entitys` (spelled this way),
`rte-events`, `swc-internal-behaviors`, `com-specs`, `connectors`, `triggers`,
`variable-data-prototypes`, `parameter-data-prototypes`, `mode-declarations`,
`mode-declaration-groups`, `mode-declaration-group-prototypes`, `ecu-types`,
`system-configs`.

List only (no create or edit page; `connections`, `ecu-ports` and `ecu-prototypes`
create and edit in dialogs on the list): `base-types`, `connections`, `ecu-ports`,
`ecu-prototypes`, `mappings`, `system-description`.

Other model pages: `/designer/{modelId}/arxml`, `/designer/{modelId}/validation`,
`/designer/{modelId}/analytics`.

### App pages

`/` (Dashboard), `/projects`, `/users`, `/profile`, `/api-reference`,
`/element-workflow`.

## After an MCP change

- **A changed element that is already on an open diagram** updates by itself within
  about 10 seconds. No reload needed.
- **A new element** never appears on a diagram by itself. Go to its edit page
  (Go to…, type its name) or to the diagram, and tell the user where it is. To put
  a new component on a diagram, see skill `modellar-diagrams`.
