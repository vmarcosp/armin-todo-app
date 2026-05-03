# Tasks — Todo App

> **Scope**: A minimalist todo app with localStorage persistence, featuring a clean black-themed UI with sharp, modern aesthetics. The app is built with TypeScript, React, and Vite — no external UI dependencies. Linting and formatting are handled by Biome. It covers foundation, state management, task creation, list rendering, UI polish, advanced features (counters, cross-tab sync, XSS handling), and final quality validation.
> **Created**: 2026-05-02

## Phase 1: State Management & Data Layer

**Goal**: Deliver a working storage service backed by localStorage, with data validation and graceful error handling.

### storage-service

- **Title**: Implement localStorage storage service with task data model
- **Description**: In `src/storage.ts`, implement a storage service with the following API: `getTasks(): Task[]`, `saveTasks(tasks: Task[]): void`, `addTask(text: string): Task`, `toggleTask(id: string): void`, `deleteTask(id: string): void`, `clearCompleted(): void`. The `Task` shape is `{ id: string (timestamp + random), text: string, completed: boolean, createdAt: number }`. Persist under the key `'todo-tasks'` as a JSON array. Wrap all localStorage calls in try/catch; fall back to an in-memory array on failure and log a console warning. Cap stored tasks at 1000 (performance guard). On load, validate the parsed array shape to avoid crashes on corrupted data. Sanitize task text to strip HTML tags (XSS prevention). Trim whitespace and reject empty strings before creating a task.
- **Depends on**: —
- **Rationale**: A clean data layer decouples UI rendering from persistence logic and prevents XSS and data-corruption bugs from propagating upward.

## Phase 2: Foundation & Core Structure

**Goal**: Deliver a static page with semantic HTML, the full CSS design system applied, and a focused input field — nothing interactive yet.

### html-foundation

- **Title**: Add semantic HTML structure and CSS design system
- **Description**: In `src/App.tsx`, build the component tree: a `<main>` container capped at 600px, a text input with `placeholder="Add a new task..."` and `aria-label="New task"`, and an empty `<ul>` for the task list. In `src/styles.css`, implement the full design system: color palette (`#000000` primary, `#FFFFFF` background, `#F5F5F5` surface, `#E0E0E0` border, `#666666` text-secondary, `#999999` text-disabled), system-ui font stack at 16px, 24px container padding, 16px task item vertical padding, 1px solid borders, and sharp corners (no border-radius or max 2px). No external CSS frameworks. Auto-focus the input on page load via `useEffect` + `ref`. Layout must be responsive from 320px up.
- **Depends on**: storage-service
- **Rationale**: Every subsequent phase depends on the component skeleton and CSS tokens. Establishing the design system first prevents style drift across phases.

## Phase 3: Task Input & Creation

**Goal**: Deliver a fully wired input field that creates, validates, and persists tasks on Enter.

### task-input

- **Title**: Wire input field for task creation with keyboard handling
- **Description**: In `src/components/TaskInput.tsx`, handle `keyDown` events. On **Enter**: trim the value, skip if empty, call `addTask()`, clear the input, keep focus, and call the `onTaskAdded` callback. On **Escape**: clear the input and keep focus. Enforce the 200-character `maxlength` attribute on the input. Disable the input for 50 ms after each save to prevent double-submit (via `useState` + `useEffect`). Show a character counter (inline, 14px gray) only when the user has typed more than 150 characters. Apply a bottom-border-only focus style (2px solid `#000000`) instead of the default outline ring. The input must carry `aria-label="New task"`.
- **Depends on**: html-foundation
- **Rationale**: The input is the primary creation surface. Getting it right — including the double-submit guard and accessibility label — before rendering the list keeps concerns separate.

## Phase 4: Task List Rendering

**Goal**: Deliver a live task list with toggle, delete, hover states, empty state, and efficient React rendering.

### task-list-render

- **Title**: Render task list with toggle and delete
- **Description**: In `src/components/TaskList.tsx` and `src/components/TaskItem.tsx`, implement `renderTasks()` driven by a `tasks` prop. Each task renders as `<li className="task-item" data-id="...">` containing a native `<input type="checkbox" className="task-checkbox">`, a `<span className="task-text">`, and a `<button className="task-delete" aria-label="Delete task">×</button>`. Sort: newest tasks appended at the bottom (creation order). Active tasks: black text, normal weight. Completed tasks: `#999999` color, `text-decoration: line-through`, lighter font-weight. The checkbox must be custom-styled to the black theme (checked = black fill + white checkmark via CSS, unchecked = white fill + black border). The delete button has a 24×24px touch target; on desktop it is opacity 0.3 normally and 1 on hover; on mobile it is always visible. Clicking the task text also toggles the checkbox. When there are no tasks, show a centered "No tasks yet" message in `#999999` at 14px. Apply `#F5F5F5` background on row hover. Separate rows with a 1px solid `#E0E0E0` border.
- **Depends on**: task-input
- **Rationale**: Rendering is blocked on both the storage service (data) and the input handler (state mutations).

