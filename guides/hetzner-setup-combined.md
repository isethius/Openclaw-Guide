# OpenClaw on Hetzner VPS — Secure Telegram-First Setup Guide
## From zero to a running AI assistant in ~45 minutes

> **This guide documents the native OpenClaw VPS pattern**, not the Docker/Compose pattern.

> **Living document warning:** Before implementing, re-check the OpenClaw install, secrets, gateway auth, and provider docs if this guide is more than a few weeks old. OpenClaw is actively developed; specific commands and config keys may have changed.

---

## 📋 Overview

We're setting up an $8/month VPS on Hetzner to run OpenClaw 24/7 with Tailscale for secure admin access and Telegram as the primary interface.

**What you'll need:**
- Credit/debit card (Hetzner charges ~$8/month)
- SSH key for security (we'll generate one)
- Terminal access (Mac/Linux) or PuTTY (Windows)
- Anthropic API key
- Tailscale account

**Total monthly cost:**
- VPS: ~$8 (CX32, 8GB RAM)
- AI APIs (Anthropic/OpenAI): ~$10–15 minimum
- **Total: ~$18–23/month minimum** (heavier use: $40–60+)

---

## Before You Start — Non-Negotiable Rules

- Never run OpenClaw as root
- Never expose ports publicly without auth
- Never leave secrets in world-readable directories
- Never paste API keys, Telegram bot tokens, or gateway auth tokens into chat windows, screenshots, or unencrypted notes
- Always verify access works before locking anything down
- Always test after each major step — don't stack changes and debug later
- Always backup before updating

---

## 🔐 PHASE 0: PREPARATION (5 min)

### Step 1: Create SSH Key (if you don't have one)

**On Mac/Linux:**
```bash
# Check if you already have an SSH key
ls -la ~/.ssh/id_*.pub

# If you see "No such file", create one:
ssh-keygen -t ed25519 -C "you@example.com"
# When prompted for a passphrase, enter a strong one (recommended)
# This protects your key if your local machine is compromised
```

Copy your public key to clipboard:
```bash
cat ~/.ssh/id_ed25519.pub | pbcopy
```

The public key is what you share. The private key (`id_ed25519` without `.pub`) never leaves your machine.

**On Windows (PuTTY):**
- Download PuTTY from putty.org
- Open PuTTYgen → "Generate" → move mouse
- Save public key as `key.pub`
- Save private key as `key.ppk` (keep this safe!)

---

## 🌐 PHASE 1: HETZNER ACCOUNT (5 min)

### Step 2: Sign Up
- URL: https://accounts.hetzner.com/login
- Click **"Sign Up"**
- Use real email, strong password
- Verify email (check inbox, click link)

### Step 3: Add Payment Method
- Go to **Billing** → **Add Payment Method**
- Visa/Mastercard — monthly billing, no prepayment

---

## 🖥️ PHASE 2: CREATE THE VPS (10 min)

### Step 4: Login to Hetzner Cloud Console
- URL: https://console.hetzner.cloud

### Step 5: Create a New Project (optional but clean)
- Click **"Create Project"** → name it `OpenClaw`

### Step 6: Add a Server

1. **Image:** Choose **Ubuntu 24.04 LTS** (most compatible with OpenClaw)

2. **Location:** Pick the closest region with good availability. For US users, **Ashburn** is preferable when available; otherwise choose the nearest practical location.

3. **Type:** **Shared CPU** → **CX32** (4 vCPU, 8GB RAM, 80GB disk) — ~$8/month. Volume type: **Local** (fastest)

4. **SSH Key:** Click **"Add SSH Key"** → paste contents of `~/.ssh/id_ed25519.pub` (Mac/Linux: `cat ~/.ssh/id_ed25519.pub | pbcopy`)

5. **Name:** `openclaw-server` or anything you like

6. **Click "Create & Buy Now"** — wait 30 seconds for provisioning.

---

## 💻 PHASE 3: CONNECT TO YOUR SERVER (5 min)

### Step 7: Get Your Server IP
After provisioning, copy the **Public IPv4** address (e.g., `123.45.67.89`)

### Step 8: Connect via SSH

**Mac/Linux:**
```bash
ssh root@123.45.67.89
# Type "yes" the first time to accept the fingerprint
```

**Windows:** Open PuTTY → Hostname: your IP → Port: 22 → Auth: browse to your `.ppk` → Open

You should see: `root@your-server:~#` — **you're in!**

---

## 🛠️ PHASE 4: CREATE NON-ROOT USER (5 min)

OpenClaw should never run as root. Create a dedicated user:

```bash
# Create user
adduser openclaw
# Follow prompts: strong password, name "OpenClaw", Enter through the rest

# Add to sudo group
usermod -aG sudo openclaw

# Copy SSH key to new user
mkdir -p /home/openclaw/.ssh
cp ~/.ssh/authorized_keys /home/openclaw/.ssh/
chown -R openclaw:openclaw /home/openclaw/.ssh
chmod 700 /home/openclaw/.ssh
chmod 600 /home/openclaw/.ssh/authorized_keys
```

**⚠️ CRITICAL: Test the new user login BEFORE locking anything down:**

Open a **second terminal** (don't close the current one):
```bash
ssh openclaw@123.45.67.89
```

If that works, proceed. If not, fix the SSH key copy before continuing.

---

## 🌍 PHASE 5: INSTALL TAILSCALE FIRST (5 min)

**We install Tailscale before locking down SSH** so you always have a way in.

> ⚠️ **Do not disable password/root SSH or restrict firewall rules until you have confirmed you can log in over the server's Tailscale IP as the `openclaw` user.** Hardening before Tailscale works = locked out.

### Step 9: Install Tailscale

```bash
# As the openclaw user:
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

You'll see:
```
To authenticate, visit: https://login.tailscale.com/start/xxxxx
```

**Open that URL on your computer/phone** → log in with Google/GitHub → authorize.

After that, your server gets a private Tailscale IP like `100.x.y.z`:
```bash
tailscale ip -4
```

### Step 10: Confirm Tailscale SSH works

From your **local machine** (not the VPS):
```bash
ssh openclaw@100.x.y.z
```

If that works, you now have two ways in: public IP and Tailscale IP. Next we'll lock down to Tailscale only.

---

## 🔒 PHASE 6: SECURITY HARDENING (CRITICAL)

Now that Tailscale works, lock everything down.

### Step 11: Restrict SSH to Tailscale only

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow in on tailscale0 to any port 22 proto tcp
sudo ufw enable
```

Type `y` when prompted. SSH is now **only accessible via Tailscale** — invisible to the public internet.

### Step 12: Disable Password Auth & Root Login

```bash
sudo nano /etc/ssh/sshd_config
```

Find and change (or add) these lines:
```
PasswordAuthentication no
PermitRootLogin no
```

Save (Ctrl+X → Y → Enter), then:
```bash
sudo systemctl restart sshd
```

### Step 13: Install Fail2Ban

```bash
sudo apt update
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### Step 14: Enable Automatic Security Updates

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -f noninteractive unattended-upgrades
```

This covers OS packages. **Don't auto-update OpenClaw itself** — breaking changes happen. Update manually and test.

---

## 📦 PHASE 7: INSTALL OPENCLAW (10 min)

### Step 15: Install Node.js v22

```bash
# Install nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# Reload shell
source ~/.bashrc

# Install Node.js 22
nvm install 22
nvm use 22
nvm alias default 22

# Verify
node --version
# Should output: v22.x.x
```

### Step 16: Install OpenClaw

```bash
npm install -g openclaw

# Verify
openclaw --version
```

---

## ⚙️ PHASE 8: RUN SETUP WIZARD

### Step 17: Interactive Setup

```bash
openclaw onboard
```

> **Note:** If `openclaw onboard` isn't available in your version, try `openclaw setup` or check `openclaw --help` for the current guided setup command. If your version supports `--install-daemon`, use `openclaw onboard --install-daemon` to set up the systemd service automatically.

The wizard will ask:

**1. AI Provider:**
- **Anthropic (Claude)** — recommended, best quality
- OpenAI (GPT) — cheaper
- Google (Gemini) — cheapest

**2. API Key:**
- Go to console.anthropic.com (or platform.openai.com, or aistudio.google.com)
- Create account → Generate API key → paste when prompted

**3. Gateway Mode:**
- Choose **Loopback** (localhost only) — safest

**4. Model:**
- Choose a stable Claude model supported by the installed version
- Claude Sonnet 4 (recommended balance of quality + cost)
- Claude Opus 4 (best quality, higher cost)

**5. Channel:**
- Choose **Telegram** (easiest to start)

### Step 18: Validate Directly on the VPS

**Before connecting Telegram, the dashboard, or any other integration — validate the base install works locally on the VPS first.**

```bash
openclaw chat
```

Type a message, confirm you get a response, then `Ctrl+C` to exit.

If this fails, fix it now. Common issues at this stage:
- Wrong API key → re-enter via `openclaw config set`
- Stale shell → `source ~/.bashrc`
- Missing dependency → check `openclaw doctor`

**Do not proceed until `openclaw chat` works.** If the foundation is broken, adding services and Telegram on top just compounds the problem. Doctor output, config errors, and log messages are all visible directly on the VPS — use them here before you add the indirection of Telegram or dashboard.

```bash
openclaw doctor
openclaw models status --probe
```

Fix anything flagged before continuing.

---

## 🔐 PHASE 9: LOCK & MIGRATE SECRETS

**Do this immediately after setup, before starting the service.** The setup wizard stores API keys and tokens directly in `openclaw.json` as plaintext. On a hardened setup, that's not acceptable — anyone who can read the file sees every credential.

### Step 19: Lock File Permissions

```bash
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json
chmod 700 ~/.openclaw/secrets/ 2>/dev/null
chmod 600 ~/.openclaw/secrets/* 2>/dev/null
```

`700` means only the `openclaw` user can read, write, or list files. Everyone else gets "Permission denied."

### Step 20: Migrate Secrets Out of Plaintext Config

OpenClaw supports **SecretRefs** — a mechanism where secrets are stored separately and referenced in config using a structured object instead of plaintext strings. This is the production default, not an optional extra.

**The documented secrets workflow:**

```bash
# 1. Audit — see what's currently exposed in plaintext
openclaw secrets audit --check

# 2. Configure — interactively set up secret storage for each credential
openclaw secrets configure

# 3. Dry run — preview what will change without applying
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run

# 4. Apply — actually migrate the secrets
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json

# 5. Re-audit — confirm plaintext credentials are gone
openclaw secrets audit --check

# 6. Reload — pick up the new config
openclaw secrets reload

# 7. Verify everything still works
openclaw doctor
```

> **Note:** The plan file path (`/tmp/openclaw-secrets-plan.json`) is what current docs show. If `openclaw secrets configure` outputs a different path, use that instead — the command will tell you where the plan was saved.

> **Version note:** If `openclaw secrets` commands aren't available in your installed version, the `chmod` lockdown from Step 19 is your primary protection. Check https://docs.openclaw.ai/gateway/secrets for your version's capabilities.

After migration, manually inspect `~/.openclaw/openclaw.json` and confirm **no plaintext tokens remain**. The config should contain SecretRef objects pointing to your stored secrets, not the secrets themselves.

---

## 🚀 PHASE 10: START OPENCLAW AS A SERVICE

OpenClaw runs as a **systemd user service** — auto-restarts on crashes, starts on boot.

### Step 21: Start the Gateway

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

Make it survive logouts:
```bash
loginctl enable-linger openclaw
```

### Step 22: Check Logs

```bash
LATEST="$(ls -t /tmp/openclaw/openclaw-*.log 2>/dev/null | head -1)"
tail -n 120 "$LATEST" | egrep -i 'telegram|pair|401|auth|error|listening'
```

---

## 🔑 PHASE 11: GATEWAY AUTHENTICATION

Tailscale protects the network layer — only your devices can reach the server. Gateway auth adds a second lock at the application layer. If someone gets onto your tailnet (compromised device, shared account), they still can't control your agent without the auth token. Defense in depth.

### Step 23: Generate and Set the Auth Token

```bash
# Generate a strong random token
openssl rand -hex 32
```

Copy the output and configure it:

```bash
openclaw config set gateway.auth.token "YOUR_TOKEN_HERE"
openclaw gateway restart
```

> **Version note:** If `gateway.auth.token` doesn't work as the config key path, check https://docs.openclaw.ai/gateway/authentication for your installed version's exact syntax. Older versions may use `gateway.authToken`. Run `openclaw config get gateway` after setting to confirm it took.

### Step 24: Verify Auth Is Working (Negative Test)

Configuring the token isn't enough — you need to verify the gateway actually **rejects** unauthenticated access. This is the most important verification step.

```bash
# Confirm config looks right
openclaw config get gateway

# Check status
openclaw status
```

Run an auth-gated probe that your installed version exposes via `openclaw gateway --help`. Confirm it:
1. **Fails without a token** (unauthenticated access is rejected)
2. **Succeeds with the correct token**

If the probe succeeds without a token, the auth config didn't take — re-check the key path, restart the gateway, and re-test.

> **Save the token securely.** You'll need it for dashboard access (Phase 14) and potentially for API integrations. Do not store it in plaintext notes — use a password manager or your SecretRefs storage.

---

## 📱 PHASE 12: CONNECT TELEGRAM

### Step 25: Create a Telegram Bot

1. Open Telegram, search for **@BotFather**
2. Send `/newbot`
3. Choose name: `YourNameAssistant`
4. Choose username: must end in `bot` (e.g., `myopenclawbot`)
5. Copy the **Bot Token** — treat this like a password

### Step 26: Configure Telegram in OpenClaw

**First, check if the setup wizard already created a Telegram config:**

```bash
python3 - <<'PY'
import json, os, pprint
p=os.path.expanduser("~/.openclaw/openclaw.json")
cfg=json.load(open(p))
print("Telegram config:")
pprint.pp(cfg.get("channels", {}).get("telegram"))
PY
```

If a Telegram block already exists, **work within that structure** — add your bot token and set `dmPolicy` where the existing shape expects them.

If no Telegram config exists, add it to `~/.openclaw/openclaw.json`. For production, prefer the account-based structure even for a single bot — it scales cleanly to multi-bot setups later:

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "PASTE_YOUR_BOT_TOKEN",
          dmPolicy: "pairing"  // safest — requires approval on first message
        }
      }
    }
  }
}
```

> **If you completed the SecretRefs migration in Phase 9**, use a SecretRef for the bot token instead of pasting it directly. Check https://docs.openclaw.ai/gateway/secrets for the supported format.

Restart the gateway:
```bash
openclaw gateway restart
```

### Step 27: Approve Your First Message

Send any message to your bot in Telegram. You'll need to approve the pairing:

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <PAIRING_CODE>
```

