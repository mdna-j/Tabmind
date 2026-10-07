# Tabmind API Specification

## 1. Overview

This document defines the internal application interfaces and external browser APIs used by Tabmind.

The first version of Tabmind does not use a backend server or public REST API. Instead, the application communicates with Chrome through the Chrome Extension APIs and exposes reusable TypeScript service functions to the rest of the application.

The main internal services are:

- Tab Service
- Storage Service
- Reminder Service

These services provide an abstraction between the user interface and Chrome-specific functionality.

The general structure is:

```text
React UI
   │
   ▼
Internal Services
   │
   ▼
Chrome Extension APIs
```

This allows React components to use application-level functions instead of directly implementing Chrome API operations throughout the UI.

---

## 2. API Design Goals

The internal APIs should:

1. Keep Chrome API logic separate from React components.
2. Provide reusable functions for common operations.
3. Use TypeScript for type safety.
4. Return predictable values.
5. Handle errors consistently.
6. Remain simple enough for the first version.
7. Be easy to test.
8. Allow the underlying implementation to change without requiring major UI changes.

The API layer should not introduce unnecessary abstractions.

---

## 3. Service Overview

The initial service layer will contain:

```text
src/
└── services/
    ├── storage.ts
    ├── tabs.ts
    └── reminders.ts
```

Each service has a specific responsibility.

| Service | Responsibility |
|---|---|
| `storage.ts` | Read and modify saved Tabmind data |
| `tabs.ts` | Interact with browser tabs |
| `reminders.ts` | Create and remove reminders |

Additional services should only be introduced when a feature requires them.

---

# 4. Storage Service

## 4.1 Purpose

The Storage Service provides access to saved Tabmind data.

It acts as the main interface between the application and:

```text
chrome.storage.local
```

React components should generally not call `chrome.storage.local` directly.

Instead, they should use functions provided by the Storage Service.

Example:

```text
Dashboard
    │
    ▼
getSavedTabs()
    │
    ▼
Storage Service
    │
    ▼
chrome.storage.local
```

---

## 4.2 `getSavedTabs()`

Returns all saved tabs.

### Signature

```ts
getSavedTabs(): Promise<SavedTab[]>
```

### Parameters

None.

### Returns

A promise containing an array of `SavedTab` objects.

### Example

```ts
const tabs = await getSavedTabs();
```

### Expected Behavior

If saved tabs exist:

```ts
[
  {
    id: "550e8400-e29b-41d4-a716-446655440000",
    title: "FastAPI",
    url: "https://github.com/fastapi/fastapi",
    reason: "Look at how dependency injection is implemented.",
    tags: ["development", "python"],
    priority: "high",
    createdAt: "2026-10-07T17:30:00.000Z",
    updatedAt: "2026-10-07T17:30:00.000Z",
    reminderAt: null,
    completedAt: null
  }
]
```

If no saved tabs exist:

```ts
[]
```

The function should not return `undefined` simply because storage has not been initialized.

---

## 4.3 `getSavedTab()`

Returns a single saved tab by ID.

### Signature

```ts
getSavedTab(id: string): Promise<SavedTab | null>
```

### Parameters

```text
id
```

The unique ID of the saved tab.

### Example

```ts
const tab = await getSavedTab(id);
```

### Returns

The matching `SavedTab` if found.

If no saved tab exists with that ID:

```ts
null
```

---

## 4.4 `saveTab()`

Stores a new saved tab.

### Signature

```ts
saveTab(tab: SavedTab): Promise<void>
```

### Parameters

```ts
tab: SavedTab
```

### Example

```ts
await saveTab(savedTab);
```

### Expected Behavior

The function should:

1. Retrieve the existing saved tabs.
2. Add the new saved tab.
3. Write the updated collection to storage.
4. Preserve all previously stored tabs.

The function must not overwrite unrelated saved items.

---

## 4.5 `updateTab()`

Updates an existing saved tab.

### Signature

```ts
updateTab(
  id: string,
  updates: Partial<SavedTab>
): Promise<SavedTab>
```

### Parameters

```text
id
```

The ID of the saved tab being modified.

```text
updates
```

The fields that should be changed.

### Example

```ts
const updatedTab = await updateTab(id, {
  priority: "high"
});
```

### Expected Behavior

The function should:

1. Find the saved tab by ID.
2. Apply the requested changes.
3. Preserve fields that were not changed.
4. Update `updatedAt`.
5. Save the updated collection.
6. Return the updated saved tab.

