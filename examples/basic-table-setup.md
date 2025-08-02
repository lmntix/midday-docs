# Basic Table Setup

> **Reference Repository**: [midday-ai/midday](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/) - Transaction table implementation

## Overview
This guide demonstrates how to set up a basic data table following the exact structure used in the transaction table. All patterns are based on the actual codebase implementation.

## Actual Data Table Structure

### 1. Basic Data Table Component (Based on data-table.tsx)

```typescript
"use client";

import { Table, TableBody, TableCell, TableRow } from "@midday/ui/table";
import { TooltipProvider } from "@midday/ui/tooltip";
import {
  type VisibilityState,
  flexRender,
  getCoreRowModel,
  useReactTable,
} from "@tanstack/react-table";
import { useState, useMemo } from "react";

interface BasicDataTableProps<T> {
  data: T[];
  columns: any[];
  columnVisibility?: VisibilityState;
}

export function BasicDataTable<T>({ 
  data, 
  columns, 
  columnVisibility = {} 
}: BasicDataTableProps<T>) {
  const [rowSelection, setRowSelection] = useState({});
  
  const table = useReactTable({
    getRowId: (row: any) => row?.id,
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    onRowSelectionChange: setRowSelection,
    onColumnVisibilityChange: () => {}, // Add column visibility handler if needed
    state: {
      rowSelection,
      columnVisibility,
    },
  });

  return (
    <TooltipProvider>
      <div className="rounded-md border overflow-hidden">
        <Table>
          <TableBody>
            {table.getRowModel().rows?.length ? (
              table.getRowModel().rows.map((row) => (
                <TableRow
                  key={row.id}
                  data-state={row.getIsSelected() && "selected"}
                  className="group"
                >
                  {row.getVisibleCells().map((cell) => (
                    <TableCell 
                      key={cell.id}
                      className={cell.column.columnDef.meta?.className}
                    >
                      {flexRender(cell.column.columnDef.cell, cell.getContext())}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell colSpan={columns.length} className="h-24 text-center">
                  No results.
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </div>
    </TooltipProvider>
  );
}
```

### 2. Define Your Data Type

```typescript
// types/transaction.ts
export interface Transaction {
  id: string;
  description: string;
  amount: number;
  date: string;
  category: string;
  status: "pending" | "completed" | "failed";
}
```

### 3. Create Basic Columns

```typescript
// components/basic-columns.tsx
import { ColumnDef } from "@tanstack/react-table";
import { Checkbox } from "@/components/ui/checkbox";
import { Badge } from "@/components/ui/badge";
import { formatDate, formatCurrency } from "@/utils/format";

export const basicColumns: ColumnDef<Transaction>[] = [
  {
    id: "select",
    header: ({ table }) => (
      <Checkbox
        checked={table.getIsAllPageRowsSelected()}
        onCheckedChange={(value) => table.toggleAllPageRowsSelected(!!value)}
        aria-label="Select all"
      />
    ),
    cell: ({ row }) => (
      <Checkbox
        checked={row.getIsSelected()}
        onCheckedChange={(value) => row.toggleSelected(!!value)}
        aria-label="Select row"
      />
    ),
    enableSorting: false,
    enableHiding: false,
  },
  {
    accessorKey: "date",
    header: "Date",
    cell: ({ row }) => {
      const date = row.getValue("date") as string;
      return <span className="text-sm">{formatDate(date)}</span>;
    },
  },
  {
    accessorKey: "description",
    header: "Description",
    cell: ({ row }) => {
      const description = row.getValue("description") as string;
      return (
        <div className="font-medium">
          {description}
        </div>
      );
    },
  },
  {
    accessorKey: "category",
    header: "Category",
    cell: ({ row }) => {
      const category = row.getValue("category") as string;
      return <Badge variant="secondary">{category}</Badge>;
    },
  },
  {
    accessorKey: "amount",
    header: "Amount",
    cell: ({ row }) => {
      const amount = row.getValue("amount") as number;
      return (
        <div className="text-right font-medium">
          {formatCurrency(amount)}
        </div>
      );
    },
  },
  {
    accessorKey: "status",
    header: "Status",
    cell: ({ row }) => {
      const status = row.getValue("status") as string;
      return (
        <Badge variant={status === "completed" ? "default" : "secondary"}>
          {status}
        </Badge>
      );
    },
  },
];
```

