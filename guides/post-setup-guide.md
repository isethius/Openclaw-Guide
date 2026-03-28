# After Setup — What We Learned Running OpenClaw in Production

Everything the main guide doesn't cover. These are lessons from running OpenClaw in production across multiple businesses and 13 agents. Read this after your agent is running.

---

## 🔥 Day 1 Essentials (Do These Immediately)

### 1. Turn On Reasoning

This is the single most impactful setting. Without it, your agent gives surface-level answers that sound confident but miss nuance.

```bash
openclaw config set agents.defaults.thinkingDefault high
openclaw gateway restart
```

We ran for **weeks** without this and couldn't figure out why everything felt shallow. Don't make the same mistake.

### 2. Set Up Model Fallback

When your primary model hits rate limits, you need a backup or your agent goes silent.

```json5
// In openclaw.json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["openai-codex/gpt-5.4"]
      }
    }
  }
}
```

**Important:** If using Codex as fallback, the OAuth token expires every ~10 days. Set up a weekly cron to refresh it, or you'll discover it's dead at the worst possible moment.

### 3. Verify Service Management

OpenClaw runs as a **systemd user service** — auto-restarts on crashes, starts on boot. Confirm yours is running:

```bash
openclaw gateway restart
systemctl --user status openclaw-gateway.service --no-pager | head -n 25
```

If `systemctl --user` fails in your SSH session:
```bash
export XDG_RUNTIME_DIR="/run/user/$(id -u)"
export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u)/bus"
systemctl --user restart openclaw-gateway.service
```

Make sure linger is enabled so the service survives logout:
```bash
loginctl enable-linger openclaw
```

### 4. Lock Down File Permissions

Your config and secrets should be readable only by you:

```bash
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json
chmod 700 ~/.openclaw/secrets/
chmod 600 ~/.openclaw/secrets/*
```

---

## 📂 Where Things Live (Native Install)

Know this layout — you'll need it for debugging:

| Path | What |
|------|------|
| `~/.openclaw/openclaw.json` | Main config (edit this) |
| `~/.openclaw/workspace/` | Main agent workspace (SOUL.md, MEMORY.md, etc.) |
| `~/.openclaw/workspace-travel/` | Other agent workspaces (one per agent) |
| `~/.openclaw/workspace-evenpath/` | Example: EvenPath agent workspace |
| `~/.openclaw/agents/main/sessions/` | Session transcripts (can get huge) |
| `~/.openclaw/secrets/` | API keys, tokens |
| `~/.openclaw/cron/` | Cron job definitions (survives restarts) |
| `~/.openclaw/auth/` | OAuth tokens, auth profiles |
| `/tmp/openclaw/openclaw-*.log` | Gateway logs |

This is the **native user install** layout. Docker/Compose installs use different paths.

---

## 📁 Workspace Files — What They Actually Do

| File | Loaded When | Purpose |
|------|------------|---------|
| `CLAUDE.md` | Every session | Operating instructions for the agent |
| `SOUL.md` | Every session | Personality, values, boundaries |
| `USER.md` | Every session | Info about you (the human) |
| `IDENTITY.md` | Every session | Agent's name, creature, role |
| `AGENTS.md` | Every session | Workspace rules, conventions |
| `MEMORY.md` | Every session | Long-term curated memory |
| `TOOLS.md` | Every session | Environment-specific notes |
| `HEARTBEAT.md` | On heartbeat poll | What to check periodically |
| `BOOTSTRAP.md` | First run only | First-run instructions (delete after) |

**Keep these files lean.** Every token in these files is loaded every single message. A 2,000-line AGENTS.md wastes context on every turn.

**Target guidance:**
- CLAUDE.md: < 200 lines
- SOUL.md: < 50 lines
- USER.md: < 50 lines
- MEMORY.md: Keep short — active summary only (see Memory/Session Hygiene below)
- AGENTS.md: < 100 lines

---

## 📱 Telegram Architecture

### Single-Bot Basics

The main bot works as the default Telegram account. One bot, one agent, one workspace. This is the simplest pattern and where everyone should start.

### Multi-Bot / Multi-Agent

In production, you'll likely run multiple Telegram bots for different agents:
- `main` / `default` — personal assistant
- `travel` — travel planning
- `evenpath` — specialized agent
- `clawhq` — operations

