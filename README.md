# GitClaw + GitAgent Skill

A skill for AI coding assistants (Claude Code, Cursor, Codex, Copilot, Cline, Gemini CLI, and 7+ more) that teaches them how to create, configure, and run AI agents using the GitAgent spec and GitClaw runtime.

## What is GitAgent / GitClaw?

- **GitAgent** — A spec that defines how to structure an AI agent as a folder of markdown files (SOUL.md, RULES.md, skills/, etc.)
- **GitClaw** — A runtime engine that reads a GitAgent folder and runs it as a working AI agent (CLI + Node.js SDK)

Write your agent once as markdown files, run it anywhere. [Learn more](https://github.com/open-gitagent/gitagent)

## Install

Works with Claude Code, Cursor, Codex, Cline, Gemini CLI, GitHub Copilot, and 7+ more tools:

```bash
npx skills add suyash-lyzr/gitclaw-skill --skill gitclaw-agent
```

### Prerequisites

```bash
npm install -g gitclaw                      # runtime engine
npm install -g @open-gitagent/gitagent      # spec CLI
```

### Update to latest version

```bash
npx skills update
```

## What you can do after installing

Just ask your AI assistant in natural language:

| What you say | What happens |
|---|---|
| "Create an agent for code reviews" | Scaffolds a complete GitClaw agent with SOUL.md, RULES.md, and skills |
| "Add a skill for bug triage to my agent" | Creates a new `skills/bug-triage/SKILL.md` |
| "Export this agent for Cursor" | Runs `gitagent export --format cursor` |
| "Run my agent" | Runs `gitclaw --dir . "message"` |
| "Connect my agent to Gmail" | Sets up Composio integration for email |
| "Add a weekly summary workflow" | Creates a workflow in `workflows/` |
| "Add compliance rules for FINRA" | Configures compliance in `agent.yaml` |
| "Set up a cron job to run my agent daily" | Creates a schedule in `schedules/` |
| "Create a KYC compliance agent" | Auto-matches and installs enterprise skills (kyc-aml-compliance, regulatory-compliance-monitoring, etc.) |
| "Build an insurance claims agent" | Auto-adds claims-adjudication, policy-administration, subrogation skills |

No need to remember commands or file formats — the skill handles it.

### Enterprise Skills Auto-Matching

When you create an agent, the skill automatically checks against **43 pre-built enterprise skills** from [open-gitagent/enterprise-skills](https://github.com/open-gitagent/enterprise-skills) and installs matching ones. Covers: Finance, Banking, Insurance, Healthcare, Legal, Sales, Marketing, HR, and Procurement.

## Everything this skill covers

### Core
- Agent folder structure (`agent.yaml`, `SOUL.md`, `RULES.md`, `DUTIES.md`, skills/, memory/, workflows/)
- Creating, configuring, and running agents
- Writing effective SOUL.md and RULES.md files
- GitClaw CLI (init, run, voice, sandbox, sessions)
- GitAgent CLI (validate, export, import, audit, info)

### Skills & Tools
- Creating and managing skills (`SKILL.md` format)
- Installing skills from registries (skills.sh, GitHub)
- Self-evolving skills via `skill_learner`
- Built-in tools (cli, read, write, memory, task_tracker, skill_learner)
- Custom declarative tools (YAML format)
- Custom tools via SDK

### Integrations
- **Composio** — connect to 500+ apps (Gmail, Slack, Google Calendar, Notion, Jira, GitHub, etc.)
- **Telegram** — bot integration with file/photo support
- **WhatsApp** — QR auth, contact management, message triggers
- **Lyzr Studio** — create, update, and run agents on Lyzr

### Advanced Features
- **Workflows** — multi-step YAML playbooks with dependency DAGs and approval gates
- **Hooks** — lifecycle control (session start, pre/post tool use, errors) via scripts or SDK
- **Sub-agents** — nested agents with delegation config
- **Plugins** — extend agents with tools, hooks, skills, and prompt content
- **Schedules** — cron-based recurring agent execution
- **Voice mode** — browser UI with OpenAI Realtime and Gemini Live audio
- **Sandbox** — E2B cloud VMs or NVIDIA OpenShell (Docker + Landlock)
- **Sessions** — git branch-based sessions with auto-commit and resume

### Compliance & Enterprise
- Risk tiers (low, medium, high, critical)
- Human-in-the-loop controls
- Audit logging with retention policies
- Regulatory frameworks (FINRA, SEC, GDPR, EU AI Act, Federal Reserve, CFPB, and more)
- Segregation of duties (maker-checker patterns)
- Model risk management (SR 11-7)
- Data governance (PII handling, classification)

### Portability
- Export to 14 formats (Claude Code, Cursor, OpenAI, CrewAI, Lyzr, GitHub, Copilot, Codex, Gemini, OpenClaw, OpenCode, Nanobot, Kiro, system-prompt)
- Import from 3 sources (Claude Code, Cursor, CrewAI)
- Agent inheritance and dependencies
- Environment-specific configs (dev, staging, production)
- 6+ model providers (OpenAI, Anthropic, Google, Groq, Mistral, local via Ollama)

## Example

```bash
# 1. Install the skill
npx skills add suyash-lyzr/gitclaw-skill --skill gitclaw-agent

# 2. Open any folder in Claude Code / Cursor

# 3. Just ask:
#    "Create a new GitClaw agent for meeting notes"
#    → Creates the full agent structure, ready to run
#
#    "Connect it to Gmail and send me a daily summary at 9am"
#    → Sets up Composio + schedule, ready to go
```

## Links

- [GitAgent spec](https://github.com/open-gitagent/gitagent)
- [GitClaw runtime](https://github.com/open-gitagent/gitclaw)
- [Enterprise Skills](https://github.com/open-gitagent/enterprise-skills)
- [skills.sh](https://skills.sh)
