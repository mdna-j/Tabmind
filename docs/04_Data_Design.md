# Tabmind Data Design

## 1. Overview

This document defines how Tabmind will represent, store, retrieve, update, and delete application data.

The first version of Tabmind will not use a traditional database or backend server. Saved data will be stored locally in the user's browser using the Chrome Storage API.

The primary storage mechanism will be:

```text
chrome.storage.local
```

The main data object in Tabmind is a saved tab. Each saved tab represents a webpage the user wants to return to along with the context explaining why it was saved.

The data design should remain simple for the first version while still allowing future features to be added without completely restructuring existing data.

## 2. Design Goals

The Tabmind data model should:

1. Preserve the reason a webpage was saved.
2. Store enough information to identify and reopen the webpage.
3. Support organization through tags and priorities.
4. Support reminders.
5. Support active and completed items.
6. Remain easy to search and filter.
7. Avoid unnecessary data duplication.
8. Remain understandable and easy to maintain.
9. Allow future fields to be added without breaking existing saved data.

## 3. Primary Data Model

The primary data structure will be `SavedTab`.

```ts
export type Priority = "low" | "medium" | "high";

export interface SavedTab {
  id: string;
  title: string;
  url: string;
  reason: string;
  tags: string[];
  priority: Priority;
  createdAt: string;
  updatedAt: string;
  reminderAt: string | null;
  completedAt: string | null;
  summary: string | null;
}
```

Each saved webpage will be represented by one `SavedTab` object.

## 4. Field Definitions

### 4.1 `id`

```ts
id: string;
```

A unique identifier for the saved tab.

The ID allows Tabmind to identify an individual saved item without depending on its URL or title.

This is important because the same URL may potentially be saved more than once for different reasons.

Example:

```text
550e8400-e29b-41d4-a716-446655440000
```

Tabmind should generate IDs using:

```ts
crypto.randomUUID();
```

Example:

```ts
const id = crypto.randomUUID();
```

The ID must not change after the saved tab is created.

### 4.2 `title`

```ts
title: string;
```

The title of the webpage when it was saved.

Example:

```text
FastAPI
```

The title will normally be retrieved automatically from the active Chrome tab.

The title may change on the original website later, but Tabmind will preserve the title that was captured when the page was saved.

### 4.3 `url`

```ts
url: string;
```

The URL of the saved webpage.

Example:

```text
https://github.com/fastapi/fastapi
```

The URL is required because it allows Tabmind to reopen the webpage later.

The URL should be validated before the saved item is created.

### 4.4 `reason`

```ts
reason: string;
```

The reason explains why the user saved the webpage.

Example:

```text
Look at how dependency injection is implemented.
```

This is the most important user-provided field in Tabmind.

For the first version:

- A reason is required.
- A reason cannot contain only whitespace.
- Leading and trailing whitespace should be removed before storage.

Example:

```ts
const reason = input.trim();
```

### 4.5 `tags`

```ts
tags: string[];
```

Tags allow saved tabs to be categorized.

Example:

```ts
["development", "python"]
```

A saved tab may contain zero or more tags.

Tags should be normalized before storage.

For example:

```text
"  Development  "
```

could be stored as:

```text
"development"
```

Tag normalization will help prevent duplicates such as:

```text
Development
development
DEVELOPMENT
```

from being treated as separate tags.

Duplicate tags within the same saved item should be removed.

### 4.6 `priority`

```ts
priority: Priority;
```

Priority indicates how important the saved item is to the user.

Supported values are:

```text
low
medium
high
```

The TypeScript type will be:

```ts
export type Priority = "low" | "medium" | "high";
```

If the user does not select a priority, the default value will be:

```text
medium
```

Using a limited set of values prevents inconsistent values from being stored.

### 4.7 `createdAt`

```ts
createdAt: string;
```

The date and time when the saved tab was created.

Timestamps will be stored using ISO 8601 format.

Example:

```text
2026-10-07T17:30:00.000Z
```

