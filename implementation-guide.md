# Implementation Guide

> **Reference Implementations**:
> - [apps/dashboard/src/components/tables/transactions/](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/) - Complete transaction table implementation
> - [apps/dashboard/src/hooks/](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/) - All custom hooks
> - [apps/dashboard/src/store/transactions.ts](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/store/transactions.ts) - State management
> - [apps/dashboard/src/utils/](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/utils/) - Utility functions

## Overview
This guide provides step-by-step instructions for implementing a fully-featured data table based on the transaction table architecture. It covers everything from basic setup to advanced features like filtering, sorting, and infinite scrolling.

## Prerequisites

### Required Dependencies
```bash
npm install @tanstack/react-table @tanstack/react-query
npm install nuqs 
npm install framer-motion
npm install react-hotkeys-hook
npm install react-intersection-observer
npm install usehooks-ts
npm install zustand
```

### UI Dependencies (adjust to your UI library)
```bash
npm install @midday/ui
# Or your preferred component library
```

---

## Step 1: Basic Table Setup

### 1.1 Create Basic Table Structure

**File**: `components/data-table.tsx`

```typescript
"use client";

import { Table, TableBody, TableCell, TableRow } from "@/components/ui/table";
import { useReactTable, getCoreRowModel, flexRender } from "@tanstack/react-table";
import { useState } from "react";

interface DataTableProps<T> {
  data: T[];
  columns: any[];
}

export function DataTable<T>({ data, columns }: DataTableProps<T>) {
  const [rowSelection, setRowSelection] = useState({});
  
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    onRowSelectionChange: setRowSelection,
    state: {
      rowSelection,
    },
  });

  return (
    <div className="overflow-x-auto">
      <Table>
        <TableBody>
          {table.getRowModel().rows.map((row) => (
            <TableRow key={row.id}>
              {row.getVisibleCells().map((cell) => (
                <TableCell key={cell.id}>
                  {flexRender(cell.column.columnDef.cell, cell.getContext())}
                </TableCell>
              ))}
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </div>
  );
}
```

### 1.2 Define Column Structure

**File**: `components/columns.tsx`

```typescript
import { ColumnDef } from "@tanstack/react-table";
import { Checkbox } from "@/components/ui/checkbox";
import { memo } from "react";

// Example data type
type Transaction = {
  id: string;
  description: string;
  amount: number;
  date: string;
};

// Memoized cell components for performance
const SelectCell = memo(({ checked, onChange }: { 
  checked: boolean; 
  onChange: (value: boolean) => void;
}) => (
  <Checkbox
    checked={checked}
    onCheckedChange={onChange}
    aria-label="Select row"
  />
));

export const columns: ColumnDef<Transaction>[] = [
  {
    id: "select",
    header: ({ table }) => (
      <SelectCell
        checked={table.getIsAllPageRowsSelected()}
        onChange={(value) => table.toggleAllPageRowsSelected(!!value)}
      />
    ),
    cell: ({ row }) => (
      <SelectCell
        checked={row.getIsSelected()}
        onChange={(value) => row.toggleSelected(!!value)}
      />
    ),
    enableSorting: false,
    enableHiding: false,
  },
  {
    accessorKey: "description",
    header: "Description",
    cell: ({ row }) => <span>{row.getValue("description")}</span>,
  },
  {
    accessorKey: "amount",
    header: "Amount",
    cell: ({ row }) => {
      const amount = row.getValue("amount") as number;
      return <span>${amount.toFixed(2)}</span>;
    },
  },
  {
    accessorKey: "date",
    header: "Date",
    cell: ({ row }) => {
      const date = row.getValue("date") as string;
      return <span>{new Date(date).toLocaleDateString()}</span>;
    },
  },
];
```

---

## Step 2: Add State Management

### 2.1 Create Zustand Store

**File**: `store/data-table.ts`

