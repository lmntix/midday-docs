# use-local-storage Hook Documentation

> **Reference Implementation**: [apps/dashboard/src/hooks/use-local-storage.ts](https://github.com/midday-ai/midday/tree/main/apps/dashboard/src/hooks/use-local-storage.ts)

## Overview
The `use-local-storage` module provides two localStorage hooks: a general-purpose `useLocalStorage` hook with a useState-like API, and a specialized `useFilterLocalStorage` hook designed specifically for filter persistence with debounced saving and conditional storage.

## Key Features
- **SSR-safe initialization**: Handles server-side rendering gracefully
- **Cross-tab synchronization**: Listens for storage events across browser tabs
- **Debounced saving**: Optimized performance for frequent updates
- **Custom serialization**: Flexible data transformation
- **Conditional saving**: Only save when certain conditions are met
- **Error handling**: Graceful fallback when localStorage is unavailable

---

## useLocalStorage Hook

### Overview
A drop-in replacement for `useState` that persists state to localStorage with cross-tab synchronization.

### Interface
```typescript
function useLocalStorage<T>(
  key: string,
  initialValue: T,
): [T, (value: T | ((val: T) => T)) => void]
```

### Parameters
- **key**: Unique localStorage key
- **initialValue**: Default value when no stored value exists

### Return Values
- **[0]**: Current stored value
- **[1]**: Setter function (same API as useState)

### Core Features

#### SSR-Safe Initialization
```typescript
const [localState, setLocalState] = useState<T>(() => {
  if (typeof window === "undefined") {
    return initialValue;
  }
  
  try {
    const item = window.localStorage.getItem(key);
    return item ? JSON.parse(item) : initialValue;
  } catch (error) {
    console.warn(`Error reading localStorage key "${key}":`, error);
    return initialValue;
  }
});
```
- **Server-side rendering**: Returns initialValue when window is undefined
- **Error handling**: Graceful fallback on JSON parsing errors
- **Type safety**: Maintains type consistency throughout

#### Persistent State Updates
```typescript
const handleSetState = useCallback(
  (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(localState) : value;
      setLocalState(valueToStore);
      
      if (typeof window !== "undefined") {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.warn(`Error setting localStorage key "${key}":`, error);
    }
  },
  [key, localState],
);
```
- **Function support**: Accepts updater functions like useState
- **Immediate persistence**: Saves to localStorage on every update
- **Browser check**: Only attempts localStorage operations in browser

#### Cross-Tab Synchronization
```typescript
useEffect(() => {
  function handleStorageChange(event: StorageEvent) {
    if (event.key === key && event.newValue) {
      setLocalState(JSON.parse(event.newValue));
    }
  }

  if (typeof window !== "undefined") {
    window.addEventListener("storage", handleStorageChange);
  }

  return () => {
    if (typeof window !== "undefined") {
      window.removeEventListener("storage", handleStorageChange);
    }
  };
}, [key]);
```
- **Storage events**: Listens for changes in other tabs/windows
- **Key filtering**: Only responds to changes for the specific key
- **Automatic cleanup**: Removes event listeners on unmount

---

## useFilterLocalStorage Hook

### Overview
A specialized localStorage hook designed for filter persistence with advanced features like debounced saving, custom serialization, and conditional storage.

### Interface
```typescript
type UseFilterLocalStorageOptions<T> = {
  key: string;
  defaultValue?: T;
  debounceMs?: number;
  serialize?: (value: T) => string;
  deserialize?: (value: string) => T;
  shouldSave?: (value: T) => boolean;
};

function useFilterLocalStorage<T>({
  key,
  defaultValue,
  debounceMs = 500,
  serialize = JSON.stringify,
  deserialize = JSON.parse,
  shouldSave = () => true,
}: UseFilterLocalStorageOptions<T>)
```

### Parameters
- **key**: Unique localStorage key
- **defaultValue**: Optional default value
- **debounceMs**: Debounce delay for saving (default: 500ms)
- **serialize**: Custom serialization function
- **deserialize**: Custom deserialization function
- **shouldSave**: Condition function for saving

### Return Values
```typescript
{
  initializeFromStorage: (onApply: (value: T) => void, shouldApply?: () => boolean) => void;
  saveToStorage: (value: T) => void;
  clearStorage: () => void;
  loadFromStorage: () => T | null;
}
```

### Core Features

#### Debounced Saving
```typescript
const debouncedSave = useDebounceCallback((value: T) => {
  try {
    if (shouldSave(value)) {
      localStorage.setItem(key, serialize(value));
    } else {
      localStorage.removeItem(key);
    }
  } catch (error) {
    console.error(`Failed to save to localStorage (${key}):`, error);
  }
}, debounceMs);
```
- **Performance optimization**: Prevents excessive localStorage writes
- **Conditional saving**: Uses shouldSave function to determine if value should be stored
- **Automatic cleanup**: Removes key when shouldSave returns false
- **Error handling**: Logs errors and continues gracefully

#### Smart Initialization
```typescript
const initializeFromStorage = useCallback(
  (onApply: (value: T) => void, shouldApply?: () => boolean) => {
    if (hasInitialized.current) return;

    const saved = loadFromStorage();
    if (saved && (!shouldApply || shouldApply())) {
      onApply(saved);
    }

    hasInitialized.current = true;
  },
  [loadFromStorage],
);
```
- **One-time initialization**: Prevents repeated loading
- **Conditional application**: Optional shouldApply function
- **Callback pattern**: Uses onApply callback for state updates
- **Initialization tracking**: Uses ref to track initialization status

#### Custom Serialization Support
```typescript
serialize = JSON.stringify,
deserialize = JSON.parse,
```
- **Default JSON handling**: Works out of the box for most cases
- **Custom serialization**: Supports complex data transformations
- **Filter cleaning**: Can integrate with utility functions for data cleaning

#### Safe Storage Operations
```typescript
const loadFromStorage = useCallback((): T | null => {
  try {
    const item = localStorage.getItem(key);
    return item ? deserialize(item) : null;
  } catch (error) {
    console.error(`Failed to load from localStorage (${key}):`, error);
    localStorage.removeItem(key);
    return null;
  }
}, [key, deserialize]);
```
- **Error recovery**: Removes corrupted data automatically
- **Null handling**: Returns null for missing data
- **Type safety**: Maintains proper return types

## Usage Examples

### Basic useLocalStorage
```typescript
function UserPreferences() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [language, setLanguage] = useLocalStorage('language', 'en');

  return (
    <div>
      <select value={theme} onChange={(e) => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
    </div>
  );
}
```

### Advanced useFilterLocalStorage
```typescript
function FilterManager() {
  const { initializeFromStorage, saveToStorage } = useFilterLocalStorage({
    key: 'advanced-filters',
    debounceMs: 300,
    serialize: (filters) => JSON.stringify(cleanFilters(filters)),
    shouldSave: (filters) => Object.keys(cleanFilters(filters)).length > 0,
  });

  useEffect(() => {
    initializeFromStorage(
      (savedFilters) => setFilters(savedFilters),
      () => !hasActiveUrlFilters(),
    );
  }, []);

  useEffect(() => {
    saveToStorage(currentFilters);
  }, [currentFilters]);

  // Component implementation
}
```

## Performance Optimizations

### Debounced Operations
- **Default 500ms debounce**: Balances responsiveness with performance
- **Configurable timing**: Adjustable debounce delay
- **Prevents excessive writes**: Reduces localStorage operations

### Initialization Control
```typescript
const hasInitialized = useRef(false);
```
- **One-time loading**: Prevents repeated initialization
- **Ref-based tracking**: Doesn't trigger re-renders
- **Memory efficient**: Minimal state overhead

### Conditional Persistence
```typescript
shouldSave: (filters) => Object.keys(cleanFilters(filters)).length > 0
```
- **Smart saving**: Only saves meaningful data
- **Storage optimization**: Removes unnecessary entries
- **Custom conditions**: Flexible saving logic

## Error Handling Strategy

### Graceful Degradation
- **Console warnings**: Logs errors without breaking functionality
- **Fallback values**: Returns safe defaults on errors
- **Automatic cleanup**: Removes corrupted data

### Browser Compatibility
- **localStorage detection**: Checks for localStorage availability
- **SSR safety**: No browser APIs during server rendering
- **Cross-browser support**: Works across modern browsers

## Integration with Filter Systems

### URL Coordination
```typescript
initializeFromStorage(
  (savedFilters) => setFilters(savedFilters),
  () => !hasActiveUrlFilters(currentFilters),
);
```
- **URL priority**: Respects URL-based filter state
- **Conditional loading**: Only loads when appropriate
- **State hierarchy**: Maintains proper precedence

### Filter Cleaning Integration
```typescript
serialize: (filters) => JSON.stringify(cleanFilters(filters))
```
- **Data cleaning**: Removes empty/default values
- **Storage efficiency**: Reduces localStorage usage
- **Consistent format**: Standardized storage format