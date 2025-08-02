# Dependency Mapping Documentation

> **Reference Implementations**:
> - [apps/dashboard/src/components/tables/transactions/](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/) - Transaction table components
> - [apps/dashboard/src/hooks/](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/) - Custom hooks
> - [apps/dashboard/src/store/transactions.ts](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/store/transactions.ts) - Zustand store
> - [apps/dashboard/src/utils/transaction-filters.ts](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/utils/transaction-filters.ts) - Filter utilities

## Overview
This document provides a comprehensive mapping of file relationships, dependencies, and integration points within the transaction table implementation. It serves as a guide for understanding how components, hooks, utilities, and external libraries work together.

## Visual Dependency Graph

```mermaid
graph TD
    %% Main Components
    DT[data-table.tsx] --> DTH[data-table-header.tsx]
    DT --> COL[columns.tsx]
    DT --> BB[bottom-bar.tsx]
    DT --> EB[export-bar.tsx]
    DT --> ES[empty-states.tsx]
    DT --> L[loading.tsx]
    
    %% Hook Dependencies
    DT --> UTFP[use-transaction-filter-params-with-persistence]
    DT --> USP[use-sort-params]
    DT --> USC[use-sticky-columns]
    DT --> UTS[use-table-scroll]
    DT --> UTP[use-transaction-params]
    
    %% Hook Chain Dependencies
    UTFP --> UGFP[use-generic-filter-persistence]
    UTFP --> UTF[use-transaction-filter-params]
    UGFP --> ULS[use-local-storage]
    
    %% Store Dependencies
    DT --> TS[transactions-store]
    EB --> TS
    BB --> TS
    
    %% Utility Dependencies
    UTFP --> TF[transaction-filters]
    UGFP --> TF
    DT --> CON[constants]
    
    %% External Libraries
    DT --> RT[@tanstack/react-table]
    DT --> FM[framer-motion]
    DT --> RHH[react-hotkeys-hook]
    UTF --> NUQS[nuqs]
    UTS --> RHH
    
    %% UI Components
    COL --> UI[@midday/ui/*]
    DTH --> UI
    BB --> UI
    EB --> UI
    
    %% API/TRPC
    DT --> TRPC[@api/trpc]
    
    classDef component fill:#e1f5fe
    classDef hook fill:#f3e5f5
    classDef store fill:#fff3e0
    classDef utility fill:#e8f5e8
    classDef external fill:#ffebee
    
    class DT,DTH,COL,BB,EB,ES,L component
    class UTFP,USP,USC,UTS,UTP,UGFP,UTF,ULS hook
    class TS store
    class TF,CON utility
    class RT,FM,RHH,NUQS,UI,TRPC external
```

## Component Dependencies

### data-table.tsx (Main Orchestrator)
**Role**: Central component that coordinates all table functionality

**Direct Dependencies**:
- **Components**: `DataTableHeader`, `columns`, `BottomBar`, `ExportBar`, `NoResults`, `NoTransactions`, `Loading`
- **Hooks**: `useTransactionFilterParamsWithPersistence`, `useSortParams`, `useStickyColumns`, `useTableScroll`, `useTransactionParams`
- **Store**: `useTransactionsStore`
- **External**: `@tanstack/react-table`, `framer-motion`, `react-hotkeys-hook`, `@tanstack/react-query`

**Integration Points**:
```typescript
// Hook integrations
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

### columns.tsx (Column Definitions)
**Role**: Defines table structure and cell rendering logic

**Direct Dependencies**:
- **Components**: `AssignedUser`, `Category`, `FormatAmount`, `TransactionBankAccount`, `TransactionMethod`, `TransactionStatus`
- **UI**: `@midday/ui` components (Badge, Button, Checkbox, DropdownMenu, etc.)
- **External**: `@tanstack/react-table` (ColumnDef type)

**Performance Patterns**:
```typescript
// Memoized cell components
const SelectCell = memo(({ checked, onChange }) => (/* ... */));
const DateCell = memo(({ date }) => (/* ... */));
const DescriptionCell = memo(({ description, name }) => (/* ... */));
```

### data-table-header.tsx (Header with Sorting)
**Role**: Provides column headers with sorting and selection functionality

**Direct Dependencies**:
- **Hooks**: `useSortParams`, `useStickyColumns` (indirectly through styling)
- **UI**: `@midday/ui` components
- **External**: `@tanstack/react-table`

**Integration Pattern**:
```typescript
const { setSortBy } = useSortParams();
// Receives sticky positioning from parent
```

## Hook Dependency Chain

### Primary Filter Chain
```
useTransactionFilterParamsWithPersistence
├── useTransactionFilterParams (URL state)
└── useGenericFilterPersistence (localStorage)
    └── useFilterLocalStorage
        └── useLocalStorage (base functionality)
```

**Data Flow**:
1. **URL Parameters** → `useTransactionFilterParams` (nuqs integration)
2. **Filter State** → `useGenericFilterPersistence` (persistence logic)
3. **Storage Operations** → `useFilterLocalStorage` (debounced saving)
4. **Base Storage** → `useLocalStorage` (core localStorage wrapper)

### Table Enhancement Chain
```
data-table.tsx
├── useStickyColumns (positioning)
├── useTableScroll (navigation)
├── useSortParams (sorting state)
└── useTransactionParams (URL params)
```

## Store Integration

### Transactions Store (Zustand)
**File**: `apps/dashboard/src/store/transactions.ts`

**State Management**:
```typescript
interface TransactionsState {
  canDelete?: boolean;           // Bulk actions permission
  columns: Column<any, unknown>[];  // Table column definitions
  rowSelection: Record<string, boolean>;  // Selected rows
  setColumns: (columns?) => void;
  setCanDelete: (canDelete?) => void;
  setRowSelection: (updater) => void;
}
```

**Usage Pattern**:
- **data-table.tsx**: Manages row selection and column state
- **export-bar.tsx**: Reads selection state for export functionality
- **bottom-bar.tsx**: Reads selection state for summary display

## Utility Dependencies

### transaction-filters.ts
**Role**: Provides type definitions and filter utilities

**Key Exports**:
```typescript
export type TransactionFilters = {
  q?: string | null;
  attachments?: "exclude" | "include" | null;
  start?: string | null;
  end?: string | null;
  categories?: string[] | null;
  // ... other filter properties
};

