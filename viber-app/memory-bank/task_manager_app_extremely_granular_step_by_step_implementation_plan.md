# Task Manager App — Extremely Granular Step‑by‑Step Implementation Plan

**Tech stack:** Next.js (App Router) · TypeScript (strict) · Material UI (MUI) · Vercel · LocalStorage (v1)

---

## 0) Assumptions & Guardrails
- Single user; no auth; no backend.
- Data lives in the browser; v1 uses `localStorage`. (Optional v2: IndexedDB.)
- Strict TypeScript everywhere; no `any`.
- Minimal dependencies; keep the bundle small.
- Accessibility and keyboard navigation are first-class.

**Definition of Done (MVP):**
- Create/edit/delete/complete tasks with title, description, due date, priority.
- Light/dark mode that respects system preference with manual override persisted locally.
- Responsive UI with MUI.
- Tasks and theme preference persist across reloads.
- Linting, formatting, basic tests, and deploy on Vercel.

---

## 1) Repository & Tooling Setup

### 1.1 Initialize repo
1. Create a GitHub repo (private or public).
2. Clone locally: `git clone <repo>`.
3. `cd` into the repo.

### 1.2 Bootstrap Next.js + TS
1. `pnpm dlx create-next-app@latest` (or `npx`) → choose **TypeScript**, **App Router**, **ESLint**.
2. Remove example CSS and boilerplate you won’t use.

### 1.3 Package manager & scripts
1. Choose `pnpm` (recommended) or `npm`/`yarn`.
2. Ensure scripts in `package.json`:
   - `dev`, `build`, `start` (from Next).
   - `lint` → `next lint`.
   - `typecheck` → `tsc --noEmit`.
   - `format` → `prettier --write .`.

### 1.4 TypeScript strict mode
1. In `tsconfig.json`: set `"strict": true`, `"noImplicitAny": true`, `"noUncheckedIndexedAccess": true`, `"noFallthroughCasesInSwitch": true`.
2. Add path alias:
   ```json
   {
     "compilerOptions": {
       "baseUrl": ".",
       "paths": {"@/*": ["src/*"]}
     }
   }
   ```
3. Create `src/` and move app files under it (`src/app`, `src/components`, `src/lib`, `src/context`, `src/hooks`). Update imports to `@/`.

### 1.5 Linting & formatting
1. ESLint: keep Next + TS rules; add `eslint-config-prettier` to avoid conflicts.
2. Prettier: add `.prettierrc` (printWidth 100/110, semi true, singleQuote true).
3. Add `.editorconfig` for consistent line endings and indentation.

### 1.6 Git hooks (optional but recommended)
1. `pnpm add -D husky lint-staged`.
2. `npx husky init`; create pre-commit hook: `pnpm lint-staged`.
3. `package.json`:
   ```json
   {
     "lint-staged": {
       "**/*.{ts,tsx}": ["eslint --fix", "prettier --write"],
       "**/*.{json,md,css}": ["prettier --write"]
     }
   }
   ```

**Acceptance criteria:** repo builds locally; `pnpm dev` starts app; `pnpm typecheck` and `pnpm lint` pass.

---

## 2) Base App Structure (App Router)

### 2.1 Directories & files
```
src/
  app/
    layout.tsx
    page.tsx
    globals.css
  components/
  context/
  hooks/
  lib/
  styles/ (optional)
```

### 2.2 Root layout
- Implement `<html lang="en">` and `<body>`.
- Insert ThemeProvider wrapper (to be built in Section 4).
- Add MUI `CssBaseline`.

### 2.3 Home page
- Minimal placeholder content to verify layout + theme.

**Acceptance criteria:** App renders with baseline styles; no console errors.

---

## 3) MUI Setup & Baseline Theme

### 3.1 Install MUI
`pnpm add @mui/material @emotion/react @emotion/styled @mui/icons-material`

