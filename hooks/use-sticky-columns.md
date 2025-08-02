# use-sticky-columns

> **Reference Implementation**: [apps/dashboard/src/hooks/use-sticky-columns.ts](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/use-sticky-columns.ts)

## Purpose

The `use-sticky-columns` hook provides dynamic positioning calculations and CSS custom properties for implementing sticky columns in data tables. It manages the complex logic of calculating left positions for sticky columns based on column visibility, ensuring that sticky columns remain properly positioned as other columns are shown or hidden.

## Key Features

- Dynamic position calculations based on column visibility
- CSS custom properties integration for flexible styling
- Responsive behavior with mobile-first approach
- Performance optimization through memoized calculations
- Type-safe column visibility handling
- Utility functions for styling and class name generation

## Dependencies

### External Dependencies

- `@midday/ui/cn` - Class name utility for conditional styling
- `@tanstack/react-table` - Table library types for visibility state
- `react` - React hooks (useMemo) for performance optimization

### Internal Dependencies

- Used by transaction table header and cell components
- Integrates with table column visibility state
- Works alongside other table hooks for comprehensive table behavior

## Implementation Details

### Column Visibility Detection

The hook provides a flexible `isVisible` function that handles multiple visibility sources:

```typescript
const isVisible = (id: string) =>
  loading ||
  table
    ?.getAllLeafColumns()
    .find((col) => col.id === id)
    ?.getIsVisible() ||
  (columnVisibility && columnVisibility[id] !== false);
```

**Visibility Logic**:

- **Loading State**: All columns considered visible during loading
- **Table State**: Uses TanStack Table's built-in visibility methods
- **Manual Override**: Respects explicit columnVisibility prop
- **Default Behavior**: Columns are visible unless explicitly hidden

### Dynamic Position Calculations

The hook calculates sticky positions dynamically based on visible columns:

```typescript
const stickyPositions = useMemo(() => {
  let position = 0;
  const positions: Record<string, number> = {};

  // Select column (always visible)
  positions.select = position;
  position += 50; // width of select column

  // Date column
  if (isVisible("date")) {
    positions.date = position;
    position += 110; // width of date column
  }

  // Description column
  if (isVisible("description")) {
    positions.description = position;
  }

  return positions;
}, [isVisible]);
```

**Position Calculation Logic**:

1. **Cumulative Positioning**: Each column's position is the sum of previous visible columns' widths
2. **Conditional Inclusion**: Only visible columns contribute to position calculations
3. **Fixed Widths**: Uses predefined column widths for accurate positioning
4. **Memoization**: Recalculates only when visibility changes

### CSS Custom Properties Usage

The hook generates CSS custom properties for flexible styling:

```typescript
const getStickyStyle = (columnId: string) => {
  const position = stickyPositions[columnId];
  return position !== undefined
    ? ({ "--stick-left": `${position}px` } as React.CSSProperties)
    : {};
};
```

**CSS Integration**:

- **Custom Properties**: Uses `--stick-left` for dynamic positioning
- **Type Safety**: Returns properly typed React CSSProperties
- **Fallback Handling**: Returns empty object for non-sticky columns
- **Pixel Precision**: Provides exact pixel values for positioning

### Responsive Behavior Patterns

The hook implements mobile-first responsive behavior:

```typescript
const getStickyClassName = (columnId: string, baseClassName?: string) => {
  const stickyColumns = ["select", "date", "description"];
  const isSticky = stickyColumns.includes(columnId);
  return cn(baseClassName, isSticky && "md:sticky md:left-[var(--stick-left)]");
};
```

**Responsive Strategy**:

- **Mobile First**: No sticky behavior on mobile devices
- **Breakpoint Activation**: Sticky behavior starts at `md` breakpoint (768px+)
- **Progressive Enhancement**: Graceful degradation for smaller screens
- **Performance Consideration**: Avoids sticky positioning overhead on mobile

### Column Width Configuration

The hook uses predefined column widths for accurate positioning:

```typescript
// Column width definitions
const COLUMN_WIDTHS = {
  select: 50, // Checkbox column
  date: 110, // Date column
  description: 320, // Description column (variable, but positioned)
};
```

**Width Management**:

- **Fixed Widths**: Consistent sizing for predictable positioning
- **Responsive Considerations**: Widths optimized for different screen sizes
- **Layout Stability**: Prevents layout shifts during visibility changes

## Performance Considerations

- **Memoized Calculations**: Position calculations only run when visibility changes
- **Efficient Re-renders**: Minimal re-renders through optimized dependency arrays
- **CSS Custom Properties**: Leverages browser-native CSS variables for performance
- **Conditional Rendering**: Only applies sticky styles when necessary
- **Mobile Optimization**: Avoids sticky positioning overhead on mobile devices

## Integration Patterns

### With Table Header Components

```typescript
// In data-table-header.tsx
function DataTableHeader({ table, loading }) {
  const { getStickyStyle, getStickyClassName, isVisible } = useStickyColumns({
    table,
    loading,
  });

  return (
    <TableHeader>
      <TableRow>
        <TableHead
          className={getStickyClassName(
            "select",
            "min-w-[50px] w-[50px] px-3 md:px-4 py-2 bg-background z-10"
          )}
          style={getStickyStyle("select")}
        >
          <Checkbox />
        </TableHead>

        {isVisible("date") && (
          <TableHead
            className={getStickyClassName(
              "date",
              "w-[110px] min-w-[110px] px-3 md:px-4 py-2 bg-background z-10"
            )}
            style={getStickyStyle("date")}
          >
            Date
          </TableHead>
        )}
      </TableRow>
    </TableHeader>
  );
}
```

### With Table Cell Components

