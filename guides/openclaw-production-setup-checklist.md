# OpenClaw Production Setup Checklist

**Based on official recommendations from Corvis (OpenClaw founder).**

For clients who want a battle-tested, secure, 24/7 AI assistant on their own server.

---

## ✅ Before You Start

- [ ] VPS provisioned (Hetzner CX31 or better, Ubuntu 24.04)
- [ ] SSH key-only access configured (`PasswordAuthentication no`)
- [ ] Firewall active (`ufw default deny incoming`, allow SSH)
- [ ] Fail2Ban installed and running
- [ ] Tailscale installed and authenticated (or Cloudflare Tunnel ready)
- [ ] Node.js 22 installed via nvm
- [ ] OpenClaw CLI installed globally (`npm install -g openclaw`)

---

## 1. Install & Configure OpenClaw

- [ ] Run `openclaw setup` and complete wizard
  - [ ] Choose AI provider (Anthropic recommended)
  - [ ] Enter API key (one key per client, never share)
  - [ ] Select model (Claude Sonnet 4)
  - [ ] Gateway mode: **Loopback** (localhost only)
  - [ ] Select at least 1 channel (Telegram first, easiest)

---

## 2. Security Hardening

- [ ] Gateway listening only on `127.0.0.1:18789` (default, verify)
- [ ] No public port forwarding to 18789 (use Tailscale or Cloudflare Tunnel for remote access)
- [ ] SSH key-only auth, root login disabled
- [ ] Fail2Ban enabled for ssh/nginx/gateway
- [ ] `maxPingPongTurns: 1` set in config (prevents agent infinite loops)
- [ ] Automatic updates configured (`unattended-upgrades`)
- [ ] Log rotation configured for `~/.openclaw/logs/`

---

## 3. Channel Configuration

### Telegram (Easiest — start here)
- [ ] Create bot via @BotFather → get token
- [ ] In OpenClaw config:
  ```yaml
  channels:
    telegram:
      token: "YOUR_TOKEN"
      allowFrom: [YOUR_USER_ID]  # numeric ID, not @username
  ```
- [ ] Test: DM your bot, should respond

### Discord
- [ ] Create application in Discord Developer Portal
- [ ] Enable "Presence Intent", "Server Members Intent", "Message Content Intent"
- [ ] Generate bot token
- [ ] Create OAuth2 invite URL with `bot` + `applications.commands` scopes
- [ ] Invite bot to server (needs "Send Messages", "Read Message History" permissions)
- [ ] Config:
  ```yaml
  channels:
    discord:
      token: "YOUR_TOKEN"
      guildId: "YOUR_GUILD_ID"
      channelIds: ["CHANNEL_ID"]  # specific channels or leave for all
  ```

### Gmail (App Password wins)
- [ ] Enable 2FA on Gmail account
- [ ] Generate App Password (16-char)
- [ ] Config:
  ```yaml
  channels:
    gmail:
      email: "seth@runyourownai.com"
      password: "APP_PASSWORD"
      watch: ["INBOX"]
  ```
- [ ] Test: send email to monitored inbox, OpenClaw should process

### WhatsApp (via Baileys)
- [ ] Install `@whiskeysockets/baileys` skill (from ClawHub)
- [ ] QR scan to link WhatsApp Web session
- [ ] No Business API or Twilio needed

---

## 4. Essential Skills (Bundled)

These 13 skills come with OpenClaw:

- `weather` — get weather for any location
- `healthcheck` — system health monitoring
- `himalaya` — email reading/sending
- `coding-agent` — code generation & review
- `goplaces` — Google Maps directions/places
- `session-logs` — view recent agent conversations
- `summarize` — text summarization
- `time` — current time/timezone
- `echo` — simple reply (testing)
- `logger` — custom event logging
- `web-search` — web search (Perplexity)
- `youtube-search` — find YouTube videos
- `basic` — calculator, conversions, etc.

**Install essential ones:**
```bash
openclaw skill install weather healthcheck himalaya coding-agent goplaces session-logs summarize web-search youtube-search
```

Use `clawhub search` to discover more.

---

## 5. AI Provider Management

- [ ] **One API key per client** (never share keys between clients)
- [ ] Keys stored in `~/.openclaw/secrets/` (auto-created)
- [ ] Hot-swap: Change key in config, no gateway restart needed
- [ ] Monitor usage: `openclaw usage --days 7` (if available) or check provider dashboard
- [ ] **No built-in budget caps** — set up manual alerts via cron:
  ```bash
  # Example: check Anthropic usage daily, email if >$15
  0 9 * * * curl -s "ANTHROPIC_USAGE_URL" | jq '.daily > 15' && mail -s "High Anthropic Usage" you@example.com
  ```

---

## 6. Backups

**What to back up:**
- `~/.openclaw/openclaw.json` — main config
- `~/.openclaw/secrets/` — API keys, tokens
- `~/.openclaw/workspace/` — agent workspaces, skills data
- Optional: `~/.openclaw/logs/` if you want history

**How:**
```bash
# Daily backup via rclone to Backblaze B2 (cheap)
rclone sync ~/.openclaw/ b2:openclaw-backups/$(hostname)-$(date +%Y-%m-%d) --progress
```

**Schedule:** Daily at 2 AM via cron.
**Test restore quarterly.**

---

## 7. SSL & Reverse Proxy (Only if needed)

