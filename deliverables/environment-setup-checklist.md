# Environment Setup Checklist
*Module 0 Deliverable — Vibe Code from Zero*

---

## One-time setup (do this once, never again)

**VS Code**
- [ ] VS Code downloaded from code.visualstudio.com
- [ ] VS Code installed and opens correctly

**Node.js**
- [ ] Node.js LTS downloaded from nodejs.org
- [ ] Node.js installed with all defaults
- [ ] Verified: `node --version` shows v18 or higher

**Claude Code CLI**
- [ ] Installed: `npm install -g @anthropic-ai/claude-code` completed without errors
- [ ] Verified: `claude --version` shows a version number (not an error)
- [ ] Logged in: `claude login` completed — terminal shows ✓ Logged in as your email

**Claude account**
- [ ] Account is Pro ($20/month) or Max ($100/month) — Claude Code does not work on the free plan

**VS Code Extensions**
- [ ] Claude Code extension installed (publisher: Anthropic — not a third-party extension)
- [ ] Claude Code extension connected to account — chat interface visible in left sidebar
- [ ] GitLens extension installed (publisher: GitKraken)
- [ ] Material Icon Theme installed (publisher: Philipp Kief)

---

## Before every build session (do this every time)

- [ ] VS Code open
- [ ] Terminal open in VS Code (`` Ctrl+` `` on Windows/Linux, `` Cmd+` `` on Mac)
- [ ] Navigated to project folder: `cd your-project-folder`
- [ ] Claude Code working: `claude --version` shows version number
- [ ] CLAUDE.md exists in project root (after Module 3)

---

## Quick verification commands

| Check | Command | Expected output |
|-------|---------|-----------------|
| Node.js version | `node --version` | v18.x.x or higher |
| Claude Code version | `claude --version` | claude 1.x.x |
| Claude login status | `claude --version` | no "not logged in" error |
| Plugins installed | `claude mcp list` | lists your installed plugins |

---

## Troubleshooting

**`node` is not recognized / command not found**
Node.js did not add itself to your PATH. Re-run the Node.js installer from nodejs.org. On Windows, make sure you check "Add to PATH" during installation.

**`claude` is not recognized / command not found**
Either Node.js is not installed (run `node --version` to check) or the npm install did not complete. Re-run `npm install -g @anthropic-ai/claude-code`.

**`claude login` shows "billing required" error**
Your Claude account is on the free plan. Claude Code requires Pro ($20/month) or Max ($100/month). Go to claude.ai → Settings → Billing to upgrade.

**VS Code Extension shows "Not connected"**
Click the Claude icon in the sidebar → click Sign in with Claude → use the same account as `claude login`.