```typescript
import type { Column, RowSelectionState, Updater } from "@tanstack/react-table";
import { create } from "zustand";

interface DataTableState {
  columns: Column<any, unknown>[];
  rowSelection: Record<string, boolean>;
  canDelete: boolean;
  setColumns: (columns?: Column<any, unknown>[]) => void;
  setRowSelection: (updater: Updater<RowSelectionState>) => void;
  setCanDelete: (canDelete?: boolean) => void;
}

export const useDataTableStore = create<DataTableState>()((set) => ({
  columns: [],
  rowSelection: {},
  canDelete: false,
  
  setColumns: (columns) => set({ columns: columns || [] }),
  
  setRowSelection: (updater: Updater<RowSelectionState>) =>
    set((state) => ({
      rowSelection:
        typeof updater === "function" ? updater(state.rowSelection) : updater,
    })),
    
  setCanDelete: (canDelete) => set({ canDelete }),
}));
```

### 2.2 Integrate Store with Table

Update `data-table.tsx`:

```typescript
import { useDataTableStore } from "@/store/data-table";

export function DataTable<T>({ data, columns }: DataTableProps<T>) {
  const { rowSelection, setRowSelection } = useDataTableStore();
  
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    onRowSelectionChange: setRowSelection,
    state: {
      rowSelection,
    },
  });

  // Rest of component...
}
```

---

## Step 3: Add URL-Based Filtering

### 3.1 Create Filter Parameter Hook

**File**: `hooks/use-filter-params.ts`

```typescript
import { useQueryStates } from "nuqs";
import {
  parseAsArrayOf,
  parseAsString,
  parseAsStringLiteral,
} from "nuqs/server";

export const filterParamsSchema = {
  q: parseAsString,
  categories: parseAsArrayOf(parseAsString),
  status: parseAsStringLiteral(["active", "inactive", "pending"] as const),
  start: parseAsString,
  end: parseAsString,
};

export function useFilterParams() {
  const [filter, setFilter] = useQueryStates(filterParamsSchema, {
    clearOnDefault: true,
  });

  return {
    filter,
    setFilter,
    hasFilters: Object.values(filter).some((value) => value !== null),
  };
}
```

### 3.2 Create Filter Types

**File**: `utils/filters.ts`

```typescript
export type FilterState = {
  q?: string | null;
  categories?: string[] | null;
  status?: "active" | "inactive" | "pending" | null;
  start?: string | null;
  end?: string | null;
};

export const EMPTY_FILTER_STATE: FilterState = {
  q: null,
  categories: null,
  status: null,
  start: null,
  end: null,
};

export function cleanFilters(filters: FilterState): Partial<FilterState> {
  const cleaned: Partial<FilterState> = {};
  
  for (const [key, value] of Object.entries(filters)) {
    if (value !== null && value !== undefined && value !== "" && 
        (!Array.isArray(value) || value.length > 0)) {
      cleaned[key as keyof FilterState] = value;
    }
  }
  
  return cleaned;
}

export function hasActiveUrlFilters(filters: FilterState): boolean {
  return Object.keys(cleanFilters(filters)).length > 0;
}
```

---

## Step 4: Add localStorage Persistence

### 4.1 Create localStorage Hook

**File**: `hooks/use-local-storage.ts`

