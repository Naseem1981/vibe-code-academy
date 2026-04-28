# The 5-Minute Claude Code Setup Checklist
*Lead Magnet — Vibe Code from Zero*

Everything you need to go from zero to your first Claude Code prompt. Do these 7 steps in order.

---

## Step 1 — Install VS Code (2 minutes)

Go to `https://code.visualstudio.com`. Click Download. Run the installer with all defaults.

**Check:** VS Code opens. You see a dark editor with a sidebar on the left.

---

## Step 2 — Install Node.js (2 minutes)

Go to `https://nodejs.org`. Click **Download Node.js (LTS)**. Run installer with all defaults.

Open a terminal (Windows: press Win+R → type `cmd` → Enter. Mac: Cmd+Space → type Terminal → Enter).

Run: `node --version`

**Check:** you see `v18.x.x` or higher (not an error).

---

## Step 3 — Install Claude Code CLI (1 minute)

In the same terminal, run:
```
npm install -g @anthropic-ai/claude-code
```
Wait for it to finish (30–90 seconds).

**Check:** run `claude --version` → you see a version number like `claude 1.x.x`.

---

## Step 4 — Log in to your Claude account (1 minute)

Run:
```
claude login
```

Your browser opens. Sign in with your Claude account. Return to terminal.

**Check:** terminal shows `✓ Logged in as your@email.com`

**Note:** Claude Code requires Pro ($20/month) or Max ($100/month). It does not work on the free plan.

---

## Step 5 — Install the VS Code Extension (1 minute)

Open VS Code. Press Ctrl+Shift+X (Windows/Linux) or Cmd+Shift+X (Mac) to open Extensions.

Search for: `Claude Code`

Find the extension published by **Anthropic**. Click Install.

**Check:** a Claude icon appears in the left sidebar.

---

## Step 6 — Connect the extension (1 minute)

Click the Claude icon in the VS Code sidebar. Click **Sign in with Claude**. Use the same account as Step 4.

**Check:** you see a chat interface. Type `hello` → Claude responds.

---

## Step 7 — Run your first command (30 seconds)

In VS Code, press Ctrl+` (backtick) to open the terminal. Navigate into a project folder:
```
mkdir my-first-app
cd my-first-app
claude
```

**Check:** you see a `>` prompt. Claude is ready.

---

## You're set up.

Type your first prompt at the `>` prompt. The full *Vibe Code from Zero* guide shows you exactly what to write.

---

*If any step shows an error, the most common fix is: re-run the Node.js installer from nodejs.org and make sure "Add to PATH" is checked (Windows) or complete the installation fully (Mac). Then re-run Step 3.*
