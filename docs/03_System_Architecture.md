# Tabmind System Architecture

## 1. Overview

This document describes the planned system architecture for Tabmind.

Tabmind is a Chrome extension built using React, TypeScript, Vite, and Chrome Manifest V3. The architecture separates the user interface from browser-specific functionality and shared application logic.

The main parts of Tabmind are:

- Popup
- Dashboard
- Background service worker
- Content script
- Service layer
- Shared types and utilities
- Chrome Extension APIs

The first version of Tabmind will run entirely inside the browser and will not require a backend server.

## 2. Architecture Goals

The architecture is designed around several goals.

### Separation of Responsibilities

Each part of the extension should have a clear responsibility.

React components should focus mainly on displaying information and handling user interaction. Storage operations, reminder logic, and Chrome API interactions should be handled outside of UI components when possible.

### Reusability

Logic needed by multiple parts of the extension should be shared instead of duplicated.

For example, both the popup and dashboard may need to access saved tabs. Instead of implementing storage logic twice, both interfaces should use the same storage service.

### Maintainability

The project should remain understandable as more features are added.

Features should be separated into modules instead of placing most of the extension logic inside a few large files.

### Testability

Core application logic should be separated from the UI so it can be tested independently when possible.

### Extensibility

The architecture should support future features such as AI summaries, collections, synchronization, and additional browsers without requiring the entire application to be rewritten.

## 3. High-Level Architecture

Tabmind will use a client-side Chrome extension architecture.

```text
                        ┌─────────────────────┐
                        │       Chrome        │
                        │      Browser        │
                        └──────────┬──────────┘
                                   │
                 ┌─────────────────┼─────────────────┐
                 │                 │                 │
                 ▼                 ▼                 ▼
        ┌────────────────┐ ┌───────────────┐ ┌─────────────────┐
        │     Popup      │ │   Dashboard   │ │ Content Script  │
        │                │ │               │ │                 │
        │ React + TS     │ │ React + TS    │ │ TypeScript      │
        └───────┬────────┘ └───────┬───────┘ └────────┬────────┘
                │                  │                   │
                └────────────┬─────┘                   │
                             │                         │
                             ▼                         │
                    ┌─────────────────┐                │
                    │ Service Layer   │◄───────────────┘
                    │                 │
                    │ Storage         │
                    │ Tabs            │
                    │ Reminders       │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Chrome APIs     │
                    │                 │
                    │ storage         │
                    │ tabs            │
                    │ alarms          │
                    │ notifications   │
                    └────────┬────────┘
                             │
                             │
                    ┌────────▼────────┐
                    │ Background      │
                    │ Service Worker  │
                    │                 │
                    │ Extension events│
                    │ Alarm handling  │
                    │ Notifications   │
                    └─────────────────┘
```

The exact communication paths may change as implementation begins, but the separation between UI, application logic, and Chrome APIs should remain.

## 4. Main Components

### 4.1 Popup

The popup is the small interface displayed when the user selects the Tabmind icon from the Chrome toolbar.

It will be built using React and TypeScript.

The popup's primary responsibility is allowing the user to quickly save the currently active webpage.

The popup will eventually allow the user to:

- View the current page title
- View the current page URL
- Enter a reason
- Add tags
- Select a priority
- Set an optional reminder
- Save the webpage
- Open the dashboard

The popup should remain focused on quick actions.

More complex tab management should take place in the dashboard.

### 4.2 Dashboard

The dashboard is the main interface for viewing and managing saved tabs.

It will open as a full browser page and will also be built using React and TypeScript.

The dashboard will eventually support:

- Viewing saved tabs
- Searching saved tabs
- Filtering saved tabs
- Reopening webpages
- Editing saved information
- Marking items as completed
- Restoring completed items
- Deleting saved tabs
- Viewing reminders
- Managing tags and priorities

The dashboard should use the same service layer and shared types as the popup.

### 4.3 Background Service Worker

Chrome Manifest V3 extensions use a service worker for background functionality.

The Tabmind background service worker will handle tasks that should continue to work independently of the popup and dashboard.

Responsibilities may include:

- Listening for extension events
- Handling scheduled alarms
- Creating reminder notifications
- Handling keyboard commands
- Handling context menu actions
- Coordinating background operations

The service worker should not contain UI logic.

It should also avoid becoming the location for every piece of application logic. Reusable operations should remain in services when appropriate.

### 4.4 Content Script

The content script allows Tabmind to interact with webpages loaded in Chrome.

