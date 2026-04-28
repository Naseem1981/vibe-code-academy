# Day 3: Build a Real Landing Page with Bolt.new

## Outcome artifact
By the end of this day you will have a complete, multi-section landing page for "Roast & Rise" coffee shop — with hero, menu, about, and contact form — running in Bolt.new's live preview and responsive on mobile.

---

## The core idea

Building a landing page with Bolt.new is not a one-shot operation. You do not write one giant prompt and hope for the best. You build in layers. Each prompt adds one well-defined piece to what already exists. This layered approach gives you control: if one prompt produces something wrong, the fix is isolated and small. If you try to specify everything at once, debugging a bad result means starting over.

The four prompts in today's build follow a deliberate order: structure first, navigation second, responsiveness third, visual polish last. Structure without style is easy to fix. Style without structure is a mess. Lock in the HTML and content before you think about colour.

Bolt.new has two features that make this workflow practical. The **Diff view** shows you exactly which lines changed between your last prompt and the current version — like a Git diff, directly in the browser. The **Undo** button (in the top toolbar, labelled with a backward arrow) reverts the entire project to the state before your last prompt. These two tools together mean you can move fast without fear: inspect what changed, and roll back anything that breaks something that was already working.

---

## Step-by-step walkthrough

### Step 1 — Open Bolt and start a new project

1. Go to `https://bolt.new`.
2. Click inside the prompt box to start a new project.

---

### Step 2 — Prompt 1: Full page structure

Paste this prompt into the Bolt.new prompt box exactly as written:

```
Build a landing page for a coffee shop called "Roast & Rise" using React and Tailwind CSS.

The page must have four sections in this order:

1. HERO SECTION
- Full-viewport-height section
- Large headline: "Wake up to something better"
- Subheadline: "Specialty coffee, freshly roasted every morning, served with care"
- Two CTA buttons: "See Our Menu" (primary) and "Find Us" (secondary)
- Background: a placeholder image (https://placehold.co/1400x800) with a dark overlay so text is readable

2. MENU SECTION
- Section heading: "What We Brew"
- Grid of 6 menu items, 3 columns on desktop
- Each item has: name, short description (one sentence), and price
- Items:
  - Flat White — Velvety espresso with steamed milk. R42
  - Cold Brew — 18-hour slow-steeped, served over ice. R48
  - Cortado — Equal parts espresso and warm milk. R38
  - Oat Latte — Our house espresso with creamy oat milk. R46
  - Filter Coffee — Single-origin pour-over, black. R35
  - Chai Latte — Spiced tea concentrate with steamed milk. R40

3. ABOUT SECTION
- Section heading: "Our Story"
- Two-column layout: text on the left, image on the right (https://placehold.co/600x400)
- Body text (3 short paragraphs):
  Paragraph 1: "Roast & Rise opened in 2019 in a converted warehouse in Cape Town's Woodstock neighbourhood. We started with one espresso machine and a single-origin Ethiopian bean."
  Paragraph 2: "Everything we serve is roasted in-house, three times a week. We cup every batch before it hits the grinder. If it's not right, it doesn't go out."
  Paragraph 3: "We believe great coffee is a daily ritual, not a luxury. Our prices reflect that."

4. CONTACT SECTION
- Section heading: "Get In Touch"
- A contact form with: Name field, Email field, Message textarea, Submit button
- Form should be centered, max width 600px
- Below the form: address line "12 Albert Road, Woodstock, Cape Town" and a phone number "021 123 4567"

No navigation bar yet — just the four sections. Use placeholder colours for now (we'll add the colour scheme later).
```

Wait for Bolt to generate. The preview should show all four sections rendering with content.

---

### Step 3 — Check the Diff view after Prompt 1

After Bolt finishes generating:

1. Look for the **Diff** button or icon near the top of the code editor panel (it may appear as a split-view icon or the label "Changes").
2. Click it. You will see a colour-coded view of every file that was created or modified:
   - Green lines = added
   - Red lines = removed
   - Grey/unchanged lines = context

