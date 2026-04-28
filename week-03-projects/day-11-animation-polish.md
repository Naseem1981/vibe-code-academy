# Day 11: Animation and Polish

## Outcome artifact

By the end of this day you will have your existing landing page updated with four working animations — page fade-in on load, button hover lift, back-to-top button fade, and smooth scroll behaviour — all verified in the browser.

---

## The core idea

There is a gap between an app that works and an app that feels good. When something appears instantly with no transition, the brain registers it as a jump. When a button responds to a hover with a subtle lift, the brain registers it as physical — something you can push. That physical feedback is what makes users trust an interface. Animation is not decoration. It is communication.

The rule for performant animation is simple: only ever animate `transform` and `opacity`. These two properties are handled directly by the GPU and never trigger a layout recalculation. Everything else — `width`, `height`, `color`, `background`, `margin`, `padding` — forces the browser to recalculate the entire page layout on every frame. The result is jank: stuttering, dropped frames, sluggish feel. If you find yourself wanting to animate a color change, animate an `opacity` overlay instead. If you want to animate width, use a `scaleX` transform instead.

Every app needs four animations as a baseline. First, a fade-in on page load — the whole page eases in over 0.4 seconds so the user's eye has time to land. Second, a hover lift on buttons — a `translateY(-2px)` with a faint shadow increase gives every clickable element physical depth. Third, a slide-in for new items added to a list — new content announces itself without shouting. Fourth, a fade-out on removal — items that disappear abruptly feel like bugs; a 0.2-second fade-out feels intentional. Beyond these four, every state the user waits through needs a visual — a loading skeleton while data fetches, a clear empty state when a list is empty, and a friendly error state when something goes wrong.

---

## Step-by-step walkthrough

**Step 1: Open your project in Claude Code**

Navigate to your landing page project folder and open Claude Code.

```
claude
```

**Step 2: Check what you currently have**

Before adding animation, understand the structure. Ask Claude Code to describe the page.

```
List all the interactive elements on this page — buttons, links, form inputs. Also list any sections that load content dynamically.
```

**Step 3: Add the page fade-in on load**

This is the foundational animation. The entire page eases in when it first renders.

```
Add a fade-in animation to the page on load. Requirements:
- The body starts at opacity 0 and transitions to opacity 1
- Duration: 0.4s
- Easing: ease-out
- Trigger: on DOMContentLoaded
- Use only CSS animation (no JavaScript animation libraries)
- Do not animate any layout properties — only opacity
```

**Step 4: Add hover lift to all buttons**

Every button on the page should respond to hover with a subtle physical lift.

```
Add a hover lift effect to every button and anchor tag styled as a button on this page. Requirements:
- On hover: translateY(-2px) and a slightly stronger box shadow
- On active (click): translateY(0px) — snaps back on press
- Transition duration: 0.15s for hover-in, 0.1s for active snap-back
- Easing: ease-out
- Only animate transform and box-shadow — nothing else
- Apply using a CSS class so it works on all buttons including any added later
```

**Step 5: Add a back-to-top button with fade behaviour**

```
Add a back-to-top button to the page. Requirements:
- Fixed position, bottom-right corner, 24px from each edge
- Shows only when the user has scrolled more than 300px down the page
- Fades in when it appears (opacity 0 to 1, 0.2s ease-out, transform: translateY(0) from translateY(8px))
- Fades out when hidden (opacity 0, translateY(8px), 0.2s ease-in)
- Clicking it scrolls smoothly to the top
- Only animate opacity and transform — no display toggling that causes jank
- Style it to match the existing button style on the page
```

**Step 6: Add smooth scroll behaviour**

```
Add smooth scrolling to all anchor links on this page that point to an ID on the same page (href starting with #). Use CSS scroll-behavior: smooth on the html element. Also add a scroll-margin-top of 80px to all section elements so they are not hidden under a fixed header when navigated to.
```

**Step 7: Add a slide-in animation for list items**

If your landing page has a list of features, testimonials, or cards, add entrance animations.

