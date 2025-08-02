# use-transaction-filter-params-with-persistence

> **Reference Implementation**: [apps/dashboard/src/hooks/use-transaction-filter-params-with-persistence.ts](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/use-transaction-filter-params-with-persistence.ts)

## Purpose

The `use-transaction-filter-params-with-persistence` hook extends the basic transaction filter functionality with localStorage persistence capabilities. It combines URL synchronization from `use-transaction-filter-params` with persistent storage, allowing users to maintain their filter preferences across browser sessions while providing enhanced filter management capabilities.

## Key Features

- Combines URL synchronization with localStorage persistence
- Automatic state hydration on page load
- Enhanced clear filters functionality
- Consistent API with other filter hooks
- Type-safe filter state management
- Cross-session filter preference retention

## Dependencies

### Internal Dependencies

- `use-transaction-filter-params` - Base URL synchronization functionality
- `use-generic-filter-persistence` - Generic persistence patterns
- `@/utils/transaction-filters` - Type definitions and utilities

### External Dependencies

- React hooks for state management
- Browser localStorage API (via generic persistence hook)

## Implementation Details

### Hook Composition Pattern

The hook follows a composition pattern, extending base functionality:

```typescript
export function useTransactionFilterParamsWithPersistence(): FilterHookReturn<TransactionFilters> {
  const { filter, setFilter, hasFilters } = useTransactionFilterParams();

  const { clearAllFilters } = useGenericFilterPersistence({
    storageKey: "transaction-filters",
    emptyState: EMPTY_FILTER_STATE,
    currentFilters: filter,
    setFilters: setFilter,
  });

  return {
    filter,
    setFilter,
    hasFilters,
    clearAllFilters,
  };
}
```

### Type System Integration

The hook uses comprehensive TypeScript types for filter state:

```typescript
type TransactionFilters = {
  q?: string | null;
  attachments?: "exclude" | "include" | null;
  start?: string | null;
  end?: string | null;
  categories?: string[] | null;
  tags?: string[] | null;
  accounts?: string[] | null;
  assignees?: string[] | null;
  amount_range?: number[] | null;
  amount?: string[] | null;
  recurring?: ("all" | "weekly" | "monthly" | "annually")[] | null;
  statuses?: ("completed" | "uncompleted" | "archived" | "excluded")[] | null;
};
```

### localStorage Integration

The hook integrates with localStorage through the generic persistence layer:

**Storage Key**: `"transaction-filters"`

- Unique identifier for transaction filter storage
- Prevents conflicts with other stored data
- Enables multiple filter types in the same application

**Empty State Management**:

```typescript
const EMPTY_FILTER_STATE: TransactionFilters = {
  q: null,
  attachments: null,
  start: null,
  end: null,
  categories: null,
  tags: null,
  accounts: null,
  assignees: null,
  amount_range: null,
  amount: null,
  recurring: null,
  statuses: null,
};
```

### State Hydration Patterns

The hook handles state hydration automatically:

1. **Initial Load**: Checks localStorage for saved filters
2. **URL Priority**: URL parameters take precedence over stored values
3. **Merge Strategy**: Combines URL and stored filters intelligently
4. **Fallback Handling**: Gracefully handles corrupted or invalid stored data

### Clear Filters Functionality

Enhanced clear functionality beyond basic URL clearing:

```typescript
const { clearAllFilters } = useGenericFilterPersistence({
  storageKey: "transaction-filters",
  emptyState: EMPTY_FILTER_STATE,
  currentFilters: filter,
  setFilters: setFilter,
});
```

The `clearAllFilters` function:

- Clears URL parameters
- Removes localStorage entries
- Resets state to empty values
- Triggers re-renders with clean state

### Empty State Management

The hook uses a predefined empty state for consistency:

- **Null Values**: All filter properties default to `null`
- **Type Safety**: Ensures proper TypeScript inference
- **Clean URLs**: Prevents empty parameters in URL
- **Storage Efficiency**: Minimizes localStorage usage

## Performance Considerations

- **Lazy Loading**: localStorage operations are performed only when needed
- **Debounced Persistence**: Prevents excessive storage writes during rapid filter changes
- **Memory Efficiency**: Only active filters are stored and synchronized
- **Hydration Optimization**: Minimizes client-server state mismatches

