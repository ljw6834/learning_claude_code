# Learning Claude Code

A Docker-based project that runs Claude AI in a container and uses it to develop/modify a Spring Boot application.

## Overview

This project uses `docker-compose.yml` to pull Claude into a container environment where you can interact with Claude CLI to develop the learningGit Spring Boot application.

## Quick Start

### 1. Start the Docker Container
```bash
docker compose up
```
Runs in terminal 1 and starts all services.

### 2. Verify Container is Running
```bash
docker ps -a
```

### 3. Launch Claude
### 3.1 Launch Claude in a New Terminal
```bash
docker exec -it learning_claude_code-claude-1 claude
```
Runs in terminal 2 for Claude interaction.
You have to manually
copy the long url and make it to one line, then paste it in browser url. Once you get the code, copy back to claude cli.

### 3.2 Launch Claude in KASM VCN workspace via browser
open your browser, type: https://localhost:6901, user=kasm_user, pw=1234test (configured in `docker-compose.yml` (line 11) )
once log into KASM VCN, wait for claude code terminal show up or click "connect" on the left panel, you will be prompted to 
claude code login. 

### 4. Use Claude to Work on the Project
- Ask Claude to clone the repo: `https://github.com/ljw6834/learningGit.git`
- Request Claude to make changes to the Spring Boot application
- Claude will modify the code inside the container

### 5. Access the Application from Host Machine

Once the Spring Boot application starts, open your browser on the host machine:

- Students endpoint: `http://localhost:8080/students`
- Books endpoint: `http://localhost:8080/books`

> Port mapping is configured in `docker-compose.yml` (line 6)

## Useful Docker Commands

| Command | Purpose |
|---------|---------|
| `docker compose up` | Start all containers |
| `docker ps -a` | List all containers |
| `docker logs learning_claude_code-claude-1` | View container logs |
| `docker stop learning_claude_code-claude-1` | Stop the container |
| `docker restart learning_claude_code-claude-1` | Restart the container |
| `docker exec -it learning_claude_code-claude-1 bash` | Access container shell |
| `docker exec -it learning_claude_code-claude-1 claude` | Start Claude CLI |

## Pushing Changes Back to GitHub

If Claude makes changes you want to keep:

1. **Access the container shell:**
   ```bash
   docker exec -it learning_claude_code-claude-1 bash
   ```

2. **Push changes to GitHub:**
   - Refer to `commands_used.sh` for Git commands
   - Example:
     ```bash
     git add .
     git commit -m "Changes made by Claude"
     git push origin main
     ```

## Project Structure

- `docker-compose.yml` - Container configuration and port mappings
- `commands_used.sh` - Reference for common Git and Docker commands
- Spring Boot application - Located at `https://github.com/ljun6834/learning_claude_code.git`

## Notes

- All development happens inside the Claude container
- Access applications from host via mapped ports
- Git operations require container shell access