```
Find any list of cards, features, or testimonial items on this page. Add a staggered slide-up entrance animation. Requirements:
- Each item starts at opacity 0 and translateY(20px)
- Animates to opacity 1 and translateY(0)
- Duration: 0.35s per item
- Easing: ease-out
- Stagger delay: 0.08s between each item (first item: 0s, second: 0.08s, third: 0.16s, etc.)
- Trigger: when the section enters the viewport (use IntersectionObserver)
- Only animate opacity and transform
```

**Step 8: Verify all animations in the browser**

Open your browser and check each animation manually:

1. Reload the page — confirm the fade-in plays.
2. Hover over every button — confirm the lift happens.
3. Scroll down past 300px — confirm the back-to-top button fades in.
4. Click a nav link — confirm smooth scroll happens.
5. Scroll to the cards/features section — confirm staggered entrance plays.

If any animation does not work, paste the relevant HTML and CSS back to Claude Code with a specific description of what you see versus what you expected.

**Step 9: Fix any jank**

```
Check the animations on this page. Are any of them animating properties other than transform and opacity? If yes, rewrite them to only use transform and opacity.
```

**Step 10: Final polish pass**

```
Do a final pass on all animations on this page. Check:
1. No animation lasts longer than 0.5s
2. No animation uses linear easing (should be ease-out or cubic-bezier)
3. All hover states have a matching active (pressed) state
4. The page does not flash or jump on load

Fix any issues found.
```

---

## Practical workflow

1. Open Claude Code in project folder: `claude`
2. Prompt: describe current interactive elements
3. Add page fade-in — prompt Step 3
4. Add button hover lift — prompt Step 4
5. Add back-to-top button — prompt Step 5
6. Add smooth scroll — prompt Step 6
7. Add staggered list entrance — prompt Step 7
8. Open browser, reload, manually test all 5 animation points
9. Paste broken sections back to Claude Code with exact failure description
10. Run jank check — prompt Step 9
11. Run final polish pass — prompt Step 10
12. Reload browser one last time, confirm all animations play correctly
13. `git add -A && git commit -m "feat: add page animations and polish"`

---

## Common mistakes

**Mistake 1: Animating `display: none` to `display: block`**

The animation does not play. `display` cannot be transitioned — the browser skips the transition entirely and jumps between states.

Fix: Never toggle `display` for animated elements. Instead, toggle `opacity` from 0 to 1 and `visibility` from `hidden` to `visible`. Or use `pointer-events: none` to make an element non-interactive without hiding it from the render tree. Prompt Claude Code:

```
Replace any display:none/block toggles used for animated elements with opacity and visibility toggles instead.
```

**Mistake 2: Applying `transition: all`**

This transitions every CSS property simultaneously, including layout properties. It causes jank and unexpected transitions on properties you did not intend to animate (e.g., padding changes during responsive resize).

Fix: Always specify exact properties. If Claude Code writes `transition: all`, correct it immediately:

```
Replace every instance of "transition: all" with explicit transition declarations. Only transition transform, opacity, and box-shadow.
```

**Mistake 3: Stagger animation delay makes the page feel slow**

Setting stagger delays too high (0.3s+ between items) makes the page feel like it is loading slowly rather than animating elegantly.

Fix: Keep stagger delays at 0.05–0.1s maximum. If there are more than 8 items in a staggered list, cap the maximum delay at 0.6s total so the last items are not still animating after the user has moved on.

```
The stagger animation delay is too long. Cap the per-item delay at 0.08s and set a maximum total delay of 0.6s regardless of how many items are in the list.
```

---

## Your turn

Add all four animations to your landing page using the prompts in the walkthrough above, then open the page in your browser and trigger each animation manually.

Expected output: The page fades in on load (you see it ease in rather than snap on), buttons visibly lift when you hover over them, the back-to-top button appears after scrolling and fades out when you return to the top, and any card/feature lists animate in as you scroll down.

