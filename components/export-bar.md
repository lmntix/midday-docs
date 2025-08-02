# export-bar.tsx

> **Reference Implementation**: [apps/dashboard/src/components/tables/transactions/export-bar.tsx](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/export-bar.tsx)

## Purpose

The `export-bar.tsx` file implements a floating action bar that appears when users select one or more transactions in the table. It provides bulk export functionality with animated selection count display, deselect all capability, and integration with server-side export processing. The component serves as the primary interface for bulk operations on selected transactions.

## Key Features

- **Animated Selection Count**: Uses NumberFlow for smooth count transitions when selection changes
- **Bulk Export Functionality**: Integrates with server actions for CSV/Excel export processing
- **Selection Management**: Provides deselect all functionality for easy selection clearing
- **Loading States**: Shows loading indicators during export processing
- **Error Handling**: Displays toast notifications for export failures
- **Responsive Design**: Adapts to different screen sizes with appropriate positioning
- **Smooth Animations**: Uses Framer Motion for elegant show/hide transitions
- **User Preference Integration**: Respects user's date format and locale preferences for exports

## Dependencies

### Internal Dependencies

- `exportTransactionsAction`: Server action for processing transaction exports
- `useUserQuery`: Hook providing user preferences (date format, locale)
- `useExportStore`: Zustand store for managing export state and progress
- `useTransactionsStore`: Zustand store for managing row selection state

### External Dependencies

- `@midday/ui`: UI components (Button, Icons, SubmitButton, useToast)
- `@number-flow/react`: Animated number component for selection count
- `framer-motion`: Animation library for smooth transitions
- `next-safe-action/hooks`: Hook for server action integration with loading states

## Implementation Details

### Component Structure and State Management

```typescript
export function ExportBar() {
  const { toast } = useToast();
  const { setExportData } = useExportStore();
  const { rowSelection, setRowSelection } = useTransactionsStore();
  const [isOpen, setOpen] = useState(false);
  const { data: user } = useUserQuery();

  const ids = Object.keys(rowSelection);
  const totalSelected = ids.length;
```

**State Dependencies**:

- **Global Selection State**: `rowSelection` from `useTransactionsStore` provides selected transaction IDs
- **Export State**: `setExportData` from `useExportStore` manages export progress tracking
- **Local UI State**: `isOpen` controls the visibility animation of the export bar
- **User Preferences**: `user` data provides locale and date format preferences

### Selection Count Display with NumberFlow

#### Animated Count Implementation

```typescript
<span className="text-sm text-[#878787]">
  <NumberFlow value={Object.keys(rowSelection).length} /> selected
</span>
```

**NumberFlow Integration**:

- **Smooth Transitions**: Automatically animates between different count values
- **Performance**: Efficiently handles rapid selection changes
- **Visual Feedback**: Provides immediate visual confirmation of selection changes
- **Accessibility**: Maintains semantic meaning while adding visual enhancement

#### Selection State Calculation

```typescript
const ids = Object.keys(rowSelection);
const totalSelected = ids.length;
```

**Efficient Calculation**:

- **Object Keys**: Extracts transaction IDs from selection object
- **Count Derivation**: Uses array length for total count
- **Memoization**: Recalculates only when `rowSelection` changes

### Export Action Integration

#### Server Action Setup

```typescript
const { execute, status } = useAction(exportTransactionsAction, {
  onSuccess: ({ data }) => {
    if (data?.id && data?.publicAccessToken) {
      setExportData({
        runId: data.id,
        accessToken: data.publicAccessToken,
      });

      setRowSelection(() => ({}));
    }

    setOpen(false);
  },
  onError: () => {
    toast({
      duration: 3500,
      variant: "error",
      title: "Something went wrong please try again.",
    });
  },
});
```

**Action Handling**:

- **Success Flow**: Stores export job ID and access token for progress tracking
- **Selection Clearing**: Automatically clears selection after successful export initiation
- **UI State**: Closes the export bar after successful export
- **Error Handling**: Shows user-friendly error messages via toast notifications

#### Export Execution

```typescript
<SubmitButton
  isSubmitting={status === "executing"}
  onClick={() =>
    execute({
      transactionIds: ids,
      dateFormat: user?.dateFormat ?? undefined,
      locale: user?.locale ?? undefined,
    })
  }
>
```

**Export Parameters**:

- **Transaction IDs**: Passes selected transaction IDs to server action
- **User Preferences**: Includes user's preferred date format and locale
- **Loading State**: Button shows loading indicator during export processing
- **Type Safety**: Uses proper TypeScript types for action parameters

### Animation Implementation

#### Visibility Animation

```typescript
<AnimatePresence>
  <motion.div
    className="h-12 fixed left-[50%] bottom-2 w-[400px] -ml-[200px]"
    animate={{ y: isOpen ? 0 : 100 }}
    initial={{ y: 100 }}
  >
```

**Animation Configuration**:

- **Initial State**: `y: 100` starts the bar below the viewport
- **Open State**: `y: 0` slides the bar into view
- **Closed State**: `y: 100` slides the bar back down
- **AnimatePresence**: Handles exit animations when component unmounts

#### Positioning Strategy

```typescript
className = "h-12 fixed left-[50%] bottom-2 w-[400px] -ml-[200px]";
```

**Centering Technique**:

- **Fixed Positioning**: `fixed` ensures bar stays in place during scrolling
- **Horizontal Centering**: `left-[50%] -ml-[200px]` centers 400px wide bar
- **Bottom Positioning**: `bottom-2` provides small gap from screen bottom
- **Fixed Dimensions**: `h-12 w-[400px]` ensures consistent size

### Visibility Control Logic

#### Automatic Show/Hide

```typescript
useEffect(() => {
  if (totalSelected) {
    setOpen(true);
  } else {
    setOpen(false);
  }
}, [totalSelected]);
```

**Visibility Rules**:

- **Show Condition**: Bar appears when any transactions are selected
- **Hide Condition**: Bar disappears when no transactions are selected
- **Automatic**: No manual control needed, responds to selection changes
- **Immediate**: Updates immediately when selection state changes

### Bulk Actions Implementation

#### Deselect All Functionality

```typescript
<Button variant="ghost" onClick={() => setRowSelection({})}>
  <span>Deselect all</span>
</Button>
```

**Clear Selection**:

- **Empty Object**: Sets `rowSelection` to empty object to clear all selections
- **Immediate Effect**: Instantly clears all selected rows
- **UI Update**: Triggers automatic hiding of export bar
- **Ghost Variant**: Uses subtle button styling for secondary action

#### Export Button with Loading State

```typescript
<SubmitButton
  isSubmitting={status === "executing"}
  onClick={() =>
    execute({
      transactionIds: ids,
      dateFormat: user?.dateFormat ?? undefined,
      locale: user?.locale ?? undefined,
    })
  }
>
  <div className="flex items-center space-x-2">
    <span>Export</span>
    <Icons.ArrowCoolDown className="size-4" />
  </div>
</SubmitButton>
```

**Button Features**:

- **Loading State**: `isSubmitting` prop shows loading spinner during export
- **Icon Integration**: Export icon provides visual context
- **Disabled State**: Button automatically disables during export processing
- **Consistent Styling**: Uses design system button component

### Visual Design and Styling

#### Glass Morphism Effect

```typescript
className =
  "mx-2 md:mx-0 backdrop-filter backdrop-blur-lg dark:bg-[#1A1A1A]/80 bg-[#F6F6F3]/80 h-12 justify-between items-center flex px-4 border dark:border-[#2C2C2C]";
```

**Design Elements**:

- **Backdrop Blur**: `backdrop-filter backdrop-blur-lg` creates glass effect
- **Semi-Transparent Background**: 80% opacity allows content to show through
- **Theme Support**: Different colors for light and dark themes
- **Responsive Margins**: `mx-2 md:mx-0` adds margins on mobile only
- **Border Styling**: Subtle borders that adapt to theme

#### Content Layout

```typescript
<div className="/* ... */ justify-between items-center flex px-4 /* ... */">
  <span className="text-sm text-[#878787]">
    {/* Left: Selection count */}
  </span>

  <div className="flex items-center space-x-2">
    {/* Right: Action buttons */}
  </div>
</div>
```

**Layout Structure**:

- **Space Between**: Distributes content to left and right edges
- **Vertical Centering**: Aligns all content vertically
- **Button Grouping**: Groups action buttons with consistent spacing
- **Horizontal Padding**: Provides breathing room from edges

### Error Handling and User Feedback

#### Toast Notifications

