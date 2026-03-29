# OpenClaw Production Setup Checklist

**Native VPS pattern — the real production SOP.**

> **This checklist targets the native OpenClaw VPS pattern:**
> - Dedicated `openclaw` user (never root)
> - Tailscale-only admin access
> - Loopback-bound gateway (`127.0.0.1:18789`)
> - Telegram-first operation with pairing
> - Dashboard access via SSH tunnel only
> - User-level systemd service management
> - Explicit memory and session hygiene
>
> If you're running Docker/Compose or a local install, this checklist does not apply directly.

---

## ✅ Before You Start

- [ ] VPS provisioned (Hetzner CX32 or better, Ubuntu 24.04)
- [ ] Dedicated `openclaw` user created (never run as root)
- [ ] Tailscale installed and verified
- [ ] Confirm `ssh openclaw@TAILSCALE_IP` works before disabling password auth or locking down firewall rules
- [ ] SSH/firewall hardened only after Tailscale is confirmed working
- [ ] Firewall active (`ufw default deny incoming`, allow SSH on `tailscale0` only)
- [ ] Fail2Ban installed and running
- [ ] Node.js 22 installed via nvm
- [ ] OpenClaw CLI installed globally (`npm install -g openclaw`)

> ⚠️ **Security ordering matters:** Install Tailscale and verify SSH access via Tailscale IP **before** hardening SSH/firewall. Otherwise you can lock yourself out.

---

## 1. Install & Configure OpenClaw

- [ ] Run the guided onboarding command for the installed version
  - Prefer `openclaw onboard`
  - If unavailable, try `openclaw setup`
  - Check `openclaw --help` for the current guided setup command
- [ ] Choose AI provider (Anthropic recommended)
- [ ] Enter API key (one key per client, never share)
- [ ] Choose a stable Claude model supported by the installed version
- [ ] Gateway mode: **Loopback** (localhost only)
- [ ] Select at least 1 channel (Telegram first, easiest)

**Verify provider/model validity after setup:**

```bash
openclaw doctor
openclaw models status --probe
```

> Do not hardcode a specific model string unless you have verified it against the installed version. Model names and availability change between releases.

---

## 2. Security Hardening

**Network & access:**

- [ ] Gateway listening only on `127.0.0.1:18789` (default, verify)
- [ ] No public port forwarding to 18789
- [ ] Tailscale IP is the normal admin/SSH path
- [ ] Dashboard should not be publicly exposed (use SSH tunnel — see Dashboard Architecture below)
- [ ] SSH key-only auth, root login disabled
- [ ] Fail2Ban enabled for SSH
- [ ] Automatic updates configured (`unattended-upgrades`)
- [ ] Gateway auth token enabled (even with Tailscale)

**File permissions:**

- [ ] `~/.openclaw` permissions locked (`chmod 700`)
- [ ] `~/.openclaw/openclaw.json` permissions locked (`chmod 600`)
- [ ] `~/.openclaw/secrets/` permissions locked (`chmod 700`, files `chmod 600`)

**Operational behavior:**

- [ ] `maxPingPongTurns: 1` set in config (prevents agent-to-agent infinite loops — this is behavioral, not network hardening, but important for cost and stability)

---

## 3. Channel Configuration

### Telegram (Primary — start here)

**Step 1: Inspect your existing config before editing anything.**

The onboarding wizard may have already created a Telegram block. Do not blindly paste a schema from a guide.

```bash
python3 - <<'PY'
import json, os, pprint
p=os.path.expanduser("~/.openclaw/openclaw.json")
cfg=json.load(open(p))
print("Telegram config:")
pprint.pp(cfg.get("channels", {}).get("telegram"))
PY
```

- If the output shows an existing Telegram config, **work within that structure** — add your bot token and set `dmPolicy` where the existing shape expects them.
- If the output is `None`, you need to add Telegram config manually (see Step 2).

**Step 2: Create your bot and add the token.**

- [ ] Create bot via @BotFather → get token

Follow the structure already present instead of blindly replacing the Telegram block.