Now message your bot again — it should respond! 🎉

> **For known users only:** After pairing, you can switch to `dmPolicy: "allowlist"` and add `allowFrom: ["tg:YOUR_USER_ID"]` to skip future pairing.

> **Group messages:** If you set `groupPolicy: "allowlist"` but don't populate the allowlist, group messages will be silently dropped. This is separate from DM pairing — DM pairing controls direct messages, `groupPolicy` controls group messages.

---

## ✅ PHASE 13: VERIFY EVERYTHING

### Step 28: Run Diagnostics

```bash
openclaw doctor
openclaw status
openclaw models status --probe
systemctl --user status openclaw-gateway.service --no-pager | head -n 25
```

Fix anything flagged.

### Step 29: Turn On Reasoning (CRITICAL)

This is the #1 setting people miss. Without it, your agent sounds shallow:

```bash
openclaw config set agents.defaults.thinkingDefault high
openclaw gateway restart
```

### Step 30: Reboot Test

**"Installed" doesn't mean "working after reboot."** This is the most important test.

```bash
sudo reboot
```

Wait a minute, then SSH back in via Tailscale:

```bash
ssh openclaw@YOUR_TAILSCALE_IP
systemctl --user status openclaw-gateway.service --no-pager | head -n 25
```

Send a Telegram message to your bot. If it responds, everything survived the reboot. If the service didn't come back, revisit Phase 10 — check linger, XDG_RUNTIME_DIR, and the systemd service.