### 3.2 Create theme factory
- `src/lib/theme.ts`:
  - `createAppTheme(mode: 'light' | 'dark')` returning `Theme` with palette, typography, shape, and components overrides.
  - Use accessible color contrast; avoid pure black/white.

### 3.3 Add ThemeProvider shell
- `src/components/ThemeRegistry.tsx` to configure MUI SSR cache (Emotion) if needed.

**Acceptance criteria:** MUI components render, and toggling the `mode` in dev updates visuals.

---

## 4) Color Mode (Light/Dark) with System Preference + Persistence

### 4.1 Define types
- `src/lib/theme-types.ts`:
  ```ts
  export type ThemePreference = 'light' | 'dark' | 'system';
  ```

### 4.2 Color mode context
- `src/context/ColorModeContext.tsx`:
  - State: `{ preference: ThemePreference }` + derived `mode: 'light'|'dark'`.
  - Detect system via `window.matchMedia('(prefers-color-scheme: dark)')`.
  - Persist `preference` to `localStorage`.
  - Provide `setPreference` method.

### 4.3 Provider integration
- Wrap `app/layout.tsx` with `ColorModeProvider` and MUI `ThemeProvider(createAppTheme(mode))`.
- Add `CssBaseline`.

### 4.4 UI control
- `src/components/ThemeToggle.tsx`:
  - MUI `IconButton` + `Menu` with options: System, Light, Dark.
  - Updates `preference` and closes menu.

**Acceptance criteria:**
- Default follows system on first load.
- User can select Light/Dark/System.
- Choice persists after reload.

---

## 5) Domain Model & Utilities

### 5.1 Task type
`src/lib/types.ts`:
```ts\export type Priority = 'low' | 'medium' | 'high';
export type Task = {
  id: string;
  title: string;
  description?: string;
  dueDate?: string; // ISO date string (YYYY-MM-DD or full ISO)
  priority: Priority;
  category?: string;
  completed: boolean;
  createdAt: string; // ISO timestamp
  updatedAt: string; // ISO timestamp
};
```

### 5.2 Schemas (optional but recommended)
- Add `zod` for runtime validation:
  - `pnpm add zod`.
  - `src/lib/schemas.ts` with `TaskSchema`.

### 5.3 Helpers
- `src/lib/id.ts` — `uuid()` using `crypto.randomUUID()` fallback.
- `src/lib/date.ts` — parse/format helpers; `isOverdue`, `isToday`, etc.

**Acceptance criteria:** Strongly typed Task model; utility functions have unit tests.

---

## 6) Persistence Layer (LocalStorage v1)

### 6.1 Storage contract
- `src/lib/storage.ts`:
  - Keys: `tasks`, `themePreference`, `storageVersion`.
  - Functions:
    - `loadTasks(): Task[]`
    - `saveTasks(tasks: Task[]): void`
    - `loadThemePreference(): ThemePreference | null`
    - `saveThemePreference(p: ThemePreference): void`

### 6.2 Versioning & migrations
- Add `STORAGE_VERSION = 1`.
- `migrateIfNeeded(raw: unknown): Task[]` placeholder for future schema changes.

### 6.3 Safety
- Wrap all access in `try/catch` to tolerate corrupted data; on error, log and reset to empty array.

**Acceptance criteria:**
- Loading empty storage returns `[]`.
- Saving and reloading preserves tasks.
- Bad/corrupted data does not crash the app.

---

## 7) State Management (Context + Reducer)

### 7.1 Tasks context
- `src/context/TasksContext.tsx`:
  - State: `tasks: Task[]`.
  - Reducer actions:
    - `ADD_TASK`, `UPDATE_TASK`, `DELETE_TASK`, `TOGGLE_COMPLETE`.
  - Side effect: on every state change, `saveTasks(state.tasks)`.
  - Selectors (pure functions): `getActiveTasks`, `getCompletedTasks`, `findTaskById`.

