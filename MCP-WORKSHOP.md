# MCP Servers for Code Review — Workshop Guide

> A 30-minute hands-on session covering **Rovo (Atlassian)** and **GitHub MCP Server** integration for code review workflows using Copilot CLI.

---

## 🎯 Session Objectives

By the end of this session, participants will:
1. Understand what **MCP (Model Context Protocol)** is and why it matters
2. Know how **Rovo** and **GitHub MCP Server** enhance code review workflows
3. See live demos of MCP-powered code review in action
4. Try hands-on exercises using MCP tools

---

## 📋 Agenda (30 min)

| Time | Topic |
|------|-------|
| 0–5 min | What is MCP? Why MCP for code review? |
| 5–10 min | Rovo MCP Server — Overview & Demo |
| 10–18 min | GitHub MCP Server — Overview & Demo |
| 18–25 min | Hands-on Exercise |
| 25–30 min | Q&A and Wrap-up |

---

## 🧠 Part 1: What is MCP? (5 min)

### The Problem
- AI assistants are powerful, but **isolated** — they can't see your Jira tickets, Confluence docs, PRs, or CI status unless you copy-paste context manually.
- Code review requires cross-tool context: **PRs ↔ Jira issues ↔ Confluence docs ↔ CI pipelines**.

### The Solution: MCP
- **Model Context Protocol (MCP)** is an open standard (by Anthropic) that lets AI assistants connect to external tools and data sources.
- Think of it as **USB-C for AI** — a universal plug that connects your AI to any tool.

### How It Works
```
┌──────────────┐      MCP Protocol       ┌──────────────────┐
│  AI Client   │ ◄──────────────────────► │   MCP Server     │
│ (Copilot CLI)│    (JSON-RPC over STDIO) │ (Rovo, GitHub,   │
│              │                          │  Playwright...)   │
└──────────────┘                          └──────────────────┘
                                                  │
                                          ┌───────┴────────┐
                                          │ External Tools  │
                                          │ (Jira, GitHub,  │
                                          │  Confluence...) │
                                          └────────────────┘
```

### Key Concepts
| Concept | Description |
|---------|-------------|
| **MCP Server** | A service that exposes tools/resources via the MCP protocol |
| **MCP Client** | An AI application (e.g., Copilot CLI) that connects to MCP servers |
| **Tools** | Functions the AI can call (e.g., search Jira, get PR diff) |
| **Resources** | Data the AI can read (e.g., Confluence pages, repo files) |

---

## 🔵 Part 2: Rovo MCP Server (Atlassian) (5 min)

### What is Rovo?
Rovo is Atlassian's AI-powered MCP server that gives your AI assistant direct access to:
- **Jira** — issues, transitions, comments, worklogs
- **Confluence** — pages, spaces, search, comments
- **Rovo Search** — unified search across all Atlassian products

### Tools Available

#### Jira Tools
| Tool | What It Does |
|------|-------------|
| `searchJiraIssuesUsingJql` | Search issues using JQL |
| `getJiraIssue` | Get full issue details |
| `editJiraIssue` | Update issue fields |
| `transitionJiraIssue` | Move issue through workflow |
| `addCommentToJiraIssue` | Add comments to issues |
| `createIssueLink` | Link related issues |

#### Confluence Tools
| Tool | What It Does |
|------|-------------|
| `searchConfluenceUsingCql` | Search pages using CQL |
| `getConfluencePage` | Read page content |
| `createConfluencePage` | Create new pages |
| `getConfluencePageInlineComments` | Read inline comments |

#### Search
| Tool | What It Does |
|------|-------------|
| `searchAtlassian` | Unified search across Jira + Confluence |

### 🎬 Demo: Rovo in Action

**Scenario:** You're reviewing a PR and need context from Jira and Confluence.

**Demo Steps:**

1. **Search for related Jira issues:**
   ```
   "Find all open bugs in project XYZ related to authentication"
   ```
   → Uses `searchJiraIssuesUsingJql` with JQL: `project = XYZ AND type = Bug AND status != Done AND text ~ "authentication"`

