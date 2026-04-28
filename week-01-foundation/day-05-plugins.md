# Module 4: Supercharge with claude.ai/plugins

## Outcome artifact
By the end of this module you will have 3 plugins installed and active in your Claude environment — GitHub, Filesystem, and Web Search — with Claude successfully using at least one during a real project task, confirmed by running `claude mcp list` and seeing all three listed.

<!-- ASSET: diagram | Three plugin cards side by side: GitHub (git cat icon), Filesystem (folder icon), Web Search (magnifying glass icon), each with a checkmark and "Active" label below -->

## The core idea

Claude Code out of the box can read and write files inside your project folder and run shell commands. That is already powerful. Plugins extend what Claude can reach — GitHub repositories, folders outside your project, the live web, databases, calendars, and hundreds of other data sources.

Plugins use a standard called MCP — Model Context Protocol. The name sounds technical, but the idea is simple: MCP is a bridge. It lets Claude talk to external tools using the same protocol, regardless of what the tool is. A GitHub plugin and a Web Search plugin look completely different under the hood, but Claude interacts with both through the same interface. You install the bridge once, and Claude gains access to the tool.

The marketplace for these bridges is `https://claude.ai/plugins`. This is the official Anthropic-hosted directory of plugins you can add to your Claude environment. Each listing shows what the plugin does, how to install it, and what it allows Claude to do. You browse it in your browser and install plugins by running a terminal command it gives you.

Three plugins are the right starting point for every beginner:

**GitHub** — lets Claude read your repositories, create branches, commit code directly to GitHub, open pull requests, and check what has changed. Useful once you want to share your projects or collaborate.

**Filesystem** — gives Claude access to folders outside your current project directory. By default, Claude Code can only see files in the folder where you ran `claude`. The Filesystem plugin removes that boundary for folders you explicitly grant access to.

**Web Search** — lets Claude search the web and use the results inside its responses. When you are building something that needs up-to-date information (current documentation, API specs, real pricing), Claude can search for it instead of guessing from its training data.

Once installed, plugins show up as tools Claude can call. Claude decides when to use them based on what you ask. You do not need to trigger them manually — they are just available, and Claude uses them when relevant.

## Step-by-step walkthrough

### Open the plugins marketplace

**1.** Open your browser and go to:
```
https://claude.ai/plugins
```

You will see the Claude plugins marketplace — a grid of available plugins. Each card shows the plugin name, a short description, and an "Install" or "Add" button. Browse for a minute to see what's available. You will find plugins for Notion, Slack, GitHub, file systems, web search, databases, and more.

For this module, you are installing three specific plugins. The steps below walk through each one.

### Install the GitHub plugin

**2.** In the plugins marketplace, search for **GitHub** or find it in the list.

**3.** Click on the GitHub plugin card. You will see a page with:
- What the plugin does
- The MCP server install command (a terminal command starting with `claude mcp add`)
- Any authentication steps required

**4.** Copy the install command shown on the page. It will look like this:
```
claude mcp add github --env GITHUB_TOKEN=your_token_here
```

**5.** Before running this, you need a GitHub personal access token. If you do not have a GitHub account: go to `https://github.com`, click **Sign up**, create a free account.

With your account created:
- Click your profile photo (top-right) → **Settings**
- Scroll down in the left sidebar and click **Developer settings**
- Click **Personal access tokens** → **Tokens (classic)**
- Click **Generate new token (classic)**
- Give it a name: "Claude Code"
- Check these boxes under **Select scopes**: `repo` (all), `read:org`
- Click **Generate token**
- Copy the token immediately — you will only see it once

**6.** Back in your terminal, run the install command with your actual token replacing `your_token_here`:
```
claude mcp add github --env GITHUB_TOKEN=ghp_your_actual_token_here
```

You will see output confirming the plugin was added.

### Install the Filesystem plugin

**7.** In the plugins marketplace, find and click the **Filesystem** plugin.

**8.** Copy the install command. It will look like:
```
claude mcp add filesystem /path/to/folder
```

