<!--
SYNC IMPACT REPORT - Constitution v1.0.0 (Initial Ratification)

VERSION CHANGE: None → 1.0.0
BUMP RATIONALE: Initial constitution created from existing project documentation

PRINCIPLES DEFINED:
  1. Single Responsibility (from coding-guidelines.md SOLID principles)
  2. Test-Driven Development (from testing-guidelines.md)
  3. DRY & Code Reusability (from coding-guidelines.md)
  4. KISS & Simplicity (from coding-guidelines.md)
  5. Comprehensive Testing Coverage (from testing-guidelines.md)
  6. Error Handling & User Feedback (from coding-guidelines.md)
  7. Code Quality & Standards (from coding-guidelines.md)

TEMPLATES STATUS:
  ✅ plan-template.md - Constitution Check section aligns with principles
  ✅ spec-template.md - Requirements structure supports testability principle
  ✅ tasks-template.md - Task organization reflects testing & modularity principles

FOLLOW-UP TODOS: None - all principles derived from existing documentation
-->

# Todo App Project Constitution

## Core Principles

### I. Single Responsibility Principle

Every module, component, and function MUST have one clear, well-defined responsibility and only one reason to change.

**Rules**:
- React components handle ONLY presentation logic; no direct API calls or business logic
- Functions perform one task well; complex operations are decomposed into smaller, focused functions
- Services handle data access; controllers handle business logic; routes handle HTTP concerns
- File organization reflects responsibility boundaries (components/, services/, utils/, etc.)

**Rationale**: Single responsibility ensures code is maintainable, testable, and easier to understand. Changes to one concern don't ripple through unrelated code.

### II. Test-Driven Development (NON-NEGOTIABLE)

Tests MUST be written as part of the development process to validate and document functionality.

**Rules**:
- Tests describe expected behavior before or alongside implementation
- Test names clearly indicate what is being tested
- Tests follow Arrange-Act-Assert pattern for clarity
- Tests are independent, isolated, and can run in any order
- Mock all external dependencies (API calls, timers, etc.)
- Test behavior, not implementation details

**Rationale**: TDD ensures code correctness, provides living documentation, and enables confident refactoring. Tests catch regressions early and guide design decisions.

### III. DRY (Don't Repeat Yourself)

Code duplication MUST be eliminated through extraction of common functionality into shared utilities, components, or services.

**Rules**:
- Repeated code blocks are extracted into reusable functions or utilities
- UI patterns are extracted into reusable React components
- Common operations (date formatting, API calls, validation) use shared utility modules
- Test fixtures and mock data are centralized to ensure consistency

**Rationale**: DRY reduces maintenance burden, ensures consistency, and minimizes the risk of divergent behavior across similar code paths.

### IV. KISS (Keep It Simple)

Implementations MUST favor simplicity and readability over cleverness or premature optimization.

**Rules**:
- Prefer straightforward solutions that are easy to understand at first glance
- Avoid premature optimization; write clear code first, optimize only when necessary
- Break complex logic into smaller, understandable functions with descriptive names
- Use descriptive variable and function names that clearly indicate purpose
- Avoid single-letter variables except in loops or destructuring

**Rationale**: Simple code is easier to understand, debug, and maintain. Team velocity increases when code is self-documenting and accessible to all skill levels.

### V. Comprehensive Testing Coverage

The codebase MUST maintain 80%+ test coverage across all packages with focus on behavior verification.

**Rules**:
- Unit tests for individual components, functions, and units in isolation
- Integration tests for component interactions and API communication
- Tests are colocated with source code in `__tests__/` directories
- Coverage gaps are identified and addressed before merging
- Test quality over quantity; avoid brittle tests that break with minor refactoring

**Rationale**: High test coverage ensures reliability, catches regressions early, and provides confidence when refactoring or adding features.

### VI. Error Handling & User Feedback

All error-prone operations MUST include graceful error handling with meaningful feedback to users.

**Rules**:
- Try-catch blocks wrap operations that can fail (API calls, parsing, etc.)
- Error messages are clear, actionable, and user-friendly
- Users receive feedback when operations fail (visual indicators, messages)
- Console errors log technical details for debugging; user messages are non-technical
- Guard clauses and default values prevent undefined errors