For production, prefer the account-based Telegram structure, even if you are starting with only one bot. This keeps single-bot and multi-bot setups consistent and makes later expansion safer:

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "YOUR_TOKEN",
          dmPolicy: "pairing"  // safest — requires approval on first message
        }
      }
    }
  }
}
```

**Step 3: Restart and pair.**

- [ ] `openclaw gateway restart`
- [ ] Send a DM to your bot in Telegram
- [ ] Approve the pairing:
  ```bash
  openclaw pairing approve telegram <PAIRING_CODE>
  ```
- [ ] If using groups: set `groupPolicy` and allowlist (separate from DM pairing)

**Telegram rules to understand:**

| Rule | Detail |
|------|--------|
| Pairing is per bot/account | Being paired for `main` does not mean you're paired for `travel` |
| `dmPolicy: "pairing"` is safest | Requires explicit approval before the bot responds to a new user |
| `groupPolicy` is separate from DM pairing | A user can be paired for DMs but still blocked in groups |
| One Telegram token per bot/account | Do not reuse the same token across multiple live OpenClaw instances — updates get consumed unpredictably |
| Multi-bot setups use accounts + bindings | See Multi-Agent Setup below |

### Discord / Gmail / WhatsApp (Optional — verify against official docs)

These channels are version-dependent.  Before configuring, check the official OpenClaw documentation for current config schema and supported options.

- **Discord:** Requires bot application, Message Content Intent, and guild/channel IDs. Verify config shape against `openclaw --help` or official docs.
- **Gmail:** Requires App Password (2FA must be enabled). Config shape may differ between versions.
- **WhatsApp:** Via Baileys integration. QR scan to link. More complex setup — verify current skill/plugin availability.

> Telegram should remain the primary documented channel. Other channels are supplemental and may not match the patterns described here.

---

## 4. Skills Management

**Recommended approach — not a fixed bundled set:**

- [ ] Start with a minimal skill set
- [ ] Audit permissions before installing any skill:
  - `shell`, `browser_unrestricted`, `file_write` outside `workspace/` = **dangerous**
- [ ] Prefer official/trusted skills from ClawHub
- [ ] Skills are part of the attack surface — treat new installs like new dependencies
- [ ] Evaluate recommended starter skills: weather, healthcheck, web-search, coding-agent, time, summarize

```bash
# Install after reviewing what each does
openclaw skill install <skill-name>

# Discover available skills
clawhub search
```

**Do not auto-update skills in production.** Update manually after testing:

```bash
openclaw skill update <skill-name>
```

---

## 5. AI Provider Management

- [ ] **One API key per client** (never share keys between clients)
- [ ] Keys stored in `~/.openclaw/secrets/` (auto-created)
- [ ] Monitor usage via provider dashboard or `openclaw usage --days 7` (if available)
- [ ] **No built-in budget caps** — set up manual alerts:
  ```bash
  # Example: daily usage check via cron
  0 9 * * * curl -s "ANTHROPIC_USAGE_URL" | jq '.daily > 15' && mail -s "High Anthropic Usage" you@example.com
  ```

**After any provider or model change, verify and restart:**

```bash
openclaw doctor
openclaw models status --probe
openclaw gateway restart
```

> Do not assume provider/model changes take effect without a gateway restart. Always verify.

---

## 6. Backups

**What to back up:**

- `~/.openclaw/openclaw.json` — main config
- `~/.openclaw/secrets/` — API keys, tokens
- `~/.openclaw/workspace/` — main agent workspace
- `~/.openclaw/workspace-*/` — additional agent workspaces

**Session history (if needed):**

If long-lived bots produce session transcripts you want to keep, archive important `.jsonl` session files from:

- `~/.openclaw/agents/main/sessions/`
- `~/.openclaw/agents/*/sessions/`

> Native install logs live in `/tmp/openclaw/openclaw-*.log` — these are ephemeral and typically do not need backup, but you may want to capture them before a reboot if debugging.

**How:**

```bash
# Daily backup via rclone to Backblaze B2 (cheap)
rclone sync ~/.openclaw/ b2:openclaw-backups/$(hostname)-$(date +%Y-%m-%d) --progress
```

**Schedule:** Daily at 2 AM via cron.
**Test restore quarterly.**

---

## 7. Native Path Layout

Know where things live on a native install:

| Path | What |
|------|------|
| `~/.openclaw/openclaw.json` | Main config (edit this) |
| `~/.openclaw/workspace/` | Main agent workspace (SOUL.md, MEMORY.md, etc.) |
| `~/.openclaw/workspace-*/` | Additional agent workspaces |
| `~/.openclaw/agents/main/sessions/` | Session transcripts (.jsonl files) |
| `~/.openclaw/secrets/` | API keys, tokens |
| `/tmp/openclaw/openclaw-*.log` | Gateway logs (ephemeral) |

---

## 8. Dashboard Architecture

The dashboard runs on the VPS. Your browser is just a control surface — all actual work (tools, memory, provider calls) happens on the VPS.

**Dashboard should be accessed through SSH tunnel only:**

```bash
# From your LOCAL machine:
ssh -N -L 127.0.0.1:28789:127.0.0.1:18789 openclaw@TAILSCALE_IP
```

Then open: http://127.0.0.1:28789/

- Gateway auth token may still be required (check `~/.openclaw/openclaw.json`)
- Keep gateway auth enabled even when accessing via Tailscale tunnel

**Do not publicly expose the dashboard.** If there is a deliberate need for external access, consider in this order:

1. No public exposure (default)
2. Loopback gateway + Tailscale admin
3. Dashboard via SSH tunnel
4. Only then consider public reverse proxy / Cloudflare tunnel

---

## 9. Service Management & Monitoring

OpenClaw runs as a **user-level systemd service** — auto-restarts on crashes, starts on boot:

```bash
# Start/restart
openclaw gateway restart
systemctl --user restart openclaw-gateway.service

