---
name: gitclaw-agent
description: Create, configure, and run AI agents using the GitAgent spec and GitClaw runtime. Use when the user wants to build an AI agent, scaffold an agent folder, write SOUL.md/RULES.md/skills, run agents with GitClaw, or export agents to other formats.
---

# GitClaw Agent Builder

You help the user create and manage AI agents using the GitAgent specification and GitClaw runtime.

## What is what

- **GitAgent** — a spec/standard that defines how an AI agent is structured as a folder of files
- **GitClaw** — the runtime engine that reads a GitAgent folder and runs it (CLI + Node.js SDK)
- **gitagent** (CLI) — scaffolds, validates, and exports agent folders
- **gitclaw** (CLI) — runs agents, manages sessions, commits memory

## Prerequisites

```bash
npm install -g gitclaw              # runtime
npm install -g @open-gitagent/gitagent  # spec CLI (for validate/export)
```

## Creating a new agent

### Step 1: Initialize

```bash
cd <project-folder>
gitclaw init
```

This creates the scaffolded folder:
```
├── agent.yaml        # manifest — name, model, tools
├── SOUL.md           # identity — who the agent is
├── memory/           # persistent storage (auto-committed to git)
├── workspace/        # scratch space
└── .git/             # memory is versioned with git
```

### Step 2: Configure agent.yaml

```yaml
spec_version: "0.1.0"
name: <agent-name>
version: 0.1.0
description: <what the agent does>
model:
  preferred: "openai:gpt-4o-mini"    # or anthropic:claude-sonnet-4-6, google:gemini-pro, etc.
  fallback: []
tools: [cli, read, write, memory]     # available tools
skills:                                # list skill folders
  - skills/<skill-name>
runtime:
  max_turns: 50
```

### Step 3: Write SOUL.md (the most important file)

SOUL.md is the system prompt — who the agent IS. Be specific, not vague. Include:
- Identity (who are you)
- Purpose (what do you do)
- Behavior rules (how do you act)
- Explicit tool usage instructions (HOW to use memory, cli, etc.)

Tips for good SOUL.md:
- Vague instructions = vague behavior. Be explicit.
- Tell the agent HOW to check memory: "Run `ls memory/` then read recent files"
- Tell the agent HOW to get the date: "Run `date +%Y-%m-%d`"
- Tell the agent to append, not overwrite: "Read existing files before writing"

### Step 4: Write RULES.md (hard boundaries)

RULES.md defines what the agent must NEVER do. These are non-negotiable constraints.
- What to never do (delete files, run destructive commands, make things up)
- What to always do (check date, append to files, use structured format)

### Step 5: Add skills (optional)

Create `skills/<skill-name>/SKILL.md` for each specialized task:

```markdown
---
name: <skill-name>
description: <when to use this skill — one line>
---

# <Skill Name>

When the user asks to <trigger condition>:

1. Step one
2. Step two
3. Step three

## Output Format
<define expected output structure>
```

### Step 6: Add .env for API keys

Create `.env` in the agent folder root:
```
OPENAI_API_KEY=sk-xxx
# or ANTHROPIC_API_KEY=sk-ant-xxx
# or GEMINI_API_KEY=xxx
```

Add `.env` to `.gitignore` so keys are never committed.

## Running an agent

```bash
# Run with a message
gitclaw --dir <agent-folder> "your message"

# Run interactively
gitclaw --dir <agent-folder>

# Override model
gitclaw --dir <agent-folder> --model anthropic:claude-sonnet-4-6 "your message"

# Run on a GitHub repo
gitclaw --repo https://github.com/org/repo --pat ghp_xxx "your message"
```

## Exporting to other runtimes

```bash
# Export for Claude Code (creates CLAUDE.md)
gitagent export --format claude-code --output CLAUDE.md

# Export for Cursor (creates .mdc rule files)
gitagent export --format cursor --output cursor-rules.md

# Other formats: system-prompt, openai, crewai, lyzr, github, copilot, opencode, gemini, codex
gitagent export --format <format> --output <file>
```

## Other gitagent commands

```bash
gitagent validate           # check folder follows the spec
gitagent info               # display agent summary
gitagent audit              # generate compliance report
```

## Full folder structure (all optional except agent.yaml + SOUL.md)

```
my-agent/
├── agent.yaml        # manifest (REQUIRED)
├── SOUL.md           # identity (REQUIRED)
├── RULES.md          # hard constraints
├── DUTIES.md         # segregation of duties (compliance)
├── .env              # API keys (never commit)
├── skills/           # specialized task instructions
│   └── <name>/SKILL.md
├── knowledge/        # reference docs the agent can consult
├── memory/           # persists across sessions via git
├── workflows/        # multi-step YAML playbooks
├── hooks/            # bootstrap.md (session start), teardown.md (session end)
├── agents/           # sub-agents (nested, same structure)
└── compliance/       # regulatory compliance artifacts
```

## Available models

- `openai:gpt-4o-mini`, `openai:gpt-4o`, `openai:o3`
- `anthropic:claude-sonnet-4-6`, `anthropic:claude-opus-4-6`
- `google:gemini-pro`, `google:gemini-2.0-flash`
- `groq:llama-3.3-70b-versatile`
- `mistral:mistral-large-latest`

## Built-in GitClaw tools

- `cli` — execute shell commands
- `read` — read files
- `write` — write/create files
- `memory` — load/save git-committed memory
- `skill_learner` — auto-creates skills from complex tasks
- `task_tracker` — tracks multi-step task progress

## Composio integrations (connect to 500+ external apps)

GitClaw integrates with Composio to connect agents to external tools like Gmail, Slack, Google Calendar, Notion, Jira, GitHub, and 500+ more.

### Setup

1. Get an API key from https://composio.dev
2. Add to `.env`:
```
COMPOSIO_API_KEY=ak_xxx
```
3. Connect apps through Composio's auth flow (OAuth)
4. GitClaw automatically discovers connected tools and makes them available to the agent

### How it works

- Composio tools appear as `composio_<toolkit>_<action>` (e.g., `composio_gmail_SEND_EMAIL`)
- The agent can use them like any other tool
- GitClaw's Composio adapter handles auth, tool discovery, and execution

### Example use cases

```bash
# Agent that sends email summaries
gitclaw --dir . "Email Rahul a summary of this week's work"

# Agent that creates calendar events
gitclaw --dir . "Schedule a meeting with Priya tomorrow at 3pm"

# Agent that posts to Slack
gitclaw --dir . "Post the weekly summary to #engineering channel"
```

### SOUL.md tip for Composio agents

In SOUL.md, tell the agent which integrations it has:
```markdown
# Integrations
You are connected to Gmail and Slack via Composio.
- Use Gmail to send emails and read inbox
- Use Slack to post messages to channels
Always confirm with the user before sending any external message.
```

## Key principles

- Agent = folder of markdown files, not code
- Change behavior by editing a markdown file — no redeploy needed
- Memory is persisted via git commits automatically
- Skills are loaded dynamically at runtime
- Write once (GitAgent spec) → export to any runtime
