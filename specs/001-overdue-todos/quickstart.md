# Quickstart Guide: Overdue Todo Items Feature

**Feature**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md)  
**For**: Developers implementing the overdue todos feature

---

## Overview

This guide provides step-by-step instructions for implementing the overdue todo items feature. Follow the phases in order to ensure proper integration with the existing codebase.

**Estimated Time**: 4-6 hours (including testing)

---

## Prerequisites

Before starting:
- [ ] Node.js v16+ installed
- [ ] Repository cloned and dependencies installed (`npm install`)
- [ ] Existing tests passing (`npm test`)
- [ ] Feature branch checked out (`001-overdue-todos`)
- [ ] Familiarize yourself with existing codebase:
  - `packages/frontend/src/components/TodoCard.js`
  - `packages/frontend/src/components/TodoList.js`
  - `packages/frontend/src/App.js`
  - `docs/ui-guidelines.md`

---

## Phase 1: Create Date Utilities (1-2 hours)

### Step 1.1: Create utility file

**File**: `packages/frontend/src/utils/dateUtils.js`

```javascript
/**
 * Date utility functions for overdue todo calculations
 */

/**
 * Normalize a date to the start of day (00:00:00) in local timezone
 * @param {Date} date - Date to normalize
 * @returns {Date} Normalized date
 */
export function normalizeToStartOfDay(date) {
  const normalized = new Date(date);
  normalized.setHours(0, 0, 0, 0);
  return normalized;
}

/**
 * Check if a todo is overdue
 * @param {string|null} dueDate - ISO 8601 date string or null
 * @param {boolean} completed - Whether todo is completed
 * @returns {boolean} True if overdue, false otherwise
 */
export function isOverdue(dueDate, completed) {
  if (!dueDate || completed) {
    return false;
  }

  try {
    const today = normalizeToStartOfDay(new Date());
    const due = normalizeToStartOfDay(new Date(dueDate));

    if (isNaN(due.getTime())) {
      return false; // Invalid date
    }

    return due < today;
  } catch (error) {
    return false; // Parse error
  }
}

/**
 * Calculate number of days overdue
 * @param {string|null} dueDate - ISO 8601 date string or null
 * @returns {number} Days overdue (0 if not overdue)
 */
export function getDaysOverdue(dueDate) {
  if (!dueDate) {
    return 0;
  }

  try {
    const today = normalizeToStartOfDay(new Date());
    const due = normalizeToStartOfDay(new Date(dueDate));

    if (isNaN(due.getTime())) {
      return 0; // Invalid date
    }

    const diffMs = today - due;
    const days = Math.floor(diffMs / (1000 * 60 * 60 * 24));

    return days > 0 ? days : 0;
  } catch (error) {
    return 0; // Parse error
  }
}

/**
 * Format days overdue into human-readable string
 * @param {number} days - Number of days
 * @returns {string} Formatted string
 */
export function formatDaysOverdue(days) {
  if (days === 0) return '';
  if (days === 1) return '1 day overdue';
  return `${days} days overdue`;
}
```

### Step 1.2: Create utility tests

**File**: `packages/frontend/src/utils/__tests__/dateUtils.test.js`

