# Data Model: Support for Overdue Todo Items

**Feature**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md) | **Research**: [research.md](./research.md)  
**Date**: January 29, 2026

## Overview

This feature adds **derived** overdue status to existing Todo entities. No database schema changes are required. Overdue status is calculated client-side from existing `dueDate` and `completed` fields.

---

## Existing Entity: Todo

**Source**: Backend database (SQLite via better-sqlite3)

### Schema (Unchanged)

```sql
CREATE TABLE todos (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  dueDate TEXT,           -- ISO 8601 date string (YYYY-MM-DD) or NULL
  completed INTEGER DEFAULT 0,  -- 0 = false, 1 = true
  createdAt TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Attributes

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | INTEGER | PRIMARY KEY, AUTO INCREMENT | Unique identifier |
| `title` | TEXT | NOT NULL, max 255 chars | Todo title/description |
| `dueDate` | TEXT | NULLABLE | ISO 8601 date (YYYY-MM-DD) or NULL if no due date |
| `completed` | INTEGER | DEFAULT 0 | 0 = incomplete, 1 = complete |
| `createdAt` | TEXT | NOT NULL, DEFAULT NOW | Creation timestamp (ISO 8601) |

### No Changes Required

- Existing fields (`dueDate`, `completed`) provide all data needed
- No new database columns
- No backend service modifications
- No API endpoint changes

---

## Derived Properties (Client-Side)

These properties are **computed** in the frontend from existing Todo data.

### 1. `isOverdue` (boolean)

**Calculation**:
```javascript
isOverdue = (dueDate !== null) 
         && (completed === false)
         && (normalizedDueDate < normalizedToday)
```

**Rules**:
- `true` if todo is incomplete AND due date is before today (00:00:00 local time)
- `false` if completed (regardless of due date)
- `false` if no due date set
- `false` if due date is today or future

**Example**:
```javascript
// Current date: 2026-01-15
const todo1 = { dueDate: '2026-01-14', completed: false }; // isOverdue = true
const todo2 = { dueDate: '2026-01-15', completed: false }; // isOverdue = false (today)
const todo3 = { dueDate: '2026-01-14', completed: true };  // isOverdue = false (completed)
const todo4 = { dueDate: null, completed: false };         // isOverdue = false (no due date)
```

### 2. `daysOverdue` (number)

**Calculation**:
```javascript
if (isOverdue) {
  daysOverdue = floor((today - dueDate) / millisecondsPerDay)
} else {
  daysOverdue = 0
}
```

**Rules**:
- Integer representing days past due date
- Always >= 1 if overdue
- 0 if not overdue
- Uses start-of-day normalization for accurate day count

**Example**:
```javascript
// Current date: 2026-01-15
const todo1 = { dueDate: '2026-01-14', completed: false }; // daysOverdue = 1
const todo2 = { dueDate: '2026-01-10', completed: false }; // daysOverdue = 5
const todo3 = { dueDate: '2026-01-15', completed: false }; // daysOverdue = 0 (not overdue)
```

### 3. `overdueCount` (number)

**Scope**: Aggregate across all todos  
**Calculation**:
```javascript
overdueCount = todos.filter(todo => isOverdue(todo)).length
```

**Rules**:
- Count of incomplete todos with past due dates
- Recalculated when todo list changes
- Used by OverdueCounter component

---

## Data Flow

```
Backend (SQLite)
    │
    │ GET /api/todos
    ▼
Todo[] with dueDate & completed
    │
    │ Frontend receives
    ▼
Client-Side Calculation
    │
    ├─► isOverdue(todo) → boolean
    │
    ├─► getDaysOverdue(todo) → number
    │
    └─► todos.filter(isOverdue).length → overdueCount
    │
    ▼
React Components
    │
    ├─► TodoCard: Display overdue indicator + days
    │
    └─► OverdueCounter: Display total count
```

---

## Calculation Logic (Pseudocode)

### Function: `isOverdue(dueDate, completed)`

```
INPUT: dueDate (string | null), completed (boolean)
OUTPUT: boolean