export const EMPTY_FILTER_STATE: TransactionFilters;
export function cleanFilters(filters: any): any;
export function hasActiveUrlFilters(filters: any): boolean;
```

**Usage**:
- **Filter hooks**: Type definitions and state management
- **Persistence hooks**: Empty state and filter cleaning

### constants.ts
**Role**: Application-wide constants

**Key Usage**:
```typescript
import { Cookies } from "@/utils/constants";
// Used for column visibility persistence
updateColumnVisibilityAction({
  key: Cookies.TransactionsColumns,
  data: columnVisibility,
});
```

## External Library Integration

### @tanstack/react-table
**Usage**: Core table functionality
**Integration Points**:
- **data-table.tsx**: Main table instance creation
- **columns.tsx**: Column definitions and cell rendering
- **data-table-header.tsx**: Header rendering and sorting

**Key Patterns**:
```typescript
const table = useReactTable({
  data: tableData,
  columns,
  getCoreRowModel: getCoreRowModel(),
  onRowSelectionChange: setRowSelection,
  onColumnVisibilityChange: setColumnVisibility,
});
```

### nuqs (URL State Management)
**Usage**: URL parameter synchronization
**Integration Points**:
- **use-transaction-filter-params.ts**: Filter state in URL
- **use-sort-params.ts**: Sort state in URL
- **use-transaction-params.ts**: Transaction-specific params

**Key Features**:
- Server-side parsing support
- Type-safe parameter definitions
- Automatic URL synchronization

### framer-motion
**Usage**: Animation and transitions
**Integration Points**:
- **bottom-bar.tsx**: Slide-up animation
- **export-bar.tsx**: Slide-up animation
- **data-table.tsx**: AnimatePresence for conditional rendering

### react-hotkeys-hook
**Usage**: Keyboard navigation
**Integration Points**:
- **use-table-scroll.ts**: Arrow key navigation
- **data-table.tsx**: Table-specific hotkeys

## Performance Optimization Dependencies

### React Performance Hooks
```typescript
// data-table.tsx
const deferredSearch = useDeferredValue(filter.q);  // Deferred search input
const tableData = useMemo(() => data?.pages.flatMap((page) => page.data) ?? [], [data]);

// columns.tsx
const SelectCell = memo(({ checked, onChange }) => (/* ... */));  // Memoized cells

// bottom-bar.tsx
const totalAmount = useMemo(() => {  // Memoized calculations
  return transactions.reduce((sum, transaction) => sum + transaction.amount, 0);
}, [transactions]);
```

### Optimization Strategies
1. **Memoized Cell Components**: Prevent unnecessary re-renders of individual cells
2. **Deferred Values**: Debounce expensive search operations
3. **Memoized Calculations**: Cache computed values like totals
4. **Callback Memoization**: Stable function references for event handlers

## API Integration

### tRPC Integration
**Pattern**: Type-safe API calls with React Query
```typescript
const { data, fetchNextPage, hasNextPage, refetch } = useSuspenseInfiniteQuery(
  trpc.transactions.get.infiniteQueryOptions({
    ...filter,
    q: deferredSearch,
    sort: params.sort,
  })
);
```

## File Relationship Summary

### Core Components (7 files)
1. **data-table.tsx** ← Main orchestrator
2. **columns.tsx** ← Column definitions
3. **data-table-header.tsx** ← Header with sorting
4. **bottom-bar.tsx** ← Filter summary bar
5. **export-bar.tsx** ← Bulk actions bar
6. **empty-states.tsx** ← No data states
7. **loading.tsx** ← Loading states

### Supporting Hooks (7 files)
1. **use-transaction-filter-params-with-persistence.ts**
2. **use-transaction-filter-params.ts**
3. **use-generic-filter-persistence.ts**
4. **use-local-storage.ts**
5. **use-sort-params.ts**
6. **use-sticky-columns.ts**
7. **use-table-scroll.ts**

### State Management (1 file)
1. **transactions.ts** (Zustand store)

### Utilities (2 files)
1. **transaction-filters.ts**
2. **constants.ts**

## Integration Testing Points

When testing the transaction table system, focus on these integration points:

1. **Filter Persistence**: URL ↔ localStorage synchronization
2. **Table State**: Row selection ↔ Store synchronization
3. **Sorting State**: URL ↔ Table header synchronization
4. **Scroll Coordination**: Table scroll ↔ Sticky columns
5. **Performance**: Memoization effectiveness under load
6. **Keyboard Navigation**: Hotkeys ↔ Table actions

## Troubleshooting Common Issues

### Filter State Conflicts
- **Issue**: URL filters not loading from localStorage
- **Check**: `hasActiveUrlFilters` logic in persistence hook

### Performance Issues
- **Issue**: Excessive re-renders during typing
- **Check**: `useDeferredValue` implementation and cell memoization

### Scroll Coordination
- **Issue**: Sticky columns misaligned during scroll
- **Check**: ResizeObserver and CSS custom property updates

### State Synchronization
- **Issue**: Row selection not persisting across navigation
- **Check**: Zustand store integration and cleanup