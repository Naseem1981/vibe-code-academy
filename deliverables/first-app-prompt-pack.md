# First App Prompt Pack
*Module 1 Deliverable — Vibe Code from Zero*

---

## First App Prompt Template

Fill in every blank before sending. A prompt with blanks left unfilled will produce guesswork.

```
Create a [TYPE OF APP] using plain HTML, CSS, and JavaScript — no frameworks, no npm, no build steps.

The page should show:
- [MAIN CONTENT ITEM 1]
- [MAIN CONTENT ITEM 2]
- [MAIN CONTENT ITEM 3]

Visual style:
- Background colour: [HEX CODE, e.g. #0F172A]
- Text colour: [HEX CODE, e.g. #F8FAFC]
- Accent colour: [HEX CODE, e.g. #6EE7B7]
- Font feel: [e.g. "clean and modern", "bold and dramatic", "warm and friendly"]

Interactions:
- [HOVER EFFECT, e.g. "buttons glow with accent colour on hover"]
- [OTHER INTERACTION, e.g. "clicking a button highlights it"]

Save everything in the current folder. Create an index.html file I can open directly in any browser — no server needed.
```

---

## 5 Ready-to-Use Starter Prompts

Copy any of these and paste directly into Claude Code. They are complete and ready to use as-is, or customise any part of them.

---

### Prompt 1 — Personal Landing Page

```
Create a personal landing page using plain HTML, CSS, and JavaScript — no frameworks, no npm, no build steps.

The page should show:
- A name (use "Alex" as placeholder)
- A short bio: "I build things with AI. This is my first vibe-coded app."
- Three buttons: "My Projects", "Contact Me", "About"

Visual style:
- Background: #0F172A (deep navy)
- Text: #F8FAFC (off-white)
- Accent: #6EE7B7 (mint green)
- Font: clean, modern sans-serif

Interactions:
- Buttons glow with a mint green shadow on hover
- Smooth transition on all hover states (0.2s ease)

Save everything in the current folder. Create an index.html I can open directly in any browser — no server needed.
```

---

### Prompt 2 — Simple Calculator

```
Create a calculator app using plain HTML, CSS, and JavaScript — no frameworks, no npm, no build steps.

Features:
- A display at the top showing the current number or calculation
- A grid of buttons: digits 0–9, decimal point, four operations (+ - × ÷), equals (=), and clear (C)
- Clicking digit buttons builds the number on the display
- Clicking an operation button stores the first number and waits for the second
- Clicking equals shows the result
- Clicking C resets the display

Visual style:
- Background: #1E293B (dark charcoal)
- Display background: #0F172A (darker)
- Digit button background: #334155
- Operation button background: #38BDF8 (blue accent)
- Text: #F8FAFC
- Border-radius: 8px on buttons, 12px on the calculator container

Interactions:
- Buttons darken on hover and scale down slightly on click (transform: scale(0.97))

Save as index.html in the current folder. No server needed.
```

---

### Prompt 3 — Random Quote Generator

```
Create a random quote generator using plain HTML, CSS, and JavaScript — no frameworks, no API calls.

Features:
- 10 motivational quotes hardcoded in the JavaScript (choose varied authors)
- One quote displayed at a time with the author's name shown below in smaller text
- A "New Quote" button that picks a random quote (never repeats the same quote twice in a row)
- A fade animation (opacity 0 → 1 over 0.4s) when a new quote appears

Visual style:
- Background: #FFFBEB (warm cream)
- Text: #1C1917 (dark brown)
- Author name: #78716C (muted brown)
- Button background: #D97706 (gold)
- Button text: white
- Quote text: 22px, italic, generous line height (1.8)
- Centred layout, max-width 600px, vertically centred in the viewport

Interactions:
- Button darkens on hover
- Quote fades out then fades in when changing

Save as index.html in the current folder. No server needed.
```

---

### Prompt 4 — To-Do List

```
Create a to-do list app using plain HTML, CSS, and JavaScript — no frameworks.

Features:
- A text input at the top with an "Add" button — pressing Enter or clicking Add adds the task to the list
- Each task shows: a checkbox, the task text, and a delete button (×)
- Clicking the checkbox marks the task as complete: strike through the text and reduce opacity to 50%
- Clicking the delete button removes the task from the list
- A task count at the top: "[N] tasks remaining"
- All tasks saved to localStorage so they survive a page refresh

Visual style:
- Background: #0F172A (deep navy)
- Card/container background: #1E293B
- Text: #F8FAFC
- Completed task text: #94A3B8 (muted, struck through)
- Delete button: #EF4444 (red) on hover
- Accent / checkbox colour: #A78BFA (purple)
- Border-radius: 8px on tasks

Interactions:
- Tasks slide in with a subtle fade when added
- Delete button only visible on hover of the task row

Save as index.html in the current folder. No server needed.
```

---

### Prompt 5 — Countdown Timer

```
Create a countdown timer using plain HTML, CSS, and JavaScript — no frameworks.

Features:
- Three number inputs: Hours, Minutes, Seconds — each labelled, numeric only
- A "Start" button that begins the countdown from the entered time
- A "Pause" button that pauses the countdown (Start resumes it)
- A "Reset" button that stops the timer and restores the input values
- When the timer reaches 00:00:00: the display flashes red and shows "Time's up!" in large text
- Display the remaining time in large bold digits in HH:MM:SS format

Visual style:
- Background: #0F172A (deep navy)
- Timer display: 64px monospace font, white
- "Time's up!" state: display turns red (#EF4444), text pulses (opacity animation)
- Start button: #F97316 (orange)
- Pause button: #64748B (slate)
- Reset button: #334155 (dark slate)
- All buttons: 8px border-radius, white text

Interactions:
- Start button changes label to "Resume" when paused
- Buttons have hover and active states

Save as index.html in the current folder. No server needed.
```