```javascript
import { isOverdue, getDaysOverdue, formatDaysOverdue, normalizeToStartOfDay } from '../dateUtils';

describe('dateUtils', () => {
  beforeEach(() => {
    // Mock current date to 2026-01-15 12:00:00
    jest.spyOn(Date, 'now').mockImplementation(() =>
      new Date('2026-01-15T12:00:00Z').getTime()
    );
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  describe('normalizeToStartOfDay', () => {
    test('sets time to 00:00:00', () => {
      const date = new Date('2026-01-15T14:30:45.123Z');
      const normalized = normalizeToStartOfDay(date);
      
      expect(normalized.getHours()).toBe(0);
      expect(normalized.getMinutes()).toBe(0);
      expect(normalized.getSeconds()).toBe(0);
      expect(normalized.getMilliseconds()).toBe(0);
    });
  });

  describe('isOverdue', () => {
    test('returns true for yesterday', () => {
      expect(isOverdue('2026-01-14', false)).toBe(true);
    });

    test('returns false for today', () => {
      expect(isOverdue('2026-01-15', false)).toBe(false);
    });

    test('returns false for future date', () => {
      expect(isOverdue('2026-01-16', false)).toBe(false);
    });

    test('returns false for completed todo', () => {
      expect(isOverdue('2026-01-14', true)).toBe(false);
    });

    test('returns false for null due date', () => {
      expect(isOverdue(null, false)).toBe(false);
    });

    test('returns false for invalid date', () => {
      expect(isOverdue('invalid', false)).toBe(false);
    });

    test('returns false for empty string', () => {
      expect(isOverdue('', false)).toBe(false);
    });
  });

  describe('getDaysOverdue', () => {
    test('calculates 1 day overdue', () => {
      expect(getDaysOverdue('2026-01-14')).toBe(1);
    });

    test('calculates 5 days overdue', () => {
      expect(getDaysOverdue('2026-01-10')).toBe(5);
    });

    test('returns 0 for today', () => {
      expect(getDaysOverdue('2026-01-15')).toBe(0);
    });

    test('returns 0 for future date', () => {
      expect(getDaysOverdue('2026-01-16')).toBe(0);
    });

    test('returns 0 for null', () => {
      expect(getDaysOverdue(null)).toBe(0);
    });

    test('returns 0 for invalid date', () => {
      expect(getDaysOverdue('invalid')).toBe(0);
    });
  });

  describe('formatDaysOverdue', () => {
    test('returns empty string for 0', () => {
      expect(formatDaysOverdue(0)).toBe('');
    });

    test('returns singular for 1 day', () => {
      expect(formatDaysOverdue(1)).toBe('1 day overdue');
    });

    test('returns plural for multiple days', () => {
      expect(formatDaysOverdue(5)).toBe('5 days overdue');
    });
  });
});
```

### Step 1.3: Run tests

```bash
npm test -- dateUtils.test.js
```

**Expected**: All tests pass ✅

---

## Phase 2: Add CSS Styling (30 minutes)

### Step 2.1: Update theme.css

**File**: `packages/frontend/src/styles/theme.css`

Add to the end of the file:

```css
/* Overdue todo styling */
:root {
  --color-overdue-text: #c62828;
  --color-overdue-bg: #ffebee;
  --color-overdue-border: #ef5350;
  --color-overdue-icon: #d32f2f;
  --color-overdue-badge-bg: #c62828;
  --color-overdue-badge-text: #ffffff;
}

[data-theme="dark"] {
  --color-overdue-text: #ef5350;
  --color-overdue-bg: #3d1f1f;
  --color-overdue-border: #ff8a80;
  --color-overdue-icon: #ff8a80;
  --color-overdue-badge-bg: #ef5350;
  --color-overdue-badge-text: #1a1a1a;
}

/* Overdue TodoCard styling */
.todo-card.overdue {
  border-left: 4px solid var(--color-overdue-border);
  background-color: var(--color-overdue-bg);
}

.overdue-indicator {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--color-overdue-text);
  font-size: 12px;
  font-weight: 600;
  margin-top: 8px;
}

.overdue-icon {
  font-size: 16px;
}

.overdue-duration {
  color: var(--color-text-secondary);
  font-size: 12px;
  margin-top: 4px;
}

/* Overdue counter in header */
.overdue-counter {
  display: inline-flex;
  align-items: center;
  padding: 4px 12px;
  background-color: var(--color-overdue-badge-bg);
  color: var(--color-overdue-badge-text);
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
  margin-left: 16px;
}
```

---

## Phase 3: Create OverdueCounter Component (30 minutes)

### Step 3.1: Create component

**File**: `packages/frontend/src/components/OverdueCounter.js`

```javascript
import React, { useMemo } from 'react';
import { isOverdue } from '../utils/dateUtils';

/**
 * OverdueCounter Component
 * Displays count of overdue todos in the header
 */
function OverdueCounter({ todos }) {
  const overdueCount = useMemo(() => {
    return todos.filter(todo => isOverdue(todo.dueDate, todo.completed)).length;
  }, [todos]);

  // Hide counter when no overdue items
  if (overdueCount === 0) {
    return null;
  }

  return (
    <span className="overdue-counter" role="status" aria-live="polite">
      {overdueCount} overdue
    </span>
  );
}

export default OverdueCounter;
```

