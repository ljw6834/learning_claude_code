# GitHub MCP Server Setup and Usage Guide

## What is MCP?

MCP (Model Context Protocol) is a protocol that allows AI assistants like Claude to interact with external tools and services. The GitHub MCP server enables Claude Code to perform GitHub operations directly — creating issues, branches, pull requests, reviews, and more.

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- A GitHub account with a personal access token (PAT)

## Setup

### 1. Generate a GitHub Personal Access Token

1. Go to **GitHub > Settings > Developer settings > Personal access tokens > Tokens (classic)**
2. Click **Generate new token (classic)**
3. Select the required scopes:
   - `repo` — full control of private repositories
   - `read:org` — read org membership (if working with org repos)
4. Copy the generated token

### 2. Configure the GitHub MCP Server in Claude Code

Add the GitHub MCP server to your Claude Code settings file (`~/.claude/settings.json` or project-level `.claude/settings.json`):

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-github-pat>"
      }
    }
  }
}
```

### 3. Verify the Server is Ready

Launch Claude Code and check that the `mcp__github__*` tools appear in the available tools list. You can ask Claude:

```
Is the GitHub MCP server ready?
```

## Usage Examples

### Create an Issue

```
Create an issue in owner/repo with title "Bug: login fails"
```

### Create a Branch

```
Create a branch called "feature/new-feature" in owner/repo
```

### Add a File to a Branch

```
Add a new file src/Example.java to the feature/new-feature branch in owner/repo
```

### Create a Pull Request

```
Create a PR in owner/repo from feature/new-feature to master with title "Add new feature"
```

### Review a Pull Request

```
Review PR #3 in owner/repo and add comments
```

## Available MCP GitHub Tools

| Tool | Description |
|------|-------------|
| `create_issue` | Create a new issue |
| `get_issue` / `list_issues` | Read issues |
| `update_issue` | Update an existing issue |
| `create_branch` | Create a new branch |
| `create_or_update_file` | Add or modify files in a branch |
| `create_pull_request` | Open a new pull request |
| `get_pull_request` / `list_pull_requests` | Read pull requests |
| `get_pull_request_files` | View changed files in a PR |
| `create_pull_request_review` | Post a review with inline comments |
| `merge_pull_request` | Merge a pull request |
| `search_code` / `search_issues` / `search_repositories` | Search across GitHub |
| `fork_repository` / `create_repository` | Manage repositories |

## Walkthrough: End-to-End Example

This is what we did in our session with the `ljw6834/StreamReduce` repo:

1. **Created an issue** — `mcp-server-issue-2` ([#2](https://github.com/ljw6834/StreamReduce/issues/2))
2. **Created a branch** — `feature/testing-mcp` from `master`
3. **Added a Java file** — `src/McpTest.java` committed to the new branch
4. **Opened a PR** — "Testing MCP" ([#3](https://github.com/ljw6834/StreamReduce/pull/3)) merging `feature/testing-mcp` into `master`
5. **Reviewed the PR** — Posted a review with inline code suggestions

All steps were performed entirely through the GitHub MCP server without leaving Claude Code.
