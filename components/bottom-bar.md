# bottom-bar.tsx

> **Reference Implementation**: [apps/dashboard/src/components/tables/transactions/bottom-bar.tsx](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/components/tables/transactions/bottom-bar.tsx)

## Purpose

The `bottom-bar.tsx` file implements a floating summary bar that appears at the bottom of the screen when filters are applied to the transactions table. It provides users with real-time totals and transaction counts for the currently filtered data, supporting multi-currency calculations and internationalization. The component enhances user experience by giving immediate feedback about the impact of applied filters.

## Key Features

- **Multi-Currency Support**: Calculates and displays totals for each currency separately
- **Real-Time Filter Integration**: Updates automatically when filters change
- **Smooth Animations**: Uses Framer Motion for elegant enter/exit animations
- **Internationalization**: Supports multiple locales for number formatting and text
- **Tooltip Information**: Provides detailed breakdown on hover for multi-currency scenarios
- **Responsive Design**: Fixed positioning that works across different screen sizes
- **Loading State Handling**: Gracefully handles loading states with conditional rendering
- **Backdrop Blur Effect**: Modern glass-morphism design with backdrop filtering

## Dependencies

### Internal Dependencies

- `useTransactionFilterParams`: Hook providing current filter parameters
- `useUserQuery`: Hook providing user data including locale preferences
- `useI18n`: Internationalization hook for localized text
- `formatAmount`: Utility function for consistent currency formatting

### External Dependencies

- `@tanstack/react-query`: Data fetching and caching with useQuery
- `framer-motion`: Animation library for smooth transitions
- `@midday/ui/icons`: Icon components for visual elements
- `@midday/ui/tooltip`: Tooltip components for additional information display

## Implementation Details

### Component Structure and Data Fetching

```typescript
export function BottomBar() {
  const { data: user } = useUserQuery();
  const t = useI18n();
  const trpc = useTRPC();
  const { filter } = useTransactionFilterParams();

  const { data: transactions, isLoading } = useQuery({
    ...trpc.transactions.get.queryOptions({
      ...filter,
      pageSize: 10000,
    }),
  });
```

**Data Fetching Strategy**:

- **Large Page Size**: Uses `pageSize: 10000` to fetch all filtered transactions for accurate totals
- **Filter Integration**: Automatically includes current filter parameters in the query
- **React Query Caching**: Leverages React Query's caching to avoid unnecessary refetches
- **Loading State**: Tracks loading state to prevent rendering during data fetch

### Multi-Currency Total Calculation

#### Core Calculation Logic

```typescript
const totalAmount = useMemo(() => {
  const totals: Transaction[] = [];

  for (const transaction of transactions?.data ?? []) {
    if (!transaction.amount || !transaction.currency) continue;

    const existingTotal = totals.find(
      (total) => total.currency === transaction.currency
    );

    if (existingTotal) {
      existingTotal.amount += transaction.amount;
    } else {
      totals.push({
        currency: transaction.currency,
        amount: transaction.amount,
      });
    }
  }

  return totals;
}, [transactions]);
```

**Algorithm Breakdown**:

1. **Initialize Empty Totals**: Start with empty array to accumulate currency totals
2. **Iterate Transactions**: Process each transaction in the filtered dataset
3. **Data Validation**: Skip transactions with missing amount or currency data
4. **Currency Grouping**: Find existing total for the transaction's currency
5. **Accumulation**: Add to existing total or create new currency entry
6. **Memoization**: Use `useMemo` to prevent recalculation on every render

**Performance Considerations**:

- **Memoized Calculation**: Only recalculates when transactions data changes
- **Efficient Lookup**: Uses `Array.find()` for currency matching (acceptable for typical currency counts)
- **Data Validation**: Early exit for invalid transactions prevents errors

#### Multi-Currency Detection and Display

```typescript
const multiCurrency = totalAmount && totalAmount.length > 1;

const amountPerCurrency = totalAmount
  .map((total) =>
    formatAmount({
      amount: total?.amount,
      currency: total.currency,
      locale: user?.locale ?? undefined,
    })
  )
  .join(", ");
```

**Display Logic**:

- **Multi-Currency Detection**: Determines if more than one currency is present
- **Formatted Display**: Creates comma-separated list of all currency totals
- **Locale-Aware Formatting**: Uses user's locale for appropriate number formatting

### Animation Implementation with Framer Motion

#### Enter/Exit Animation Configuration

