# Naming Conventions Documentation

> **Reference Implementations**:
> - [apps/dashboard/src/components/tables/transactions/](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/) - Component naming patterns
> - [apps/dashboard/src/hooks/](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/) - Hook naming conventions
> - [apps/dashboard/src/store/](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/store/) - Store organization
> - [apps/dashboard/src/utils/](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/utils/) - Utility naming patterns

## Overview
This document outlines the comprehensive naming conventions and organizational patterns used throughout the transaction table implementation. Following these conventions ensures consistency, maintainability, and developer productivity across the codebase.

## Directory Structure Patterns

### 1. Component Organization

#### Table-Specific Components
```
components/tables/transactions/
├── data-table.tsx              # Main orchestrator component
├── columns.tsx                 # Column definitions and cell components
├── data-table-header.tsx       # Header with sorting functionality
├── bottom-bar.tsx              # Filter summary and totals
├── export-bar.tsx              # Bulk actions and export
├── empty-states.tsx            # No data and no results states
└── loading.tsx                 # Loading skeleton components
```

**Naming Pattern**: `{purpose}.tsx` or `{component-type}.tsx`
- **Main component**: `data-table.tsx` (descriptive of functionality)
- **Supporting components**: `{specific-purpose}.tsx` (e.g., `export-bar`, `bottom-bar`)
- **State components**: `{state-type}.tsx` (e.g., `loading`, `empty-states`)

#### Component Grouping Logic
- **Related functionality**: Components that work together are grouped in same directory
- **Feature-based**: Each table type (transactions, invoices, customers) has its own directory
- **Reusable vs. Specific**: Generic components go in `/components`, specific ones in `/components/tables/{entity}`

### 2. Hook Organization

#### Hook Directory Structure
```
hooks/
├── use-transaction-filter-params.ts                    # URL parameter management
├── use-transaction-filter-params-with-persistence.ts   # URL + localStorage integration
├── use-generic-filter-persistence.ts                   # Reusable persistence logic
├── use-local-storage.ts                               # Base localStorage utilities
├── use-sort-params.ts                                 # Sorting state management
├── use-sticky-columns.ts                             # Column positioning logic
├── use-table-scroll.ts                               # Scroll behavior and navigation
└── use-transaction-params.ts                         # Transaction-specific URL params
```

**Naming Pattern**: `use-{functionality}[-{specificity}].ts`
- **Base hooks**: `use-{core-functionality}.ts` (e.g., `use-local-storage`)
- **Entity-specific**: `use-{entity}-{functionality}.ts` (e.g., `use-transaction-filter-params`)
- **Enhanced versions**: `use-{functionality}-with-{enhancement}.ts`
- **Generic utilities**: `use-generic-{functionality}.ts`

### 3. Store Organization

#### Store Structure
```
store/
├── transactions.ts             # Transaction table state
├── invoice.ts                  # Invoice table state
├── assistant.ts                # AI assistant state
└── search.ts                   # Global search state
```

**Naming Pattern**: `{entity}.ts` or `{feature}.ts`
- **Entity stores**: Named after the primary data entity (e.g., `transactions`, `invoices`)
- **Feature stores**: Named after the feature they support (e.g., `assistant`, `search`)

### 4. Utility Organization

#### Utility Structure
```
utils/
├── transaction-filters.ts      # Filter-related utilities
├── columns.ts                  # Column management utilities
├── constants.ts                # Application constants
└── format.ts                   # Data formatting utilities
```

**Naming Pattern**: `{domain}-{purpose}.ts` or `{purpose}.ts`
- **Domain-specific**: `{entity}-{functionality}.ts` (e.g., `transaction-filters`)
- **Generic utilities**: `{functionality}.ts` (e.g., `format`, `constants`)

---

## File Naming Conventions

### 1. Component Files

#### Component File Patterns
```typescript
// ✅ Good: Descriptive, kebab-case
data-table.tsx
data-table-header.tsx
export-bar.tsx
bottom-bar.tsx
empty-states.tsx

// ❌ Avoid: Generic, unclear purpose
table.tsx
header.tsx
bar.tsx
states.tsx
```

**Rules**:
- **kebab-case**: All lowercase with hyphens
- **Descriptive**: Name should indicate the component's purpose
- **Consistent suffixes**: Use consistent patterns like `-bar`, `-header`, `-states`
- **Avoid abbreviations**: Write out full words for clarity

