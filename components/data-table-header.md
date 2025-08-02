# data-table-header.tsx

> **Reference Implementation**: [apps/dashboard/src/components/tables/transactions/data-table-header.tsx](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/data-table-header.tsx)

## Purpose

The `data-table-header.tsx` file implements the table header component for the transactions table, providing column headers with integrated sorting functionality, sticky column positioning, select-all checkbox, and horizontal pagination controls. It serves as the interactive header that allows users to sort data, select all rows, and navigate horizontally through the table columns.

## Key Features

- **Three-State Sorting Logic**: Implements ascending → descending → no sort cycle for each sortable column
- **Sticky Column Positioning**: Maintains key columns (select, date, description, actions) in view during horizontal scrolling
- **Select All Functionality**: Provides master checkbox for bulk row selection with indeterminate state support
- **Horizontal Pagination Controls**: Integrates navigation arrows for horizontal scrolling through columns
- **Dynamic Column Visibility**: Conditionally renders headers based on column visibility settings
- **CSS Custom Properties Integration**: Uses CSS variables for dynamic sticky positioning
- **Responsive Design**: Adapts column widths and sticky behavior for different screen sizes
- **Visual Sort Indicators**: Shows arrow icons to indicate current sort direction

## Dependencies

### Internal Dependencies

- `HorizontalPagination`: Component providing left/right navigation arrows for horizontal scrolling
- `useSortParams`: Hook managing sort parameters and URL synchronization
- `useStickyColumns`: Hook providing sticky column positioning logic and utilities

### External Dependencies

- `@midday/ui`: UI components (Button, Checkbox, TableHead, TableHeader, TableRow)
- `lucide-react`: Icons for sort direction indicators (ArrowDown, ArrowUp)
- `react`: Core React hooks (useCallback)

## Implementation Details

### Component Structure and Props

```typescript
interface Props {
  table?: TableInterface;
  loading?: boolean;
  tableScroll?: TableScrollState;
}

export function DataTableHeader({ table, loading, tableScroll }: Props) {
  // Hook integrations and logic
}
```

**Props Interface**:

- `table`: React Table instance providing column visibility and selection methods
- `loading`: Loading state flag for conditional rendering
- `tableScroll`: Scroll state and control methods from useTableScroll hook

### Three-State Sorting Implementation

#### Sort State Management

```typescript
const { params, setParams } = useSortParams();
const [column, value] = params.sort || [];

const createSortQuery = useCallback(
  (name: string) => {
    if (value === "asc") {
      // If currently ascending, switch to descending
      setParams({ sort: [name, "desc"] });
    } else if (value === "desc") {
      // If currently descending, clear sort
      setParams({ sort: null });
    } else {
      // If not sorted on this column, set to ascending
      setParams({ sort: [name, "asc"] });
    }
  },
  [value, setParams]
);
```

**Three-State Cycle**:

1. **No Sort** → **Ascending**: First click sets ascending sort
2. **Ascending** → **Descending**: Second click switches to descending
3. **Descending** → **No Sort**: Third click clears sort entirely

**URL Synchronization**:

- Sort state is persisted in URL parameters via `useSortParams`
- Enables shareable URLs with sort state
- Maintains sort state across page refreshes

#### Sort Button Implementation

```typescript
<Button
  className="p-0 hover:bg-transparent space-x-2"
  variant="ghost"
  onClick={() => createSortQuery("date")}
>
  <span>Date</span>
  {"date" === column && value === "asc" && <ArrowDown size={16} />}
  {"date" === column && value === "desc" && <ArrowUp size={16} />}
</Button>
```

**Visual Indicators**:

- **No Icon**: Column is not currently sorted
- **ArrowDown**: Column is sorted in ascending order
- **ArrowUp**: Column is sorted in descending order
- **Conditional Rendering**: Icons only show for the currently sorted column

### Sticky Column Positioning Logic

#### Hook Integration

```typescript
const { getStickyStyle, isVisible } = useStickyColumns({
  table,
  loading,
});
```

**Sticky Columns Hook Usage**:

- `getStickyStyle(columnId)`: Returns CSS styles for sticky positioning
- `isVisible(columnId)`: Checks if column should be rendered based on visibility settings

#### Left Sticky Columns Implementation

```typescript
<TableHead
  className={cn(
    "min-w-[50px] w-[50px] px-3 md:px-4 py-2 md:sticky md:left-[var(--stick-left)] bg-background z-10 border-r border-border",
    "before:absolute before:right-0 before:top-0 before:bottom-0 before:w-px before:bg-border",
    "after:absolute after:right-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-l after:from-transparent after:to-background after:z-[-1]",
  )}
  style={getStickyStyle("select")}
>
```

**CSS Technique Breakdown**:

- **`md:sticky md:left-[var(--stick-left)]`**: Sticky positioning using CSS custom property
- **`bg-background z-10`**: Background and z-index for proper layering
- **`border-r border-border`**: Right border for visual separation
- **Pseudo-elements for Visual Effects**:
  - `before:`: Creates a 1px border line on the right edge
  - `after:`: Creates a 24px gradient shadow extending to the right

#### Right Sticky Column (Actions)

```typescript
<TableHead
  className={cn(
    "w-[100px] md:sticky md:right-0 bg-background z-10",
    "before:absolute before:left-0 before:top-0 before:bottom-0 before:w-px before:bg-border",
    "after:absolute after:left-[-24px] after:top-0 after:bottom-0 after:w-6 after:bg-gradient-to-r after:from-transparent after:to-background after:z-[-1]",
  )}
>
```

**Right-Side Differences**:

- **`md:right-0`**: Sticks to right side instead of left
- **Reversed Gradients**: `bg-gradient-to-r` creates left-side shadow
- **Reversed Positioning**: Pseudo-elements positioned on left side

### Select All Functionality

#### Checkbox State Management

```typescript
<Checkbox
  checked={
    table?.getIsAllPageRowsSelected() ||
    (table?.getIsSomePageRowsSelected() && "indeterminate")
  }
  onCheckedChange={(value) =>
    table?.toggleAllPageRowsSelected(!!value)
  }
/>
```

**Three-State Checkbox Logic**:

- **Unchecked**: No rows selected (`getIsAllPageRowsSelected() === false`)
- **Indeterminate**: Some but not all rows selected (`getIsSomePageRowsSelected() === true`)
- **Checked**: All rows selected (`getIsAllPageRowsSelected() === true`)

**Integration with React Table**:

- Uses React Table's built-in selection methods
- `toggleAllPageRowsSelected()`: Toggles selection state for all visible rows
- Automatically handles indeterminate state based on current selection

### Horizontal Pagination Controls Integration

#### Conditional Rendering

```typescript
{tableScroll?.isScrollable && (
  <HorizontalPagination
    canScrollLeft={tableScroll.canScrollLeft}
    canScrollRight={tableScroll.canScrollRight}
    onScrollLeft={tableScroll.scrollLeft}
    onScrollRight={tableScroll.scrollRight}
    className="hidden md:flex"
  />
)}
```

**Integration Points**:

- **Positioned in Description Column**: Navigation controls placed in the widest sticky column
- **Conditional Display**: Only shows when table content is horizontally scrollable
- **Responsive Behavior**: Hidden on mobile devices (`hidden md:flex`)
- **State Integration**: Uses scroll state from `useTableScroll` hook

#### Scroll Control Methods

```typescript
interface TableScrollState {
  containerRef: React.RefObject<HTMLDivElement | null>;
  canScrollLeft: boolean;
  canScrollRight: boolean;
  isScrollable: boolean;
  scrollLeft: () => void;
  scrollRight: () => void;
}
```

**Scroll State Properties**:

- `canScrollLeft/Right`: Enable/disable navigation buttons based on scroll position
- `isScrollable`: Determines if pagination controls should be shown
- `scrollLeft/Right`: Methods to programmatically scroll the table container

### Dynamic Column Visibility

#### Conditional Column Rendering

```typescript
{isVisible("date") && (
  <TableHead /* ... date column configuration ... */>
    {/* Date column content */}
  </TableHead>
)}

{isVisible("description") && (
  <TableHead /* ... description column configuration ... */>
    {/* Description column content */}
  </TableHead>
)}
```

**Visibility Control**:

- Each column header is conditionally rendered based on `isVisible()` check
- Integrates with column visibility settings from React Table
- Maintains consistent header structure regardless of visible columns

#### Column Width Management

```typescript
// Fixed width columns
className = "w-[110px] min-w-[110px]"; // Date column
className = "w-[320px] min-w-[320px]"; // Description column
className = "w-[170px] min-w-[170px]"; // Amount column

// Flexible width columns
className = "w-[280px] max-w-[280px]"; // Tags column (max-width constraint)
className = "w-[250px]"; // Category column (no min-width)
```

**Width Strategy**:

- **Fixed Widths**: Critical columns maintain consistent sizing
- **Min-Width Constraints**: Prevent columns from becoming too narrow
- **Max-Width Constraints**: Prevent columns from expanding too much
- **Responsive Considerations**: Widths optimized for different screen sizes

### Event Handling Patterns

#### Memoized Sort Handlers

```typescript
const createSortQuery = useCallback(
  (name: string) => {
    // Sort logic implementation
  },
  [value, setParams],
);

// Usage in column headers
onClick={() => createSortQuery("date")}
onClick={() => createSortQuery("amount")}
onClick={() => createSortQuery("category")}
```

