---
name: tanstack-table
description: >
  Build powerful, fully-featured data tables and datagrids using TanStack Table v8
  (formerly React Table). Use this skill whenever the user wants to create a table
  component, data grid, or any tabular UI with features like sorting, filtering,
  pagination, row selection, column pinning, column visibility, resizing, grouping,
  or virtualization. Trigger even for simpler table requests ("just show my data in
  a table") since TanStack Table is the right tool and this skill encodes the correct
  patterns. Also use for server-side data fetching with tables, editable cells, or
  integrating tables with shadcn/ui, Material UI, or Tailwind CSS.
---

# TanStack Table v8 Skill

TanStack Table is a **headless UI library** — it provides all the logic, state, and APIs
for tables, but zero markup or styles. You wire up the rendering yourself.

---

## Rules

| #   | Rule                                                                                 | Why                                                  |
| --- | ------------------------------------------------------------------------------------ | ---------------------------------------------------- |
| 1   | Define `columns` **outside** the component (or `useMemo`)                            | Inline arrays cause infinite re-renders              |
| 2   | Always use `flexRender()` for headers and cells                                      | Headers can be strings, functions, or JSX            |
| 3   | `id` is **required** when using `accessorFn`                                         | Table can't auto-derive an id from a function        |
| 4   | Always include `getCoreRowModel()`                                                   | Required for the table to work at all                |
| 5   | Never mutate `data` directly — return a new array                                    | Table uses reference equality to detect changes      |
| 6   | Check `header.isPlaceholder` before rendering                                        | Grouped columns create empty placeholder headers     |
| 7   | Use `initialState` for defaults; `state` only when you need external control         | Over-controlling state adds unnecessary complexity   |
| 8   | For server-side, set `manualSorting`, `manualFiltering`, `manualPagination` together | Mixing client + server processing causes subtle bugs |
| 9   | Pass `rowCount` for server-side pagination                                           | Without it, `getPageCount()` returns `-1`            |
| 10  | Reset `pageIndex: 0` when filters or sorting change                                  | Users should land on page 1 after filtering          |
| 11  | Add row models in order: core → filter → group → sort → expand → paginate            | Each model feeds into the next                       |
| 12  | Virtualize rows for 1000+ rows (`@tanstack/react-virtual`)                           | Rendering thousands of DOM rows kills performance    |
| 13  | Set `enableSorting: false` on non-data columns (actions, checkboxes)                 | Prevents accidental sort on UI-only columns          |
| 14  | Use `createColumnHelper<T>()` in TypeScript projects                                 | Gives fully typed `getValue()` without casting       |

---

## Installation

```bash
# React (most common)
npm install @tanstack/react-table

# Vue 3
npm install @tanstack/vue-table

# Svelte
npm install @tanstack/svelte-table

# Solid
npm install @tanstack/solid-table

# Vanilla JS
npm install @tanstack/table-core
```

---

## Core Concepts

### 1. Column Definitions

Always define columns **outside** the component (or memoize them) to avoid infinite re-renders.

```tsx
import { ColumnDef } from "@tanstack/react-table";

type Person = {
  id: number;
  name: string;
  age: number;
  status: "active" | "inactive";
};

// accessor columns (most common)
const columns: ColumnDef<Person>[] = [
  {
    accessorKey: "name", // maps to data.name
    header: "Name",
    cell: (info) => info.getValue(),
  },
  {
    accessorFn: (row) => row.age, // functional accessor
    id: "age", // id required when using accessorFn
    header: () => <span>Age</span>,
    cell: (info) => info.getValue(),
  },
  // display column (no data accessor — for actions, checkboxes, etc.)
  {
    id: "actions",
    header: "Actions",
    cell: ({ row }) => (
      <button onClick={() => handleEdit(row.original)}>Edit</button>
    ),
  },
  // group column (nests other columns)
  {
    header: "Info",
    columns: [
      { accessorKey: "name", header: "Name" },
      { accessorKey: "age", header: "Age" },
    ],
  },
];
```

### 2. Minimal Table Setup (React)

```tsx
import {
  useReactTable,
  getCoreRowModel,
  flexRender,
  ColumnDef,
} from "@tanstack/react-table";

function MyTable({ data }: { data: Person[] }) {
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
  });

  return (
    <table>
      <thead>
        {table.getHeaderGroups().map((headerGroup) => (
          <tr key={headerGroup.id}>
            {headerGroup.headers.map((header) => (
              <th key={header.id} colSpan={header.colSpan}>
                {header.isPlaceholder
                  ? null
                  : flexRender(
                      header.column.columnDef.header,
                      header.getContext(),
                    )}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody>
        {table.getRowModel().rows.map((row) => (
          <tr key={row.id}>
            {row.getVisibleCells().map((cell) => (
              <td key={cell.id}>
                {flexRender(cell.column.columnDef.cell, cell.getContext())}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

**Key rule:** Always use `flexRender()` to render header and cell content — it handles
both JSX elements and plain values correctly.

---

## Feature Recipes

### Sorting

```tsx
import { getSortedRowModel, SortingState } from '@tanstack/react-table'

const [sorting, setSorting] = useState<SortingState>([])

const table = useReactTable({
  data,
  columns,
  state: { sorting },
  onSortingChange: setSorting,
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
})

// In the header:
<th onClick={header.column.getToggleSortingHandler()} style={{ cursor: 'pointer' }}>
  {flexRender(header.column.columnDef.header, header.getContext())}
  {{ asc: ' 🔼', desc: ' 🔽' }[header.column.getIsSorted() as string] ?? null}
</th>

// Disable sorting per column:
{ accessorKey: 'id', header: 'ID', enableSorting: false }
```

### Column Filtering

```tsx
import { getFilteredRowModel, ColumnFiltersState } from '@tanstack/react-table'

const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([])

const table = useReactTable({
  data,
  columns,
  state: { columnFilters },
  onColumnFiltersChange: setColumnFilters,
  getCoreRowModel: getCoreRowModel(),
  getFilteredRowModel: getFilteredRowModel(),
})

// Filter input for a column:
<input
  value={(table.getColumn('name')?.getFilterValue() as string) ?? ''}
  onChange={e => table.getColumn('name')?.setFilterValue(e.target.value)}
  placeholder="Filter name..."
/>

// Custom filter function per column:
{
  accessorKey: 'status',
  filterFn: 'equals',  // built-ins: 'auto','equals','includesString','weakEquals', etc.
}
```

### Global Filtering

```tsx
import { getFilteredRowModel } from '@tanstack/react-table'

const [globalFilter, setGlobalFilter] = useState('')

const table = useReactTable({
  data,
  columns,
  state: { globalFilter },
  onGlobalFilterChange: setGlobalFilter,
  globalFilterFn: 'includesString', // or a custom fn
  getCoreRowModel: getCoreRowModel(),
  getFilteredRowModel: getFilteredRowModel(),
})

<input value={globalFilter} onChange={e => setGlobalFilter(e.target.value)} />
```

### Pagination

```tsx
import { getPaginationRowModel, PaginationState } from '@tanstack/react-table'

const [pagination, setPagination] = useState<PaginationState>({
  pageIndex: 0,
  pageSize: 10,
})

const table = useReactTable({
  data,
  columns,
  state: { pagination },
  onPaginationChange: setPagination,
  getCoreRowModel: getCoreRowModel(),
  getPaginationRowModel: getPaginationRowModel(),
})

// Controls:
<button onClick={() => table.previousPage()} disabled={!table.getCanPreviousPage()}>
  Previous
</button>
<span>Page {table.getState().pagination.pageIndex + 1} of {table.getPageCount()}</span>
<button onClick={() => table.nextPage()} disabled={!table.getCanNextPage()}>
  Next
</button>
<select value={table.getState().pagination.pageSize}
  onChange={e => table.setPageSize(Number(e.target.value))}>
  {[10, 20, 50].map(size => <option key={size} value={size}>Show {size}</option>)}
</select>
```

### Row Selection

```tsx
import { RowSelectionState } from "@tanstack/react-table";

const [rowSelection, setRowSelection] = useState<RowSelectionState>({});

// Add a checkbox column:
const selectionColumn: ColumnDef<Person> = {
  id: "select",
  header: ({ table }) => (
    <input
      type="checkbox"
      checked={table.getIsAllPageRowsSelected()}
      ref={(el) => {
        if (el) el.indeterminate = table.getIsSomePageRowsSelected();
      }}
      onChange={table.getToggleAllPageRowsSelectedHandler()}
    />
  ),
  cell: ({ row }) => (
    <input
      type="checkbox"
      checked={row.getIsSelected()}
      disabled={!row.getCanSelect()}
      onChange={row.getToggleSelectedHandler()}
    />
  ),
};

const table = useReactTable({
  data,
  columns: [selectionColumn, ...columns],
  state: { rowSelection },
  onRowSelectionChange: setRowSelection,
  getCoreRowModel: getCoreRowModel(),
  enableRowSelection: true, // or: row => row.original.age >= 18
  enableMultiRowSelection: true,
});

// Get selected rows:
const selectedRows = table.getSelectedRowModel().rows.map((r) => r.original);
```

### Column Visibility

```tsx
import { VisibilityState } from "@tanstack/react-table";

const [columnVisibility, setColumnVisibility] = useState<VisibilityState>({});

const table = useReactTable({
  data,
  columns,
  state: { columnVisibility },
  onColumnVisibilityChange: setColumnVisibility,
  getCoreRowModel: getCoreRowModel(),
});

// Toggle UI:
{
  table.getAllColumns().map((column) => (
    <label key={column.id}>
      <input
        type="checkbox"
        checked={column.getIsVisible()}
        onChange={column.getToggleVisibilityHandler()}
      />
      {column.id}
    </label>
  ));
}
```

### Column Pinning

```tsx
import { ColumnPinningState } from "@tanstack/react-table";

const [columnPinning, setColumnPinning] = useState<ColumnPinningState>({
  left: ["name"], // pin columns to left
  right: ["actions"], // pin columns to right
});

const table = useReactTable({
  data,
  columns,
  state: { columnPinning },
  onColumnPinningChange: setColumnPinning,
  getCoreRowModel: getCoreRowModel(),
});

// In headers, use table.getLeftHeaderGroups(), getCenterHeaderGroups(), getRightHeaderGroups()
// In rows, use row.getLeftVisibleCells(), getCenterVisibleCells(), getRightVisibleCells()
```

### Column Resizing

```tsx
import { ColumnResizeMode } from '@tanstack/react-table'

const [columnResizeMode] = useState<ColumnResizeMode>('onChange') // or 'onEnd'

const table = useReactTable({
  data,
  columns,
  columnResizeMode,
  getCoreRowModel: getCoreRowModel(),
  enableColumnResizing: true,
})

// Render resize handle in each header:
<th style={{ width: header.getSize() }}>
  {flexRender(header.column.columnDef.header, header.getContext())}
  <div
    onMouseDown={header.getResizeHandler()}
    onTouchStart={header.getResizeHandler()}
    className={`resizer ${header.column.getIsResizing() ? 'isResizing' : ''}`}
  />
</th>

// Per-column width config in column def:
{ accessorKey: 'name', size: 200, minSize: 100, maxSize: 400 }
```

### Expanding / Sub-rows

```tsx
import { getExpandedRowModel, ExpandedState } from '@tanstack/react-table'

const [expanded, setExpanded] = useState<ExpandedState>({})

const table = useReactTable({
  data,
  columns,
  state: { expanded },
  onExpandedChange: setExpanded,
  getCoreRowModel: getCoreRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
  getSubRows: row => row.subRows, // tell table how to find sub-rows
})

// Expand toggle in a cell:
<button onClick={row.getToggleExpandedHandler()}>
  {row.getIsExpanded() ? '▼' : '▶'}
</button>
```

### Grouping

```tsx
import {
  getGroupedRowModel,
  getExpandedRowModel,
  GroupingState,
} from "@tanstack/react-table";

const [grouping, setGrouping] = useState<GroupingState>([]);

const table = useReactTable({
  data,
  columns,
  state: { grouping },
  onGroupingChange: setGrouping,
  getCoreRowModel: getCoreRowModel(),
  getGroupedRowModel: getGroupedRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
});

// In rows, check row.getIsGrouped() to render group headers differently
```

---

## Server-Side Data (Sorting, Filtering, Pagination)

For large datasets, disable client-side processing and handle server-side:

```tsx
const [sorting, setSorting] = useState<SortingState>([]);
const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([]);
const [pagination, setPagination] = useState<PaginationState>({
  pageIndex: 0,
  pageSize: 10,
});

// Fetch data based on state (e.g. with React Query)
const { data, isLoading } = useQuery({
  queryKey: ["people", sorting, columnFilters, pagination],
  queryFn: () => fetchPeople({ sorting, columnFilters, pagination }),
});

const table = useReactTable({
  data: data?.rows ?? [],
  columns,
  rowCount: data?.totalCount, // required for server-side pagination page count
  state: { sorting, columnFilters, pagination },
  onSortingChange: setSorting,
  onColumnFiltersChange: setColumnFilters,
  onPaginationChange: setPagination,
  getCoreRowModel: getCoreRowModel(),
  manualSorting: true, // disable client-side sorting
  manualFiltering: true, // disable client-side filtering
  manualPagination: true, // disable client-side pagination
});
```

---

## State Management Tips

```tsx
// Access all table state at once:
console.log(table.getState());

// Provide initial state without controlling it:
const table = useReactTable({
  data,
  columns,
  initialState: {
    sorting: [{ id: "name", desc: false }],
    pagination: { pageIndex: 0, pageSize: 20 },
    columnVisibility: { id: false },
  },
  getCoreRowModel: getCoreRowModel(),
});

// Fully controlled state (own all state externally):
const table = useReactTable({
  data,
  columns,
  state: { sorting, pagination }, // pass all state in
  onSortingChange: setSorting, // handle all changes
  onPaginationChange: setPagination,
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
  getPaginationRowModel: getPaginationRowModel(),
});
```

---

## Row Models (in order of processing)

TanStack Table processes row models in this order:

1. `getCoreRowModel()` — **required always**
2. `getFilteredRowModel()` — after filtering
3. `getGroupedRowModel()` — after grouping
4. `getSortedRowModel()` — after sorting
5. `getExpandedRowModel()` — after expanding
6. `getPaginationRowModel()` — last, for current page

Import only the row models you need. They compose automatically.

---

## TypeScript Tips

```tsx
// Declare your data type once and use it throughout:
type User = { id: string; name: string; role: string };

// Typed column helper (cleaner than ColumnDef<User>[]):
import { createColumnHelper } from "@tanstack/react-table";
const columnHelper = createColumnHelper<User>();

const columns = [
  columnHelper.accessor("name", {
    header: "Name",
    cell: (info) => info.getValue(), // getValue() is typed as string
  }),
  columnHelper.accessor((row) => row.role.toUpperCase(), {
    id: "role",
    header: "Role",
  }),
  columnHelper.display({
    id: "actions",
    cell: (props) => <RowActions row={props.row} />,
  }),
];
```

---

## Common Pitfalls

| Problem                           | Fix                                                                          |
| --------------------------------- | ---------------------------------------------------------------------------- |
| Infinite re-renders               | Define `columns` and `data` outside the component, or memoize with `useMemo` |
| Pagination not working            | Make sure `getPaginationRowModel()` is added to `useReactTable`              |
| Sorting doesn't update            | Pass `state: { sorting }` and `onSortingChange: setSorting`                  |
| `flexRender` not needed?          | Always use it — plain strings break with JSX column defs                     |
| Server-side: page count wrong     | Pass `rowCount: totalFromServer` to `useReactTable`                          |
| Column filter not clearing        | Call `column.setFilterValue(undefined)` to clear (not `''`)                  |
| TypeScript errors on `getValue()` | Use `createColumnHelper<T>()` for better inference                           |

---

## Integration with UI Libraries

**shadcn/ui (Tailwind):** Use `<Table>`, `<TableHead>`, `<TableBody>` etc. from shadcn — wire
TanStack Table APIs into those components. Follow the shadcn [Data Table guide](https://ui.shadcn.com/docs/components/data-table).

**Material UI:** Use MUI `<Table>` components for markup; plug in TanStack Table's state and
row model APIs.

**Tailwind CSS:** Apply utility classes directly to `<table>`, `<th>`, `<td>` — TanStack Table
is fully style-agnostic.

---

## Useful APIs Quick Reference

```tsx
// Table instance
table.getState(); // all current state
table.getHeaderGroups(); // for thead rendering
table.getRowModel().rows; // for tbody rendering (current page if paginated)
table.getSelectedRowModel().rows; // selected rows
table.getPageCount(); // total pages (pagination)
table.setGlobalFilter(value); // programmatic filter

// Column
column.getFilterValue(); // current filter value
column.setFilterValue(value); // set filter
column.getToggleSortingHandler(); // click handler for sort
column.getIsSorted(); // 'asc' | 'desc' | false
column.getToggleVisibilityHandler(); // click handler for visibility
column.getIsVisible();

// Row
row.original; // raw data object
row.getVisibleCells(); // cells for this row
row.getIsSelected();
row.getToggleSelectedHandler();
row.getIsExpanded();
row.getToggleExpandedHandler();
```

---

## Docs Reference

- Full docs: https://tanstack.com/table/v8/docs
- Examples: https://tanstack.com/table/v8/docs/framework/react/examples/basic
- API reference: https://tanstack.com/table/v8/docs/api/core/table
