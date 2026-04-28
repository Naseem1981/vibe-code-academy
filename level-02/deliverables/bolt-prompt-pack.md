# Bolt.new Prompt Pack — Level 2

10 ready-to-use prompts for practising Bolt.new in Week 1. Each prompt is specific enough to produce a high-quality result on the first run. Copy and paste the entire prompt block into Bolt.new's chat input.

---

## Prompt 1 — Coffee Shop Landing Page

```
Build a single-page landing website for a specialty coffee shop called "Grounds & Glory" using React and Tailwind CSS.

Sections:
1. Hero — full-viewport height, background image of coffee beans (use a placeholder at 1440x900), large heading "Where Every Cup Tells a Story", subheading, and a "View Our Menu" button in warm amber (#D97706)
2. Featured drinks — 3 card grid with image, drink name, short description, and price. Cards have a soft cream (#FEF9F0) background and rounded-xl corners
3. About us — two-column layout, image left, text right. Warm brand copy about artisan roasting
4. Location & hours — dark roast brown (#1C0A00) background, white text, two columns showing address and opening hours
5. Footer — logo, social links, copyright

Style: warm earthy palette (amber, cream, dark brown). Body font Inter, heading font Playfair Display. Smooth scroll. Hover states on all buttons and cards.
```

---

## Prompt 2 — Portfolio / Profile Card

```
Build a personal portfolio profile card page using React and Tailwind CSS for a developer named "Jordan Lee".

Layout: centred card, max-width 480px, on a dark slate background (#0F172A).

Card contents:
- Avatar image (circular, 96px, use placehold.co/96x96)
- Name in large bold text
- Title: "Full-Stack Developer & UI Designer"
- Short bio (2 sentences about building products with React and Node.js)
- 3 skill tags: React, TypeScript, Supabase — pill-shaped, emerald green (#10B981) background
- Social links row: GitHub, LinkedIn, Twitter — icon buttons with hover highlight
- "Download CV" button — full width, emerald green, white text, rounded-xl

Card style: white background, large shadow with slight green tint, 32px padding. Subtle entrance animation (fade up, 0.4s ease-out).
```

---

## Prompt 3 — Restaurant Menu Page

```
Build a restaurant menu page for "Mezze & Co." — a modern Mediterranean restaurant — using React and Tailwind CSS.

Sections:
1. Header — restaurant name in a serif font, tagline "Fresh. Bold. Mediterranean.", a sticky nav with menu category links
2. Menu categories — starters, mains, desserts. Each category is a section with a bold heading
3. Menu items — each item is a row with: item name (bold), description (grey text), dietary tags (pill badges: V = vegetarian, GF = gluten-free), and price right-aligned
4. At least 4 items per category with realistic Mediterranean dish names

Style: white background, text-zinc-900 for headings, text-zinc-500 for descriptions. Accent colour terracotta (#C2684A). Sticky header with a subtle bottom border. Mobile-responsive with single column on small screens.
```

---

## Prompt 4 — Todo App with CRUD

```
Build a fully functional Todo app using React and Tailwind CSS with localStorage persistence.

Features:
- Add a new todo: text input + "Add" button
- Mark todo as complete: checkbox on the left, completed todos get strikethrough text
- Delete a todo: trash icon button on the right of each row
- Filter tabs: All | Active | Completed
- Todo count: "X tasks remaining" shown below the list
- Clear completed button: removes all done todos at once

State: use React useState and useEffect to persist todos to localStorage so they survive a page refresh.

Design: dark theme (#111827 background, #1F2937 card). White text, indigo-500 for the add button and active filter tab. Each todo row has a hover background highlight. Input has a focus ring in indigo.
```

---

## Prompt 5 — Weather App (Open-Meteo API)

```
Build a weather app using React and Tailwind CSS that fetches real weather data from the Open-Meteo API (no API key required).

Features:
- Search box: user types a city name, app geocodes it using the Open-Meteo geocoding API (https://geocoding-api.open-meteo.com/v1/search)
- On search, fetch current weather from https://api.open-meteo.com/v1/forecast with parameters: temperature_2m, weathercode, windspeed_10m, relative_humidity_2m
- Display: city name, current temperature in Celsius, weather condition (map WMO weathercode to a label like "Clear", "Cloudy", "Rain"), wind speed, humidity
- Show a weather emoji based on condition (sun for clear, cloud for cloudy, umbrella for rain, snowflake for snow)
- Loading skeleton while fetching
- Error message if city not found

Design: gradient background from sky-400 to blue-600. White card with rounded-2xl, shadow-xl. Bold temperature display at 72px font size. Subtle backdrop blur on the card.
```

---

## Prompt 6 — Simple Blog Homepage

