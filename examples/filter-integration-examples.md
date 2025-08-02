# Filter Integration Examples

> **Reference Repository**: [midday-ai/midday](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/) - Transaction filter hooks and patterns

## Overview
This guide demonstrates how to integrate filtering following the exact patterns used in the transaction table. All examples are based on the actual filter hooks implementation.

## Actual Filter Hook Patterns

### 1. Basic Filter Parameters Hook

From `use-transaction-filter-params.ts`:

```typescript
import { useQueryStates } from "nuqs";
import {
  createLoader,
  parseAsArrayOf,
  parseAsInteger,
  parseAsString,
  parseAsStringLiteral,
} from "nuqs/server";

export const transactionFilterParamsSchema = {
  q: parseAsString,
  attachments: parseAsStringLiteral(["exclude", "include"] as const),
  start: parseAsString,
  end: parseAsString,
  categories: parseAsArrayOf(parseAsString),
  tags: parseAsArrayOf(parseAsString),
  accounts: parseAsArrayOf(parseAsString),
  assignees: parseAsArrayOf(parseAsString),
  amount_range: parseAsArrayOf(parseAsInteger),
  amount: parseAsArrayOf(parseAsString),
  recurring: parseAsArrayOf(
    parseAsStringLiteral(["all", "weekly", "monthly", "annually"] as const),
  ),
  statuses: parseAsArrayOf(
    parseAsStringLiteral([
      "completed",
      "uncompleted",
      "archived",
      "excluded",
    ] as const),
  ),
};

export function useTransactionFilterParams() {
  const [filter, setFilter] = useQueryStates(transactionFilterParamsSchema, {
    // Clear URL when values are null/default
    clearOnDefault: true,
  });

  return {
    filter,
    setFilter,
    hasFilters: Object.values(filter).some((value) => value !== null),
  };
}

export const loadTransactionFilterParams = createLoader(
  transactionFilterParamsSchema,
);
```

### 2. Filter Persistence Hook

From `use-transaction-filter-params-with-persistence.ts`:

```typescript
"use client";

import {
  EMPTY_FILTER_STATE,
  type FilterHookReturn,
  type TransactionFilters,
} from "@/utils/transaction-filters";
import { useGenericFilterPersistence } from "./use-generic-filter-persistence";
import { useTransactionFilterParams } from "./use-transaction-filter-params";

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

### 3. Generic Filter Persistence Implementation

From `use-generic-filter-persistence.ts`:

```typescript
"use client";

import { cleanFilters, hasActiveUrlFilters } from "@/utils/transaction-filters";
import { useCallback, useEffect } from "react";
import { useFilterLocalStorage } from "./use-local-storage";

type UseFilterPersistenceOptions<T> = {
  storageKey: string;
  emptyState: T;
  currentFilters: T;
  setFilters: (filters: T) => void;
};

/**
 * Generic hook for persisting any kind of filters to localStorage
 * Can be reused for transactions, invoices, customers, etc.
 */
export function useGenericFilterPersistence<T extends Record<string, any>>({
  storageKey,
  emptyState,
  currentFilters,
  setFilters,
}: UseFilterPersistenceOptions<T>) {
  const { initializeFromStorage, saveToStorage } = useFilterLocalStorage<T>({
    key: storageKey,
    serialize: (filters) => JSON.stringify(cleanFilters(filters)),
    shouldSave: (filters) => Object.keys(cleanFilters(filters)).length > 0,
  });

  // Initialize from localStorage on mount (only if URL is empty)
  useEffect(() => {
    initializeFromStorage(
      (savedFilters: T) => setFilters(savedFilters),
      () => !hasActiveUrlFilters(currentFilters),
    );
  }, [initializeFromStorage, setFilters, currentFilters]);

  // Save changes to localStorage
  useEffect(() => {
    saveToStorage(currentFilters);
  }, [currentFilters, saveToStorage]);

  // Clear all filters helper
  const clearAllFilters = useCallback(() => {
    setFilters(emptyState);
  }, [setFilters, emptyState]);

  return {
    clearAllFilters,
  };
}
```

## Usage in Data Table

From `data-table.tsx`:

```typescript
import { useTransactionFilterParamsWithPersistence } from "@/hooks/use-transaction-filter-params-with-persistence";

