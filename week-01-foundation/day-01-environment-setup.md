# Module 0: Your Launchpad — Set Up Your Complete Claude Environment

## Outcome artifact
By the end of this module you will have VS Code open and configured, Claude Code CLI installed and authenticated, and the claude.ai VS Code extension connected to your account — confirmed by running `claude --version` in the terminal and seeing a version number, not an error.

<!-- ASSET: diagram | Three boxes in a row: VS Code (editor) → Claude Code CLI (terminal brain) → claude.ai Extension (editor brain), with arrows showing they work together -->

## The core idea

You need three things installed before you can vibe code with Claude. Most tutorials tell you to just install one tool — that's why most beginners get stuck halfway through their first project.

**Claude Code CLI** is the command-line version of Claude. You run it from your terminal (the black window where you type commands). It can read your files, write code, create folders, and run commands on your behalf. Think of it as Claude with hands — it doesn't just give you code, it actually makes the changes.

**VS Code** is where your project lives. It's a free code editor made by Microsoft. Even though you're not writing code yourself, you still need VS Code to see what Claude builds, review changes, and manage your files. The claude.ai VS Code extension turns VS Code into a chat interface — you can talk to Claude right inside your editor, not just in the terminal.

**The claude.ai VS Code extension** and the CLI are different tools that serve different purposes. The CLI (terminal) is better for big tasks: "build me a full login system." The extension (inside VS Code) is better for focused tasks: "fix this one function" or "explain what this file does." You'll use both. They share your account, so you only log in once.

Here's the exact setup order that works: install VS Code first, then Node.js (which Claude Code needs to run), then Claude Code CLI, then the extension. Installing in any other order causes errors.

## Step-by-step walkthrough

### Install VS Code

**1.** Open your browser and go to `https://code.visualstudio.com`. Click the large blue **Download** button. It auto-detects your operating system.

**2.** Run the installer. On Windows: double-click the `.exe` file and accept all defaults. On Mac: drag VS Code to your Applications folder. On Linux: follow the `.deb` or `.rpm` instructions shown on the download page.

**3.** Open VS Code. You'll see a welcome screen. Close any popups. You should see a dark editor with a sidebar on the left.

### Install Node.js

Claude Code CLI requires Node.js version 18 or higher. Node.js is a runtime that lets JavaScript programs run on your computer — Claude Code is built on it.

**4.** Go to `https://nodejs.org`. Click **Download Node.js (LTS)**. LTS means Long Term Support — it's the stable version. Download and run the installer with all defaults.

**5.** Verify Node.js installed correctly. Open a terminal:
- **Windows:** Press `Windows key + R`, type `cmd`, press Enter. Or open VS Code and press `` Ctrl+` `` (backtick) to open the built-in terminal.
- **Mac:** Press `Cmd+Space`, type `Terminal`, press Enter.
- **Linux:** Press `Ctrl+Alt+T`.

Type this command and press Enter:
```
node --version
```

Expected output (your version number may differ):
```
v22.14.0
```

If you see `v18` or higher, you're good. If you see `'node' is not recognized` (Windows) or `command not found` (Mac/Linux), the installer didn't complete correctly — re-download and run it again, making sure to check "Add to PATH" on Windows.

### Install Claude Code CLI

**6.** In the same terminal, type this command exactly and press Enter:
```
npm install -g @anthropic-ai/claude-code
```

This downloads and installs Claude Code. You'll see a lot of text scrolling — that's normal. Wait for it to finish (30–90 seconds). You'll know it's done when you see a `>` or `$` prompt again.

**7.** Verify Claude Code installed:
```
claude --version
```

Expected output:
```
claude 1.x.x
```

**8.** Log in to your Claude account:
```
claude login
```

This opens your browser and shows a Claude login page. Sign in with your Claude account (the one with Pro or Max access). After logging in, return to the terminal — you'll see:
```
✓ Logged in as your@email.com
```

You're authenticated. Claude Code can now access your account.

### Install the claude.ai VS Code Extension

**9.** Open VS Code. On the left sidebar, click the **Extensions icon** (it looks like four squares, one slightly separated from the others). Or press `Ctrl+Shift+X` (Windows/Linux) or `Cmd+Shift+X` (Mac).

**10.** In the search bar at the top of the Extensions panel, type:
```
Claude Code
```

**11.** Find the extension published by **Anthropic** (not third-party extensions with similar names). Click **Install**.

**12.** After installation, a Claude icon appears in your left sidebar. Click it. A panel opens on the left side of VS Code asking you to sign in. Click **Sign in with Claude** and use the same account you used in Step 8.

**13.** Once connected, you'll see a chat interface inside VS Code. Type `hello` and press Enter. Claude should respond in the panel.

### Install Recommended VS Code Extensions

**14.** While you have the Extensions panel open, also install:
- **GitLens** (by GitKraken) — shows you what changed in your files and when
- **Material Icon Theme** (by Philipp Kief) — makes your file sidebar easier to read with icons

Search for each by name and click Install.

### Final verification

**15.** In the VS Code terminal (press `` Ctrl+` `` to open it), run:
```
claude --version
```

