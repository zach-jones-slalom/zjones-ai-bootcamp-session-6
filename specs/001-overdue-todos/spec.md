# Feature Specification: Support for Overdue Todo Items

**Feature Branch**: `001-overdue-todos`  
**Created**: January 29, 2026  
**Status**: Draft  
**Input**: User description: "Support for Overdue Todo Items - Users need a clear, visual way to identify which todos have not been completed by their due date"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Visual Identification of Overdue Items (Priority: P1)

Users with incomplete tasks past their due date need to immediately identify which todos are overdue when viewing their todo list. The system compares each incomplete todo's due date against the current date and applies distinct visual indicators to overdue items.

**Why this priority**: This is the core value proposition - without visual distinction of overdue items, users must manually compare dates, defeating the purpose of the feature. This delivers immediate value by solving the primary user pain point.

**Independent Test**: Can be fully tested by creating todos with past due dates and verifying they display with distinct visual indicators (e.g., different color, icon, or styling) compared to non-overdue items. Delivers immediate value by making overdue items instantly recognizable.

**Acceptance Scenarios**:

1. **Given** a user has an incomplete todo with a due date of yesterday or earlier, **When** they view their todo list, **Then** that todo displays with a distinct visual indicator showing it is overdue
2. **Given** a user has a completed todo with a past due date, **When** they view their todo list, **Then** that todo does NOT display as overdue (completion status takes precedence)
3. **Given** a user has an incomplete todo with a due date of today, **When** they view their todo list at any time during that day, **Then** that todo does NOT display as overdue (due today is not yet overdue)
4. **Given** a user has an incomplete todo with no due date set, **When** they view their todo list, **Then** that todo does NOT display as overdue (items without due dates cannot be overdue)
5. **Given** a user has an incomplete todo with a future due date, **When** they view their todo list, **Then** that todo does NOT display as overdue

---

### User Story 2 - Overdue Count Summary (Priority: P2)

Users need to see at a glance how many overdue items they have without counting individual items in the list. A summary counter displays the total number of overdue todos.

**Why this priority**: Provides quick awareness of workload backlog and helps users prioritize their time. This is secondary to the visual identification (P1) but adds significant value for users with many todos.

**Independent Test**: Can be tested by creating multiple todos with various due dates and verifying the counter accurately reflects the number of incomplete todos with past due dates. Works independently as a standalone feature.

**Acceptance Scenarios**:

1. **Given** a user has 3 incomplete todos with past due dates and 2 incomplete todos with future due dates, **When** they view their todo list, **Then** an overdue counter displays "3 overdue"
2. **Given** a user has no incomplete todos with past due dates, **When** they view their todo list, **Then** the overdue counter displays "0 overdue" or is hidden
3. **Given** a user marks an overdue todo as complete, **When** the list updates, **Then** the overdue counter decrements by 1
4. **Given** the current date advances and a todo's due date becomes past, **When** the user refreshes or views their list, **Then** the overdue counter increments by 1

---

### User Story 3 - Overdue Duration Display (Priority: P3)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value and why it has this priority level]

Users want to know not just that an item is overdue, but how long it has been overdue. The system calculates and displays the number of days overdue for each overdue todo item.

**Why this priority**: Enhances awareness of priority by showing which items are most severely overdue. This is a nice-to-have enhancement that provides additional context but is not essential for the core functionality.

**Independent Test**: Can be tested by creating todos with various past due dates and verifying each displays the correct number of days overdue (e.g., "2 days overdue"). Adds value independently as contextual information.

**Acceptance Scenarios**:

1. **Given** a user has an incomplete todo with a due date of 5 days ago, **When** they view that todo, **Then** it displays "5 days overdue"
2. **Given** a user has an incomplete todo with a due date of yesterday, **When** they view that todo, **Then** it displays "1 day overdue"
3. **Given** a user has an incomplete todo with a due date of today, **When** they view that todo at any time during that day, **Then** it does NOT display any overdue duration (not yet overdue)
4. **Given** an overdue todo displays a duration, **When** the user marks it complete, **Then** the overdue duration is no longer displayed

---

### Edge Cases

- What happens when the system date/time changes (e.g., user travels across time zones or changes system clock)?
- How does the system handle todos with due dates far in the past (e.g., years ago)?
- What happens when a completed todo's due date was in the past - should historical overdue status be visible?
- How does the system handle date calculations at midnight (boundary between today and tomorrow)?
- What happens if a user has a very large number of overdue items (hundreds)?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST compare each incomplete todo's due date against the current date to determine overdue status
- **FR-002**: System MUST apply distinct visual styling to incomplete todos with due dates before the current date
- **FR-003**: System MUST NOT mark completed todos as overdue regardless of their due date
- **FR-004**: System MUST NOT mark todos without due dates as overdue
- **FR-005**: System MUST treat todos with a due date of "today" as NOT overdue during that calendar day
- **FR-006**: System MUST recalculate overdue status dynamically when the date changes (e.g., at midnight or when user refreshes)
- **FR-007**: System MUST display a count of the total number of overdue todos
- **FR-008**: System MUST update the overdue count immediately when a todo's status changes (completed/uncompleted) or when due dates are modified
- **FR-009**: System MUST calculate and display the number of days between the due date and current date for overdue items
- **FR-010**: System MUST update overdue duration calculations when the current date changes

### Key Entities *(include if feature involves data)*

- **Todo**: Existing entity with attributes including title, due date (optional), completion status, and created date. The overdue status is derived from comparing due date to current date and checking completion status.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can identify overdue todos within 2 seconds of viewing their list without reading dates
- **SC-002**: 95% of users correctly identify which todos are overdue based on visual indicators alone
- **SC-003**: Overdue status updates accurately within 1 second when a todo is marked complete or incomplete
- **SC-004**: The overdue counter displays the correct count with 100% accuracy across all scenarios
- **SC-005**: Users report reduced time spent managing overdue tasks by at least 30% (measured through user feedback or task completion metrics)

## Assumptions

- The system has access to accurate current date/time information
- Users understand that "overdue" means the due date has passed and the item is incomplete
- Visual distinction will follow the existing UI design system and color palette
- Date calculations use local time zone of the user's system
- The existing todo list already displays due dates, so users are familiar with date information
- "Today" is defined as the current calendar day in the user's local time zone (00:00:00 to 23:59:59)
- Overdue calculations happen client-side based on the user's system date/time