**On a single OpenClaw install, multiple bots are modeled as accounts:**

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "TOKEN_FOR_MAIN_BOT",
          dmPolicy: "pairing"
        },
        travel: {
          botToken: "TOKEN_FOR_TRAVEL_BOT",
          dmPolicy: "pairing"
        }
      }
    }
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", account: "default" } },
    { agentId: "travel", match: { channel: "telegram", account: "travel" } }
  ]
}
```

### Important Telegram Rules

**A. Pairing is per bot/account.**
Being paired for `main` does not mean you're paired for `travel`. If Telegram says "access not configured," you need to approve pairing for that specific bot:

```bash
openclaw pairing approve telegram <PAIRING_CODE>
```

**B. Do not reuse a token across multiple OpenClaw instances.**
If two separate OpenClaw installs use the same bot token, updates get consumed unpredictably — one instance eats the message before the other sees it.

**C. Account/binding structure matters for startup.**
We hit cases where only `travel` started polling but `main` did not. The gateway must be configured to poll all the accounts you need. Check your `accounts` and `bindings` config if an agent doesn't respond.

**D. Default/main account handling.**
If you have multiple accounts but forget to configure a `default`, the gateway may not know which account handles unmatched messages. Be explicit.

**E. Group policy ≠ DM pairing.**
`groupPolicy: "allowlist"` with no allowlist means group messages get silently dropped. That is separate from DM pairing — a user can be paired for DMs but still blocked in groups.

---

## 🖥️ Dashboard Access (SSH Tunnel)

The dashboard shows agent status, sessions, and system health. It runs on the VPS but should **never** be exposed publicly.

### How It Works

- Gateway runs on the VPS, listening on `127.0.0.1:18789`
- Your browser runs locally on your machine
- An SSH tunnel connects local port `28789` to the remote `18789`
- **All actual work still runs on the VPS** — the dashboard is only a control surface

### Access Pattern

From your **local machine**:
```bash
ssh -N -L 127.0.0.1:28789:127.0.0.1:18789 openclaw@TAILSCALE_IP
```

Then open: http://127.0.0.1:28789/

If prompted, use the gateway auth token from `~/.openclaw/openclaw.json`.

### Important

The dashboard does **not** use your local machine for agent work. Tools, memory, workspace, and provider calls all happen on the VPS. The dashboard is purely a viewing/control interface.

Keep gateway auth enabled even when accessing via Tailscale tunnel — it's an additional layer of protection.

---

## 🧠 Memory & Session Hygiene

This is a **critical operational lesson**. Bots can appear completely broken when context overflows.

### Symptoms

- `Context overflow: prompt too large for the model`
- No reply from agent (silently fails)
- `MEMORY.md is ... chars (limit ...) truncating`
- Giant `.jsonl` session files in `~/.openclaw/agents/*/sessions/`

### What Causes It

1. **MEMORY.md gets too long** — OpenClaw injects it every message. If it's huge, context overflows.
2. **A Telegram session grows too large** — long conversation history fills context.
3. **Workspace files are bloated** — AGENTS.md, CLAUDE.md, etc. all load every turn.

### Fixes

**Immediate relief:**
- Use `/reset` or `/new` in Telegram to start a fresh session
- Export/archive giant session files from `~/.openclaw/agents/*/sessions/`

**Ongoing hygiene:**
- Keep MEMORY.md short — use it for active summary only, not raw logs
- Move long research into separate files:
  - `workspace/research_exports/`
  - `workspace/notes/`
  - Archived session exports
- Archive daily memory files older than a week to `memory/archive/`

### The Architecture to Follow

```
MEMORY.md          ← Short, curated, active summary only
memory/YYYY-MM-DD.md  ← Daily session logs
memory/archive/       ← Old dailies after summarizing
workspace/research_exports/  ← Long research outputs
workspace/notes/      ← Reference notes
```

---

## ⚠️ Model & Provider Cautions

### Do Not Casually Edit Model Strings

This was a major source of breakage. Common issues we hit:
- `HTTP 401: Invalid Authentication` — stale auth profile, wrong provider
- `Unknown model: anthropic/claude-sonnet-4-6` — typo or invalid model string
- `agents.list.2.model: Invalid input` — malformed model object in config

### Keep Model Objects Minimal

**Good:**
```json5
{ "primary": "anthropic/claude-opus-4-6" }
```

**Bad (caused errors):**
```json5
{
  "primary": "anthropic/claude-sonnet-4-6",
  "_note": "Temporary downgrade for cost testing"
}
```

Extra fields under the model object cause schema validation failures.

### Always Verify After Model/Provider Changes

```bash
openclaw models status --probe
openclaw doctor
```

Run both after any change to model config, API keys, or auth profiles.

---

## ⏰ Automation — Cron vs Heartbeat

### Heartbeat
Your agent wakes up every N minutes and checks `HEARTBEAT.md` for tasks.

Good for: batching multiple checks together, things that can drift ±15 min.

```json5
// In openclaw.json
{
  agents: {
    defaults: {
      heartbeat: { intervalMinutes: 30 }
    }
  }
}
```

### Cron (Built-in Scheduler)
Jobs persist to disk and survive gateway restarts.

Good for: exact timing, isolated tasks, one-shot reminders.

```bash
# One-shot reminder
openclaw cron add --name "Check email" --at "20m" \
  --session isolated --message "Check inbox for anything urgent" \
  --announce --channel telegram --delete-after-run

