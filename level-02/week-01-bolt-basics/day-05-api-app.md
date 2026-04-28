# Day 5: Connect to a Live API — Build a Weather App

## Outcome artifact
By the end of this day you will have a working Weather app in Bolt.new that accepts a city name, fetches live weather data from the Open-Meteo API (no API key required), and displays the current temperature, wind speed, and weather condition with an emoji icon.

---

## The core idea

Every real application fetches data from somewhere outside itself. A CRM pulls customer records from a database. An e-commerce site checks live stock levels. A weather app gets atmospheric readings from a sensor network. The mechanism that connects your app to that outside data is an API — an Application Programming Interface. An API is a URL you can request, and the server at that URL responds with data, usually in JSON format.

Today's app uses two APIs from Open-Meteo, both completely free with no account or API key required. The first API converts a city name into geographic coordinates (latitude and longitude). The second API takes those coordinates and returns the current weather. You need both because the weather API does not accept city names — it only accepts coordinates. This two-step pattern is called **geocoding** and it appears in almost every app that works with geographic locations: maps, delivery trackers, store finders, and event discovery apps all use it.

Understanding the JSON response from an API is a core skill. When your app fetches from a URL, the server returns a text string formatted as JSON. Your code parses that string into a JavaScript object, and then you access the data you need using dot notation — `data.current.temperature_2m`, for example. Knowing where in the response the data lives requires reading the API's response structure. Both response structures are documented in full in this module so you do not need to look anything up.

---

## The APIs you will use

### Geocoding API — convert city name to coordinates

**URL pattern:**
```
https://geocoding-api.open-meteo.com/v1/search?name=CITY&count=1
```

Replace `CITY` with the city name, e.g. `Cape+Town` or `London`.

**Example request:**
```
https://geocoding-api.open-meteo.com/v1/search?name=Cape+Town&count=1
```

**Response JSON structure:**
```json
{
  "results": [
    {
      "id": 3369157,
      "name": "Cape Town",
      "latitude": -33.9258,
      "longitude": 18.4232,
      "elevation": 1.0,
      "feature_code": "PPLA",
      "country_code": "ZA",
      "admin1_id": 1028907,
      "timezone": "Africa/Johannesburg",
      "country": "South Africa",
      "admin1": "Western Cape"
    }
  ],
  "generationtime_ms": 0.8
}
```

The data you need: `results[0].latitude` and `results[0].longitude`.

If the city is not found, the `results` array will be empty (`[]`) or the `results` key will be absent entirely. Your app must handle this case and show an error message.

---

### Forecast API — get current weather

**URL pattern:**
```
https://api.open-meteo.com/v1/forecast?latitude=LAT&longitude=LON&current=temperature_2m,wind_speed_10m,weather_code
```

Replace `LAT` and `LON` with the coordinates from the geocoding step.

**Example request:**
```
https://api.open-meteo.com/v1/forecast?latitude=-33.9258&longitude=18.4232&current=temperature_2m,wind_speed_10m,weather_code
```

**Response JSON structure:**
```json
{
  "latitude": -33.9258,
  "longitude": 18.4232,
  "generationtime_ms": 0.06,
  "utc_offset_seconds": 7200,
  "timezone": "Africa/Johannesburg",
  "timezone_abbreviation": "SAST",
  "elevation": 10.0,
  "current_units": {
    "time": "iso8601",
    "interval": "seconds",
    "temperature_2m": "°C",
    "wind_speed_10m": "km/h",
    "weather_code": "wmo code"
  },
  "current": {
    "time": "2026-04-28T14:00",
    "interval": 900,
    "temperature_2m": 19.3,
    "wind_speed_10m": 14.8,
    "weather_code": 2
  }
}
```

The data you need: `current.temperature_2m`, `current.wind_speed_10m`, `current.weather_code`.

---

### Weather code meanings

The `weather_code` field is a WMO (World Meteorological Organisation) code. Map it to a human-readable label and an emoji using these rules:

| Code range | Condition | Emoji |
|---|---|---|
| 0 | Clear sky | ☀️ |
| 1, 2, 3 | Partly cloudy | ⛅ |
| 45, 48 | Foggy | 🌫️ |
| 51, 53, 55, 61, 63, 65, 66, 67 | Rain | 🌧️ |
| 71, 73, 75, 77 | Snow | ❄️ |
| 80, 81, 82 | Rain showers | 🌦️ |
| 95 | Thunderstorm | ⛈️ |

