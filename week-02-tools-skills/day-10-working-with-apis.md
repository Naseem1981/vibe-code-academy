# Day 10: Working with APIs

## Outcome artifact

By the end of this day you will have a working Country Info app — a user types a country name, clicks Search, and sees the country's flag, capital city, population, and currency fetched live from the REST Countries API.

---

## The core idea

An API — Application Programming Interface — is a service that gives you data when you ask for it. Think of it like a waiter in a restaurant. You tell the waiter what you want (your request), the waiter goes to the kitchen (the server), and comes back with what you asked for (the response). You do not see or control the kitchen. You just make requests and receive data. Almost every real-world app uses APIs: a weather app fetches temperature from a weather API, a flight booking site fetches prices from an airline API, a currency converter fetches exchange rates from a finance API.

The good news: many APIs are free, need no account, and no API key. You just make a request to a web address (called an endpoint) and get data back in a format called JSON. JSON is just structured text — it looks like a list of labels and values. You do not need to understand how to write JSON. You just need to know it exists and that Claude Code knows how to read it and pull out the pieces you want.

The mechanism your app uses to talk to an API is a JavaScript function called `fetch()`. You tell `fetch()` the API address, it sends the request, waits for the response, and hands the data to your code. Claude Code writes all of this for you. Your job is to know how to prompt it correctly: give it the API endpoint URL, tell it which fields from the response you want to display, and tell it where on the page to show them. You also need to handle two edge cases: loading state (the moment between the user clicking Search and the data arriving — show a "Loading..." message) and error state (what happens if the user types a country that does not exist — show a clear error message instead of a broken screen).

---

## Step-by-step walkthrough

### Step 1 — Open Claude Code

Open your terminal. Navigate to your project folder. Start Claude Code.

```
claude
```

### Step 2 — Create the project file

You will build this as a new standalone HTML file — not your landing page. This keeps it clean.

```
Create a new file called country-lookup.html in my project folder.

It should be a single self-contained HTML file with all styles inline. No external frameworks or libraries except Tailwind CSS via CDN: <script src="https://cdn.tailwindcss.com"></script>

The page should have:
- A clean, centered layout, max 600px wide, centered on the page.
- A heading: "Country Info"
- A text input where the user types a country name, with placeholder text "Enter a country name..."
- A Search button next to the input.
- A results area below the input, initially empty, where data will be displayed once fetched.
- A neutral background color, not white. Pick something tasteful.

Do not add any JavaScript or API functionality yet — just the layout and styles.
```

### Step 3 — Understand the API endpoint

The REST Countries API returns information about any country in the world. The endpoint URL you will use is:

```
https://restcountries.com/v3.1/name/{countryname}
```

Replace `{countryname}` with any country name. For example:

```
https://restcountries.com/v3.1/name/france
```

Open that URL in your browser right now. You will see a wall of JSON text. That is the raw data the API sends back. You do not need to read it all. The fields you want are:

- `flags.svg` — the URL of the country's flag image
- `capital[0]` — the capital city (it is an array, first item)
- `population` — the population number
- `currencies` — an object where each key is a currency code and the value contains the currency name

Claude Code knows how to extract all of these. You just need to tell it which ones you want.

### Step 4 — Add the fetch() functionality

Now you add the live data fetching.

```
Add JavaScript to country-lookup.html that fetches country data from the REST Countries API when the user clicks the Search button.

API endpoint: https://restcountries.com/v3.1/name/{countryname}
Replace {countryname} with whatever the user typed in the input field, URL-encoded.

Steps the code should follow:
1. When Search is clicked, read the value from the text input.
2. If the input is empty, show an error message in the results area: "Please enter a country name." Do not make an API request.
3. Clear any previous results from the results area.
4. Show a loading message in the results area: "Loading..." with a subtle animated spinner or pulsing text effect.
5. Fetch from the API using: fetch('https://restcountries.com/v3.1/name/' + encodeURIComponent(inputValue))
6. When the response arrives, check if it is successful (response.ok is true). If not, show: "Country not found. Check the spelling and try again."
7. If successful, parse the JSON and extract from the first result in the array:
   - flags.svg (flag image URL)
   - name.common (country name)
   - capital[0] (capital city)
   - population (population number)
   - currencies (first currency name — get the first key of the currencies object, then read .name from its value)
8. Display the results in the results area with this layout:
   - Flag image: 120px wide, left-aligned, with a subtle border and border-radius.
   - Country name as a heading below the flag.
   - Three labeled rows: "Capital:", "Population:", "Currency:" — each label bold, value normal weight.
   - Format the population number with commas (e.g. 67,000,000 not 67000000).
9. If fetch() itself throws an error (network failure), show: "Network error. Check your connection and try again."

Also add this usability feature: allow the user to press Enter in the input field to trigger the search, same as clicking the button.
```