Because this is the first prompt, everything will be green (all added). Use this view to confirm that Bolt created the files you expected: `App.tsx`, component files for each section if it split them, and updated `index.css`.

Click **Diff** again to toggle back to the normal editor view.

---

### Step 4 — Prompt 2: Sticky navigation bar

Type this directly into the Bolt chat/prompt input (not a new project — continue in the same session):

```
Add a sticky navigation bar to the top of the page.

Requirements:
- Stays fixed at the top when the user scrolls (position: sticky or fixed)
- Left side: the logo text "Roast & Rise" in a slightly stylised font weight
- Right side: four navigation links — Home, Menu, Our Story, Contact
- Each link uses smooth scrolling to jump to the correct section (use anchor IDs on each section)
- Add appropriate id attributes to each of the four sections: id="hero", id="menu", id="about", id="contact"
- The nav bar has a semi-transparent white background with a subtle backdrop blur so the page content shows faintly through it when scrolled
- On desktop, links are horizontal in a row. Do not add a mobile hamburger menu yet — we will do that in the next step.
```

After generation, scroll the preview. Confirm the nav sticks to the top and the links jump to the correct sections.

---

### Step 5 — Check what changed with Diff view

1. Click the **Diff** button again.
2. You should see changes in `App.tsx` (or whatever file holds the page structure) — the `id` attributes added to sections, and a new navigation component or section added at the top.
3. If the diff shows changes to a file you didn't expect (like `index.css` being deleted entirely), that is a signal something went wrong. Click **Undo** (the backward arrow in the top toolbar) to revert to before Prompt 2, then rewrite the prompt with more specific language.

---

### Step 6 — How to undo a bad prompt

If Bolt produces something that breaks what was already working:

1. Do not type a new prompt trying to fix it. That often makes things worse.
2. Click the **Undo** button (backward arrow icon) in the top toolbar of the Bolt interface.
3. Bolt reverts the entire project to the state before your last prompt.
4. Now retype your prompt with more precise language. Specify what should NOT change as well as what should change. Example: `"Add X to the nav bar. Do not change any of the existing sections or their styling."`

Undo only goes back one step in the Bolt free plan. Do not rely on it as a multi-step history — use it only immediately after a bad prompt.

---

### Step 7 — Prompt 3: Mobile responsiveness and hamburger menu

```
Make the page fully responsive for mobile screens.

Changes needed:

Navigation:
- On mobile (screens smaller than 768px), hide the horizontal nav links
- Show a hamburger menu icon (three horizontal lines) on the right side of the nav bar
- Clicking the hamburger opens a vertical dropdown menu with the same four links
- Clicking a link closes the dropdown
- Use useState to control open/closed state of the mobile menu

Hero section:
- On mobile, reduce the headline font size so it fits without overflowing
- Stack the two CTA buttons vertically on mobile

Menu section:
- Change the 3-column grid to 1-column on mobile, 2-column on tablet (768px+), 3-column on desktop (1024px+)

About section:
- Change the two-column layout to a single column on mobile (image stacks below the text)

Contact form:
- Already centered with max-width, should be fine — confirm it has horizontal padding on mobile so it doesn't touch screen edges

Do not change any existing content, text, or placeholder images.
```

After generation, use the **device toggle** in Bolt's preview panel. This is the phone/tablet icon at the top of the preview — click it to switch the preview to a mobile viewport (usually 375px wide). Confirm:
- The hamburger menu appears.
- Tapping it opens the dropdown.
- The menu grid is single column.
- The about section stacks vertically.

---

### Step 8 — Prompt 4: Colour scheme and visual polish

