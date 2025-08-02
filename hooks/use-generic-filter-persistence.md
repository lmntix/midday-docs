# use-generic-filter-persistence Hook Documentation

> **Reference Implementation**: [apps/dashboard/src/hooks/use-generic-filter-persistence.ts](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/use-generic-filter-persistence.ts)

## Overview
The `useGenericFilterPersistence` hook provides a reusable solution for persisting any type of filters to localStorage. It's designed as a generic abstraction that can be used across different entities (transactions, invoices, customers, etc.) while maintaining consistency in filter persistence behavior.

## Key Features
- **Generic implementation**: Works with any filter object type
- **localStorage integration**: Automatic saving and loading of filter state
- **URL awareness**: Only loads from localStorage when URL is empty
- **Filter cleaning**: Automatically removes empty/default values before saving
- **Reusable across entities**: Can be used for transactions, invoices, customers, etc.

## Interface

```typescript
type UseFilterPersistenceOptions<T> = {
  storageKey: string;           // Unique key for localStorage
  emptyState: T;               // Default/empty filter state
  currentFilters: T;           // Current filter values
  setFilters: (filters: T) => void;  // Function to update filters
};
```

## Generic Type Parameter
```typescript
T extends Record<string, any>
```
- Accepts any object-based filter structure
- Ensures type safety across different filter types
- Maintains intellisense and type checking

## Return Values

```typescript
{
  clearAllFilters: () => void;  // Function to reset all filters to empty state
}
```

## Core Dependencies

### useFilterLocalStorage Integration
```typescript
const { initializeFromStorage, saveToStorage } = useFilterLocalStorage<T>({
  key: storageKey,
  serialize: (filters) => JSON.stringify(cleanFilters(filters)),
  shouldSave: (filters) => Object.keys(cleanFilters(filters)).length > 0,
});
```
- **Custom serialization**: Uses `cleanFilters` to remove empty values
- **Conditional saving**: Only saves when filters have content
- **Type-safe operations**: Maintains generic type throughout

### Filter Cleaning Utility
```typescript
serialize: (filters) => JSON.stringify(cleanFilters(filters))
```
- Removes empty, null, undefined, and default values
- Prevents saving unnecessary filter state
- Reduces localStorage usage

## Lifecycle Management

### Initialization from localStorage
```typescript
useEffect(() => {
  initializeFromStorage(
    (savedFilters: T) => setFilters(savedFilters),
    () => !hasActiveUrlFilters(currentFilters),
  );
}, [initializeFromStorage, setFilters, currentFilters]);
```
- **URL priority**: Only loads from localStorage if URL has no active filters
- **Conditional loading**: Respects URL-based filter state
- **One-time initialization**: Prevents repeated loading

### Automatic Saving
```typescript
useEffect(() => {
  saveToStorage(currentFilters);
}, [currentFilters, saveToStorage]);
```
- **Reactive saving**: Automatically saves when filters change
- **Debounced writes**: Uses debounced saving from useFilterLocalStorage
- **Conditional persistence**: Only saves non-empty filter states

## URL and localStorage Coordination

### Priority System
1. **URL filters take precedence** over localStorage
2. **localStorage only loads** when URL is empty
3. **Changes save to localStorage** regardless of source

### hasActiveUrlFilters Integration
```typescript
() => !hasActiveUrlFilters(currentFilters)
```
- Checks if current filters came from URL parameters
- Prevents overriding URL-based filter state
- Maintains proper state hierarchy

## Clear Filters Functionality

### clearAllFilters Implementation
```typescript
const clearAllFilters = useCallback(() => {
  setFilters(emptyState);
}, [setFilters, emptyState]);
```
- **Resets to empty state**: Uses provided emptyState object
- **Triggers persistence**: Will save empty state (or remove from localStorage)
- **Type-safe reset**: Ensures proper type for empty state

## Reusability Examples

### Transaction Filters
```typescript
export function useTransactionFilterPersistence() {
  const { filter, setFilter, hasFilters } = useTransactionFilterParams();
  const { clearAllFilters } = useGenericFilterPersistence({
    storageKey: "transaction-filters",
    emptyState: EMPTY_TRANSACTION_FILTER_STATE,
    currentFilters: filter,
    setFilters: setFilter,
  });
  return { filter, setFilter, hasFilters, clearAllFilters };
}
```

### Invoice Filters (Example)
```typescript
export function useInvoiceFilterPersistence() {
  const { filter, setFilter, hasFilters } = useInvoiceFilterParams();
  const { clearAllFilters } = useGenericFilterPersistence({
    storageKey: "invoice-filters",
    emptyState: EMPTY_INVOICE_FILTER_STATE,
    currentFilters: filter,
    setFilters: setFilter,
  });
  return { filter, setFilter, hasFilters, clearAllFilters };
}
```

### Customer Filters (Example)
```typescript
export function useCustomerFilterPersistence() {
  const { filter, setFilter, hasFilters } = useCustomerFilterParams();
  const { clearAllFilters } = useGenericFilterPersistence({
    storageKey: "customer-filters",
    emptyState: EMPTY_CUSTOMER_FILTER_STATE,
    currentFilters: filter,
    setFilters: setFilter,
  });
  return { filter, setFilter, hasFilters, clearAllFilters };
}
```

## Storage Key Conventions
- **Format**: `{entity}-filters` (e.g., "transaction-filters")
- **Uniqueness**: Each entity uses a unique storage key
- **Consistency**: Follows predictable naming pattern

## Filter State Requirements

### Empty State Object
```typescript
const EMPTY_FILTER_STATE = {
  search: "",
  categories: [],
  accounts: [],
  dateRange: null,
  // ... other filter properties
};
```
- Must match the structure of current filters
- Provides default values for all filter properties
- Used for reset functionality

### Current Filter Object
- Should come from URL parameter hooks (e.g., useTransactionFilterParams)
- Must be reactive to URL changes
- Should trigger re-renders when changed

## Performance Considerations

### Debounced Saving
- Inherits debounced saving from useFilterLocalStorage
- Prevents excessive localStorage writes
- Default 500ms debounce delay

### Memoized Callbacks
```typescript
const clearAllFilters = useCallback(() => {
  setFilters(emptyState);
}, [setFilters, emptyState]);
```
- Prevents unnecessary re-renders
- Stable function reference
- Optimized for component performance

## Error Handling
- Delegates error handling to useFilterLocalStorage
- Graceful fallback when localStorage is unavailable
- Console logging for debugging

## Usage Pattern

```typescript
function FilterComponent() {
  const { 
    filter, 
    setFilter, 
    hasFilters, 
    clearAllFilters 
  } = useTransactionFilterPersistence();

  return (
    <div>
      {/* Filter controls */}
      <FilterControls filter={filter} onChange={setFilter} />
      
      {/* Clear button */}
      {hasFilters && (
        <button onClick={clearAllFilters}>
          Clear All Filters
        </button>
      )}
    </div>
  );
}
```

## Integration Dependencies
- **useFilterLocalStorage**: Core localStorage functionality
- **cleanFilters**: Utility for removing empty values
- **hasActiveUrlFilters**: URL state detection
- **URL parameter hooks**: Entity-specific filter parameter management