# Recurring task
openclaw cron add --name "Weekly cleanup" --cron "0 6 * * 0" \
  --tz "America/Phoenix" --session isolated \
  --message "Consolidate memory files from this week" \
  --announce --channel telegram
```

### ⚠️ The #1 Cron Mistake

**Never set up work in your active session and then restart the gateway.** The restart kills your session and the work never runs.

**Always:** Schedule cron jobs FIRST → verify they appear in `openclaw cron list` → THEN restart.

---

## 🔗 Connecting Your Phone (Tailscale + OpenClaw Companion)

Tailscale creates a private network between your devices. After installing on your VPS:

### On Your Phone
1. Install the Tailscale app (iOS/Android)
2. Log in with the same account you used on the VPS
3. Your phone and VPS are now on the same private network

### Connecting OpenClaw Companion App (if available)
1. Install OpenClaw companion app
2. It will scan for your gateway on the Tailscale network
3. Or manually enter: `http://100.x.y.z:18789` (your VPS Tailscale IP)

### SSH from Phone
Install Termius, Blink, or any SSH app → connect to your Tailscale IP.

---

## 🛡️ Security Checklist

After your first week, verify:

- [ ] SSH password auth disabled (`PasswordAuthentication no` in `/etc/ssh/sshd_config`)
- [ ] UFW firewall active (`sudo ufw status`)
- [ ] Fail2ban running (`sudo systemctl status fail2ban`)
- [ ] Auto-updates enabled (`sudo systemctl status unattended-upgrades`)
- [ ] Tailscale connected (`tailscale status`)
- [ ] `~/.openclaw` permissions locked (`chmod 700 ~/.openclaw`)
- [ ] `~/.openclaw/openclaw.json` permissions locked (`chmod 600`)
- [ ] Secrets permissions locked (`ls -la ~/.openclaw/secrets/`)
- [ ] Not running as root (`whoami` → should say `openclaw`)
- [ ] Gateway auth token set (not running with open access)
- [ ] Gateway auth enabled even with Tailscale tunnel

---

## 💬 Multi-Channel Setup

### Telegram (Recommended Start)
Easiest to set up. One bot per agent via @BotFather.

### Discord (For Communities)
One bot per agent. Each needs Message Content Intent enabled.

### WhatsApp (For Business)
Links via QR code. More complex but great for client-facing work.

Each channel can route to different agents:
```json5
{
  bindings: [
    { agentId: "main", match: { channel: "telegram" } },
    { agentId: "work", match: { channel: "discord" } }
  ]
}
```

---

## 🤖 When to Add More Agents

**Don't add agents until you feel the pain.** Signs you need a specialist:

- Your main agent's context is regularly > 50% full
- You're switching between very different tasks (personal vs business)
- You want different personalities for different channels
- A specific task (SEO, coding, monitoring) needs deep dedicated context

### Adding Your Second Agent
```bash
openclaw agents add work
```

This creates:
- New workspace at `~/.openclaw/workspace-work/`
- New session store at `~/.openclaw/agents/work/`
- Blank SOUL.md, USER.md, IDENTITY.md for the new agent

Then create a Telegram bot for it and add bindings (see Telegram Architecture above).

### Communication Between Agents
```json5
{
  tools: {
    agentToAgent: { enabled: true, allow: ["main", "work"] },
    sessions: { visibility: "all" }
  }
}
```

Set `session.agentToAgent.maxPingPongTurns: 1` — otherwise agents will chat back and forth burning tokens on "thanks!" "you're welcome!" loops.

---

## 💰 Cost Management

### What Burns Tokens
1. **Large workspace files** — loaded every message. Keep them lean.
2. **Heartbeats** — every 30 min = 48 agent turns/day. Use sparingly.
3. **Multi-agent chat loops** — set maxPingPongTurns to 1.
4. **Long conversations** — use `/reset` when switching topics.
5. **Subagent spawns** — each spawn is a full agent turn.

### Realistic Cost Expectations

| Item | Cost | Notes |
|------|------|-------|
| Hetzner CX32 (8GB RAM) | ~$8/month | Relatively fixed |
| AI Provider APIs | Variable | Depends on provider, model, and usage |

**Expect VPS cost plus variable provider spend.** There is no single fixed total.

- **Light use:** ~$18–25/month
- **Moderate use:** ~$30–50/month (multiple agents, daily builds)
- **Heavy use:** ~$60+/month (many agents, heavy build sessions)

### What Burns Tokens Fastest
- Opus: ~$15/million input tokens, $75/million output tokens
- GPT-5.4: included with ChatGPT Pro, or ~$12/million via API
- Typical day (moderate use): $1–3
- Heavy build session: $5–15

