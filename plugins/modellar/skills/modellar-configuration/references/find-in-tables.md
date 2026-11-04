# Find elements, or show them in a table

Answer "which / how many / is there …" questions about configuration elements, or
show the user the table of one element type with a filter applied.

## 1. MCP route (default): answer in the chat

- `search_elements({ modelId, elementType: "<type>", search: "<part of a name>", limit: 50 })`.
  `search` matches a part of the short name or of the qualified name, ignoring
  case, also inside words: "CHR" finds "Syn**chr**on…". Prefer a whole name or a
  prefix with `_` ("CHR_"); `_` and `%` are plain characters. Leave out
  `elementType` to search every type. Add `includeElementData: true` when you need
  field values (a data type's category, a runnable's interval): only the first 10
  results of a call carry them, so page with `limit: 10`.
- At most 50 results per call, newest first. `totalCount` is how many match and
  `count` how many came back; when `nextOffset` is not null there are more: call
  again with `offset: <nextOffset>`, until it is null.
- **Children of an element, or everything in a package:**
  `qualifiedNamePrefix` with the parent's qualified name **followed by `/`** (a
  child's qualified name starts with it), or the package path. Example:
  `qualifiedNamePrefix: "Pkg/Interfaces/If_Speed/"` and `elementType: "Operation"`.
  Leave out `elementType` to get every child type. If the tool has no
  `qualifiedNamePrefix` yet, pass the same text as `search`.

## 2. Visual route: the table

### Open it

Menubar **Configuration** → the page (catalog: **Menu**). A sub-menu whose label is
also a page (**Interfaces**, **SW Component Types**, **Runnable Entities**, **ECU
Types**, **System Configuration**, **Mode Declaration Groups**) opens that page when
you click its label. Or **Go to…**, type the page name, **Enter**.

SWC Internal Behaviors has no menu entry: use **Go to…** (`SWC Internal Behaviors`).

### Read it

- The heading ends in **Manager** for most pages (e.g. **Port Interfaces Manager**,
  **Data Types Manager**). **RTE Events**, **Runnable Entities** and **Access
  Points** have none.
- The summary line above the table: **"Showing 1 to 50 of 56 interfaces"**, or
  **"No interfaces"** when empty (the table then reads "No interfaces found.").
  This total is the real count.
- 50 rows per page. Under the table: **"Page 1 of 2"**, **Previous**, **Page 1**,
  **Page 2**, **Next**.

### One parent's children

A row's menu (**Actions for <name>**, the **⋯** at the row's end) lists that
element's children in a dialog: **Open Runnables**, **Open Events**, **Open
Ports**, **Open Access Points**, **Open ECU Prototypes**, **Open Connections**, or
**Manage …** on an interface. Same table, filters and paging, limited to that
parent; its create button makes a child of it (`create-element.md`, "From the
parent's row").

### Filter and sort

- **Search box** (e.g. "Search interfaces..."): matches part of the **short name
  only** (not the path), ignoring case, also inside words. **Reset** appears and
  clears every filter.
- **Wait for the result.** The table keeps showing the old rows until the new ones
  arrive, a few seconds on a large model. Newer versions show **Updating…** next
  to the summary line and dim the rows meanwhile; older ones show no sign. Read
  the rows only once **Updating…** is gone and the summary line ("Showing 1 to 50
  of N …") has changed. A sort arrow appearing doesn't mean the rows have.
- **Facet buttons** (**Type**, **Status**, **Category**, **Direction**, **Scope**,
  …): click, then click option texts to tick them; ticked options combine. A number
  on the button shows how many are ticked. Press **Escape** to close.
- **Extra text filters** on some pages (e.g. "Filter by SW Component...",
  "Interface name..."): type into them like the search box.
- **Port Prototypes** also has **Owner**: **SWC Ports** (default), **ECU Ports**,
  **All**.
- Any filter change jumps back to page 1.
- **Sort:** click a column header (**Short Name**, **Created**, **Updated**, and
  **Type** on interfaces). Click again to reverse. The sort carries across pages.
- **Columns** shows or hides columns (e.g. **Extraction Source**, hidden by
  default).

Filters, sort and page aren't in the address: a reload or a shared link starts
unfiltered.

## 3. Report

The answer (names, count), and how it was filtered. For a table, leave it open
with the filter applied and say which filter is set.

## Traps

- The facet options and **Go to…** suggestions may not be readable by name
  through the page's accessibility tree. If you can't find an option by its text,
  take a screenshot to read it.
- Default order differs per table (newest first on some, by name on others). Sort
  explicitly when order matters.
- A table row's **⋯** menu **Delete** does nothing useful yet ("Delete not
  available yet" / "Not supported yet").
