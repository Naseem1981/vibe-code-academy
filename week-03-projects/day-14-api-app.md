# Day 14: API-Powered App (Full Build)

## Outcome artifact

By the end of this day you will have a deployed Movie Explorer app — live at a GitHub Pages URL — where users can search for any movie, see real results with posters and ratings, click a movie to read the full plot and cast, and save favourites that persist across page refreshes.

---

## The core idea

An API-powered app is any app that fetches live data from a third-party service instead of storing that data itself. The Movie Explorer fetches data from OMDB (Open Movie Database) — a free service that returns JSON data about movies, including titles, posters, ratings, plot summaries, cast, and director. You give it a search term and an API key; it gives you back a structured list of results. Your app's only job is to display that data clearly.

The OMDB free tier gives you 1,000 requests per day, which is more than enough to build and test a personal project. You register once with your email address, receive a key in your inbox, and that key goes directly in your JavaScript file. This is different from production apps where keys are hidden on a server — for a personal HTML/JS project there is no server, so the key lives in the code. That is acceptable for a personal project that is not handling money or private user data. Never do this with payment keys or secret credentials.

The build today follows the one-feature-at-a-time rule you have been practising. You do not ask Claude Code to build the entire app in one prompt. You scaffold the structure first, add search functionality second, add the detail modal third, add favourites fourth, and add the favourites view fifth. Each step produces working code you can run in your browser before moving to the next. This is how real apps are built — incrementally, with a working state after every change.

---

## Step-by-step walkthrough

### Step 1: Get your free OMDB API key

1. Open your browser and go to `https://www.omdbapi.com/apikey.aspx`
2. Select **FREE** (1,000 daily limit)
3. Enter your email address, fill in the form, and click **Submit**
4. Check your inbox — you will receive an email with your API key (a short alphanumeric string like `a1b2c3d4`)
5. Keep this email open. You will paste the key into a prompt in the next step.

---

### Step 2: Create the project folder

1. Open a terminal (Command Prompt, PowerShell, or the terminal inside Cursor)
2. Run the following commands:

```bash
mkdir movie-explorer
cd movie-explorer
```

3. Open Claude Code in this folder:

```bash
claude
```

---

### Step 3: Scaffold — search bar and empty results grid

In the Claude Code terminal, run this prompt:

```
Build an HTML/CSS/JS single-page app called Movie Explorer.

Requirements:
- Single index.html file with all styles and scripts inline
- A header with the app name "Movie Explorer"
- A search section with a text input (placeholder "Search for a movie...") and a Search button
- A results section below with a grid layout — 4 columns on desktop, 2 on tablet, 1 on mobile
- Each grid cell will be a movie card (leave the grid empty for now — just the container)
- A "Favourites" button in the header (top right) that does nothing yet
- Colour scheme: dark background (#0f0f0f), white text, accent colour #e50914 (Netflix red)
- Font: system font stack
- The search input and button should sit on the same row, centred on the page
- No placeholder images yet — the cards are empty

Do not add any JavaScript functionality yet. This is just the visual scaffold.
```

After Claude Code responds, open `index.html` in your browser (double-click the file or drag it into Chrome) and confirm the layout looks right before continuing.

---

### Step 4: Add search functionality

In Claude Code:

```
Add search functionality to the Movie Explorer.

Context:
- OMDB API endpoint: https://www.omdbapi.com/?apikey=YOUR_KEY_HERE&s=SEARCH_TERM&type=movie
- Replace YOUR_KEY_HERE with the actual key string in the code — I will swap it myself
- The API returns a JSON object. When results exist: { Search: [ { Title, Year, Poster, imdbID }, ... ] }
- When no results: { Response: "False", Error: "Movie not found!" }

Requirements:
- When the user clicks Search (or presses Enter in the input), fetch from OMDB using the search term
- Display results as cards in the grid. Each card shows:
  - The movie poster image (use the Poster URL from the API — if Poster is "N/A", show a grey placeholder div with the movie title centred)
  - The movie title (bold)
  - The year
- Show a loading state ("Searching...") in the results area while the fetch is in progress
- Show an error message if the API returns no results or if the fetch fails
- The cards should have a subtle hover effect (scale up slightly, show a border)

Add a constant at the top of the script section: const API_KEY = 'REPLACE_WITH_YOUR_KEY';
```