For any code not in this list, use "Unknown conditions" and ❓.

---

## Step-by-step walkthrough

### Step 1 — Start a new Bolt.new project

1. Go to `https://bolt.new`.
2. Click inside the prompt box to start a fresh project.
3. Do not continue a session from previous days.

---

### Step 2 — Prompt 1: App shell with search input and empty results card

Paste this prompt exactly:

```
Build a Weather app using React and Tailwind CSS.

Do not fetch any data yet — just build the UI shell.

Components needed:

1. SEARCH BAR
- A text input with placeholder "Enter a city name..."
- A "Search" button next to the input
- Pressing Enter in the input triggers the same action as clicking Search
- Store the input value in useState

2. RESULTS CARD (empty state)
- A card below the search bar
- When no search has been made yet, show a centred message: "Search for a city to see the weather"
- When a search is in progress, show a loading spinner (an animated spinning circle — use CSS animation, not an image)
- When data is loaded, the card will display: city name as a heading, a large temperature display, wind speed, and a weather condition with an emoji (leave these as placeholder text for now)
- When an error occurs (city not found), show: "City not found. Try a different name."

State variables needed (all useState):
- cityInput: string — the text in the search input
- loading: boolean — whether a fetch is in progress
- error: string | null — an error message, or null
- weather: object | null — the fetched weather data, or null

The card should change what it shows based on these state values:
- loading is true → show the spinner
- error is not null → show the error message
- weather is not null → show the weather data (placeholder text for now)
- all three are default (loading false, error null, weather null) → show the "Search for a city" message

Layout:
- Centred on the page, max width 480px
- Page background: a gradient from #0f172a to #1e3a5f (dark navy to deep blue)
- Card background: white with rounded-2xl and shadow-xl
- 24px padding inside the card
```

After generation, confirm:
- The search bar renders with an input and button.
- The card shows the "Search for a city" message.
- No fetch happens yet (clicking Search does nothing visible yet — that is correct).

---

### Step 3 — Prompt 2: Wire up the two-step API fetch

```
Add the fetch logic to the weather app.

When the user clicks "Search" or presses Enter:

STEP 1 — Geocoding fetch:
- Set loading to true, clear any previous error and weather data
- Fetch from: https://geocoding-api.open-meteo.com/v1/search?name=ENCODED_CITY&count=1
  Replace ENCODED_CITY with encodeURIComponent(cityInput.trim())
- If the response's "results" array is empty or missing, set error to "City not found. Try a different name." and set loading to false. Stop here.
- Extract latitude and longitude from results[0]

STEP 2 — Weather fetch (only runs if Step 1 succeeded):
- Fetch from: https://api.open-meteo.com/v1/forecast?latitude=LAT&longitude=LON&current=temperature_2m,wind_speed_10m,weather_code
  Replace LAT and LON with the values from Step 1
- Extract from the response: current.temperature_2m, current.wind_speed_10m, current.weather_code
- Also store the city name from the geocoding result (results[0].name) to display in the card heading
- Set weather to an object: { cityName, temperature, windSpeed, weatherCode }
- Set loading to false

Display in the results card when weather is not null:
- City name as a large heading (text-3xl, font-bold)
- Temperature: display as a very large number (text-6xl, font-bold) followed by "°C" in text-2xl
- Wind speed: "💨 Wind: X km/h" in text-lg
- Weather code: display the human-readable condition label and emoji (use a helper function to map the code)

Weather code mapping function (write this as a separate function in the component file):
function getWeatherInfo(code) {
  if (code === 0) return { label: 'Clear sky', emoji: '☀️' };
  if (code <= 3) return { label: 'Partly cloudy', emoji: '⛅' };
  if (code <= 48) return { label: 'Foggy', emoji: '🌫️' };
  if (code <= 67) return { label: 'Rainy', emoji: '🌧️' };
  if (code <= 77) return { label: 'Snow', emoji: '❄️' };
  if (code <= 82) return { label: 'Rain showers', emoji: '🌦️' };
  if (code === 95) return { label: 'Thunderstorm', emoji: '⛈️' };
  return { label: 'Unknown conditions', emoji: '❓' };
}

Wrap both fetches in a try/catch block. If any fetch throws (network error), set error to "Something went wrong. Check your connection and try again."
```