### 7.2 Public API
- Export hooks:
  - `useTasks()` → `{ tasks, addTask, updateTask, deleteTask, toggleComplete }`.

### 7.3 Tests
- Reducer unit tests for each action; ensure immutability.

**Acceptance criteria:** Dispatching actions updates state and persists to storage.

---

## 8) Layout & Shell

### 8.1 App bar / header
- `src/components/AppHeader.tsx`:
  - App title (e.g., “Tasks”).
  - `ThemeToggle` on the right.

### 8.2 Container
- `src/components/AppLayout.tsx`:
  - MUI `Container` + responsive paddings.
  - Slot for primary content.

### 8.3 Floating Action Button (FAB)
- Bottom-right FAB to open “Add Task” dialog.

**Acceptance criteria:** Shell renders consistently across breakpoints; header is sticky; FAB appears on mobile/desktop.

---

## 9) Task List & Item

### 9.1 List
- `src/components/TaskList.tsx`:
  - Accepts `tasks: Task[]`.
  - Renders empty state if none.
  - Otherwise map to `TaskItem`.

### 9.2 Item
- `src/components/TaskItem.tsx`:
  - Checkbox → toggles complete.
  - Title + optional chips: Priority, Due Date (with overdue style).
  - Actions: Edit (opens dialog), Delete (confirm dialog or inline `Menu`).
  - Style: line-through + muted for completed.

### 9.3 Keyboard & a11y
- Ensure `Tab` order, `Enter/Space` activate checkbox, ARIA labels for buttons.

**Acceptance criteria:**
- Toggling checkbox updates UI and persists.
- Delete removes item after confirmation.
- Overdue tasks visually distinct.

---

## 10) Task Form Dialog (Create/Edit)

### 10.1 Dialog component
- `src/components/TaskFormDialog.tsx` props:
  - `open: boolean`, `mode: 'create' | 'edit'`, `initial?: Task`, `onClose()`, `onSubmit(taskDraft)`.

### 10.2 Fields
- Title (required, text).
- Description (multiline).
- Due date (date picker or text with validation).
- Priority (select: Low/Medium/High).
- Category (optional text or select; v1 keep text).

### 10.3 Validation
- Built-in with MUI + TS types; optional `zod` refine for due date format.
- Disable submit until required fields valid; show helper text.

### 10.4 Create flow
- FAB → open dialog → submit → `addTask({...})` with `id`, timestamps, default `completed=false`.

### 10.5 Edit flow
- Item → Edit → open with initial values → submit → `updateTask` + `updatedAt`.

**Acceptance criteria:**
- Cannot save without a title.
- Invalid date is rejected or normalized.
- Dialog closes and list updates on success.

---

## 11) Empty State & Microcopy

### 11.1 Empty state
- Friendly message and CTA to add first task.
- Consider a small illustration (optional, keep bundle light).

### 11.2 Microcopy
- Concise labels: “Add task”, “Save”, “Cancel”, “Delete task”, “Mark complete”.

**Acceptance criteria:** Empty state renders on fresh install; CTA opens dialog.

---

## 12) Sorting & Filtering (Post‑MVP toggleable)

### 12.1 Sorting
- Control: `Select` with: Created (new→old), Due date (soonest), Priority (high→low).
- Pure selector functions (no mutation).

### 12.2 Filtering
- Tabs or Segmented control: All / Active / Completed.

### 12.3 Search (optional)
- Debounced text input filtering by title/description.

**Acceptance criteria:** Sort/filter/search act purely on in‑memory state and re-render in <100ms on 1k tasks.

---

## 13) Styling, Theming, and Density

### 13.1 Theme tokens
- Define spacing, radius, shadows once; reuse across components.

### 13.2 Dark mode specifics
- Ensure contrast ≥ WCAG AA for text and interactive elements.
- Avoid pure black; use deep gray background.

### 13.3 Density
- Provide comfortable defaults; optionally expose compact density in v2.

**Acceptance criteria:** Visual parity across light/dark; consistent paddings and typography scale.