```typescript
import { useCallback, useEffect, useRef } from "react";
import { useDebounceCallback } from "usehooks-ts";

type UseFilterLocalStorageOptions<T> = {
  key: string;
  debounceMs?: number;
  serialize?: (value: T) => string;
  deserialize?: (value: string) => T;
  shouldSave?: (value: T) => boolean;
};

export function useFilterLocalStorage<T>({
  key,
  debounceMs = 500,
  serialize = JSON.stringify,
  deserialize = JSON.parse,
  shouldSave = () => true,
}: UseFilterLocalStorageOptions<T>) {
  const hasInitialized = useRef(false);

  const debouncedSave = useDebounceCallback((value: T) => {
    try {
      if (shouldSave(value)) {
        localStorage.setItem(key, serialize(value));
      } else {
        localStorage.removeItem(key);
      }
    } catch (error) {
      console.error(`Failed to save to localStorage (${key}):`, error);
    }
  }, debounceMs);

  const loadFromStorage = useCallback((): T | null => {
    try {
      const item = localStorage.getItem(key);
      return item ? deserialize(item) : null;
    } catch (error) {
      console.error(`Failed to load from localStorage (${key}):`, error);
      localStorage.removeItem(key);
      return null;
    }
  }, [key, deserialize]);

  const initializeFromStorage = useCallback(
    (onApply: (value: T) => void, shouldApply?: () => boolean) => {
      if (hasInitialized.current) return;

      const saved = loadFromStorage();
      if (saved && (!shouldApply || shouldApply())) {
        onApply(saved);
      }

      hasInitialized.current = true;
    },
    [loadFromStorage],
  );

  const saveToStorage = useCallback(
    (value: T) => {
      if (!hasInitialized.current) return;
      debouncedSave(value);
    },
    [debouncedSave],
  );

  return {
    initializeFromStorage,
    saveToStorage,
    loadFromStorage,
  };
}
```

### 4.2 Create Filter Persistence Hook

**File**: `hooks/use-filter-persistence.ts`

```typescript
import { useCallback, useEffect } from "react";
import { useFilterLocalStorage } from "./use-local-storage";
import { cleanFilters, hasActiveUrlFilters } from "@/utils/filters";

type UseFilterPersistenceOptions<T> = {
  storageKey: string;
  emptyState: T;
  currentFilters: T;
  setFilters: (filters: T) => void;
};

export function useFilterPersistence<T extends Record<string, any>>({
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

  const clearAllFilters = useCallback(() => {
    setFilters(emptyState);
  }, [setFilters, emptyState]);

  return {
    clearAllFilters,
  };
}
```

### 4.3 Combine URL and localStorage

**File**: `hooks/use-filter-params-with-persistence.ts`

```typescript
import { EMPTY_FILTER_STATE, type FilterState } from "@/utils/filters";
import { useFilterPersistence } from "./use-filter-persistence";
import { useFilterParams } from "./use-filter-params";

export function useFilterParamsWithPersistence() {
  const { filter, setFilter, hasFilters } = useFilterParams();

  const { clearAllFilters } = useFilterPersistence({
    storageKey: "data-table-filters",
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

---

## Step 5: Add Infinite Scrolling

### 5.1 Setup tRPC Infinite Query

Update `data-table.tsx`:

```typescript
import { useSuspenseInfiniteQuery } from "@tanstack/react-query";
import { useInView } from "react-intersection-observer";
import { useEffect, useMemo } from "react";