After generation, test with real cities:
- Type "Cape Town" and press Search. Wait 1-2 seconds. Confirm temperature and wind speed appear.
- Type "London" and press Search. Confirm different values appear.
- Type "zzzznotacity" and press Search. Confirm the "City not found" error message appears.

---

### Step 4 — Understand the two-step fetch pattern

The reason the app makes two API calls instead of one is that the weather API only accepts coordinates, not city names. Coordinates are the universal language of location — every GPS system, map service, and weather station uses latitude/longitude. City names are ambiguous (there are multiple cities named "Springfield" around the world), so geocoding converts a name to a precise point on Earth first.

This two-step pattern is called a **dependent fetch** or **chained fetch**: the second request depends on the result of the first. You will see this pattern throughout the CRM project — for example, fetching an event's ticket types after first loading the event details. Understanding why the chain exists (data dependency) makes it easier to debug when one step fails.

---

### Step 5 — Prompt 3: Weather condition icons using emoji

The weather code mapping is already built into Prompt 2's output, but Prompt 3 makes the condition display more prominent:

```
Make the weather condition display more prominent in the results card.

Changes to the results card layout (only when weather data is loaded):

Current layout from top to bottom:
1. City name heading
2. Temperature
3. Wind speed
4. Condition label

New layout:
1. City name heading (keep as is)
2. A large emoji display — take the emoji from getWeatherInfo(weather.weatherCode).emoji and display it at 80px font size (use style={{ fontSize: '80px' }} on a div, text-center)
3. The condition label text below the emoji — text-xl, font-medium, text-center, text-gray-600
4. A horizontal divider line (a simple <hr> with 16px top and bottom margin)
5. Two stats side by side in a row:
   Left: 🌡️ Temperature — display "19.3°C" (round to 1 decimal place using .toFixed(1))
   Right: 💨 Wind — display "14.8 km/h" (round to 1 decimal place)
   Each stat has a small label above it ("Temperature", "Wind Speed") in text-xs text-gray-400
   Each stat has the value below in text-2xl font-bold text-gray-800

The two stats should be in a flex row with justify-around.

Do not change the search bar, loading state, error state, or empty state. Only change the weather data display section of the card.
```

After generation, search for a city again. Confirm:
- The large emoji appears in the centre of the card.
- The condition label is below the emoji.
- Temperature and wind speed are displayed side by side with labels.

---

### Step 6 — Prompt 4: Loading state and error state polish

```
Polish the loading and error states.

LOADING STATE:
- Replace any basic spinner with this specific animation:
  A circle div (width 48px, height 48px) with a border (4px solid #e2e8f0) and the top border coloured #3b82f6 (blue)
  Apply a CSS spin animation: @keyframes spin { to { transform: rotate(360deg) } } with animation duration 0.8s linear infinite
  Centre it in the card vertically and horizontally
  Below the spinner, show the text "Fetching weather for [cityInput]..." in text-sm text-gray-500
  Replace [cityInput] with the actual city name the user typed

ERROR STATE:
- Style the error message as a red alert box inside the card:
  Red background: #fee2e2
  Red border: 1px solid #fca5a5
  Border-radius: 8px
  Padding: 12px 16px
  Text colour: #991b1b
  Icon before the text: ⚠️
  Below the error message, show a "Try again" link (styled as a button) that clears the error state and focuses the search input

EMPTY STATE:
- Style the "Search for a city to see the weather" message:
  Show a large 🌍 emoji at 64px font size centred in the card
  Below it: the message text in text-gray-400, text-center
  Below that: three example city names as clickable chips — "Cape Town", "Tokyo", "New York"
  Each chip is a small button with light grey background (#f1f5f9), rounded-full, px-3 py-1, text-sm
  Clicking a chip sets cityInput to that city name and immediately triggers the search

Do not change the weather data display or the search bar.
```

After generation:
1. Click the "Cape Town" chip in the empty state. Confirm it searches automatically.
2. Search for an invalid city. Confirm the red error box appears with the "Try again" link.
3. Click "Try again". Confirm the error clears and the input is focused.
4. Search for a real city. While loading, confirm the spinner appears with "Fetching weather for X..." text.

---

## Practical workflow

