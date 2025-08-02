# use-sort-params

> **Reference Implementation**: [apps/dashboard/src/hooks/use-sort-params.ts](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/use-sort-params.ts)

## Purpose

The `use-sort-params` hook provides URL-synchronized sorting state management for data tables. It implements a three-state sorting system (ascending, descending, null) with URL persistence, enabling shareable sorted views and consistent sorting behavior across different table implementations in the application.

## Key Features

- Three-state sorting logic (asc → desc → null → asc)
- URL synchronization with automatic parameter management
- Multi-column sorting support through array-based parameters
- Server-side rendering compatibility with loader function
- Clean URL management with automatic parameter cleanup
- Type-safe parameter handling with nuqs integration

## Dependencies

### External Dependencies

- `nuqs` - URL state management library for client-side operations
- `nuqs/server` - Server-side utilities for SSR support and parameter loading

### Internal Dependencies

- Used by all data table header components for sorting functionality
- Integrated with table components for sort state management
- Works alongside filter hooks for comprehensive table state management

## Implementation Details

### Sort Schema Definition

The hook uses a simple but flexible schema for sort parameters:

```typescript
export const sortParamsSchema = {
  sort: parseAsArrayOf(parseAsString),
};
```

**Schema Structure**:

- `sort`: Array of strings representing `[column, direction]` pairs
- Supports multi-column sorting through array format
- Uses `parseAsArrayOf(parseAsString)` for type-safe URL parsing

### URL Parameter Management

The hook leverages `nuqs` for automatic URL synchronization:

```typescript
export function useSortParams() {
  const [params, setParams] = useQueryStates(sortParamsSchema);
  return { params, setParams };
}
```

**URL Format Examples**:

- Single column ascending: `?sort=date,asc`
- Single column descending: `?sort=amount,desc`
- No sorting: URL parameter is omitted
- Multi-column: `?sort=date,asc&sort=amount,desc` (future extension)

### Three-State Sorting Implementation

The hook enables a three-state sorting cycle commonly implemented in table headers:

```typescript
// Example implementation in data-table-header.tsx
const createSortQuery = useCallback(
  (name: string) => {
    if (value === "asc") {
      // If currently ascending, switch to descending
      setParams({ sort: [name, "desc"] });
    } else if (value === "desc") {
      // If currently descending, clear sort
      setParams({ sort: null });
    } else {
      // If not sorted on this column, set to ascending
      setParams({ sort: [name, "asc"] });
    }
  },
  [value, setParams]
);
```

**State Transitions**:

1. **No Sort** → **Ascending**: First click on column header
2. **Ascending** → **Descending**: Second click on same column
3. **Descending** → **No Sort**: Third click clears sorting
4. **Any State** → **Ascending**: Click on different column

### Multi-Column Sorting Logic

While currently implemented for single-column sorting, the array-based schema supports multi-column sorting:

```typescript
// Current single-column usage
const [column, value] = params.sort || [];

// Potential multi-column extension
const sortColumns = params.sort
  ? params.sort.reduce((acc, item, index) => {
      if (index % 2 === 0) {
        acc.push({ column: item, direction: params.sort[index + 1] });
      }
      return acc;
    }, [])
  : [];
```

### URL Synchronization Logic

The hook automatically manages URL parameters:

**Parameter Addition**:

- Adds `sort` parameter when sorting is applied
- Updates existing parameter when sort changes
- Maintains clean URL format with proper encoding

**Parameter Removal**:

- Removes `sort` parameter when sorting is cleared
- Prevents empty or null values in URL
- Maintains URL cleanliness for sharing

### Sort State Persistence

Sort state persists across:

- Page refreshes through URL parameters
- Browser navigation (back/forward buttons)
- Direct URL sharing and bookmarking
- Server-side rendering with proper hydration

## Performance Considerations

- **URL Updates**: Debounced by `nuqs` to prevent excessive browser history entries
- **State Synchronization**: Minimal re-renders through efficient state management
- **Memory Usage**: Lightweight state with only active sort parameters stored
- **SSR Optimization**: Server-side loader prevents hydration mismatches

## Integration Patterns

### With Table Header Components

```typescript
// In data-table-header.tsx
function DataTableHeader() {
  const { params, setParams } = useSortParams();
  const [column, value] = params.sort || [];

  const createSortQuery = useCallback(
    (columnName: string) => {
      if (value === "asc") {
        setParams({ sort: [columnName, "desc"] });
      } else if (value === "desc") {
        setParams({ sort: null });
      } else {
        setParams({ sort: [columnName, "asc"] });
      }
    },
    [value, setParams],
  );

  return (
    <TableHead>
      <Button onClick={() => createSortQuery("date")}>
        <span>Date</span>
        {"date" === column && value === "asc" && <ArrowDown />}
        {"date" === column && value === "desc" && <ArrowUp />}
      </Button>
    </TableHead>
  );
}
```

