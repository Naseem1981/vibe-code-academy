# Day 9: Responsive Design

## Outcome artifact

By the end of this day you will have your existing landing page fully responsive — tested at 480px, 768px, and 1200px in browser DevTools with no layout breaks at any screen size.

---

## The core idea

Responsive design means you write one codebase and it works correctly on every screen — a phone in someone's pocket, a tablet on a coffee table, a laptop at a desk. The layout, text, and buttons all adjust automatically based on how wide the screen is. You do not build three separate pages. You build one page that responds to its environment.

Three things break most often when a desktop layout is viewed on mobile. First, font sizes set in pixels become too small to read without zooming — a heading that looks fine at 1200px is tiny on a 375px phone screen. Second, elements wider than the screen overflow horizontally, creating an ugly sideways scroll that makes the page feel broken. Third, tap targets — buttons and links — are sized for a mouse pointer, not a finger. A button that is 24px tall is nearly impossible to tap accurately on a touchscreen. The fix for all three is the same: tell Claude Code exactly what you want at each screen width.

The mechanism that makes responsive design work is breakpoints. A breakpoint is just a screen width at which your layout changes. You write a CSS rule that says "at widths below 768px, do this instead." You only need two breakpoints for most projects. Use 768px for tablet: anything between 768px and 1199px gets a tablet layout. Use 480px for mobile: anything below 480px gets a mobile layout. Everything above 1199px is your desktop layout, which is what you already have. When you prompt Claude Code, always name all three widths explicitly — vague instructions like "make it mobile friendly" produce inconsistent results.

---

## Step-by-step walkthrough

### Step 1 — Open your project in Claude Code

Open your terminal. Navigate to your project folder. Launch Claude Code.

```
claude
```

You should see the Claude Code prompt ready to accept input.

### Step 2 — Run a responsive audit on your landing page

Before fixing anything, ask Claude Code to tell you what is broken. This gives you a checklist to work through.

```
Look at my landing page (index.html). List every element that will break or look bad on a 480px mobile screen. Be specific — name each element and describe the problem. Do not fix anything yet, just list the issues.
```

Claude Code will return a list. Common items include: navigation links wrapping or overflowing, hero section text too large, images wider than the screen, buttons too small to tap, grid layouts that stay in multi-column instead of stacking.

### Step 3 — Fix the navigation first

Navigation is the most visible element. Fix it before anything else.

```
Make the navigation bar in index.html fully responsive.

On desktop (1200px and above): keep the current horizontal layout with all links visible.
On tablet (768px to 1199px): keep horizontal layout but reduce font size and padding slightly.
On mobile (below 480px): hide the navigation links and show a hamburger menu icon (three horizontal lines). When the hamburger is clicked, the links should drop down in a vertical stack below the header. Add a close button or clicking the hamburger again should hide the menu.

Use only HTML, CSS, and vanilla JavaScript — no libraries. Add the hamburger functionality with a small inline script tag.
```

### Step 4 — Fix the hero section

The hero section — the large top area with a heading and subtitle — usually has oversized text on mobile.

```
Make the hero section in index.html responsive.

On desktop (1200px+): keep the current layout and font sizes.
On tablet (768px): reduce the main heading font size to 36px and the subtitle to 18px. Keep side-by-side layout if there is one.
On mobile (480px): set the main heading to 28px and subtitle to 16px. Stack everything vertically — heading, then subtitle, then button. Center-align all text. Make the call-to-action button full width with 16px vertical padding.
```

### Step 5 — Fix images

Images that use fixed pixel widths will overflow on small screens.

```
Fix all images in index.html so they are responsive.

Apply max-width: 100% and height: auto to every img element. This prevents any image from exceeding the width of its container.

If any image has a fixed pixel width set inline or in a style attribute, replace it with width: 100% and max-width: [original value]px.
```

### Step 6 — Fix grid and card layouts

Multi-column grids collapse badly on mobile if you do not handle them.

```
Find every CSS grid or flexbox row layout in index.html that uses multiple columns.

For each one:
- On desktop (1200px+): keep the current number of columns.
- On tablet (768px): reduce to 2 columns maximum. If it is already 2 columns, keep it.
- On mobile (480px): collapse to a single column. Each card or item should be full width and stack vertically with 16px gap between them.
```

### Step 7 — Fix button tap targets

Buttons need to be at least 44px tall on mobile for comfortable tapping. This is Apple's Human Interface Guidelines minimum.