---

## 14) Accessibility (a11y) Pass

### 14.1 Keyboard
- All interactive elements reachable and operable via keyboard.
- Visible focus ring; avoid outline removal.

### 14.2 ARIA
- Dialog has `aria-labelledby`/`aria-describedby`.
- Buttons have `aria-label` where icon-only.

### 14.3 Color contrast
- Verify with tooling (e.g., Lighthouse/axe). Adjust palette if any < AA.

**Acceptance criteria:** Lighthouse Accessibility ≥ 95 on both themes.

---

## 15) Testing

### 15.1 Unit tests
- Reducer tests for all actions.
- Utilities (date, id, storage) tests.

### 15.2 Component tests (React Testing Library)
- TaskFormDialog: validation, submit callbacks.
- TaskItem: toggle, delete.
- ThemeToggle: preference persistence.

### 15.3 E2E (optional/bonus)
- Playwright: create task → complete → edit → delete flow.

**Acceptance criteria:** `pnpm test` green; basic flows covered.

---

## 16) Performance & Stability

### 16.1 Render perf
- Memoize `TaskItem` with `React.memo`.
- Key by `task.id`; avoid index keys.
- Virtualize list (v2) if > 1k tasks using `@tanstack/react-virtual`.

### 16.2 Storage perf
- Batch writes via reducer; avoid redundant saves inside tight loops.

### 16.3 Error boundaries
- Add a simple error boundary around the app shell.

**Acceptance criteria:** Smooth scrolling and interactions with 1k tasks on mid‑range laptop.

---

## 17) Observability (Lightweight)

### 17.1 Runtime logs
- Development: `console.debug` gated by `process.env.NODE_ENV`.
- Production: no noisy logs.

### 17.2 Basic analytics (optional)
- Add a simple pageview (if desired) with privacy in mind; default off for v1.

**Acceptance criteria:** No PII collected; app works fully offline.

---

## 18) Build & Deployment (Vercel)

### 18.1 Project setup
- Create a Vercel project; connect GitHub repo.
- Default build command: `pnpm build`; output: `.next`.

### 18.2 Environments
- Preview deployments for PRs; Production on main branch merges.

### 18.3 Health checks
- Verify SSR/CSR pages render; test theme toggle and persistence in preview.

**Acceptance criteria:** Successful production deployment; URL shared.

---

## 19) Documentation

### 19.1 README
- Quickstart, scripts, architecture overview, decisions (ADR‑style bullets), limitations, roadmap.

### 19.2 In‑code docs
- TSDoc on public functions.

**Acceptance criteria:** New developer can run and understand the project within minutes.

---

## 20) Optional v2 Enhancements (Backlog)
- IndexedDB persistence using `idb` for scalability and atomic updates.
- Import/Export JSON (and simple CSV export).
- Categories/tags with color chips.
- Stats panel and simple charts.
- PWA: service worker, install prompt, offline assets cache.
- Theming customization (primary color picker).
- i18n (react‑intl or next‑intl).

---

## 21) Concrete Implementation Checklist (Copy/Paste‑able)

- [ ] Initialize Next.js TS App Router project
- [ ] Configure TS strict, path aliases
- [ ] Add ESLint + Prettier + Husky + lint‑staged
- [ ] Install MUI + icons + Emotion
- [ ] Implement theme factory (`createAppTheme`)
- [ ] Implement ColorModeContext with system detection & persistence
- [ ] Add ThemeToggle component
- [ ] Build AppLayout, AppHeader, FAB
- [ ] Define `Task` type, helpers (id, date)
- [ ] Implement storage adapter (load/save + versioning)
- [ ] Implement TasksContext + reducer + selectors
- [ ] Build TaskList + TaskItem (checkbox, edit, delete)
- [ ] Build TaskFormDialog (create/edit with validation)
- [ ] Wire empty state; microcopy polish
- [ ] Basic unit tests (reducer, utils)
- [ ] Component tests (dialog, item, theme toggle)
- [ ] Performance pass (memoize, avoid unnecessary renders)
- [ ] Accessibility pass (keyboard, ARIA, contrast)
- [ ] Vercel deploy (preview + production)
- [ ] README and code comments

