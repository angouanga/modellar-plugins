# Find PDUs, signals, systems and ECUs

Look up a PDU with its signals, system, frame, transmitters and receivers; list the
PDUs of an ECU or a system; list the systems and where each ECU is placed.

## 1. Where am I?

- Standalone page: `https://<host>/designer/{modelId}/configuration/system-description`,
  heading **System Description**.
- On an ECU, system or workspace diagram the dock button that shows **PDU** (its
  name and tooltip: **System Description**) opens the same editor as a panel,
  view only at first: press **Edit** to change things (it then reads **View
  only**).
- Two tabs: **PDU based** (one row per PDU) and **ECU based** (per ECU, the PDUs it
  sends and receives).

## 2. MCP route (default)

1. **One PDU by name**: `list_pdus({ modelId, search: "<name>" })`. Several rows with
   the same `shortName`: show the user the `qualifiedName`s and ask which.
2. **That PDU in full**: `get_pdu({ modelId, pduId })`. It returns:
   - `signals`, sorted by `startBit`, each with `startBit`, `bitLength` and `endBit` in
     **bits**. A signal stored without a position has `startBit` and `endBit`
     `null`: report it as having no position, never as bit 0;
   - `lengthBytes` (**bytes**), `system` (name, file), `frame`, `busType`;
   - `transmitters` and `receivers`: ECU names, their port, and `placedInSystem`;
   - `missing`: what the PDU still lacks (`system`, `frame`, `transmitter`,
     `receiver`, `signals`, `signalPositions` = a signal has no start bit or
     length).
   - `timing`: the transmission timing in **milliseconds**, or `null`:
     `transmissionMode` (`CYCLIC`, `EVENT`, `MIXED`), `cyclic: { periodMs,
     offsetMs }`, `event: { numberOfRepetitions, repetitionPeriodMs }`,
     `minimumDelayMs`;
   - `timingSource`: `stored` (set on the PDU; a `null` timing means it was
     cleared), `imported` (read from the imported ARXML, the PDU has none of its
     own yet) or `none`.
   - Every system comes with its `sourceFile`: name it with the system.
   `list_pdus` rows carry the same `timing` and `timingSource`.
3. **The PDUs of an ECU**: `list_ecus({ modelId, search: "<ECU name>" })` for its id,
   then `list_pdus({ modelId, ecuId, direction: "tx" })` (sent) or `"rx"` (received).
4. **The PDUs of a system**: `list_systems({ modelId })` for its id (several
   "CanSystem": pick by `sourceFile`), then `list_pdus({ modelId, systemId })`.
5. **PDUs nobody sends or receives**: `list_pdus({ modelId, unassigned: true })`.
6. Lists are paged (`totalCount`, `nextOffset`). Narrow the filter rather than
   paging through hundreds.

To show the user the PDU in the app, take them to `pagePath` (the System
Description page) and tell them to type the PDU's name in **Search PDUs, frames,
ECUs...**.

## 3. Visual route

1. Open the editor (section 1).
2. Type the PDU's name into **Search PDUs, frames, ECUs...**. The table filters as you
   type.
3. Click **Expand** on its row: the row shows the PDU's bit layout and its signals,
   sorted by start bit. A signal without a start bit shows "—" and isn't drawn;
   a note above the layout counts them. The columns show **Transmitting ECUs** and
   **Receiving ECUs**. The header's badges show the PDU type, **Frame:**,
   **Controller:**, **Length: … bytes** and, when it has a timing, a timing badge
   in ms: "Cyclic 100 ms", "Cyclic 100 ms (offset 5 ms)", "Event ×3 / 10 ms",
   "Cyclic 100 ms + Event ×3", followed by " · min delay 4 ms" when one is set.
   No timing, no badge.
4. For one ECU's PDUs, open the **ECU based** tab and search the ECU's name.
5. Changes made elsewhere (MCP, another tab) show after **Refresh** (top right; its
   icon spins while it reloads).

## 4. Report

The PDU: name, qualified name, length in bytes, system and its file, frame, timing
in ms (and whether it is imported). Its
signals with start bit and bit length. Its transmitting and receiving ECUs. What is
`missing` for it to reach the bus.

## Traps

- **A signal's qualified name is usually not under its PDU's path**: imported
  signals live in `ISignal/...`. Never look up a PDU's signals with
  `search_elements` and a path prefix; use `get_pdu`.
- Several systems share a name ("CanSystem"): always say which file.
- `timing` is in **ms** here, but in **seconds** in `update_staged_element`
  (task `create-pdu-and-signals.md`). An imported PDU in a model imported before
  timing existed shows only a cyclic period and minimum delay until the ARXML is
  imported again; a renamed imported PDU shows no imported timing.
- Searchable lists in the app show at most 100 rows: type to narrow.