```
Find every button and anchor link styled as a button in index.html.

On mobile (below 480px): set minimum height to 44px and minimum width to 44px. Add vertical padding of at least 12px. Make all primary action buttons full width (width: 100%).
```

### Step 8 — Test in browser DevTools

Open your `index.html` in a browser (Chrome or Edge recommended). Do this:

1. Press **F12** to open DevTools.
2. Click the **Toggle device toolbar** icon — it looks like a phone and tablet overlapping, in the top-left area of the DevTools panel. Keyboard shortcut: **Ctrl + Shift + M** (Windows) or **Cmd + Shift + M** (Mac).
3. At the top of the page, a device selector bar appears. Click the dropdown that says "Responsive" or shows a device name.
4. Select **iPhone SE** (375px wide) — this tests your 480px mobile breakpoint.
5. Scroll through the entire page. Look for: horizontal scrollbars, text touching the edges, overlapping elements, buttons too small to tap.
6. Change the device to **iPad Mini** (768px wide) — this tests your tablet breakpoint.
7. Scroll through again. Look for the same issues.
8. Click **Responsive** in the dropdown and drag the width handle to 1280px — this tests desktop.
9. Note every problem you see.

### Step 9 — Fix any remaining issues found in DevTools

For each problem spotted in Step 8, go back to Claude Code with a specific description.

```
In DevTools at 375px width I can see [describe the exact problem — for example: "the logo and nav links are overlapping" or "the features grid is still showing 3 columns"]. Fix this so it looks correct at that width.
```

Be specific. The more precisely you describe what you see, the more accurate Claude Code's fix will be.

### Step 10 — Final check with a responsive audit prompt

Once you believe all fixes are in, run one final audit prompt.

```
Review index.html one more time. Check every CSS rule and every layout section. Confirm:
1. No element has a fixed pixel width that could overflow a 480px screen.
2. All font sizes scale down appropriately on mobile.
3. All buttons and links have at least 44px touch targets on mobile.
4. No horizontal scroll exists at 480px, 768px, or 1200px.

If you find any remaining issues, fix them now.
```

---

## Practical workflow

1. Open terminal, navigate to project folder, run `claude`.
2. Run audit prompt — get list of issues before touching any code.
3. Fix navigation: hamburger on mobile, horizontal on desktop.
4. Fix hero: reduce heading size, stack vertically, full-width button on mobile.
5. Fix images: `max-width: 100%; height: auto` on all `img` elements.
6. Fix grids: 3-col desktop → 2-col tablet → 1-col mobile.
7. Fix buttons: 44px minimum height, full-width primary buttons on mobile.
8. Open DevTools: F12 → Ctrl+Shift+M → iPhone SE → iPad Mini → 1280px Responsive.
9. Note problems, return to Claude Code, fix with specific descriptions.
10. Run final audit prompt, apply any remaining fixes.

---

## Common mistakes

**Mistake 1: Writing a vague responsive prompt**

Prompt: "Make my site mobile friendly."

This gives Claude Code no information about breakpoints, which elements to change, or what the mobile layout should look like. The fix is incomplete and inconsistent.

Fix: Always name the three widths explicitly and describe the desired layout at each one. "On desktop (1200px+): two-column grid. On tablet (768px): two-column grid. On mobile (480px): single column, stacked vertically."

---

**Mistake 2: Testing in the browser by resizing the window by dragging**

Dragging a browser window to be narrow is not the same as DevTools device emulation. The browser does not simulate mobile pixel density, and some CSS media queries behave differently. You also cannot easily lock to an exact width.

Fix: Always use DevTools device toolbar (F12 → Ctrl+Shift+M). Select a named device like iPhone SE or iPad Mini. Type an exact pixel width into the width field at the top of the emulator bar.

---

**Mistake 3: Fixing responsive issues one tiny request at a time without context**

Sending ten separate single-sentence fix requests causes Claude Code to lose track of the full picture. It may fix one element and break another.

Fix: Batch related fixes into one prompt. Describe the element, the problem at each breakpoint, and the desired result. One thorough prompt beats five vague ones.

---

## Your turn

Open your landing page in Claude Code and run the full audit prompt from Step 2. Then fix the navigation using the prompt from Step 3.

Expected output: Claude Code will modify your `index.html` to include a hamburger menu on mobile. The navigation links will be hidden below 480px and revealed on tap.

