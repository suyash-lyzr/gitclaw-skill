---
name: gitclaw-agent
description: Create, configure, and run AI agents using the GitAgent spec and GitClaw runtime. Use when the user wants to build an AI agent, scaffold an agent folder, write SOUL.md/RULES.md/skills, run agents with GitClaw, export agents, set up compliance, connect external apps, create workflows, configure hooks, manage sub-agents, or anything related to GitAgent/GitClaw.
---

# GitClaw + GitAgent — Complete Agent Builder

You help the user create and manage AI agents using the GitAgent specification and GitClaw runtime.

## What is what

- **GitAgent** — a spec/standard that defines how an AI agent is structured as a folder of files. Like HTML for agents.
- **GitClaw** — the runtime engine that reads a GitAgent folder and runs it. Like Chrome for HTML. Available as CLI + Node.js SDK.
- **gitagent** (CLI) — scaffolds, validates, imports, exports, and manages agent folders
- **gitclaw** (CLI) — runs agents, manages sessions, commits memory, connects to external apps

## Prerequisites

```bash
npm install -g gitclaw                      # runtime engine
npm install -g @open-gitagent/gitagent      # spec CLI (validate/export/import)
```

---

# PART 1: CREATING AGENTS

## Step 1: Initialize

Three template options:

```bash
gitagent init --template minimal    # agent.yaml + SOUL.md only
gitagent init --template standard   # + RULES.md, AGENTS.md, skills/, knowledge/, tools/
gitagent init --template full       # + memory/, hooks/, examples/, agents/, compliance/, config/
```

Or use GitClaw directly (also scaffolds + starts interactive session):
```bash
gitclaw init
```

## Step 2: Configure agent.yaml (the manifest)

```yaml
spec_version: "0.1.0"
name: my-agent                          # kebab-case, required
version: 1.0.0                          # semver, required
description: "What this agent does"     # required
author: "name"
license: "MIT"
tags: [tag1, tag2]

model:
  preferred: "anthropic:claude-sonnet-4-6"
  fallback: ["openai:gpt-4o", "google:gemini-2.0-flash"]
  constraints:
    temperature: 0.7
    max_tokens: 4096
    top_p: 0.9

tools: [cli, read, write, memory]

skills:
  - skills/skill-name

agents:
  sub-agent-name:
    delegation:
      mode: auto            # or manual
      triggers: ["review", "audit"]

extends: "https://github.com/org/base-agent.git"   # inherit from parent agent

dependencies:
  - name: fact-checker
    source: https://github.com/org/fact-checker.git
    version: "^1.0.0"
    mount: agents/fact-checker

runtime:
  max_turns: 50
  timeout: 120

# Compliance (see Part 6)
compliance:
  risk_level: medium
  human_in_the_loop: conditional

# A2A protocol
a2a:
  url: https://api.example.com/agent
  capabilities: [review, summarize]
  authentication: bearer
  protocols: ["a2a/1.0"]

# Plugins
plugins:
  plugin-name:
    enabled: true
    source: "https://github.com/org/plugin.git"
    version: "main"
    config:
      api_key: "${MY_API_KEY}"
```

## Step 3: Write SOUL.md (agent identity — MOST IMPORTANT FILE)

SOUL.md is the system prompt. It defines WHO the agent is. Tips:
- Be specific, not vague. Vague instructions = vague behavior.
- Tell the agent HOW to use tools: "Run `ls memory/` then read recent files"
- Tell the agent HOW to get the date: "Run `date +%Y-%m-%d`"
- Tell the agent to append, not overwrite: "Read existing files before writing"
- Include: identity, purpose, behavior rules, tool usage instructions

## Step 4: Write RULES.md (hard boundaries)

Non-negotiable constraints the agent must NEVER break.
- What to never do (delete files, run destructive commands, fabricate data)
- What to always do (check date, append to files, use structured format)

## Step 5: Write DUTIES.md (segregation of duties — for compliance)

Defines role separation for regulated environments:
```markdown
# Duties
- Role: analyst — can research, draft reports
- Role: reviewer — can approve, reject, escalate
- Conflict: analyst and reviewer must be different agents
```

In agent.yaml:
```yaml
compliance:
  segregation_of_duties:
    roles:
      - id: maker
        permissions: [draft, submit]
      - id: checker
        permissions: [review, approve]
    conflicts: [[maker, checker]]
    assignments:
      maker: agent-a
      checker: agent-b
    isolation: full         # full, shared, or none
    enforcement: strict     # strict or advisory
    handoffs:
      regulatory_filing:
        requires: [analyst, reviewer]
```