After Claude Code writes the code, open `index.html`, find the line `const API_KEY = 'REPLACE_WITH_YOUR_KEY';` and replace `REPLACE_WITH_YOUR_KEY` with your actual OMDB key. Save the file. Search for "Inception" and confirm real results appear.

---

### Step 5: Using Cursor's Chat to understand the API response

Before building the detail modal, you need to know exactly what data OMDB returns for a single movie. This is a good use of Cursor's Chat feature — paste in a real API response and ask it to explain the structure.

1. Open your browser and go to this URL (replace `YOUR_KEY` with your key):
   `https://www.omdbapi.com/?apikey=YOUR_KEY&i=tt1375666`
   This fetches the full record for Inception.
2. Copy the entire JSON response from the browser.
3. Open Cursor. Press `Ctrl+L` to open Chat.
4. Paste this prompt:

```
Here is a JSON response from the OMDB API for a single movie. List every field and what it contains, and tell me which fields are most useful for a movie detail view (title, plot, cast, director, ratings, runtime, genre):

[PASTE THE JSON HERE]
```

5. Cursor's Chat will explain the structure. Note which fields you want to display — you will use this in the next prompt.

Key fields to use: `Title`, `Year`, `Rated`, `Released`, `Runtime`, `Genre`, `Director`, `Actors`, `Plot`, `Poster`, `imdbRating`, `Ratings` (array of sources).

---

### Step 6: Add the movie detail modal

In Claude Code:

```
Add a movie detail modal to the Movie Explorer.

Context:
- OMDB single movie endpoint: https://www.omdbapi.com/?apikey=${API_KEY}&i=IMDB_ID
- Each card in the results grid already has the imdbID value from the search results
- The single movie response includes: Title, Year, Rated, Released, Runtime, Genre, Director, Actors, Plot, Poster, imdbRating

Requirements:
- Make each movie card clickable. When clicked, fetch the full movie details using the imdbID.
- Show a modal overlay (dark semi-transparent background) with:
  - Left column: the movie poster
  - Right column: Title (large), Year, Rated, Runtime, Genre, imdbRating (shown as a star rating display), Director, Actors, Plot
  - A close button (X) in the top-right corner of the modal
  - Clicking outside the modal (on the dark overlay) also closes it
- Show a loading spinner inside the modal while fetching
- The modal should be responsive — stack into a single column on mobile

Do not change any existing functionality.
```

Test by searching for a movie and clicking a card. Confirm the modal opens with real data.

---

### Step 7: Add favourites (heart button on each card)

In Claude Code:

```
Add a favourites system to the Movie Explorer.

Requirements:
- Add a heart icon button to each movie card (bottom-right corner, overlaid on the poster)
- When clicked, the movie is saved to localStorage under the key "movieExplorerFavourites" as an array of objects: { imdbID, Title, Year, Poster }
- If the movie is already a favourite, clicking the heart removes it
- The heart button should visually toggle: filled red heart when saved, outline heart when not saved
- On page load, read localStorage and mark any displayed cards that are already favourites with the filled heart
- The heart button click should NOT open the detail modal — stop event propagation

Do not change any existing functionality.
```

Test: search for a movie, click the heart on a card, refresh the page, search again — the heart should still be filled.

---

### Step 8: Add the favourites section

In Claude Code:

```
Add a Favourites view to the Movie Explorer.

Requirements:
- The "Favourites" button in the header already exists. Make it toggle a favourites panel.
- When clicked, hide the search section and results grid, and show a Favourites section instead
- The Favourites section shows all saved movies as cards (same card design as the results grid)
- Each card in the Favourites section has a "Remove" button that removes it from localStorage and re-renders the list
- If there are no favourites, show the message: "No favourites yet. Search for a movie and click the heart to save it."
- Add a "Back to Search" button at the top of the Favourites section that returns to the main search view
- When the user returns to search, their previous search results should still be visible (do not clear them)

Do not change any existing functionality.
```

Test the full flow: search, save a favourite, open Favourites, remove it, go back to search.

---

### Step 9: Deploy to GitHub Pages

1. Create a new GitHub repository. Go to `https://github.com/new`, name it `movie-explorer`, set it to Public, and click **Create repository**.

2. In your terminal (in the `movie-explorer` folder), run:

```bash
git init
git add index.html
git commit -m "Movie Explorer — initial build"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/movie-explorer.git
git push -u origin main
```