Failure state: The hamburger icon appears but clicking it does nothing. This means the JavaScript event listener was not added, or it is targeting a class or ID that does not match the HTML. Fix: run this prompt in Claude Code:

```
The hamburger menu icon in my navigation is not working — clicking it does not show or hide the menu. Find the JavaScript that controls it and fix the event listener. Make sure the class or ID it targets matches the actual HTML element.
```

---

## Prompt / Template / Checklist pack

### Responsive Design Prompt Templates

**Template 1 — Responsive Navigation**

```
Make the navigation bar in [filename] fully responsive.

On desktop (1200px and above): horizontal layout, all links visible in a row.
On tablet (768px to 1199px): horizontal layout, reduce font size to 15px and horizontal padding to 12px.
On mobile (below 480px): hide all nav links. Show a hamburger icon (three 20px horizontal lines, 2px tall, spaced 5px apart) in the top-right corner. On click, reveal nav links in a vertical stack below the header, full width, each link 48px tall with 16px left padding. Clicking the hamburger again closes the menu.

Use only HTML, CSS, and inline vanilla JavaScript. No libraries.
```

---

**Template 2 — Responsive Images**

```
Make all images in [filename] responsive.

1. Apply max-width: 100% and height: auto to every img tag.
2. If any image has a fixed width set via an inline style or width attribute, remove the fixed width and replace with width: 100%; max-width: [original width]px.
3. For any image inside a flex or grid container, make sure the container does not constrain the image to overflow on screens narrower than 480px.
4. On mobile (below 480px), images should never touch the left or right edges of the screen — add a minimum of 16px horizontal padding on their parent container if not already present.
```

---

**Template 3 — Responsive Buttons and Tap Targets**

```
Make all buttons and button-styled links in [filename] responsive for mobile.

On desktop (1200px+): keep existing styles.
On tablet (768px): no changes needed unless buttons are currently smaller than 40px height — if so, set height to 40px.
On mobile (below 480px):
- Set minimum height to 44px on all buttons and anchor links styled as buttons.
- Set minimum width to 44px.
- Add vertical padding of 12px.
- Make all primary call-to-action buttons full width (width: 100%).
- Set font size to 16px minimum — iOS Safari auto-zooms on inputs and buttons with font sizes below 16px.
```

---

**Template 4 — Responsive Text and Typography**

```
Make all text in [filename] scale appropriately for mobile.

Apply these font size reductions using CSS media queries:

At 768px (tablet):
- Main page heading (h1): reduce to 80% of current size.
- Section headings (h2): reduce to 85% of current size.
- Body text: keep current size unless it is above 18px, in which case set to 17px.

At 480px (mobile):
- Main page heading (h1): set to 28px maximum.
- Section headings (h2): set to 22px maximum.
- Body text: set to 15px minimum, 16px maximum.
- Line height on body text: 1.6.

Do not change font family or weight, only size and line height.
```

---

**Template 5 — Responsive Grid Layouts**

```
Make every grid and multi-column flex layout in [filename] responsive.

For each layout found, apply these rules using CSS media queries:

At 768px (tablet):
- 4-column grids → 2 columns.
- 3-column grids → 2 columns.
- 2-column grids → keep 2 columns unless items are below 280px wide, in which case → 1 column.
- Reduce grid gap from current value by 4px if it is above 20px.

At 480px (mobile):
- All multi-column grids → 1 column, full width.
- Grid gap: 16px.
- Each card or grid item: add 16px of internal padding if not already present.

Do not change the desktop layout.
```

---

### Responsive Design Checklist

Run through this after completing your responsive fixes.

- [ ] Navigation works on mobile: hamburger icon visible, links hidden, tap to reveal
- [ ] No horizontal scroll at 375px (iPhone SE width)
- [ ] No horizontal scroll at 768px (iPad Mini width)
- [ ] Main heading is 28px or smaller on mobile
- [ ] All images have `max-width: 100%` applied
- [ ] All buttons are at least 44px tall on mobile
- [ ] Primary CTA buttons are full width on mobile
- [ ] Grid layouts collapse to single column on mobile
- [ ] Font sizes are minimum 15px on mobile
- [ ] Tested in DevTools: iPhone SE, iPad Mini, and 1280px Responsive
- [ ] No elements overlap each other at any tested width
- [ ] All text is readable without zooming on iPhone SE
