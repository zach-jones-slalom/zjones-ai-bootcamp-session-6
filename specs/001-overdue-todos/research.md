# Research: Support for Overdue Todo Items

**Feature**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md)  
**Date**: January 29, 2026

## Research Tasks

From Technical Context and feature requirements, the following areas require research:
1. Best practices for client-side date comparison in JavaScript
2. Accessibility patterns for visual indicators (color + icon)
3. React patterns for derived state (overdue calculation)
4. CSS design tokens for overdue styling in existing theme
5. Testing patterns for date-dependent logic

---

## 1. Client-Side Date Comparison in JavaScript

### Decision: Use Date object with local timezone normalization

**Rationale**:
- JavaScript `Date` object provides built-in timezone handling
- Normalize dates to start of day (00:00:00) for accurate "overdue" determination
- Avoid external date libraries (moment.js deprecated, date-fns adds dependency)
- Native `Date` methods sufficient for day-level granularity

**Implementation Pattern**:
```javascript
// Normalize date to start of day in local timezone
function normalizeToStartOfDay(date) {
  const normalized = new Date(date);
  normalized.setHours(0, 0, 0, 0);
  return normalized;
}

// Check if todo is overdue
function isOverdue(dueDate, completedStatus) {
  if (!dueDate || completedStatus) return false;
  
  const today = normalizeToStartOfDay(new Date());
  const due = normalizeToStartOfDay(new Date(dueDate));
  
  return due < today; // Overdue if due date is before today
}

// Calculate days overdue
function getDaysOverdue(dueDate) {
  const today = normalizeToStartOfDay(new Date());
  const due = normalizeToStartOfDay(new Date(dueDate));
  
  const diffMs = today - due;
  return Math.floor(diffMs / (1000 * 60 * 60 * 24));
}
```

**Alternatives Considered**:
- **date-fns**: Rejected - adds 13KB+ dependency for simple date comparison
- **moment.js**: Rejected - deprecated, large bundle size (67KB)
- **Luxon**: Rejected - overkill for day-level comparison
- **String comparison**: Rejected - error-prone with timezone edge cases

**Edge Cases Handled**:
- Invalid date strings → return false (not overdue)
- Null/undefined due dates → return false
- Completed todos → never overdue regardless of date
- Midnight boundary → 00:00:00 comparison ensures correct transition

---

## 2. Accessibility Patterns for Visual Indicators

### Decision: Color + Icon + ARIA attributes