### Free/Cheap Alternatives
- OpenRouter free models (Step 3.5 Flash, etc.) for low-priority agents
- Codex CLI (free with ChatGPT Pro subscription) for builds + fallback

---

## 🆘 Troubleshooting

### Real errors you'll actually hit

#### A. `HTTP 401: Invalid Authentication`

**Likely causes:**
- Wrong API key in config
- Stale auth profile or expired OAuth token
- Wrong provider winning (check `~/.openclaw/auth/`)
- Gateway needs restart after key change

**Fix:** Check key in `~/.openclaw/openclaw.json`, verify with `openclaw models status --probe`, restart gateway.

#### B. `access not configured` in Telegram

**Cause:** Telegram pairing not approved for this bot/account.

**Fix:**
```bash
openclaw pairing approve telegram <PAIRING_CODE>
```

Remember: pairing is per bot/account. Being paired for `main` doesn't mean you're paired for `travel`.

#### C. `prompt too large for the model` (Context Overflow)

**Fix:**
- Use `/reset` or `/new` in Telegram
- Archive giant session files from `~/.openclaw/agents/*/sessions/`
- Shorten MEMORY.md — keep it as an active summary only

#### D. `agents.list.X.model: Invalid input`

**Cause:** Malformed agent model object in config (extra fields, wrong structure).

**Fix:** Replace with minimal valid object:
```json5
{ "primary": "anthropic/claude-opus-4-6" }
```

#### E. `Failed to connect to bus`

**Cause:** systemd user session not available in SSH.

**Fix:**
```bash
export XDG_RUNTIME_DIR="/run/user/$(id -u)"
export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u)/bus"
systemctl --user restart openclaw-gateway.service
```

#### F. Bot Not Responding At All

**Possible causes (check in order):**
1. Wrong bot username or token in config
2. Pairing not approved
3. Another instance consuming the same bot token's updates
4. Provider not starting (check `openclaw models status --probe`)
5. Webhook/update conflict

**Debug:**
```bash
LATEST="$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
tail -n 120 "$LATEST" | egrep -i 'telegram|pair|401|auth|error|Unknown model|listening'
```

#### G. Only One Telegram Account Starts

**Cause:** Account/binding/default config mismatch. The gateway polls only the accounts it's configured for.

**Fix:** Check `channels.telegram.accounts` — make sure every bot you need is listed. Check `bindings` — make sure each account is bound to the correct agent. Restart gateway after fixing.

#### H. `openclaw: command not found`

**Cause:** nvm not loaded in this shell session.

**Fix:**
```bash
source ~/.bashrc
nvm use 22
```

---

## 🛠️ Operator Cheat Sheet

Keep these handy:

```bash
# Connect
ssh openclaw@TAILSCALE_IP

# Service management
openclaw gateway restart
systemctl --user restart openclaw-gateway.service
systemctl --user status openclaw-gateway.service --no-pager | head -n 25

# If systemctl --user fails in SSH session
export XDG_RUNTIME_DIR="/run/user/$(id -u)"
export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u)/bus"

# Logs
LATEST="$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
tail -n 120 "$LATEST" | egrep -i 'telegram|pair|401|auth|error|Unknown model|listening'

# Diagnostics
openclaw doctor
openclaw status
openclaw models status --probe

# Telegram pairing
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>

# Cron management
openclaw cron list
openclaw cron add --name "task" --every 6h --message "Do X" --announce

# Dashboard (from local machine)
ssh -N -L 127.0.0.1:28789:127.0.0.1:18789 openclaw@TAILSCALE_IP

# Context reset (in Telegram)
/reset
/new
```

---

## 📂 Recommended Folder Structure (After 1 Month)

```
~/.openclaw/workspace/
├── CLAUDE.md
├── SOUL.md / USER.md / IDENTITY.md / AGENTS.md
├── MEMORY.md
├── TOOLS.md
├── HEARTBEAT.md           ← add when you have recurring checks
├── memory/
│   ├── 2026-03-24.md      ← daily logs
│   ├── 2026-W12.md        ← weekly summaries
│   └── archive/           ← old dailies
├── research/              ← research outputs
├── research_exports/      ← exported research from long sessions
├── notes/                 ← reference notes
├── plans/                 ← project plans
├── scripts/               ← automation scripts
└── work-logs/             ← task completion summaries
```

---

## 🚀 30-Day Milestones

**Week 1:** Agent running, Telegram connected, first real conversations, identity established.

**Week 2:** Memory system working (daily notes accumulating), HEARTBEAT.md added for periodic checks, first cron job set up.

**Week 3:** Skills installed (web search, GitHub, email), agent helping with real work, considering second agent.

**Week 4:** Agent team forming (if needed), automation running, workflow patterns established. You should be spending less time managing the agent and more time working alongside it.