1. Open bolt.new, start a new project.
2. Paste Prompt 1 — build the UI shell with state variables.
3. Confirm: input renders, card shows empty state, no fetch yet.
4. Paste Prompt 2 — wire up the two-step geocoding + weather fetch.
5. Test: search "Cape Town" → live temperature appears. Search "zzz" → error appears.
6. Paste Prompt 3 — make condition emoji prominent, side-by-side stats.
7. Test: confirm emoji and layout.
8. Paste Prompt 4 — loading spinner, error box, city chips.
9. Test: click a chip → auto-search. Invalid city → red error. Valid city → spinner then results.
10. Download the project.

---

## Common mistakes

**Mistake 1 — Forgetting to encode the city name in the URL, causing fetch errors for cities with spaces.**

City names like "Cape Town" or "New York" contain spaces. URLs cannot contain spaces. You must encode spaces as `%20` (or `+`) before putting the city name in a URL. The correct way is `encodeURIComponent(cityInput.trim())`. If you see a network error or "City not found" for cities with spaces but not for single-word cities, the city name is not being encoded. Write a prompt: `"The city name in the geocoding URL is not being encoded. Replace the raw cityInput variable in the URL with encodeURIComponent(cityInput.trim()) to handle spaces and special characters correctly."`

**Mistake 2 — Not handling the case where the geocoding API returns an empty results array.**

When a city is not found, the Open-Meteo geocoding API returns `{ "generationtime_ms": 0.3 }` with no `results` key at all, or returns `{ "results": [] }` with an empty array. If your code does `results[0].latitude` without checking whether `results` exists and has at least one item, it will throw `TypeError: Cannot read properties of undefined`. Always check: `if (!data.results || data.results.length === 0)` before accessing `results[0]`. If Bolt generates code that crashes on invalid cities, write a prompt to add this guard.

**Mistake 3 — Making the fetch run on every keystroke instead of only on button click or Enter.**

If the fetch runs inside a `useEffect` with `cityInput` as a dependency, it will fire an API call for every character the user types. This hammers the API with hundreds of requests per session. The fetch should only run inside the Search button's `onClick` handler and the input's `onKeyDown` handler (when `event.key === 'Enter'`). Never put a fetch inside a `useEffect` that watches the search input. If you see network requests firing as you type, write a prompt: `"Move the fetch logic out of any useEffect. The fetch should only trigger when the user clicks the Search button or presses Enter in the input. Remove any useEffect that watches cityInput and triggers a fetch."`

---

## Your turn

Open bolt.new now. Run all four prompts. After Prompt 4, test the app with five real cities: one two-word city (e.g. "Cape Town"), one city with an accent (e.g. "São Paulo" — type it as "Sao Paulo"), one single-word city (e.g. "Tokyo"), one invalid entry (e.g. "zzzzxxx"), and click one of the city chips in the empty state.

**Expected output:** A weather app with a dark blue gradient background, white card, live weather data from the Open-Meteo API for valid cities, a red error box for invalid cities, a spinner with city name during loading, and clickable city chips in the empty state.

**Failure state 1:** The app shows "City not found" for every city including real ones. Fix: open the browser's developer tools (F12 in Chrome), click the **Network** tab, run a search, and click the failed geocoding request. Check the URL it is trying to fetch — if the URL looks malformed (wrong quotes, spaces, extra characters), the URL is being constructed incorrectly. Write a prompt: `"Show me the exact URL being fetched in the geocoding step by logging it to the console with console.log before the fetch call. The URL must follow this pattern exactly: https://geocoding-api.open-meteo.com/v1/search?name=ENCODED_CITY&count=1"`

**Failure state 2:** The app crashes with `TypeError: Cannot read properties of undefined (reading 'temperature_2m')`. Fix: the code is trying to access `data.current.temperature_2m` but the forecast API response does not have a `current` field. This usually means the API URL is missing the `&current=temperature_2m,wind_speed_10m,weather_code` parameter. Without that parameter, Open-Meteo returns an empty `current` object. Write a prompt: `"The forecast API fetch URL is missing the current parameters. The URL must include &current=temperature_2m,wind_speed_10m,weather_code at the end. Check the constructed URL and add these parameters if they are missing."`

**Failure state 3:** The spinner never goes away even after data loads. Fix: the `loading` state is being set to `true` before the fetch but never set back to `false` after it completes. Write a prompt: `"The loading spinner never stops. Make sure loading is set to false in three places: after successfully setting weather data, after setting an error (city not found), and inside the catch block (network error). Every exit path from the fetch function must call setLoading(false)."`