### Step 3.2: Create component tests

**File**: `packages/frontend/src/components/__tests__/OverdueCounter.test.js`

```javascript
import React from 'react';
import { render, screen } from '@testing-library/react';
import OverdueCounter from '../OverdueCounter';

describe('OverdueCounter', () => {
  beforeEach(() => {
    jest.spyOn(Date, 'now').mockImplementation(() =>
      new Date('2026-01-15T12:00:00Z').getTime()
    );
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  test('displays count for overdue todos', () => {
    const todos = [
      { id: 1, dueDate: '2026-01-14', completed: false },
      { id: 2, dueDate: '2026-01-10', completed: false },
      { id: 3, dueDate: '2026-01-16', completed: false },
    ];

    render(<OverdueCounter todos={todos} />);

    expect(screen.getByText('2 overdue')).toBeInTheDocument();
  });

  test('is hidden when no overdue todos', () => {
    const todos = [
      { id: 1, dueDate: '2026-01-16', completed: false },
      { id: 2, dueDate: '2026-01-17', completed: false },
    ];

    const { container } = render(<OverdueCounter todos={todos} />);

    expect(container.firstChild).toBeNull();
  });

  test('excludes completed todos from count', () => {
    const todos = [
      { id: 1, dueDate: '2026-01-14', completed: false },
      { id: 2, dueDate: '2026-01-10', completed: true },
    ];

    render(<OverdueCounter todos={todos} />);

    expect(screen.getByText('1 overdue')).toBeInTheDocument();
  });
});
```

### Step 3.3: Run tests

```bash
npm test -- OverdueCounter.test.js
```

---

## Phase 4: Update TodoCard Component (1 hour)

### Step 4.1: Modify TodoCard.js

**File**: `packages/frontend/src/components/TodoCard.js`

Add imports at the top:
```javascript
import { isOverdue, getDaysOverdue, formatDaysOverdue } from '../utils/dateUtils';
```

Inside the component function, add derived state calculations:
```javascript
function TodoCard({ todo, onToggle, onEdit, onDelete, isLoading }) {
  // ... existing state declarations ...
  
  // Overdue calculations
  const overdue = useMemo(() => {
    return isOverdue(todo.dueDate, todo.completed);
  }, [todo.dueDate, todo.completed]);

  const daysOverdue = useMemo(() => {
    return getDaysOverdue(todo.dueDate);
  }, [todo.dueDate]);

  // ... rest of component ...
```

In the render section, update the className and add overdue indicator:
```javascript
return (
  <div className={`todo-card ${overdue ? 'overdue' : ''} ${todo.completed ? 'completed' : ''}`}>
    {/* ... existing content ... */}
    
    {/* Add after the due date display */}
    {overdue && (
      <div className="overdue-indicator" role="status" aria-label="This todo is overdue">
        <span className="overdue-icon" aria-hidden="true">⚠️</span>
        <span>{formatDaysOverdue(daysOverdue)}</span>
      </div>
    )}
    
    {/* ... rest of content ... */}
  </div>
);
```

### Step 4.2: Update TodoCard tests

**File**: `packages/frontend/src/components/__tests__/TodoCard.test.js`

Add tests for overdue functionality:
```javascript
describe('TodoCard overdue display', () => {
  beforeEach(() => {
    jest.spyOn(Date, 'now').mockImplementation(() =>
      new Date('2026-01-15T12:00:00Z').getTime()
    );
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  test('displays overdue indicator for past due date', () => {
    const todo = {
      id: 1,
      title: 'Test Todo',
      dueDate: '2026-01-14',
      completed: false
    };

    render(<TodoCard todo={todo} onToggle={jest.fn()} onEdit={jest.fn()} onDelete={jest.fn()} />);

    expect(screen.getByRole('status', { name: /overdue/i })).toBeInTheDocument();
    expect(screen.getByText('1 day overdue')).toBeInTheDocument();
  });

  test('does not display overdue indicator for today', () => {
    const todo = {
      id: 1,
      title: 'Test Todo',
      dueDate: '2026-01-15',
      completed: false
    };

    render(<TodoCard todo={todo} onToggle={jest.fn()} onEdit={jest.fn()} onDelete={jest.fn()} />);

    expect(screen.queryByRole('status', { name: /overdue/i })).not.toBeInTheDocument();
  });

  test('does not display overdue for completed todo', () => {
    const todo = {
      id: 1,
      title: 'Test Todo',
      dueDate: '2026-01-14',
      completed: true
    };

    render(<TodoCard todo={todo} onToggle={jest.fn()} onEdit={jest.fn()} onDelete={jest.fn()} />);

    expect(screen.queryByRole('status', { name: /overdue/i })).not.toBeInTheDocument();
  });
});
```