Replace `YOUR_GITHUB_USERNAME` with your actual GitHub username.

3. In the GitHub repository page, click **Settings** → **Pages** (left sidebar).
4. Under **Source**, select **Deploy from a branch**, choose **main**, folder **/ (root)**, and click **Save**.
5. Wait 60–90 seconds, then refresh the Settings → Pages page. Your live URL will appear: `https://YOUR_GITHUB_USERNAME.github.io/movie-explorer/`

6. Open the URL in your browser and test the live app.

---

## Practical workflow

```bash
# Create project folder
mkdir movie-explorer && cd movie-explorer

# Open Claude Code
claude

# After building, initialise git
git init
git add index.html
git commit -m "Movie Explorer — initial build"
git branch -M main
git remote add origin https://github.com/USERNAME/movie-explorer.git
git push -u origin main
```

GitHub Pages: Settings → Pages → Source: main branch → / (root) → Save → wait 90 seconds → live URL appears.

OMDB key: get from `https://www.omdbapi.com/apikey.aspx` → FREE tier → check email → paste into `const API_KEY = '...'`.

Cursor Chat for API inspection: `Ctrl+L` → paste JSON → ask for field explanation.

---

## Common mistakes

**Mistake 1: The API key is in the prompt but not in the actual file.**
You paste the prompt with `REPLACE_WITH_YOUR_KEY` as a placeholder and forget to swap it in the code. Every API call returns a 401 error and no results appear.
Fix: After every prompt that involves the API key, open `index.html`, search for `REPLACE_WITH_YOUR_KEY`, and replace it with your actual key before testing.

**Mistake 2: The heart button opens the modal instead of toggling the favourite.**
The card's click handler fires before the heart button's click handler, so clicking the heart also triggers the modal open.
Fix: In Claude Code, run this prompt:

```
The heart button on each movie card is triggering the modal open when clicked. Fix this by calling event.stopPropagation() inside the heart button's click handler so the click does not bubble up to the card's click handler.
```

**Mistake 3: Favourites disappear after a page refresh even though localStorage was implemented.**
The code saves favourites to localStorage but does not read them on page load to restore the filled-heart state on the currently displayed cards.
Fix: In Claude Code, run:

```
On page load, read the movieExplorerFavourites array from localStorage. After any search renders new cards, check each card's imdbID against the saved favourites array and set the heart to filled if there is a match. This check should run every time new cards are rendered, not just on initial load.
```

---

## Your turn

Action: Complete all five build steps (scaffold → search → modal → favourites → favourites view) and deploy to GitHub Pages.

Expected output: A live URL in the format `https://YOUR_USERNAME.github.io/movie-explorer/` where searching for "The Matrix" returns real movie cards with posters, clicking a card opens a modal with the plot and cast, and saving a favourite persists after a page refresh.

Failure state: GitHub Pages shows a 404 error after you set it up. This almost always means the repository is set to Private or the branch name does not match. Check: repository must be Public, and the branch selected in Settings → Pages must match the branch you pushed to (usually `main`). If you used `master` instead, change the branch in the Pages settings to `master`.

---

## Prompt / Template / Checklist pack

### Full 5-prompt build sequence

**Prompt 1 — Scaffold**
```
Build an HTML/CSS/JS single-page app called Movie Explorer.

Requirements:
- Single index.html file with all styles and scripts inline
- A header with the app name "Movie Explorer"
- A search section with a text input (placeholder "Search for a movie...") and a Search button
- A results section below with a grid layout — 4 columns on desktop, 2 on tablet, 1 on mobile
- Each grid cell will be a movie card (leave the grid empty for now — just the container)
- A "Favourites" button in the header (top right) that does nothing yet
- Colour scheme: dark background (#0f0f0f), white text, accent colour #e50914
- The search input and button should sit on the same row, centred on the page

Do not add any JavaScript functionality yet. This is just the visual scaffold.
```

**Prompt 2 — Search functionality**
```
Add search functionality to the Movie Explorer.

Context:
- OMDB API endpoint: https://www.omdbapi.com/?apikey=${API_KEY}&s=SEARCH_TERM&type=movie
- The API returns: { Search: [ { Title, Year, Poster, imdbID }, ... ] } on success
- On failure: { Response: "False", Error: "Movie not found!" }

Requirements:
- Add a constant at the top of the script: const API_KEY = 'REPLACE_WITH_YOUR_KEY';
- When the user clicks Search or presses Enter, fetch from OMDB using the search term
- Display results as cards: poster image, title, year
- If Poster is "N/A", show a grey placeholder div with the title centred
- Show a loading state while fetching
- Show an error message if no results or if fetch fails
- Cards have a hover effect (scale up slightly, border appears)

Do not change the existing HTML structure.
```

