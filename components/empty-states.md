# empty-states.tsx

> **Reference Implementation**: [apps/dashboard/src/components/tables/transactions/empty-states.tsx](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/empty-states.tsx)

## Purpose

The `empty-states.tsx` file provides two distinct empty state components for the transactions table: `NoResults` for when filters return no matches, and `NoTransactions` for when no transactions exist at all. These components enhance user experience by providing contextual guidance and clear calls-to-action based on the specific empty state scenario.

## Key Features

- **Contextual Empty States**: Different components for different scenarios (no results vs no data)
- **Clear Visual Hierarchy**: Consistent typography and spacing for optimal readability
- **Actionable CTAs**: Relevant call-to-action buttons for each empty state
- **Filter Integration**: Direct integration with filter clearing functionality
- **Responsive Design**: Adaptive layouts that work across different screen sizes
- **Consistent Iconography**: Uses branded icons for visual consistency
- **User-Friendly Messaging**: Clear, helpful text that guides users toward resolution

## Dependencies

### Internal Dependencies

- `AddAccountButton`: Component for initiating bank account connection process
- `useTransactionFilterParamsWithPersistence`: Hook providing filter clearing functionality

### External Dependencies

- `@midday/ui/button`: Button component with variant styling support
- `@midday/ui/icons`: Icon library providing branded transaction icons

## Implementation Details

### NoResults Component

#### Purpose and Usage

The `NoResults` component displays when the user has applied filters but no transactions match the current filter criteria.

```typescript
export function NoResults() {
  const { clearAllFilters } = useTransactionFilterParamsWithPersistence();

  return (
    <div className="h-[calc(100vh-300px)] flex items-center justify-center">
      <div className="flex flex-col items-center">
        <Icons.Transactions2 className="mb-4" />
        <div className="text-center mb-6 space-y-2">
          <h2 className="font-medium text-lg">No results</h2>
          <p className="text-[#606060] text-sm">
            Try another search, or adjusting the filters
          </p>
        </div>

        <Button variant="outline" onClick={clearAllFilters}>
          Clear filters
        </Button>
      </div>
    </div>
  );
}
```

#### Key Implementation Details

**Filter Integration**:

```typescript
const { clearAllFilters } = useTransactionFilterParamsWithPersistence();
```

- Directly integrates with the filter persistence system
- `clearAllFilters` function removes all active filters and resets the table view
- Maintains consistency with the filter state management used throughout the application

**Layout Structure**:

- **Container**: Full viewport height minus header space (`h-[calc(100vh-300px)]`)
- **Centering**: Uses flexbox for perfect vertical and horizontal centering
- **Content Flow**: Vertical flex layout for icon, text, and button stacking

**Visual Hierarchy**:

```typescript
<Icons.Transactions2 className="mb-4" />                    // Visual anchor
<h2 className="font-medium text-lg">No results</h2>         // Primary message
<p className="text-[#606060] text-sm">                      // Secondary guidance
  Try another search, or adjusting the filters
</p>
<Button variant="outline" onClick={clearAllFilters}>        // Call-to-action
  Clear filters
</Button>
```

**User Experience Considerations**:

- **Clear Problem Statement**: "No results" immediately communicates the issue
- **Helpful Guidance**: Suggests specific actions (search differently, adjust filters)
- **Direct Solution**: Provides one-click filter clearing functionality
- **Non-Destructive Action**: Uses outline button variant to indicate reversible action

### NoTransactions Component

#### Purpose and Usage

The `NoTransactions` component displays when the user has no transactions at all in their account, typically for new users who haven't connected any bank accounts.

```typescript
export function NoTransactions() {
  return (
    <div className="absolute w-full h-[calc(100vh-300px)] top-0 left-0 flex items-center justify-center z-20">
      <div className="text-center max-w-sm mx-auto flex flex-col items-center justify-center">
        <h2 className="text-xl font-medium mb-2">No transactions</h2>
        <p className="text-sm text-[#878787] mb-6">
          Connect your bank account to automatically import transactions and
          unlock powerful financial insights to help you make smarter money
          decisions.
        </p>

        <AddAccountButton />
      </div>
    </div>
  );
}
```

#### Key Implementation Details

**Positioning Strategy**:

```typescript
className =
  "absolute w-full h-[calc(100vh-300px)] top-0 left-0 flex items-center justify-center z-20";
```

- **Absolute Positioning**: Overlays the entire table area
- **Full Coverage**: `w-full h-[calc(100vh-300px)]` covers the complete table space
- **High Z-Index**: `z-20` ensures it appears above other table elements
- **Perfect Centering**: Flexbox centering for optimal visual balance

**Content Constraints**:

```typescript
className =
  "text-center max-w-sm mx-auto flex flex-col items-center justify-center";
```

- **Max Width**: `max-w-sm` (384px) prevents text from becoming too wide
- **Auto Margins**: `mx-auto` centers the content block horizontally
- **Text Alignment**: `text-center` ensures all text is center-aligned

**Typography Hierarchy**:

```typescript
<h2 className="text-xl font-medium mb-2">No transactions</h2>           // Primary heading
<p className="text-sm text-[#878787] mb-6">                            // Explanatory text
  Connect your bank account to automatically import transactions and
  unlock powerful financial insights to help you make smarter money
  decisions.
</p>
```

**Messaging Strategy**:

- **Clear State Description**: "No transactions" immediately explains the situation
- **Value Proposition**: Explains benefits of connecting accounts (automatic import, insights)
- **Motivational Language**: "unlock powerful financial insights" and "smarter money decisions"
- **Action-Oriented**: Guides user toward the primary onboarding action

### Conditional Rendering Logic

#### Usage in Parent Component

The empty states are conditionally rendered in the main `DataTable` component based on data and filter state:

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

if (!tableData.length && hasFilters) {
  return (
    <div className="relative h-[calc(100vh-200px)] overflow-hidden">
      <NoResults />
      <Loading isEmpty />
    </div>
  );
}
```

#### State Detection Patterns

**No Data State**:

- **Condition**: `!tableData.length && !hasFilters`
- **Meaning**: No transactions exist and no filters are applied
- **Component**: `NoTransactions`
- **Action**: Encourage account connection

**No Results State**:

- **Condition**: `!tableData.length && hasFilters`
- **Meaning**: Transactions exist but current filters return no matches
- **Component**: `NoResults`
- **Action**: Suggest filter adjustment or clearing

### User Experience Considerations

#### NoResults UX Design

**Problem Recognition**:

- Immediately communicates that filters are the cause of empty results
- Distinguishes between "no data" and "no matching data" scenarios

**Solution Guidance**:

- Provides two specific suggestions: "try another search" and "adjusting the filters"
- Offers immediate resolution with "Clear filters" button

**Visual Consistency**:

- Uses same icon as transaction table for brand consistency
- Maintains consistent spacing and typography with other empty states

#### NoTransactions UX Design

**Onboarding Focus**:

- Treats empty state as an onboarding opportunity
- Explains the value proposition of connecting accounts

**Motivation Building**:

- Uses aspirational language about "financial insights" and "smarter decisions"
- Positions account connection as an empowering action

**Clear Next Steps**:

- Single, prominent call-to-action button
- Removes decision paralysis by providing one clear path forward

### Responsive Design Patterns

#### Mobile Considerations

**Flexible Heights**:

```typescript
className = "h-[calc(100vh-300px)]"; // Adapts to viewport height
```

- Uses viewport-relative units for consistent appearance across devices
- Accounts for header and navigation space

**Content Width Management**:

```typescript
className = "max-w-sm mx-auto"; // Constrains content width on larger screens
```

- Prevents text from becoming too wide on desktop
- Maintains readability across all screen sizes

**Typography Scaling**:

- Uses responsive text sizes (`text-lg`, `text-xl`, `text-sm`)
- Maintains hierarchy while adapting to screen constraints

## State Management

### Filter State Integration

**NoResults Component**:

- Directly integrates with `useTransactionFilterParamsWithPersistence`
- Accesses `clearAllFilters` function for immediate filter clearing
- Maintains consistency with filter state used throughout the application

**State Dependencies**:

- Relies on parent component's `hasFilters` state for conditional rendering
- Does not manage its own state, focusing purely on presentation and actions

### Component State

**Stateless Design**:

- Both components are purely presentational
- No internal state management required
- All state dependencies come from parent component or hooks

## Performance Considerations

### Rendering Optimization

**Conditional Rendering**:

- Components only render when specific conditions are met
- Prevents unnecessary DOM elements when table has data

**Lightweight Implementation**:

- Minimal component complexity reduces rendering overhead
- No complex state management or side effects

### Memory Efficiency

**Hook Usage**:

- Only `NoResults` uses hooks, and only for filter clearing functionality
- `NoTransactions` is completely static, minimizing memory footprint

**Component Simplicity**:

- Simple JSX structure with minimal computational overhead
- No expensive operations or complex calculations

## Integration Patterns

### Parent Component Integration

```typescript
// Conditional rendering based on data and filter state
if (!tableData.length && !hasFilters) {
  return <NoTransactions />;
}

if (!tableData.length && hasFilters) {
  return <NoResults />;
}
```

**Clear Separation of Concerns**:

- Parent handles state detection and conditional rendering
- Empty state components focus on presentation and user actions
- Clean interface between data state and UI presentation

### Hook Integration

```typescript
// Direct integration with filter management
const { clearAllFilters } = useTransactionFilterParamsWithPersistence();
```

**Consistent State Management**:

- Uses same filter management system as parent table
- Ensures filter clearing works consistently with table behavior
- Maintains single source of truth for filter state

### Component Composition

```typescript
// Integration with other UI components
<AddAccountButton />  // Reusable account connection component
<Button variant="outline" onClick={clearAllFilters}>  // Consistent button styling
```

**Reusable Components**:

- Leverages existing UI components for consistency
- Maintains design system compliance
- Reduces code duplication

## Related Files

### Direct Dependencies

- `@/components/add-account-button` - Bank account connection component
- `@/hooks/use-transaction-filter-params-with-persistence` - Filter state management hook

### Integration Points

- `./data-table.tsx` - Parent component that conditionally renders these empty states
- `./loading.tsx` - Often rendered alongside empty states for consistent loading experience

### UI Dependencies

- `@midday/ui/button` - Button component with variant support
- `@midday/ui/icons` - Icon library for branded transaction icons

### State Dependencies

- Filter state from `useTransactionFilterParamsWithPersistence` hook
- Data state from parent DataTable component (tableData.length, hasFilters)
