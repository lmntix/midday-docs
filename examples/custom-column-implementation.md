# Custom Column Implementation

> **Reference Repository**: [midday-ai/midday](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/columns.tsx) - Transaction table column patterns

## Overview
This guide demonstrates how to create custom columns following the exact patterns used in the transaction table implementation. All examples are based on the actual codebase.

## Actual Cell Components from Transaction Table

### 1. DateCell Implementation

```typescript
import { memo } from "react";
import { formatDate } from "@/utils/format";

const DateCell = memo(
  ({
    date,
    format,
    noSort,
  }: { date: string; format?: string | null; noSort?: boolean }) =>
    formatDate(date, format, noSort),
);

DateCell.displayName = "DateCell";
```

**Usage in Column Definition:**
```typescript
{
  accessorKey: "date",
  header: "Date",
  meta: {
    className: "md:sticky bg-background group-hover:bg-[#F2F1EF] group-hover:dark:bg-secondary z-10 border-r border-border before:absolute before:right-0 before:top-0 before:bottom-0 before:w-px before:bg-border after:absolute after:right-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-l after:from-transparent after:to-background group-hover:after:to-muted after:z-[-1]",
  },
  cell: ({ row }) => (
    <DateCell
      date={row.getValue("date")}
      format={row.original.date_format}
      noSort={true}
    />
  ),
}
```

### 2. DescriptionCell Implementation

```typescript
import { memo } from "react";
import { cn } from "@midday/ui/cn";
import { Tooltip, TooltipContent, TooltipTrigger } from "@midday/ui/tooltip";

const DescriptionCell = memo(
  ({
    name,
    description,
    status,
    categorySlug,
  }: {
    name: string;
    description?: string;
    status?: string;
    categorySlug?: string | null;
  }) => (
    <div className="flex items-center space-x-2">
      <Tooltip>
        <TooltipTrigger asChild>
          <span className={cn(categorySlug === "income" && "text-[#00C969]")}>
            <div className="flex space-x-2 items-center">
              <span className="line-clamp-1 text-ellipsis max-w-[100px] md:max-w-none">
                {name}
              </span>

              {status === "pending" && (
                <div className="flex space-x-1 items-center border rounded-md text-[10px] py-1 px-2 h-[22px] text-[#878787]">
                  <span>Pending</span>
                </div>
              )}
            </div>
          </span>
        </TooltipTrigger>

        {description && (
          <TooltipContent
            className="px-3 py-1.5 text-xs max-w-[380px]"
            side="right"
            sideOffset={10}
          >
            {description}
          </TooltipContent>
        )}
      </Tooltip>
    </div>
  ),
);

DescriptionCell.displayName = "DescriptionCell";
```

**Usage in Column Definition:**
```typescript
{
  accessorKey: "description",
  header: "Description",
  meta: {
    className: "md:sticky bg-background group-hover:bg-[#F2F1EF] group-hover:dark:bg-secondary z-10 border-r border-border before:absolute before:right-0 before:top-0 before:bottom-0 before:w-px before:bg-border after:absolute after:right-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-l after:from-transparent after:to-background group-hover:after:to-muted after:z-[-1]",
  },
  cell: ({ row }) => (
    <DescriptionCell
      name={row.original.name}
      description={row.original.description}
      status={row.original.status}
      categorySlug={row.original.category?.slug}
    />
  ),
}
```

### 3. AmountCell Implementation

```typescript
import { memo } from "react";
import { cn } from "@midday/ui/cn";
import { FormatAmount } from "@/components/format-amount";

const AmountCell = memo(
  ({
    amount,
    currency,
    categorySlug,
  }: {
    amount: number;
    currency: string;
    categorySlug?: string | null;
  }) => (
    <span
      className={cn("text-sm", categorySlug === "income" && "text-[#00C969]")}
    >
      <FormatAmount amount={amount} currency={currency} />
    </span>
  ),
);

AmountCell.displayName = "AmountCell";
```

**Usage in Column Definition:**
```typescript
{
  accessorKey: "amount",
  header: "Amount",
  cell: ({ row }) => (
    <AmountCell
      amount={row.original.amount}
      currency={row.original.currency}
      categorySlug={row.original.category?.slug}
    />
  ),
}
```

### 4. TagsCell Implementation

```typescript
import { memo } from "react";
import { Badge } from "@midday/ui/badge";

const TagsCell = memo(
  ({ tags }: { tags?: { id: string; name: string | null }[] }) => (
    <div className="relative w-full">
      <div className="flex items-center space-x-2 overflow-x-auto scrollbar-hide">
        {tags?.map(({ id, name }) => (
          <Badge
            key={id}
            variant="tag-rounded"
            className="whitespace-nowrap flex-shrink-0"
          >
            {name}
          </Badge>
        ))}
      </div>
      <div className="group-hover:hidden right-0 top-0 bottom-0 w-8 bg-gradient-to-l from-background to-transparent pointer-events-none z-10" />
    </div>
  ),
);

TagsCell.displayName = "TagsCell";
```

