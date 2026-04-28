# Plugin Power Stack — Quick Reference

All 10 Claude Code plugins you need active for the Level 2 build. Install once, use every session.

## Plugin Table

| Plugin | Install Command | What it does | When to use it |
|--------|----------------|--------------|----------------|
| **Claude Code** | built-in | Core AI coding assistant — reads, writes, and edits files with context | Every session, always active |
| **MCP Filesystem** | `claude mcp add filesystem` | Gives Claude direct read/write access to your local file system | Any time Claude needs to read or write files outside the editor |
| **MCP GitHub** | `claude mcp add github` | Connects Claude to GitHub — read PRs, issues, and commit history | When reviewing code, creating PRs, or checking repo status |
| **MCP Supabase** | `claude mcp add supabase` | Connects Claude to your Supabase project — query tables, check schema | When building or debugging database queries and RLS policies |
| **MCP Postgres** | `claude mcp add postgres` | Direct PostgreSQL access for running raw SQL in context | When you need raw SQL execution or schema inspection |
| **MCP Playwright** | `claude mcp add playwright` | Browser automation — Claude can open pages, click, fill forms, screenshot | Day 19 testing and any time you need automated browser checks |
| **MCP Puppeteer** | `claude mcp add puppeteer` | Screenshot and PDF capture from a running browser | Taking screenshots of your app during development |
| **MCP Fetch** | `claude mcp add fetch` | Lets Claude fetch web pages and API responses directly | Debugging API integrations and reading documentation |
| **MCP Memory** | `claude mcp add memory` | Persistent project memory — Claude remembers context across sessions | Store project decisions, schema notes, and naming conventions |
| **MCP Sequential Thinking** | `claude mcp add sequentialthinking` | Step-by-step reasoning mode for complex multi-part problems | Planning architecture, debugging complex bugs, writing tests |

---

## Copy-Paste Install Block

Run all 10 install commands in your terminal at once:

```bash
claude mcp add filesystem
claude mcp add github
claude mcp add supabase
claude mcp add postgres
claude mcp add playwright
claude mcp add puppeteer
claude mcp add fetch
claude mcp add memory
claude mcp add sequentialthinking
claude mcp add context7
```

> **Note:** After installing, restart Claude Code and verify each plugin shows a green status dot in the MCP panel (`/mcp` in the Claude Code sidebar). If a plugin shows red, check the setup docs for that specific plugin — most require an API key or connection string set in your environment.

---

## Quick Activation Check

At the start of every session, type `/mcp` in Claude Code to see the status of all installed plugins. All 10 should show as connected before you start building.
