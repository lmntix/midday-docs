# data-table.tsx

> **Reference Implementation**: [apps/dashboard/src/components/tables/transactions/data-table.tsx](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/data-table.tsx)

## Purpose

The `data-table.tsx` file serves as the main orchestrator component for the transactions table implementation. It coordinates all table functionality including data fetching, state management, row selection, infinite scrolling, keyboard navigation, and integration with various UI components and hooks. This component acts as the central hub that brings together all the different pieces of the transactions table system.

## Key Features

- **Infinite Scrolling**: Implements server-side pagination with automatic loading of additional data as user scrolls
- **Row Selection**: Manages multi-row selection state with bulk operations support
- **Keyboard Navigation**: Provides arrow key navigation between transaction rows
- **Real-time Updates**: Handles transaction updates and deletions with optimistic UI updates
- **Responsive Design**: Adapts to different screen sizes with appropriate styling
- **Performance Optimization**: Uses memoization, deferred values, and efficient re-rendering strategies
- **State Persistence**: Integrates with URL parameters and localStorage for filter persistence
- **Column Visibility**: Manages dynamic column show/hide functionality
- **Sticky Columns**: Implements sticky positioning for key columns during horizontal scrolling

## Dependencies

### Internal Dependencies

- `useTransactionFilterParamsWithPersistence`: Manages filter state with URL and localStorage persistence
- `useSortParams`: Handles sorting parameters and URL synchronization
- `useStickyColumns`: Provides sticky column positioning logic and CSS utilities
- `useTableScroll`: Manages horizontal scrolling behavior with column-width-based navigation
- `useTransactionParams`: Manages transaction-specific URL parameters (like selected transaction ID)
- `useTransactionsStore`: Zustand store for global transaction table state management
- `columns`: Column definitions and cell renderers
- `DataTableHeader`: Table header component with sorting and selection controls
- `BottomBar`: Filter summary and totals display component
- `ExportBar`: Bulk actions and export functionality component
- `NoResults`, `NoTransactions`: Empty state components
- `Loading`: Loading skeleton component

### External Dependencies

- `@tanstack/react-table`: Core table functionality and state management
- `@tanstack/react-query`: Data fetching, caching, and infinite query management
- `react-hotkeys-hook`: Keyboard shortcut handling
- `react-intersection-observer`: Infinite scroll trigger detection
- `framer-motion`: Animation support for bottom bar
- `@midday/ui`: UI component library (Table, Tooltip, Toast)

## Implementation Details

### Key Code Patterns

#### Main Component Structure

```typescript
export function DataTable({
  columnVisibility: columnVisibilityPromise,
}: Props) {
  // Hook integrations
  const { filter, hasFilters } = useTransactionFilterParamsWithPersistence();
  const { setRowSelection, rowSelection, setColumns, setCanDelete } = useTransactionsStore();
  const { params } = useSortParams();
  const { transactionId, setParams } = useTransactionParams();

  // Performance optimizations
  const deferredSearch = useDeferredValue(filter.q);

  // Infinite scrolling setup
  const { data, fetchNextPage, hasNextPage, refetch } = useSuspenseInfiniteQuery(infiniteQueryOptions);
```

#### Hook Integrations

**Filter Management with Persistence**:

```typescript
const { filter, hasFilters } = useTransactionFilterParamsWithPersistence();
const deferredSearch = useDeferredValue(filter.q);
```

- Integrates with URL parameters and localStorage for filter persistence
- Uses `useDeferredValue` to debounce search input for performance
- `hasFilters` determines when to show the bottom bar with filter summaries

**Sorting Integration**:

```typescript
const { params } = useSortParams();
// Used in query options
const infiniteQueryOptions = trpc.transactions.get.infiniteQueryOptions({
  ...filter,
  q: deferredSearch,
  sort: params.sort,
});
```

- Seamlessly integrates sorting parameters into data fetching
- Maintains sort state in URL for shareability and persistence

**Sticky Columns Integration**:

```typescript
const { getStickyStyle, getStickyClassName } = useStickyColumns({
  columnVisibility,
  table,
});
```

- Provides dynamic CSS styles and classes for sticky column positioning
- Adapts to column visibility changes automatically

**Table Scrolling Integration**:

```typescript
const tableScroll = useTableScroll({
  useColumnWidths: true,
  startFromColumn: 3, // Skip sticky columns: select, date, description
});
```

- Enables keyboard-based horizontal scrolling
- Configured to start scrolling after sticky columns

### Infinite Scrolling Implementation

#### Query Setup with useSuspenseInfiniteQuery

