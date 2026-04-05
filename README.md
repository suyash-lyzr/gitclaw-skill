# GitClaw Agent Skill

A Claude Code skill that teaches Claude how to create, configure, and run AI agents using the GitAgent spec and GitClaw runtime.

## Install

```bash
npx skills add suyash-lyzr/gitclaw-skill --skill gitclaw-agent
```

## What it does

Once installed, Claude Code knows how to:
- Scaffold new AI agents (`gitclaw init`)
- Write `SOUL.md`, `RULES.md`, and skill files
- Run agents with GitClaw (`gitclaw --dir . "message"`)
- Export agents to other runtimes (Cursor, OpenAI, Claude Code, etc.)
- Manage memory, workflows, and the full GitAgent folder structure

## Prerequisites

```bash
npm install -g gitclaw
npm install -g @open-gitagent/gitagent
```

## Usage

After installing, just ask Claude Code things like:
- "Create a code review agent"
- "Scaffold a new GitClaw agent for meeting notes"
- "Add a skill for PR summaries to my agent"
- "Export this agent for Cursor"

Claude will know the right folder structure, file formats, and commands.