The initial MVP may not require extensive content-script functionality because the Chrome Tabs API can already provide basic information such as the page title and URL.

The content script will become more useful when Tabmind needs information directly from the webpage.

Possible future responsibilities include:

- Reading page metadata
- Extracting useful page text
- Collecting page descriptions
- Preparing content for AI summaries

Content scripts should only be given access to information required by implemented features.

### 4.5 Service Layer

The service layer contains reusable application logic that should not depend directly on React components.

Planned services include:

```text
services/
├── storage.ts
├── tabs.ts
└── reminders.ts
```

Additional services can be introduced when they have a clear responsibility.

#### Storage Service

`storage.ts` will provide a consistent interface for interacting with `chrome.storage.local`.

Possible responsibilities include:

- Getting all saved tabs
- Getting one saved tab
- Saving a new tab
- Updating a saved tab
- Deleting a saved tab
- Handling storage errors

React components should not repeatedly implement their own `chrome.storage.local` operations.

#### Tab Service

`tabs.ts` will handle browser-tab operations.

Possible responsibilities include:

- Getting the active tab
- Extracting the current title and URL
- Opening a saved URL
- Validating supported URLs

This keeps Chrome Tabs API logic separate from the UI.

#### Reminder Service

`reminders.ts` will handle reminder-related operations.

Possible responsibilities include:

- Creating alarms
- Updating alarms
- Removing alarms
- Generating consistent alarm identifiers
- Connecting alarms to saved tabs

The background service worker will listen for alarm events, while the reminder service will provide reusable operations for managing those alarms.

## 5. Shared Types

Shared TypeScript types will provide a consistent representation of data throughout the extension.

The initial saved-tab model is expected to resemble:

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
  reminderAt: string | null;
  completedAt: string | null;
  summary: string | null;
}
```

This type should be shared by:

- Popup
- Dashboard
- Storage services
- Reminder services
- Tests

The final data model will be documented in `04_Data_Design.md`.

## 6. Planned Project Structure

The initial project structure is expected to look similar to:

```text
tabmind/
├── docs/
│   ├── 01_Project_Vision.md
│   ├── 02_Software_Requirements_Specification.md
│   └── 03_System_Architecture.md
│
├── public/
│   └── icons/
│
├── src/
│   ├── popup/
│   │   ├── components/
│   │   ├── App.tsx
│   │   └── main.tsx
│   │
│   ├── dashboard/
│   │   ├── components/
│   │   ├── App.tsx
│   │   └── main.tsx
│   │
│   ├── background/
│   │   └── service-worker.ts
│   │
│   ├── content/
│   │   └── content.ts
│   │
│   ├── services/
│   │   ├── storage.ts
│   │   ├── tabs.ts
│   │   └── reminders.ts
│   │
│   ├── types/
│   │   └── saved-tab.ts
│   │
│   └── utils/
│
├── tests/
├── manifest.json
├── package.json
├── tsconfig.json
├── vite.config.ts
├── README.md
└── .gitignore
```

This structure is not permanent. It can change if implementation shows that another organization is easier to maintain.

Folders should be introduced because they serve a purpose, not simply to make the project appear more complex.

## 7. Data Flow

### 7.1 Saving a Tab

The basic save flow will be:

```text
User
  │
  ▼
Opens Tabmind Popup
  │
  ▼
Popup requests active tab
  │
  ▼
Tab Service
  │
  ▼
Chrome Tabs API
  │
  ▼
Title + URL returned
  │
  ▼
Popup displays page information
  │
  ▼
User enters reason
  │
  ▼
Input validated
  │
  ▼
SavedTab object created
  │
  ▼
Storage Service
  │
  ▼
chrome.storage.local
```

For the MVP, this is the most important data flow in the application.

### 7.2 Loading the Dashboard

```text
User opens Dashboard
        │
        ▼
React Dashboard
        │
        ▼
Storage Service
        │
        ▼
chrome.storage.local
        │
        ▼
SavedTab[]
        │
        ▼
React Dashboard
        │
        ▼
Saved tabs displayed
```

Search and filtering can initially operate on the locally loaded collection of saved tabs.

### 7.3 Reopening a Saved Tab

```text
User selects saved item
        │
        ▼
Dashboard
        │
        ▼
Tab Service
        │
        ▼
Chrome Tabs API
        │
        ▼
Webpage opens
```

### 7.4 Reminder Flow

Once reminders are implemented:

```text
User sets reminder
        │
        ▼