Expected output: a version number like `claude 1.x.x`.

You now have your complete Claude environment. All three tools are installed, connected, and ready.

<!-- ASSET: diagram | Screenshot-style illustration showing VS Code with the Claude extension panel open on the left, terminal at the bottom showing claude --version output -->

## Practical workflow

1. Open VS Code
2. Press `` Ctrl+` `` to open the terminal
3. Navigate to your project folder: `cd path/to/your/project`
4. Run `claude` to start a Claude Code session
5. For focused in-editor help: click the Claude icon in the left sidebar
6. To check login status: `claude --version`
7. To re-login if needed: `claude login`

## Common mistakes

**Mistake 1: Installing Claude Code CLI without Node.js first.**
Fix: Run `node --version` before `npm install -g @anthropic-ai/claude-code`. If you get an error, install Node.js from nodejs.org first, then retry the Claude Code install.

**Mistake 2: Installing a third-party "Claude" extension instead of the official Anthropic one.**
Fix: In the Extensions panel, check that the publisher says "Anthropic" before installing. Delete any other Claude extensions you accidentally installed (right-click → Uninstall).

**Mistake 3: Logging in on the free plan and getting an error when running Claude Code.**
Fix: Claude Code requires Pro ($20/month) or Max ($100/month). Go to claude.ai → Settings → Billing and upgrade before continuing.

## Your turn

Run this in your terminal:
```
claude --version
```

You should see a version number like `claude 1.x.x`. Then open VS Code, click the Claude icon in the left sidebar, and confirm you see the chat interface.

If you see `command not found` instead of a version number, check that Node.js installed correctly (`node --version`) and re-run `npm install -g @anthropic-ai/claude-code`.

Do not move to Module 1 until both checks pass: terminal shows a version number, VS Code shows the chat interface.

## Prompt / Template / Checklist pack

### Environment Setup Checklist

Use this before starting every new project session. Tick every box.

**One-time setup (do this once, never again):**
- [ ] VS Code installed and opens correctly
- [ ] Node.js v18+ installed (`node --version` shows v18 or higher)
- [ ] Claude Code CLI installed (`claude --version` shows a version number)
- [ ] Claude account logged in (`claude login` completed successfully)
- [ ] claude.ai VS Code extension installed (publisher: Anthropic)
- [ ] Extension connected to account (chat interface visible in VS Code sidebar)
- [ ] GitLens extension installed
- [ ] Material Icon Theme installed

**Each session (do this every time you start building):**
- [ ] VS Code open
- [ ] Terminal open in VS Code (`` Ctrl+` ``)
- [ ] Navigate to project folder (`cd your-project-folder`)
- [ ] Confirm Claude Code works: `claude --version`
- [ ] CLAUDE.md exists in project root (after Module 3)