## Integration Patterns

### With Transaction Table Components

```typescript
// In data-table.tsx
function TransactionTable() {
  const { filter, setFilter, hasFilters, clearAllFilters } =
    useTransactionFilterParamsWithPersistence();

  // Use filters for data fetching
  const { data } = useTransactions({ filters: filter });

  return (
    <div>
      <DataTableHeader
        hasFilters={hasFilters}
        onClearFilters={clearAllFilters}
      />
      {/* Table content */}
    </div>
  );
}
```

### With Filter UI Components

```typescript
// In filter components
function TransactionFilters() {
  const { filter, setFilter, clearAllFilters } =
    useTransactionFilterParamsWithPersistence();

  return (
    <div>
      <SearchInput
        value={filter.q || ""}
        onChange={(q) => setFilter({ ...filter, q })}
      />

      <CategorySelect
        value={filter.categories || []}
        onChange={(categories) => setFilter({ ...filter, categories })}
      />

      <Button onClick={clearAllFilters}>
        Clear All Filters
      </Button>
    </div>
  );
}
```

### State Restoration Example

```typescript
// Automatic state restoration on page load
function TransactionPage() {
  const { filter, hasFilters } = useTransactionFilterParamsWithPersistence();

  // Filters are automatically restored from localStorage and URL
  // No additional setup required

  return (
    <div>
      {hasFilters && <FilterIndicator count={getActiveFilterCount(filter)} />}
      <TransactionTable />
    </div>
  );
}
```

## Error Handling

- **Storage Failures**: Graceful fallback when localStorage is unavailable
- **Data Corruption**: Handles invalid stored data by falling back to empty state
- **Type Validation**: Ensures stored data matches expected filter schema
- **Browser Compatibility**: Works across different browser localStorage implementations

## Related Files

- `use-transaction-filter-params.ts` - Base URL synchronization functionality
- `use-generic-filter-persistence.ts` - Generic persistence patterns
- `@/utils/transaction-filters.ts` - Type definitions and utilities
- `data-table.tsx` - Primary consumer of persistent filter state

## Usage Examples

### Basic Persistent Filtering

```typescript
function PersistentTransactionFilters() {
  const {
    filter,
    setFilter,
    hasFilters,
    clearAllFilters
  } = useTransactionFilterParamsWithPersistence();

  // Filters persist across browser sessions
  const handleCategoryChange = (categories: string[]) => {
    setFilter({ ...filter, categories });
    // Automatically saved to localStorage
  };

  return (
    <div>
      <CategoryFilter
        value={filter.categories || []}
        onChange={handleCategoryChange}
      />

      {hasFilters && (
        <div>
          <span>Active filters will be remembered</span>
          <Button onClick={clearAllFilters}>
            Clear & Forget All Filters
          </Button>
        </div>
      )}
    </div>
  );
}
```

### Advanced Filter Management

```typescript
function AdvancedFilterManager() {
  const { filter, setFilter, clearAllFilters } =
    useTransactionFilterParamsWithPersistence();

  const saveFilterPreset = (name: string) => {
    // Custom preset saving (beyond basic persistence)
    localStorage.setItem(`filter-preset-${name}`, JSON.stringify(filter));
  };

  const loadFilterPreset = (name: string) => {
    const preset = localStorage.getItem(`filter-preset-${name}`);
    if (preset) {
      setFilter(JSON.parse(preset));
    }
  };

  return (
    <div>
      <Button onClick={() => saveFilterPreset('monthly-expenses')}>
        Save Current Filters as Preset
      </Button>

      <Button onClick={() => loadFilterPreset('monthly-expenses')}>
        Load Monthly Expenses Preset
      </Button>

      <Button onClick={clearAllFilters}>
        Reset to Default
      </Button>
    </div>
  );
}
```

### Filter State Debugging

```typescript
function FilterDebugger() {
  const { filter, hasFilters } = useTransactionFilterParamsWithPersistence();

  return (
    <div>
      <h3>Current Filter State:</h3>
      <pre>{JSON.stringify(filter, null, 2)}</pre>

      <p>Has Active Filters: {hasFilters ? 'Yes' : 'No'}</p>

      <p>Stored Filters: {
        localStorage.getItem('transaction-filters') || 'None'
      }</p>
    </div>
  );
}
```