## Step 6: Add .env for API keys

```
OPENAI_API_KEY=sk-xxx
ANTHROPIC_API_KEY=sk-ant-xxx
GEMINI_API_KEY=xxx
COMPOSIO_API_KEY=ak_xxx
COMPOSIO_USER_ID=default
GITHUB_TOKEN=ghp_xxx
TELEGRAM_BOT_TOKEN=xxx
TELEGRAM_ALLOWED_USERS=user1,user2
LYZR_API_KEY=xxx
```

Always add `.env` to `.gitignore`.

---

# PART 2: SKILLS

## Creating skills

Each skill lives in `skills/<name>/SKILL.md`:

```markdown
---
name: skill-name
description: When to use this skill — one line (max 1024 chars)
license: MIT
allowed-tools: cli read write
metadata:
  author: your-name
  version: 1.0.0
  category: development
  risk_tier: low
  regulatory_frameworks: []
---

# Skill Name

When the user asks to <trigger condition>:

1. Step one
2. Step two
3. Step three

## Output Format
<define expected output structure>
```

Skill folder can include:
```
skills/my-skill/
├── SKILL.md          # instructions (required)
├── scripts/          # executable implementations
├── references/       # supplementary docs
├── assets/           # static resources
└── agents/           # skill-specific sub-agents
```

## Skill discovery priority (first match wins)

1. `<agent>/skills/`
2. `<agent>/.agents/skills/`
3. `<agent>/.claude/skills/`
4. `<agent>/.github/skills/`
5. `~/.agents/skills/` (user global)

## Installing skills from registries

```bash
gitagent skills search "code review"              # search registries
gitagent skills install <skill-id>                 # install
gitagent skills install <skill-id> --global        # install globally
gitagent skills list                               # list installed
gitagent skills list --local                       # list local only
gitagent skills info <skill-id>                    # inspect details
```

Or via skills.sh:
```bash
npx skills add open-gitagent/enterprise-skills --skill contract-review-analysis
```

## Self-evolving skills (skill_learner)

GitClaw has a built-in `skill_learner` tool that auto-creates skills from complex tasks:
- After completing a task, it evaluates: multi-step? non-trivial? novel? generalizable?
- Needs all 4 checks to pass
- Creates `skills/<name>/SKILL.md` and git commits it
- Tracks confidence (0.0-1.0) with reinforcement learning — success increases, failure penalizes 2x
- Skills flagged when confidence drops below 0.4

---

# PART 3: TOOLS

## Built-in GitClaw tools

- `cli` — execute shell commands
- `read` — read files
- `write` — write/create files
- `memory` — load/save git-committed memory
- `skill_learner` — auto-creates skills from complex tasks
- `task_tracker` — tracks multi-step task progress
- `capture_photo` — capture screenshots (voice mode)

## Custom declarative tools

Define in `tools/<name>.yaml`:

```yaml
name: search_docs
description: "Search the documentation"
version: 1.0.0
input_schema:
  properties:
    query: { type: string, description: "Search query" }
    limit: { type: number, description: "Max results" }
  required: [query]
output_schema:
  properties:
    results: { type: array }
implementation:
  script: scripts/search.sh
  runtime: bash           # or node (default)
  timeout: 120
annotations:
  requires_confirmation: false
  read_only: true
  cost: low
```

Tool scripts receive JSON via stdin, return JSON via stdout. 120-second timeout.

## Tool metadata

Each tool can declare:
- `isConcurrencySafe` — can run in parallel (default: false)
- `isReadOnly` — only reads (default: false)
- `isDestructive` — irreversible action (default: false)
- `maxResultSizeChars` — truncate output (default: 50000)

## Tool control via SDK

```typescript
query({
  tools: [customTool],          // add custom tools
  replaceBuiltinTools: true,    // skip cli/read/write/memory
  allowedTools: ["read"],       // allowlist
  disallowedTools: ["cli"],     // denylist
})
```

---

# PART 4: RUNNING AGENTS

## GitClaw CLI