**Prompt 3 — Movie detail modal**
```
Add a movie detail modal to the Movie Explorer.

Context:
- OMDB single movie endpoint: https://www.omdbapi.com/?apikey=${API_KEY}&i=IMDB_ID
- Each card already has the imdbID from search results

Requirements:
- Make each movie card clickable. On click, fetch full details using the imdbID.
- Show a modal overlay with:
  - Left column: poster image
  - Right column: Title, Year, Rated, Runtime, Genre, imdbRating (star display), Director, Actors, Plot
  - X close button top-right
  - Clicking the dark overlay also closes the modal
- Show a loading spinner inside the modal while fetching
- Responsive: stack to single column on mobile

Do not change any existing functionality.
```

**Prompt 4 — Favourites (heart button)**
```
Add a favourites system to the Movie Explorer.

Requirements:
- Add a heart icon button to each movie card (bottom-right, overlaid on poster)
- Clicking saves the movie to localStorage key "movieExplorerFavourites" as array of: { imdbID, Title, Year, Poster }
- If already a favourite, clicking removes it
- Heart toggles visually: filled red when saved, outline when not
- On page load, read localStorage and mark any currently-displayed cards that are already favourites
- Heart button click must NOT open the detail modal (use event.stopPropagation())

Do not change any existing functionality.
```

**Prompt 5 — Favourites view**
```
Add a Favourites view to the Movie Explorer.

Requirements:
- The existing "Favourites" header button should toggle a favourites panel
- When toggled: hide search section and results grid, show Favourites section
- Favourites section: same card grid as results, shows all saved movies
- Each card has a "Remove" button that removes from localStorage and re-renders
- Empty state message: "No favourites yet. Search for a movie and click the heart to save it."
- "Back to Search" button at top of Favourites section
- Returning to search: previous results remain visible

Do not change any existing functionality.
```

---

### API-Powered App Template (reusable scaffold for any search-based API app)

Use this template any time you are building a new app that searches a third-party API and displays results.

```
Build a single-page HTML/CSS/JS app called [APP NAME].

Data source: [API ENDPOINT URL]
API key variable: const API_KEY = 'REPLACE_WITH_YOUR_KEY';
Search parameter: [HOW THE API ACCEPTS A SEARCH TERM — e.g., ?s=TERM or ?q=TERM]
Results field in response: [FIELD NAME — e.g., response.Search or response.results]

Scaffold requirements:
- Single index.html file, all styles and scripts inline
- Header: app name + optional navigation button (top right)
- Search section: text input + Search button, same row, centred
- Results grid: [N] columns desktop, [N] columns tablet, 1 column mobile
- Colour scheme: [BACKGROUND] background, [TEXT] text, [ACCENT] accent colour

Each result card displays:
- [FIELD 1 — e.g., image/poster]
- [FIELD 2 — e.g., title]
- [FIELD 3 — e.g., subtitle/year]

Interaction:
- Clicking a card opens a detail modal fetching from [DETAIL ENDPOINT] using [ID FIELD]
- Modal shows: [LIST OF FIELDS TO DISPLAY]
- Modal closes on X click or overlay click

State:
- Loading state while fetching
- Error state if no results or fetch fails
- [Optional: localStorage persistence for saved items]

Do not add functionality not listed above.
```

---

### Day 14 completion checklist

- [ ] OMDB API key received and placed in `const API_KEY`
- [ ] Search returns real movie results with posters
- [ ] Poster fallback works (grey div when Poster is "N/A")
- [ ] Clicking a card opens a modal with full details
- [ ] Modal closes on X and on overlay click
- [ ] Heart button toggles and saves to localStorage
- [ ] Favourites persist after page refresh
- [ ] Favourites view shows saved movies
- [ ] Remove button works in Favourites view
- [ ] Back to Search returns to previous results
- [ ] App is responsive (test on mobile or use browser dev tools)
- [ ] Deployed to GitHub Pages
- [ ] Live URL tested in browser
