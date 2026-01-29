# Implementation Plan: Support for Overdue Todo Items

**Branch**: `001-overdue-todos` | **Date**: January 29, 2026 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-overdue-todos/spec.md`

## Summary

Add visual indicators and counters to help users quickly identify incomplete todos that have passed their due date. The feature compares each incomplete todo's due date against the current date (client-side) and applies color + icon styling to overdue items. A summary counter in the header displays the total number of overdue items, and individual items show how many days they are overdue.

## Technical Context

**Language/Version**: JavaScript (ES6+) / Node.js v16+  
**Primary Dependencies**: React 18.2, Express.js 4.18, Jest 29.7, React Testing Library  
**Storage**: SQLite (better-sqlite3) - existing backend persistence  
**Testing**: Jest for unit/integration tests; React Testing Library for component tests  
**Target Platform**: Desktop web browsers (Chrome, Firefox, Safari, Edge)  
**Project Type**: Web application (monorepo with frontend + backend packages)  
**Performance Goals**: <1s overdue status calculation, <2s visual identification, 60 fps UI rendering  
**Constraints**: Client-side date calculations only, no backend changes required, 80%+ test coverage  
**Scale/Scope**: Single-user application, ~10-100 todos typical use case, desktop-focused UI

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Principle I: Single Responsibility ✅ PASS
- Overdue calculation logic will be isolated in utility functions
- Visual styling will be handled by presentation components
- Date comparison logic separated from UI rendering
- No violations expected

### Principle II: Test-Driven Development ✅ PASS
- Tests will be written for overdue calculation logic
- Component tests for visual indicators and counter display
- Integration tests for date boundary scenarios
- All acceptance criteria are testable

### Principle III: DRY (Don't Repeat Yourself) ✅ PASS
- Overdue calculation logic centralized in shared utility
- Date formatting reused from existing utilities
- Visual styling via reusable CSS classes
- No code duplication expected

### Principle IV: KISS (Keep It Simple) ✅ PASS
- Straightforward date comparison logic
- Simple counter calculation (filter + count)
- No complex algorithms or optimizations needed
- Clear, readable implementation

### Principle V: Comprehensive Testing Coverage ✅ PASS
- Unit tests for date calculation utilities
- Component tests for TodoCard visual indicators
- Component tests for OverdueCounter
- Integration tests for App-level behavior
- Target: 80%+ coverage maintained

### Principle VI: Error Handling & User Feedback ✅ PASS
- Graceful handling of missing/invalid due dates
- No user-facing errors expected (pure calculation)
- System handles edge cases (far past dates, time zones)
- No API calls, so no network error handling needed

### Principle VII: Code Quality & Standards ✅ PASS
- 2-space indentation, camelCase naming maintained
- ESLint compliance enforced
- Imports organized per standards
- JSDoc comments for utility functions
- No console.log in production code

**Gate Status**: ✅ ALL GATES PASS - Proceed to Phase 0

## Project Structure

### Documentation (this feature)

```text
specs/001-overdue-todos/
├── spec.md              # Feature specification (completed)
├── plan.md              # This file (in progress)
├── research.md          # Phase 0 output (to be created)
├── data-model.md        # Phase 1 output (to be created)
├── quickstart.md        # Phase 1 output (to be created)
├── contracts/           # Phase 1 output (to be created)
│   └── overdue-api.md   # Client-side utility contracts
├── checklists/
│   └── requirements.md  # Requirements validation (completed)
└── tasks.md             # Phase 2 output (created by /speckit.tasks)
```

### Source Code (repository root)

```text
packages/
├── frontend/
│   └── src/
│       ├── components/
│       │   ├── TodoCard.js              # [MODIFY] Add overdue visual indicators
│       │   ├── TodoList.js              # [MODIFY] Add OverdueCounter integration
│       │   ├── OverdueCounter.js        # [NEW] Summary counter component
│       │   └── __tests__/
│       │       ├── TodoCard.test.js     # [MODIFY] Add overdue tests
│       │       ├── TodoList.test.js     # [MODIFY] Add counter tests
│       │       └── OverdueCounter.test.js # [NEW] Counter unit tests
│       ├── utils/
│       │   ├── dateUtils.js             # [NEW] Overdue calculation utilities
│       │   └── __tests__/
│       │       └── dateUtils.test.js    # [NEW] Date utility tests
│       ├── styles/
│       │   └── theme.css                # [MODIFY] Add overdue color variables
│       └── App.js                       # [MODIFY] Pass overdue data to components
│
└── backend/
    └── src/
        └── services/
            └── todoService.js           # [NO CHANGES] Backend unchanged