## Phase 5: UI Polish & Animations

**Goal**: Deliver GPU-accelerated entry/exit animations, smooth checkbox transitions, and mobile-ready touch targets.

### ui-animations

- **Title**: Add CSS animations for task entry, exit, and checkbox transitions
- **Description**: In `src/styles.css`, add a `@keyframes slideIn` animation (`from { opacity: 0; transform: translateY(-10px) }` → `to { opacity: 1; transform: translateY(0) }`) applied to newly added `.task-item` elements. Add a `@keyframes slideOut` animation (`to { opacity: 0; transform: translateX(-100%) }`) triggered before a task is removed from the DOM — hold removal until the 300 ms animation completes via a `useState` flag in `TaskItem`. Add `transition: all 0.15s ease-out` to checkboxes. Apply `will-change: transform, opacity` on animated elements. All touch targets (checkbox, delete button, task row) must be at least 44×44px on mobile. Ensure animations use only `transform` and `opacity` (GPU-accelerated, no layout thrashing). Add `@media (prefers-reduced-motion: reduce)` to disable all transitions and keyframe animations. Input focus style: 2px solid `#000000` bottom border, no outline ring. Responsive: font sizes scale slightly on mobile; container is full-width below 600px.
- **Depends on**: task-list-render
- **Rationale**: Animations run on top of a stable DOM — adding them before the list is complete would require rework. Reduced-motion support is a WCAG 2.1 AA requirement.

## Phase 6: Advanced Features & Edge Cases

**Goal**: Deliver task counter, "Clear completed" bulk action, cross-tab sync, and full edge-case hardening.

### task-counter-clear

- **Title**: Add task counter and clear-completed bulk action
- **Description**: In `src/components/TaskFooter.tsx`, add a footer row below the task list. Left side: `<span id="task-count">` showing "X tasks remaining" (active task count), styled at 14px, `#666666`. Right side: a `<button id="clear-completed">Clear completed</button>` visible only when `tasks.some(t => t.completed)` — 14px, black text, no background, underline on hover, no confirmation dialog. Wire `clearCompleted()` from the storage service to this button. Both elements share a flex container that disappears when the list is empty. Update both on every render cycle.
- **Depends on**: ui-animations
- **Rationale**: Counter and bulk-clear are standard todo-app features that require a stable rendering pipeline before they can be wired in cleanly.

### cross-tab-sync

- **Title**: Sync task state across browser tabs via storage event
- **Description**: In `src/hooks/useStorageSync.ts`, implement a custom hook that calls `window.addEventListener('storage', (e) => { if (e.key === 'todo-tasks') onSync(); })` and removes the listener on unmount. Use it in `App.tsx` to trigger a state refresh whenever another tab mutates `todo-tasks`. Verify that the state-reading function reads fresh data from `getTasks()` on each call (i.e., it does not close over a stale snapshot).
- **Depends on**: task-counter-clear
- **Rationale**: Without this listener, two tabs editing the same list will silently diverge and whichever tab saves last wins — a data-loss hazard.

## Phase 7: Accessibility & Quality

**Goal**: Ship a WCAG 2.1 AA–compliant, cross-browser, zero-console-error build under 20 KB total.

### accessibility-focus

- **Title**: Add ARIA live region and focus management for task operations
- **Description**: In `src/components/A11yAnnouncer.tsx`, render `<div id="a11y-announcer" aria-live="polite" aria-atomic="true" className="visually-hidden"></div>` (`.visually-hidden` uses the standard clip-path off-screen technique). In `App.tsx`, update the announcer text after each task creation ("Task added"), deletion ("Task deleted"), and toggle ("Task marked complete / incomplete"). After deleting a task, move focus to the next task's delete button; if none, move to the previous; if the list is empty, move focus back to the input. Verify keyboard-only navigation: Tab through input → tasks → footer actions, Enter/Space on checkboxes, Delete button reachable. Add `role="list"` to the `<ul>` and `role="listitem"` to each `<li>` for VoiceOver compatibility on Safari. Confirm the page title is descriptive (e.g., `<title>Todo</title>`) and a `<meta name="viewport">` tag is present.
- **Depends on**: cross-tab-sync
- **Rationale**: ARIA live regions and post-delete focus management are the last pieces needed for screen-reader users to operate the app without visual feedback. They require a complete component tree to target.
