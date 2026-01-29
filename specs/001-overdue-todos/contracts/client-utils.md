# Client-Side Utility Contracts

**Feature**: Support for Overdue Todo Items  
**Type**: Frontend Utility Functions  
**Location**: `packages/frontend/src/utils/dateUtils.js`

---

## Overview

This document defines the interface contracts for client-side date utility functions used to calculate overdue status. These functions operate entirely in the browser with no backend API calls.

---

## Function: `isOverdue`

**Purpose**: Determine if a todo is overdue based on due date and completion status

### Signature

```javascript
isOverdue(dueDate: string | null, completed: boolean): boolean
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `dueDate` | `string \| null` | Yes | ISO 8601 date string (YYYY-MM-DD) or null |
| `completed` | `boolean` | Yes | Whether the todo is marked as complete |

### Return Value

- **Type**: `boolean`
- **`true`**: Todo is incomplete AND due date is before today (start of day, local time)
- **`false`**: Any of the following:
  - `dueDate` is `null`
  - `dueDate` is invalid/unparseable
  - `completed` is `true`
  - `dueDate` is today or in the future

### Behavior

1. Returns `false` immediately if `dueDate` is `null` or `completed` is `true`
2. Parses `dueDate` into a Date object
3. Normalizes both current date and due date to start of day (00:00:00) in local timezone
4. Compares normalized dates: `normalizedDueDate < normalizedToday`
5. Returns `false` if date parsing fails (invalid date string)

### Examples

```javascript
// Assume current date: 2026-01-15

isOverdue('2026-01-14', false); // true  - yesterday, incomplete
isOverdue('2026-01-15', false); // false - today is not overdue
isOverdue('2026-01-16', false); // false - tomorrow, future date
isOverdue('2026-01-10', false); // true  - 5 days ago, incomplete
isOverdue('2026-01-14', true);  // false - completed todos never overdue
isOverdue(null, false);         // false - no due date
isOverdue('invalid', false);    // false - invalid date string
isOverdue('', false);           // false - empty string
```

### Edge Cases

| Input | Expected Output | Rationale |
|-------|----------------|-----------|
| `dueDate = null` | `false` | Todos without due dates cannot be overdue |
| `dueDate = ""` | `false` | Empty string is invalid, treat as no due date |
| `dueDate = "invalid"` | `false` | Parse failure, cannot determine overdue status |
| `completed = true`, past date | `false` | Completed todos are never overdue |
| Due date = today, 23:59:59 | `false` | Uses start-of-day comparison, today is not overdue |
| Due date far in past (years) | `true` | Accurately identifies as overdue |

### Performance

- **Time Complexity**: O(1) - constant time date comparison
- **Space Complexity**: O(1) - no additional memory allocation

---

## Function: `getDaysOverdue`

**Purpose**: Calculate the number of days a todo is overdue

### Signature

```javascript
getDaysOverdue(dueDate: string | null): number
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `dueDate` | `string \| null` | Yes | ISO 8601 date string (YYYY-MM-DD) or null |

### Return Value

- **Type**: `number` (integer)
- **Range**: `0` to `MAX_SAFE_INTEGER`
- **`0`**: Not overdue (today, future, null, or invalid)
- **`> 0`**: Number of days past the due date

### Behavior

1. Returns `0` if `dueDate` is `null` or invalid
2. Parses `dueDate` into a Date object
3. Normalizes both current date and due date to start of day (00:00:00) in local timezone
4. Calculates difference in milliseconds: `today - dueDate`
5. Converts to days: `Math.floor(diffMs / (1000 * 60 * 60 * 24))`
6. Returns `0` if result is negative (due date in future or today)

### Examples

```javascript
// Assume current date: 2026-01-15

getDaysOverdue('2026-01-14'); // 1  - yesterday
getDaysOverdue('2026-01-10'); // 5  - 5 days ago
getDaysOverdue('2026-01-15'); // 0  - today (not overdue)
getDaysOverdue('2026-01-16'); // 0  - tomorrow (future)
getDaysOverdue('2025-12-31'); // 15 - two weeks ago
getDaysOverdue('2020-01-01'); // 2205 - years ago (accurate calculation)
getDaysOverdue(null);         // 0  - no due date
getDaysOverdue('invalid');    // 0  - invalid date
```

### Edge Cases

| Input | Expected Output | Rationale |
|-------|----------------|-----------|
| `dueDate = null` | `0` | No due date, cannot be overdue |
| `dueDate = "invalid"` | `0` | Parse failure, return safe default |
| Due date = today | `0` | Today is not overdue |
| Due date in future | `0` | Negative difference, clamp to 0 |
| Due date far past (years) | Accurate count | No maximum limit, calculate exact days |
| Midnight boundary | Accurate count | Start-of-day normalization ensures correct day count |