The following fields should not normally be changed through general updates:

```text
id
createdAt
```

---

## 4.6 `deleteTab()`

Deletes a saved tab.

### Signature

```ts
deleteTab(id: string): Promise<void>
```

### Parameters

```text
id
```

The ID of the saved tab.

### Example

```ts
await deleteTab(id);
```

### Expected Behavior

The function should remove only the saved tab matching the provided ID.

Other saved tabs must remain unchanged.

Reminder cleanup should also occur when a saved item with a reminder is deleted.

The exact coordination between storage deletion and reminder deletion will be determined during implementation.

---

# 5. Tab Service

## 5.1 Purpose

The Tab Service provides functions for interacting with Chrome browser tabs.

It will primarily use the Chrome Tabs API.

The UI should be able to request information about the current webpage without implementing Chrome tab queries itself.

---

## 5.2 Active Tab Model

The Tab Service does not need to expose the entire Chrome `Tab` object to the rest of Tabmind.

Instead, it can return only the information Tabmind needs.

```ts
export interface ActiveTabInfo {
  title: string;
  url: string;
}
```

This keeps the rest of the application less dependent on Chrome-specific types.

---

## 5.3 `getActiveTab()`

Retrieves information about the currently active browser tab.

### Signature

```ts
getActiveTab(): Promise<ActiveTabInfo>
```

### Parameters

None.

### Example

```ts
const currentTab = await getActiveTab();
```

### Example Result

```ts
{
  title: "FastAPI",
  url: "https://github.com/fastapi/fastapi"
}
```

### Expected Behavior

The function should:

1. Query the active tab in the current browser window.
2. Retrieve its title and URL.
3. Validate that the required information exists.
4. Return an `ActiveTabInfo` object.

If the active page cannot be saved, the function should report an error rather than returning incomplete data.

---

## 5.4 `openTab()`

Opens a saved webpage.

### Signature

```ts
openTab(url: string): Promise<void>
```

### Parameters

```text
url
```

The webpage URL that should be opened.

### Example

```ts
await openTab(savedTab.url);
```

### Expected Behavior

The function should validate the URL and then request Chrome to open it in a browser tab.

---

## 5.5 `isSupportedUrl()`

Determines whether Tabmind supports a URL.

### Signature

```ts
isSupportedUrl(url: string): boolean
```

### Example

```ts
if (!isSupportedUrl(url)) {
  // prevent save
}
```

### Initial Supported Protocols

```text
http:
https:
```

### Examples

Supported:

```text
https://github.com/
http://example.com/
```

Unsupported:

```text
chrome://extensions/
file:///example.txt
javascript:void(0)
```

Support for additional URL types can be added later if required.

---

# 6. Reminder Service

## 6.1 Purpose

The Reminder Service manages reminders associated with saved tabs.

It will interact with:

```text
chrome.alarms
```

The background service worker will listen for alarms and use:

```text
chrome.notifications
```

to notify the user.

---

## 6.2 Alarm Naming

Each reminder needs to be connected to a saved tab.

Tabmind will use a predictable alarm name:

```text
tabmind-reminder:<savedTabId>
```

Example:

```text
tabmind-reminder:550e8400-e29b-41d4-a716-446655440000
```

This allows the background service worker to determine which saved tab belongs to an alarm.

---

## 6.3 `createReminder()`

Creates or replaces a reminder for a saved tab.

### Signature

```ts
createReminder(
  savedTabId: string,
  reminderAt: string
): Promise<void>
```

### Parameters

```text
savedTabId
```

The ID of the saved tab.

```text
reminderAt
```

The ISO 8601 timestamp representing when the reminder should occur.

### Example

```ts
await createReminder(
  savedTab.id,
  savedTab.reminderAt
);
```

### Expected Behavior

The function should:

1. Validate the reminder timestamp.
2. Confirm that the reminder is scheduled for the future.
3. Generate the alarm name.
4. Convert the timestamp into the format required by Chrome.
5. Create the Chrome alarm.

If an alarm with the same name already exists, the new reminder should replace the previous schedule for that saved tab.

---

## 6.4 `deleteReminder()`

Removes the reminder associated with a saved tab.

### Signature

```ts
deleteReminder(savedTabId: string): Promise<void>
```

### Example

```ts
await deleteReminder(savedTab.id);
```

### Expected Behavior

The function should generate the appropriate alarm name and remove the matching Chrome alarm.

---

## 6.5 `getReminderName()`

Generates the Chrome alarm name for a saved tab.

### Signature