```bash
# Run with a message
gitclaw --dir ./my-agent "your message"

# Run interactively (REPL)
gitclaw --dir ./my-agent

# Short flag
gitclaw -d ./my-agent -p "your message"

# Override model
gitclaw --dir ./my-agent --model anthropic:claude-sonnet-4-6 "message"

# Run on a GitHub repo (clones, creates session branch, auto-commits)
gitclaw --repo https://github.com/org/repo --pat ghp_xxx "Fix the login bug"

# Resume an existing session
gitclaw --session gitclaw/session-a1b2c3d4 --repo https://github.com/org/repo "Continue"

# Run in sandbox
gitclaw --dir ./my-agent --sandbox "message"

# Use environment config
gitclaw --dir ./my-agent --env production "message"

# Voice mode (opens browser UI)
gitclaw --dir ./my-agent --voice
```

## GitAgent run (auto-detects adapter)

```bash
gitagent run --dir ./my-agent --prompt "message"
gitagent run --repo https://github.com/org/agent-repo --prompt "message"
gitagent run --adapter claude --dir ./my-agent            # force specific adapter
gitagent run --repo https://github.com/org/repo --refresh # pull latest
gitagent run --repo https://github.com/org/repo --no-cache # fresh clone
```

## SDK usage (Node.js — in-process, no subprocess)

```typescript
import { query } from "gitclaw";

const q = query({
  prompt: "Review this codebase",
  dir: "./my-agent",
  model: "anthropic:claude-sonnet-4-6",
  maxTurns: 30,
  constraints: { temperature: 0.7, maxTokens: 4096 },
  hooks: { /* see hooks section */ },
});

for await (const msg of q) {
  if (msg.type === "delta") process.stdout.write(msg.content);
  if (msg.type === "tool_use") console.log(`Tool: ${msg.toolName}`);
  if (msg.type === "tool_result") console.log(`Result: ${msg.content}`);
  if (msg.type === "assistant") console.log("\nDone.");
  if (msg.type === "system") console.log(`System: ${msg.subtype}`);
}

// Helper methods on the query object
q.abort();                  // cancel execution
q.steer("new message");    // inject message mid-stream
q.sessionId();             // get session ID
q.manifest();              // get loaded manifest
q.messages();              // get all messages
q.costs();                 // get token/cost tracking
```

## SDK message types

| Type | Description | Key Fields |
|---|---|---|
| `delta` | Streaming text chunk | `deltaType` (text/thinking), `content` |
| `assistant` | Complete LLM response | `content`, `model`, `provider`, `usage`, `thinking` |
| `tool_use` | Tool invocation | `toolName`, `args`, `toolCallId` |
| `tool_result` | Tool output | `content`, `isError`, `toolCallId` |
| `system` | Lifecycle events | `subtype` (session_start/end, hook_blocked, error) |
| `user` | User message | `content` |

## Custom tools via SDK

```typescript
import { query, tool } from "gitclaw";

const weatherTool = tool(
  "get_weather",
  "Get current weather for a city",
  { properties: { city: { type: "string" } }, required: ["city"] },
  async ({ city }) => `Weather in ${city}: 72°F, sunny`
);

for await (const msg of query({
  prompt: "What's the weather in NYC?",
  dir: "./my-agent",
  tools: [weatherTool],
})) { ... }
```

---

# PART 5: WORKFLOWS, HOOKS, MEMORY, KNOWLEDGE

## Workflows

Multi-step YAML playbooks in `workflows/<name>.yaml`:

```yaml
name: deploy-flow
description: "Deploy to production"
version: 1.0.0
inputs:
  - name: branch
    type: string
    required: true
    default: main
outputs:
  - name: deploy_url
    type: string
steps:
  - id: review
    action: "Review changes on branch"
    skill: code-review
    inputs:
      branch: "${{ inputs.branch }}"
    outputs: [review_result]
  - id: deploy
    action: "Deploy to staging"
    skill: deploy
    depends_on: [review]
    inputs:
      branch: "${{ inputs.branch }}"
      review: "${{ steps.review.outputs.review_result }}"
    conditions:
      - "${{ steps.review.outputs.review_result }} == 'approved'"
    compliance:
      audit_level: full
      requires_approval: true
error_handling:
  on_step_failure: escalate
  escalation_target: engineering-lead
```

Trigger workflows with `@flow_name` syntax in GitClaw.

## Hooks (lifecycle control)

### Script-based hooks (`hooks/hooks.yaml`)

