# MCP Workshop — Prerequisites

> **Complete all steps below before the workshop session.**
> Estimated setup time: **15–20 minutes**

---

## ✅ Checklist Summary

| # | Requirement | Verify Command / Action |
|---|-------------|------------------------|
| 1 | Node.js 20+ | `node --version` |
| 2 | npm / npx | `npx --version` |
| 3 | GitHub Copilot CLI | `copilot --version` |
| 4 | GitHub Personal Access Token | Token saved |
| 5 | Atlassian Account + API Token | Token saved |
| 6 | MCP Servers Configured | `copilot` → MCP servers listed |

---

## 1. Install Node.js (v20 or later)

MCP servers run via `npx`, so Node.js is required.

- **Download:** https://nodejs.org/en/download
- Install the **LTS** version (must be v20+)

**Verify:**

```sh
node --version    # Expected: v20.x.x or higher
npm --version     # Expected: 10.x.x or higher
npx --version     # Expected: 10.x.x or higher
```

---

## 2. Install GitHub Copilot CLI

The Copilot CLI is our MCP client — it connects to MCP servers and lets you interact with them via natural language.

### Install via npm

```sh
npm install -g @githubnext/github-copilot-cli
```

### Or install via GitHub CLI extension

```sh
# Install GitHub CLI first (if not installed)
# macOS
brew install gh

# Then install Copilot extension
gh extension install github/gh-copilot
```

**Verify:**

```sh
copilot --version
# or
gh copilot --version
```

### Authenticate

```sh
gh auth login
```

Follow the prompts to authenticate with your GitHub account. Make sure your account has an **active GitHub Copilot subscription** (Individual, Business, or Enterprise).

---

## 3. Create a GitHub Personal Access Token (PAT)

The GitHub MCP Server needs a token to access your repositories, PRs, and issues.

### Steps

1. Go to: https://github.com/settings/tokens?type=beta
2. Click **"Generate new token"** (Fine-grained token)
3. Give it a name: `mcp-workshop`
4. Set expiration: **7 days** (or longer if needed)
5. Under **Repository access**, choose:
   - **"All repositories"** — for full access during the workshop
   - Or select specific repos you want to review
6. Under **Permissions**, grant:

   | Permission | Access Level |
   |------------|-------------|
   | **Contents** | Read |
   | **Issues** | Read & Write |
   | **Pull requests** | Read & Write |
   | **Actions** | Read |
   | **Metadata** | Read (auto-selected) |

7. Click **"Generate token"**
8. **Copy the token immediately** — you won't see it again!

### Save the token

Store it in your shell profile for the session:

```sh
export GITHUB_TOKEN="github_pat_xxxxxxxxxxxxxxxxxxxx"
```

> **💡 Tip:** Add it to your `~/.zshrc` or `~/.bashrc` so it persists across terminal sessions.

---

## 4. Set Up Atlassian API Token (for Rovo MCP)

Rovo MCP connects to your Jira and Confluence. You'll need an Atlassian API token.

### Steps

1. Go to: https://id.atlassian.com/manage-profile/security/api-tokens
2. Click **"Create API token"**
3. Label it: `mcp-workshop`
4. Click **"Create"**
5. **Copy the token immediately**

### Save your Atlassian credentials

Store these in your shell profile:

```sh
export ATLASSIAN_EMAIL="your-email@company.com"
export ATLASSIAN_API_TOKEN="your-atlassian-api-token"
export ATLASSIAN_SITE_URL="https://your-site.atlassian.net"
```

> **⚠️ Important:** The `ATLASSIAN_SITE_URL` must be your full Atlassian site URL (e.g., `https://myteam.atlassian.net`). Ask your team admin if unsure.

### Quick verify

Open your site URL in a browser and confirm you can access Jira and Confluence.

---

## 5. Configure MCP Servers

Now connect both MCP servers to your Copilot CLI.

### Option A: Config file setup

Create or edit your MCP configuration file:

**macOS / Linux:**
```sh
mkdir -p ~/.config/github-copilot
nano ~/.config/github-copilot/mcp.json
```

Add the following configuration:

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "<paste-your-github-pat>"
      }
    },
    "rovo": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-rovo-server"],
      "env": {
        "ATLASSIAN_API_TOKEN": "<paste-your-atlassian-token>",
        "ATLASSIAN_EMAIL": "<your-email@company.com>",
        "ATLASSIAN_SITE_URL": "https://<your-site>.atlassian.net"
      }
    }
  }
}
```

> **🔒 Security Note:** Never commit this file to version control. It contains sensitive tokens.

### Option B: Using environment variables

If you exported the tokens in Step 3 & 4, you can reference them:

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "rovo": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-rovo-server"],
      "env": {
        "ATLASSIAN_API_TOKEN": "${ATLASSIAN_API_TOKEN}",
        "ATLASSIAN_EMAIL": "${ATLASSIAN_EMAIL}",
        "ATLASSIAN_SITE_URL": "${ATLASSIAN_SITE_URL}"
      }
    }
  }
}
```

---

## 6. Verify Everything Works

### Test 1: GitHub MCP Server

Open Copilot CLI and try:

```
"List open pull requests in <owner>/<repo>"
```

Replace `<owner>/<repo>` with a repo you have access to. You should see a list of PRs.

### Test 2: Rovo MCP Server

Try:

```
"Search for recent issues in project <YOUR-PROJECT-KEY>"
```

Replace `<YOUR-PROJECT-KEY>` with a Jira project key you have access to (e.g., `PROJ`, `ENG`). You should see Jira issues returned.

### Test 3: Confirm both servers are connected

When you start Copilot CLI, you should see MCP servers listed in the startup output. Look for lines mentioning:
- `github` — GitHub MCP Server
- `rovo` — Rovo (Atlassian) MCP Server

---

## 🛠 Troubleshooting

### "npx: command not found"
- Node.js is not installed or not in your PATH.
- Reinstall Node.js from https://nodejs.org and restart your terminal.

### "GitHub token unauthorized / 401"
- Token may have expired or lack required permissions.
- Regenerate the token at https://github.com/settings/tokens and ensure the permissions from Step 3 are granted.

### "Atlassian authentication failed"
- Double-check `ATLASSIAN_EMAIL` matches your Atlassian login email.
- Ensure `ATLASSIAN_SITE_URL` uses `https://` and ends with `.atlassian.net`.
- Verify the API token at https://id.atlassian.com/manage-profile/security/api-tokens.

### "MCP server not found / failed to start"
- Run `npx -y @modelcontextprotocol/server-github --help` to verify the package installs.
- Check your `mcp.json` for JSON syntax errors (missing commas, brackets).
- Try running the MCP server manually:
  ```sh
  GITHUB_TOKEN="your-token" npx -y @modelcontextprotocol/server-github
  ```

### "No MCP servers listed on startup"
- Verify config file location: `~/.config/github-copilot/mcp.json`
- Ensure the JSON is valid — use https://jsonlint.com to check.

---

## 📝 What to Bring to the Workshop

1. ✅ A laptop with everything above installed and verified
2. ✅ Access to at least one GitHub repo with open PRs (for the GitHub MCP exercise)
3. ✅ Access to a Jira project with some issues (for the Rovo exercise)
4. ✅ A terminal / shell you're comfortable with

---

## 🆘 Need Help?

If you run into issues during setup, reach out before the session so we can troubleshoot in advance. Don't wait until the workshop day!

---

**See you at the workshop! 🚀**