### 4. Usage Example

```typescript
// pages/transactions.tsx
import { BasicDataTable } from "@/components/basic-data-table";
import { basicColumns } from "@/components/basic-columns";
import { useState, useEffect } from "react";

export default function TransactionsPage() {
  const [data, setData] = useState<Transaction[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Fetch your data
    fetchTransactions()
      .then(setData)
      .finally(() => setLoading(false));
  }, []);

  if (loading) {
    return <div>Loading...</div>;
  }

  return (
    <div className="container mx-auto py-10">
      <h1 className="text-2xl font-bold mb-4">Transactions</h1>
      <BasicDataTable data={data} columns={basicColumns} />
    </div>
  );
}
```

## Adding Header Row

### Enhanced Table with Header

```typescript
import { TableHeader, TableHead } from "@/components/ui/table";

export function BasicDataTableWithHeader<T>({ data, columns }: BasicDataTableProps<T>) {
  // ... existing table setup

  return (
    <div className="rounded-md border">
      <Table>
        <TableHeader>
          {table.getHeaderGroups().map((headerGroup) => (
            <TableRow key={headerGroup.id}>
              {headerGroup.headers.map((header) => (
                <TableHead key={header.id}>
                  {header.isPlaceholder
                    ? null
                    : flexRender(
                        header.column.columnDef.header,
                        header.getContext()
                      )}
                </TableHead>
              ))}
            </TableRow>
          ))}
        </TableHeader>
        <TableBody>
          {/* ... existing body content */}
        </TableBody>
      </Table>
    </div>
  );
}
```

## Static Data Example

### Sample Data for Testing

```typescript
// data/sample-transactions.ts
export const sampleTransactions: Transaction[] = [
  {
    id: "1",
    description: "Coffee Shop Purchase",
    amount: -4.50,
    date: "2024-01-15",
    category: "Food & Dining",
    status: "completed",
  },
  {
    id: "2",
    description: "Salary Deposit",
    amount: 5000.00,
    date: "2024-01-01",
    category: "Income",
    status: "completed",
  },
  {
    id: "3",
    description: "Electric Bill",
    amount: -89.99,
    date: "2024-01-10",
    category: "Utilities",
    status: "pending",
  },
  {
    id: "4",
    description: "Online Purchase",
    amount: -129.99,
    date: "2024-01-12",
    category: "Shopping",
    status: "failed",
  },
  {
    id: "5",
    description: "Rent Payment",
    amount: -1200.00,
    date: "2024-01-01",
    category: "Housing",
    status: "completed",
  },
];
```

### Using Sample Data

```typescript
import { sampleTransactions } from "@/data/sample-transactions";

export default function SampleTransactionsPage() {
  return (
    <div className="container mx-auto py-10">
      <h1 className="text-2xl font-bold mb-4">Sample Transactions</h1>
      <BasicDataTable data={sampleTransactions} columns={basicColumns} />
    </div>
  );
}
```

## Adding Basic Styling

### CSS Classes for Enhanced Appearance

```css
/* styles/table.css */
.data-table {
  @apply w-full;
}

.table-header {
  @apply bg-muted/50;
}

.table-row {
  @apply border-b transition-colors hover:bg-muted/50;
}

.table-row[data-state="selected"] {
  @apply bg-muted;
}

.table-cell {
  @apply p-4 align-middle;
}

.amount-positive {
  @apply text-green-600 font-medium;
}

.amount-negative {
  @apply text-red-600 font-medium;
}

.status-completed {
  @apply bg-green-100 text-green-800;
}

.status-pending {
  @apply bg-yellow-100 text-yellow-800;
}

.status-failed {
  @apply bg-red-100 text-red-800;
}
```

### Enhanced Amount Cell with Styling

```typescript
{
  accessorKey: "amount",
  header: "Amount",
  cell: ({ row }) => {
    const amount = row.getValue("amount") as number;
    const isPositive = amount > 0;
    
    return (
      <div className={`text-right font-medium ${
        isPositive ? "amount-positive" : "amount-negative"
      }`}>
        {isPositive ? "+" : ""}{formatCurrency(Math.abs(amount))}
      </div>
    );
  },
}
```