```typescript
<motion.div
  className="h-12 fixed bottom-4 left-0 right-0 pointer-events-none flex justify-center"
  initial={{ y: 100, opacity: 0 }}
  animate={{ y: 0, opacity: 1 }}
  exit={{ y: 100, opacity: 0 }}
  transition={{ type: "spring", stiffness: 400, damping: 25 }}
>
```

**Animation Properties**:

- **Initial State**: `y: 100, opacity: 0` - starts below viewport and transparent
- **Animate State**: `y: 0, opacity: 1` - slides up to final position and becomes opaque
- **Exit State**: `y: 100, opacity: 0` - slides down and fades out when removed
- **Spring Transition**: Uses spring physics with high stiffness (400) and moderate damping (25)

**Animation Characteristics**:

- **Smooth Entry**: Bar slides up from bottom with spring animation
- **Natural Feel**: Spring physics create organic, bouncy motion
- **Quick Response**: High stiffness ensures responsive animation
- **Stable Settling**: Moderate damping prevents excessive oscillation

#### Positioning and Layout Strategy

```typescript
className =
  "h-12 fixed bottom-4 left-0 right-0 pointer-events-none flex justify-center";
```

**Positioning Technique**:

- **Fixed Positioning**: `fixed bottom-4 left-0 right-0` creates full-width bottom positioning
- **Pointer Events**: `pointer-events-none` on container allows clicks to pass through
- **Centered Content**: `flex justify-center` centers the actual content bar
- **Selective Interaction**: Inner content has `pointer-events-auto` to enable interactions

### Conditional Display Logic

#### Single vs Multi-Currency Display

```typescript
<span className="text-sm">
  {multiCurrency
    ? t("bottom_bar.multi_currency")
    : totalAmount.length > 0 &&
      totalAmount[0] &&
      formatAmount({
        amount: totalAmount[0].amount,
        currency: totalAmount[0].currency,
        locale: user?.locale ?? undefined,
        maximumFractionDigits: 0,
        minimumFractionDigits: 0,
      })}
</span>
```

**Display Rules**:

- **Multi-Currency**: Shows localized "Multiple currencies" text
- **Single Currency**: Shows formatted amount with no decimal places
- **Empty State**: Shows nothing if no totals exist
- **Locale Formatting**: Respects user's locale for number formatting

#### Tooltip Content Strategy

```typescript
<TooltipContent sideOffset={30} className="px-3 py-1.5 text-xs">
  {multiCurrency ? amountPerCurrency : t("bottom_bar.description")}
</TooltipContent>
```

**Tooltip Logic**:

- **Multi-Currency**: Shows detailed breakdown of all currency totals
- **Single Currency**: Shows descriptive text about the total
- **Positioning**: `sideOffset={30}` ensures tooltip appears above the bar
- **Styling**: Consistent with design system typography and spacing

### Internationalization Integration

#### Localized Text Handling

```typescript
const t = useI18n();

// Usage examples
t("bottom_bar.multi_currency");
t("bottom_bar.description");
t("bottom_bar.transactions", { count: transactions?.data?.length ?? 0 });
```

**i18n Features**:

- **Dynamic Text**: All user-facing text is localized
- **Pluralization**: Transaction count uses proper plural forms
- **Context-Aware**: Different messages for different scenarios

#### Locale-Aware Number Formatting

```typescript
formatAmount({
  amount: totalAmount[0].amount,
  currency: totalAmount[0].currency,
  locale: user?.locale ?? undefined,
  maximumFractionDigits: 0,
  minimumFractionDigits: 0,
});
```

**Formatting Options**:

- **User Locale**: Uses user's preferred locale for formatting
- **Currency Display**: Includes currency symbols appropriate to locale
- **Decimal Control**: Configurable decimal places for different display contexts
- **Fallback Handling**: Graceful fallback when user locale is unavailable

### Visual Design and Styling

#### Glass Morphism Effect

```typescript
className =
  "pointer-events-auto backdrop-filter dark:bg-[#1A1A1A]/80 bg-[#F6F6F3]/80 h-12 justify-between items-center flex px-4 border dark:border-[#2C2C2C] border-[#DCDAD2] space-x-2";
```

**Design Elements**:

- **Backdrop Filter**: Creates blur effect behind the bar
- **Semi-Transparent Background**: 80% opacity allows content to show through
- **Theme-Aware Colors**: Different colors for light and dark themes
- **Border Styling**: Subtle borders that adapt to theme
- **Consistent Height**: Fixed 48px height (`h-12`) for predictable layout