### Step 5 — Test the app in your browser

Open `country-lookup.html` in your browser. Do these tests:

1. Type "France" and click Search. You should see the French flag, "Paris", the population, and "Euro".
2. Type "Japan" and press Enter. You should see Japan's results.
3. Clear the input and click Search without typing anything. You should see: "Please enter a country name."
4. Type "zzzzz" and click Search. You should see: "Country not found. Check the spelling and try again."
5. While the results are showing, type a new country and search again. The previous results should disappear before the new ones appear.

### Step 6 — Fix any issues found in testing

If any test in Step 5 produced the wrong output, report it to Claude Code with the exact symptom.

```
When I search for [country name], the result shows [describe what you see]. It should show [describe what you expect]. Fix this.
```

For example:

```
When I search for "Japan", the currency shows as "undefined" instead of the currency name. Fix the currency extraction — it should show the name of the first currency in the currencies object.
```

### Step 7 — Polish the results display

Once the data is displaying correctly, ask for a final style pass.

```
Polish the results display in country-lookup.html.

1. Add a smooth fade-in animation when results appear — animate opacity from 0 to 1 over 300ms.
2. Add a subtle card style to the results area: white background, rounded corners (8px), padding (24px), a soft box-shadow.
3. Format the flag image with a 1px solid light border and 4px border-radius.
4. Make the country name heading 24px, bold.
5. Add 8px vertical spacing between each data row (capital, population, currency).
6. Make the Search button show a visual disabled state and change its text to "Searching..." while the fetch is in progress. Restore it when done.

Do not change the layout or the data being fetched.
```

---

## Practical workflow

1. Open terminal, run `claude`.
2. Prompt to create `country-lookup.html` — layout only, no JS.
3. Note the API endpoint: `https://restcountries.com/v3.1/name/{countryname}`.
4. Prompt to add `fetch()` — name the endpoint, list the four fields, describe all three states: loading, success, error.
5. Open in browser, run five tests: valid country, Enter key, empty input, invalid name, second search clears first.
6. Fix any failures with specific symptom-first prompts.
7. Prompt for polish: fade-in, card style, disabled button during fetch.

---

## Common mistakes

**Mistake 1: Not specifying the exact fields you want from the API response**

Prompt: "Fetch country data and show it on the page."

Claude Code does not know which of the dozens of available fields you want. It may guess and include fields that are formatted oddly or irrelevant.

Fix: Always name every field explicitly in your prompt. Use the exact JSON path — for example "flags.svg", "capital[0]", "population", "currencies". Test the API endpoint in your browser first so you can see the field names.

---

**Mistake 2: Forgetting to handle the loading and error states**

A prompt that only describes the success case will produce code that leaves the page blank while loading and crashes or shows broken output when the API returns an error.

Fix: Every API prompt must describe three states: (1) while waiting — show a loading message, (2) on success — display the data, (3) on error — show a human-readable message. Include all three in your prompt explicitly.

---

**Mistake 3: Testing the app by opening the HTML file directly with file:///

Some browsers block `fetch()` requests when the page is loaded from the filesystem with a `file:///` URL due to security restrictions. The page opens but the Search button does nothing — no results, no error message.

Fix: Always serve the file through a local server. Run `node serve.mjs` in your project root first, then open `http://localhost:3000/country-lookup.html` in your browser. If `serve.mjs` is not in your project, ask Claude Code to create a minimal one.

```
Create a file called serve.mjs in my project root that starts a simple static HTTP server on port 3000 using Node.js built-in modules only — no npm packages. It should serve all files in the current directory.
```

