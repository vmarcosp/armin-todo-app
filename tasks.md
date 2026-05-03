# Todo App - Implementation Roadmap

## Project Overview
A minimalist todo app with localStorage persistence, featuring a clean black-themed UI with sharp, modern aesthetics.

---

## Phase 1: Foundation & Core Structure
**Goal:** Set up the project foundation with HTML structure and basic styling system.

### Technical Specs
- **Project Structure:**
  ```
  /
  ├── index.html
  ├── styles.css
  ├── app.js
  └── README.md
  ```

- **HTML Requirements:**
  - Semantic HTML5 structure
  - Main container with max-width constraint (600px)
  - Input field with placeholder "Add a new task..."
  - Empty task list container (ul/ol)
  - No external CSS frameworks - pure CSS

- **CSS Design System:**
  - Color Palette:
    - Primary: `#000000` (black)
    - Background: `#FFFFFF` (white)
    - Surface: `#F5F5F5` (light gray for hover states)
    - Text Primary: `#000000`
    - Text Secondary: `#666666`
    - Border: `#E0E0E0`
    - Accent (checked): `#000000`
  - Typography:
    - Font: System UI / -apple-system / BlinkMacSystemFont / 'Segoe UI' / Roboto
    - Input: 16px (prevents iOS zoom)
    - Task text: 16px
    - Sharp, clean letter-spacing
  - Spacing:
    - Container padding: 24px
    - Task item padding: 16px vertical
    - Gap between tasks: 0 (border separation)
  - Visual Style:
    - Sharp corners (no border-radius or max 2px)
    - 1px solid borders
    - No shadows - flat design
    - Clean divider lines between items

- **Acceptance Criteria:**
  - [ ] Static HTML renders correctly
  - [ ] CSS loads and applies minimalist black theme
  - [ ] Responsive layout works on mobile (320px+) and desktop
  - [ ] Input field is focused on page load

---

## Phase 2: State Management & Data Layer
**Goal:** Implement localStorage persistence and task data model.

### Technical Specs

- **Data Model:**
  ```javascript
  Task {
    id: string (timestamp + random),
    text: string,
    completed: boolean,
    createdAt: number (timestamp)
  }
  ```

- **localStorage Schema:**
  - Key: `'todo-tasks'`
  - Value: JSON array of Task objects
  - Max tasks: 1000 (performance guard)

- **Storage Service API:**
  ```javascript
  // Core methods to implement
  getTasks(): Task[]
  saveTasks(tasks: Task[]): void
  addTask(text: string): Task
  toggleTask(id: string): void
  deleteTask(id: string): void
  clearCompleted(): void
  ```