```
Apply a warm coffee shop colour scheme to the entire page.

Colour palette:
- Primary background: #1a0f0a (very dark espresso brown)
- Card/surface background: #2d1a10 (dark roast brown)
- Accent colour: #c8893a (warm amber/caramel)
- Body text: #e8d5c4 (warm cream)
- Muted text: #9c7b6a (dusty rose-brown)
- White elements: #fdf6f0 (warm off-white)

Apply these colours:
- Page background: #1a0f0a
- Nav bar background: semi-transparent #1a0f0a with backdrop-blur
- Hero overlay: dark gradient from #1a0f0a at 80% opacity over the placeholder image
- Hero headline: #fdf6f0, Hero subheadline: #e8d5c4
- CTA primary button: #c8893a background, #1a0f0a text, hover darken to #a06e2a
- CTA secondary button: transparent with #c8893a border and text, hover fill #c8893a with #1a0f0a text
- Menu section background: #1a0f0a, menu card background: #2d1a10
- Menu card border: 1px solid #3d2418
- Menu item name: #fdf6f0, description: #9c7b6a, price: #c8893a
- About section background: #2d1a10
- About text: #e8d5c4
- Contact section background: #1a0f0a
- Form inputs: #2d1a10 background, #e8d5c4 text, #3d2418 border, focus border #c8893a
- Submit button: #c8893a background, #1a0f0a text
- Nav links: #e8d5c4, hover colour #c8893a

Do not change any text content, layout, or responsive behaviour. Only change colours and any minimal spacing improvements needed to make the dark theme look balanced.
```

After generation, scroll through the entire preview. Confirm every section has the warm brown palette and no section is showing default Tailwind blues or greys.

---

## Practical workflow

1. Open bolt.new, start a new project.
2. Paste Prompt 1 — full page structure.
3. Confirm all four sections render in the preview.
4. Click Diff — verify the files created match expectations.
5. Paste Prompt 2 — sticky nav bar.
6. Scroll the preview — confirm nav sticks and anchor links work.
7. If a prompt breaks something, click Undo immediately, then rewrite the prompt.
8. Paste Prompt 3 — mobile responsiveness.
9. Toggle the device preview to mobile — confirm hamburger and stacked layout.
10. Paste Prompt 4 — colour scheme.
11. Scroll all sections in the preview — confirm warm brown palette throughout.
12. Download the project via the three-dot menu → Download Project.

---

## Common mistakes

**Mistake 1 — Writing vague prompts like "make it look nicer" or "improve the styling".**

Bolt does not know what "nicer" means. It will make arbitrary changes — some you wanted, some you didn't — and you will spend time trying to revert parts of what it did. Every prompt must say what to change, where to change it, and to what value. Instead of "make it nicer", write: "Increase the menu card padding from the current value to 24px on all sides. Add a 1px border in colour #3d2418 around each card."

**Mistake 2 — Manually editing the code in Bolt's editor, then prompting on top of those edits.**

When you prompt Bolt, it regenerates the affected files. Any manual edits you made to those files will be overwritten. Manual edits in Bolt are safe only for the absolute last change you make before downloading. If you want to change a heading text, a button label, or a single colour value and you're done prompting — edit manually. If you have more prompts coming, write a prompt instead.

**Mistake 3 — Declaring the page done without checking mobile in the preview.**

Landing pages look fine on the desktop preview and completely broken on mobile — nav links overflowing, text too large, images cropped wrong. Before you download or show the page to anyone, always click the device toggle icon in Bolt's preview to switch to mobile view. Scroll through every section on the mobile viewport. If anything is broken, write a targeted fix prompt before downloading.

---

## Your turn

Start a new Bolt.new project right now. Run all four prompts in sequence — structure, nav, mobile, colour. After Prompt 4, click the device toggle in the preview and scroll through the page on mobile. Then switch back to desktop and scroll through again.

**Expected output:** A dark-themed, warm-brown coffee shop landing page with a sticky nav, four distinct sections, a working hamburger menu on mobile, and no default Tailwind blue or grey anywhere on the page.

**Failure state 1:** After Prompt 2, clicking a nav link does not scroll to the correct section. Fix: the section `id` attributes are probably missing or mismatched with the link `href` values. Write a prompt: `"Check that each section has an id attribute matching its nav link href. The hero section should have id='hero', the menu section id='menu', the about section id='about', and the contact section id='contact'. Fix any mismatches."`