Popup / Dashboard
        │
        ▼
Reminder Service
        │
        ▼
chrome.alarms
        │
        │
        │ Scheduled time reached
        ▼
Background Service Worker
        │
        ▼
chrome.notifications
        │
        ▼
User receives reminder
```

This allows reminders to function without requiring the popup or dashboard to remain open.

## 8. Storage Architecture

The first version of Tabmind will use:

```text
chrome.storage.local
```

The application should interact with storage primarily through the storage service.

Instead of doing this throughout the UI:

```ts
chrome.storage.local.get(...);
```

React components should generally call application-level functions such as:

```ts
getSavedTabs();
saveTab();
updateTab();
deleteTab();
```

This provides one place to manage storage behavior and makes the application easier to test or change later.

The exact storage structure will be defined in `04_Data_Design.md`.

## 9. React Architecture

React will be used only where a user interface is needed.

The two main React entry points will be:

```text
Popup
Dashboard
```

They are separate interfaces but can share components, hooks, services, types, and utilities when appropriate.

React components should primarily handle:

- Rendering UI
- User interaction
- Local interface state
- Calling services
- Displaying success and error states

React components should generally not be responsible for:

- Directly managing Chrome storage throughout the application
- Scheduling Chrome alarms
- Implementing large amounts of business logic
- Handling background extension events

This separation should prevent UI components from becoming tightly coupled to Chrome APIs.

## 10. Chrome Extension Architecture

### Manifest V3

Tabmind will use Chrome Manifest V3.

The manifest will define:

- Extension metadata
- Popup entry point
- Background service worker
- Content scripts when required
- Permissions
- Icons
- Keyboard commands when implemented

### Permissions

Permissions should be added only when required by an implemented feature.

Possible permissions include:

```json
{
  "permissions": [
    "activeTab",
    "storage",
    "alarms",
    "notifications"
  ]
}
```

Additional permissions should not be requested simply because they might be useful later.

This keeps the extension's access easier to understand and reduces unnecessary privileges.

## 11. Build Architecture

Vite will be responsible for compiling and bundling the React and TypeScript source code into files Chrome can load as an extension.

The development flow will generally be:

```text
TypeScript + React source
          │
          ▼
         Vite
          │
          ▼
    Production build
          │
          ▼
    Extension files
          │
          ▼
        Chrome
```

The exact Vite configuration will be determined during initial project setup.

The build process must support multiple extension entry points, including the popup, dashboard, background service worker, and content script when needed.

## 12. Error Handling

Tabmind should handle failures without unnecessarily losing user data.

Possible failures include:

- Unable to access the current tab
- Storage read failure
- Storage write failure
- Invalid URL
- Missing reason
- Alarm creation failure
- Notification failure

Services should report failures to the calling layer rather than silently ignoring them.

The UI should provide understandable feedback when an operation the user initiated cannot be completed.

## 13. Security and Privacy

The first version of Tabmind will operate locally.

Saved-tab data will remain in `chrome.storage.local`.

The extension should:

- Request only required Chrome permissions
- Avoid collecting unnecessary webpage information
- Avoid transmitting saved content to external services
- Never hardcode private API keys
- Validate data before storing or processing it

Future AI functionality will require additional privacy and security decisions because webpage content may need to be sent to an external service.

That architecture will be designed separately before AI functionality is implemented.

## 14. Future Architecture

The first version intentionally avoids a backend.

A future version may introduce additional components if cross-device synchronization, accounts, collaboration, or AI functionality requires them.

A possible future architecture could look like:

```text
                Chrome Extension
                       │
                       ▼
                  Tabmind API
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
         Database            AI Provider
```

Possible backend responsibilities could include:

- User authentication
- Cross-device synchronization
- Shared collections
- Secure AI API access
- Server-side search
- Data backup

A backend should only be introduced when a feature actually requires one.

## 15. Architecture Principles

Development of Tabmind should follow these principles:

1. Keep the core save workflow simple.
2. Keep React focused on user interfaces.
3. Keep Chrome API interactions behind clear modules when practical.
4. Reuse shared logic instead of duplicating it.
5. Request only the permissions currently required.
6. Keep AI optional.
7. Avoid introducing a backend until there is a clear reason for one.
8. Prefer understandable code over unnecessary abstraction.
9. Add architectural complexity only when the project needs it.
10. Keep the architecture aligned with the actual implementation.

The architecture documentation should be updated whenever major technical decisions change.