**Rationale**:
- WCAG 2.1 Level AA requires not relying on color alone
- Icon provides visual redundancy for colorblind users
- ARIA attributes ensure screen reader accessibility
- Existing Halloween theme uses danger color (#c62828 light, #ef5350 dark)

**Implementation Pattern**:
```javascript
// TodoCard component enhancement
{isOverdue && (
  <div className="overdue-indicator" role="status" aria-label="This todo is overdue">
    <span className="overdue-icon" aria-hidden="true">⚠️</span>
    <span className="overdue-text">Overdue</span>
  </div>
)}
```

**CSS Variables (theme.css)**:
```css
/* Light mode */
--color-overdue: #c62828;
--color-overdue-bg: #ffebee;
--color-overdue-border: #ef5350;

/* Dark mode */
[data-theme="dark"] {
  --color-overdue: #ef5350;
  --color-overdue-bg: #3d1f1f;
  --color-overdue-border: #ff8a80;
}
```

**Icon Selection**:
- ⚠️ Warning triangle - universally recognized for attention/alert
- Alternative: 🔔 Bell icon for notification/reminder
- Must use emoji or SVG (no external icon library)

**Alternatives Considered**:
- **Color only**: Rejected - fails WCAG accessibility
- **Icon only**: Rejected - less immediate than color + icon
- **Background highlight**: Considered but combine with border for emphasis
- **React Icons library**: Rejected - adds dependency for single icon

**Accessibility Checklist**:
- ✅ Color contrast ratio > 4.5:1 for text
- ✅ Non-color indicator (icon/text) present
- ✅ ARIA label for screen readers
- ✅ Focus indicators for keyboard navigation
- ✅ Works in light and dark mode

---

## 3. React Patterns for Derived State

### Decision: Compute overdue status in render (no useState)

**Rationale**:
- Overdue status is derived from todo.dueDate + current date
- Recomputing on each render ensures accuracy (no stale state)
- No synchronization issues between state and props
- Performance impact negligible (simple date comparison)
- Follows React best practices for derived data

**Implementation Pattern**:
```javascript
function TodoCard({ todo }) {
  // Derived state - compute on each render
  const isOverdue = useMemo(() => {
    return calculateIsOverdue(todo.dueDate, todo.completed);
  }, [todo.dueDate, todo.completed]);
  
  const daysOverdue = useMemo(() => {
    return isOverdue ? calculateDaysOverdue(todo.dueDate) : 0;
  }, [isOverdue, todo.dueDate]);
  
  // Render with derived values
  return (
    <div className={isOverdue ? 'todo-card overdue' : 'todo-card'}>
      {/* ... */}
    </div>
  );
}
```

**For OverdueCounter**:
```javascript
function OverdueCounter({ todos }) {
  const overdueCount = useMemo(() => {
    return todos.filter(todo => 
      calculateIsOverdue(todo.dueDate, todo.completed)
    ).length;
  }, [todos]);
  
  if (overdueCount === 0) return null; // Hide when zero
  
  return <div className="overdue-counter">{overdueCount} overdue</div>;
}
```

**Alternatives Considered**:
- **useState for overdue status**: Rejected - requires sync with props changes
- **Redux/Context for overdue**: Rejected - overkill for derived calculation
- **Calculate in parent, pass as prop**: Rejected - violates component encapsulation
- **useEffect to update state**: Rejected - unnecessary side effect for pure calculation

**Performance Considerations**:
- `useMemo` prevents recalculation on unrelated renders
- Date comparison is O(1) operation
- For 100 todos, filtering takes <1ms
- No optimization needed unless 1000+ todos

---

## 4. CSS Design Tokens for Overdue Styling

### Decision: Extend existing theme.css with overdue variables

**Rationale**:
- Project uses CSS custom properties (variables) in theme.css
- Halloween theme established with danger colors
- Maintain consistency with existing design system
- Support light/dark mode switching

**Design Tokens**:
```css
/* Add to theme.css */

/* Light mode overdue colors */
:root {
  /* Existing danger colors repurposed for overdue */
  --color-overdue-text: #c62828;
  --color-overdue-bg: #ffebee;
  --color-overdue-border: #ef5350;
  --color-overdue-icon: #d32f2f;
  
  /* Overdue badge/counter */
  --color-overdue-badge-bg: #c62828;
  --color-overdue-badge-text: #ffffff;
}

/* Dark mode overdue colors */
[data-theme="dark"] {
  --color-overdue-text: #ef5350;
  --color-overdue-bg: #3d1f1f;
  --color-overdue-border: #ff8a80;
  --color-overdue-icon: #ff8a80;
  
  --color-overdue-badge-bg: #ef5350;
  --color-overdue-badge-text: #1a1a1a;
}
```

**CSS Classes**:
```css
/* Overdue TodoCard styling */
.todo-card.overdue {
  border-left: 4px solid var(--color-overdue-border);
  background-color: var(--color-overdue-bg);
}

.overdue-indicator {
  display: flex;
  align-items: center;
  gap: var(--spacing-xs);
  color: var(--color-overdue-text);
  font-size: var(--font-size-caption);
  font-weight: 600;
}

.overdue-icon {
  font-size: 16px;
}

/* Days overdue text */
.overdue-duration {
  color: var(--color-text-secondary);
  font-size: var(--font-size-caption);
  margin-top: var(--spacing-xs);
}

/* Overdue counter in header */
.overdue-counter {
  display: inline-flex;
  align-items: center;
  padding: 4px 12px;
  background-color: var(--color-overdue-badge-bg);
  color: var(--color-overdue-badge-text);
  border-radius: 12px;
  font-size: var(--font-size-caption);
  font-weight: 600;
}
```

**Alternatives Considered**:
- **Inline styles**: Rejected - harder to maintain, no theme support
- **CSS-in-JS (styled-components)**: Rejected - adds dependency, project uses CSS files
- **Tailwind classes**: Rejected - not used in project
- **Separate overdue.css**: Rejected - prefer centralized theme

**Spacing System**:
- Use existing 8px grid: `--spacing-xs`, `--spacing-sm`, etc.
- Icon size: 16px-20px for inline indicators
- Counter padding: 4px 12px for compact badge

---

## 5. Testing Patterns for Date-Dependent Logic

### Decision: Mock Date.now() and test with fixed dates

**Rationale**:
- Date-dependent tests are flaky if using real current date
- Jest provides `jest.spyOn()` to mock Date methods
- Fixed date values ensure reproducible tests
- Test edge cases (today, yesterday, far past, future)

**Testing Pattern**:
```javascript
// dateUtils.test.js
describe('isOverdue', () => {
  beforeEach(() => {
    // Mock Date.now() to return fixed date: 2026-01-15
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

  test('returns false for future date', () => {
    expect(isOverdue('2026-01-16', false)).toBe(false);
  });

  test('returns false for completed todo', () => {
    expect(isOverdue('2026-01-14', true)).toBe(false);
  });

  test('returns false for null due date', () => {
    expect(isOverdue(null, false)).toBe(false);
  });
});

describe('getDaysOverdue', () => {
  beforeEach(() => {
    jest.spyOn(Date, 'now').mockImplementation(() => 
      new Date('2026-01-15T12:00:00Z').getTime()
    );
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  test('calculates 1 day overdue', () => {
    expect(getDaysOverdue('2026-01-14')).toBe(1);
  });

  test('calculates 5 days overdue', () => {
    expect(getDaysOverdue('2026-01-10')).toBe(5);
  });

  test('calculates 0 for today', () => {
    expect(getDaysOverdue('2026-01-15')).toBe(0);
  });
});
```

**Component Testing**:
```javascript
// TodoCard.test.js
import { render, screen } from '@testing-library/react';
import TodoCard from '../TodoCard';

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
    
    render(<TodoCard todo={todo} />);
    
    expect(screen.getByRole('status', { name: /overdue/i })).toBeInTheDocument();
  });

  test('does not display overdue indicator for today', () => {
    const todo = {
      id: 1,
      title: 'Test Todo',
      dueDate: '2026-01-15',
      completed: false
    };
    
    render(<TodoCard todo={todo} />);
    
    expect(screen.queryByRole('status', { name: /overdue/i })).not.toBeInTheDocument();
  });
});
```

**Alternatives Considered**:
- **Real current date**: Rejected - tests fail next day
- **Fixed date in code**: Rejected - inflexible for edge case testing
- **External date mocking library**: Rejected - Jest built-in sufficient
- **Skip date tests**: Rejected - core feature requires testing

**Edge Cases to Test**:
- ✅ Due date = today → not overdue
- ✅ Due date = yesterday → overdue
- ✅ Due date far past (years) → overdue with correct day count
- ✅ Future due date → not overdue
- ✅ Null/undefined due date → not overdue
- ✅ Invalid date string → not overdue
- ✅ Completed + past due → not overdue
- ✅ Midnight boundary → correct calculation

---

## Summary of Decisions

| Research Area | Decision | Key Rationale |
|--------------|----------|---------------|
| Date Comparison | Native Date object with normalization | No dependencies, sufficient for day-level granularity |
| Visual Indicators | Color + Icon + ARIA | WCAG compliant, accessible to all users |
| React State | Derived state with useMemo | No sync issues, follows React best practices |
| CSS Styling | Extend theme.css with CSS variables | Maintains design system consistency, light/dark mode support |
| Testing | Mock Date.now() with fixed dates | Reproducible, covers edge cases |

**Next Steps**: Proceed to Phase 1 (Design & Contracts) with these technology decisions locked in.
