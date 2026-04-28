# Deploy Checklist
*Module 6 Deliverable — Vibe Code from Zero*

Run through every item before sharing your URL with anyone.

---

## Pre-Launch Checklist

### App behaviour
- [ ] Page loads at the live Netlify URL without errors or blank screen
- [ ] All buttons/interactive elements respond to clicks
- [ ] Hover states work on all interactive elements
- [ ] Dark/light mode toggle works and saves preference after page refresh
- [ ] Footer is visible with correct text
- [ ] Back to top button appears on scroll and disappears when back at top
- [ ] No placeholder text remains visible (replace "Alex" with intended name/content)

### Cross-device testing
- [ ] Tested on desktop browser (Chrome or Firefox recommended)
- [ ] Tested on mobile phone — open the URL on your actual phone
- [ ] No layout elements cut off, overflowing, or misaligned on small screens
- [ ] Text is readable without zooming on mobile

### Technical checks
- [ ] No console errors: press F12 → Console → no red text at all
- [ ] No hardcoded local file paths anywhere in the HTML (no `C:\Users\...` or `/Users/...`)
- [ ] All images load (no broken image icons)
- [ ] CLAUDE.md, fix-log.md, feature-log.md are not linked or visible to site visitors

### GitHub
- [ ] Latest version of all files pushed: `git push` completed without errors
- [ ] Netlify dashboard shows "Published" for the most recent deploy (not "Failed")

### URL
- [ ] Site renamed to something readable: Site configuration → Change site name
- [ ] URL copied and ready to paste or share

---

## Deploy Commands

**First-time deploy (new project):**
```
# Step 1: Add GitHub as remote (use your actual repo URL from GitHub)
git remote add origin https://github.com/YourUsername/your-repo.git

# Step 2: Set main as primary branch
git branch -M main

# Step 3: Push
git push -u origin main
```

Then: netlify.com → Add new site → Import from GitHub → select repo → leave Build command empty → Deploy site.

**Every deploy after the first:**
```
git add .
git commit -m "description of what changed"
git push
```
Netlify auto-deploys within ~60 seconds after each push.

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| Netlify deploy shows "Failed" | Build command is set and failing | Site configuration → Build settings → clear the Build command field |
| Page loads but shows folder listing | Netlify can't find index.html | Build settings → set Publish directory to the folder containing index.html |
| Page loads blank (white screen) | JavaScript file path is wrong | F12 → Console → find the file load error → check the `<script src="">` path in index.html |
| Changes not showing after push | Browser cache | Press Ctrl+Shift+R (Windows/Linux) or Cmd+Shift+R (Mac) to hard refresh |
| git push rejected for password | GitHub no longer accepts passwords | Create a personal access token (Settings → Developer settings → Personal access tokens) and use it as your password |