Failure state: Animations are not playing at all. Most likely cause: the CSS was added but the JavaScript trigger is missing or the element selectors in the JS do not match the actual HTML. Fix: paste the current HTML for the affected element and the current animation CSS/JS to Claude Code and ask "The animation is not triggering — the selector or event listener may be wrong. Fix it."

---

## Prompt / Template / Checklist pack

### Animation Prompt Pack — 8 Copy-Paste Prompts

**1. Fade in on page load**
```
Add a fade-in animation to the entire page on load. The body starts at opacity 0 and eases to opacity 1 over 0.4s using ease-out. Trigger on DOMContentLoaded. Only use opacity — do not animate any layout properties.
```

**2. Slide up entrance**
```
Add a slide-up entrance animation to [element]. It starts at opacity 0 and translateY(24px) and animates to opacity 1 and translateY(0) over 0.35s with ease-out easing. Trigger when the element enters the viewport using IntersectionObserver with a threshold of 0.1.
```

**3. Hover lift on buttons**
```
Add a hover lift to all buttons. On hover: translateY(-2px) with a slightly deeper box-shadow. On active: translateY(0). Transition: 0.15s ease-out for hover, 0.1s ease-out for active. Only animate transform and box-shadow.
```

**4. Pulse animation (for notifications or badges)**
```
Add a subtle pulse animation to [element — e.g. a notification dot]. It should scale from 1 to 1.08 and back to 1, repeating indefinitely. Duration: 1.4s. Easing: ease-in-out. Only animate transform (scale). Do not animate color or opacity.
```

**5. Shake animation for form errors**
```
Add a shake animation that triggers on [element — e.g. a form input] when it receives an error state (class: .error). The shake should use translateX, oscillating between -6px and +6px over 0.4s total. It should play once and stop. Only animate transform.
```

**6. Skeleton loading screen**
```
Add a skeleton loading state to [element — e.g. a card list]. While loading is true, show placeholder rectangles in place of the actual content. The placeholders should have a shimmer effect: a linear gradient that sweeps left to right over 1.5s on repeat. Only animate background-position using transform where possible. Remove skeletons and show real content when loading becomes false.
```

**7. Staggered list entrance**
```
Add a staggered entrance animation to the items inside [element — e.g. ul.features-list]. Each item starts at opacity 0 and translateY(16px) and animates to opacity 1 and translateY(0). Duration per item: 0.3s. Easing: ease-out. Stagger delay: 0.07s per item. Cap total stagger at 0.56s (8 items max). Trigger with IntersectionObserver when the list enters the viewport.
```

**8. Smooth scroll with active nav highlight**
```
Add smooth scrolling to all same-page anchor links. Set scroll-behavior: smooth on the html element. Add a scroll-margin-top of 80px to all section elements. Also: as the user scrolls, highlight the corresponding nav link (add class .active) based on which section is currently in view. Use IntersectionObserver to track sections.
```

---

### Animation Quality Checklist

Before shipping any page with animations, run through this list:

- [ ] No animation lasts longer than 0.5s (except looping animations like pulse/skeleton)
- [ ] No animation uses `linear` easing — use `ease-out`, `ease-in-out`, or `cubic-bezier`
- [ ] No `transition: all` anywhere in the stylesheet
- [ ] No layout properties animated: no `width`, `height`, `margin`, `padding`, `top`, `left`, `color`, `background-color`
- [ ] Only `transform` and `opacity` animated (plus `box-shadow` on hover states is acceptable)
- [ ] Every button has hover, active (pressed), and focus-visible states
- [ ] No `display: none` toggled on animated elements — use `opacity` + `visibility` instead
- [ ] Page does not flash on load
- [ ] Stagger delays do not exceed 0.1s per item
- [ ] All animations tested in browser by hand (not just assumed to work from code)
- [ ] IntersectionObserver used for scroll-triggered animations (not scroll event listener)
- [ ] `prefers-reduced-motion` media query respected — add `@media (prefers-reduced-motion: reduce)` block that sets all animation durations to 0.01ms
