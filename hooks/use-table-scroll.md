# use-table-scroll Hook Documentation

> **Reference Implementation**: [apps/dashboard/src/hooks/use-table-scroll.ts](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/use-table-scroll.ts)

## Overview
The `useTableScroll` hook provides advanced horizontal scrolling functionality for data tables with support for keyboard navigation, column-width-based scrolling, and smooth scroll animations. It's specifically designed for wide tables that require horizontal navigation.

## Key Features
- **Column-based scrolling**: Navigate table columns one at a time
- **Keyboard navigation**: Arrow key support for table scrolling
- **Programmatic scroll detection**: Prevents conflicts between manual and programmatic scrolling
- **ResizeObserver integration**: Automatically updates scroll state on content changes
- **Debounced scroll handling**: Optimized performance during scroll events
- **Smooth animations**: CSS smooth scrolling for better user experience

## Interface

```typescript
interface UseTableScrollOptions {
  scrollAmount?: number;         // Fixed scroll amount for non-column mode (default: 120px)
  useColumnWidths?: boolean;     // Enable column-width-based scrolling
  startFromColumn?: number;      // Starting column index (default: 0)
}
```

## Return Values

```typescript
{
  containerRef: RefObject<HTMLDivElement>;  // Ref for the scrollable container
  canScrollLeft: boolean;                   // Whether left scrolling is possible
  canScrollRight: boolean;                  // Whether right scrolling is possible
  isScrollable: boolean;                    // Whether the table is scrollable
  scrollLeft: (smooth?: boolean) => void;   // Function to scroll left
  scrollRight: (smooth?: boolean) => void;  // Function to scroll right
}
```

## Core Functionality

### Column Position Calculation
```typescript
const getColumnPositions = useCallback(() => {
  // Calculates cumulative position of each table column
  // Used for column-width-based scrolling
}, []);
```
- Queries table headers to determine column widths
- Returns array of cumulative column positions
- Essential for precise column-to-column navigation

### Column Index Synchronization
```typescript
const syncColumnIndex = useCallback(() => {
  // Synchronizes current column index with scroll position
  // Prevents conflicts during programmatic scrolling
}, [useColumnWidths, startFromColumn, getColumnPositions]);
```
- Detects current visible column based on scroll position
- Optimized algorithm for fast column detection
- Handles edge cases at table boundaries

### Scroll State Management
```typescript
const checkScrollability = useCallback(() => {
  // Updates scroll state indicators
  // Handles both column-based and fixed-amount scrolling
}, [useColumnWidths, startFromColumn, getColumnPositions, syncColumnIndex]);
```
- Determines if scrolling is possible in each direction
- Updates `canScrollLeft`, `canScrollRight`, and `isScrollable` states
- Supports both column-based and pixel-based scrolling modes

## Scrolling Modes

### Column-Width-Based Scrolling (`useColumnWidths: true`)
```typescript
// Scrolls to specific column boundaries
// Calculates exact positions for each column
// Provides precise column-to-column navigation
```
- **Benefits**: Precise column alignment, better UX for wide tables
- **Use case**: Tables with varying column widths
- **Behavior**: Snaps to column boundaries

### Fixed-Amount Scrolling (`useColumnWidths: false`)
```typescript
// Scrolls by fixed pixel amount
// Simple horizontal scrolling
// Traditional scroll behavior
```
- **Benefits**: Simple implementation, consistent scroll distance
- **Use case**: Tables with uniform column widths
- **Behavior**: Scrolls by specified `scrollAmount`

## Performance Optimizations

### Programmatic Scroll Detection
```typescript
const isScrollingProgrammatically = useRef(false);
```
- Prevents infinite loops between manual and programmatic scrolling
- Uses timeout-based flag management
- Ensures smooth user experience

### Debounced Scroll Handling
```typescript
const handleScroll = () => {
  if (isScrollingProgrammatically.current) return;
  
  if (scrollTimeoutRef.current) {
    clearTimeout(scrollTimeoutRef.current);
  }
  
  scrollTimeoutRef.current = setTimeout(() => {
    checkScrollability();
  }, 100);
};
```
- Reduces excessive calculations during scroll events
- 100ms debounce prevents performance issues
- Ignores events during programmatic scrolling

### ResizeObserver Integration
```typescript
const resizeObserver = new ResizeObserver(() => {
  currentColumnIndex.current = startFromColumn;
  checkScrollability();
});
```
- Automatically updates on table content changes
- Resets column index on resize
- Handles dynamic content scenarios

## Keyboard Navigation

### Hotkey Integration
```typescript
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
  },
);
```
- **Enabled only when table is scrollable**
- **Prevents default browser behavior**
- **Respects scroll boundaries**

## Edge Case Handling

### Boundary Detection
```typescript
// Fast edge case detection
if (currentScrollLeft <= 10) {
  currentColumnIndex.current = startFromColumn;
  return;
}
if (currentScrollLeft >= maxScrollLeft - 10) {
  currentColumnIndex.current = allColumnPositions.length - 1;
  return;
}
```
- 10px tolerance for boundary detection
- Handles rounding errors in scroll calculations
- Prevents index out-of-bounds issues

### Smooth Scroll Coordination
```typescript
setTimeout(() => {
  isScrollingProgrammatically.current = false;
  syncColumnIndex();
  checkScrollability();
}, 500);
```
- 500ms timeout matches CSS smooth scroll duration
- Coordinates state updates with animation completion
- Prevents premature state updates

## Usage Example

```typescript
function DataTable() {
  const {
    containerRef,
    canScrollLeft,
    canScrollRight,
    isScrollable,
    scrollLeft,
    scrollRight,
  } = useTableScroll({
    useColumnWidths: true,
    startFromColumn: 1, // Skip first sticky column
  });

  return (
    <div className="relative">
      <div ref={containerRef} className="overflow-x-auto">
        <table>
          {/* Table content */}
        </table>
      </div>
      
      {isScrollable && (
        <>
          <button 
            onClick={() => scrollLeft()} 
            disabled={!canScrollLeft}
          >
            ←
          </button>
          <button 
            onClick={() => scrollRight()} 
            disabled={!canScrollRight}
          >
            →
          </button>
        </>
      )}
    </div>
  );
}
```

## Integration with Other Hooks
- **useStickyColumns**: Coordinates with sticky column positioning
- **useHotkeys**: Provides keyboard navigation
- **useTransactionParams**: Can be combined for enhanced table functionality

## Browser Compatibility
- **ResizeObserver**: Modern browsers (polyfill available for older browsers)
- **Smooth scrolling**: CSS scroll-behavior support
- **SSR safe**: No window access during initialization