**Usage in Column Definition:**
```typescript
{
  accessorKey: "tags",
  header: "Tags",
  cell: ({ row }) => <TagsCell tags={row.original.tags} />,
}
```

### 5. ActionsCell Implementation

```typescript
import { memo, useCallback } from "react";
import { Button } from "@midday/ui/button";
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from "@midday/ui/dropdown-menu";
import { Icons } from "@midday/ui/icons";

const ActionsCell = memo(
  ({
    transaction,
    onViewDetails,
    onCopyUrl,
    onUpdateTransaction,
    onDeleteTransaction,
  }: {
    transaction: Transaction;
    onViewDetails?: (id: string) => void;
    onCopyUrl?: (id: string) => void;
    onUpdateTransaction?: (data: { id: string; status: string }) => void;
    onDeleteTransaction?: (id: string) => void;
  }) => {
    const handleViewDetails = useCallback(() => {
      onViewDetails?.(transaction.id);
    }, [transaction.id, onViewDetails]);

    const handleCopyUrl = useCallback(() => {
      onCopyUrl?.(transaction.id);
    }, [transaction.id, onCopyUrl]);

    const handleUpdateToPosted = useCallback(() => {
      onUpdateTransaction?.({ id: transaction.id, status: "posted" });
    }, [transaction.id, onUpdateTransaction]);

    const handleUpdateToCompleted = useCallback(() => {
      onUpdateTransaction?.({ id: transaction.id, status: "completed" });
    }, [transaction.id, onUpdateTransaction]);

    const handleUpdateToExcluded = useCallback(() => {
      onUpdateTransaction?.({ id: transaction.id, status: "excluded" });
    }, [transaction.id, onUpdateTransaction]);

    const handleDeleteTransaction = useCallback(() => {
      onDeleteTransaction?.(transaction.id);
    }, [transaction.id, onDeleteTransaction]);

    return (
      <DropdownMenu>
        <DropdownMenuTrigger asChild>
          <Button variant="ghost" className="h-8 w-8 p-0">
            <span className="sr-only">Open menu</span>
            <Icons.MoreHoriz />
          </Button>
        </DropdownMenuTrigger>
        <DropdownMenuContent align="end">
          <DropdownMenuItem onClick={handleViewDetails}>
            View details
          </DropdownMenuItem>
          <DropdownMenuItem onClick={handleCopyUrl}>Share URL</DropdownMenuItem>
          <DropdownMenuSeparator />
          {!transaction.manual && transaction.status === "excluded" && (
            <DropdownMenuItem onClick={handleUpdateToPosted}>
              Include
            </DropdownMenuItem>
          )}

          {!transaction.isFulfilled && (
            <DropdownMenuItem onClick={handleUpdateToCompleted}>
              Mark as completed
            </DropdownMenuItem>
          )}

          {transaction.isFulfilled && transaction.status === "completed" && (
            <DropdownMenuItem onClick={handleUpdateToPosted}>
              Mark as uncompleted
            </DropdownMenuItem>
          )}

          {!transaction.manual && transaction.status !== "excluded" && (
            <DropdownMenuItem onClick={handleUpdateToExcluded}>
              Exclude
            </DropdownMenuItem>
          )}

          {transaction.manual && (
            <DropdownMenuItem
              className="text-destructive"
              onClick={handleDeleteTransaction}
            >
              Delete
            </DropdownMenuItem>
          )}
        </DropdownMenuContent>
      </DropdownMenu>
    );
  },
);

ActionsCell.displayName = "ActionsCell";
```

**Usage in Column Definition:**
```typescript
{
  id: "actions",
  enableSorting: false,
  enableHiding: false,
  meta: {
    className:
      "text-right md:sticky md:right-0 bg-background group-hover:bg-[#F2F1EF] group-hover:dark:bg-secondary z-10 before:absolute before:left-0 before:top-0 before:bottom-0 before:w-px before:bg-border after:absolute after:left-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-r after:from-transparent after:to-background group-hover:after:to-muted after:z-[-1]",
  },
  cell: ({ row, table }) => {
    const meta = table.options.meta;

    return (
      <ActionsCell
        transaction={row.original}
        onViewDetails={meta?.setOpen}
        onCopyUrl={meta?.copyUrl}
        onUpdateTransaction={meta?.updateTransaction}
        onDeleteTransaction={meta?.onDeleteTransaction}
      />
    );
  },
}
```

## Complete Column Definition Pattern

Here's how the actual transaction table defines its columns:

```typescript
import type { ColumnDef } from "@tanstack/react-table";
import type { RouterOutputs } from "@api/trpc/routers/_app";

type Transaction = RouterOutputs["transactions"]["get"]["data"][number];

export const columns: ColumnDef<Transaction>[] = [
  {
    id: "select",
    meta: {
      className:
        "md:sticky bg-background group-hover:bg-[#F2F1EF] group-hover:dark:bg-secondary z-10 border-r border-border before:absolute before:right-0 before:top-0 before:bottom-0 before:w-px before:bg-border after:absolute after:right-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-l after:from-transparent after:to-background group-hover:after:to-muted after:z-[-1]",
    },
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
    accessorKey: "date",
    header: "Date",
    meta: {
      className:
        "md:sticky bg-background group-hover:bg-[#F2F1EF] group-hover:dark:bg-secondary z-10 border-r border-border before:absolute before:right-0 before:top-0 before:bottom-0 before:w-px before:bg-border after:absolute after:right-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-l after:from-transparent after:to-background group-hover:after:to-muted after:z-[-1]",
    },
    cell: ({ row }) => (
      <DateCell
        date={row.getValue("date")}
        format={row.original.date_format}
        noSort={true}
      />
    ),
  },
  {
    accessorKey: "description",
    header: "Description",
    meta: {
      className:
        "md:sticky bg-background group-hover:bg-[#F2F1EF] group-hover:dark:bg-secondary z-10 border-r border-border before:absolute before:right-0 before:top-0 before:bottom-0 before:w-px before:bg-border after:absolute after:right-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-l after:from-transparent after:to-background group-hover:after:to-muted after:z-[-1]",
    },
    cell: ({ row }) => (
      <DescriptionCell
        name={row.original.name}
        description={row.original.description}
        status={row.original.status}
        categorySlug={row.original.category?.slug}
      />
    ),
  },
  {
    accessorKey: "amount",
    header: "Amount",
    cell: ({ row }) => (
      <AmountCell
        amount={row.original.amount}
        currency={row.original.currency}
        categorySlug={row.original.category?.slug}
      />
    ),
  },
  {
    accessorKey: "category",
    header: "Category",
    cell: ({ row }) => (
      <Category
        category={row.original.category}
        categoryId={row.original.id}
      />
    ),
  },
  {
    accessorKey: "tags",
    header: "Tags",
    cell: ({ row }) => <TagsCell tags={row.original.tags} />,
  },
  {
    accessorKey: "bank_account",
    header: "Account",
    cell: ({ row }) => (
      <TransactionBankAccount
        name={row.original?.account?.name ?? undefined}
        logoUrl={row.original?.account?.connection?.logoUrl ?? undefined}
      />
    ),
  },
  {
    accessorKey: "method",
    header: "Method",
    cell: ({ row }) => <TransactionMethod method={row.original.method} />,
  },
  {
    accessorKey: "assigned",
    header: "Assigned",
    cell: ({ row }) => {
      if (!row.original.assigned) {
        return null;
      }

      return (
        <AssignedUser
          fullName={row.original.assigned?.fullName}
          avatarUrl={row.original.assigned?.avatarUrl}
        />
      );
    },
  },
  {
    accessorKey: "status",
    cell: ({ row }) => {
      const fullfilled =
        row.original.status === "completed" || row.original.isFulfilled;

      return <TransactionStatus fullfilled={fullfilled} />;
    },
  },
  {
    id: "actions",
    enableSorting: false,
    enableHiding: false,
    meta: {
      className:
        "text-right md:sticky md:right-0 bg-background group-hover:bg-[#F2F1EF] group-hover:dark:bg-secondary z-10 before:absolute before:left-0 before:top-0 before:bottom-0 before:w-px before:bg-border after:absolute after:left-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-r after:from-transparent after:to-background group-hover:after:to-muted after:z-[-1]",
    },
    cell: ({ row, table }) => {
      const meta = table.options.meta;

      return (
        <ActionsCell
          transaction={row.original}
          onViewDetails={meta?.setOpen}
          onCopyUrl={meta?.copyUrl}
          onUpdateTransaction={meta?.updateTransaction}
          onDeleteTransaction={meta?.onDeleteTransaction}
        />
      );
    },
  },
];
```

## Key Patterns from the Actual Implementation

### 1. Memo Pattern
All cell components use `memo` with `displayName` for debugging:
```typescript
const CellComponent = memo(({ prop1, prop2 }) => (
  // Component JSX
));

CellComponent.displayName = "CellComponent";
```

### 2. Sticky Column CSS Classes
Sticky columns use complex CSS classes for positioning and gradients:
```typescript
meta: {
  className: "md:sticky bg-background group-hover:bg-[#F2F1EF] group-hover:dark:bg-secondary z-10 border-r border-border before:absolute before:right-0 before:top-0 before:bottom-0 before:w-px before:bg-border after:absolute after:right-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-l after:from-transparent after:to-background group-hover:after:to-muted after:z-[-1]",
}
```

### 3. Conditional Rendering in Cells
Many cells include conditional logic based on data properties:
```typescript
{!transaction.manual && transaction.status === "excluded" && (
  <DropdownMenuItem onClick={handleUpdateToPosted}>
    Include
  </DropdownMenuItem>
)}
```

### 4. Table Meta for Actions
Actions are passed through table meta options:
```typescript
cell: ({ row, table }) => {
  const meta = table.options.meta;
  return (
    <ActionsCell
      transaction={row.original}
      onViewDetails={meta?.setOpen}
      onCopyUrl={meta?.copyUrl}
      // ...
    />
  );
}
```

This implementation shows the actual patterns used in the transaction table, including the specific CSS classes, component structure, and data flow patterns.