### 2. Hook Files

#### Hook File Patterns
```typescript
// ✅ Good: Clear purpose and scope
use-transaction-filter-params.ts
use-transaction-filter-params-with-persistence.ts
use-generic-filter-persistence.ts
use-local-storage.ts
use-sticky-columns.ts
use-table-scroll.ts

// ❌ Avoid: Unclear scope or purpose
use-filters.ts
use-params.ts
use-storage.ts
use-columns.ts
```

**Rules**:
- **use- prefix**: All hooks start with `use-`
- **kebab-case**: Consistent with component naming
- **Specific scope**: Include entity or feature name when not generic
- **Descriptive purpose**: Name should clearly indicate what the hook does

### 3. Type and Interface Files

#### Type File Patterns
```typescript
// ✅ Good: Clear domain and purpose
transaction-filters.ts    // Types related to transaction filtering
table-types.ts           // Generic table types
api-types.ts            // API response types

// ❌ Avoid: Generic or unclear
types.ts
interfaces.ts
models.ts
```

---

## Component Naming Patterns

### 1. Component Names

#### React Component Naming
```typescript
// ✅ Good: PascalCase, descriptive
export function DataTable() { }
export function DataTableHeader() { }
export function ExportBar() { }
export function BottomBar() { }
export function NoResults() { }
export function NoTransactions() { }

// ❌ Avoid: Unclear or generic
export function Table() { }
export function Header() { }
export function Bar() { }
export function EmptyState() { }
```

**Rules**:
- **PascalCase**: First letter of each word capitalized
- **Descriptive**: Name should indicate component's purpose
- **Consistent patterns**: Use similar naming for related components
- **Avoid generic terms**: Be specific about what the component does

### 2. Memoized Component Naming

#### Cell Component Patterns
```typescript
// ✅ Good: Descriptive cell components
const SelectCell = memo(({ checked, onChange }) => (/* ... */));
const DateCell = memo(({ date }) => (/* ... */));
const DescriptionCell = memo(({ description, name }) => (/* ... */));
const AmountCell = memo(({ amount, currency }) => (/* ... */));
const TagsCell = memo(({ tags }) => (/* ... */));
const ActionsCell = memo(({ transaction, onAction }) => (/* ... */));

// ❌ Avoid: Generic or unclear names
const Cell1 = memo(() => (/* ... */));
const CheckboxCell = memo(() => (/* ... */)); // Too generic
const ActionDropdown = memo(() => (/* ... */)); // Not following pattern
```

**Rules**:
- **{Purpose}Cell pattern**: All cell components end with `Cell`
- **Descriptive prefix**: Prefix indicates the cell's content or purpose
- **memo wrapper**: All cell components should be memoized for performance

---

## Hook Naming Patterns

### 1. Hook Function Names

#### Hook Naming Rules
```typescript
// ✅ Good: Clear purpose and scope
export function useTransactionFilterParams() { }
export function useTransactionFilterParamsWithPersistence() { }
export function useGenericFilterPersistence() { }
export function useLocalStorage() { }
export function useStickyColumns() { }
export function useTableScroll() { }

// ❌ Avoid: Unclear or too generic
export function useFilters() { }
export function useParams() { }
export function useStorage() { }
export function useColumns() { }
```

### 2. Hook Return Value Naming

#### Return Object Patterns
```typescript
// ✅ Good: Consistent and descriptive
return {
  filter,              // Current filter state
  setFilter,           // Filter setter function
  hasFilters,          // Boolean indicator
  clearAllFilters,     // Action function
};

return {
  containerRef,        // DOM reference
  canScrollLeft,       // Boolean state
  canScrollRight,      // Boolean state
  isScrollable,        // Boolean state
  scrollLeft,          // Action function
  scrollRight,         // Action function
};

// ❌ Avoid: Inconsistent or unclear
return {
  data,               // Too generic
  setState,           // Unclear what state
  flag,               // Non-descriptive
  handler,            // Unclear purpose
};
```

**Patterns**:
- **State values**: Direct, descriptive names (e.g., `filter`, `rowSelection`)
- **Setters**: `set{StateName}` pattern (e.g., `setFilter`, `setRowSelection`)
- **Booleans**: `is{Condition}` or `can{Action}` or `has{Property}` patterns
- **Actions**: Verb-based names (e.g., `clearAllFilters`, `scrollLeft`)

