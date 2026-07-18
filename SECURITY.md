# Security Policy

## What this repository is

This repository contains only **markdown instructions** (an [Agent Skill](https://skills.sh)) that teach an AI agent how to use [Agentcard](https://agentcard.sh) through its MCP server and CLI. No code from this repository executes on your machine. The commands the skill instructs an agent to run — `npx skills add`, `npm install -g agent-cards`, `agent-cards setup-mcp` — install and configure Agentcard's published tooling; review them before running, as with any skill.

The skill's safety rules require agents to confirm with the user before any action that spends money, closes a card, or reveals card credentials.

## Reporting a vulnerability

If you find a security issue in this skill's guidance, the `agent-cards` CLI, the MCP server, or any Agentcard surface:

- Email **felipe@agentcard.sh**
- Include steps to reproduce and the surface affected (skill / CLI / MCP / API)

We respond quickly — usually the same day.

## Canonical source

This repository is a read-only mirror published from the Agentcard monorepo. The canonical skill is served with a checksum at:

```
https://www.agentcard.sh/.well-known/agent-skills/index.json
```