```typescript
const infiniteQueryOptions = trpc.transactions.get.infiniteQueryOptions(
  {
    ...filter,
    q: deferredSearch,
    sort: params.sort,
  },
  {
    getNextPageParam: ({ meta }) => meta?.cursor,
  }
);

const { data, fetchNextPage, hasNextPage, refetch } =
  useSuspenseInfiniteQuery(infiniteQueryOptions);
```

**Key Implementation Details**:

- Uses cursor-based pagination with `getNextPageParam`
- Integrates all filter and sort parameters into the query
- `useSuspenseInfiniteQuery` ensures data is available before rendering
- Provides `fetchNextPage` for loading additional data

#### Intersection Observer Integration

```typescript
const { ref, inView } = useInView();

useEffect(() => {
  if (inView) {
    fetchNextPage();
  }
}, [inView]);

// In JSX
<LoadMore ref={ref} hasNextPage={hasNextPage} />
```

**Automatic Loading Trigger**:

- Uses `react-intersection-observer` to detect when load more element is visible
- Automatically triggers `fetchNextPage` when user scrolls to bottom
- `LoadMore` component serves as the intersection trigger point

### Row Selection Logic and State Management

#### Selection State Management

```typescript
const { setRowSelection, rowSelection, setColumns, setCanDelete } =
  useTransactionsStore();

const table = useReactTable({
  // ... other config
  onRowSelectionChange: setRowSelection,
  state: {
    rowSelection,
    columnVisibility,
  },
});
```

**Global State Integration**:

- Uses Zustand store for global selection state management
- Enables selection state to persist across component re-renders
- Integrates with React Table's built-in selection handling

#### Delete Permission Logic

```typescript
useEffect(() => {
  const transactions = tableData.filter((transaction) => {
    if (!transaction?.id) return false;
    const found = rowSelection[transaction.id];

    if (found) {
      return !transaction?.manual;
    }
    return false;
  });

  if (Object.keys(rowSelection)?.length > 0) {
    if (transactions.length === 0) {
      setCanDelete(true);
    } else {
      setCanDelete(false);
    }
  }
}, [rowSelection]);
```

**Business Logic Implementation**:

- Determines if selected transactions can be deleted
- Manual transactions cannot be deleted (business rule)
- Updates global state to enable/disable bulk delete actions

### Performance Optimization Techniques

#### Memoization Strategies

```typescript
const tableData = useMemo(() => {
  return data?.pages.flatMap((page) => page.data) ?? [];
}, [data]);

const ids = useMemo(() => {
  return tableData.map((row) => row?.id);
}, [tableData]);
```

**Data Processing Optimization**:

- `tableData` memoization prevents unnecessary flattening of paginated data
- `ids` memoization optimizes keyboard navigation lookup performance
- Dependencies carefully chosen to minimize re-computation

#### Deferred Values for Search

```typescript
const deferredSearch = useDeferredValue(filter.q);
```

**Search Performance**:

- Defers search query execution during rapid typing
- Reduces API calls and improves user experience
- Maintains responsive UI during search input

#### Column Visibility Effects

```typescript
useEffect(() => {
  setColumns(table.getAllLeafColumns());
}, [columnVisibility]);

useEffect(() => {
  updateColumnVisibilityAction({
    key: Cookies.TransactionsColumns,
    data: columnVisibility,
  });
}, [columnVisibility]);
```

**State Synchronization**:

- Updates global store when column visibility changes
- Persists column preferences to cookies via server action
- Minimizes re-renders by using targeted effects

### Keyboard Navigation Implementation

```typescript
useHotkeys(
  "ArrowUp, ArrowDown",
  ({ key }) => {
    if (key === "ArrowUp" && transactionId) {
      const currentIndex = ids?.indexOf(transactionId) ?? 0;
      const prevId = ids[currentIndex - 1];

      if (prevId) {
        setParams({ transactionId: prevId });
      }
    }

    if (key === "ArrowDown" && transactionId) {
      const currentIndex = ids?.indexOf(transactionId) ?? 0;
      const nextId = ids[currentIndex + 1];

      if (nextId) {
        setParams({ transactionId: nextId });
      }
    }
  },
  { enabled: !!transactionId }
);
```

**Navigation Logic**:

- Enables arrow key navigation between transactions
- Only active when a transaction is selected (`enabled: !!transactionId`)
- Updates URL parameters to maintain navigation state
- Uses memoized `ids` array for efficient index lookup

### Mutation Handling and Optimistic Updates

#### Transaction Update Mutation

```typescript
const updateTransactionMutation = useMutation(
  trpc.transactions.update.mutationOptions({
    onSuccess: () => {
      refetch();
      toast({
        title: "Transaction updated",
        variant: "success",
      });
    },
  })
);
```