2. **Get issue details:**
   ```
   "Show me the details of PROJ-123"
   ```
   → Uses `getJiraIssue` to fetch full context including description, comments, linked issues

3. **Search Confluence for design docs:**
   ```
   "Find the authentication design document in Confluence"
   ```
   → Uses `searchAtlassian` to find relevant docs

4. **Add a review comment to Jira:**
   ```
   "Add a comment to PROJ-123 saying the fix has been reviewed and looks good"
   ```
   → Uses `addCommentToJiraIssue` to update the ticket

---

## 🐙 Part 3: GitHub MCP Server (8 min)

### What is GitHub MCP Server?
The GitHub MCP Server connects your AI assistant to GitHub, providing access to:
- **Pull Requests** — diffs, reviews, comments, status checks
- **Issues** — search, read, comments
- **Actions** — workflow runs, job logs
- **Code Search** — search across repositories
- **Repository** — files, branches, commits

### Tools Available

#### Pull Request Tools
| Tool | What It Does |
|------|-------------|
| `pull_request_read` (get) | Get PR details |
| `pull_request_read` (get_diff) | Get the full diff |
| `pull_request_read` (get_files) | List changed files |
| `pull_request_read` (get_review_comments) | Get review threads |
| `pull_request_read` (get_check_runs) | Get CI/CD status |
| `list_pull_requests` | List PRs in a repo |
| `search_pull_requests` | Search PRs across repos |

#### Issue Tools
| Tool | What It Does |
|------|-------------|
| `list_issues` | List issues in a repo |
| `issue_read` (get) | Get issue details |
| `issue_read` (get_comments) | Get issue comments |
| `search_issues` | Search issues across repos |

#### CI/CD Tools
| Tool | What It Does |
|------|-------------|
| `actions_list` (list_workflow_runs) | List workflow runs |
| `get_job_logs` | Get CI/CD job logs |
| `actions_get` (get_workflow_run) | Get run details |

#### Code Tools
| Tool | What It Does |
|------|-------------|
| `search_code` | Search code across GitHub |
| `get_file_contents` | Read file from a repo |
| `list_commits` | List commits on a branch |
| `get_commit` | Get commit details + diff |

### 🎬 Demo: GitHub MCP Server for Code Review

**Scenario:** Review a PR end-to-end using MCP tools.

**Demo Steps:**

1. **List open PRs:**
   ```
   "Show me the open pull requests in owner/repo"
   ```
   → Uses `list_pull_requests` with state: "open"

2. **Get PR diff and review:**
   ```
   "Review PR #42 in owner/repo"
   ```
   → Uses `pull_request_read` (get_diff) → AI analyzes the diff for bugs, security issues, logic errors

3. **Check CI status:**
   ```
   "What's the CI status on PR #42?"
   ```
   → Uses `pull_request_read` (get_check_runs) to show pipeline status

4. **Investigate a failing CI job:**
   ```
   "Show me the logs for the failed job on PR #42"
   ```
   → Uses `get_job_logs` with `failed_only: true`

5. **Find related issues:**
   ```
   "Find issues related to this PR's changes"
   ```
   → Uses `search_issues` to find connected work

---

## ✋ Part 4: Hands-On Exercise (7 min)

### Exercise 1: Jira + Confluence Lookup (Rovo)

> **Goal:** Use Rovo MCP to gather context for a code review.

**Try these prompts in Copilot CLI:**

```
# 1. Search for recent issues in your project
"Find the 5 most recently updated issues in project <YOUR-PROJECT>"

# 2. Get details of a specific issue
"Show me the details of <YOUR-PROJECT>-<ISSUE-NUMBER>"

# 3. Search Confluence for related docs
"Search Confluence for pages about <topic relevant to your team>"
```

**What to observe:**
- How the AI translates natural language → JQL/CQL queries
- The structured data returned from Jira/Confluence
- How context from these tools could inform a code review