### Step 31: Take a Hetzner Snapshot

Capture the fully working state so you can roll back if a future update breaks things:

1. Go to https://console.hetzner.cloud
2. Click your server → **Snapshots** tab
3. Click **Create Snapshot** → name it `post-setup-clean`

Take additional snapshots before major OpenClaw updates or significant config changes.

---

## 🖥️ PHASE 14: DASHBOARD ACCESS (Optional)

The OpenClaw dashboard runs on your VPS but should NOT be exposed publicly. Access it via SSH tunnel:

```bash
# From your LOCAL machine:
ssh -N -L 127.0.0.1:28789:127.0.0.1:18789 openclaw@TAILSCALE_IP
```

Then open: http://127.0.0.1:28789/ in your browser.

If prompted, use the gateway auth token you set in Phase 11.

The dashboard shows agent status, sessions, and system health. The tunnel keeps everything private — the dashboard is only a control surface; all actual work runs on the VPS.

---

## 📂 Where Things Live

| Path | What |
|------|------|
| `~/.openclaw/openclaw.json` | Main config (edit this) |
| `~/.openclaw/workspace/` | Main agent workspace (SOUL.md, MEMORY.md, etc.) |
| `~/.openclaw/workspace-*/` | Other agent workspaces |
| `~/.openclaw/agents/*/sessions/` | Session transcripts |
| `~/.openclaw/secrets/` | API keys, tokens |
| `~/.openclaw/cron/` | Cron job definitions (survives restarts) |
| `~/.openclaw/auth/` | OAuth tokens, auth profiles |
| `/tmp/openclaw/openclaw-*.log` | Gateway logs |

