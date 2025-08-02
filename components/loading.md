# loading.tsx

> **Reference Implementation**: [apps/dashboard/src/components/tables/transactions/loading.tsx](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/loading.tsx)

## Purpose

The `loading.tsx` file implements a skeleton loading component that provides visual feedback during data fetching and serves as a backdrop for empty states. It creates a realistic preview of the transactions table structure with animated skeleton elements that match the actual table layout, enhancing perceived performance and user experience during loading states.

## Key Features

- **Realistic Table Structure**: Mimics the exact layout and dimensions of the actual transactions table
- **Animated Skeletons**: Uses skeleton components with shimmer animations for engaging loading feedback
- **Dual Mode Support**: Functions as both loading indicator and empty state backdrop
- **Consistent Dimensions**: Matches column widths and row heights of the real table
- **Progressive Loading**: Shows 40 skeleton rows to simulate a full table view
- **Responsive Design**: Adapts to different screen sizes with appropriate styling
- **Performance Optimized**: Lightweight implementation with minimal computational overhead

## Dependencies

### Internal Dependencies

- `DataTableHeader`: Table header component rendered in loading mode

### External Dependencies

- `@midday/ui/cn`: Utility for conditional class name merging
- `@midday/ui/skeleton`: Skeleton component with shimmer animation
- `@midday/ui/table`: Table components (Table, TableBody, TableCell, TableRow)

## Implementation Details

### Component Structure and Props

```typescript
export function Loading({ isEmpty }: { isEmpty?: boolean }) {
  return (
    <div className="w-full">
      <div
        className={cn(
          "overflow-x-auto",
          !isEmpty && "md:border-l md:border-r border-border",
        )}
      >
        <Table
          className={cn(
            "min-w-[1600px]",
            isEmpty && "opacity-20 pointer-events-none blur-[7px]",
          )}
        >
          <DataTableHeader loading />
          <TableBody>
            {/* Skeleton rows */}
          </TableBody>
        </Table>
      </div>
    </div>
  );
}
```

**Props Interface**:

- `isEmpty?: boolean` - Controls whether component functions as loading indicator or empty state backdrop

### Dual Mode Implementation

#### Loading Mode (isEmpty = false or undefined)

```typescript
className={cn(
  "overflow-x-auto",
  !isEmpty && "md:border-l md:border-r border-border",
)}
```

**Loading Mode Characteristics**:

- **Full Opacity**: Table appears at normal opacity with animated skeletons
- **Interactive Appearance**: No pointer-events restrictions
- **Border Styling**: Includes left and right borders for visual consistency
- **Animation Active**: Skeleton components show shimmer animations

#### Empty State Backdrop Mode (isEmpty = true)

```typescript
className={cn(
  "min-w-[1600px]",
  isEmpty && "opacity-20 pointer-events-none blur-[7px]",
)}
```

**Empty State Characteristics**:

- **Reduced Opacity**: `opacity-20` makes table appear faded (20% opacity)
- **Disabled Interaction**: `pointer-events-none` prevents user interaction
- **Blur Effect**: `blur-[7px]` creates subtle blur for background effect
- **Static Animation**: Skeleton animations are disabled for performance

### Skeleton Row Generation

#### Data Array Creation

```typescript
const data = [...Array(40)].map((_, i) => ({ id: i.toString() }));
```

**Row Generation Strategy**:

- **Fixed Count**: Creates exactly 40 skeleton rows for consistent appearance
- **Unique Keys**: Each row gets a unique string ID for React key prop
- **Minimal Data**: Only includes necessary ID field to reduce memory usage
- **Array Spread**: Uses spread operator for clean array creation

#### Row Structure Implementation

```typescript
{data?.map((row) => (
  <TableRow key={row.id} className="h-[45px]">
    <TableCell className="w-[50px]">
      <Skeleton
        className={cn("h-3.5 w-[15px]", isEmpty && "animate-none")}
      />
    </TableCell>
    {/* Additional cells... */}
  </TableRow>
))}
```

**Row Configuration**:

- **Consistent Height**: `h-[45px]` matches actual table row height
- **Unique Keys**: Uses row ID for React reconciliation
- **Cell Structure**: Mirrors actual table cell layout and widths

### Column-Specific Skeleton Implementation

#### Select Column (Checkbox)

```typescript
<TableCell className="w-[50px]">
  <Skeleton
    className={cn("h-3.5 w-[15px]", isEmpty && "animate-none")}
  />
</TableCell>
```

**Design Rationale**:

- **Small Width**: `w-[15px]` approximates checkbox size
- **Standard Height**: `h-3.5` (14px) matches typical checkbox height
- **Fixed Width**: `w-[50px]` cell matches actual select column width

#### Date Column

```typescript
<TableCell className="w-[100px]">
  <Skeleton
    className={cn("h-3.5 w-[60%]", isEmpty && "animate-none")}
  />
</TableCell>
```

**Design Characteristics**:

- **Percentage Width**: `w-[60%]` creates realistic date text width
- **Narrow Cell**: `w-[100px]` matches actual date column width
- **Consistent Height**: Maintains uniform skeleton height across columns

#### Description Column (Widest)

```typescript
<TableCell className="w-[430px]">
  <Skeleton
    className={cn("h-3.5 w-[50%]", isEmpty && "animate-none")}
  />
</TableCell>
```

**Layout Considerations**:

- **Largest Cell**: `w-[430px]` reflects description column's prominence
- **Moderate Fill**: `w-[50%]` suggests variable-length transaction descriptions
- **Primary Content**: Represents the main content column

#### Amount Column

```typescript
<TableCell className="w-[200px]">
  <Skeleton
    className={cn("h-3.5 w-[50%]", isEmpty && "animate-none")}
  />
</TableCell>
```

**Numeric Representation**:

- **Medium Width**: `w-[200px]` accommodates currency formatting
- **Half Fill**: `w-[50%]` represents typical amount text length
- **Right-Aligned Context**: Positioned for numeric data display

#### Category Column with Icon

```typescript
<TableCell className="w-[200px]">
  <div className="flex items-center space-x-2 w-[80%]">
    <Skeleton className="h-5 w-5 rounded-full" />
    <Skeleton
      className={cn("h-3.5 w-[70%]", isEmpty && "animate-none")}
    />
  </div>
</TableCell>
```

**Complex Cell Structure**:

- **Icon Placeholder**: `h-5 w-5 rounded-full` represents category color indicator
- **Text Placeholder**: `w-[70%]` represents category name text
- **Flex Layout**: `flex items-center space-x-2` matches actual category cell layout
- **Realistic Spacing**: Mirrors the spacing used in actual category cells

#### Status Column (Circular Indicator)

```typescript
<TableCell className="w-50px">
  <Skeleton
    className={cn(
      "h-[20px] w-[20px] rounded-full",
      isEmpty && "animate-none",
    )}
  />
</TableCell>
```

**Status Indicator**:

- **Circular Shape**: `rounded-full` represents status dot/indicator
- **Fixed Size**: `h-[20px] w-[20px]` matches typical status indicator size
- **Narrow Column**: Compact width for simple status display

#### Empty Action Column

```typescript
<TableCell className="w-60px" />
```

**Minimal Placeholder**:

- **Empty Cell**: No skeleton content for actions column
- **Width Preservation**: Maintains column width for layout consistency
- **Clean Appearance**: Avoids cluttering the loading state

### Animation Control System

#### Conditional Animation

```typescript
className={cn("h-3.5 w-[15px]", isEmpty && "animate-none")}
```

**Animation States**:

- **Loading Mode**: Default skeleton shimmer animation active
- **Empty State Mode**: `animate-none` disables animations for performance
- **Consistent Pattern**: Applied to all skeleton elements uniformly

#### Performance Optimization

**Animation Disabling Benefits**:

- **Reduced CPU Usage**: No animation calculations when used as backdrop
- **Better Performance**: Especially important with 40+ skeleton elements
- **Visual Hierarchy**: Static backdrop doesn't compete with foreground content
- **Battery Efficiency**: Reduces power consumption on mobile devices

### Table Structure Integration

#### Header Integration

```typescript
<DataTableHeader loading />
```

**Header Consistency**:

- **Loading Prop**: Passes loading state to header component
- **Structural Integrity**: Maintains complete table structure during loading
- **Visual Continuity**: Header appears consistent with actual table header

#### Table Styling

```typescript
<Table
  className={cn(
    "min-w-[1600px]",
    isEmpty && "opacity-20 pointer-events-none blur-[7px]",
  )}
>
```

**Layout Preservation**:

- **Minimum Width**: `min-w-[1600px]` ensures horizontal scrolling behavior matches actual table
- **Responsive Behavior**: Maintains same responsive characteristics as real table
- **Visual Effects**: Conditional styling for different usage modes

### Responsive Design Considerations

#### Border Management