```yaml
on_session_start:
  - script: hooks/scripts/init.sh
    timeout: 10
    description: "Initialize session"
    fail_open: true           # continue on failure

pre_tool_use:
  - script: hooks/scripts/guard.sh
    timeout: 10
    description: "Block dangerous commands"
    fail_open: false          # halt on failure

post_tool_failure:
  - script: hooks/scripts/on-fail.sh

post_response:
  - script: hooks/scripts/log.sh

pre_query:
  - script: hooks/scripts/pre-query.sh

file_changed:
  - script: hooks/scripts/on-change.sh

on_error:
  - script: hooks/scripts/on-error.sh
```

Hook scripts receive JSON via stdin, return `{ action: "allow" | "block" | "modify", reason?, args? }`.

### SDK programmatic hooks

```typescript
query({
  hooks: {
    sessionStart: async (ctx) => { console.log("Session started"); },
    preToolUse: async (ctx) => {
      if (ctx.toolName === "cli" && ctx.args.command?.includes("rm -rf"))
        return { action: "block", reason: "Destructive command blocked" };
      return { action: "allow" };
    },
    postToolFailure: async (ctx) => { /* handle */ },
    postResponse: async (ctx) => { /* log response */ },
    preQuery: async (ctx) => { /* pre-process */ },
    fileChanged: async (ctx) => { /* react to changes */ },
    onError: async (ctx) => { /* handle errors */ },
  },
})
```

## Memory

Memory persists across sessions via git commits.

```
memory/
├── MEMORY.md           # working memory (200-line max, auto-included in prompt)
└── runtime/
    ├── dailylog.md     # daily activity log
    ├── context.md      # session context
    └── key-decisions.md
```

Optional `memory/memory.yaml` for config:
```yaml
retention: 30d
embedding: true
retrieval: semantic
```

GitClaw auto-commits memory changes to git after each session.

## Knowledge

Reference documents the agent can consult. Define in `knowledge/index.yaml`:

```yaml
entries:
  - path: docs/architecture.md
    tags: [architecture, design]
    priority: high
    always_load: true       # injected into system prompt
  - path: docs/api-reference.md
    tags: [api]
    priority: medium        # available on-demand via read tool
```

## Few-shot examples

Put example interactions in `examples/` as markdown files. They're loaded alphabetically and injected into the system prompt as `<example>` blocks for in-context learning.

---

# PART 6: COMPLIANCE & AUDIT

## Compliance config in agent.yaml

```yaml
compliance:
  risk_level: high               # low, medium, high, critical
  human_in_the_loop: always      # always, conditional, advisory, none

  supervision:
    designated_supervisor: "compliance-officer"
    review_cadence: weekly
    escalation_triggers:
      confidence_threshold: 0.6
      action_types: [financial, regulatory]
      error_patterns: ["compliance*"]
    override_capability: true
    kill_switch: true

  recordkeeping:
    audit_logging: true
    log_format: structured_json
    retention_period: "7y"
    log_contents: [decisions, tool_calls, reasoning]
    immutable: true

  model_risk:                     # SR 11-7
    inventory_id: "MRM-2024-001"
    validation_cadence: quarterly
    validation_type: independent
    conceptual_soundness: true
    ongoing_monitoring: true
    outcomes_analysis: true
    drift_detection: true

  data_governance:
    pii_handling: redact          # redact, encrypt, prohibit, allow
    data_classification: confidential
    consent_required: true
    cross_border: false
    bias_testing: true

  communications:                 # FINRA 2210
    type: retail
    pre_review_required: true
    fair_balanced: true
    no_misleading: true
    disclosures_required: true

  segregation_of_duties:          # see DUTIES.md section
    roles: [...]
    conflicts: [...]

  vendor_management:              # SR 23-4
    due_diligence_complete: true
    soc_report_required: true
```

## Supported regulatory frameworks

FINRA (3110, 4511, 2210, 2010, 2111, 3120, 4370), Federal Reserve (SR 11-7, SR 23-4, SR 21-8), SEC (Reg S-P, Rule 17a-4), CFPB (Circular 2022-03), OCC, FDIC, BSA/AML, EU AI Act, UK FCA, GDPR, MiCA

## Compliance artifacts

```
compliance/
├── risk-assessment.md
├── regulatory-map.yaml
└── validation-schedule.yaml
```

## Commands

```bash
gitagent validate --compliance    # validate compliance rules
gitagent audit                    # generate full compliance report
```