---

## 🤖 Multi-Bot / Multi-Agent Setup

If you want multiple bots (e.g., personal assistant + work agent):

### Rules:
- **One Telegram bot token per bot/account** — do not reuse a token across multiple OpenClaw instances if both need to receive updates reliably (updates get consumed unpredictably)
- On a single OpenClaw install, model multiple bots as accounts using `channels.telegram.accounts.*`
- Use `bindings` to route each bot to the correct agent
- Pairing must be approved separately for each bot/account

### Example config with two bots:
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

---

## 🔒 Optional Security Upgrades

These are not part of the critical path but significantly improve your security posture.

### SSH Hardening Extras

The base guide disables password auth and root login. These additional settings further reduce attack surface:

```bash
# Whitelist only the openclaw user — no other user can SSH in
echo "AllowUsers openclaw" | sudo tee -a /etc/ssh/sshd_config

# Disable SSH agent forwarding (prevents exploitation if someone gets a foothold)
echo "AllowAgentForwarding no" | sudo tee -a /etc/ssh/sshd_config

# Disable X11 forwarding (you're running a headless server, not a desktop)
echo "X11Forwarding no" | sudo tee -a /etc/ssh/sshd_config

sudo systemctl restart sshd
```

**What each does:**
- `AllowUsers openclaw` — even if other users exist on the system, only `openclaw` can SSH in
- `AllowAgentForwarding no` — prevents SSH agent forwarding, which could be exploited
- `X11Forwarding no` — disables graphical session forwarding (unnecessary on a headless server)