**Performance Optimization**:

- `useCallback` prevents unnecessary re-renders of header buttons
- Dependencies array ensures handler updates when sort state changes
- Single handler function reduces memory footprint

#### Selection Event Handling

```typescript
onCheckedChange={(value) =>
  table?.toggleAllPageRowsSelected(!!value)
}
```

**Type Safety**:

- Converts checkbox value to boolean with `!!value`
- Optional chaining (`table?.`) handles undefined table prop
- Integrates directly with React Table's selection system

## State Management

### Sort State

- **Source**: `useSortParams` hook manages sort parameters
- **Format**: `[columnName, direction]` tuple where direction is "asc" | "desc"
- **Persistence**: Automatically synced to URL parameters
- **Updates**: Handled through `setParams({ sort: [...] })` calls

### Selection State

- **Source**: React Table instance manages selection state
- **Methods**: `getIsAllPageRowsSelected()`, `getIsSomePageRowsSelected()`, `toggleAllPageRowsSelected()`
- **Scope**: Operates on currently visible page rows only
- **Integration**: Connected to global selection state via parent component

### Column Visibility State

- **Source**: React Table instance and `useStickyColumns` hook
- **Method**: `isVisible(columnId)` determines rendering
- **Dynamic**: Headers appear/disappear based on user preferences
- **Persistence**: Managed by parent DataTable component

## Performance Considerations

### Rendering Optimizations

**Conditional Rendering**:

```typescript
{isVisible("columnName") && (
  <TableHead>...</TableHead>
)}
```

- Only renders headers for visible columns
- Reduces DOM complexity and improves performance
- Maintains consistent layout regardless of column count

**Memoized Callbacks**:

```typescript
const createSortQuery = useCallback(/* ... */, [value, setParams]);
```

- Prevents unnecessary re-renders of header buttons
- Reduces memory allocation for event handlers
- Optimizes React's reconciliation process

### CSS Performance

**Sticky Positioning**:

- Uses native CSS `position: sticky` for hardware acceleration
- Leverages CSS custom properties for dynamic positioning
- Minimizes JavaScript-based positioning calculations

**Pseudo-element Shadows**:

- Uses CSS pseudo-elements instead of additional DOM elements
- Reduces DOM complexity while maintaining visual effects
- Optimizes for browser rendering performance

### State Update Efficiency

**URL Parameter Updates**:

- Sort changes update URL parameters efficiently
- Debounced updates prevent excessive URL changes
- Maintains browser history for navigation

## Integration Patterns

### Hook Composition

```typescript
const { params, setParams } = useSortParams();
const { getStickyStyle, isVisible } = useStickyColumns({ table, loading });
```

**Separation of Concerns**:

- `useSortParams`: Handles sort state and URL synchronization
- `useStickyColumns`: Manages column positioning and visibility
- Component focuses on rendering and user interaction

### Parent-Child Communication

```typescript
// Props from parent DataTable component
interface Props {
  table?: TableInterface; // React Table instance
  loading?: boolean; // Loading state
  tableScroll?: TableScrollState; // Scroll controls
}
```

**Data Flow**:

- Parent provides table instance and scroll controls
- Header component handles user interactions
- State changes propagate back through hook systems

### Component Integration

```typescript
// Horizontal pagination integration
<HorizontalPagination
  canScrollLeft={tableScroll.canScrollLeft}
  canScrollRight={tableScroll.canScrollRight}
  onScrollLeft={tableScroll.scrollLeft}
  onScrollRight={tableScroll.scrollRight}
/>
```

**Seamless Integration**:

- Scroll controls embedded within header structure
- State and methods passed through from parent component
- Maintains consistent user experience

## Related Files

### Direct Dependencies

- `@/components/horizontal-pagination` - Navigation controls for horizontal scrolling
- `@/hooks/use-sort-params` - Sort parameter management and URL synchronization
- `@/hooks/use-sticky-columns` - Sticky column positioning and visibility logic

### Integration Points

- `./data-table.tsx` - Parent component that provides table instance and scroll state
- `./columns.tsx` - Column definitions that determine available sortable columns
- `./loading.tsx` - Loading state component that may affect header rendering

### Hook Dependencies

- `@/hooks/use-sort-params` - Manages sort state with URL persistence
- `@/hooks/use-sticky-columns` - Provides positioning utilities and visibility checks
- `@/hooks/use-table-scroll` - Provides scroll state and control methods (via parent)

### UI Dependencies

- `@midday/ui/button` - Sort buttons with ghost variant styling
- `@midday/ui/checkbox` - Select all checkbox with indeterminate state support
- `@midday/ui/table` - Table header components (TableHead, TableHeader, TableRow)
- `lucide-react` - Arrow icons for sort direction indicators
