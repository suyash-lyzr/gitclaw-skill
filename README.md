# GitClaw + GitAgent Skill

A skill for AI coding assistants (Claude Code, Cursor, Codex, Copilot, and more) that teaches them how to create, configure, and run AI agents using the GitAgent spec and GitClaw runtime.

## What is GitAgent / GitClaw?

- **GitAgent** — A spec that defines how to structure an AI agent as a folder of markdown files (SOUL.md, RULES.md, skills/, etc.)
- **GitClaw** — A runtime engine that reads a GitAgent folder and runs it as a working AI agent

Write your agent once as markdown files, run it anywhere. [Learn more](https://github.com/open-gitagent/gitagent)

## Install

Works with Claude Code, Cursor, Codex, Cline, Gemini CLI, GitHub Copilot, and 7+ more tools:

```bash
npx skills add suyash-lyzr/gitclaw-skill --skill gitclaw-agent
```

### Prerequisites

```bash
npm install -g gitclaw
npm install -g @open-gitagent/gitagent
```

## What you can do after installing

Just ask your AI assistant in natural language:

| What you say | What happens |
|---|---|
| "Create an agent for code reviews" | Scaffolds a complete GitClaw agent with SOUL.md, RULES.md, and skills |
| "Add a skill for bug triage to my agent" | Creates a new `skills/bug-triage/SKILL.md` |
| "Export this agent for Cursor" | Runs `gitagent export --format cursor` |
| "Run my agent" | Runs `gitclaw --dir . "message"` |

No need to remember commands or file formats — the skill handles it.

## Example

```bash
# 1. Install the skill
npx skills add suyash-lyzr/gitclaw-skill --skill gitclaw-agent

# 2. Open any folder in Claude Code / Cursor

# 3. Just ask:
#    "Create a new GitClaw agent for meeting notes"
#    → Creates the full agent structure, ready to run
```

## What the skill covers

- Agent folder structure (agent.yaml, SOUL.md, RULES.md, skills/, memory/, workflows/)
- GitClaw CLI commands (init, run, model override, GitHub repo mode)
- GitAgent CLI commands (validate, export, info, audit)
- Export formats (Claude Code, Cursor, OpenAI, CrewAI, Codex, Gemini, and more)
- Available models (OpenAI, Anthropic, Google, Groq, Mistral)
- Built-in tools (cli, read, write, memory, skill_learner, task_tracker)

## Links

- [GitAgent spec](https://github.com/open-gitagent/gitagent)
- [GitClaw runtime](https://github.com/open-gitagent/gitclaw)
- [Enterprise Skills](https://github.com/open-gitagent/enterprise-skills)
- [skills.sh](https://skills.sh)