A timestamp can be generated using:

```ts
new Date().toISOString();
```

Storing timestamps in this format provides a consistent representation regardless of the user's local timezone.

The UI can convert the timestamp into the user's local time when displaying it.

### 4.8 `updatedAt`

```ts
updatedAt: string;
```

The date and time when the saved tab was most recently modified.

When the item is first created:

```text
updatedAt = createdAt
```

If the user later changes the reason, tags, priority, reminder, or completion state, `updatedAt` should be updated.

Example:

```ts
updatedAt = new Date().toISOString();
```

This field also provides a foundation for future synchronization features.

### 4.9 `reminderAt`

```ts
reminderAt: string | null;
```

The date and time when the user wants to be reminded about the saved tab.

If no reminder exists:

```ts
reminderAt: null
```

If a reminder exists:

```text
2026-10-10T21:00:00.000Z
```

Reminder timestamps should also use ISO 8601 format.

The user interface will handle converting between local time and the stored timestamp.

A reminder must represent a future time when it is initially created.

### 4.10 `completedAt`

```ts
completedAt: string | null;
```

This field represents whether a saved item has been completed.

An active item will contain:

```ts
completedAt: null
```

When the user marks the item as completed:

```ts
completedAt = new Date().toISOString();
```

Using a timestamp instead of a simple Boolean provides both pieces of information:

- Whether the item is completed
- When the item was completed

For example, instead of:

```ts
completed: true
```

Tabmind can determine completion using:

```ts
const isCompleted = savedTab.completedAt !== null;
```

If the user restores the item:

```ts
completedAt = null;
```

### 4.11 `summary`

```ts
summary: string | null;
```

This field is reserved for future AI-generated summaries.

For the first version:

```ts
summary: null
```

AI functionality is not required for the core Tabmind workflow.

Keeping the field in the model makes the planned relationship between saved pages and future summaries clear, although this field may be reconsidered before AI functionality is implemented.

## 5. Example Saved Tab

A complete saved item may look like:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "FastAPI",
  "url": "https://github.com/fastapi/fastapi",
  "reason": "Look at how dependency injection is implemented.",
  "tags": [
    "development",
    "python"
  ],
  "priority": "high",
  "createdAt": "2026-10-07T17:30:00.000Z",
  "updatedAt": "2026-10-07T17:30:00.000Z",
  "reminderAt": "2026-10-10T21:00:00.000Z",
  "completedAt": null,
  "summary": null
}
```

## 6. Chrome Storage Structure

Tabmind will use `chrome.storage.local`.

The initial storage structure will use a top-level key named:

```text
savedTabs
```

Example:

```json
{
  "savedTabs": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "title": "FastAPI",
      "url": "https://github.com/fastapi/fastapi",
      "reason": "Look at how dependency injection is implemented.",
      "tags": ["development", "python"],
      "priority": "high",
      "createdAt": "2026-10-07T17:30:00.000Z",
      "updatedAt": "2026-10-07T17:30:00.000Z",
      "reminderAt": null,
      "completedAt": null,
      "summary": null
    }
  ]
}
```

For the first version, storing saved tabs as an array keeps the storage model simple and easy to understand.

If the number of saved items or application complexity grows significantly, the storage structure can be reconsidered.

## 7. Storage Key Constants

Storage key names should not be repeated as raw strings throughout the application.

Instead of repeatedly writing:

```ts
chrome.storage.local.get("savedTabs");
```

Tabmind should define storage keys in one location.

For example:

```ts
export const STORAGE_KEYS = {
  SAVED_TABS: "savedTabs",
} as const;
```

Services can then use:

```ts
STORAGE_KEYS.SAVED_TABS
```

This reduces the chance of errors caused by inconsistent storage key names.

## 8. Storage Operations

Storage operations should be handled through the storage service.

The initial service may expose functions such as:

```ts
getSavedTabs(): Promise<SavedTab[]>

getSavedTab(id: string): Promise<SavedTab | null>