> **Note on SSH tunneling:** You may see guides recommend `PermitTunnel no` and `AllowTcpForwarding no` for further hardening. Be aware that the dashboard SSH tunnel (`ssh -N -L ...` from Phase 14) uses TCP port forwarding. If you set `AllowTcpForwarding no`, the dashboard tunnel will silently stop working. Only disable TCP forwarding if you plan to access the dashboard via direct Tailscale IP instead (see Phase 14).

### Tailscale ACL Lockdown

By default, every device on your Tailscale account can reach every other device on every port. That's fine initially, but you should tighten it.

Go to https://login.tailscale.com/admin/acls and replace the default policy:

```json
{
  "acls": [
    {
      "action": "accept",
      "src":    ["autogroup:owner"],
      "dst":    ["tag:server:22"]
    }
  ],
  "tagOwners": {
    "tag:server": ["autogroup:owner"]
  }
}
```

This says: only devices owned by the account owner can reach devices tagged `server`, and only on port 22 (SSH). Everything else is denied by default.

After saving, tag your VPS in the Tailscale admin console as `tag:server`.

> **Grants alternative:** Tailscale now recommends **grants** as the modern policy syntax. Check https://tailscale.com/kb/1324/acl-grants for details. The ACL example above continues to work — use whichever your tailnet is already set up with.

Check https://tailscale.com/kb/1018/acls for more patterns. The key principle: deny everything by default, allow only what you need.

---

## 🆘 TROUBLESHOOTING

### Real errors you'll actually hit:

| Error | Cause | Fix |
|-------|-------|-----|
| `HTTP 401: Invalid Authentication` | Wrong/stale API key | Check key in config, restart gateway |
| `access not configured` in Telegram | Pairing not approved | `openclaw pairing approve telegram <CODE>` |
| `prompt too large for the model` | Context overflow | `/reset` in chat, trim MEMORY.md |
| `agents.list.X.model: Invalid input` | Bad model config format | Use `"primary": "anthropic/claude-opus-4-6"` |
| `Failed to connect to bus` | systemd user session missing | Export XDG_RUNTIME_DIR (see Phase 10) |
| `openclaw: command not found` | nvm not loaded | `source ~/.bashrc` then `nvm use 22` |
| Bot doesn't respond | Multiple possible | Check logs (Phase 10), verify pairing, check `openclaw doctor` |
| Telegram says "access not configured" | dmPolicy blocks you | Approve pairing or add your user ID to allowFrom |
| Only one Telegram account starts | Account/binding mismatch | Check `channels.telegram.accounts` and `bindings` config |
| Service doesn't survive reboot | Linger not enabled or XDG issue | `loginctl enable-linger openclaw`, set XDG vars |
| Gateway accepts unauthenticated requests | Auth config didn't take | Re-check key path, restart, re-test (Phase 11) |

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

## 🧠 Keep It Healthy

- **Keep workspace files lean** — every token loads every message
- **Use `/reset`** when switching topics or context fills up
- **Archive old memory files** — move daily notes older than a week to `memory/archive/`
- **MEMORY.md under 200 lines** — this is curated wisdom, not raw logs
- **Set `thinkingDefault: "high"`** — without this, everything is shallow
- Customize your agent's personality through `SOUL.md`, `IDENTITY.md`, and `USER.md` in the workspace

---

## 💰 Ongoing Costs

| Item | Cost | Notes |
|------|------|-------|
| Hetzner CX32 | ~$8/month | Billed hourly |
| Anthropic (Claude) | ~$10–20/month | Depends on usage |
| OpenAI (optional fallback) | ~$0–20/month | Free with ChatGPT Pro for Codex |
| **Minimum total** | **~$18/month** | Light use |
| **Typical total** | **~$30–50/month** | Moderate use |
| **Heavy use** | **~$60+/month** | Multiple agents, lots of builds |

---

## 🔄 Update Procedure

**Never just update and hope.** Back up first, update, verify.

```bash
# 1. Backup
tar -czf ~/openclaw-backup-$(date +%F).tar.gz ~/.openclaw/

# 2. Take a Hetzner snapshot (Console → Snapshots → Create)

# 3. Upgrade CLI
npm update -g openclaw

# 4. Check config compatibility
openclaw doctor

# 5. Restart
openclaw gateway restart
systemctl --user status openclaw-gateway.service --no-pager | head -n 25

# 6. Verify models
openclaw models status --probe

# 7. Check logs
LATEST="$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
tail -n 120 "$LATEST" | egrep -i 'telegram|pair|401|auth|error|Unknown model|listening'
```

If something broke, restore from your backup or rebuild from the Hetzner snapshot.

---

## 📞 Need Help?

- OpenClaw Discord: https://discord.gg/clawd (#users-helping-users)
- OpenClaw Docs: https://docs.openclaw.ai
- Run `openclaw doctor` first — it catches most issues

---

## ⏱️ Quick Recap

1. Sign up for Hetzner → create CX32 Ubuntu 24.04 server
2. SSH in as root → create `openclaw` user → test login
3. Install Tailscale → confirm SSH via Tailscale IP works
4. Lock down: UFW (Tailscale-only SSH), disable password auth, fail2ban
5. Install nvm → Node.js 22 → OpenClaw CLI
6. Run `openclaw onboard` → pick provider → paste API key → loopback mode
7. **Validate locally: `openclaw chat` → `openclaw doctor`**
8. **Lock secrets → migrate with SecretRefs**
9. Start gateway: `openclaw gateway restart` → verify systemd service
10. **Set up gateway auth → negative test to confirm rejection**
11. Set up Telegram: @BotFather → token → config → approve pairing
12. Turn on reasoning: `thinkingDefault: "high"`
13. **Reboot test → Hetzner snapshot**
14. Optional: Dashboard via SSH tunnel, SSH hardening extras, Tailscale ACL

Done. Your AI assistant is live 24/7. 🦞