---

## 22) Reference Skeletons (Snippets)

### 22.1 Color mode provider (sketch)
```tsx
// src/context/ColorModeContext.tsx
'use client';
import React from 'react';
import { ThemeProvider, CssBaseline } from '@mui/material';
import { createAppTheme } from '@/lib/theme';

type ThemePreference = 'light' | 'dark' | 'system';

type Ctx = {
  preference: ThemePreference;
  setPreference: (p: ThemePreference) => void;
  mode: 'light' | 'dark';
};

export const ColorModeContext = React.createContext<Ctx | null>(null);

export function ColorModeProvider({ children }: { children: React.ReactNode }) {
  const [preference, setPreference] = React.useState<ThemePreference>(() => {
    if (typeof window === 'undefined') return 'system';
    return (localStorage.getItem('themePreference') as ThemePreference) || 'system';
  });

  const systemPrefersDark = typeof window !== 'undefined' &&
    window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches;

  const mode = preference === 'system' ? (systemPrefersDark ? 'dark' : 'light') : preference;

  React.useEffect(() => {
    if (typeof window !== 'undefined') {
      localStorage.setItem('themePreference', preference);
    }
  }, [preference]);

  const theme = React.useMemo(() => createAppTheme(mode), [mode]);

  return (
    <ColorModeContext.Provider value={{ preference, setPreference, mode }}>
      <ThemeProvider theme={theme}>
        <CssBaseline />
        {children}
      </ThemeProvider>
    </ColorModeContext.Provider>
  );
}
```

### 22.2 Tasks reducer (sketch)
```ts
// src/context/tasks-reducer.ts
import { Task } from '@/lib/types';

export type Action =
  | { type: 'ADD_TASK'; payload: Task }
  | { type: 'UPDATE_TASK'; payload: Task }
  | { type: 'DELETE_TASK'; payload: { id: string } }
  | { type: 'TOGGLE_COMPLETE'; payload: { id: string } };

export function tasksReducer(state: Task[], action: Action): Task[] {
  switch (action.type) {
    case 'ADD_TASK':
      return [action.payload, ...state];
    case 'UPDATE_TASK':
      return state.map(t => (t.id === action.payload.id ? action.payload : t));
    case 'DELETE_TASK':
      return state.filter(t => t.id !== action.payload.id);
    case 'TOGGLE_COMPLETE':
      return state.map(t => (t.id === action.payload.id ? { ...t, completed: !t.completed } : t));
    default:
      return state;
  }
}
```

### 22.3 Storage adapter (sketch)
```ts
// src/lib/storage.ts
import { Task } from '@/lib/types';
const TASKS_KEY = 'tasks';

export function loadTasks(): Task[] {
  try {
    const raw = localStorage.getItem(TASKS_KEY);
    if (!raw) return [];
    const data = JSON.parse(raw) as Task[];
    return Array.isArray(data) ? data : [];
  } catch {
    return [];
  }
}

export function saveTasks(tasks: Task[]): void {
  try {
    localStorage.setItem(TASKS_KEY, JSON.stringify(tasks));
  } catch {
    // ignore
  }
}
```

---

## 23) Rollout Plan
1. Land foundational PR: tooling, theme, color mode provider.
2. Land domain + storage + tasks context.
3. Land UI shell (layout/header/FAB).
4. Land Task CRUD (list, item, dialog) behind feature flag if desired.
5. Polish (copy, a11y, performance), add tests.
6. Ship to production; monitor and fix.

---

This plan is designed to be followed top‑to‑bottom. If you want, we can now generate the initial code scaffolding (files and minimal implementations) to drop into your repo and run immediately.