```ts
getReminderName(savedTabId: string): string
```

### Example

```ts
getReminderName("123");
```

Returns:

```text
tabmind-reminder:123
```

Keeping this logic in one place prevents inconsistent alarm names.

---

# 7. Background Service Worker API Usage

The background service worker will listen for Chrome extension events.

The initial reminder system will listen for:

```ts
chrome.alarms.onAlarm
```

Conceptually:

```ts
chrome.alarms.onAlarm.addListener(async (alarm) => {
  // Determine whether this is a Tabmind reminder.
  // Extract the saved tab ID.
  // Retrieve the saved tab.
  // Display a notification.
});
```

The service worker may later handle:

- Keyboard commands
- Context menu actions
- Installation events
- Notification interactions

These features should be added only when required.

---

# 8. Chrome API Dependencies

Tabmind will depend on several Chrome Extension APIs.

## 8.1 Tabs API

Used for:

- Retrieving the active tab
- Opening saved webpages

Relevant API:

```text
chrome.tabs
```

---

## 8.2 Storage API

Used for:

- Saving tabs
- Reading saved tabs
- Updating saved tabs
- Deleting saved tabs

Relevant API:

```text
chrome.storage.local
```

Tabmind will not use `chrome.storage.sync` for the first version.

---

## 8.3 Alarms API

Used for:

- Scheduling reminders
- Triggering reminder events

Relevant API:

```text
chrome.alarms
```

---

## 8.4 Notifications API

Used for:

- Displaying reminder notifications

Relevant API:

```text
chrome.notifications
```

---

# 9. Service Error Handling

Internal service functions should not silently ignore failures.

For example:

```ts
try {
  await saveTab(tab);
} catch (error) {
  // UI can display an appropriate error state.
}
```

Services should throw or return meaningful errors when an operation cannot be completed.

Possible error situations include:

```text
Active tab unavailable
Unsupported URL
Invalid saved tab
Saved tab not found
Storage read failure
Storage write failure
Invalid reminder time
Alarm creation failure
Notification failure
```

The exact error model may be expanded during implementation.

---

# 10. Validation Boundary

Data should be validated before it crosses important application boundaries.

For example:

```text
User Input
    │
    ▼
Validation
    │
    ▼
Application Logic
    │
    ▼
Storage Service
    │
    ▼
Chrome Storage
```

Tabmind should not rely entirely on TypeScript interfaces for validation.

TypeScript provides compile-time type checking, but data retrieved from browser storage still exists at runtime and may be incomplete or outdated.

---

# 11. UI and Service Interaction

React components should call services instead of directly implementing Chrome operations throughout the application.

Preferred:

```ts
const tabs = await getSavedTabs();
```

Instead of repeatedly writing:

```ts
const result = await chrome.storage.local.get("savedTabs");
```

inside React components.

Likewise:

```ts
const currentTab = await getActiveTab();
```

is preferred over placing Chrome tab queries directly inside the popup component.

The goal is not to completely hide the fact that Tabmind is a Chrome extension. The goal is to keep browser-specific operations organized.

---

# 12. Example Save Workflow

A simplified save workflow may look like:

```ts
const currentTab = await getActiveTab();

const timestamp = new Date().toISOString();

const savedTab: SavedTab = {
  id: crypto.randomUUID(),
  title: currentTab.title,
  url: currentTab.url,
  reason: reason.trim(),
  tags: normalizeTags(tags),
  priority: priority,
  createdAt: timestamp,
  updatedAt: timestamp,
  reminderAt: reminderAt,
  completedAt: null
};

await saveTab(savedTab);

if (savedTab.reminderAt) {
  await createReminder(
    savedTab.id,
    savedTab.reminderAt
  );
}
```

The exact implementation may differ, but the responsibility of each layer should remain clear.

---

# 13. Example Dashboard Workflow

Loading saved tabs:

```ts
const tabs = await getSavedTabs();
```

Searching:

```ts
const query = searchInput.trim().toLowerCase();

const results = tabs.filter((tab) => {
  return (
    tab.title.toLowerCase().includes(query) ||
    tab.url.toLowerCase().includes(query) ||
    tab.reason.toLowerCase().includes(query) ||
    tab.tags.some((tag) => tag.includes(query))
  );
});
```

Reopening a saved tab:

```ts
await openTab(savedTab.url);
```

Deleting:

```ts
await deleteReminder(savedTab.id);
await deleteTab(savedTab.id);
```

Some of this coordination may eventually be moved into higher-level application functions if multiple interfaces need the same workflow.

