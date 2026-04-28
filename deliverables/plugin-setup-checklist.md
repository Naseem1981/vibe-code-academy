# Plugin Setup Checklist + Top 10 Plugins for Beginners
*Module 4 Deliverable — Vibe Code from Zero*

---

## Plugin Setup Checklist

### GitHub Plugin
- [ ] GitHub account created (github.com)
- [ ] Personal access token generated: Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token
- [ ] Token scopes checked: `repo` (all), `read:org`
- [ ] Token copied before closing the page (shown once only)
- [ ] Install command run in terminal (outside Claude session):
  ```
  claude mcp add github --env GITHUB_TOKEN=your_token_here
  ```
- [ ] Verified: `claude mcp list` shows `github (connected)`

### Filesystem Plugin
- [ ] Target folder chosen (e.g. Desktop or main projects folder)
- [ ] Install command run:
  ```
  claude mcp add filesystem /path/to/your/folder
  ```
  Windows example: `claude mcp add filesystem C:\Users\YourName\Desktop`
  Mac example: `claude mcp add filesystem ~/Desktop`
- [ ] Verified: `claude mcp list` shows `filesystem (connected)`
- [ ] Tested: asked Claude to list files in the folder — correct files returned

### Web Search Plugin (Tavily)
- [ ] Tavily account created (tavily.com) — free tier available
- [ ] API key copied from Tavily dashboard
- [ ] Install command run:
  ```
  claude mcp add tavily --env TAVILY_API_KEY=your_key_here
  ```
- [ ] Verified: `claude mcp list` shows `tavily (connected)`
- [ ] Tested: asked Claude to search for something — search tool was used and current results returned

---

## Top 10 Plugins for Beginners

Each entry shows: plugin name, what it does, install command template, and best use case.

---

### 1. GitHub
Read repos, create commits, open pull requests directly from Claude.
```
claude mcp add github --env GITHUB_TOKEN=your_token
```
**Best for:** pushing your project to GitHub, checking what changed between versions, creating backup branches before risky changes.

---

### 2. Filesystem
Give Claude access to folders outside your current project.
```
claude mcp add filesystem /path/to/folder
```
**Best for:** referencing files from other projects, accessing a shared documents folder, letting Claude see your Desktop.

---

### 3. Tavily Web Search
Search the live web from inside Claude Code.
```
claude mcp add tavily --env TAVILY_API_KEY=your_key
```
**Best for:** getting current documentation, checking API limits or pricing, researching how to implement a feature you haven't tried before.

---

### 4. Brave Search
Alternative web search using Brave's index.
```
claude mcp add brave-search --env BRAVE_API_KEY=your_key
```
**Best for:** web search as an alternative to Tavily. Get a free API key at search.brave.com/app/keys.

---

### 5. Puppeteer
Control a real browser — navigate, click, fill forms, take screenshots.
```
claude mcp add puppeteer
```
**Best for:** testing your web app automatically (click through it without using your hands), taking programmatic screenshots of your app at different sizes.

---

### 6. Memory
Persistent storage that Claude remembers across sessions — beyond CLAUDE.md.
```
claude mcp add memory
```
**Best for:** storing project decisions, ideas you want to revisit, or preferences that apply across multiple projects (not just one).

---

### 7. Notion
Read and write Notion pages and databases.
```
claude mcp add notion --env NOTION_TOKEN=your_token
```
**Best for:** if you plan your projects in Notion and want Claude to reference your notes or update a project tracker. Get your token at notion.so/my-integrations.

---

### 8. SQLite
Read and write a local SQLite database file.
```
claude mcp add sqlite --db-path /path/to/your/database.db
```
**Best for:** apps that store data in a local database file. Claude can query, update, and inspect the database directly.

---

### 9. Slack
Post messages and read channel history.
```
claude mcp add slack --env SLACK_BOT_TOKEN=your_token
```
**Best for:** teams using Slack. Claude can post status updates, read a specific channel for context, or check recent discussions.

---

### 10. Google Drive
Read Google Docs and Sheets.
```
claude mcp add google-drive --env CREDENTIALS_PATH=/path/to/credentials.json
```
**Best for:** projects with a spec in Google Docs, or apps that work with spreadsheet data.

---

## Plugin Management Commands

| Action | Command |
|--------|---------|
| List all installed plugins | `claude mcp list` |
| Remove a plugin | `claude mcp remove [plugin-name]` |
| Add a plugin | `claude mcp add [plugin-name] [options]` |

**Note:** All `claude mcp` commands must be run in the terminal **outside** a Claude session. Exit any running `claude` session (Ctrl+C) before running mcp commands.

---

## Where to Find More Plugins

Browse the full marketplace: `https://claude.ai/plugins`

Each plugin page shows: what it does, what credentials it needs, and the exact install command. Install any plugin the same way you installed the three above.
