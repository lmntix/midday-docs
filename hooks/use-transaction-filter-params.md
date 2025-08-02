# use-transaction-filter-params

> **Reference Implementation**: [apps/dashboard/src/hooks/use-transaction-filter-params.ts](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/use-transaction-filter-params.ts)

## Purpose

The `use-transaction-filter-params` hook provides URL-synchronized filter state management for the transactions table. It leverages the `nuqs` library to maintain filter parameters in the URL query string, enabling shareable and bookmarkable filtered views while providing a clean API for filter state management.

## Key Features

- URL synchronization with automatic serialization/deserialization
- Type-safe filter schema with comprehensive validation
- Automatic URL cleanup when filters are cleared
- Server-side rendering support with loader function
- Comprehensive filter state detection

## Dependencies

### External Dependencies

- `nuqs` - URL state management library
- `nuqs/server` - Server-side utilities for SSR support

### Internal Dependencies

- Used by transaction table components for filter state management
- Integrated with `use-transaction-filter-params-with-persistence` for localStorage persistence

## Implementation Details

### Filter Schema Definition

The hook defines a comprehensive filter schema using `nuqs` parsers:

```typescript
export const transactionFilterParamsSchema = {
  q: parseAsString, // Search query
  attachments: parseAsStringLiteral(["exclude", "include"] as const),
  start: parseAsString, // Date range start
  end: parseAsString, // Date range end
  categories: parseAsArrayOf(parseAsString),
  tags: parseAsArrayOf(parseAsString),
  accounts: parseAsArrayOf(parseAsString),
  assignees: parseAsArrayOf(parseAsString),
  amount_range: parseAsArrayOf(parseAsInteger),
  amount: parseAsArrayOf(parseAsString),
  recurring: parseAsArrayOf(
    parseAsStringLiteral(["all", "weekly", "monthly", "annually"] as const)
  ),
  statuses: parseAsArrayOf(
    parseAsStringLiteral([
      "completed",
      "uncompleted",
      "archived",
      "excluded",
    ] as const)
  ),
};
```

### URL Synchronization Logic

The hook uses `useQueryStates` with automatic URL synchronization:

```typescript
const [filter, setFilter] = useQueryStates(transactionFilterParamsSchema, {
  clearOnDefault: true, // Removes parameters from URL when set to null/default
});
```

### State Management Patterns

**Filter State Access**:

```typescript
const { filter, setFilter, hasFilters } = useTransactionFilterParams();

// Access individual filter values
const searchQuery = filter.q;
const selectedCategories = filter.categories;
const dateRange = { start: filter.start, end: filter.end };
```

**Filter Updates**:

```typescript
// Update single filter
setFilter({ q: "expense" });

// Update multiple filters
setFilter({
  categories: ["office", "travel"],
  start: "2024-01-01",
  end: "2024-01-31",
});

// Clear specific filter
setFilter({ categories: null });
```

### clearOnDefault Behavior

The `clearOnDefault: true` option ensures clean URLs by:

- Removing query parameters when values are set to `null`
- Preventing empty arrays from appearing in the URL
- Maintaining clean, shareable URLs without redundant parameters

### hasFilters Calculation

The hook provides a `hasFilters` boolean that detects active filters:

```typescript
hasFilters: Object.values(filter).some((value) => value !== null);
```

This calculation:

- Checks all filter values for non-null states
- Returns `true` if any filter is actively applied
- Used by UI components to show/hide filter indicators
- Enables conditional rendering of filter-related UI elements

## Performance Considerations

- **URL Synchronization**: Changes are debounced by `nuqs` to prevent excessive URL updates
- **Type Safety**: Schema validation prevents runtime errors from malformed URL parameters
- **Memory Efficiency**: Only active filters are maintained in state
- **SSR Optimization**: Server-side loader enables proper hydration without client-server mismatches

## Integration Patterns

### With Transaction Table Components

```typescript
// In data-table.tsx
const { filter, setFilter, hasFilters } = useTransactionFilterParams();

// Pass filters to data fetching
const { data } = useTransactions({
  filters: filter,
  // other params
});

// Show filter indicators
{hasFilters && <FilterSummary filters={filter} />}
```

### With Filter UI Components

```typescript
// In filter components
const { filter, setFilter } = useTransactionFilterParams();

// Category filter
<CategorySelect
  value={filter.categories || []}
  onChange={(categories) => setFilter({ categories })}
/>

// Search input
<SearchInput
  value={filter.q || ""}
  onChange={(q) => setFilter({ q })}
/>
```

### Server-Side Rendering

```typescript
// In page component
export async function generateMetadata({ searchParams }) {
  const filters = loadTransactionFilterParams(searchParams);
  // Use filters for metadata generation
}
```

## Error Handling

- **Invalid URL Parameters**: `nuqs` parsers handle malformed parameters gracefully
- **Type Validation**: Schema ensures only valid values are accepted
- **Fallback Values**: Invalid parameters default to `null` rather than causing errors
- **Browser Compatibility**: Graceful degradation for browsers without full URL API support

## Related Files

- `use-transaction-filter-params-with-persistence.ts` - Adds localStorage persistence
- `use-generic-filter-persistence.ts` - Generic persistence patterns
- `data-table.tsx` - Primary consumer of filter state
- `transactions-search-filter.tsx` - UI components for filter management

## Usage Examples

### Basic Filter Management

```typescript
function TransactionFilters() {
  const { filter, setFilter, hasFilters } = useTransactionFilterParams();

  return (
    <div>
      <SearchInput
        value={filter.q || ""}
        onChange={(q) => setFilter({ q })}
      />

      <CategoryFilter
        selected={filter.categories || []}
        onChange={(categories) => setFilter({ categories })}
      />

      {hasFilters && (
        <Button onClick={() => setFilter({})}>
          Clear All Filters
        </Button>
      )}
    </div>
  );
}
```

### Advanced Filter Combinations

```typescript
function AdvancedFilters() {
  const { filter, setFilter } = useTransactionFilterParams();

  const applyQuickFilter = (type: string) => {
    switch (type) {
      case 'recent':
        setFilter({
          start: format(subDays(new Date(), 30), 'yyyy-MM-dd'),
          end: format(new Date(), 'yyyy-MM-dd')
        });
        break;
      case 'expenses':
        setFilter({
          amount: ['negative'],
          statuses: ['completed']
        });
        break;
    }
  };

  return (
    <div>
      <Button onClick={() => applyQuickFilter('recent')}>
        Last 30 Days
      </Button>
      <Button onClick={() => applyQuickFilter('expenses')}>
        Expenses Only
      </Button>
    </div>
  );
}
```