---

### Exercise 2: PR Review (GitHub MCP)

> **Goal:** Use GitHub MCP Server to review a pull request.

**Try these prompts in Copilot CLI:**

```
# 1. List open PRs in a repo you have access to
"List open pull requests in <owner>/<repo>"

# 2. Get the diff for a specific PR
"Show me what changed in PR #<number> in <owner>/<repo>"

# 3. Check CI status
"What's the CI/CD status on PR #<number> in <owner>/<repo>"

# 4. Full AI-powered code review
"Review PR #<number> in <owner>/<repo> — focus on bugs and security issues"
```

**What to observe:**
- How the AI fetches the diff and analyzes it
- The quality of automated review comments
- How CI logs are used to diagnose failures

---

### Exercise 3: Cross-Tool Workflow (Bonus)

> **Goal:** Combine Rovo + GitHub MCP for a full review workflow.

```
# Scenario: A PR fixes a Jira bug. Verify the fix covers all requirements.

# 1. Get the Jira issue requirements
"Show me the details and acceptance criteria for PROJ-456"

# 2. Review the PR that fixes it
"Review PR #78 in owner/repo and check if it addresses all the requirements from PROJ-456"

# 3. Update the Jira issue with review status
"Add a comment to PROJ-456: Code review complete — PR #78 addresses all acceptance criteria"
```

---

## 💡 Tips & Best Practices

### For Effective MCP-Powered Code Reviews

1. **Be specific in your prompts** — mention repo names, PR numbers, issue keys
2. **Combine tools** — ask the AI to cross-reference Jira issues with PR changes
3. **Use the `code-review` agent** — purpose-built for high-signal reviews (bugs, security, logic errors only)
4. **Leverage CI logs** — when a check fails, ask for the logs instead of manually digging

### Common Pitfalls
- ❌ Vague prompts like "review my code" → ✅ "Review PR #42 in owner/repo for security issues"
- ❌ Ignoring CI status → ✅ Always check `get_check_runs` before approving
- ❌ Not linking work → ✅ Cross-reference Jira issues with PRs for traceability

---

## 🔧 Setup Reference

### How MCP Servers are Configured

MCP servers are configured in your Copilot CLI settings. Here's what a typical config looks like:

```json
{
  "mcpServers": {
    "rovo": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-rovo-server"],
      "env": {
        "ATLASSIAN_API_TOKEN": "<your-token>",
        "ATLASSIAN_EMAIL": "<your-email>",
        "ATLASSIAN_SITE_URL": "https://<site>.atlassian.net"
      }
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "<your-token>"
      }
    }
  }
}
```

> **Note:** Exact configuration may vary based on your setup. Refer to each MCP server's documentation for the latest instructions.

---

## 📚 Resources

- [MCP Specification](https://modelcontextprotocol.io/) — The official MCP protocol docs
- [GitHub MCP Server](https://github.com/github/github-mcp-server) — GitHub's official MCP server
- [Atlassian Rovo](https://www.atlassian.com/rovo) — Atlassian's AI assistant platform
- [MCP Server Registry](https://github.com/modelcontextprotocol/servers) — Community MCP servers

---

## ❓ Q&A Talking Points

Anticipate these common questions:

| Question | Answer |
|----------|--------|
| "Is my code sent to the cloud?" | MCP servers run locally via STDIO — your code stays on your machine. The AI model processes it, but MCP itself doesn't send code externally. |
| "Can I build my own MCP server?" | Yes! MCP is an open protocol. You can build servers in Python, TypeScript, or any language. See the [MCP SDK](https://modelcontextprotocol.io/). |
| "How is this different from plugins?" | MCP is a standardized protocol — one server works with any MCP-compatible AI client. Plugins are vendor-specific. |
| "What about security?" | MCP servers use token-based auth. Tokens are stored locally and never shared with the AI model itself. |

---

**Happy reviewing! 🚀**
