# columns.tsx

> **Reference Implementation**: [apps/dashboard/src/components/tables/transactions/columns.tsx](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/columns.tsx)

## Purpose

The `columns.tsx` file defines the column configuration and cell renderers for the transactions table. It implements a comprehensive set of memoized cell components that handle different data types and interactions, including selection, date formatting, descriptions, amounts, tags, and actions. The file serves as the blueprint for how each column should be displayed, styled, and behave within the table structure.

## Key Features

- **Memoized Cell Components**: All cell renderers are memoized for optimal performance
- **Sticky Column Configuration**: Implements sticky positioning for key columns (select, date, description, actions)
- **Custom Cell Renderers**: Specialized components for different data types (dates, amounts, tags, etc.)
- **Action Dropdown Integration**: Comprehensive dropdown menu with transaction-specific actions
- **Responsive Design**: Adaptive styling and truncation for different screen sizes
- **CSS Custom Properties**: Uses advanced CSS techniques for sticky column shadows and gradients
- **Type Safety**: Full TypeScript integration with proper type definitions
- **Accessibility**: Proper ARIA labels and keyboard navigation support

## Dependencies

### Internal Dependencies

- `AssignedUser`: Component for displaying assigned user information
- `Category`: Component for displaying transaction categories with colors
- `FormatAmount`: Component for consistent currency formatting
- `TransactionBankAccount`: Component for displaying bank account information
- `TransactionMethod`: Component for displaying payment methods
- `TransactionStatus`: Component for displaying transaction status indicators
- `formatDate`: Utility function for date formatting

### External Dependencies

- `@tanstack/react-table`: Column definition types and table integration
- `@midday/ui`: UI components (Badge, Button, Checkbox, DropdownMenu, Icons, Tooltip)
- `react`: Core React hooks (memo, useCallback)

## Implementation Details

### Column Definition Structure

The columns are defined as an array of `ColumnDef<Transaction>[]` objects, each containing:

```typescript
{
  id?: string,                    // Unique identifier for non-accessor columns
  accessorKey?: string,           // Data property to access
  header: string,                 // Column header text
  meta?: {                        // Additional metadata
    className?: string,           // CSS classes for styling
  },
  cell: ({ row, table }) => JSX.Element,  // Cell renderer function
  enableSorting?: boolean,        // Whether column supports sorting
  enableHiding?: boolean,         // Whether column can be hidden
}
```

### Memoized Cell Components

#### SelectCell Component

```typescript
const SelectCell = memo(
  ({
    checked,
    onChange,
  }: { checked: boolean; onChange: (value: boolean) => void }) => (
    <Checkbox checked={checked} onCheckedChange={onChange} />
  ),
);
```

**Key Features**:

- Memoized to prevent unnecessary re-renders during selection changes
- Integrates with React Table's row selection system
- Provides consistent checkbox styling across the table

#### DateCell Component

```typescript
const DateCell = memo(
  ({
    date,
    format,
    noSort,
  }: {
    date: string;
    format?: string | null;
    noSort?: boolean;
  }) => formatDate(date, format, noSort)
);
```

**Key Features**:

- Handles flexible date formatting based on user preferences
- Supports conditional sorting indicators
- Memoized to optimize rendering performance for date columns

#### DescriptionCell Component

```typescript
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
          <TooltipContent className="px-3 py-1.5 text-xs max-w-[380px]" side="right" sideOffset={10}>
            {description}
          </TooltipContent>
        )}
      </Tooltip>
    </div>
  ),
);
```

**Key Features**:

- **Responsive Text Truncation**: Uses `line-clamp-1` and `text-ellipsis` with responsive max-widths
- **Conditional Styling**: Income transactions get green text color
- **Status Indicators**: Shows pending status with styled badge
- **Tooltip Integration**: Displays full description on hover when available
- **Accessibility**: Proper tooltip positioning and content structure

#### AmountCell Component

```typescript
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
```

**Key Features**:

- **Consistent Currency Formatting**: Uses shared `FormatAmount` component
- **Visual Income Distinction**: Income amounts displayed in green
- **Memoization**: Prevents re-renders when other row data changes

#### TagsCell Component

```typescript
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
```

**Key Features**:

- **Horizontal Scrolling**: Handles overflow with hidden scrollbar
- **Gradient Fade Effect**: Right-side gradient that hides on row hover
- **Flexible Layout**: Tags don't wrap, maintaining consistent row height
- **Performance**: Memoized to prevent re-rendering when tags don't change

#### ActionsCell Component