**9.** Decide which folder you want to give Claude access to. A good choice for beginners is your Desktop or your main projects folder:
- **Windows:** `C:\Users\YourName\Desktop`
- **Mac:** `~/Desktop`

Run the command with your chosen path:
```
claude mcp add filesystem C:\Users\YourName\Desktop
```
(Windows example — replace `YourName` with your actual Windows username)

Or on Mac:
```
claude mcp add filesystem ~/Desktop
```

Claude can now read and write files in that folder, not just in the current project directory.

### Install the Web Search plugin

**10.** In the plugins marketplace, find and click the **Web Search** plugin (sometimes listed as "Brave Search" or "Tavily Search" depending on the provider).

**11.** Copy the install command. You may need an API key from the search provider — the plugin page will explain this. For Tavily (one of the most common options):
- Go to `https://tavily.com`
- Click **Get API Key** (free tier is available)
- Copy your API key

**12.** Run the install command with your API key:
```
claude mcp add tavily --env TAVILY_API_KEY=tvly_your_key_here
```

### Verify all three plugins are installed

**13.** In your terminal, run:
```
claude mcp list
```

Expected output:
```
MCP Servers:
  github       (connected)
  filesystem   (connected)
  tavily       (connected)
```

All three should show `(connected)`. If any shows `(disconnected)` or an error, re-run its install command and check that the API key or token is correct.

### Use a plugin in a real task

**14.** Start Claude Code in your project folder:
```
claude
```

**15.** Test the Web Search plugin with a real task:
```
Search the web for the current Netlify free tier limits — specifically how many sites and what storage/bandwidth is included. I want to know before I deploy my app.
```

Claude will use the Web Search tool to look this up and give you a current answer — not a guess from its training data. You will see Claude indicate it is searching (something like `Searching for: Netlify free tier limits...`), then return results.

**16.** Test the Filesystem plugin by asking Claude to list files in the folder you granted access to:
```
List the files on my Desktop.
```

Claude will use the Filesystem tool and return the contents of your Desktop folder.

<!-- ASSET: diagram | Screenshot-style illustration showing Claude terminal with "Searching for: Netlify free tier limits" then a structured answer with current data — a clear "Claude used a plugin" moment -->

## Practical workflow

1. Open browser → go to `https://claude.ai/plugins`
2. Browse or search for the plugin you want → click it → read what it does and what credentials it needs
3. Copy the install command from the plugin page
4. If the plugin needs a token/API key: get one from the linked service (GitHub, Tavily, etc.)
5. Run: `claude mcp add [plugin-name] [--env KEY=value]` in terminal
6. Verify: run `claude mcp list` — all installed plugins should show `(connected)`
7. Start `claude` in your project → test the plugin by asking a task that would use it
8. To remove a plugin: `claude mcp remove [plugin-name]`

## Common mistakes

**Mistake 1: Running `claude mcp add` while inside a Claude Code session.**
The `claude mcp` commands are terminal commands, not Claude prompts. You run them directly in your terminal (outside of a `claude` session). If you are inside a `claude` session (you see the `>` prompt), press `Ctrl+C` to exit first, then run the mcp command, then start `claude` again.

**Mistake 2: Copying the GitHub token but then closing the page before using it.**
GitHub personal access tokens are shown once. If you navigate away, you cannot retrieve the token — only create a new one. Fix: keep the token tab open until you have run `claude mcp add github --env GITHUB_TOKEN=...` and verified the plugin is connected.

**Mistake 3: Expecting Claude to automatically use plugins without relevant tasks.**
Plugins are tools Claude uses when the task calls for them. If you ask Claude to "fix the button colour" it will not search the web — that task doesn't need web access. If you ask "what is the latest version of this JavaScript API?" Claude will search. The trigger is the task, not a manual command. If Claude is not using a plugin you expected it to use, your prompt may not be clear that external data is needed — add "search the web for" or "check the filesystem at" to make the need explicit.

## Your turn