## Adding Loading State

### Loading Skeleton Component

```typescript
import { Skeleton } from "@/components/ui/skeleton";

export function BasicTableLoading({ rows = 5, columns = 6 }) {
  return (
    <div className="rounded-md border">
      <Table>
        <TableHeader>
          <TableRow>
            {Array.from({ length: columns }).map((_, index) => (
              <TableHead key={index}>
                <Skeleton className="h-4 w-full" />
              </TableHead>
            ))}
          </TableRow>
        </TableHeader>
        <TableBody>
          {Array.from({ length: rows }).map((_, rowIndex) => (
            <TableRow key={rowIndex}>
              {Array.from({ length: columns }).map((_, cellIndex) => (
                <TableCell key={cellIndex}>
                  <Skeleton className="h-4 w-full" />
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

### Usage with Loading State

```typescript
export default function TransactionsWithLoading() {
  const [data, setData] = useState<Transaction[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchTransactions()
      .then(setData)
      .finally(() => setLoading(false));
  }, []);

  return (
    <div className="container mx-auto py-10">
      <h1 className="text-2xl font-bold mb-4">Transactions</h1>
      {loading ? (
        <BasicTableLoading />
      ) : (
        <BasicDataTable data={data} columns={basicColumns} />
      )}
    </div>
  );
}
```

## Error Handling

### Error State Component

```typescript
import { AlertCircle } from "lucide-react";
import { Button } from "@/components/ui/button";

interface ErrorStateProps {
  error: Error;
  onRetry: () => void;
}

export function TableErrorState({ error, onRetry }: ErrorStateProps) {
  return (
    <div className="rounded-md border border-red-200 bg-red-50 p-8">
      <div className="flex flex-col items-center text-center">
        <AlertCircle className="h-12 w-12 text-red-500 mb-4" />
        <h3 className="text-lg font-semibold text-red-900 mb-2">
          Failed to load data
        </h3>
        <p className="text-red-700 mb-4">
          {error.message || "Something went wrong while loading the table data."}
        </p>
        <Button onClick={onRetry} variant="outline">
          Try Again
        </Button>
      </div>
    </div>
  );
}
```

### Complete Example with Error Handling

```typescript
export default function RobustTransactionsPage() {
  const [data, setData] = useState<Transaction[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const transactions = await fetchTransactions();
      setData(transactions);
    } catch (err) {
      setError(err instanceof Error ? err : new Error("Unknown error"));
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return (
    <div className="container mx-auto py-10">
      <h1 className="text-2xl font-bold mb-4">Transactions</h1>
      
      {loading && <BasicTableLoading />}
      
      {error && (
        <TableErrorState 
          error={error} 
          onRetry={fetchData} 
        />
      )}
      
      {!loading && !error && (
        <BasicDataTable data={data} columns={basicColumns} />
      )}
    </div>
  );
}
```

## Next Steps

Once you have a basic table working, you can enhance it with:

1. **[Custom Column Implementation](./custom-column-implementation.md)** - Add specialized cell types and formatting
2. **[Filter Integration Examples](./filter-integration-examples.md)** - Add filtering and search capabilities
3. **[Advanced Features](../implementation-guide.md)** - Implement infinite scrolling, sorting, and persistence

## Common Pitfalls

### 1. Missing Key Props
```typescript
// ❌ Wrong: Missing key prop
{rows.map((row) => <TableRow>...</TableRow>)}

// ✅ Correct: Always include key
{rows.map((row) => <TableRow key={row.id}>...</TableRow>)}
```

### 2. Incorrect Data Typing
```typescript
// ❌ Wrong: Generic any type
const columns: ColumnDef<any>[] = [];

// ✅ Correct: Proper typing
const columns: ColumnDef<Transaction>[] = [];
```

### 3. Missing Error Boundaries
```typescript
// ✅ Always wrap tables in error boundaries
<ErrorBoundary fallback={<TableErrorState />}>
  <BasicDataTable data={data} columns={columns} />
</ErrorBoundary>
```

This basic setup provides a solid foundation for building more complex data tables while maintaining good performance and user experience.