```typescript
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
    // Memoized callback handlers
    const handleViewDetails = useCallback(() => {
      onViewDetails?.(transaction.id);
    }, [transaction.id, onViewDetails]);

    // ... other handlers

    return (
      <DropdownMenu>
        <DropdownMenuTrigger asChild>
          <Button variant="ghost" className="h-8 w-8 p-0">
            <span className="sr-only">Open menu</span>
            <Icons.MoreHoriz />
          </Button>
        </DropdownMenuTrigger>
        <DropdownMenuContent align="end">
          {/* Conditional menu items based on transaction state */}
        </DropdownMenuContent>
      </DropdownMenu>
    );
  },
);
```

**Key Features**:

- **Memoized Callbacks**: All event handlers are memoized with `useCallback`
- **Conditional Menu Items**: Menu options change based on transaction properties
- **Business Logic Integration**: Handles different transaction states (manual, excluded, completed)
- **Accessibility**: Proper screen reader labels and keyboard navigation

### Sticky Column Configuration

#### Sticky Column CSS Classes

```typescript
meta: {
  className:
    "md:sticky bg-background group-hover:bg-[#F2F1EF] group-hover:dark:bg-secondary z-10 border-r border-border before:absolute before:right-0 before:top-0 before:bottom-0 before:w-px before:bg-border after:absolute after:right-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-l after:from-transparent after:to-background group-hover:after:to-muted after:z-[-1]",
},
```

**CSS Technique Breakdown**:

- **`md:sticky`**: Sticky positioning only on medium screens and up
- **Background Colors**: Maintains background consistency during hover states
- **Z-Index Management**: `z-10` ensures sticky columns stay above scrolling content
- **Border Management**: Right border for visual separation
- **Pseudo-elements for Shadows**:
  - `before:`: Creates a 1px border line
  - `after:`: Creates a gradient shadow effect extending 24px to the right
- **Hover State Handling**: Background colors change on row hover

#### Left Sticky Columns (Select, Date, Description)

These columns use identical sticky styling and remain fixed during horizontal scrolling:

```typescript
{
  id: "select",
  meta: { className: "md:sticky bg-background ... z-10 border-r ..." },
  // ...
},
{
  accessorKey: "date",
  meta: { className: "md:sticky bg-background ... z-10 border-r ..." },
  // ...
},
{
  accessorKey: "description",
  meta: { className: "md:sticky bg-background ... z-10 border-r ..." },
  // ...
}
```

#### Right Sticky Column (Actions)

The actions column uses right-side sticky positioning:

```typescript
{
  id: "actions",
  meta: {
    className:
      "text-right md:sticky md:right-0 bg-background group-hover:bg-[#F2F1EF] group-hover:dark:bg-secondary z-10 before:absolute before:left-0 before:top-0 before:bottom-0 before:w-px before:bg-border after:absolute after:left-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-r after:from-transparent after:to-background group-hover:after:to-muted after:z-[-1]",
  },
}
```

**Right Sticky Differences**:

- **`md:right-0`**: Sticks to right side instead of left
- **`text-right`**: Right-aligns content
- **Reversed Gradients**: `bg-gradient-to-r` creates left-side shadow
- **Reversed Positioning**: `before:left-0` and `after:left-[-24px]`

### Column Definitions and Data Access

#### Standard Data Columns

```typescript
{
  accessorKey: "amount",
  header: "Amount",
  meta: { className: "border-l border-border" },
  cell: ({ row }) => (
    <AmountCell
      amount={row.original.amount}
      currency={row.original.currency}
      categorySlug={row.original?.category?.slug}
    />
  ),
}
```

**Pattern**:

- **`accessorKey`**: Direct property access from transaction data
- **`header`**: Display name for column header
- **`cell`**: Custom renderer with access to full row data via `row.original`

#### Nested Data Access

```typescript
{
  accessorKey: "category",
  header: "Category",
  cell: ({ row }) => (
    <Category
      name={row.original?.category?.name ?? ""}
      color={row.original?.category?.color ?? ""}
    />
  ),
}
```

**Safe Navigation**:

- Uses optional chaining (`?.`) for nested properties
- Provides fallback values with nullish coalescing (`??`)
- Handles undefined/null data gracefully

#### Complex Data Transformations

```typescript
{
  accessorKey: "status",
  cell: ({ row }) => {
    const fullfilled =
      row.original.status === "completed" || row.original.isFulfilled;

    return <TransactionStatus fullfilled={fullfilled} />;
  },
}
```

**Business Logic in Cells**:

- Combines multiple data properties for derived state
- Implements business rules within cell renderers
- Maintains separation between data and presentation logic

### Action Dropdown Implementation

#### Conditional Menu Items

The actions dropdown shows different options based on transaction state:

```typescript
// Include/Exclude Logic
{!transaction.manual && transaction.status === "excluded" && (
  <DropdownMenuItem onClick={handleUpdateToPosted}>
    Include
  </DropdownMenuItem>
)}

{!transaction.manual && transaction.status !== "excluded" && (
  <DropdownMenuItem onClick={handleUpdateToExcluded}>
    Exclude
  </DropdownMenuItem>
)}

// Completion Status Logic
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

// Delete Logic (Manual Transactions Only)
{transaction.manual && (
  <DropdownMenuItem
    className="text-destructive"
    onClick={handleDeleteTransaction}
  >
    Delete
  </DropdownMenuItem>
)}
```

**Business Rules**:

- **Manual vs Automatic**: Manual transactions can be deleted, automatic ones can be excluded
- **Status Transitions**: Different options based on current status and fulfillment state
- **Visual Hierarchy**: Destructive actions (delete) use warning colors

#### Callback Integration with Table Meta

```typescript
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
}
```

**Meta System Usage**:

- Accesses parent component callbacks via `table.options.meta`
- Enables cell components to trigger parent-level actions
- Maintains loose coupling between cell and parent components

## Performance Considerations

### Memoization Strategy

**Component-Level Memoization**:

```typescript
const SelectCell = memo(({ checked, onChange }) => (
  <Checkbox checked={checked} onCheckedChange={onChange} />
));
```

**Benefits**:

- Prevents re-rendering when other row data changes
- Reduces computational overhead for complex cell renderers
- Maintains React's reconciliation efficiency

**Callback Memoization**:

```typescript
const handleViewDetails = useCallback(() => {
  onViewDetails?.(transaction.id);
}, [transaction.id, onViewDetails]);
```

**Benefits**:

- Prevents child component re-renders due to callback reference changes
- Optimizes dropdown menu performance
- Reduces memory allocation for event handlers

### CSS Performance Optimizations

**Sticky Column Implementation**:

- Uses CSS `position: sticky` instead of JavaScript-based solutions
- Leverages GPU acceleration for smooth scrolling
- Minimizes layout thrashing with proper z-index management

**Gradient Shadows**:

- Uses CSS pseudo-elements instead of additional DOM elements
- Reduces DOM complexity while maintaining visual effects
- Optimizes for browser rendering performance

### Data Access Patterns

**Direct Property Access**:

```typescript
accessorKey: "amount"; // Direct access, no computation
```

**Computed Properties**:

```typescript
cell: ({ row }) => {
  const fullfilled = row.original.status === "completed" || row.original.isFulfilled;
  return <TransactionStatus fullfilled={fullfilled} />;
}
```

**Performance Impact**:

- Direct access is fastest for simple data display
- Computed properties are memoized at the component level
- Complex transformations are isolated to specific cells

## Integration Patterns

### Table Meta Communication

The columns integrate with the parent DataTable component through the meta system:

```typescript
// In DataTable component
const table = useReactTable({
  // ... other config
  meta: {
    setOpen: (id: string) => setParams({ transactionId: id }),
    copyUrl: (id: string) => { /* clipboard logic */ },
    updateTransaction: (data) => updateTransactionMutation.mutate(data),
    onDeleteTransaction: (id: string) => deleteTransactionMutation.mutate([id]),
  },
});

// In columns definition
cell: ({ row, table }) => {
  const meta = table.options.meta;
  return <ActionsCell onViewDetails={meta?.setOpen} /* ... */ />;
}
```

### Component Composition Pattern

Each cell renderer is a focused, single-responsibility component:

```typescript
// Specialized components for different data types
<DateCell date={row.original.date} format={table.options.meta?.dateFormat} />
<AmountCell amount={row.original.amount} currency={row.original.currency} />
<TagsCell tags={row.original.tags} />
```

### Responsive Design Integration

```typescript
// Mobile-first responsive classes
className = "line-clamp-1 text-ellipsis max-w-[100px] md:max-w-none";
className = "md:sticky bg-background"; // Sticky only on larger screens
```

## Related Files

### Direct Dependencies

- `@/components/assigned-user` - User assignment display component
- `@/components/category` - Category display with color coding
- `@/components/format-amount` - Currency formatting component
- `@/components/transaction-bank-account` - Bank account display component
- `@/components/transaction-method` - Payment method display component
- `@/components/transaction-status` - Status indicator component
- `@/utils/format` - Date formatting utilities

### Integration Points

- `./data-table.tsx` - Main table component that uses these column definitions
- `./data-table-header.tsx` - Header component that renders column headers
- `@/hooks/use-sticky-columns` - Hook that provides sticky column utilities (used by parent)

### Type Dependencies

- `@api/trpc/routers/_app` - Transaction type definitions from API router
- `@tanstack/react-table` - Column definition types and table integration types