# Status
systemctl --user status openclaw-gateway.service --no-pager | head -n 25

# Make it survive logouts
loginctl enable-linger openclaw
```

If `systemctl --user` fails in SSH session:

```bash
export XDG_RUNTIME_DIR="/run/user/$(id -u)"
export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u)/bus"
systemctl --user restart openclaw-gateway.service
```

**Log inspection:**

```bash
LATEST="$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
tail -n 120 "$LATEST" | egrep -i 'telegram|pair|401|auth|error|Unknown model|listening'
```

**Primary health/diagnostic commands:**

```bash
openclaw doctor
openclaw models status --probe
systemctl --user status openclaw-gateway.service --no-pager | head -n 25
```

> **Health-check caution:** `curl http://localhost:18789/health` may work as a simple check, but it is version-dependent. Do not rely on it as your sole monitoring — use `openclaw doctor` and systemd status as primary diagnostics.

**Heartbeat cron (optional):**

```bash
*/5 * * * * curl -s http://localhost:18789/health > /dev/null || echo "OpenClaw may be down" | mail -s "ALERT" you@example.com
```

---

## 10. Memory & Session Hygiene

This is a **critical operational discipline** for long-lived bots.

**Rules:**

- **MEMORY.md should stay short and curated** — it is loaded every single message. Keep it as an active summary, not raw logs.
- Use `/reset` or `/new` in Telegram when context is bloated or when switching topics.
- Archive important large `.jsonl` session files from `~/.openclaw/agents/*/sessions/`
- Do not treat MEMORY.md as a dumping ground for raw research or conversation logs.
- Long research should live in exported notes / markdown files in the workspace, not in the main memory file.

**Recommended memory architecture:**

```
MEMORY.md                     ← Short, curated, active summary only
memory/YYYY-MM-DD.md          ← Daily session logs
memory/archive/               ← Old dailies after summarizing
workspace/research_exports/   ← Long research outputs
workspace/notes/              ← Reference notes
```

**Symptoms of poor hygiene:**

- `Context overflow: prompt too large for the model`
- `MEMORY.md is ... chars (limit ...) truncating`
- No reply from agent (silent context overflow)
- Giant `.jsonl` session files consuming disk

---

## 11. Multi-Agent Setup

**Capacity:**

Multiple agents are possible on an 8GB VPS. Actual capacity depends on model choice, concurrency, active skills, and memory usage. Start small, monitor resource usage, and scale gradually.

```bash
# Monitor resource usage
htop
```

**Isolation:**

- Separate API keys for each client environment
- Each agent gets its own workspace (`~/.openclaw/workspace-<agent>/`)
- Per-agent config in `~/.openclaw/agents/<agent-id>/`
- Use the current agent-creation command for your installed version (e.g. `openclaw agents add <name>` if available)