### With Data Fetching

```typescript
// In data-table.tsx
function TransactionTable() {
  const { params } = useSortParams();
  const [sortColumn, sortDirection] = params.sort || [];

  const { data } = useTransactions({
    sortBy: sortColumn,
    sortOrder: sortDirection,
    // other parameters
  });

  return <Table data={data} />;
}
```

### Server-Side Usage

```typescript
// In page.tsx (App Router)
export default async function TransactionsPage({ searchParams }) {
  const sortParams = loadSortParams(searchParams);
  const [sortColumn, sortDirection] = sortParams.sort || [];

  // Use sort parameters for server-side data fetching
  const initialData = await fetchTransactions({
    sortBy: sortColumn,
    sortOrder: sortDirection,
  });

  return <TransactionTable initialData={initialData} />;
}
```

### Visual Sort Indicators

```typescript
// Sort indicator component
function SortIndicator({ column, currentSort }: {
  column: string;
  currentSort: [string, string] | undefined;
}) {
  const [sortColumn, sortDirection] = currentSort || [];

  if (sortColumn !== column) return null;

  return (
    <span className="ml-2">
      {sortDirection === "asc" && <ArrowDown size={16} />}
      {sortDirection === "desc" && <ArrowUp size={16} />}
    </span>
  );
}
```

## Error Handling

- **Invalid Parameters**: `nuqs` parsers handle malformed URL parameters gracefully
- **Missing Values**: Defaults to no sorting when parameters are invalid
- **Type Safety**: Schema validation prevents runtime errors from incorrect parameter types
- **Browser Compatibility**: Graceful fallback for browsers with limited URL API support

## Related Files

- `data-table-header.tsx` - Primary consumer implementing sort UI
- `data-table.tsx` - Uses sort parameters for data fetching
- `use-transaction-filter-params.ts` - Complementary filter state management
- `use-sticky-columns.ts` - Works with sorting in sticky column scenarios

## Usage Examples

### Basic Sorting Implementation

```typescript
function SortableTableHeader() {
  const { params, setParams } = useSortParams();
  const [column, direction] = params.sort || [];

  const handleSort = (columnName: string) => {
    if (column === columnName) {
      // Cycle through states for same column
      if (direction === "asc") {
        setParams({ sort: [columnName, "desc"] });
      } else if (direction === "desc") {
        setParams({ sort: null }); // Clear sort
      }
    } else {
      // New column, start with ascending
      setParams({ sort: [columnName, "asc"] });
    }
  };

  return (
    <thead>
      <tr>
        <th onClick={() => handleSort("name")}>
          Name
          {column === "name" && (
            <span>{direction === "asc" ? "↑" : "↓"}</span>
          )}
        </th>
        <th onClick={() => handleSort("date")}>
          Date
          {column === "date" && (
            <span>{direction === "asc" ? "↑" : "↓"}</span>
          )}
        </th>
      </tr>
    </thead>
  );
}
```

### Advanced Sort Management

```typescript
function AdvancedSortControls() {
  const { params, setParams } = useSortParams();
  const [column, direction] = params.sort || [];

  const sortOptions = [
    { value: "date", label: "Date" },
    { value: "amount", label: "Amount" },
    { value: "name", label: "Description" },
  ];

  const clearSort = () => setParams({ sort: null });

  const applySortPreset = (preset: string) => {
    switch (preset) {
      case "recent":
        setParams({ sort: ["date", "desc"] });
        break;
      case "amount-high":
        setParams({ sort: ["amount", "desc"] });
        break;
      case "alphabetical":
        setParams({ sort: ["name", "asc"] });
        break;
    }
  };

  return (
    <div>
      <div>
        Current Sort: {column ? `${column} (${direction})` : "None"}
      </div>

      <div>
        <Button onClick={() => applySortPreset("recent")}>
          Most Recent
        </Button>
        <Button onClick={() => applySortPreset("amount-high")}>
          Highest Amount
        </Button>
        <Button onClick={() => applySortPreset("alphabetical")}>
          A-Z
        </Button>
      </div>

      {column && (
        <Button onClick={clearSort}>
          Clear Sort
        </Button>
      )}
    </div>
  );
}
```

### Sort State Debugging

```typescript
function SortDebugger() {
  const { params } = useSortParams();

  return (
    <div>
      <h3>Sort State Debug:</h3>
      <pre>{JSON.stringify(params, null, 2)}</pre>

      <p>
        Current URL: {typeof window !== 'undefined' ? window.location.search : 'SSR'}
      </p>

      <p>
        Sort Array: {params.sort ? params.sort.join(" → ") : "No sorting"}
      </p>
    </div>
  );
}
```