export function DataTable({
  columnVisibility: columnVisibilityPromise,
}: Props) {
  const { filter, hasFilters } = useTransactionFilterParamsWithPersistence();
  const deferredSearch = useDeferredValue(filter.q);

  const infiniteQueryOptions = trpc.transactions.get.infiniteQueryOptions(
    {
      ...filter,
      q: deferredSearch,
      sort: params.sort,
    },
    {
      getNextPageParam: ({ meta }) => meta?.cursor,
    },
  );

  const { data, fetchNextPage, hasNextPage, refetch } =
    useSuspenseInfiniteQuery(infiniteQueryOptions);

  const showBottomBar = hasFilters && !Object.keys(rowSelection).length;

  // ... rest of component
}
```

## Filter Types and Utilities

From `utils/transaction-filters.ts`:

```typescript
// Type for transaction filters based on the schema
export type TransactionFilters = {
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

// Generic filter state type
export type FilterState = Record<string, any>;

// Filter hook return type
export type FilterHookReturn<T> = {
  filter: T;
  setFilter: (filters: T | ((current: T) => T)) => void;
  hasFilters: boolean;
  clearAllFilters: () => void;
};

// Empty filter state
export const EMPTY_FILTER_STATE: TransactionFilters = {
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

// Utility functions
export function cleanFilters(filters: any): any {
  const cleaned: any = {};
  
  for (const [key, value] of Object.entries(filters)) {
    if (value !== null && value !== undefined && value !== "" && 
        (!Array.isArray(value) || value.length > 0)) {
      cleaned[key] = value;
    }
  }
  
  return cleaned;
}

export function hasActiveUrlFilters(filters: any): boolean {
  return Object.keys(cleanFilters(filters)).length > 0;
}
```

## Creating Custom Filter Implementations

### 1. Invoice Filter Example

Following the transaction pattern for invoices:

```typescript
// hooks/use-invoice-filter-params.ts
import { useQueryStates } from "nuqs";
import {
  parseAsArrayOf,
  parseAsString,
  parseAsStringLiteral,
} from "nuqs/server";

export const invoiceFilterParamsSchema = {
  q: parseAsString,
  status: parseAsArrayOf(
    parseAsStringLiteral(["draft", "sent", "paid", "overdue"] as const),
  ),
  customers: parseAsArrayOf(parseAsString),
  start: parseAsString,
  end: parseAsString,
};

export function useInvoiceFilterParams() {
  const [filter, setFilter] = useQueryStates(invoiceFilterParamsSchema, {
    clearOnDefault: true,
  });

  return {
    filter,
    setFilter,
    hasFilters: Object.values(filter).some((value) => value !== null),
  };
}
```

### 2. Invoice Filter with Persistence

```typescript
// hooks/use-invoice-filter-params-with-persistence.ts
import { useGenericFilterPersistence } from "./use-generic-filter-persistence";
import { useInvoiceFilterParams } from "./use-invoice-filter-params";

type InvoiceFilters = {
  q?: string | null;
  status?: ("draft" | "sent" | "paid" | "overdue")[] | null;
  customers?: string[] | null;
  start?: string | null;
  end?: string | null;
};

const EMPTY_INVOICE_FILTER_STATE: InvoiceFilters = {
  q: null,
  status: null,
  customers: null,
  start: null,
  end: null,
};

export function useInvoiceFilterParamsWithPersistence() {
  const { filter, setFilter, hasFilters } = useInvoiceFilterParams();

  const { clearAllFilters } = useGenericFilterPersistence({
    storageKey: "invoice-filters",
    emptyState: EMPTY_INVOICE_FILTER_STATE,
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

## Filter Component Integration

### Basic Filter Bar Component

```typescript
import { Button } from "@midday/ui/button";
import { Input } from "@midday/ui/input";
import { Badge } from "@midday/ui/badge";
import { X } from "lucide-react";

interface FilterBarProps {
  filter: TransactionFilters;
  setFilter: (filters: TransactionFilters | ((current: TransactionFilters) => TransactionFilters)) => void;
  hasFilters: boolean;
  clearAllFilters: () => void;
}

export function FilterBar({ filter, setFilter, hasFilters, clearAllFilters }: FilterBarProps) {
  return (
    <div className="flex items-center gap-4 p-4 border-b">
      <Input
        placeholder="Search transactions..."
        value={filter.q || ""}
        onChange={(e) => setFilter({ ...filter, q: e.target.value || null })}
        className="max-w-sm"
      />
      
      {/* Status filters */}
      <div className="flex gap-2">
        {filter.statuses?.map((status) => (
          <Badge key={status} variant="secondary" className="gap-1">
            {status}
            <Button
              size="sm"
              variant="ghost"
              className="h-auto p-0 w-4 h-4"
              onClick={() => {
                const newStatuses = filter.statuses?.filter(s => s !== status);
                setFilter({ 
                  ...filter, 
                  statuses: newStatuses?.length ? newStatuses : null 
                });
              }}
            >
              <X className="w-3 h-3" />
            </Button>
          </Badge>
        ))}
      </div>

      {hasFilters && (
        <Button variant="ghost" onClick={clearAllFilters} size="sm">
          Clear filters
        </Button>
      )}
    </div>
  );
}
```

### Using Filter Bar with Data Table

```typescript
import { DataTable } from "@/components/tables/transactions/data-table";
import { FilterBar } from "@/components/filter-bar";
import { useTransactionFilterParamsWithPersistence } from "@/hooks/use-transaction-filter-params-with-persistence";

export function TransactionsPage() {
  const { filter, setFilter, hasFilters, clearAllFilters } = 
    useTransactionFilterParamsWithPersistence();

  return (
    <div className="space-y-4">
      <FilterBar 
        filter={filter} 
        setFilter={setFilter} 
        hasFilters={hasFilters} 
        clearAllFilters={clearAllFilters} 
      />
      <DataTable columnVisibility={getColumnVisibility()} />
    </div>
  );
}
```

## Key Patterns from the Implementation

### 1. URL Synchronization with nuqs
- Use `parseAsString`, `parseAsArrayOf`, `parseAsStringLiteral` for type-safe URL params
- Set `clearOnDefault: true` to clean URLs when filters are empty

### 2. localStorage Persistence
- Use `useGenericFilterPersistence` for reusable persistence logic
- Only save non-empty filters to avoid clutter
- Respect URL precedence over localStorage

### 3. Filter State Management
- Combine URL and localStorage hooks for complete solution
- Use deferred values for search to improve performance
- Provide clear filter functions for user experience

### 4. Type Safety
- Define specific filter types for each entity
- Use literal types for enum-like filter values
- Maintain empty state constants for consistent resets

This implementation follows the exact patterns used in the transaction table for robust, performant, and user-friendly filtering.