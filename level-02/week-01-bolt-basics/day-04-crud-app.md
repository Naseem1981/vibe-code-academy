# Day 4: Build a Full CRUD Todo App with Bolt.new

## Outcome artifact
By the end of this day you will have a working Todo app in Bolt.new with add task, mark complete, inline edit, delete with confirmation, and filter by status (All / Active / Completed) — all running in Bolt's live preview.

---

## The core idea

CRUD stands for Create, Read, Update, Delete. These four operations are the foundation of almost every application that handles data — from a simple todo list to a production CRM. In plain terms:

- **Create**: add a new item (typing a task and pressing Add)
- **Read**: display existing items (the task list that appears on screen)
- **Update**: change an existing item (clicking a task to edit it, or marking it complete)
- **Delete**: remove an item (clicking the delete button)

Every app you build in this course — including the CRM in Week 2 — implements CRUD. The shapes of the data change (tasks become contacts, then orders, then events) but the four operations stay constant. Building a todo app first lets you understand CRUD in isolation, with simple data, before the complexity of a real database.

In today's build, tasks are stored in React's `useState` hook. `useState` holds data in the component's memory — fast, simple, and zero setup. The limitation is that `useState` is wiped every time the page refreshes. Close the tab, reopen it, and all tasks are gone. This is intentional for today. In the enterprise CRM build starting in Week 2, you will replace `useState` with Supabase — a PostgreSQL database with a JavaScript API. Every task you create will be written to a real database row, and it will persist across sessions, devices, and users. Today you learn the shape of CRUD. Week 2 you make it permanent.

---

## Step-by-step walkthrough

### Step 1 — Start a new Bolt.new project

1. Go to `https://bolt.new`.
2. Click inside the prompt box.
3. Do not continue an existing session from previous days — start a fresh project.

---

### Step 2 — Prompt 1: Base app with Create and Read

Paste this prompt exactly:

```
Build a Todo app using React and Tailwind CSS. Store all tasks in useState — no database or localStorage.

Data structure for each task:
{
  id: number (use Date.now() for unique IDs),
  text: string,
  completed: boolean,
  isEditing: boolean
}

Features to build now:

CREATE:
- A text input at the top with placeholder "Add a new task..."
- An "Add" button next to the input
- Pressing the Add button (or pressing Enter in the input) creates a new task with completed: false and isEditing: false
- Clear the input after adding
- Do not add empty tasks — if the input is blank, do nothing

READ:
- Display all tasks in a list below the input
- Each task shows: a checkbox on the left, the task text in the centre, and a placeholder area on the right (we will add buttons in the next step)
- Show a task count below the list: "X tasks remaining" (count only incomplete tasks)
- If there are no tasks, show a message: "No tasks yet. Add one above."

No editing, deleting, or filtering yet — just add and display.

Layout:
- Centered card, max width 520px, with a light grey (#f3f4f6) page background
- White card with subtle shadow and 24px padding
- "Todo" as the page/card heading
```

Wait for Bolt to generate. Test it by typing a task and pressing Enter. Confirm the task appears and the count updates.

---

### Step 3 — Prompt 2: Update — inline editing

```
Add inline editing to the todo app.

Editing behaviour:
- When the user clicks on the task text, that task enters edit mode (set isEditing: true on that task)
- In edit mode, the task text is replaced by a text input pre-filled with the current task text
- The input should be auto-focused when it appears
- Pressing Enter saves the new text: update the task's text in state, set isEditing: false
- Pressing Escape cancels the edit: restore the original text, set isEditing: false
- Clicking anywhere outside the input (onBlur) also saves the edit
- If the edited text is empty after trimming, delete the task instead of saving an empty string

Update state immutably:
- Use .map() to update only the edited task, do not mutate the array directly

Visual behaviour:
- In edit mode, the task text turns into a full-width input styled to look like the text (no border until focused)
- On focus, the input shows a bottom border in a muted blue colour
- Task text in read mode should have a cursor: pointer style to hint it is clickable

Do not change anything else — keep the Add input, task count, and layout exactly as they are.
```

After generation, click a task text. Confirm it becomes an input. Edit the text and press Enter. Confirm the updated text appears.

---

### Step 4 — Prompt 3: Delete with confirmation, and filter tabs

```
Add two features to the todo app:

FEATURE 1 — Delete with confirmation:
- Add a delete button (use a trash icon — a simple "×" character or an SVG trash icon) on the right side of each task row
- When clicked, show an inline confirmation: replace the delete button with two small buttons — "Yes, delete" and "Cancel"
- If the user clicks "Yes, delete", remove the task from the state array using .filter()
- If the user clicks "Cancel", hide the confirmation and show the delete button again
- Track the "pending deletion" state per task — add a field pendingDelete: boolean to the task data structure, defaulting to false
- Do not show a browser alert() — the confirmation must be inline in the task row

FEATURE 2 — Filter tabs:
- Above the task list, add three filter tabs: "All", "Active", "Completed"
- "All" shows every task
- "Active" shows only tasks where completed is false
- "Completed" shows only tasks where completed is true
- The active tab should have a visual indicator (underline or background highlight)
- The task count at the bottom ("X tasks remaining") always counts incomplete tasks from the full list, regardless of which filter is active
- Store the active filter in a separate useState variable (not inside the task objects)

Do not change the Add input, inline editing, or the card layout.
```