---

## Your turn

Build the Country Info app using the prompts in Steps 2, 3, and 4 above. Then run all five tests from Step 5.

Expected output: Searching for "Germany" returns a card with the German flag, capital "Berlin", population formatted with commas, and currency "Euro". The loading state appears briefly before the card. Searching again for another country replaces the previous card cleanly.

Failure state: The fetch works but the currency field shows "undefined". This means the code is trying to access the currency name using the wrong path. Fix it with:

```
The currency field in my Country Info app is showing "undefined". The currencies field in the REST Countries API response is an object where each key is a currency code (like "EUR") and the value is an object with a "name" property. Fix the currency extraction to: get the first key of the currencies object, then read currencies[firstKey].name.
```

---

## Prompt / Template / Checklist pack

### API Starter Prompt Pack

Each prompt below builds a complete, self-contained API-powered feature. Paste the prompt into Claude Code and name the file to create.

---

**Prompt 1 — Country Lookup (REST Countries API)**

```
Create a file called country-lookup.html. Single self-contained HTML file, all styles inline, Tailwind CSS via CDN.

Build a Country Info lookup app:
- Text input: "Enter a country name..." placeholder.
- Search button. Also trigger search on Enter key.
- Results area showing: flag image (120px wide), country name as heading, capital city, population (formatted with commas), and currency name.

API endpoint: https://restcountries.com/v3.1/name/{countryname}
Fields to extract: flags.svg, name.common, capital[0], population, first currency name from currencies object (currencies[Object.keys(currencies)[0]].name).

Three states:
1. Loading: show "Searching..." text, disable Search button, change button label to "Searching..."
2. Success: show results in a card with white background, 8px border-radius, 24px padding, soft shadow.
3. Error (response.ok is false): show "Country not found. Check the spelling and try again."
4. Network error (fetch throws): show "Network error. Check your connection."
5. Empty input: show "Please enter a country name." without making a request.

Fade in results with a 300ms opacity transition.
```

---

**Prompt 2 — Random Advice Generator (Advice Slip API)**

```
Create a file called advice-generator.html. Single self-contained HTML file, all styles inline, Tailwind CSS via CDN.

Build a Random Advice generator:
- A centered card layout, max 480px wide.
- A large quote-style display area showing the advice text.
- A button labeled "Get Advice".
- Clicking the button fetches a new random piece of advice.

API endpoint: https://api.adviceslip.com/advice
Field to extract: slip.advice (the advice text string).

Three states:
1. Loading: show "Fetching wisdom..." in the advice display area.
2. Success: show the advice text in the display area with a fade-in animation (300ms).
3. Error: show "Could not load advice. Try again."

On first load, fetch advice automatically — do not wait for the user to click.
Style the advice text at 22px, centered, italic, with generous line height (1.6). Style the card with a soft gradient background.
```

---

**Prompt 3 — Dictionary Word Lookup (Free Dictionary API)**

```
Create a file called dictionary.html. Single self-contained HTML file, all styles inline, Tailwind CSS via CDN.

Build a Dictionary app:
- Text input: "Look up a word..." placeholder.
- Search button. Also trigger on Enter key.
- Results area showing: the word as a large heading, its phonetic pronunciation if available, and up to 3 definitions from the first meaning entry.

API endpoint: https://api.dictionaryapi.dev/api/v2/entries/en/{word}

Fields to extract from the first item in the response array:
- word (the word itself)
- phonetic (phonetic string, may be absent — show nothing if missing)
- meanings[0].partOfSpeech (e.g. "noun", "verb")
- meanings[0].definitions[0].definition through definitions[2].definition (up to 3 definitions, skip if fewer exist)

Three states:
1. Loading: show "Looking up..." in the results area.
2. Success: show word, phonetic, part of speech label, and numbered definitions (1, 2, 3).
3. Error (404 or response.ok false): show "Word not found. Try a different spelling."
4. Empty input: show "Please enter a word."

Style each definition with a left border (3px, accent color) and light background. Number them 1, 2, 3.
```

---

**Prompt 4 — Random Joke Generator (Official Joke API)**