#### Content Layout

```typescript
<div className="/* ... */ justify-between items-center flex px-4 /* ... */ space-x-2">
  <TooltipProvider>
    {/* Left side: Total amount with info icon */}
  </TooltipProvider>

  <span className="text-sm text-[#878787]">
    {/* Right side: Transaction count */}
  </span>
</div>
```

**Layout Structure**:

- **Justify Between**: Spreads content to left and right edges
- **Vertical Centering**: `items-center` aligns content vertically
- **Horizontal Padding**: `px-4` provides breathing room
- **Content Spacing**: `space-x-2` adds consistent spacing between elements

### Loading State Management

```typescript
if (isLoading) return null;
```

**Loading Strategy**:

- **Conditional Rendering**: Component doesn't render during loading
- **Prevents Flash**: Avoids showing incomplete or incorrect data
- **Clean UX**: No loading spinner needed due to conditional rendering
- **Performance**: Reduces DOM complexity during loading states

## State Management

### Filter State Integration

```typescript
const { filter } = useTransactionFilterParams();

const { data: transactions, isLoading } = useQuery({
  ...trpc.transactions.get.queryOptions({
    ...filter,
    pageSize: 10000,
  }),
});
```

**Automatic Updates**:

- Directly integrates with filter parameters from URL/localStorage
- Query automatically refetches when filter parameters change
- No manual state synchronization required

### Computed State

```typescript
const totalAmount = useMemo(() => {
  // Multi-currency calculation logic
}, [transactions]);

const multiCurrency = totalAmount && totalAmount.length > 1;
```

**Derived State**:

- `totalAmount`: Computed from transaction data, memoized for performance
- `multiCurrency`: Boolean flag derived from total amount array length
- `amountPerCurrency`: Formatted string for tooltip display

## Performance Considerations

### Memoization Strategy

```typescript
const totalAmount = useMemo(() => {
  // Expensive calculation only runs when transactions change
}, [transactions]);
```

**Benefits**:

- Prevents recalculation on every render
- Only recalculates when transaction data actually changes
- Reduces computational overhead for large transaction sets

### Query Optimization

```typescript
const { data: transactions, isLoading } = useQuery({
  ...trpc.transactions.get.queryOptions({
    ...filter,
    pageSize: 10000,
  }),
});
```

**Caching Strategy**:

- React Query automatically caches results based on filter parameters
- Prevents redundant API calls when filters haven't changed
- Background refetching keeps data fresh

### Conditional Rendering

```typescript
if (isLoading) return null;
```

**Performance Benefits**:

- Avoids rendering complex DOM structure during loading
- Prevents unnecessary animation calculations
- Reduces memory usage during data fetching

## Integration Patterns

### Parent Component Integration

The BottomBar is conditionally rendered in the main DataTable component:

```typescript
// In data-table.tsx
const showBottomBar = hasFilters && !Object.keys(rowSelection).length;

return (
  <div className="relative">
    {/* Table content */}
    <AnimatePresence>{showBottomBar && <BottomBar />}</AnimatePresence>
  </div>
);
```

**Conditional Display Rules**:

- Only shows when filters are active (`hasFilters`)
- Hides when rows are selected (bulk actions take precedence)
- Uses `AnimatePresence` for smooth enter/exit animations

### Hook Integration

```typescript
const { filter } = useTransactionFilterParams(); // Filter state
const { data: user } = useUserQuery(); // User preferences
const t = useI18n(); // Localization
```

**Seamless Integration**:

- Uses same filter hook as parent table for consistency
- Integrates with user preferences for locale-aware formatting
- Leverages centralized i18n system for text localization

## Related Files

### Direct Dependencies

- `@/hooks/use-transaction-filter-params` - Filter parameter management
- `@/hooks/use-user` - User data and preferences
- `@/locales/client` - Internationalization support
- `@/utils/format` - Currency formatting utilities

### Integration Points

- `./data-table.tsx` - Parent component that conditionally renders the bottom bar
- `./export-bar.tsx` - Alternative bar that appears during row selection

### Hook Dependencies

- `@/hooks/use-transaction-filter-params` - Provides current filter state
- `@/hooks/use-user` - Provides user locale for formatting
- `@/locales/client` - Provides localized text strings

### UI Dependencies

- `@midday/ui/icons` - Info icon for tooltip trigger
- `@midday/ui/tooltip` - Tooltip components for additional information
- `framer-motion` - Animation library for smooth transitions