---

# PART 7: SUB-AGENTS

## Directory format (full agent)

```
agents/
└── reviewer/
    ├── agent.yaml
    ├── SOUL.md
    ├── skills/
    └── ...
```

## File format (lightweight)

```
agents/reviewer.md
```

With YAML frontmatter:
```markdown
---
name: reviewer
description: Reviews code for quality
model: anthropic:claude-sonnet-4-6
---

You are a code reviewer. Focus on correctness, security, and readability.
```

## Delegation config in parent agent.yaml

```yaml
agents:
  reviewer:
    delegation:
      mode: auto
      triggers: ["review", "check code", "audit"]
```

---

# PART 8: PLUGINS

Plugins extend agents with tools, hooks, skills, and prompt content.

## Plugin locations

1. `<agent>/plugins/<name>/` (local)
2. `~/.gitclaw/plugins/<name>/` (global)
3. `<agent>/.gitagent/plugins/<name>/` (installed from remote)

## Plugin manifest (`plugin.yaml`)

```yaml
id: my-plugin
name: My Plugin
version: 1.0.0
description: "What this plugin does"
author: "name"
license: "MIT"
provides:
  tools: true
  hooks:
    on_session_start: [{ script: "scripts/init.sh", description: "..." }]
    pre_tool_use: [{ script: "scripts/guard.sh" }]
  skills: true
  prompt: "prompt.md"       # extra system prompt content
config:
  properties:
    api_key: { type: string, env: "MY_API_KEY" }
  required: [api_key]
entry: "index.ts"           # programmatic entry point
engine: ">=0.3.0"           # minimum gitclaw version
```

Enable/disable per plugin in agent.yaml:
```yaml
plugins:
  my-plugin:
    enabled: true
    config:
      api_key: "${MY_PLUGIN_KEY}"
```

---

# PART 9: COMPOSIO INTEGRATIONS (500+ external apps)

Connect agents to Gmail, Slack, Google Calendar, Notion, Jira, GitHub, and 500+ more.

## Setup

1. Get API key from https://composio.dev
2. Add to `.env`:
```
COMPOSIO_API_KEY=ak_xxx
COMPOSIO_USER_ID=default
```
3. Connect apps through Composio's OAuth flow
4. GitClaw auto-discovers connected tools

## How it works

- Tools appear as `composio_<toolkit>_<action>` (e.g., `composio_gmail_SEND_EMAIL`)
- Semantic tool matching — GitClaw picks relevant tools per query
- OAuth handled by Composio's `initiateConnection()` with redirect URLs

## SOUL.md tip

```markdown
# Integrations
You are connected to Gmail and Slack via Composio.
- Use Gmail to send emails and read inbox
- Use Slack to post messages to channels
Always confirm with the user before sending external messages.
```

---

# PART 10: MESSAGING INTEGRATIONS

## Telegram

Set in `.env`:
```
TELEGRAM_BOT_TOKEN=xxx
TELEGRAM_ALLOWED_USERS=user1,user2
```

Supports file/photo upload and download (50MB limit).

## WhatsApp

Uses Baileys library with QR authentication. Tools available:
- `send_whatsapp_message` (by name or phone)
- `save_whatsapp_contact`
- `list_whatsapp_contacts`

## Message triggers

Auto-reply patterns for specific contacts/platforms:
- `create_trigger`, `list_triggers`, `delete_trigger`, `toggle_trigger`
- Regex or substring pattern matching
- Approval gates: pause workflows, await yes/no via Telegram/WhatsApp (5-min timeout)

---

# PART 11: VOICE MODE

```bash
gitclaw --dir ./my-agent --voice
```

Opens browser UI at http://localhost:3333 with:
- **OpenAI Realtime** — bidirectional audio streaming, VAD, Whisper transcription
- **Gemini Live** — audio with resampling, default voice "Aoede"
- Video/camera/screen capture support
- File upload handling
- Mood tracking (`memory/mood.md`)
- Photo capture on celebratory language
- Session journaling
- System vitals monitoring

---

# PART 12: SCHEDULES (Cron Jobs)

Define in `schedules/<name>.yaml`:

```yaml
id: daily-report
prompt: "Generate a daily summary report"
cron: "0 9 * * *"           # every day at 9am
mode: repeat                 # or "once"
enabled: true
```