```
Create a file called joke-generator.html. Single self-contained HTML file, all styles inline, Tailwind CSS via CDN.

Build a Random Joke generator:
- A centered card, max 560px wide.
- A "setup" display area showing the joke setup text.
- A "Tell me the punchline" button.
- When clicked, reveal the punchline below the setup with a slide-down animation.
- A "Next Joke" button that fetches a new joke and resets the punchline to hidden.

API endpoint: https://official-joke-api.appspot.com/random_joke
Fields to extract: setup (the joke setup), punchline (the punchline).

Flow:
1. On page load, fetch one joke automatically. Show the setup. Hide the punchline.
2. "Tell me the punchline" button: reveal punchline with a 400ms slide-down animation.
3. "Next Joke" button: fetch a new joke, show new setup, hide punchline again.
4. While fetching: show "Loading joke..." in the setup area, disable both buttons.
5. On fetch error: show "Could not load a joke. Try again."

Style: playful but clean. Setup text at 20px. Punchline at 18px, bold, different color from setup.
```

---

**Prompt 5 — Book Search (Open Library API)**

```
Create a file called book-search.html. Single self-contained HTML file, all styles inline, Tailwind CSS via CDN.

Build a Book Search app:
- Text input: "Search for a book title or author..." placeholder.
- Search button. Also trigger on Enter key.
- Results area showing up to 5 book results in a vertical list.

API endpoint: https://openlibrary.org/search.json?q={query}&limit=5
URL-encode the query with encodeURIComponent().

Fields to extract from each item in response.docs array (up to 5 items):
- title
- author_name[0] (first author, may be absent — show "Unknown author" if missing)
- first_publish_year (may be absent — show nothing if missing)
- cover_i (cover image ID — if present, build URL as: https://covers.openlibrary.org/b/id/{cover_i}-M.jpg. If absent, show a placeholder: https://placehold.co/60x80?text=No+Cover)

Display each result as a horizontal card: cover image on the left (60px wide, 80px tall, object-fit cover), title and author stacked on the right, year below author in a lighter color.

Three states:
1. Loading: show "Searching books..." in the results area.
2. Success: show up to 5 book cards. If docs array is empty, show "No books found for that search."
3. Error: show "Search failed. Check your connection and try again."
4. Empty input: show "Please enter a title or author name."

Add a 200ms stagger animation — each book card fades in 200ms after the previous one.
```

---

### Free API Quick Reference

APIs that require no account and no API key — ready to use immediately.

| API | What it returns | Endpoint pattern |
|-----|----------------|-----------------|
| REST Countries | Country data: flag, capital, population, currency | `https://restcountries.com/v3.1/name/{name}` |
| Advice Slip | Random advice quotes | `https://api.adviceslip.com/advice` |
| Free Dictionary | Word definitions, phonetics, parts of speech | `https://api.dictionaryapi.dev/api/v2/entries/en/{word}` |
| Official Joke API | Random setup/punchline jokes | `https://official-joke-api.appspot.com/random_joke` |
| Open Library | Book search by title or author | `https://openlibrary.org/search.json?q={query}&limit=5` |
| Cat Facts | Random cat facts | `https://catfact.ninja/fact` |
| Dog CEO | Random dog images | `https://dog.ceo/api/breeds/image/random` |
| Numbers API | Facts about numbers and dates | `http://numbersapi.com/{number}` |

### Where to find more free APIs

- **Public APIs list:** `https://github.com/public-apis/public-apis` — searchable directory, filter by "No" in the Auth column to find key-free APIs.
- **RapidAPI free tier:** `https://rapidapi.com` — thousands of APIs, many with free tiers, requires account.
- **Any API:** `https://any-api.com` — another browsable directory.

### API Prompt Formula

When you write your own API prompts to Claude Code, always include all five of these elements:

```
1. The exact endpoint URL with the variable part clearly marked.
2. The exact JSON field names you want extracted (test the URL in your browser first to see the fields).
3. Loading state: what to show while waiting.
4. Success state: exactly how to display each field.
5. Error state: what to show if the API returns an error or the fetch fails.
```

Missing any one of these produces incomplete code. Missing the error state produces an app that breaks silently when something goes wrong.