#### Delete Mutation

```typescript
const deleteTransactionMutation = useMutation(
  trpc.transactions.deleteMany.mutationOptions({
    onSuccess: () => {
      refetch();
    },
  })
);
```

**Mutation Integration**:

- Provides mutation functions to table meta for use in cell components
- Handles success states with data refetching and user feedback
- Integrates with React Query's caching and invalidation system

### Table Meta Configuration

```typescript
meta: {
  setOpen: (id: string) => {
    setParams({ transactionId: id });
  },
  copyUrl: (id: string) => {
    // URL copying logic with clipboard API
  },
  updateTransaction: (data: { id: string; status: string }) => {
    updateTransactionMutation.mutate({
      id: data.id,
      status: data.status as "pending" | "archived" | "completed" | "posted" | "excluded",
    });
  },
  onDeleteTransaction: (id: string) => {
    deleteTransactionMutation.mutate([id]);
  },
},
```

**Component Communication**:

- Provides callback functions to child components via table meta
- Enables cell components to trigger parent-level actions
- Maintains separation of concerns while enabling deep integration

## State Management

### Local State

- `columnVisibility`: Controls which columns are visible/hidden
- Component manages its own column visibility state with persistence

### Global State Integration

- `useTransactionsStore`: Row selection, column references, delete permissions
- `useTransactionParams`: Selected transaction ID and URL management
- `useTransactionFilterParamsWithPersistence`: Filter state with persistence

### URL State Synchronization

- Transaction selection state persisted in URL parameters
- Filter and sort parameters maintained in URL for shareability
- Column visibility persisted in cookies for user preferences

## Performance Considerations

### Rendering Optimizations

- **Memoized Data Processing**: Table data and ID arrays are memoized to prevent unnecessary recalculations
- **Deferred Search**: Search queries are deferred to reduce API calls during typing
- **Selective Re-rendering**: Effects are scoped to specific dependencies to minimize re-renders

### Data Fetching Optimizations

- **Infinite Query Caching**: React Query handles caching and background updates
- **Cursor-based Pagination**: Efficient server-side pagination with cursor-based approach
- **Suspense Integration**: Uses Suspense for better loading state management

### Memory Management

- **Efficient Selection State**: Row selection uses object keys for O(1) lookup performance
- **Memoized Computations**: Expensive calculations are memoized with appropriate dependencies
- **Cleanup Effects**: Proper cleanup of event listeners and subscriptions

## Integration Patterns

### Hook Composition Pattern

The component demonstrates effective hook composition by integrating multiple specialized hooks:

```typescript
// Each hook handles a specific concern
const { filter, hasFilters } = useTransactionFilterParamsWithPersistence();
const { params } = useSortParams();
const { getStickyStyle, getStickyClassName } = useStickyColumns({
  columnVisibility,
  table,
});
const tableScroll = useTableScroll({
  useColumnWidths: true,
  startFromColumn: 3,
});
```

### Component Communication Pattern

Uses React Table's meta system for child component communication:

```typescript
// Parent provides callbacks via meta
meta: {
  setOpen: (id: string) => setParams({ transactionId: id }),
  updateTransaction: (data) => updateTransactionMutation.mutate(data),
  // ... other callbacks
}

// Child components access via cell.getContext().table.options.meta
```

### Conditional Rendering Pattern

```typescript
// Smart empty state handling
if (!tableData.length && !hasFilters) {
  return <NoTransactions />;
}

if (!tableData.length && hasFilters) {
  return <NoResults />;
}

// Conditional bottom bar display
<AnimatePresence>{showBottomBar && <BottomBar />}</AnimatePresence>
```

## Related Files

### Direct Dependencies

- `./columns.tsx` - Column definitions and cell renderers
- `./data-table-header.tsx` - Table header with sorting and selection
- `./bottom-bar.tsx` - Filter summary and totals display
- `./export-bar.tsx` - Bulk actions and export functionality
- `./empty-states.tsx` - NoResults and NoTransactions components
- `./loading.tsx` - Loading skeleton component

### Hook Dependencies

- `@/hooks/use-transaction-filter-params-with-persistence` - Filter state management
- `@/hooks/use-sort-params` - Sorting parameter management
- `@/hooks/use-sticky-columns` - Sticky column positioning
- `@/hooks/use-table-scroll` - Horizontal scrolling behavior
- `@/hooks/use-transaction-params` - Transaction URL parameter management

### Store Dependencies

- `@/store/transactions` - Global transaction table state management

### Action Dependencies

- `@/actions/update-column-visibility-action` - Server action for persisting column preferences