### Performance

- **Time Complexity**: O(1) - constant time calculation
- **Space Complexity**: O(1) - no additional memory allocation

---

## Function: `normalizeToStartOfDay`

**Purpose**: Normalize a Date object to the start of day (00:00:00) in local timezone

### Signature

```javascript
normalizeToStartOfDay(date: Date): Date
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date` | `Date` | Yes | Date object to normalize |

### Return Value

- **Type**: `Date`
- **Value**: New Date object with hours, minutes, seconds, and milliseconds set to 0

### Behavior

1. Creates a new Date object from input
2. Sets hours to 0
3. Sets minutes to 0
4. Sets seconds to 0
5. Sets milliseconds to 0
6. Returns normalized Date in local timezone

### Examples

```javascript
const now = new Date('2026-01-15T14:30:45.123Z');
const normalized = normalizeToStartOfDay(now);
// Result: 2026-01-15T00:00:00.000 (local time)

const midnight = new Date('2026-01-15T00:00:00.000Z');
const normalizedMidnight = normalizeToStartOfDay(midnight);
// Result: 2026-01-15T00:00:00.000 (local time, same value)
```

### Performance

- **Time Complexity**: O(1)
- **Space Complexity**: O(1)

---

## Helper Function: `formatDaysOverdue`

**Purpose**: Format the days overdue count into a human-readable string

### Signature

```javascript
formatDaysOverdue(days: number): string
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `days` | `number` | Yes | Number of days overdue (integer >= 0) |

### Return Value

- **Type**: `string`
- **Format**: `"{days} day(s) overdue"`
- **Examples**: `"1 day overdue"`, `"5 days overdue"`

### Behavior

1. Returns empty string if `days` is 0
2. Returns `"1 day overdue"` if `days` is 1 (singular)
3. Returns `"{days} days overdue"` if `days` > 1 (plural)

### Examples

```javascript
formatDaysOverdue(0);   // ""
formatDaysOverdue(1);   // "1 day overdue"
formatDaysOverdue(2);   // "2 days overdue"
formatDaysOverdue(10);  // "10 days overdue"
formatDaysOverdue(365); // "365 days overdue"
```

---

## Testing Requirements

### Unit Tests

Each function must have comprehensive unit tests covering:

1. **Happy path**: Valid inputs, expected outputs
2. **Edge cases**: Null, undefined, invalid dates
3. **Boundary conditions**: Today, yesterday, tomorrow, midnight
4. **Date mocking**: Fixed `Date.now()` for reproducible tests

### Test Coverage Target

- **Minimum**: 90% line coverage
- **Target**: 100% branch coverage
- **Critical paths**: All edge cases tested

### Example Test Structure

```javascript
describe('isOverdue', () => {
  beforeEach(() => {
    jest.spyOn(Date, 'now').mockImplementation(() => 
      new Date('2026-01-15T12:00:00Z').getTime()
    );
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  test('returns true for yesterday', () => {
    expect(isOverdue('2026-01-14', false)).toBe(true);
  });

  test('returns false for today', () => {
    expect(isOverdue('2026-01-15', false)).toBe(false);
  });

  // ... more tests
});
```

---

## Error Handling

### Invalid Inputs

| Input Type | Handling | Return Value |
|------------|----------|--------------|
| `null` date | Check before parsing | `false` or `0` |
| Empty string `""` | Date parse fails | `false` or `0` |
| Invalid format `"invalid"` | Date parse fails | `false` or `0` |
| Invalid Date object | `isNaN(date.getTime())` | `false` or `0` |
| Undefined | Type coercion to `null` | `false` or `0` |

### No Exceptions Thrown

- All functions return safe default values on error
- No `throw` statements
- Graceful degradation: invalid date → not overdue

---

## Dependencies

### Internal

- None - utility functions are self-contained

### External

- **Native JavaScript**:
  - `Date` object (built-in)
  - `Math.floor` (built-in)
  - No external libraries required

---

## Usage Example

```javascript
import { isOverdue, getDaysOverdue, formatDaysOverdue } from '../utils/dateUtils';

function TodoCard({ todo }) {
  const overdue = isOverdue(todo.dueDate, todo.completed);
  const days = getDaysOverdue(todo.dueDate);
  
  return (
    <div className={overdue ? 'todo-card overdue' : 'todo-card'}>
      <h3>{todo.title}</h3>
      {overdue && (
        <div className="overdue-indicator">
          <span>⚠️</span>
          <span>{formatDaysOverdue(days)}</span>
        </div>
      )}
    </div>
  );
}
```

---

## Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-01-29 | Initial contract definition |
