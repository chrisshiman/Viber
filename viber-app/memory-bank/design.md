# Design Document

**Project:** Task Manager App
**Owner:** \[You / Your Team]
**Date:** \[Insert Date]

---

## 1. Introduction

This design document outlines the technical design for the Task Manager App, a single-user, browser-based task management tool. It describes the architecture, data structures, components, UI/UX flow, theming approach, and deployment strategy, ensuring alignment with the PRD.

---

## 2. Architecture

### 2.1 Application Type

* **Single Page Application (SPA)** built with **Next.js (App Router)**.
* Deployed as a **static site on Vercel**.
* No backend required; all persistence handled client-side.

### 2.2 Data Flow

* Tasks are stored in **LocalStorage** (v1) with potential migration to **IndexedDB** (v2) for scalability.
* **Context API** + **React hooks** will manage global state.
* **TaskProvider** wraps the app to expose task state/actions.

### 2.3 Architecture Diagram

```
 ┌──────────────────┐
 │   User Interface │   ← MUI Components (TaskList, TaskForm, Toolbar, ThemeToggle)
 └─────────┬────────┘
           │
 ┌─────────▼────────┐
 │  State Management│   ← Context API + Reducer (Tasks, Theme)
 └─────────┬────────┘
           │
 ┌─────────▼────────┐
 │ Local Storage    │   ← Persistence Layer
 └──────────────────┘
```

---

## 3. Data Model

### 3.1 Task Object

```ts
type Task = {
  id: string;             // UUID
  title: string;          // Required
  description?: string;   // Optional
  dueDate?: string;       // ISO 8601 format
  priority: "low" | "medium" | "high";
  category?: string;      // Optional, e.g. "Work", "Personal"
  completed: boolean;
  createdAt: string;      // ISO timestamp
  updatedAt: string;      // ISO timestamp
};
```

### 3.2 Theme Preference

```ts
type ThemePreference = "light" | "dark" | "system";
```

* Stored in LocalStorage (`themePreference`).
* Defaults to `"system"` (detects `prefers-color-scheme`).

---

## 4. UI / Component Design

### 4.1 Layout

* **AppBar/Header**: App title, theme toggle.
* **Main Content Area**: Task list, empty state.
* **Floating Action Button (FAB)**: Add Task.
* **Task Dialog**: Modal form for create/edit.
* **Filter/Sort Controls** (v2).

### 4.2 Components

| Component        | Description                                                                     |
| ---------------- | ------------------------------------------------------------------------------- |
| `AppLayout`      | Wraps app with header, theme provider, and layout grid.                         |
| `TaskList`       | Displays list of tasks with MUI List or DataGrid.                               |
| `TaskItem`       | Single task row/card with checkbox, title, metadata, and actions (edit/delete). |
| `TaskFormDialog` | Modal for creating/editing tasks.                                               |
| `ThemeToggle`    | Switch for light/dark mode, persists selection.                                 |
| `StatsPanel`     | (v2) Shows completed vs active tasks.                                           |
| `FilterControls` | (v2) Dropdowns/toggles for filtering/sorting.                                   |

---

## 5. State Management

* **TasksContext**:

  * State: `Task[]`
  * Actions: `addTask`, `updateTask`, `deleteTask`, `toggleComplete`
  * Persistence: Save to LocalStorage on every state update

* **ThemeContext**:

  * State: `ThemePreference`
  * Actions: `setThemePreference`
  * Persistence: Save to LocalStorage

---

## 6. Theming

* **Implementation**: MUI `ThemeProvider` with `createTheme()`.
* **Themes**:

  * Light: White/gray background, dark text.
  * Dark: Dark gray background, light text.
* **System Preference**: Detect `prefers-color-scheme`.
* **User Override**: Toggle sets `ThemePreference`, overrides system.

---

## 7. Persistence Layer

* **Storage Strategy**:

  * Use `localStorage` with JSON serialization.
  * Keys:

    * `tasks` → JSON array of `Task` objects
    * `themePreference` → `"light" | "dark" | "system"`

* **Migration Path**:

  * If LocalStorage exceeds \~5MB, migrate to IndexedDB in v2.

---

## 8. Deployment

* Hosted on **Vercel** with automatic deployments from GitHub/GitLab.
* CI/CD pipeline runs TypeScript checks and ESLint.
* Preview deployments available for feature branches.

---

## 9. Security & Privacy

* No authentication required.
* All data stored locally, not shared externally.
* No PII beyond what the user manually enters in tasks.

---

## 10. Future Enhancements

* **Filtering/Sorting/Search**: Implement selectors in TasksContext.
* **Categories/Tags**: Add `category` field to task schema.
* **Stats Panel**: Compute derived state (e.g., % completed).
* **Export/Import**: JSON download/upload for task backup.
* **PWA**: Service Worker for offline + installable app.
* **Cloud Sync (Optional)**: Integrations with third-party APIs.