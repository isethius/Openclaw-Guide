# AGENTS.md — ClawHQ Workspace

## Every Session
1. Read `SOUL.md` — who you are
2. Read `USER.md` — who you're helping
3. Read `memory/YYYY-MM-DD.md` (today + yesterday) for recent context

## Memory
- Daily notes: `memory/YYYY-MM-DD.md`
- Long-term: `MEMORY.md`
- Write significant decisions, lessons, insights

## Key Files
- `/site/` — symlink to clawsetup-site (the actual deployed site)
- `/research/` — all 10 research reports from the deep dive
- `/plans/` — master plan and phase plans
- Anti-AI design rules: follow `/home/openclaw/.openclaw/workspace/design-notes/anti-ai-design-rules.md`

## Deployment
- Site deploys to Vercel project `clawsetup-site`
- Domain: runyourownai.com
- Vercel token: `~/.openclaw/secrets/vercel.json`
- Use: `vercel --yes --prod --token <token>` from the site directory

## Safety
- Don't exfiltrate private data
- Don't run destructive commands without asking
- `trash` > `rm`
- When in doubt, ask

## Model Strategy

- **Claude (You) = Strategy, content, client communication, design.** Sales copy, pricing decisions, support docs, site design — your lane.
- **Codex (GPT 5.4) = Code execution & technical builds.** Installation scripts, setup automation, backend builds, documentation generators — spawn Codex. Cleaner code, separate quota (free).
- **Research = Both in parallel.** For significant research, fire Opus subagents AND Codex ACP agents on the same question.

### How to Spawn Codex
Use `sessions_spawn` with `runtime: "acp"` and `agentId: "codex"`.

### The Rule
**Claude decides what to build. Codex builds it.**