After generation, test:
1. Add three tasks.
2. Mark one complete with the checkbox.
3. Click the filter tabs — confirm "Active" hides the completed task and "Completed" shows only it.
4. Click the delete button on a task — confirm the inline confirmation appears.
5. Click "Yes, delete" — confirm the task is removed.
6. Click "Cancel" on another task's confirmation — confirm the task stays.

---

### Step 5 — Prompt 4: Visual polish

```
Restyle the todo app with a clean dark theme.

Colour palette:
- Page background: #111827 (dark charcoal)
- Card background: #1f2937 (slightly lighter dark)
- Card border: 1px solid #374151
- Heading text: #f9fafb (near white)
- Task text (incomplete): #e5e7eb (light grey)
- Task text (completed): #6b7280 (muted grey) with line-through
- Input background: #374151, input text: #f9fafb, input border: #4b5563
- Input focus border: #34d399 (mint green)
- Add button: #34d399 background, #111827 text, font-weight 600
- Add button hover: #10b981 (slightly darker mint)
- Filter tab active: #34d399 text with a bottom border of 2px #34d399
- Filter tab inactive: #6b7280 text, no border, hover colour #9ca3af
- Checkbox: use accent-color: #34d399 in CSS so the browser-native checkbox shows mint when checked
- Delete button (×): #6b7280 colour, hover #ef4444 (red)
- "Yes, delete" button: #ef4444 background, white text
- "Cancel" button: #374151 background, #e5e7eb text
- Task count text: #6b7280
- "No tasks yet" message: #6b7280, italic

Typography:
- Use the Inter font from Google Fonts (add @import in CSS)
- Heading: 24px, font-weight 700
- Task text: 16px, font-weight 400
- Task count: 14px

Spacing:
- 8px gap between task rows
- 16px padding inside each task row
- Add a subtle left border (3px solid #34d399) on each incomplete task row
- Completed tasks: left border colour changes to #374151

Do not change any functionality — only change styles.
```

After generation, confirm:
- The page background is dark, not white.
- Completed tasks have strikethrough and are greyed out.
- The mint green colour appears on the Add button, active filter tab, and checkbox.
- The delete confirmation shows a red "Yes, delete" button.

---

## Practical workflow

1. Open bolt.new, start a new project.
2. Paste Prompt 1 — add and display tasks.
3. Test: type a task, press Enter, confirm it appears.
4. Paste Prompt 2 — inline editing.
5. Test: click task text, edit it, press Enter, press Escape.
6. Paste Prompt 3 — delete and filters.
7. Test: delete with confirmation, all three filter tabs.
8. Paste Prompt 4 — dark theme styling.
9. Confirm: dark background, mint accents, strikethrough on complete.
10. Add 10 tasks, mark some complete, filter to "Completed", delete one, edit one.
11. Refresh the page — confirm all tasks are gone (expected — useState is intentionally not persistent today).
12. Download the project.

---

## Common mistakes

**Mistake 1 — Mutating state directly instead of updating it immutably.**

This is the most common React bug and Bolt sometimes generates it incorrectly. If tasks are not updating on screen when you click the checkbox or save an edit, it is usually because the code is doing something like `tasks[index].completed = true` directly, rather than using `.map()` to return a new array. If you see this, write a prompt: `"The checkbox is not updating task state correctly. Rewrite the toggle complete handler to use .map() to return a new array where only the clicked task's completed property is changed. Do not mutate the tasks array directly."`

**Mistake 2 — Using the array index as the task ID.**

When you delete a task, the remaining tasks shift indices. If the task's key is its array index, React will re-render the wrong tasks, causing visual glitches or edit mode opening on the wrong item. Bolt sometimes generates `key={index}` on list items. If you notice erratic behaviour after deleting tasks, write a prompt: `"Change all task list item keys from using the array index to using task.id. The task.id is already a unique number set with Date.now() when the task is created."`

**Mistake 3 — Expecting data to persist after a page refresh with useState.**

`useState` stores data in memory for the lifetime of the component. When you refresh the page, the component unmounts and remounts — all state is reset to its initial value (an empty array of tasks). This is not a bug. It is how React state works. If you want data to survive a refresh, you need either `localStorage` (a browser-side storage, covered in the CRUD upgrade pack below) or a real database (Supabase, coming in Week 2). Do not waste time debugging "data disappearing on refresh" — it is working as intended.

---

## Your turn

Open bolt.new now. Run all four prompts in sequence. After Prompt 3, manually test every CRUD operation:

1. Create: add three tasks by typing and pressing Enter.
2. Read: confirm all three appear in the list.
3. Update (complete): click the checkbox on one task — confirm it shows as completed with strikethrough.
4. Update (edit): click the text of a task — confirm it becomes an editable input — change the text — press Enter — confirm the new text is saved.
5. Delete: click the delete button — confirm the inline confirmation appears — click "Yes, delete" — confirm the task is gone.
6. Filter: click "Active" — confirm only incomplete tasks show. Click "Completed" — confirm only the completed task shows. Click "All" — confirm all tasks show.

**Expected output after all four prompts:** A dark-themed todo app with mint green accents, full CRUD operations, inline editing, inline delete confirmation, and three working filter tabs.

**Failure state 1:** After Prompt 2, clicking a task text does not make it editable. Fix: the `isEditing` flag on the task is probably not triggering a re-render. Write a prompt: `"Clicking on the task text is not switching to edit mode. Check that the onClick handler on the task text paragraph calls setTasks() with a new array where only the clicked task has isEditing set to true. The handler must use .map() to return a new array, not mutate the existing one."`

**Failure state 2:** After Prompt 3, the filter tabs show but clicking them does not change which tasks are visible. Fix: the filter state is set but probably not applied to the rendered list. Write a prompt: `"The filter tabs are not filtering the task list. Make sure the task list renders a filtered version of the tasks array. Create a derived variable: const visibleTasks = filter === 'active' ? tasks.filter(t => !t.completed) : filter === 'completed' ? tasks.filter(t => t.completed) : tasks; and use visibleTasks.map() in the JSX, not tasks.map()."`

---

## Prompt / Template / Checklist pack

The four build prompts appear in the walkthrough above (Steps 2, 3, 4, 5). Here is the CRUD upgrade pack — three prompts to extend the app with new capabilities.

---

### CRUD Upgrade Prompt 1 — localStorage persistence

```
Add localStorage persistence to the todo app so tasks survive a page refresh.

Implementation:
- When the app first loads, read tasks from localStorage using: const saved = localStorage.getItem('tasks'); initialize tasks state with: saved ? JSON.parse(saved) : []
- Every time the tasks array changes, write it to localStorage using a useEffect: useEffect(() => { localStorage.setItem('tasks', JSON.stringify(tasks)); }, [tasks]);
- Import useEffect from React at the top of the file

Do not change any other functionality. After this change, adding, editing, completing, and deleting tasks should all be reflected in localStorage automatically, and the task list should survive a page refresh.

Add a small "Clear all" button at the bottom of the card that:
- Only shows if there is at least one task
- Calls localStorage.removeItem('tasks') and then resets tasks state to []
- Shows an inline confirmation before clearing ("Are you sure? This removes all tasks.")
```

---

### CRUD Upgrade Prompt 2 — Due dates

```
Add a due date field to each task.

Data structure change: add a dueDate field to each task — it can be a string in YYYY-MM-DD format or null if no date is set.

CREATE change:
- Next to the Add button, add a date picker input (type="date")
- When adding a task, include the selected date (or null if no date chosen)
- Clear the date input after adding a task

READ change:
- Display the due date below the task text in a smaller font
- Format the date as "Due: Mon 28 Apr" (use JavaScript's Date.toLocaleDateString with options {weekday: 'short', day: 'numeric', month: 'short'})
- If no due date, show nothing
- If the due date is today or in the past and the task is not complete, show the date in red (#ef4444) with the label "Overdue:"
- If the due date is tomorrow, show it in amber (#f59e0b) with the label "Due tomorrow:"

UPDATE change:
- In edit mode, also show the date input so the user can change the due date

Sort the task list automatically:
- Tasks with no due date appear last
- Tasks with due dates are sorted ascending (earliest due date first)
- Completed tasks always appear at the bottom regardless of due date
```

---

### CRUD Upgrade Prompt 3 — Priority levels

```
Add a priority level to each task.

Priority options: Low, Medium, High (default to Medium when adding a new task)

CREATE change:
- Add a priority selector (a row of three buttons: Low, Medium, High) below the task input
- The selected priority button is visually highlighted
- Default to Medium

READ change:
- Show a priority badge on each task row (to the left of the task text, before the checkbox)
- Badge colours:
  - Low: #6b7280 background (grey), white text
  - Medium: #f59e0b background (amber), dark text
  - High: #ef4444 background (red), white text
- Badge text: just the word "Low", "Medium", or "High" in a small pill shape (8px horizontal padding, 4px vertical padding, border-radius 9999px, font-size 11px)

FILTER change:
- Add three more filter buttons next to the existing All/Active/Completed tabs: "P: High", "P: Medium", "P: Low"
- These filter to show only tasks of that priority
- Combine with the existing status filter: if "Active" and "P: High" are both selected, show only active high-priority tasks
- Store the priority filter in a separate useState variable, default to null (no priority filter)

UPDATE change:
- In edit mode, show the priority selector so the user can change the task's priority

Sort tasks by priority within each filter:
- High priority tasks appear before Medium, Medium before Low
- Within the same priority, maintain the order tasks were added
```
