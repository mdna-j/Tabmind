# Tabmind Software Requirements Specification

## 1. Introduction

### 1.1 Purpose

This Software Requirements Specification defines the functional and non-functional requirements for Tabmind.

Tabmind is a Chrome extension that allows users to save webpages along with the reason they want to revisit them. The extension is intended to provide more context than a traditional bookmark by allowing saved pages to include information such as tags, priority, reminders, and completion status.

This document focuses primarily on the requirements for the first version of Tabmind.

### 1.2 Scope

The first version of Tabmind will allow users to:

- Save the currently active browser tab
- Record a reason for saving the page
- Automatically capture the page title and URL
- Add tags
- Assign a priority
- Set an optional reminder
- View saved tabs through a dashboard
- Search and filter saved tabs
- Reopen saved webpages
- Mark saved tabs as completed
- Delete saved tabs
- Store saved data locally

The first version will not require:

- User accounts
- A backend server
- Cloud storage
- Cross-device synchronization
- AI functionality
- Team collaboration

These features may be introduced in future versions.

## 2. Product Overview

### 2.1 Product Perspective

Tabmind will operate as a Chrome extension using Chrome Manifest V3.

The user interface will be built using React and TypeScript. Vite will be used for development and building the extension.

The extension will contain several major components:

- Popup interface
- Dashboard interface
- Background service worker
- Content script
- Storage services
- Reminder services
- Shared TypeScript types and utilities

The first version will store user data using `chrome.storage.local`.

### 2.2 Primary User

The primary user is someone who regularly keeps browser tabs open because they want to return to the content later.

The user is expected to have a basic understanding of browser extensions but should not need technical knowledge to use Tabmind.

### 2.3 Operating Environment

The initial version of Tabmind will target:

- Google Chrome
- Chrome Manifest V3
- Desktop operating systems supported by Chrome

Firefox and other browsers are outside the scope of the first version.

## 3. Functional Requirements

### FR-001: Read Current Tab

**Description**

Tabmind shall retrieve information about the currently active browser tab when the user opens the save interface.

**Required information**

- Page title
- Page URL

**Acceptance Criteria**

- The active tab is detected automatically.
- The page title is displayed to the user.
- The page URL is captured automatically.
- The user does not need to manually copy and paste the URL.

---

### FR-002: Save Current Tab

**Description**

Tabmind shall allow the user to save the currently active browser tab.

**Acceptance Criteria**

- A saved tab receives a unique identifier.
- The page title is stored.
- The page URL is stored.
- The reason is stored.
- The creation date and time are stored.
- The saved item persists after the popup is closed.
- The saved item persists after Chrome is restarted.

---

### FR-003: Add a Reason

**Description**

Tabmind shall allow the user to record why they want to return to a webpage.

The reason represents the main contextual information associated with a saved tab.

**Acceptance Criteria**

- A reason can be entered before saving.
- A reason cannot contain only whitespace.
- The reason is stored with the saved tab.
- The reason is displayed when viewing the saved tab later.

For the first version, a reason shall be required when saving a tab.

---

### FR-004: Assign Tags

**Description**

Tabmind shall allow users to assign tags to saved tabs.

Tags provide a way to group related saved content.

Examples include:

- Development
- School
- Research
- Work
- Shopping
- Travel

**Acceptance Criteria**

- A saved tab can contain zero or more tags.
- Multiple tags can be assigned to one saved tab.
- Tags are stored with the saved tab.
- Tags are visible from the dashboard.

---

### FR-005: Assign Priority

**Description**

Tabmind shall allow users to assign a priority to a saved tab.

Supported priority levels for the first version are:

- Low
- Medium
- High

**Acceptance Criteria**

- The user can select a priority before saving.
- The selected priority is stored.
- The priority is visible from the dashboard.
- A default priority is assigned if the user does not manually select one.

The initial default priority will be `Medium`.

---

### FR-006: Set Reminder

**Description**

Tabmind shall allow users to optionally schedule a reminder for a saved tab.

**Acceptance Criteria**