```
Build a blog homepage called "The Build Log" for a developer who writes about coding and building products, using React and Tailwind CSS.

Sections:
1. Header — blog name, tagline "Lessons from shipping real products", navigation links (Home, Articles, About)
2. Featured post — large card with a cover image (placehold.co/800x400), category badge, title, excerpt (2-3 sentences), author avatar + name, read time, and a "Read more" link
3. Recent posts grid — 3-column on desktop, 1-column on mobile. Each card: image (placehold.co/400x240), category, title, excerpt, date
4. Newsletter CTA — full-width section with email input and "Subscribe" button
5. Footer — copyright, social links

Style: clean white background, zinc colour palette for text, accent colour electric purple (#7C3AED). Headings in a serif font (Georgia or Playfair Display). Cards have subtle hover shadow lift animation.
```

---

## Prompt 7 — Event Landing Page

```
Build an event landing page for a tech conference called "BuildCon 2025" using React and Tailwind CSS.

Sections:
1. Hero — dark background (#0A0A0A), large heading "BuildCon 2025", subheading "The conference for developers who ship", date badge "12–13 September 2025 · Cape Town", and two buttons: "Get Tickets" (accent orange #F97316) and "View Schedule" (outline)
2. Stats row — 3 columns: 500+ Attendees, 30+ Speakers, 2 Days. Large numbers, bold
3. Speaker lineup — 6 speaker cards in a 3x2 grid. Each: circular avatar (placehold.co/80x80), name, role/company
4. Schedule preview — 2 tabs (Day 1 / Day 2), each showing 4 time slots with talk title and speaker name
5. Venue — map placeholder image (placehold.co/800x300), venue name, address
6. Ticket CTA — dark section, heading "Secure your seat", pricing tiers (Early Bird R1,500 / Standard R2,500), button

Style: dark mode throughout. Orange accent. Mono font for times and codes.
```

---

## Prompt 8 — Product Showcase Page

```
Build a product showcase page for a fictional SaaS tool called "FlowDesk — Task Management Reimagined" using React and Tailwind CSS.

Sections:
1. Navbar — logo text "FlowDesk", nav links (Features, Pricing, Log in), "Start free" button in violet (#7C3AED)
2. Hero — centred, large heading with a violet gradient text effect, subheading, product screenshot mockup (placehold.co/900x500 with a browser chrome frame around it), two CTA buttons
3. Features — 3-column grid of feature cards. Each card: icon (use an emoji or simple SVG), bold title, 1-sentence description. Features: Drag & drop boards, Team collaboration, Smart reminders
4. Testimonials — 2 quote cards side by side. Each: quote text, avatar (placehold.co/48x48), name, role
5. Pricing — 2 plan cards side by side: Free (R0/month) and Pro (R299/month). List 5 features per plan. Pro card has a violet border and "Most popular" badge
6. Footer — logo, links, copyright

Style: white/light grey background. Violet (#7C3AED) as primary accent. Rounded-2xl cards, soft shadows.
```

---

## Prompt 9 — Simple Calculator

```
Build a fully functional calculator app using React and Tailwind CSS.

Functionality:
- Standard arithmetic: addition, subtraction, multiplication, division
- Decimal point support
- Clear (AC) button that resets to 0
- Plus/minus toggle (±)
- Percentage button (%)
- Display shows current input and the running expression above it (like iOS calculator)
- Handle division by zero gracefully (show "Error")
- Keyboard support: number keys, +/-/*/ keys, Enter for equals, Escape for clear, Backspace for delete

Design: dark grey calculator body (#1C1C1E), buttons in dark grey (#2C2C2E) for numbers, slightly lighter for operators, orange (#FF9F0A) for the equals button and operator highlights. Rounded-2xl overall, rounded-full buttons. Display right-aligned, large font (48px for result, 20px for expression).
```

---

## Prompt 10 — Countdown Timer App

```
Build a countdown timer app using React and Tailwind CSS.

Features:
- User inputs a target date and time using a datetime-local input
- Timer displays: Days, Hours, Minutes, Seconds in a 4-block grid — each block shows the number large and the label below
- Numbers count down in real time (update every second using setInterval)
- When the timer reaches zero, show a "Time's up!" message with a confetti-style animation (use CSS keyframes to animate coloured dots falling)
- A "Reset" button clears the timer and returns to the input screen
- A preset button row: "+ 1 Hour", "+ 24 Hours", "+ 7 Days" — clicking these sets the target time relative to now

Design: dark background (#0D1117), card in dark blue-grey (#161B22). Timer blocks in rounded-xl with a subtle border (#30363D). Number text in white at 64px bold. Labels in slate-400 at 14px uppercase. Accent colour electric blue (#58A6FF). Smooth transition on the number change.
```