IF dueDate is null THEN
  RETURN false
END IF

IF completed is true THEN
  RETURN false
END IF

today = normalizeToStartOfDay(currentDate)
due = normalizeToStartOfDay(parseDate(dueDate))

RETURN due < today
```

### Function: `getDaysOverdue(dueDate)`

```
INPUT: dueDate (string)
OUTPUT: number (integer)

today = normalizeToStartOfDay(currentDate)
due = normalizeToStartOfDay(parseDate(dueDate))

IF due >= today THEN
  RETURN 0
END IF

diffMilliseconds = today - due
diffDays = floor(diffMilliseconds / (1000 * 60 * 60 * 24))

RETURN diffDays
```

### Function: `normalizeToStartOfDay(date)`

```
INPUT: date (Date object)
OUTPUT: Date object normalized to 00:00:00 local time

normalized = clone(date)
SET normalized hours to 0
SET normalized minutes to 0
SET normalized seconds to 0
SET normalized milliseconds to 0

RETURN normalized
```

---

## State Management

### Location: React Component State (No Redux/Context)

**App.js**:
- Fetches `todos[]` from backend
- Passes to child components

**TodoCard.js**:
- Receives `todo` prop
- Calculates `isOverdue` and `daysOverdue` via `useMemo`
- Renders visual indicators based on derived values

**OverdueCounter.js**:
- Receives `todos[]` prop
- Calculates `overdueCount` via `useMemo`
- Conditionally renders (hidden if count = 0)

**TodoList.js**:
- Receives `todos[]` prop
- Renders `OverdueCounter` and `TodoCard` components

---

## Validation Rules

### Date Format Validation

- **Input**: ISO 8601 date string (YYYY-MM-DD)
- **Validation**: Must parse to valid Date object
- **Invalid handling**: Treat as null (not overdue)

### Edge Cases

| Scenario | Behavior |
|----------|----------|
| `dueDate = null` | Not overdue |
| `dueDate = ""` (empty string) | Not overdue |
| `dueDate = "invalid"` | Not overdue (parse fails) |
| `dueDate = "2025-12-31"` (far past) | Overdue with accurate day count |
| `dueDate = "2030-01-01"` (far future) | Not overdue |
| `completed = true`, `dueDate` past | Not overdue (completion status wins) |
| Current time 23:59:59, `dueDate = today` | Not overdue (uses start-of-day) |

---

## Performance Considerations

### Calculation Frequency

- **Per TodoCard**: O(1) - single date comparison
- **Per OverdueCounter**: O(n) - filter over todos array
- **Recalculation Triggers**:
  - Todo added/updated/deleted
  - Date changes (midnight boundary)
  - Component re-renders

### Optimization Strategy

- Use `useMemo` to cache calculations
- Memoize dependencies: `[todo.dueDate, todo.completed]`
- No optimization needed for typical use case (<100 todos)
- If 1000+ todos, consider pagination or virtualization (out of scope)

### Memory Footprint

- No additional data stored
- Derived values computed on-demand
- No state duplication

---

## Testing Requirements

### Unit Tests (dateUtils.test.js)

- `isOverdue()` with various date scenarios
- `getDaysOverdue()` with edge cases
- `normalizeToStartOfDay()` timezone handling
- Date parsing error handling

### Component Tests

- TodoCard displays overdue indicator when appropriate
- TodoCard does NOT display indicator for non-overdue
- OverdueCounter shows correct count
- OverdueCounter hidden when count = 0

### Integration Tests

- App correctly calculates overdue status on load
- Overdue status updates when todo marked complete
- Counter updates when todo deleted
- Date boundary edge cases (midnight transition)

---

## Summary

| Aspect | Value |
|--------|-------|
| **Database Changes** | None |
| **API Changes** | None |
| **Backend Changes** | None |
| **Frontend Calculation** | Client-side derived state |
| **Performance Impact** | Negligible (O(1) per item, O(n) for count) |
| **State Management** | React component state with useMemo |
| **Dependencies** | None (native Date API) |