```

**Structure Decision**: Web application structure (Option 2) selected. Frontend-only changes for client-side overdue detection. Backend remains unchanged as all date calculations happen client-side using existing todo data (dueDate and completed fields).

---

## Phase 0: Research & Technology Decisions

### Research Areas Identified

From Technical Context unknowns and feature requirements:
1. Best practices for client-side date comparison in JavaScript
2. Accessibility patterns for visual indicators (color + icon)
3. React patterns for derived state (overdue calculation)
4. CSS design tokens for overdue styling in existing theme
5. Testing patterns for date-dependent logic

**Research Output**: [research.md](./research.md) - All decisions documented with rationale

---

## Phase 1: Design & Contracts

### Data Model

**Output**: [data-model.md](./data-model.md)

**Summary**:
- No database schema changes required
- Overdue status derived from existing `dueDate` and `completed` fields
- Client-side calculation using native JavaScript Date API
- Three derived properties: `isOverdue`, `daysOverdue`, `overdueCount`

### API Contracts

**Output**: [contracts/client-utils.md](./contracts/client-utils.md)

**Summary**:
- **No backend API changes** - all logic client-side
- Utility function contracts defined:
  - `isOverdue(dueDate, completed)` → boolean
  - `getDaysOverdue(dueDate)` → number
  - `normalizeToStartOfDay(date)` → Date
  - `formatDaysOverdue(days)` → string
- Comprehensive edge case handling documented
- Performance: O(1) per todo, O(n) for counter

### Quickstart Guide

**Output**: [quickstart.md](./quickstart.md)

**Summary**:
- 6-phase implementation guide (4-6 hours total)
- Step-by-step instructions with code examples
- Test-driven approach with test code provided
- Verification checklist and troubleshooting guide

### Agent Context Update

**Status**: ✅ Completed - GitHub Copilot instructions updated

---

## Post-Phase 1 Constitution Check

*Re-evaluation after design completion*

### Principle I: Single Responsibility ✅ PASS
- `dateUtils.js` handles only date calculations
- `OverdueCounter.js` handles only counter display
- `TodoCard.js` enhanced with focused overdue display logic
- Clear separation maintained

### Principle II: Test-Driven Development ✅ PASS
- Comprehensive test suites defined in quickstart
- All utility functions have unit tests
- All components have integration tests
- Mocked Date.now() for reproducible tests

### Principle III: DRY ✅ PASS
- Date calculation logic centralized in `dateUtils.js`
- Reusable across TodoCard and OverdueCounter
- CSS variables for consistent styling
- No code duplication in design

### Principle IV: KISS ✅ PASS
- Simple date comparison logic (no complex algorithms)
- Straightforward component integration
- Native APIs only (no external dependencies)
- Clear, readable implementations

### Principle V: Comprehensive Testing Coverage ✅ PASS
- Unit tests for all utility functions
- Component tests for TodoCard and OverdueCounter
- Edge case coverage documented
- Target: 80%+ coverage maintainable

### Principle VI: Error Handling & User Feedback ✅ PASS
- Graceful handling of invalid dates (return safe defaults)
- No user-facing errors (pure calculations)
- Screen reader support via ARIA attributes
- Visual feedback through icons and colors

### Principle VII: Code Quality & Standards ✅ PASS
- JSDoc comments for all utility functions
- Consistent naming conventions followed
- Import organization per standards
- CSS follows existing theme structure

**Final Gate Status**: ✅ ALL GATES PASS - Design approved, ready for implementation

---

## Implementation Summary

### Files to Create (7 new files)

1. **`packages/frontend/src/utils/dateUtils.js`** - Date calculation utilities
2. **`packages/frontend/src/utils/__tests__/dateUtils.test.js`** - Utility tests
3. **`packages/frontend/src/components/OverdueCounter.js`** - Counter component
4. **`packages/frontend/src/components/__tests__/OverdueCounter.test.js`** - Counter tests

### Files to Modify (4 existing files)

5. **`packages/frontend/src/components/TodoCard.js`** - Add overdue indicators
6. **`packages/frontend/src/components/__tests__/TodoCard.test.js`** - Add overdue tests
7. **`packages/frontend/src/components/TodoList.js`** - Integrate OverdueCounter
8. **`packages/frontend/src/styles/theme.css`** - Add overdue CSS variables

### Files Unchanged

- Backend (`packages/backend/**`) - No changes required
- API contracts - No endpoint modifications
- Database schema - No migrations needed

### Complexity Metrics

- **Lines of Code (estimated)**: ~300 lines (utilities + components + tests)
- **Test Coverage**: 80%+ (15+ test cases defined)
- **Performance Impact**: Negligible (<1ms per todo)
- **Bundle Size Impact**: 0 KB (no dependencies added)

---

## Phase 2: Task Breakdown

**Note**: This phase is handled by the `/speckit.tasks` command, not `/speckit.plan`.

Task breakdown will include:
- Atomic, testable tasks for each file creation/modification
- Estimated effort per task
- Dependencies and ordering
- Acceptance criteria aligned with spec

**Command to generate**: `/speckit.tasks`

---

## Appendix: Technology Stack Summary

### Frontend
- **React**: 18.2.0 (functional components, hooks, useMemo)
- **Testing**: Jest 29.7, React Testing Library
- **Styling**: CSS custom properties (variables), no CSS-in-JS
- **State**: Component state (no Redux/Context for this feature)

### Backend
- **Status**: No changes required
- **Storage**: SQLite (better-sqlite3) - existing

### Development
- **Node.js**: v16+
- **Package Manager**: npm (workspaces)
- **Monorepo Structure**: `packages/frontend`, `packages/backend`

### Browser Support
- **Target**: Modern desktop browsers (Chrome, Firefox, Safari, Edge)
- **JavaScript**: ES6+ features used (useMemo, arrow functions, template literals)
- **CSS**: CSS Variables (custom properties) - IE11 not supported

---

## Sign-Off

**Planning Status**: ✅ COMPLETE  
**Phase 0 (Research)**: ✅ COMPLETE  
**Phase 1 (Design & Contracts)**: ✅ COMPLETE  
**Phase 2 (Tasks)**: ⏳ Pending `/speckit.tasks` command  

**Ready for Implementation**: ✅ YES

All technical decisions documented, constitution compliance verified, and implementation path clear. Proceed to task breakdown with `/speckit.tasks`.