Run `claude mcp list` in your terminal. You should see three connected plugins:

```
MCP Servers:
  github       (connected)
  filesystem   (connected)
  [web search] (connected)
```

Then start a Claude session and use the Web Search plugin on a real task relevant to your project. A good test prompt: **"Search the web for the best way to add a simple contact form to a plain HTML page without a backend. Show me the top options."**

You should see Claude search and return an answer based on current web results — not just its training knowledge.

If `claude mcp list` shows errors or disconnected plugins, re-run the install command for that plugin and check the API key or token again.

## Prompt / Template / Checklist pack

### Plugin Setup Checklist

Run through this every time you install a new plugin.

**GitHub plugin:**
- [ ] GitHub account created (github.com)
- [ ] Personal access token generated (Settings → Developer settings → Personal access tokens)
- [ ] Token scopes set: `repo` (all), `read:org`
- [ ] Token copied before closing the page
- [ ] `claude mcp add github --env GITHUB_TOKEN=your_token` run in terminal (outside Claude session)
- [ ] `claude mcp list` shows `github (connected)`

**Filesystem plugin:**
- [ ] Target folder chosen (e.g. Desktop, main projects folder)
- [ ] `claude mcp add filesystem /path/to/folder` run in terminal
- [ ] `claude mcp list` shows `filesystem (connected)`
- [ ] Tested: asked Claude to list files in the folder — correct files returned

**Web Search plugin (Tavily):**
- [ ] Tavily account created (tavily.com) — free tier available
- [ ] API key copied from Tavily dashboard
- [ ] `claude mcp add tavily --env TAVILY_API_KEY=your_key` run in terminal
- [ ] `claude mcp list` shows `tavily (connected)`
- [ ] Tested: asked Claude to search for something — Claude used the search tool and returned current results

---

### Top 10 Plugins for Beginners

Each entry shows: **Plugin name** — what it does — install command template — best use case.

**1. GitHub** — read repos, create commits, open pull requests
```
claude mcp add github --env GITHUB_TOKEN=your_token
```
Best for: pushing your project to GitHub, checking what changed, creating backup branches.

**2. Filesystem** — access folders outside your project
```
claude mcp add filesystem /path/to/folder
```
Best for: when you want Claude to reference files from other projects or your Documents folder.

**3. Tavily Web Search** — search the web from inside Claude
```
claude mcp add tavily --env TAVILY_API_KEY=your_key
```
Best for: getting current documentation, checking API limits, researching how to build a feature.

**4. Notion** — read and write Notion pages and databases
```
claude mcp add notion --env NOTION_TOKEN=your_token
```
Best for: if you plan your projects in Notion and want Claude to reference your notes.

**5. Brave Search** — alternative web search using Brave's index
```
claude mcp add brave-search --env BRAVE_API_KEY=your_key
```
Best for: web search with less tracking. Alternative to Tavily.

**6. Puppeteer** — control a browser (screenshot, click, fill forms)
```
claude mcp add puppeteer
```
Best for: testing your web app automatically, taking screenshots of your app programmatically.

**7. SQLite** — read and write SQLite database files
```
claude mcp add sqlite --db-path /path/to/database.db
```
Best for: when your app stores data in a SQLite file and you want Claude to query or update it.

**8. Memory** — persistent storage across Claude sessions
```
claude mcp add memory
```
Best for: storing project decisions, preferences, or notes that Claude remembers across sessions — in addition to CLAUDE.md.

**9. Slack** — post messages, read channels
```
claude mcp add slack --env SLACK_BOT_TOKEN=your_token
```
Best for: if your team uses Slack and you want Claude to post status updates or read channel history.

**10. Google Drive** — read Google Docs and Sheets
```
claude mcp add google-drive --env CREDENTIALS_PATH=/path/to/credentials.json
```
Best for: when your project references a Google Doc spec or a Google Sheet of data.

**Note:** Install commands shown above are templates. Check `https://claude.ai/plugins` for the exact current command for each plugin, as commands may update.
