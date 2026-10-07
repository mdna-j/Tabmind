# Tabmind 🗂️

Tabmind is a Chrome extension that helps you remember why you saved a tab in the first place.

Instead of leaving dozens of tabs open or saving bookmarks that you forget about later, Tabmind lets you save a page along with the reason you wanted to come back to it. Saved tabs can also have tags, priorities, reminders, and eventually AI-generated summaries to make them easier to find later.

> This project is currently in development.

## Why I am building it

I tend to keep tabs open because I know I want to come back to them, but after a while I forget why I kept them open.

Bookmarks help save the page, but they do not really save the context behind it.

For example, I might save a GitHub repository because I want to look at how they implemented authentication, or save an article because I want to read it before an interview. A few weeks later, the URL and page title alone might not tell me why I cared about it.

Tabmind is my attempt to solve that problem by saving the reason along with the page.

## Planned features

For the first version, I want Tabmind to support:

- Save the current tab with a reason
- Automatically capture the page title and URL
- Add tags to saved tabs
- Set a priority
- Set optional reminders
- Search and filter saved tabs
- Mark tabs as completed
- Reopen or delete saved tabs
- View everything from a dashboard
- Use keyboard shortcuts for common actions

AI summaries are also planned, but the core extension should still work without AI.

## Example

Instead of saving only:

```text
FastAPI
https://github.com/fastapi/fastapi
```

Tabmind could save:

```text
FastAPI
https://github.com/fastapi/fastapi

Reason:
Look at how dependency injection is implemented.

Tags:
Development, Python

Priority:
High

Reminder:
Saturday at 2:00 PM
```

The goal is to preserve the context behind the saved page, not just the URL.

## Tech stack

| Area | Technology |
|---|---|
| Browser extension | Chrome Manifest V3 |
| Frontend | React |
| Language | TypeScript |
| Build tool | Vite |
| Styling | CSS |
| Storage | `chrome.storage.local` |
| Reminders | `chrome.alarms` |
| Notifications | `chrome.notifications` |
| Testing | Vitest + React Testing Library |
| AI | Planned |

React and TypeScript will be used to build the popup and dashboard interfaces. Vite will handle the development and build process.

Chrome-specific functionality such as tab access, storage, reminders, notifications, background tasks, and content scripts will use the Chrome Extension APIs.

The goal is to keep the architecture simple while still separating the UI, Chrome extension logic, and shared application logic.

## Planned project structure