---

# 14. Internal API Boundaries

The initial architecture should follow these boundaries:

```text
┌───────────────────────────────────┐
│             React UI              │
│                                   │
│ Popup              Dashboard      │
└─────────────────┬─────────────────┘
                  │
                  ▼
┌───────────────────────────────────┐
│          Internal Services        │
│                                   │
│ Storage      Tabs      Reminders  │
└─────────────────┬─────────────────┘
                  │
                  ▼
┌───────────────────────────────────┐
│         Chrome Extension APIs     │
│                                   │
│ storage   tabs   alarms   notify  │
└───────────────────────────────────┘
```

This boundary should remain simple.

A separate repository layer, dependency injection system, state management library, or other abstraction should not be introduced unless the project develops a clear need for it.

---

# 15. External Web APIs

The first version of Tabmind does not require an external web API.

There will be no initial dependency on:

- Tabmind backend API
- Authentication API
- Database API
- AI API
- Cloud synchronization API

This allows the core extension to operate locally.

---

# 16. Future AI API

AI functionality may eventually require communication with an external AI service.

Possible future functionality includes:

- Page summaries
- Suggested tags
- Natural language search
- Context extraction

The AI provider has not been selected.

The initial architecture should therefore avoid coupling Tabmind to a specific AI company or model.

A future architecture may introduce an interface such as:

```ts
interface AIService {
  summarize(content: string): Promise<string>;
  suggestTags(content: string): Promise<string[]>;
}
```

This is an example only and is not part of the MVP.

Production AI credentials should not be embedded directly inside the distributed Chrome extension.

If Tabmind eventually provides built-in AI functionality, a backend service may be required to protect credentials and control API access.

---

# 17. Future Backend API

Tabmind does not currently require a backend.

If future features require:

- User accounts
- Cross-device synchronization
- Shared collections
- Secure AI access
- Cloud backups

a Tabmind backend API may be introduced.

A possible future architecture could use endpoints such as:

```text
POST   /tabs
GET    /tabs
GET    /tabs/:id
PATCH  /tabs/:id
DELETE /tabs/:id
```

These endpoints are not current requirements.

They are examples of how the internal `SavedTab` model could eventually map to a remote API.

A backend API specification should be created separately if a backend becomes part of the project.

---

# 18. API Testing

Internal services should be tested independently when practical.

Examples include testing that:

- `getSavedTabs()` returns an empty array when no data exists.
- `saveTab()` preserves existing saved tabs.
- `getSavedTab()` finds the correct item.
- `updateTab()` modifies only the intended item.
- `deleteTab()` removes only the requested item.
- `isSupportedUrl()` correctly accepts and rejects URLs.
- `getReminderName()` produces predictable alarm names.
- Reminder validation rejects past timestamps.

Chrome APIs may need to be mocked during unit tests.

Integration and manual extension testing will verify that the services work correctly inside Chrome.

Detailed testing requirements will be defined in:

```text
07_Test_Plan.md
```

---

# 19. Initial API Summary

| Service | Function | Purpose |
|---|---|---|
| Storage | `getSavedTabs()` | Retrieve all saved tabs |
| Storage | `getSavedTab(id)` | Retrieve one saved tab |
| Storage | `saveTab(tab)` | Store a new saved tab |
| Storage | `updateTab(id, updates)` | Modify a saved tab |
| Storage | `deleteTab(id)` | Delete a saved tab |
| Tabs | `getActiveTab()` | Retrieve current page information |
| Tabs | `openTab(url)` | Open a saved webpage |
| Tabs | `isSupportedUrl(url)` | Validate supported URLs |
| Reminders | `createReminder(id, time)` | Schedule a reminder |
| Reminders | `deleteReminder(id)` | Remove a reminder |
| Reminders | `getReminderName(id)` | Generate an alarm identifier |

These interfaces represent the initial contract between Tabmind's UI and its application services.

They may change during implementation if a simpler or safer design becomes clear.

---

# 20. API Design Principles

Tabmind's internal APIs should follow these principles:

1. Keep service functions small and focused.
2. Keep Chrome API calls organized.
3. Avoid duplicating Chrome operations across React components.
4. Use TypeScript types for inputs and outputs.
5. Validate external and stored data at runtime when necessary.
6. Do not silently ignore failures.
7. Do not introduce unnecessary abstraction.
8. Keep the MVP independent of external web APIs.
9. Keep future AI integrations provider-independent where practical.
10. Update this document when service contracts change.