---

## Variable and Function Naming

### 1. Variable Naming Patterns

#### State Variables
```typescript
// ✅ Good: Descriptive and consistent
const [rowSelection, setRowSelection] = useState({});
const [columnVisibility, setColumnVisibility] = useState({});
const [canScrollLeft, setCanScrollLeft] = useState(false);
const [isScrollable, setIsScrollable] = useState(false);

// ❌ Avoid: Generic or unclear
const [data, setData] = useState({});
const [state, setState] = useState(false);
const [flag, setFlag] = useState(false);
```

#### Computed Values
```typescript
// ✅ Good: Descriptive computation purpose
const deferredSearch = useDeferredValue(filter.q);
const tableData = useMemo(() => data?.pages.flatMap((page) => page.data) ?? [], [data]);
const selectedCount = useMemo(() => Object.keys(rowSelection).length, [rowSelection]);
const hasActiveFilters = Object.values(filter).some((value) => value !== null);

// ❌ Avoid: Generic or unclear
const deferred = useDeferredValue(filter.q);
const memoized = useMemo(() => /* computation */, []);
const count = useMemo(() => /* calculation */, []);
```

### 2. Function Naming Patterns

#### Event Handlers
```typescript
// ✅ Good: Action-based naming
const handleViewDetails = useCallback(() => {
  onViewDetails?.(transaction.id);
}, [transaction.id, onViewDetails]);

const handleCopyUrl = useCallback(() => {
  onCopyUrl?.(transaction.id);
}, [transaction.id, onCopyUrl]);

const handleUpdateTransaction = useCallback((status: string) => {
  onUpdateTransaction?.({ id: transaction.id, status });
}, [transaction.id, onUpdateTransaction]);

// ❌ Avoid: Generic or unclear
const handler1 = useCallback(() => { }, []);
const onClick = useCallback(() => { }, []); // Too generic
const callback = useCallback(() => { }, []);
```

#### Utility Functions
```typescript
// ✅ Good: Action-based, descriptive
const createSortQuery = useCallback((id: string) => { }, []);
const checkScrollability = useCallback(() => { }, []);
const syncColumnIndex = useCallback(() => { }, []);
const getColumnPositions = useCallback(() => { }, []);

// ❌ Avoid: Generic or unclear
const process = useCallback(() => { }, []);
const calculate = useCallback(() => { }, []);
const update = useCallback(() => { }, []);
```

---

## Type and Interface Naming

### 1. Type Definitions

#### Type Naming Patterns
```typescript
// ✅ Good: Descriptive and specific
export type TransactionFilters = {
  q?: string | null;
  categories?: string[] | null;
  // ...
};

export type SortItem = {
  id: string;
  desc: boolean;
};

export type UseTableScrollOptions = {
  scrollAmount?: number;
  useColumnWidths?: boolean;
  startFromColumn?: number;
};

// ❌ Avoid: Generic or unclear
export type Filters = { }; // Too generic
export type Options = { }; // Too generic
export type Props = { };   // Too generic
```

### 2. Interface Definitions

#### Interface Naming Patterns
```typescript
// ✅ Good: Clear purpose and scope
interface TransactionsState {
  canDelete?: boolean;
  columns: Column<any, unknown>[];
  rowSelection: Record<string, boolean>;
  setColumns: (columns?: Column<any, unknown>[]) => void;
  setRowSelection: (updater: Updater<RowSelectionState>) => void;
  setCanDelete: (canDelete?: boolean) => void;
}

interface UseStickyColumnsProps {
  columnVisibility: Record<string, boolean>;
  table: Table<any>;
  loading?: boolean;
}

// ❌ Avoid: Generic naming
interface State { }
interface Props { }
interface Options { }
```

---

## Constant Naming Patterns

### 1. Application Constants

#### Constant Naming Rules
```typescript
// ✅ Good: SCREAMING_SNAKE_CASE, descriptive
export const EMPTY_FILTER_STATE: TransactionFilters = {
  q: null,
  categories: null,
  // ...
};

export const STICKY_COLUMN_IDS = ["select", "date", "description"];

export const DEFAULT_SCROLL_AMOUNT = 120;

export const STORAGE_KEYS = {
  TRANSACTION_FILTERS: "transaction-filters",
  COLUMN_VISIBILITY: "column-visibility",
} as const;

// ❌ Avoid: Unclear or inconsistent
export const EMPTY = { };
export const IDS = ["select"];
export const DEFAULT = 120;
```