saveTab(tab: SavedTab): Promise<void>

updateTab(id: string, updates: Partial<SavedTab>): Promise<void>

deleteTab(id: string): Promise<void>
```

The exact function signatures may change during implementation.

React components should generally use these functions instead of interacting directly with `chrome.storage.local`.

## 9. Creating a Saved Tab

When a new tab is saved, Tabmind should:

1. Retrieve the current page title and URL.
2. Validate the URL.
3. Validate the user's reason.
4. Normalize tags.
5. Determine the selected priority.
6. Validate the optional reminder.
7. Generate a unique ID.
8. Generate creation and update timestamps.
9. Create the `SavedTab` object.
10. Store the object using the storage service.

Conceptually:

```ts
const timestamp = new Date().toISOString();

const savedTab: SavedTab = {
  id: crypto.randomUUID(),
  title,
  url,
  reason: reason.trim(),
  tags: normalizedTags,
  priority,
  createdAt: timestamp,
  updatedAt: timestamp,
  reminderAt,
  completedAt: null,
  summary: null,
};
```

## 10. Updating a Saved Tab

When a saved tab is modified, its `id` and `createdAt` values should remain unchanged.

Editable fields may eventually include:

- Reason
- Tags
- Priority
- Reminder
- Completion state

Every successful modification should update:

```ts
updatedAt
```

For example:

```ts
const updatedTab = {
  ...existingTab,
  reason: newReason.trim(),
  updatedAt: new Date().toISOString(),
};
```

Updates should not accidentally remove fields that were not changed.

## 11. Deleting a Saved Tab

Saved tabs will be deleted using their unique ID.

Example:

```ts
savedTabs.filter((tab) => tab.id !== id);
```

Deleting a saved tab should also remove any Chrome alarm associated with that saved item.

Deletion will be permanent in the first version.

A trash or recovery system is not currently part of the MVP.

## 12. Search Design

The initial search system will operate locally against stored `SavedTab` objects.

Searchable fields should include:

```text
title
url
reason
tags
```

Search should be case-insensitive.

For example, searching:

```text
python
```

should match both:

```text
Python
```

and:

```text
python
```

A normalized query could be created using:

```ts
const query = searchInput.trim().toLowerCase();
```

AI or semantic search is not required for the initial search system.

## 13. Filtering Design

Saved tabs should eventually support filtering by:

### Priority

```text
low
medium
high
```

### Tags

Example:

```text
development
```

### Completion State

An item is active when:

```ts
completedAt === null
```

An item is completed when:

```ts
completedAt !== null
```

Filtering should operate on the stored data without modifying it.

## 14. Sorting

The dashboard should initially support displaying newer saved items before older items.

This can be determined using:

```text
createdAt
```

Future sorting options may include:

- Oldest first
- Recently updated
- Priority
- Reminder date
- Completion date

These additional sorting options are not required for the MVP.

## 15. Tag Normalization

Tags should be normalized before storage to avoid unnecessary duplicates.

A basic normalization process can:

1. Trim whitespace.
2. Convert the tag to lowercase.
3. Remove empty values.
4. Remove duplicate tags.

For example:

```text
[" Python ", "Development", "python", ""]
```

becomes:

```text
["python", "development"]
```

A helper function may eventually handle this behavior.

Example:

```ts
normalizeTags(tags: string[]): string[]
```

## 16. URL Validation

Tabmind should validate URLs before saving them.

For the initial version, normal webpages using the following protocols should be supported:

```text
http:
https:
```

Some Chrome internal pages cannot be accessed or reopened by extensions in the same way as normal webpages.

Unsupported URLs should not be saved as normal Tabmind entries unless support is intentionally added later.

## 17. Reminder Data

The `SavedTab` object stores the reminder time:

```ts
reminderAt: string | null;
```

Chrome will separately maintain the actual scheduled alarm through `chrome.alarms`.

The saved data and Chrome alarm should remain consistent.

A predictable alarm name should connect an alarm to a saved tab.

For example:

```text
tabmind-reminder:<savedTabId>
```

Example:

```text
tabmind-reminder:550e8400-e29b-41d4-a716-446655440000
```

This makes it possible to determine which saved tab belongs to an alarm event.

## 18. Data Validation

Before storing a new saved tab, Tabmind should verify that:

- `id` exists.
- `title` is a string.
- `url` is valid and supported.
- `reason` is not empty after trimming.
- `tags` is an array.
- `priority` contains a supported value.
- `createdAt` is a valid timestamp.
- `updatedAt` is a valid timestamp.
- `reminderAt` is either a valid timestamp or `null`.
- `completedAt` is either a valid timestamp or `null`.
- `summary` is either a string or `null`.

Validation should occur at application boundaries rather than assuming all stored data is always valid.

This becomes especially important if the data structure changes in future versions.

## 19. Data Migration

The structure of `SavedTab` may change as Tabmind develops.

For example, a future version might add:

```ts
collectionId
faviconUrl
archivedAt
aiMetadata
```

Existing users may already have saved data that does not contain those fields.

Tabmind should avoid assuming that every stored object automatically matches the newest TypeScript interface.

If the data model changes after release, a migration process should convert older saved data into the current format.

A future storage version may be introduced:

```json
{
  "schemaVersion": 1,
  "savedTabs": []
}
```

The first development version does not require a full migration system, but the architecture should leave room for one.

## 20. Storage Limitations

`chrome.storage.local` is appropriate for the first version because Tabmind primarily stores small pieces of structured text.

Examples include:

- URLs
- Titles
- Reasons
- Tags
- Timestamps

Tabmind should avoid storing large amounts of full webpage content directly inside each saved item unless there is a clear need.

Future AI functionality may require a different approach if large amounts of extracted webpage content need to be stored or processed.

## 21. Privacy

Saved-tab information may reveal what webpages a user has visited and why they considered those pages important.

For that reason, Tabmind should treat saved-tab data as private user information.

For the initial version:

- Saved data remains in `chrome.storage.local`.
- No Tabmind account is required.
- Saved data is not sent to a Tabmind server.
- Reasons and tags are not transmitted externally.

If future AI functionality sends webpage content or user-written reasons to an external AI provider, that behavior must be clearly documented before the feature is implemented.

## 22. Future Data Models

Future features may require additional models.

Possible examples include:

```text
Collection
UserPreferences
ReminderSettings
SyncMetadata
AIResult
```

These models should only be introduced when the corresponding feature is being designed or implemented.

For example, collections may eventually use a model similar to:

```ts
interface Collection {
  id: string;
  name: string;
  createdAt: string;
}
```

This is not part of the current data model and should not be implemented for the MVP.

## 23. Initial Data Design Decisions

The first version of Tabmind will follow these decisions:

| Decision | Choice |
|---|---|
| Storage | `chrome.storage.local` |
| Main model | `SavedTab` |
| ID format | UUID |
| ID generation | `crypto.randomUUID()` |
| Timestamp format | ISO 8601 |
| Default priority | `medium` |
| Required reason | Yes |
| Tags | Normalized lowercase strings |
| Reminder | Optional |
| Completion tracking | `completedAt` timestamp |
| Initial sorting | Newest first |
| AI summary | Optional future field |
| Backend | None |
| User accounts | None |
| Cloud sync | None |

These decisions may change if implementation reveals a better approach, but changes should be reflected in this document.

## 24. Final MVP Data Model

The initial Tabmind model will therefore be:

```ts
export type Priority = "low" | "medium" | "high";

export interface SavedTab {
  id: string;
  title: string;
  url: string;
  reason: string;
  tags: string[];
  priority: Priority;
  createdAt: string;
  updatedAt: string;
  reminderAt: string | null;
  completedAt: string | null;
  summary: string | null;
}
```

This model contains enough information to support the core Tabmind workflow while leaving room for later features.