- **Error Handling:**
  - Try/catch around all localStorage operations
  - Fallback to in-memory array if localStorage unavailable
  - Console warning on storage errors (don't break UI)
  - Handle quota exceeded errors gracefully

- **Data Integrity:**
  - Validate data structure on load (migration ready)
  - Sanitize task text (XSS prevention - strip HTML tags)
  - Trim whitespace from input
  - Reject empty tasks

- **Acceptance Criteria:**
  - [ ] Tasks persist across page reloads
  - [ ] localStorage quota errors handled gracefully
  - [ ] Data validation prevents corrupted state
  - [ ] Storage operations are atomic (all-or-nothing)

---

## Phase 3: Task Input & Creation
**Goal:** Implement the input field with task creation functionality.

### Technical Specs

- **Input Component:**
  - Single text input, full width
  - Placeholder: "Add a new task..."
  - No submit button (Enter key only)
  - Auto-focus on page load
  - Character limit: 200 characters

- **Interaction Specs:**
  ```
  On Enter (keyCode 13):
    1. Trim input value
    2. If empty, do nothing
    3. If valid, create task
    4. Clear input
    5. Maintain focus
    6. Add subtle animation feedback

  On Escape (keyCode 27):
    1. Clear input
    2. Maintain focus
  ```

- **UX Details:**
  - Disable input briefly during save (50ms) to prevent double-submit
  - Show character count when > 150 chars (optional, minimalist)
  - Visual feedback on task creation (subtle flash or border pulse)

- **Accessibility:**
  - Label for input (visually hidden if needed)
  - aria-label: "New task"
  - Keyboard-only navigation works

- **Acceptance Criteria:**
  - [ ] Pressing Enter creates a task
  - [ ] Empty tasks are rejected (no creation)
  - [ ] Input clears after successful creation
  - [ ] Focus remains in input after creation
  - [ ] 200 character limit enforced
  - [ ] Works with screen readers

---

## Phase 4: Task List Rendering
**Goal:** Display tasks with check/uncheck functionality and delete capability.

### Technical Specs

- **Task Item Structure:**
  ```html
  <li class="task-item" data-id="task-id">
    <input type="checkbox" class="task-checkbox" />
    <span class="task-text">Task content</span>
    <button class="task-delete" aria-label="Delete task">×</button>
  </li>
  ```

- **Rendering Logic:**
  - Sort: Newest tasks at bottom (append mode)
  - Re-render entire list on state change (simple approach, <1000 items)
  - Use DocumentFragment for batch DOM updates
  - Empty state: Show "No tasks yet" message (subtle, gray)

- **Checkbox Behavior:**
  - Native checkbox input (accessibility)
  - Custom styled to match black theme
  - Checked state: Black fill, white checkmark
  - Unchecked state: White fill, black border
  - Smooth transition animation (150ms)
  - Clicking text also toggles checkbox

- **Delete Functionality:**
  - × button appears on hover (desktop) or always visible (mobile)
  - Click to delete immediately (no confirmation for single task)
  - Animation: Fade out + slide up before removal

- **Task States:**
  - Active: Black text, normal weight
  - Completed: Gray text (#999), strikethrough, lighter weight
  - Hover: Light gray background (#F5F5F5)

- **Visual Polish:**
  - 1px solid border between tasks
  - Sharp edges, no rounding
  - Consistent 16px padding
  - Delete button: 24px × 24px touch target

- **Acceptance Criteria:**
  - [ ] Tasks render in creation order
  - [ ] Checkbox toggles completed state
  - [ ] Completed tasks show strikethrough
  - [ ] Delete button removes task
  - [ ] Hover states work correctly
  - [ ] Empty state displays when no tasks
  - [ ] Animations are smooth (60fps)

---

## Phase 5: UI Polish & Interactions
**Goal:** Add animations, micro-interactions, and refine the visual experience.

### Technical Specs

- **Animations (CSS transitions):**
  ```css
  /* Task entry */
  @keyframes slideIn {
    from { opacity: 0; transform: translateY(-10px); }
    to { opacity: 1; transform: translateY(0); }
  }
  
  /* Task exit */
  @keyframes slideOut {
    to { opacity: 0; transform: translateX(-100%); }
  }
  
  /* Checkbox check */
  transition: all 0.15s ease-out
  ```

- **Interactions:**
  - Add task: Subtle slide-down animation
  - Delete task: Slide-left + fade out (300ms)
  - Toggle complete: Strikethrough animates (optional)
  - Input focus: Subtle border color change to black

- **Visual Refinements:**
  - Checkbox: Custom SVG or pure CSS, 20px × 20px
  - Delete button: Thin × character, opacity 0.3 → 1 on hover
  - Input: Bottom border only, 2px black when focused
  - No outline rings (use border color change instead)

- **Responsive Design:**
  - Mobile: Full width, touch-friendly (44px min touch targets)
  - Tablet: Centered, max-width 600px
  - Desktop: Centered, max-width 600px
  - Font sizes scale slightly on mobile

- **Dark Mode Preparation:**
  - CSS variables for colors (optional future enhancement)
  - Structure supports theme switching

- **Performance:**
  - Use transform and opacity for animations (GPU accelerated)
  - will-change on animated elements
  - Debounce rapid operations (if needed)

- **Acceptance Criteria:**
  - [ ] Animations run at 60fps
  - [ ] Mobile touch targets are 44px minimum
  - [ ] Visual hierarchy is clear
  - [ ] No layout shift during animations
  - [ ] Polished, premium feel

---

## Phase 6: Advanced Features & Edge Cases
**Goal:** Add task management features and handle edge cases.

### Technical Specs

- **Task Counters:**
  - Display: "X tasks remaining" (active count)
  - Position: Below task list, left-aligned
  - Style: Small text (14px), gray color

- **Bulk Actions:**
  - "Clear completed" button (appears only when completed tasks exist)
  - Position: Below task list, right-aligned
  - Style: Small text (14px), black, underline on hover
  - Confirmation: None (immediate action)

- **Task Editing (Optional Phase 6+):**
  - Double-click to edit
  - Enter to save, Escape to cancel
  - Inline input replaces text

- **Edge Cases:**
  - XSS: Sanitize all user input (strip <script> and HTML)
  - localStorage full: Graceful degradation
  - 1000+ tasks: Performance warning or pagination
  - Special characters: Handle emojis, unicode correctly
  - Browser back button: State persists (already handled)
  - Multiple tabs: Sync across tabs (storage event listener)

- **Cross-Tab Sync:**
  ```javascript
  window.addEventListener('storage', (e) => {
    if (e.key === 'todo-tasks') {
      reloadTasks();
    }
  });
  ```

- **Accessibility Enhancements:**
  - ARIA live region for task updates
  - Focus management after delete (move to next/previous)
  - High contrast mode support
  - Reduced motion media query support

- **Acceptance Criteria:**
  - [ ] Task counter displays correctly
  - [ ] Clear completed works and removes only completed
  - [ ] Changes sync across browser tabs
  - [ ] XSS attempts are sanitized
  - [ ] localStorage errors handled gracefully
  - [ ] Reduced motion preferences respected

---

## Phase 7: Testing & Optimization
**Goal:** Ensure quality, performance, and cross-browser compatibility.

### Technical Specs

- **Manual Testing Checklist:**
  - [ ] Chrome, Firefox, Safari, Edge (latest 2 versions)
  - [ ] iOS Safari (iPhone SE, iPhone Pro)
  - [ ] Android Chrome
  - [ ] Keyboard-only navigation
  - [ ] Screen reader testing (VoiceOver, NVDA)

- **Performance Metrics:**
  - First Contentful Paint: < 1.5s
  - Time to Interactive: < 2s
  - Animation frame rate: 60fps
  - localStorage operations: < 10ms

- **Code Quality:**
  - No console errors
  - JSDoc comments for all functions
  - Consistent code style
  - No unused variables

- **Bundle Size:**
  - Target: < 20KB total (HTML + CSS + JS)
  - No external dependencies
  - Minified production build (optional)

- **SEO (Minimal):**
  - Meta viewport tag
  - Semantic HTML
  - Descriptive title

- **Deployment:**
  - Static hosting ready (GitHub Pages, Netlify, Vercel)
  - No build step required
  - Single HTML file option (inline CSS/JS) for ultimate portability

- **Acceptance Criteria:**
  - [ ] All manual tests pass
  - [ ] Performance targets met
  - [ ] No JavaScript errors in console
  - [ ] Works offline (after first load)
  - [ ] Deployed and accessible via URL

---

## Implementation Order Summary

1. **Phase 1:** Foundation - Get something on screen
2. **Phase 2:** Storage - Make data persist
3. **Phase 3:** Input - Allow creating tasks
4. **Phase 4:** List - Display and basic actions
5. **Phase 5:** Polish - Animations and refinement
6. **Phase 6:** Features - Counters, clear completed, edge cases
7. **Phase 7:** Ship - Test, optimize, deploy

---

## Design References

### Color Palette
```
Primary:      #000000
Background:   #FFFFFF
Surface:      #F5F5F5
Border:       #E0E0E0
Text:         #000000
TextMuted:    #666666
TextDisabled: #999999
```

### Typography Scale
```
Title:        24px, font-weight: 600
Input:        16px, font-weight: 400
Task:         16px, font-weight: 400
TaskComplete: 16px, font-weight: 400, text-decoration: line-through
Meta:         14px, font-weight: 400
```

### Spacing Scale
```
xs:  4px
sm:  8px
md:  16px
lg:  24px
xl:  32px
```

---

## Notes

- Keep it simple: Resist adding features beyond these phases
- Mobile-first: Design for touch, enhance for desktop
- Performance: 60fps animations, instant feedback
- Accessibility: WCAG 2.1 AA compliance target