---

## Prompt / Template / Checklist pack

The four build prompts appear in the walkthrough above (Steps 2, 3, 5, 6). Here are three bonus prompts.

---

### Bonus Prompt 1 — 7-day forecast

```
Add a 7-day forecast section below the current weather card.

API change:
- Update the forecast API URL to add daily forecast data:
  &daily=temperature_2m_max,temperature_2m_min,weather_code
  &timezone=auto
  The full URL becomes:
  https://api.open-meteo.com/v1/forecast?latitude=LAT&longitude=LON&current=temperature_2m,wind_speed_10m,weather_code&daily=temperature_2m_max,temperature_2m_min,weather_code&timezone=auto

- The response will now include a "daily" object with arrays:
  daily.time — array of 7 date strings in YYYY-MM-DD format
  daily.temperature_2m_max — array of 7 maximum temperatures
  daily.temperature_2m_min — array of 7 minimum temperatures
  daily.weather_code — array of 7 weather codes

- Store the daily data in the weather state object alongside the current data

7-day forecast display:
- Show below the current conditions card as a second card with heading "7-Day Forecast"
- Seven rows, one per day
- Each row has:
  - Day name on the left (format daily.time[i] as the weekday name using new Date(daily.time[i] + 'T12:00:00').toLocaleDateString('en-US', {weekday: 'short'}))
  - The weather emoji in the centre (use getWeatherInfo(daily.weather_code[i]).emoji)
  - Min temp on the right in blue (#3b82f6)
  - Max temp on the right in red (#ef4444)
  - Format: "12° / 21°" (round to nearest integer, no decimal)
- Alternate row background: even rows #f8fafc, odd rows white
- The first row (today) is highlighted with a slightly blue-tinted background (#eff6ff)

Only show the 7-day card when weather data is loaded. Hide it during loading and error states.
```

---

### Bonus Prompt 2 — Multiple saved cities

```
Add the ability to save multiple cities and switch between them.

New feature: Saved Cities

State:
- Add a savedCities array in useState, initialised from localStorage ('savedCities') or defaulting to []
- Each saved city is an object: { name: string, weather: object } where weather is the full weather data object

Save button:
- When weather data is displayed, show a "Save city" button (bookmark icon or "⭐ Save") below the results card
- Clicking it adds the current city to savedCities (if not already saved) and persists to localStorage
- If the city is already saved, the button shows "✅ Saved" and is disabled

Saved cities panel:
- Show a "Saved Cities" section below the search bar (above the results card)
- Only show this section if savedCities has at least one city
- Display each saved city as a row with:
  - City name on the left
  - Current temperature (from the saved weather data) in the centre
  - Weather emoji on the right
  - A remove button (×) on the far right that removes the city from the saved list
- Clicking a saved city row fetches fresh weather data for that city and displays it in the main results card
- Persist savedCities to localStorage on every change using useEffect

Note: the temperature in saved cities is the temperature at the time it was last fetched, not live. Show a small "(last fetched)" label below the temperature in muted text to make this clear.
```

---

### Bonus Prompt 3 — Temperature unit toggle

```
Add a Celsius / Fahrenheit toggle to the weather app.

Toggle:
- A toggle switch or two-button selector ("°C | °F") in the top-right corner of the main weather card
- Store the unit preference in useState: 'celsius' or 'fahrenheit', defaulting to 'celsius'
- Persist the preference to localStorage ('temperatureUnit') so it survives a refresh

Conversion:
- Do not re-fetch from the API when toggling — convert in the display layer
- Celsius to Fahrenheit formula: F = (C × 9/5) + 32
- Fahrenheit to Celsius formula: C = (F - 32) × 5/9
- Apply this conversion to all temperature values shown in the app:
  - Current temperature (the large display)
  - Min and max temperatures in the 7-day forecast (if that feature is present)

Display:
- Show the unit symbol next to every temperature value: "°C" or "°F"
- When the unit changes, animate the temperature number with a brief fade-out/fade-in transition (opacity 0 to 1, duration 200ms)

Toggle styling:
- Two buttons side by side: "°C" and "°F"
- Active unit: white background, dark text, slight shadow
- Inactive unit: transparent background, grey text
- Rounded-full, small size (px-3 py-1, text-sm)
- Wrapped in a grey pill container (#f1f5f9 background, rounded-full)
```