```typescript
// In table cell components
function TransactionRow({ transaction, table }) {
  const { getStickyStyle, getStickyClassName, isVisible } = useStickyColumns({
    table,
  });

  return (
    <TableRow>
      <TableCell
        className={getStickyClassName(
          "select",
          "px-3 md:px-4 py-2 bg-background"
        )}
        style={getStickyStyle("select")}
      >
        <Checkbox />
      </TableCell>

      {isVisible("date") && (
        <TableCell
          className={getStickyClassName(
            "date",
            "px-3 md:px-4 py-2 bg-background"
          )}
          style={getStickyStyle("date")}
        >
          {transaction.date}
        </TableCell>
      )}
    </TableRow>
  );
}
```

### With Column Visibility Controls

```typescript
// Integration with column visibility
function TableWithStickyColumns() {
  const [columnVisibility, setColumnVisibility] = useState({
    date: true,
    description: true,
    amount: true,
  });

  const { getStickyStyle, isVisible } = useStickyColumns({
    columnVisibility,
    table,
  });

  return (
    <div>
      <ColumnVisibilityControls
        visibility={columnVisibility}
        onChange={setColumnVisibility}
      />
      <Table />
    </div>
  );
}
```

## Error Handling

- **Missing Table Reference**: Gracefully handles undefined table props
- **Invalid Column IDs**: Returns empty styles for unknown columns
- **Visibility State Errors**: Defaults to visible when visibility state is corrupted
- **CSS Property Fallbacks**: Provides safe fallbacks for unsupported browsers

## Related Files

- `data-table-header.tsx` - Primary consumer for header sticky positioning
- `columns.tsx` - Uses sticky utilities for cell positioning
- `data-table.tsx` - Provides table instance and visibility state
- `use-table-scroll.tsx` - Complementary horizontal scrolling behavior

## Usage Examples

### Basic Sticky Column Setup

```typescript
function StickyTableExample() {
  const { getStickyStyle, getStickyClassName, isVisible } = useStickyColumns({
    table,
    loading: false,
  });

  return (
    <table>
      <thead>
        <tr>
          <th
            className={getStickyClassName("select", "bg-white border-r")}
            style={getStickyStyle("select")}
          >
            Select
          </th>

          {isVisible("date") && (
            <th
              className={getStickyClassName("date", "bg-white border-r")}
              style={getStickyStyle("date")}
            >
              Date
            </th>
          )}

          <th className="bg-white">Amount</th>
        </tr>
      </thead>
    </table>
  );
}
```

### Advanced Sticky Configuration

```typescript
function AdvancedStickyTable() {
  const [columnVisibility, setColumnVisibility] = useState({
    date: true,
    description: true,
    amount: true,
    tags: false,
  });

  const {
    getStickyStyle,
    getStickyClassName,
    isVisible,
    stickyPositions
  } = useStickyColumns({
    columnVisibility,
    table,
  });

  // Custom sticky column with dynamic positioning
  const getCustomStickyStyle = (columnId: string, offset = 0) => {
    const baseStyle = getStickyStyle(columnId);
    if (baseStyle["--stick-left"]) {
      const position = parseInt(baseStyle["--stick-left"]) + offset;
      return { "--stick-left": `${position}px` };
    }
    return baseStyle;
  };

  return (
    <div>
      <div>
        Sticky Positions: {JSON.stringify(stickyPositions, null, 2)}
      </div>

      <table>
        <thead>
          <tr>
            <th
              className={getStickyClassName("select")}
              style={getStickyStyle("select")}
            >
              ✓
            </th>

            {isVisible("date") && (
              <th
                className={getStickyClassName("date")}
                style={getCustomStickyStyle("date", 5)} // 5px offset
              >
                Date
              </th>
            )}
          </tr>
        </thead>
      </table>
    </div>
  );
}
```

### Responsive Sticky Behavior

```typescript
function ResponsiveStickyTable() {
  const { getStickyStyle, getStickyClassName } = useStickyColumns({
    table,
  });

  return (
    <div className="overflow-x-auto">
      <table className="min-w-full">
        <thead>
          <tr>
            {/* Mobile: no sticky, Desktop: sticky */}
            <th
              className={cn(
                "px-4 py-2 bg-gray-50",
                "md:sticky md:left-0 md:z-10", // Only sticky on md+
                getStickyClassName("select")
              )}
              style={getStickyStyle("select")}
            >
              Select
            </th>

            {/* Conditional sticky based on screen size */}
            <th
              className={cn(
                "px-4 py-2 bg-gray-50",
                "hidden md:table-cell", // Hidden on mobile
                getStickyClassName("date")
              )}
              style={getStickyStyle("date")}
            >
              Date
            </th>
          </tr>
        </thead>
      </table>
    </div>
  );
}
```

### Debug Sticky Positions

```typescript
function StickyPositionDebugger() {
  const { stickyPositions, isVisible } = useStickyColumns({
    table,
    columnVisibility,
  });

  return (
    <div className="p-4 bg-gray-100 rounded">
      <h3>Sticky Column Debug Info:</h3>

      <div>
        <strong>Visible Columns:</strong>
        <ul>
          {["select", "date", "description", "amount"].map(col => (
            <li key={col}>
              {col}: {isVisible(col) ? "✓ Visible" : "✗ Hidden"}
            </li>
          ))}
        </ul>
      </div>

      <div>
        <strong>Sticky Positions:</strong>
        <pre>{JSON.stringify(stickyPositions, null, 2)}</pre>
      </div>

      <div>
        <strong>CSS Custom Properties:</strong>
        {Object.entries(stickyPositions).map(([col, pos]) => (
          <div key={col}>
            {col}: --stick-left: {pos}px
          </div>
        ))}
      </div>
    </div>
  );
}
```