```typescript
onError: () => {
  toast({
    duration: 3500,
    variant: "error",
    title: "Something went wrong please try again.",
  });
};
```

**Error Communication**:

- **User-Friendly Messages**: Clear, non-technical error descriptions
- **Appropriate Duration**: 3.5 seconds allows users to read the message
- **Error Variant**: Uses error styling for visual distinction
- **Actionable Guidance**: Suggests trying again rather than just reporting failure

#### Loading State Feedback

```typescript
<SubmitButton
  isSubmitting={status === "executing"}
  // ...
>
```

**Loading Indicators**:

- **Button State**: Submit button shows loading spinner during processing
- **Disabled Interaction**: Prevents multiple export attempts
- **Visual Feedback**: Clear indication that action is in progress

## State Management

### Global State Integration

```typescript
const { setExportData } = useExportStore();
const { rowSelection, setRowSelection } = useTransactionsStore();
```

**Store Dependencies**:

- **Export Store**: Manages export job tracking and progress
- **Transactions Store**: Manages row selection state across components
- **Shared State**: Multiple components can access and modify selection state

### Local State

```typescript
const [isOpen, setOpen] = useState(false);
```

**UI State**:

- **Visibility Control**: Manages whether export bar is visible
- **Animation Trigger**: Controls entrance/exit animations
- **Automatic Management**: Updated via useEffect based on selection count

### Derived State

```typescript
const ids = Object.keys(rowSelection);
const totalSelected = ids.length;
```

**Computed Values**:

- **Selection IDs**: Extracted from selection object for export action
- **Count**: Derived from selection object for display
- **Reactive**: Automatically updates when selection changes

## Performance Considerations

### Efficient State Updates

```typescript
setRowSelection(() => ({})); // Clear selection
```

**Functional Updates**:

- Uses functional update pattern for state clearing
- Ensures consistent state updates across components
- Prevents stale closure issues

### Animation Performance

```typescript
animate={{ y: isOpen ? 0 : 100 }}
```

**Optimized Animations**:

- Simple transform animations use GPU acceleration
- Minimal DOM manipulation during animation
- Smooth 60fps animations with Framer Motion

### Conditional Rendering

```typescript
useEffect(() => {
  if (totalSelected) {
    setOpen(true);
  } else {
    setOpen(false);
  }
}, [totalSelected]);
```

**Efficient Updates**:

- Only updates visibility when selection count changes
- Prevents unnecessary re-renders
- Automatic cleanup when component unmounts

## Integration Patterns

### Store Integration

```typescript
// Global state access
const { rowSelection, setRowSelection } = useTransactionsStore();
const { setExportData } = useExportStore();
```

**Centralized State**:

- Selection state shared across table components
- Export state managed globally for progress tracking
- Consistent state updates across component tree

### Server Action Integration

```typescript
const { execute, status } = useAction(exportTransactionsAction, {
  onSuccess: ({ data }) => {
    // Handle successful export initiation
  },
  onError: () => {
    // Handle export errors
  },
});
```

**Type-Safe Actions**:

- Uses next-safe-action for type-safe server actions
- Automatic loading state management
- Built-in error handling with callbacks

### Parent Component Integration

The ExportBar is rendered in the main DataTable component:

```typescript
// In data-table.tsx
return (
  <div className="relative">
    {/* Table content */}
    <ExportBar />
    <AnimatePresence>{showBottomBar && <BottomBar />}</AnimatePresence>
  </div>
);
```

**Conditional Display**:

- Always rendered but controls its own visibility
- Coordinates with BottomBar for mutually exclusive display
- Integrates with table's animation system

## Related Files

### Direct Dependencies

- `@/actions/export-transactions-action` - Server action for processing exports
- `@/hooks/use-user` - User preferences for export formatting
- `@/store/export` - Export job state management
- `@/store/transactions` - Row selection state management

### Integration Points

- `./data-table.tsx` - Parent component that renders the export bar
- `./bottom-bar.tsx` - Alternative bar that appears when no rows are selected

### Store Dependencies

- `@/store/export` - Manages export job progress and results
- `@/store/transactions` - Manages global selection state

### UI Dependencies

- `@midday/ui/button` - Button components with variant support
- `@midday/ui/icons` - Export and action icons
- `@midday/ui/submit-button` - Button with loading state support
- `@midday/ui/use-toast` - Toast notification system
- `@number-flow/react` - Animated number display component