```typescript
className={cn(
  "overflow-x-auto",
  !isEmpty && "md:border-l md:border-r border-border",
)}
```

**Responsive Borders**:

- **Medium Screens Up**: `md:border-l md:border-r` adds borders on larger screens
- **Mobile Optimization**: No borders on mobile for cleaner appearance
- **Conditional Application**: Only applies borders in loading mode

#### Overflow Handling

```typescript
className = "overflow-x-auto";
```

**Scroll Behavior**:

- **Horizontal Scrolling**: Matches actual table scroll behavior
- **Content Preservation**: Maintains full table width even on narrow screens
- **Consistent UX**: Users experience same scrolling patterns during loading

## State Management

### Stateless Design

The Loading component is purely presentational with no internal state management:

```typescript
export function Loading({ isEmpty }: { isEmpty?: boolean }) {
  // No useState, useEffect, or other state hooks
  // Pure function of props
}
```

**Benefits**:

- **Predictable Rendering**: Output depends only on props
- **Performance**: No state updates or side effects
- **Simplicity**: Easy to understand and maintain
- **Reusability**: Can be used in different contexts without state conflicts

### Prop-Driven Behavior

All component behavior is controlled through the `isEmpty` prop:

```typescript
// Loading indicator usage
<Loading />

// Empty state backdrop usage
<Loading isEmpty />
```

**Clear Interface**:

- **Single Prop**: Simple boolean controls all behavioral differences
- **Default Behavior**: Undefined/false provides standard loading behavior
- **Explicit Empty State**: True value explicitly enables backdrop mode

## Performance Considerations

### Efficient Rendering

#### Static Data Generation

```typescript
const data = [...Array(40)].map((_, i) => ({ id: i.toString() }));
```

**Optimization Benefits**:

- **Module Level**: Array created once when module loads
- **No Re-computation**: Same array reference used across renders
- **Minimal Memory**: Only stores necessary ID strings
- **Fast Iteration**: Simple array structure for efficient mapping

#### Conditional Animation

```typescript
isEmpty && "animate-none";
```

**Performance Impact**:

- **Reduced CPU**: Disables 40+ skeleton animations when used as backdrop
- **Battery Savings**: Important for mobile devices
- **Smooth Interaction**: Prevents animation interference with foreground content

#### Lightweight Structure

**Minimal Complexity**:

- No complex state management
- No expensive calculations
- Simple conditional class names
- Efficient DOM structure

### Memory Efficiency

#### Component Reuse

```typescript
// Same component, different modes
<Loading />          // Loading mode
<Loading isEmpty />  // Backdrop mode
```

**Benefits**:

- **Single Component**: No duplicate code for different use cases
- **Shared Styles**: CSS classes reused across modes
- **Bundle Size**: Smaller JavaScript bundle

## Integration Patterns

### Parent Component Integration

#### Loading State Usage

```typescript
// In data-table.tsx
if (!tableData.length && !hasFilters) {
  return (
    <div className="relative h-[calc(100vh-200px)] overflow-hidden">
      <NoTransactions />
      <Loading isEmpty />
    </div>
  );
}
```

**Backdrop Pattern**:

- **Layered Rendering**: Loading component serves as background layer
- **Empty State Overlay**: Actual empty state content appears on top
- **Visual Depth**: Creates layered visual hierarchy

#### Standard Loading Usage

```typescript
// During data fetching
{isLoading && <Loading />}
```

**Loading Indicator**:

- **Full Opacity**: Component appears as primary content
- **Animated Feedback**: Skeleton animations provide loading feedback
- **Structural Preview**: Shows users what to expect

### Header Component Integration

```typescript
<DataTableHeader loading />
```

**Consistent Structure**:

- **Loading Prop**: Header adapts to loading state
- **Visual Continuity**: Header styling remains consistent
- **Complete Table**: Maintains full table structure during loading

## Related Files

### Direct Dependencies

- `./data-table-header.tsx` - Table header component rendered in loading mode

### Integration Points

- `./data-table.tsx` - Parent component that conditionally renders loading states
- `./empty-states.tsx` - Components that use Loading as backdrop (NoTransactions, NoResults)

### UI Dependencies

- `@midday/ui/skeleton` - Skeleton component with shimmer animations
- `@midday/ui/table` - Table structure components
- `@midday/ui/cn` - Utility for conditional class name merging

### Usage Patterns

- **Loading Indicator**: Primary loading state for data fetching
- **Empty State Backdrop**: Background layer for empty state components
- **Structural Preview**: Shows table layout before data loads