**Rationale**: Graceful error handling creates a robust user experience and aids debugging. Users should never see cryptic technical errors or be left confused when something fails.

### VII. Code Quality & Standards

All code MUST adhere to established formatting, naming, and organizational standards to ensure consistency across the codebase.

**Rules**:
- 2-space indentation for JavaScript, JSON, CSS, Markdown
- camelCase for variables/functions, PascalCase for components/classes, UPPER_SNAKE_CASE for constants
- Import order: external libraries → internal modules → styles (with blank lines between groups)
- ESLint rules enforced; all linting errors and warnings addressed before merging
- Comments explain "why", not "what"; JSDoc for public functions and components
- No console.log statements left in production code

**Rationale**: Consistent code style reduces cognitive load, improves team collaboration, and makes code reviews more effective. Standards ensure code is professional and maintainable.

## Code Review Requirements

All code changes MUST pass review gates before merging to ensure compliance with constitution principles.

**Pre-Merge Checklist**:
- [ ] Code follows naming conventions (camelCase, PascalCase, UPPER_SNAKE_CASE)
- [ ] Imports are organized correctly (external → internal → styles)
- [ ] No linting errors or warnings
- [ ] Code is DRY and avoids repetition
- [ ] Functions/components have single responsibility
- [ ] Error handling is implemented for error-prone operations
- [ ] Comments are clear and explain "why" not "what"
- [ ] Tests are written for new functionality (unit and/or integration)
- [ ] Test coverage meets 80%+ threshold
- [ ] Git commits are atomic and well-described
- [ ] No console.log statements left in production code

**Pull Request Process**:
- Feature branches used for all new work (e.g., `feature/todo-editing`)
- PRs include descriptive title and summary of changes
- All automated tests pass before review
- At least one team member reviews code before merge
- Constitution violations must be explicitly justified and documented

## Technology Standards

The project MUST adhere to the following technology stack and architectural decisions.

**Stack Requirements**:
- **Frontend**: React with functional components and hooks
- **Backend**: Node.js with Express.js
- **Testing**: Jest for unit and integration tests; React Testing Library for frontend
- **Architecture**: Monorepo structure using npm workspaces
- **Node Version**: v16 or higher
- **Package Manager**: npm v7 or higher

**File Organization**:
- Frontend: `packages/frontend/src/` with components/, services/, utils/, styles/
- Backend: `packages/backend/src/` with routes/, controllers/, services/, middleware/
- Tests: Colocated in `__tests__/` directories near source files
- Documentation: Centralized in `docs/` directory at repository root

**Performance & Scale**:
- Single-user application scope (no multi-user optimization required)
- Desktop-focused UI (no mobile-specific optimization required)
- Backend persistence via Express.js API (in-memory or file-based)
- All changes persisted immediately upon user action

## Governance

This constitution supersedes all other development practices and documentation. Amendments require explicit documentation, team agreement, and version increment.

**Version Management**:
- **MAJOR**: Backward incompatible governance/principle removals or redefinitions
- **MINOR**: New principle/section added or materially expanded guidance
- **PATCH**: Clarifications, wording, typo fixes, non-semantic refinements

**Amendment Process**:
1. Proposed changes documented with rationale
2. Impact analysis performed on existing code and templates
3. Team review and approval (consensus-based)
4. Constitution version incremented per semantic versioning rules
5. Dependent templates and documentation updated for consistency
6. Migration plan created if changes affect existing code

**Compliance Review**:
- All pull requests reviewed for constitution compliance
- Deviations require explicit justification documented in PR description
- Periodic audits ensure ongoing alignment with principles
- Constitution is living document; continuous improvement encouraged

**Guidance Documents**:
- Runtime development guidance: `docs/coding-guidelines.md`, `docs/testing-guidelines.md`
- Functional requirements: `docs/functional-requirements.md`
- UI/UX standards: `docs/ui-guidelines.md`
- Project overview: `docs/project-overview.md`
- GitHub Copilot instructions: `.github/copilot-instructions.md`

**Version**: 1.0.0 | **Ratified**: 2026-01-28 | **Last Amended**: 2026-01-28