**Failure state 2:** After Prompt 3, the hamburger menu opens but the dropdown links don't close the menu after clicking. Fix: write a prompt: `"When a user clicks a nav link in the mobile dropdown menu, close the dropdown. Add an onClick handler to each mobile nav link that sets the menu open state to false."`

**Failure state 3:** After Prompt 4, the form input text is invisible (dark text on dark background). Fix: write a prompt: `"The contact form input text is not visible. Set the input text colour explicitly to #e8d5c4 and make sure the placeholder text colour is #9c7b6a. The background of each input should be #2d1a10."`

---

## Prompt / Template / Checklist pack

The four build prompts appear in the walkthrough above (Steps 2, 4, 7, 8). Here are the three bonus prompts.

---

### Bonus Prompt 1 — Add a testimonials section

```
Add a testimonials section between the menu section and the about section.

Section heading: "What our regulars say"

Three testimonials in a row (3 columns on desktop, 1 column on mobile):

Testimonial 1:
Quote: "I've tried every coffee shop in Cape Town. Roast & Rise is the only one I come back to every single day."
Name: Amahle Dlamini
Tag: Regular since 2020

Testimonial 2:
Quote: "The cold brew changed my mornings completely. I can't start work without it."
Name: Ryan Petersen
Tag: Cold brew devotee

Testimonial 3:
Quote: "They know my order before I reach the counter. That says everything."
Name: Farah Moosa
Tag: Daily flat white

Each testimonial is a card with: the quote text in italics, a horizontal rule, the name in bold, and the tag in muted text below.
Give the section the same dark background (#1a0f0a) and apply the existing card style (#2d1a10 background, #3d2418 border).
Add the testimonials section to the nav bar as a fifth link: "Reviews" — link to id="testimonials".
```

---

### Bonus Prompt 2 — Add a photo gallery section

```
Add a photo gallery section after the about section and before the contact section.

Section heading: "Inside Roast & Rise"
Section id: "gallery"

Gallery layout: a CSS grid with 6 images arranged in an asymmetric masonry-style layout:
- Row 1: one wide image (https://placehold.co/800x400) spanning 2 columns, one tall image (https://placehold.co/400x400) in the third column spanning 2 rows
- Row 2: two square images (https://placehold.co/400x300) side by side in the first two columns
- Row 3: one wide image (https://placehold.co/800x300) spanning all 3 columns

Each image should have:
- border-radius: 8px
- A dark overlay on hover with the text "Roast & Rise" centred in amber (#c8893a)
- A smooth opacity transition on the overlay (0.3s ease)

On mobile: switch to a 2-column grid of equal-width square images, no masonry.
Apply the same dark background (#1a0f0a) as the rest of the page.
Add "Gallery" as a link in the sticky nav bar.
```

---

### Bonus Prompt 3 — Add a Google Maps embed

```
Add a map to the contact section, displayed to the right of the contact form.

Layout change: the contact section should now be a two-column layout on desktop.
- Left column: the existing contact form (keep it exactly as is)
- Right column: a Google Maps embed

For the map embed, use this iframe (it shows a generic Cape Town location):
<iframe
  src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d3310.123456789!2d18.4536!3d-33.9249!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x0%3A0x0!2zMzPCsDU1JzI5LjYiUyAxOMKwMjcnMTMuMCJF!5e0!3m2!1sen!2sza!4v1234567890"
  width="100%"
  height="400"
  style="border:0; border-radius:8px;"
  allowfullscreen=""
  loading="lazy"
></iframe>

Below the map, show:
- Address: "12 Albert Road, Woodstock, Cape Town" in #e8d5c4
- Phone: "021 123 4567" in #c8893a
- Hours: "Mon–Fri 6:30am–4pm | Sat–Sun 7am–3pm" in #9c7b6a

On mobile, stack the form above the map in a single column.
Do not move or remove the existing contact form.
```