```text
tabmind/
├── docs/
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

The structure may change as the project develops, but the goal is to keep different parts of the extension separated by responsibility.

### Popup

The popup will be the quickest way to save the current tab.

It will capture the current page information and allow the user to add a reason, tags, priority, and an optional reminder.

React will be used to manage the popup interface and its state.

### Dashboard

The dashboard will provide a larger interface for managing everything saved in Tabmind.

Users will be able to search, filter, reopen, complete, and delete saved tabs from the dashboard.

The dashboard will also be built with React since it will contain more interactive UI and application state as the project grows.

### Background service worker

The background service worker will handle tasks that should not depend on the popup or dashboard being open.

This will include things such as:

- Scheduled reminders
- Browser notifications
- Extension events
- Keyboard shortcuts
- Background Chrome API interactions

### Content script

The content script will allow Tabmind to interact with the currently open webpage when more information is needed than the browser already provides.

This will become more important later when AI summaries are added and Tabmind needs to extract useful page content.

### Services

Shared application logic will be separated from the React components.

For example:

```text
storage.ts
```

will handle reading and writing saved tabs.

```text
tabs.ts
```

will handle tab-related operations.

```text
reminders.ts
```

will handle creating, updating, and removing reminders.

Keeping this logic outside of React components should make it easier to reuse and test.

### Types

Shared TypeScript types and interfaces will be stored separately.

For example, a saved tab may eventually follow a structure similar to:

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

This structure will be finalized as the data design for the project is developed.

## Roadmap

### Phase 1: Core extension

- [ ] Set up React, TypeScript, and Vite
- [ ] Configure Chrome Manifest V3
- [ ] Load Tabmind as an unpacked Chrome extension
- [ ] Read the current browser tab
- [ ] Build the popup
- [ ] Save a tab with a reason
- [ ] Store saved tabs locally
- [ ] Build the dashboard
- [ ] Display saved tabs
- [ ] Reopen saved tabs
- [ ] Delete saved tabs
- [ ] Mark saved tabs as completed
- [ ] Add search and filtering
- [ ] Add tags and priorities

### Phase 2: Reminders and organization

- [ ] Add reminders with `chrome.alarms`
- [ ] Add browser notifications
- [ ] Add reminder snoozing
- [ ] Detect duplicate URLs
- [ ] Add a right-click save option
- [ ] Add keyboard shortcuts
- [ ] Save multiple tabs as a collection
- [ ] Support bulk saving

### Phase 3: AI features

- [ ] Extract useful page content
- [ ] Generate page summaries
- [ ] Suggest tags
- [ ] Improve search using saved context
- [ ] Experiment with natural language search

AI features should remain optional so the main functionality of Tabmind does not depend on an AI service.

### Phase 4: Sync and platform features

- [ ] Explore cross-device sync
- [ ] Add user accounts if a backend becomes necessary
- [ ] Explore a Firefox version
- [ ] Explore bookmark importing
- [ ] Add shared tab collections
- [ ] Explore integrations with other productivity tools

## Storage

The first version of Tabmind will use `chrome.storage.local`.

This allows saved tabs to persist between browser sessions without requiring a backend or user account.

The initial version will focus on local storage first. Cross-device synchronization can be explored later once the core functionality is working.

## Reminders

Tabmind will use `chrome.alarms` for reminders.

Chrome extension service workers can be suspended when they are not being used, which makes normal JavaScript timers unreliable for long-term reminders.

Using `chrome.alarms` allows Chrome to trigger the reminder even when the Tabmind popup or dashboard is not open.

The background service worker can then use `chrome.notifications` to notify the user.

## AI summaries

AI-generated summaries are planned for a later version of Tabmind.

The idea is to extract useful information from a saved webpage and create a short summary that helps the user understand what the page contains when they return to it later.

AI will be optional. Saving, organizing, searching, and setting reminders for tabs should continue to work without an AI service.

The AI provider and backend architecture have not been finalized yet.

## Development

Tabmind will be developed as a Chrome extension using React, TypeScript, and Vite.

During development, the extension will be built locally and loaded into Chrome as an unpacked extension.

Once the initial project setup is complete, development will generally follow this process:

1. Make changes to the project
2. Build the extension with Vite
3. Open `chrome://extensions`
4. Reload Tabmind
5. Test the updated extension

Development and debugging instructions will be expanded once the initial extension setup is complete.

## Testing

Testing will be added as the core features are implemented.

The current plan is to use:

- **Vitest** for unit tests
- **React Testing Library** for React components
- Manual Chrome extension testing for browser-specific behavior

Important areas to test will include:

- Saving and loading tabs
- Input validation
- Search and filtering
- Duplicate detection
- Reminder logic
- React component behavior
- Storage operations

A more detailed testing strategy will be documented in the test plan.

## Documentation

More detailed planning and technical documentation will be kept in the `docs/` directory.

Planned documentation includes:

```text
docs/
├── 01_Project_Vision.md
├── 02_Software_Requirements_Specification.md
├── 03_System_Architecture.md
├── 04_Data_Design.md
├── 05_API_Specification.md
├── 06_UI_UX_Design.md
├── 07_Test_Plan.md
├── 08_Deployment_Guide.md
├── 09_Developer_Guide.md
└── 10_Roadmap.md
```

These documents will be updated as Tabmind is designed and built.

## Project status

Tabmind is currently in the planning and early development stage.

The first goal is to get the React and TypeScript extension running in Chrome and build a small working version that can read the current tab, save it with a reason, and store it locally.

Once that works, I plan to build the dashboard and add organization, reminder, and AI features in smaller steps.