### 2. Enum-like Objects

#### Enum Naming Patterns
```typescript
// ✅ Good: Descriptive and organized
export const Cookies = {
  TransactionsColumns: "transactions-columns",
  InvoicesColumns: "invoices-columns",
} as const;

export const FilterTypes = {
  SEARCH: "search",
  CATEGORY: "category",
  DATE_RANGE: "dateRange",
} as const;

// ❌ Avoid: Generic or unclear
export const Keys = {
  A: "a",
  B: "b",
} as const;
```

---

## Import/Export Naming

### 1. Import Patterns

#### Import Organization
```typescript
// ✅ Good: Organized and clear
import { useCallback, useEffect, useMemo, useState } from "react";
import { useHotkeys } from "react-hotkeys-hook";
import { useSuspenseInfiniteQuery } from "@tanstack/react-query";

import { useTransactionFilterParamsWithPersistence } from "@/hooks/use-transaction-filter-params-with-persistence";
import { useSortParams } from "@/hooks/use-sort-params";
import { useStickyColumns } from "@/hooks/use-sticky-columns";

import { Button } from "@midday/ui/button";
import { Table, TableBody, TableCell, TableRow } from "@midday/ui/table";

// ❌ Avoid: Mixed grouping
import { useState, Button, useHotkeys } from "mixed-imports";
```

### 2. Export Patterns

#### Export Naming
```typescript
// ✅ Good: Named exports with descriptive names
export function DataTable() { }
export function useTransactionFilterParams() { }
export const EMPTY_FILTER_STATE = { };
export type TransactionFilters = { };

// ✅ Good: Default exports when appropriate (single main export)
const TransactionTable = () => { };
export default TransactionTable;

// ❌ Avoid: Unclear exports
export { a, b, c } from "./somewhere";
export default () => { }; // Unnamed default
```

---

## Documentation Naming

### 1. Documentation Files

#### Documentation File Patterns
```
data-table-documentation/
├── components/
│   ├── data-table.md
│   ├── columns.md
│   ├── data-table-header.md
│   ├── bottom-bar.md
│   ├── export-bar.md
│   ├── empty-states.md
│   └── loading.md
├── hooks/
│   ├── use-transaction-filter-params.md
│   ├── use-transaction-filter-params-with-persistence.md
│   ├── use-generic-filter-persistence.md
│   ├── use-local-storage.md
│   ├── use-sort-params.md
│   ├── use-sticky-columns.md
│   └── use-table-scroll.md
├── dependency-mapping.md
├── performance-optimizations.md
├── implementation-guide.md
└── naming-conventions.md
```

**Rules**:
- **Match source files**: Documentation files mirror source file names
- **kebab-case**: Consistent with source file naming
- **Descriptive**: Names should clearly indicate what's documented

---

## Best Practices Summary

### 1. Consistency Rules
- **Follow established patterns**: Use the same naming patterns throughout the codebase
- **Be descriptive**: Names should clearly indicate purpose and scope
- **Avoid abbreviations**: Write out full words for better readability
- **Use consistent casing**: kebab-case for files, PascalCase for components, camelCase for variables

### 2. Organizational Principles
- **Group by feature**: Related files should be grouped together
- **Hierarchy matters**: More specific files go in subdirectories
- **Reusability**: Generic utilities go in shared locations
- **Separation of concerns**: Each file should have a single, clear purpose

### 3. Naming Guidelines
- **Start specific, generalize later**: Begin with descriptive names, abstract when patterns emerge
- **Include context**: Names should include enough context to be understood in isolation
- **Follow conventions**: Use established patterns from the React and TypeScript communities
- **Be future-proof**: Names should still make sense as the codebase evolves

### 4. Common Anti-Patterns to Avoid

```typescript
// ❌ Avoid these patterns
export function Component() { }     // Too generic
const data = useState();           // Unclear purpose
const handler = () => { };         // Non-descriptive
export const utils = { };         // Generic export
import * as everything from "lib"; // Unclear imports
```

Following these naming conventions ensures that the codebase remains maintainable, searchable, and understandable for both current and future developers.