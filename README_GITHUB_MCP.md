# GitHub MCP Server Setup and Usage Guide

## What is MCP?

MCP (Model Context Protocol) is a protocol that allows AI assistants like Claude to interact with external tools and services. The GitHub MCP server enables Claude Code to perform GitHub operations directly — creating issues, branches, pull requests, reviews, and more.

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- A GitHub account with a personal access token (PAT)

## Setup

### 1. Generate a GitHub Personal Access Token

#### Option A: Via GitHub Web UI

1. Go to **GitHub > Settings > Developer settings > Personal access tokens > Tokens (classic)**
2. Click **Generate new token (classic)**
3. Select the required scopes:
   - `repo` — full control of private repositories
   - `read:org` — read org membership (if working with org repos)
4. Copy the generated token

#### Option B: Via GitHub CLI (`gh`)

If you have the [GitHub CLI](https://cli.github.com/) installed, you can generate a PAT directly from the terminal:

1. **Install gh** (if not already installed):

   ```bash
   # macOS
   brew install gh

   # Ubuntu/Debian
   sudo apt install gh

   # Windows
   winget install GitHub.cli
   ```

2. **Authenticate with GitHub**:

   ```bash
   gh auth login
   ```

   Follow the interactive prompts to log in via browser or token.

3. **Generate a PAT with the required scopes**:

   ```bash
   gh auth token
   ```

   This prints your current authentication token. To create a new token with specific scopes:

   ```bash
   gh auth refresh -s repo,read:org
   gh auth token
   ```

   The first command requests additional scopes (`repo` and `read:org`), and the second prints the updated token.

4. **Verify your authentication status**:

   ```bash
   gh auth status
   ```

   This shows which account you're logged into and what scopes are available.

5. **Copy the token** from the output of `gh auth token` and use it in your MCP server configuration below.

#### Where is the Token Stored?

When you authenticate with `gh auth login`, the token is stored in a **secure, platform-specific credential store**:

| Platform | Default Storage |
|----------|----------------|
| macOS | macOS Keychain (`login` keychain) |
| Linux | A flat file at `~/.config/gh/hosts.yml` (if no system keyring is available) |
| Windows | Windows Credential Manager |

You can check where your credentials are stored by running:

```bash
gh auth status
```

The `hosts.yml` file (on Linux) looks like:

```yaml
github.com:
  oauth_token: gho_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  user: your-username
  git_protocol: https
```

> **Security note:** On Linux without a keyring, the token is stored in plain text in `~/.config/gh/hosts.yml`. Make sure this file has restricted permissions (`chmod 600 ~/.config/gh/hosts.yml`).

#### Does the Token Expire?

It depends on the token type:

| Token Type | Expiration |
|-----------|------------|
| **OAuth token** (created via `gh auth login`) | **Does not expire** by default. It remains valid until you manually revoke it via `gh auth logout` or from GitHub Settings > Applications. |
| **Classic PAT** (created via GitHub web UI) | You choose the expiration when creating it — options range from 7 days to no expiration. |
| **Fine-grained PAT** (created via GitHub web UI) | **Must have an expiration** — maximum 1 year. GitHub will send email reminders before it expires. |

To check if your current token is still valid:

```bash
gh auth status
```

If the token has been revoked or expired, re-authenticate with:

```bash
gh auth login
```

#### Installing `gh` Without System-wide Access

If you don't have `sudo` or admin privileges, you can install `gh` locally:

##### Option 1: Download the Binary Directly

```bash
# Create a local bin directory
mkdir -p ~/bin

# Download the latest gh release (Linux amd64 example)
GH_VERSION=$(curl -s https://api.github.com/repos/cli/cli/releases/latest | grep '"tag_name"' | cut -d'"' -f4 | sed 's/^v//')
curl -L "https://github.com/cli/cli/releases/download/v${GH_VERSION}/gh_${GH_VERSION}_linux_amd64.tar.gz" -o /tmp/gh.tar.gz

# Extract and move the binary
tar -xzf /tmp/gh.tar.gz -C /tmp
cp /tmp/gh_${GH_VERSION}_linux_amd64/bin/gh ~/bin/

# Clean up
rm -rf /tmp/gh.tar.gz /tmp/gh_${GH_VERSION}_linux_amd64
```

##### Option 2: Install via Conda (if available)

```bash
conda install -c conda-forge gh
```

##### Option 3: Install via Homebrew (Linux)

```bash
# Install Homebrew for Linux (no sudo required)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Then install gh
brew install gh
```

##### Update `.bashrc` to Add `~/bin` to PATH

After installing to `~/bin`, add it to your `PATH` so you can run `gh` from anywhere:

```bash
# Add to ~/.bashrc (or ~/.zshrc if using zsh)
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc

# Reload your shell configuration
source ~/.bashrc

# Verify gh is accessible
gh --version
```

> **Tip for containers or CI environments:** If you're running Claude Code inside a Docker container (like a Kasm workspace), the local binary approach (Option 1) is the most reliable since you may not have a package manager or `sudo` access.

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
