<script>
  /**
   * @generics {Row extends DataTableRow = DataTableRow} Row
   * @template {DataTableRow} Row
   * @typedef {import('./DataTableTypes.d.ts').PropertyPath<Row>} DataTableKey<Row=DataTableRow>
   * @typedef {any} DataTableValue
   * @typedef {{
   *    key: DataTableKey<Row> | (string & {});
   *    empty: boolean;
   *    display?: (item: DataTableValue, row: Row) => DataTableValue;
   *    sort?: false | ((a: DataTableValue, b: DataTableValue) => number);
   *    columnMenu?: boolean;
   *    width?: string;
   *    minWidth?: string;
   * }} DataTableEmptyHeader<Row=DataTableRow>
   * @typedef {{
   *    key: DataTableKey<Row>;
   *    value: DataTableValue;
   *    display?: (item: DataTableValue, row: Row) => DataTableValue;
   *    sort?: false | ((a: DataTableValue, b: DataTableValue) => number);
   *    columnMenu?: boolean;
   *    width?: string;
   *    minWidth?: string;
   * }} DataTableNonEmptyHeader<Row=DataTableRow>
   * @typedef {DataTableNonEmptyHeader<Row> | DataTableEmptyHeader<Row>} DataTableHeader<Row=DataTableRow>
   * @typedef {{ id: any; [key: string]: DataTableValue; }} DataTableRow
   * @typedef {any} DataTableRowId
   * @typedef {{
   *    key: DataTableKey<Row> | (string & {});
   *    value: DataTableValue;
   *    display?: (item: DataTableValue, row: DataTableRow) => DataTableValue;
   * }} DataTableCell<Row=DataTableRow>
   * @slot {{ row: Row; }} expanded-row
   * @slot {{ header: DataTableNonEmptyHeader; }} cell-header
   * @slot {{ row: Row; cell: DataTableCell<Row>; rowIndex: number; cellIndex: number; }} cell
   * @event {{ header?: DataTableHeader<Row>; row?: Row; cell?: DataTableCell<Row>; }} click
   * @event {{ expanded: boolean; }} click:header--expand
   * @event {{ header: DataTableHeader<Row>; sortDirection?: "ascending" | "descending" | "none" }} click:header
   * @event {{ indeterminate: boolean; selected: boolean; }} click:header--select
   * @event {Row} click:row
   * @event {Row} mouseenter:row
   * @event {Row} mouseleave:row
   * @event {{ expanded: boolean; row: Row; }} click:row--expand
   * @event {{ selected: boolean; row: Row; }} click:row--select
   * @event {DataTableCell<Row>} click:cell
   * @restProps {div}
   */

  /**
   * Specify the data table headers
   * @type {ReadonlyArray<DataTableHeader<Row>>}
   */
  export let headers = [];

  /**
   * Specify the rows the data table should render
   * keys defined in `headers` are used for the row ids
   * @type {ReadonlyArray<Row>}
   */
  export let rows = [];

  /**
   * Set the size of the data table
   * @type {"compact" | "short" | "medium" | "tall"}
   */
  export let size = undefined;

  /** Specify the title of the data table */
  export let title = "";

  /** Specify the description of the data table */
  export let description = "";

  /**
   * Specify a name attribute for the input elements
   * in a selectable data table (radio or checkbox).
   * When the table is inside a form, this name will
   * be included in the form data on submit.
   */
  export let inputName = "ccs-" + Math.random().toString(36);

  /** Set to `true` to use zebra styles */
  export let zebra = false;

  /** Set to `true` for the sortable variant */
  export let sortable = false;

  /**
   * Specify the header key to sort by
   * @type {DataTableKey<Row>}
   */
  export let sortKey = null;

  /**
   * Specify the sort direction
   * @type {"none" | "ascending" | "descending"}
   */
  export let sortDirection = "none";

  /**
   * Set to `true` for the expandable variant
   * Automatically set to `true` if `batchExpansion` is `true`
   */
  export let expandable = false;

  /**
   * Set to `true` to enable batch expansion
   */
  export let batchExpansion = false;

  /**
   * Specify the row ids to be expanded
   * @type {ReadonlyArray<DataTableRowId>}
   */
  export let expandedRowIds = [];

  /**
   * Specify the ids for rows that should not be expandable
   * @type {ReadonlyArray<DataTableRowId>}
   */
  export let nonExpandableRowIds = [];

  /** Set to `true` for the radio selection variant */
  export let radio = false;

  /**
   * Set to `true` for the selectable variant
   * Automatically set to `true` if `radio` or `batchSelection` are `true`
   */
  export let selectable = false;

  /** Set to `true` to enable batch selection */
  export let batchSelection = false;

  /**
   * Specify the row ids to be selected
   * @type {ReadonlyArray<DataTableRowId>}
   */
  export let selectedRowIds = [];

  /**
   * Specify the ids of rows that should not be selectable
   * @type {ReadonlyArray<DataTableRowId>}
   */
  export let nonSelectableRowIds = [];

  /** Set to `true` to enable a sticky header */
  export let stickyHeader = false;

  /** Set to `true` to use static width */
  export let useStaticWidth = false;

  /** Specify the number of items to display in a page */
  export let pageSize = 0;

  /** Set to `number` to set current page */
  export let page = 0;

  import { createEventDispatcher, setContext } from "svelte";
  import { writable } from "svelte/store";
  import ChevronRight from "../icons/ChevronRight.svelte";
  import InlineCheckbox from "../Checkbox/InlineCheckbox.svelte";
  import RadioButton from "../RadioButton/RadioButton.svelte";
  import Table from "./Table.svelte";
  import TableBody from "./TableBody.svelte";
  import TableCell from "./TableCell.svelte";
  import TableContainer from "./TableContainer.svelte";
  import TableHead from "./TableHead.svelte";
  import TableHeader from "./TableHeader.svelte";
  import TableRow from "./TableRow.svelte";

  const sortDirectionMap = {
    none: "ascending",
    ascending: "descending",
    descending: "none",
  };
  const dispatch = createEventDispatcher();
  const batchSelectedIds = writable(false);
  const tableRows = writable(rows);

  // Internal ID prefix for radio buttons, checkboxes, etc.
  // since there may be multiple `DataTable` instances that have overlapping row ids.
  const id = "ccs-" + Math.random().toString(36);

  // Store a copy of the original rows for filter restoration.
  $: originalRows = [...rows];

  $: thKeys = headers.reduce((a, c) => ({ ...a, [c.key]: c.key }), {});
  const resolvePath = (object, path) => {
    if (path in object) return object[path];
    return path
      .split(/[\.\[\]\'\"]/)
      .filter((p) => p)
      .reduce((o, p) => (o && typeof o === "object" ? o[p] : o), object);
  };

  setContext("DataTable", {
    batchSelectedIds,
    tableRows,
    resetSelectedRowIds: () => {
      selectAll = false;
      selectedRowIds = [];
      if (refSelectAll) refSelectAll.checked = false;
    },
    filterRows: (searchValue, customFilter) => {
      const value = searchValue.trim().toLowerCase();

      if (value.length === 0) {
        // Reset to original rows.
        tableRows.set(originalRows);
        return originalRows.map((row) => row.id);
      }

      let filteredRows = [];

      if (typeof customFilter === "function") {
        // Apply custom filter if provided.
        filteredRows = originalRows.filter((row) => customFilter(row, value));
      } else {
        // Default filter checks all non-id fields for a basic, case-insensitive match (non-fuzzy).
        filteredRows = originalRows.filter((row) => {
          return Object.entries(row)
            .filter(([key]) => key !== "id")
            .some(([key, _value]) => {
              if (typeof _value === "string" || typeof _value === "number") {
                return (_value + "")?.toLowerCase().includes(value);
              }
            });
        });
      }

      tableRows.set(filteredRows);
      return filteredRows.map((row) => row.id);
    },
  });

  let expanded = false;
  let parentRowId = null;

  $: expandedRows = expandedRowIds.reduce(
    (a, id) => ({ ...a, [id]: true }),
    {},
  );

  let refSelectAll = null;

  $: batchSelectedIds.set(selectedRowIds);
  $: rowIds = $tableRows.map((row) => row.id);
  $: expandableRowIds = rowIds.filter(
    (id) => !nonExpandableRowIds.includes(id),
  );
  $: selectableRowIds = rowIds.filter(
    (id) => !nonSelectableRowIds.includes(id),
  );
  $: selectAll =
    selectableRowIds.length > 0 &&
    selectedRowIds.length === selectableRowIds.length;
  $: indeterminate =
    selectedRowIds.length > 0 &&
    selectedRowIds.length < selectableRowIds.length;
  $: if (batchExpansion) {
    expandable = true;
    expanded = expandedRowIds.length === expandableRowIds.length;
  }
  $: if (radio || batchSelection) selectable = true;
  $: headerKeys = headers.map(({ key }) => key);
  $: tableCellsByRowId = rows.reduce((rows, row) => {
    rows[row.id] = headerKeys.map((key, index) => ({
      key,
      value: resolvePath(row, key),
      display: headers[index].display,
    }));
    return rows;
  }, {});
  $: $tableRows = rows;
  $: sortedRows = [...$tableRows];
  $: ascending = sortDirection === "ascending";
  $: sorting = sortable && sortKey != null;
  $: sortingHeader = headers.find((header) => header.key === sortKey);
  $: if (sorting) {
    if (sortDirection === "none") {
      sortedRows = $tableRows;
    } else {
      sortedRows = [...$tableRows].sort((a, b) => {
        const itemA = ascending
          ? resolvePath(a, sortKey)
          : resolvePath(b, sortKey);
        const itemB = ascending
          ? resolvePath(b, sortKey)
          : resolvePath(a, sortKey);

        if (sortingHeader?.sort) return sortingHeader.sort(itemA, itemB);

        if (typeof itemA === "number" && typeof itemB === "number")
          return itemA - itemB;

        if ([itemA, itemB].every((item) => !item && item !== 0)) return 0;
        if (!itemA && itemA !== 0) return ascending ? 1 : -1;
        if (!itemB && itemB !== 0) return ascending ? -1 : 1;

        return itemA
          .toString()
          .localeCompare(itemB.toString(), "en", { numeric: true });
      });
    }
  }
  const getDisplayedRows = (rows, page, pageSize) =>
    page && pageSize
      ? rows.slice((page - 1) * pageSize, page * pageSize)
      : rows;
  $: displayedRows = getDisplayedRows($tableRows, page, pageSize);
  $: displayedSortedRows = getDisplayedRows(sortedRows, page, pageSize);

  $: hasCustomHeaderWidth = headers.some(
    (header) => header.width || header.minWidth,
  );

  /** @type {(header: DataTableHeader) => undefined | string} */
  const formatHeaderWidth = (header) => {
    const styles = [
      header.width && `width: ${header.width}`,
      header.minWidth && `min-width: ${header.minWidth}`,
    ].filter(Boolean);
    if (styles.length === 0) return undefined;
    return styles.join(";");
  };
</script>

<TableContainer {useStaticWidth} {...$$restProps}>
  {#if title || $$slots.title || description || $$slots.description}
    <div class:bx--data-table-header={true}>
      {#if title || $$slots.title}
        <h4 class:bx--data-table-header__title={true}>
          <slot name="title">{title}</slot>
        </h4>
      {/if}
      {#if description || $$slots.description}
        <p class:bx--data-table-header__description={true}>
          <slot name="description">{description}</slot>
        </p>
      {/if}
    </div>
  {/if}
  <slot />
  <Table
    {zebra}
    {size}
    {stickyHeader}
    {sortable}
    {useStaticWidth}
    tableStyle={hasCustomHeaderWidth && "table-layout: fixed"}
  >
    <TableHead>
      <TableRow>
        {#if expandable}
          <th
            scope="col"
            class:bx--table-expand={true}
            data-previous-value={expanded ? "collapsed" : undefined}
          >
            {#if batchExpansion}
              <button
                type="button"
                class:bx--table-expand__button={true}
                aria-label={expanded ? "Collapse all rows" : "Expand all rows"}
                aria-controls={expandableRowIds
                  .map((id) => `expandable-row-${id}`)
                  .join(" ")}
                on:click={() => {
                  expanded = !expanded;
                  expandedRowIds = expanded ? expandableRowIds : [];

                  dispatch("click:header--expand", { expanded });
                }}
              >
                <ChevronRight
                  aria-hidden="true"
                  class="bx--table-expand__svg"
                />
              </button>
            {/if}
          </th>
        {/if}
        {#if selectable && !batchSelection}
          <th scope="col"></th>
        {/if}
        {#if batchSelection && !radio}
          <th scope="col" class:bx--table-column-checkbox={true}>
            <InlineCheckbox
              bind:ref={refSelectAll}
              aria-label="Select all rows"
              name="{id}-select-all"
              value="all"
              checked={selectAll}
              {indeterminate}
              on:change={(e) => {
                dispatch("click:header--select", {
                  indeterminate,
                  selected: !indeterminate && e.target.checked,
                });

                if (indeterminate) {
                  e.target.checked = false;
                  selectAll = false;
                  selectedRowIds = [];
                  return;
                }

                if (e.target.checked) {
                  selectedRowIds = selectableRowIds;
                } else {
                  selectedRowIds = [];
                }
              }}
            />
          </th>
        {/if}
        {#each headers as header (header.key)}
          {#if header.empty}
            <th scope="col" style={formatHeaderWidth(header)}></th>
          {:else}
            <TableHeader
              id={header.key}
              style={formatHeaderWidth(header)}
              sortable={sortable && header.sort !== false}
              sortDirection={sortKey === header.key ? sortDirection : "none"}
              active={sortKey === header.key}
              on:click={() => {
                dispatch("click", { header });

                if (header.sort === false) {
                  dispatch("click:header", { header });
                } else {
                  let currentSortDirection =
                    sortKey === header.key ? sortDirection : "none";
                  sortDirection = sortDirectionMap[currentSortDirection];
                  sortKey =
                    sortDirection === "none" ? null : thKeys[header.key];
                  dispatch("click:header", { header, sortDirection });
                }
              }}
            >
              <slot name="cell-header" {header}>{header.value}</slot>
            </TableHeader>
          {/if}
        {/each}
      </TableRow>
    </TableHead>
    <TableBody>
      {#each sorting ? displayedSortedRows : displayedRows as row, i (row.id)}
        <TableRow
          data-row={row.id}
          data-parent-row={expandable ? true : undefined}
          class="{selectedRowIds.includes(row.id)
            ? 'bx--data-table--selected'
            : ''} {expandedRows[row.id] ? 'bx--expandable-row' : ''} {expandable
            ? 'bx--parent-row'
            : ''} {expandable && parentRowId === row.id
            ? 'bx--expandable-row--hover'
            : ''}"
          on:click={({ target }) => {
            // forgo "click", "click:row" events if target
            // resembles an overflow menu, a checkbox, or radio button
            if (
              [...target.classList].some((name) =>
                /^bx--(overflow-menu|checkbox|radio-button)/.test(name),
              )
            ) {
              return;
            }
            dispatch("click", { row });
            dispatch("click:row", row);
          }}
          on:mouseenter={() => {
            dispatch("mouseenter:row", row);
          }}
          on:mouseleave={() => {
            dispatch("mouseleave:row", row);
          }}
        >
          {#if expandable}
            <TableCell
              class="bx--table-expand"
              headers="expand"
              data-previous-value={!nonExpandableRowIds.includes(row.id) &&
              expandedRows[row.id]
                ? "collapsed"
                : undefined}
            >
              {#if !nonExpandableRowIds.includes(row.id)}
                <button
                  type="button"
                  class:bx--table-expand__button={true}
                  aria-controls={`expandable-row-${row.id}`}
                  aria-label={expandedRows[row.id]
                    ? "Collapse current row"
                    : "Expand current row"}
                  on:click|stopPropagation={() => {
                    const rowExpanded = !!expandedRows[row.id];

                    expandedRowIds = rowExpanded
                      ? expandedRowIds.filter((id) => id !== row.id)
                      : [...expandedRowIds, row.id];

                    dispatch("click:row--expand", {
                      row,
                      expanded: !rowExpanded,
                    });
                  }}
                >
                  <ChevronRight
                    aria-hidden="true"
                    class="bx--table-expand__svg"
                  />
                </button>
              {/if}
            </TableCell>
          {/if}
          {#if selectable}
            <td
              class:bx--table-column-checkbox={true}
              class:bx--table-column-radio={radio}
            >
              {#if !nonSelectableRowIds.includes(row.id)}
                {@const inputId = `${id}-${row.id}`}
                {#if radio}
                  <RadioButton
                    id={inputId}
                    name={inputName}
                    checked={selectedRowIds.includes(row.id)}
                    value={row.id}
                    on:change={() => {
                      selectedRowIds = [row.id];
                      dispatch("click:row--select", { row, selected: true });
                    }}
                  />
                {:else}
                  <InlineCheckbox
                    id={inputId}
                    name={inputName}
                    checked={selectedRowIds.includes(row.id)}
                    value={row.id}
                    on:change={() => {
                      if (selectedRowIds.includes(row.id)) {
                        selectedRowIds = selectedRowIds.filter(
                          (id) => id !== row.id,
                        );
                        dispatch("click:row--select", { row, selected: false });
                      } else {
                        selectedRowIds = [...selectedRowIds, row.id];
                        dispatch("click:row--select", { row, selected: true });
                      }
                    }}
                  />
                {/if}
              {/if}
            </td>
          {/if}
          {#each tableCellsByRowId[row.id] as cell, j (cell.key)}
            {#if headers[j].empty}
              <td class:bx--table-column-menu={headers[j].columnMenu}>
                <slot name="cell" {row} {cell} rowIndex={i} cellIndex={j}>
                  {cell.display ? cell.display(cell.value, row) : cell.value}
                </slot>
              </td>
            {:else}
              <TableCell
                on:click={() => {
                  dispatch("click", { row, cell });
                  dispatch("click:cell", cell);
                }}
              >
                <slot name="cell" {row} {cell} rowIndex={i} cellIndex={j}>
                  {cell.display ? cell.display(cell.value, row) : cell.value}
                </slot>
              </TableCell>
            {/if}
          {/each}
        </TableRow>

        {#if expandable}
          <tr
            id={`expandable-row-${row.id}`}
            data-child-row
            class:bx--expandable-row={true}
            on:mouseenter={() => {
              if (nonExpandableRowIds.includes(row.id)) return;
              parentRowId = row.id;
            }}
            on:mouseleave={() => {
              if (nonExpandableRowIds.includes(row.id)) return;
              parentRowId = null;
            }}
          >
            {#if expandedRows[row.id] && !nonExpandableRowIds.includes(row.id)}
              <TableCell
                colspan={selectable ? headers.length + 2 : headers.length + 1}
              >
                <div class:bx--child-row-inner-container={true}>
                  <slot name="expanded-row" {row} />
                </div>
              </TableCell>
            {/if}
          </tr>
        {/if}
      {/each}
    </TableBody>
  </Table>
</TableContainer>