- A reminder is optional.
- The user can select a future date and time.
- Past reminder times cannot be scheduled.
- The reminder is associated with the correct saved tab.
- The reminder remains scheduled when the popup is closed.

---

### FR-007: Trigger Reminder Notification

**Description**

Tabmind shall notify the user when a scheduled reminder becomes due.

**Acceptance Criteria**

- Tabmind uses `chrome.alarms` to schedule reminders.
- The reminder does not depend on the popup remaining open.
- A browser notification is created when the reminder becomes due.
- The notification provides enough information to identify the saved tab.

---

### FR-008: View Saved Tabs

**Description**

Tabmind shall provide a dashboard where users can view their saved tabs.

**Acceptance Criteria**

Each saved item shall display enough information for the user to identify it.

This may include:

- Page title
- Reason
- Tags
- Priority
- Date saved
- Reminder
- Completion status

The exact dashboard layout will be defined in the UI/UX design documentation.

---

### FR-009: Reopen Saved Tab

**Description**

Tabmind shall allow the user to reopen a previously saved webpage.

**Acceptance Criteria**

- A saved item provides an action for reopening the page.
- Selecting the action opens the stored URL in Chrome.

---

### FR-010: Search Saved Tabs

**Description**

Tabmind shall allow users to search their saved tabs.

The initial search should support matching against:

- Page title
- Reason
- URL
- Tags

**Acceptance Criteria**

- Search results update based on the user's search query.
- Matching should not require exact capitalization.
- Clearing the search restores the complete applicable list.

---

### FR-011: Filter Saved Tabs

**Description**

Tabmind shall allow users to filter saved tabs.

Initial filters should include:

- Priority
- Tags
- Completion status

Reminder-related filtering may be added if needed during dashboard development.

**Acceptance Criteria**

- Users can apply supported filters.
- Only matching saved tabs are displayed.
- Filters do not modify or delete stored data.
- Users can clear active filters.

---

### FR-012: Mark Tab as Completed

**Description**

Tabmind shall allow users to mark a saved tab as completed.

**Acceptance Criteria**

- An active saved tab can be marked as completed.
- The completion date and time are recorded.
- Completed items remain stored unless the user deletes them.
- Completion status can be used when filtering saved tabs.

---

### FR-013: Restore Completed Tab

**Description**

Tabmind shall allow a completed saved tab to be returned to active status.

**Acceptance Criteria**

- A completed tab can be marked as active again.
- Its completion timestamp is removed.
- The original saved information remains unchanged.

---

### FR-014: Delete Saved Tab

**Description**

Tabmind shall allow users to permanently delete a saved tab.

**Acceptance Criteria**

- The selected saved tab is removed from local storage.
- Other saved tabs are not affected.
- Any reminder associated with the deleted item is removed.
- The deleted item no longer appears in the dashboard.

---

### FR-015: Persist Saved Data

**Description**

Tabmind shall store saved-tab data using `chrome.storage.local`.

**Acceptance Criteria**

- Data remains available after the popup is closed.
- Data remains available after the dashboard is closed.
- Data remains available after Chrome is restarted.
- Updating one saved tab does not overwrite unrelated saved tabs.

---

### FR-016: Prevent Invalid Saves

**Description**

Tabmind shall validate required information before creating a saved tab.

**Acceptance Criteria**

Tabmind shall reject a save if:

- A valid URL is unavailable.
- A required reason has not been entered.
- The reason contains only whitespace.

The user shall receive feedback explaining why the tab could not be saved.

## 4. Non-Functional Requirements

### NFR-001: Usability

The core save workflow should require as little effort as reasonably possible.

A user should be able to open Tabmind, enter a reason, and save the current page without navigating through multiple screens.

### NFR-002: Performance

Opening the popup should feel immediate during normal use.

Searching and filtering locally stored tabs should update without noticeable delays for normal amounts of saved data.

### NFR-003: Reliability

Tabmind should not lose existing saved data because one save, update, or reminder operation fails.

Storage operations should handle failures without corrupting unrelated saved items.

### NFR-004: Maintainability

The codebase should separate:

- React UI components
- Chrome API interactions
- Storage operations
- Reminder logic
- Shared types
- Shared utilities

Business logic should not be unnecessarily embedded inside React components.

### NFR-005: Type Safety

TypeScript shall be used throughout the application where practical.

Shared data structures should use common TypeScript interfaces or types.

### NFR-006: Privacy

The first version of Tabmind shall store saved-tab information locally using `chrome.storage.local`.

Tabmind shall not require users to create an account.

Data should not be transmitted to an external service unless a future feature explicitly requires it and the user is informed.

### NFR-007: Security

The extension should request only the Chrome permissions necessary for its implemented functionality.

Secrets such as third-party API keys shall not be hardcoded into the source code.

### NFR-008: Accessibility

Interactive UI elements should support keyboard navigation where practical.

Forms and controls should use meaningful labels and appropriate HTML elements.

Text should maintain sufficient readability and contrast.

### NFR-009: Testability

Core application logic should be structured so that it can be tested independently from the UI when possible.

Vitest will be used for unit testing, while React Testing Library will be used for React component behavior.

## 5. Data Requirements

Each saved tab will require a consistent data structure.

The initial model is expected to contain:

```ts
interface SavedTab {
  id: string;
  title: string;
  url: string;
  reason: string;
  tags: string[];
  priority: "low" | "medium" | "high";
  createdAt: string;
  reminderAt: string | null;
  completedAt: string | null;
  summary: string | null;
}
```

The final structure will be defined in:

`04_Data_Design.md`

The `summary` field is included to allow future AI functionality, but it is not required for the first version.

## 6. External Interface Requirements

### 6.1 Chrome Tabs API

Tabmind will use Chrome APIs to retrieve information about the active browser tab and reopen saved webpages.

### 6.2 Chrome Storage API

`chrome.storage.local` will be used to persist saved data.

### 6.3 Chrome Alarms API

`chrome.alarms` will be used to schedule reminders.

### 6.4 Chrome Notifications API

`chrome.notifications` will be used to display reminder notifications.

### 6.5 User Interface

The primary interfaces will be:

- Extension popup
- Full dashboard

Both interfaces will be implemented using React and TypeScript.

## 7. Constraints

The first version of Tabmind will operate under the following constraints:

- Chrome Manifest V3 will be used.
- The initial release will target Google Chrome.
- Saved data will remain local to the browser.
- No user authentication will be required.
- No backend will be required.
- Core functionality cannot depend on AI.
- Chrome extension permissions should be kept to the minimum necessary.

## 8. Future Requirements

The following are possible future requirements and are not part of the initial version:

- AI-generated summaries
- AI-generated tag suggestions
- Natural language search
- Duplicate URL detection
- Reminder snoozing
- Tab collections
- Bulk tab saving
- Bookmark importing
- Cross-device synchronization
- User accounts
- Shared collections
- Firefox support
- Third-party integrations

These features should not delay development of the core Tabmind workflow.

## 9. Requirement Priorities

Requirements are divided into three priority levels.

### Must Have

Required for the first usable version:

- Read current tab
- Save current tab
- Record a reason
- Persist saved data
- View saved tabs
- Reopen saved tabs
- Delete saved tabs
- Basic search
- Input validation

### Should Have

Important features that can follow the basic save and retrieval workflow:

- Tags
- Priority
- Filtering
- Completion status
- Reminders
- Browser notifications

### Could Have

Features that can be introduced after the core extension is stable:

- Duplicate detection
- Snoozing
- Tab collections
- Bulk saving
- AI summaries
- AI tag suggestions
- Natural language search

## 10. Definition of MVP

The Tabmind MVP is complete when a user can:

1. Open the extension while viewing a webpage.
2. See the current page title.
3. Enter a reason for saving the page.
4. Save the page.
5. Close the original browser tab.
6. Open the Tabmind dashboard later.
7. Find the saved page.
8. See the reason they originally saved it.
9. Reopen the webpage.
10. Delete the saved item when it is no longer needed.

Tags, priorities, reminders, and AI can improve this workflow, but they are not required to prove that the core Tabmind idea works.