### Step 4.3: Run tests

```bash
npm test -- TodoCard.test.js
```

---

## Phase 5: Update TodoList Component (30 minutes)

### Step 5.1: Modify TodoList.js

**File**: `packages/frontend/src/components/TodoList.js`

Add import:
```javascript
import OverdueCounter from './OverdueCounter';
```

Add OverdueCounter to render:
```javascript
function TodoList({ todos, onToggle, onEdit, onDelete, isLoading }) {
  return (
    <div className="todo-list">
      <OverdueCounter todos={todos} />
      
      {/* ... existing todo rendering ... */}
    </div>
  );
}
```

### Step 5.2: Update TodoList tests

**File**: `packages/frontend/src/components/__tests__/TodoList.test.js`

Add test for OverdueCounter integration:
```javascript
test('displays overdue counter with correct count', () => {
  jest.spyOn(Date, 'now').mockImplementation(() =>
    new Date('2026-01-15T12:00:00Z').getTime()
  );

  const todos = [
    { id: 1, title: 'Overdue 1', dueDate: '2026-01-14', completed: false },
    { id: 2, title: 'Overdue 2', dueDate: '2026-01-10', completed: false },
    { id: 3, title: 'Future', dueDate: '2026-01-16', completed: false },
  ];

  render(<TodoList todos={todos} onToggle={jest.fn()} onEdit={jest.fn()} onDelete={jest.fn()} />);

  expect(screen.getByText('2 overdue')).toBeInTheDocument();

  jest.restoreAllMocks();
});
```

---

## Phase 6: Integration Testing (30 minutes)

### Step 6.1: Run full test suite

```bash
npm test
```

**Expected**: All tests pass with 80%+ coverage ✅

### Step 6.2: Manual testing

Start the application:
```bash
npm start
```

**Test scenarios**:
1. ✅ Create a todo with yesterday's date → See overdue indicator
2. ✅ Create a todo with today's date → No overdue indicator
3. ✅ Create a todo with tomorrow's date → No overdue indicator
4. ✅ Mark overdue todo complete → Indicator disappears
5. ✅ Verify overdue counter in header shows correct count
6. ✅ Verify counter hidden when no overdue todos
7. ✅ Test dark mode → Overdue colors render correctly

---

## Verification Checklist

Before committing:
- [ ] All unit tests pass
- [ ] All component tests pass
- [ ] Integration tests pass
- [ ] Test coverage ≥ 80%
- [ ] No ESLint errors
- [ ] Manual testing scenarios verified
- [ ] Dark mode tested
- [ ] Accessibility checked (screen reader, keyboard navigation)
- [ ] No console errors in browser
- [ ] Code follows project conventions (naming, imports, formatting)

---

## Common Issues & Solutions

### Issue: Tests fail with "Date is not defined"
**Solution**: Ensure `jest.spyOn(Date, 'now')` is in `beforeEach` and `jest.restoreAllMocks()` in `afterEach`

### Issue: Overdue indicator not showing
**Solution**: Check that `todo.dueDate` is in ISO format (YYYY-MM-DD) and `todo.completed` is boolean

### Issue: Counter not updating
**Solution**: Verify `useMemo` dependencies include `[todos]` array

### Issue: CSS not applying
**Solution**: Check that theme.css is imported and CSS custom properties are defined

---

## Next Steps

After completing this quickstart:
1. Commit your changes with descriptive message
2. Push to feature branch
3. Create pull request
4. Address code review feedback
5. Merge to main after approval

**Congratulations!** You've successfully implemented the overdue todos feature. 🎉