**You don't need SSL** if:
- Using only Telegram/Discord/Gmail channels
- Gateway runs on `localhost` behind Tailscale

**You DO need SSL if:**
- You want a web dashboard accessible from browser
- You're exposing the gateway publicly (not recommended)

**Minimal Caddy config (3 lines):**
```caddyfile
runyourownai.com {
    reverse_proxy localhost:18789
}
```

Or use **Cloudflare Tunnel** (recommended over public SSL):
```bash
cloudflared tunnel create openclaw
cloudflared tunnel route dns openclaw openclaw.yourdomain.com
cloudflared tunnel run openclaw
```

---

## 8. Monitoring & Alerts

**Prefer systemd over PM2** for production:

```bash
# Create systemd service
sudo nano /etc/systemd/system/openclaw.service
```

```ini
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
User=openclaw
WorkingDirectory=/home/openclaw
ExecStart=/usr/bin/openclaw gateway start
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable openclaw
sudo systemctl start openclaw
```

**Health checks:**
```bash
# Full system probe
openclaw status --deep

# Simple check
curl http://localhost:18789/health || echo "DOWN"
```

**Heartbeat cron (alert if down):**
```bash
# Check every 5 minutes, email if fails
*/5 * * * * curl -s http://localhost:18789/health > /dev/null || echo "OpenClaw is down" | mail -s "ALERT" you@example.com
```

---

## 9. CLI Tools to Pre-Install

On every client VPS:

```bash
# Already done
npm install -g openclaw

# Useful extras
sudo apt install -y jq yq htop iotop curl bc
```

**Optional one-shot bootstrap script:**
```bash
#!/bin/bash
# setup-client-vps.sh
apt update && apt install -y jq yq htop iotop curl ufw fail2ban
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc
nvm install 22
nvm alias default 22
npm install -g openclaw
# Add user, harden SSH, etc.
```

---

## 10. Skills Management

- **No auto-updates** — disable automatic skill updates in production
- Update manually after testing: `openclaw skill update <skill-name>`
- **Audit skill permissions** before installing:
  - `shell`, `browser_unrestricted`, `file_write` outside `workspace/` = DANGEROUS
  - Only install skills from trusted authors (official, OpenClaw team)
- **Block dangerous skills:** Document which ones clients should never install

---

## 11. Multi-Agent Setup

**Recommendation:** Single OpenClaw instance can handle **13+ agents** on 8GB VPS. No need to split.

**Isolation:**
- Separate API keys for each client environment (already done)
- Agents share skills pool (that's fine)
- Use `openclaw agent create` to add new agents
- Per-agent config in `~/.openclaw/agents/<agent-id>/`

**Scaling threshold:**
- 8GB CX31: up to 15 agents comfortable
- 16GB CX41: up to 30+ agents
- Switch to PostgreSQL only if >50 agents or heavy memory usage

---

## 12. Production Readiness Checklist

Your OpenClaw is production-ready when:

- [ ] Gateway bound to localhost only (no public port exposure)
- [ ] At least 1 channel tested and working (Telegram recommended first)
- [ ] AI provider API key configured and responding
- [ ] Security hardened (SSH keys, firewall, fail2ban, unattended-upgrades)
- [ ] Backups scheduled (daily) and tested restore
- [ ] Monitoring configured (systemd + heartbeat cron)
- [ ] Essential skills installed (weather, healthcheck, himalaya, coding-agent, etc.)
- [ ] `maxPingPongTurns: 1` set
- [ ] Separate API keys per client environment
- [ ] Documentation reviewed with client (how to use, troubleshoot, contact support)

---

## 13. Common Gotchas

- **21,000 exposed instances problem:** Never expose port 18789 publicly. Use Tailscale or Cloudflare Tunnel.
- **Infinite agent loops:** `maxPingPongTurns: 1` prevents this.
- **API key sharing:** One key per client, never reuse.
- **Auto-updates:** Don't enable OpenClaw auto-update in production. Test before upgrading.
- **Dangerous skills:** Avoid `shell`, `browser_unrestricted`, `file_write` outside workspace.
- **Log rotation:** Set up logrotate or logs will fill disk.
- **Backup secrets:** If you lose `~/.openclaw/secrets/`, you lose API keys and channel tokens. Backup religiously.

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
sudo systemctl restart openclaw  # if using systemd
# or
pm2 restart openclaw

# 5. Verify
openclaw status --deep
```

**Cron:** Monthly upgrade check, but test in staging first. Quarterly actual upgrade.

---

## 15. Local AI Alternative (Perplexity Desktop)

For clients who want to avoid monthly API costs:

- **Perplexity Desktop** is a self-hosted AI assistant for local machines
- Requirements: 16GB RAM minimum, GPU optional (accelerates)
- Privacy: 100% local, no cloud API calls
- Cost: Hardware upfront, $0/mo ongoing
- Complexity: Higher than VPS setup (driver issues, GPU setup)
- **Not recommended** for non-technical clients — VPS + OpenClaw is simpler

If client wants local:
- Recommend 32GB RAM + RTX 4090 for best performance
- Or 16GB MacBook Pro M2 (works but slower)
- Still need to install OpenClaw locally and manage updates

---

## Resources

- OpenClaw Discord: https://discord.gg/clawd
- ClawHub (skills): https://clawhub.com
- Official docs: https://docs.openclaw.com

---

**Last updated:** 2026-03-25
**Maintainer:** Kito (RunYourOwnAI.com)
