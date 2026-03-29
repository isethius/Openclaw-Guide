# 🦞 OpenClaw Setup Guide

Your complete guide to setting up an OpenClaw AI assistant on your own VPS.

> **This guide documents the native OpenClaw VPS pattern** — dedicated `openclaw` user, Tailscale-only admin access, loopback-bound gateway, Telegram-first interaction, and optional dashboard via SSH tunnel. It does not cover Docker/Compose installs.

## What Is This?

OpenClaw is an open-source personal AI assistant that runs on your own server. It connects to your messaging apps (Telegram, Discord, WhatsApp, etc.), has access to tools (files, web search, shell commands), and remembers things between conversations.

Think of it like having a personal ChatGPT that:
- Runs 24/7 on your own VPS
- Connects to YOUR apps and services
- Has persistent memory across conversations
- Can run scheduled tasks via cron and heartbeat while you're away
- You fully control — your data stays on your machine

## 📚 Guides

### Setup
- **[Hetzner VPS Setup Guide](guides/hetzner-setup-combined.md)** — From zero to a running AI assistant in ~30 minutes (Ubuntu 24.04, Tailscale, Telegram)

### After Setup
- **[Post-Setup Guide](guides/post-setup-guide.md)** — Operational knowledge: Telegram architecture, memory hygiene, dashboard access, troubleshooting, and everything we learned running OpenClaw in production
- **[CLI Subscription Auth & Coding Tool Paths](guides/cli-subscription-auth.md)** — Codex CLI, Claude Code CLI, API model inference, and how they differ
- **[Production Setup Checklist](guides/openclaw-production-setup-checklist.md)** — Battle-tested checklist for client deployments

## 🧠 Agent Workspace Files

These are the workspace conventions used in this guide. OpenClaw loads files from `~/.openclaw/workspace/` each session — the specific filenames below are our recommended setup, not hard product requirements:

| File | Purpose |
|------|---------|
| `SOUL.md` | Your agent's personality and values |
| `USER.md` | Info about you (the human) — fill this in together |
| `IDENTITY.md` | Agent's name, creature, role — discovered through conversation |
| `AGENTS.md` | Operating rules and workspace conventions |
| `MEMORY.md` | Long-term memory — curated learnings |
| `TOOLS.md` | Your environment-specific notes (SSH hosts, API keys, etc.) |
| `memory/` | Daily session logs |

### First Session Flow

When your agent wakes up for the first time, it won't have a name or identity. That's by design. Here's what happens:

1. **It reads SOUL.md and USER.md** — gets its personality foundation and learns about you
2. **It introduces itself** — acknowledges it's new and asks what you need
3. **You work together** — through real conversation, not a questionnaire
4. **Identity emerges** — it picks a name, emoji, role that fits
5. **It updates its files** — IDENTITY.md, USER.md get filled in organically

Don't rush this. The best AI assistants develop personality through use, not configuration.

## 🏗️ Building Your Agent Team

Start with one agent. Get it working. Learn how it thinks. Then expand.

### Solo Agent (Start Here)
One agent connected to Telegram. Does everything — research, writing, organizing, answering questions. This is your foundation.

### Adding Specialists (When Ready)
Once your main agent is solid, you can add specialists:
- Each new agent gets its own workspace, personality, and Telegram bot
- Agents can communicate with each other (`sessions_send`, `sessions_spawn`)
- Your main agent can orchestrate the team

**Don't build a team of 10 agents on day one.** Start with one. Add when you feel a real need.

## 💰 Costs

| Item | Cost | Notes |
|------|------|-------|
| Hetzner VPS (CX32, 8GB RAM) | ~$8/month | Relatively fixed |
| AI Provider APIs (Anthropic, OpenAI, etc.) | Variable | Depends on usage, model choice, number of agents |

**Expect VPS cost plus variable provider spend.** Light use might run $18–25/month total. Moderate use with multiple agents and heavier builds can reach $40–60+/month. There is no single fixed total — it depends on your provider choices and how much you use the system.

## 📖 Resources

- **OpenClaw Docs:** https://docs.openclaw.ai
- **OpenClaw GitHub:** https://github.com/openclaw/openclaw
- **Community Discord:** https://discord.com/invite/clawd
- **Find Skills:** https://clawhub.com