**Multi-agent routing depends on Telegram accounts + bindings + pairing per bot/account:**

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "TOKEN_FOR_PERSONAL_BOT",
          dmPolicy: "pairing"
        },
        work: {
          botToken: "TOKEN_FOR_WORK_BOT",
          dmPolicy: "pairing"
        }
      }
    }
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "work", match: { channel: "telegram", accountId: "work" } }
  ]
}
```

- Pairing must be approved separately for each bot/account
- If only one bot starts polling, check your `accounts` and `bindings` config
- Be explicit about which account is `default` — the gateway uses this for unmatched messages

---

## 12. Production Readiness Checklist

Your OpenClaw is production-ready when:

- [ ] Gateway bound to localhost only (no public port exposure)
- [ ] At least 1 channel tested and working (Telegram recommended first)
- [ ] AI provider API key configured and responding (`openclaw models status --probe`)
- [ ] Security hardened (SSH keys, firewall, fail2ban, unattended-upgrades)
- [ ] Tailscale SSH tested and working
- [ ] Dashboard tunnel tested (can access via `ssh -N -L ...`)
- [ ] Telegram pairing tested and approved for each bot/account
- [ ] Backups scheduled (daily) and tested restore
- [ ] Monitoring configured (systemd + diagnostics)
- [ ] `maxPingPongTurns: 1` set
- [ ] Separate API keys per client environment
- [ ] Session/archive strategy documented if long-lived bots are expected
- [ ] MEMORY.md hygiene understood and practiced
- [ ] Gateway auth token enabled
- [ ] File permissions locked (`~/.openclaw` 700, config 600)

---

## 13. Common Gotchas

| Problem | Cause | Fix |
|---------|-------|-----|
| `access not configured` in Telegram | Pairing not approved for this bot/account | `openclaw pairing approve telegram <CODE>` |
| `agents.list.X.model: Invalid input` | Malformed model object in config | Use minimal: `{ "primary": "anthropic/claude-opus-4-6" }` |
| Context overflow / giant sessions | MEMORY.md too long, session too long | `/reset`, trim MEMORY.md, archive `.jsonl` files |
| Only one Telegram account starts | Account/binding mismatch | Check `channels.telegram.accounts` and `bindings` |
| Default bot vs secondary account confusion | `default` account not explicit | Be explicit about which account handles unmatched messages |
| Stale Telegram config after updates | Config shape changed between versions | Inspect config (see Python snippet above), fix structure |
| `Failed to connect to bus` | systemd user session missing | Export `XDG_RUNTIME_DIR` and `DBUS_SESSION_BUS_ADDRESS` |
| `HTTP 401: Invalid Authentication` | Wrong/stale API key or auth profile | Verify key, run `openclaw models status --probe`, restart |
| `Unknown model` errors | Invalid model string for installed version | Check `openclaw models status --probe`, fix model config |
| Never expose port 18789 publicly | 21,000+ exposed instances on the internet | Use Tailscale + SSH tunnel only |
| Infinite agent loops | Missing `maxPingPongTurns` setting | Set `maxPingPongTurns: 1` |
| API key sharing | Keys reused across clients | One key per client, always |
| Log rotation not set up | Logs fill disk over time | Set up logrotate for `/tmp/openclaw/` |
| Backup secrets missing | `~/.openclaw/secrets/` not backed up | Backup religiously — losing it means losing all keys/tokens |

---

## 14. Upgrade Process

**Safe upgrade steps:**

```bash
# 1. Backup
tar czf ~/openclaw-backup-$(date +%F).tar.gz ~/.openclaw/

# 2. Upgrade CLI
npm update -g openclaw

# 3. Check config compatibility
openclaw doctor

# 4. Restart
openclaw gateway restart
systemctl --user status openclaw-gateway.service --no-pager | head -n 25

# 5. Verify models
openclaw models status --probe
```

**Post-upgrade log verification:**

```bash
LATEST="$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
tail -n 120 "$LATEST" | egrep -i 'telegram|pair|401|auth|error|Unknown model|listening'
```

**Cron:** Monthly upgrade check, but test in staging first. Quarterly actual upgrade.

---

## 15. CLI Tools to Pre-Install

On every client VPS:

```bash
# Already done
npm install -g openclaw

# Useful extras
sudo apt install -y jq yq htop iotop curl bc
```

---

## Resources

- OpenClaw Discord: https://discord.gg/clawd
- ClawHub (skills): https://clawhub.com
- Official docs: https://docs.openclaw.ai

---

**Last updated:** 2026-03-28
**Maintainer:** Kito (RunYourOwnAI.com)