For one-time execution:
```yaml
id: migration-check
prompt: "Verify the database migration"
mode: once
runAt: "2026-04-10T09:00:00Z"
enabled: true
```

Results logged to `.gitagent/schedule-logs/`.

---

# PART 13: SESSIONS & SANDBOX

## Sessions

- Each `--repo` run creates a session branch: `gitclaw/session-<8-char-hex>`
- Resume with `--session <branch>`
- Auto-commits changes, supports push

## Sandbox modes

**E2B sandbox (cloud VM):**
```bash
gitclaw --dir ./my-agent --sandbox "message"
```

**NVIDIA OpenShell (Docker + Landlock):**
- Read-only system dirs, read-write `/sandbox` and `/tmp`
- Network allowlist (default deny)
- GPU support with `--gpu` flag
- Audit or enforce modes

---

# PART 14: ENVIRONMENTS & INHERITANCE

## Environment configs

```
config/
├── default.yaml        # base config (always loaded)
├── staging.yaml        # staging overrides
└── production.yaml     # production overrides
```

```bash
gitclaw --dir ./my-agent --env production "message"
```

Deep-merges environment over default.

## Agent inheritance

```yaml
# child agent.yaml
extends: "https://github.com/org/base-agent.git"
```

Clones parent, deep-merges manifests (child overrides), combines RULES.md from both.

## Dependencies

```bash
gitagent install    # resolves semver, clones dependencies to mount paths
```

---

# PART 15: EXPORTING & IMPORTING

## Export formats (14 targets)

```bash
gitagent export --format <format> --output <file>
```

| Format | Output |
|---|---|
| `system-prompt` | Single markdown for any LLM |
| `claude-code` | `CLAUDE.md` |
| `cursor` | `.mdc` rule files |
| `openai` | Python source for OpenAI Agents SDK |
| `crewai` | YAML config with role/goal/backstory |
| `lyzr` | JSON payload for Lyzr Studio |
| `github` | GitHub Models API payload |
| `copilot` | GitHub Copilot format |
| `codex` | OpenAI Codex format |
| `gemini` | Google Gemini format |
| `openclaw` | OpenClaw workspace |
| `opencode` | OpenCode format |
| `nanobot` | config.json + system-prompt.md |
| `kiro` | Kiro format |

## Import from existing tools

```bash
gitagent import --from claude <path>     # imports CLAUDE.md + .claude/skills/
gitagent import --from cursor <path>     # imports .cursorrules or AGENTS.md
gitagent import --from crewai <path>     # imports CrewAI YAML config
```

---

# PART 16: VALIDATION & OTHER COMMANDS

```bash
gitagent validate                  # validate spec structure
gitagent validate --compliance     # + compliance rules
gitagent info                      # display agent summary
gitagent audit                     # generate compliance report

# Lyzr Studio integration
gitagent lyzr create               # create agent on Lyzr Studio
gitagent lyzr update               # push updates
gitagent lyzr info                 # show linked agent ID
gitagent lyzr run --prompt "msg"   # clone + create + chat
```

---

# PART 17: AVAILABLE MODELS

- `openai:gpt-4o-mini`, `openai:gpt-4o`, `openai:o3`
- `anthropic:claude-sonnet-4-6`, `anthropic:claude-opus-4-6`
- `google:gemini-pro`, `google:gemini-2.0-flash`
- `groq:llama-3.3-70b-versatile`
- `mistral:mistral-large-latest`
- Custom endpoints via `@baseUrl` syntax or env vars
- Local models via Ollama (through OpenShell GPU passthrough)

---

# PART 18: GIT-NATIVE PATTERNS

- **Agent versioning** — git tags = agent versions (semver)
- **Branch-based deployment** — dev → staging → main
- **Human-in-the-loop** — agents open PRs for review before merging skills/memory
- **Agent forking** — fork a repo = fork an agent
- **Agent diff/audit** — `git diff` shows exactly what changed
- **CI/CD for agents** — GitHub Actions for validation, export, deployment
- **Secret management** — env vars, never committed

---

# KEY PRINCIPLES

- Agent = folder of markdown files, not code
- Only `agent.yaml` + `SOUL.md` are required, everything else is optional
- Change behavior by editing a markdown file — no redeploy needed
- Memory is persisted via git commits automatically
- Skills are loaded dynamically at runtime
- Write once (GitAgent spec) → export to 14+ runtimes
- Compliance is built-in, not bolted on