export function DataTable() {
  const { filter } = useFilterParamsWithPersistence();
  const { ref, inView } = useInView();

  // Replace with your API client
  const { data, fetchNextPage, hasNextPage } = useSuspenseInfiniteQuery({
    queryKey: ["data", filter],
    queryFn: ({ pageParam }) => fetchData({ ...filter, cursor: pageParam }),
    getNextPageParam: (lastPage) => lastPage.meta?.cursor,
  });

  // Trigger fetch when scroll reaches bottom
  useEffect(() => {
    if (inView && hasNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, fetchNextPage]);

  // Flatten paginated data
  const tableData = useMemo(() => {
    return data?.pages.flatMap((page) => page.data) ?? [];
  }, [data]);

  return (
    <div className="overflow-x-auto">
      <Table>
        <TableBody>
          {table.getRowModel().rows.map((row) => (
            <TableRow key={row.id}>
              {/* Row cells */}
            </TableRow>
          ))}
          {/* Infinite scroll trigger */}
          <TableRow ref={ref}>
            <TableCell colSpan={columns.length}>
              {hasNextPage ? "Loading more..." : "End of results"}
            </TableCell>
          </TableRow>
        </TableBody>
      </Table>
    </div>
  );
}
```

---

## Step 6: Add Sorting

### 6.1 Create Sort Parameters Hook

**File**: `hooks/use-sort-params.ts`

```typescript
import { useQueryStates } from "nuqs";
import { parseAsArrayOf } from "nuqs/server";

export type SortItem = {
  id: string;
  desc: boolean;
};

// Custom parser for sort objects
const parseAsSortItem = {
  parse: (value: string): SortItem => {
    const [id, desc] = value.split(":");
    return { id, desc: desc === "desc" };
  },
  serialize: (value: SortItem): string => {
    return `${value.id}:${value.desc ? "desc" : "asc"}`;
  },
};

export function useSortParams() {
  const [params, setParams] = useQueryStates({
    sort: parseAsArrayOf(parseAsSortItem),
  });

  const setSortBy = (sortItems: SortItem[]) => {
    setParams({ sort: sortItems.length > 0 ? sortItems : null });
  };

  return {
    params: {
      sort: params.sort || [],
    },
    setSortBy,
  };
}
```

### 6.2 Add Sortable Header

**File**: `components/data-table-header.tsx`

```typescript
import { useCallback } from "react";
import { useSortParams, type SortItem } from "@/hooks/use-sort-params";
import { Button } from "@/components/ui/button";
import { ArrowUpDown, ArrowUp, ArrowDown } from "lucide-react";

interface SortableHeaderProps {
  column: any;
  children: React.ReactNode;
}

export function SortableHeader({ column, children }: SortableHeaderProps) {
  const { params, setSortBy } = useSortParams();

  const createSortQuery = useCallback(
    (id: string) => {
      const existing = params.sort?.find((s) => s.id === id);

      if (!existing) {
        // First click: ascending
        return [{ id, desc: false }];
      }

      if (!existing.desc) {
        // Second click: descending
        return [{ id, desc: true }];
      }

      // Third click: remove sorting
      return params.sort?.filter((s) => s.id !== id) ?? [];
    },
    [params.sort]
  );

  const handleSort = () => {
    const newSort = createSortQuery(column.id);
    setSortBy(newSort);
  };

  const currentSort = params.sort?.find((s) => s.id === column.id);
  const getSortIcon = () => {
    if (!currentSort) return <ArrowUpDown className="h-4 w-4" />;
    return currentSort.desc ? 
      <ArrowDown className="h-4 w-4" /> : 
      <ArrowUp className="h-4 w-4" />;
  };

  return (
    <Button
      variant="ghost"
      onClick={handleSort}
      className="h-auto p-0 font-medium"
    >
      {children}
      {getSortIcon()}
    </Button>
  );
}
```

---

## Step 7: Add Sticky Columns

### 7.1 Create Sticky Columns Hook

**File**: `hooks/use-sticky-columns.ts`

```typescript
import { useCallback } from "react";
import type { Table, Column } from "@tanstack/react-table";

const STICKY_COLUMN_IDS = ["select", "date", "description"];

interface UseStickyColumnsProps {
  columnVisibility: Record<string, boolean>;
  table: Table<any>;
}

export function useStickyColumns({ columnVisibility, table }: UseStickyColumnsProps) {
  const getColumnWidth = useCallback((columnId: string): number => {
    // Return appropriate width for each column
    const widths: Record<string, number> = {
      select: 40,
      date: 100,
      description: 200,
    };
    return widths[columnId] || 150;
  }, []);

  const getStickyPosition = useCallback(
    (columnId: string): number | null => {
      const stickyColumns = table.getAllLeafColumns().filter((col) => {
        return STICKY_COLUMN_IDS.includes(col.id) && 
               (columnVisibility[col.id] !== false);
      });

      let position = 0;
      for (const col of stickyColumns) {
        if (col.id === columnId) {
          return position;
        }
        position += getColumnWidth(col.id);
      }

      return null;
    },
    [table, columnVisibility, getColumnWidth]
  );

  const getStickyStyle = useCallback(
    (column: Column<any, unknown>) => {
      const stickyPosition = getStickyPosition(column.id);

      if (stickyPosition !== null) {
        return {
          "--sticky-left": `${stickyPosition}px`,
          position: "sticky",
          left: `var(--sticky-left)`,
          zIndex: 10,
        } as React.CSSProperties;
      }

      return {};
    },
    [getStickyPosition]
  );

  const getStickyClassName = useCallback(
    (column: Column<any, unknown>) => {
      const isSticky = getStickyPosition(column.id) !== null;
      return isSticky ? "bg-background border-r border-border" : "";
    },
    [getStickyPosition]
  );

  return {
    getStickyStyle,
    getStickyClassName,
  };
}
```

### 7.2 Apply Sticky Styles

Update your table cells to use sticky positioning:

```typescript
// In your column definitions
{
  id: "select",
  meta: {
    className: "sticky left-0 bg-background border-r z-10",
  },
  // ... rest of column config
}
```

---

## Step 8: Add Keyboard Navigation

### 8.1 Create Table Scroll Hook

**File**: `hooks/use-table-scroll.ts`

```typescript
import { useCallback, useEffect, useRef, useState } from "react";
import { useHotkeys } from "react-hotkeys-hook";

interface UseTableScrollOptions {
  scrollAmount?: number;
}

export function useTableScroll(options: UseTableScrollOptions = {}) {
  const { scrollAmount = 120 } = options;
  const containerRef = useRef<HTMLDivElement>(null);
  const [canScrollLeft, setCanScrollLeft] = useState(false);
  const [canScrollRight, setCanScrollRight] = useState(false);
  const [isScrollable, setIsScrollable] = useState(false);

  const checkScrollability = useCallback(() => {
    const container = containerRef.current;
    if (!container) return;

    const { scrollWidth, clientWidth, scrollLeft } = container;
    const isTableScrollable = scrollWidth > clientWidth;

    setIsScrollable(isTableScrollable);
    setCanScrollLeft(scrollLeft > 0);
    setCanScrollRight(scrollLeft < scrollWidth - clientWidth - 1);
  }, []);

  const scrollLeft = useCallback(() => {
    const container = containerRef.current;
    if (!container) return;

    container.scrollBy({
      left: -scrollAmount,
      behavior: "smooth",
    });
  }, [scrollAmount]);

  const scrollRight = useCallback(() => {
    const container = containerRef.current;
    if (!container) return;

    container.scrollBy({
      left: scrollAmount,
      behavior: "smooth",
    });
  }, [scrollAmount]);

  useEffect(() => {
    const container = containerRef.current;
    if (!container) return;

    const handleScroll = () => checkScrollability();
    const handleResize = () => checkScrollability();

    container.addEventListener("scroll", handleScroll);
    window.addEventListener("resize", handleResize);

    // Initial check
    checkScrollability();

    return () => {
      container.removeEventListener("scroll", handleScroll);
      window.removeEventListener("resize", handleResize);
    };
  }, [checkScrollability]);

  // Keyboard navigation
  useHotkeys(
    "ArrowLeft, ArrowRight",
    (event) => {
      if (event.key === "ArrowLeft" && canScrollLeft) {
        scrollLeft();
      }
      if (event.key === "ArrowRight" && canScrollRight) {
        scrollRight();
      }
    },
    {
      enabled: isScrollable,
      preventDefault: true,
    }
  );

  return {
    containerRef,
    canScrollLeft,
    canScrollRight,
    isScrollable,
    scrollLeft,
    scrollRight,
  };
}
```

---

## Step 9: Add Loading States

### 9.1 Create Loading Component

**File**: `components/loading.tsx`

```typescript
import { Skeleton } from "@/components/ui/skeleton";
import { Table, TableBody, TableCell, TableRow } from "@/components/ui/table";

interface LoadingProps {
  rows?: number;
  columns?: number;
}

export function Loading({ rows = 10, columns = 4 }: LoadingProps) {
  return (
    <Table>
      <TableBody>
        {Array.from({ length: rows }).map((_, index) => (
          <TableRow key={index}>
            {Array.from({ length: columns }).map((_, cellIndex) => (
              <TableCell key={cellIndex}>
                <Skeleton className="h-4 w-full" />
              </TableCell>
            ))}
          </TableRow>
        ))}
      </TableBody>
    </Table>
  );
}
```

### 9.2 Add Error and Empty States

**File**: `components/empty-states.tsx`

```typescript
import { Button } from "@/components/ui/button";
import { FileX, Search } from "lucide-react";

export function NoResults() {
  return (
    <div className="flex flex-col items-center justify-center py-12">
      <Search className="h-12 w-12 text-muted-foreground mb-4" />
      <h3 className="text-lg font-semibold mb-2">No results found</h3>
      <p className="text-muted-foreground mb-4">
        Try adjusting your search or filter criteria.
      </p>
      <Button variant="outline" onClick={() => window.location.reload()}>
        Clear filters
      </Button>
    </div>
  );
}

export function NoData() {
  return (
    <div className="flex flex-col items-center justify-center py-12">
      <FileX className="h-12 w-12 text-muted-foreground mb-4" />
      <h3 className="text-lg font-semibold mb-2">No data available</h3>
      <p className="text-muted-foreground">
        Get started by adding your first item.
      </p>
    </div>
  );
}
```

---

## Step 10: Performance Optimization

### 10.1 Add useDeferredValue for Search

```typescript
import { useDeferredValue } from "react";

export function DataTable() {
  const { filter } = useFilterParamsWithPersistence();
  const deferredSearch = useDeferredValue(filter.q);

  const { data } = useSuspenseInfiniteQuery({
    queryKey: ["data", { ...filter, q: deferredSearch }],
    // ... rest of query config
  });
}
```

### 10.2 Memoize Expensive Calculations

```typescript
import { useMemo } from "react";

export function DataTable() {
  const tableData = useMemo(() => {
    return data?.pages.flatMap((page) => page.data) ?? [];
  }, [data]);

  const selectedCount = useMemo(() => {
    return Object.keys(rowSelection).length;
  }, [rowSelection]);
}
```

---

## Common Patterns and Best Practices

### 1. Error Handling
```typescript
const ErrorBoundary = ({ children }: { children: React.ReactNode }) => {
  // Implement error boundary for table components
};
```

### 2. TypeScript Integration
```typescript
// Define your data types
interface TableItem {
  id: string;
  name: string;
  status: "active" | "inactive";
  createdAt: string;
}

// Use generic types for reusability
export function DataTable<T extends { id: string }>({
  data,
  columns,
}: DataTableProps<T>) {
  // Implementation
}
```

### 3. Testing Strategy
```typescript
// Test key interactions
describe("DataTable", () => {
  it("should handle row selection", () => {
    // Test selection logic
  });

  it("should persist filters to localStorage", () => {
    // Test persistence
  });

  it("should handle infinite scrolling", () => {
    // Test scroll behavior
  });
});
```

### 4. Accessibility
```typescript
// Add proper ARIA labels
<Table role="table" aria-label="Data table">
  <TableBody>
    {rows.map((row) => (
      <TableRow 
        key={row.id}
        role="row"
        aria-selected={row.getIsSelected()}
      >
        {/* Table cells */}
      </TableRow>
    ))}
  </TableBody>
</Table>
```

## Troubleshooting Guide

### Common Issues

1. **Filters not persisting**: Check localStorage permissions and error handling
2. **Infinite scroll not working**: Verify `useInView` trigger positioning
3. **Performance issues**: Check for missing memoization and excessive re-renders
4. **Sorting conflicts**: Ensure proper URL parameter serialization
5. **Sticky columns misaligned**: Check CSS custom properties and positioning

### Debugging Tools

1. **React Developer Tools**: Monitor re-renders and state changes
2. **Network tab**: Verify API calls and infinite query behavior
3. **Console logs**: Add debugging for localStorage and URL state
4. **Performance tab**: Profile scroll and filter performance

This implementation guide provides a solid foundation for building feature-rich data tables. Customize the patterns according to your specific requirements and data structure.