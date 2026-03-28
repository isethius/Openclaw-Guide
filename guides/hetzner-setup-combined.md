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
- Never hardcode paths to Node or OpenClaw binaries — always use `which node`, `which openclaw`
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
```

**When it asks for a passphrase, set one.** A passphrase encrypts your private key so even if someone gets your laptop, they can't use the key without it. macOS Keychain remembers it after the first entry — you won't be typing it every time you connect.

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

### Step 12: Harden SSH Configuration

Since your SSH key works, password-based login becomes unnecessary attack surface. Passwords can be brute-forced; keys practically can't.

```bash
# Disable password auth
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# Disable root login
sudo sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

sudo systemctl restart sshd
```

**Why `sed` instead of `nano`:** These one-liners find and replace exact settings in the SSH config file. With `nano` you're manually hunting through the file and one typo can break SSH entirely. These commands are copy-paste safe — they handle both commented and uncommented versions of each setting.

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

### Step 15: Install OpenClaw with the Official Installer

The official installer handles Node.js detection and installation automatically, then installs OpenClaw. This is the recommended path.

OpenClaw requires Node 22 or newer. The installer will check for a compatible version and guide you if one isn't found.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw --version
```

After the installer completes, verify Node is available:

```bash
node --version    # Should show v24.x.x or v22.x.x
which node        # Record this path — you'll need it if you ever troubleshoot the service
```

> **If the installer can't find or install Node**, or if you prefer to manage Node yourself, see **Appendix B: Manual Node.js Installation** at the bottom of this guide. Install Node first, then re-run the installer above.

> **Why not `npm install -g openclaw`?** That still works, but using the official installer for both install and update means you don't have a mismatch between how it was installed and how it gets upgraded later. If you do use npm, the guide still works — just keep your update commands consistent (see the Update Procedure section).

---

## ⚙️ PHASE 8: SETUP WIZARD + FIRST TEST

### Step 16: Interactive Setup

```bash
openclaw onboard --install-daemon
```

**What `--install-daemon` does:** This runs the interactive setup wizard AND installs the background service (daemon) in one step. The flag tells OpenClaw to configure a user-level systemd service that starts automatically and survives reboots — so you don't have to hand-wire systemd later.

> **Note:** If `--install-daemon` isn't recognized in your version, run `openclaw onboard` by itself (or try `openclaw setup`). Check `openclaw --help` for available flags. You'll handle the service manually in Phase 10 if the flag isn't available.

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

### Step 17: Validate Directly on the VPS

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

### Step 18: Lock File Permissions

```bash
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json
chmod 700 ~/.openclaw/secrets/ 2>/dev/null
chmod 600 ~/.openclaw/secrets/* 2>/dev/null
```

`700` means only the `openclaw` user can read, write, or list files. Everyone else gets "Permission denied."

### Step 19: Migrate Secrets Out of Plaintext Config

OpenClaw supports **SecretRefs** — a mechanism where secrets are stored separately and referenced in config using a structured object instead of plaintext strings. This is the production default, not an optional extra.

**The hierarchy is:**

1. **SecretRefs** — use these for all supported credentials (API keys, tokens). This is the documented default for production.
2. **Environment variable fallback** — use only where the docs explicitly support it (e.g., `TELEGRAM_BOT_TOKEN` for the default Telegram account).
3. **Plaintext in JSON** — last resort, only if your installed version doesn't support SecretRefs yet.

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

> **Version note:** If `openclaw secrets` commands aren't available in your installed version, the `chmod` lockdown from Step 18 is your primary protection. Check https://docs.openclaw.ai/gateway/secrets for your version's capabilities.

The documented SecretRef object shape looks like this:

```json
{
  "source": "env",
  "provider": "default",
  "id": "OPENAI_API_KEY"
}
```

That structured form should be used wherever the docs support SecretRefs — don't use shorthand or placeholder syntax.

After migration, manually inspect `~/.openclaw/openclaw.json` and confirm **no plaintext tokens remain**. The config should contain SecretRef objects pointing to your stored secrets, not the secrets themselves.

---

## 🚀 PHASE 10: START OPENCLAW AS A SERVICE

OpenClaw runs as a **systemd user service** — auto-restarts on crashes, starts on boot.

> **If you used `openclaw onboard --install-daemon` in Step 16**, the service may already be running. Check with `systemctl --user status openclaw-gateway.service` — if it shows `active (running)`, skip to `loginctl enable-linger` below and move on.

### Step 20: Start the Gateway

```bash
openclaw gateway restart
systemctl --user status openclaw-gateway.service --no-pager | head -n 25
```

If `systemctl --user` fails in your SSH session (common issue — systemd doesn't always set up user sessions over SSH):
```bash
export XDG_RUNTIME_DIR="/run/user/$(id -u)"
export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u)/bus"
systemctl --user restart openclaw-gateway.service
```

Make it survive logouts:
```bash
loginctl enable-linger openclaw
```

> **If the built-in service commands don't work** (older version, broken install, custom requirements), see **Appendix A: Manual systemd Service** at the bottom of this guide.

### Step 21: Check Logs

```bash
LATEST="$(ls -t /tmp/openclaw/openclaw-*.log 2>/dev/null | head -1)"
tail -n 120 "$LATEST" | egrep -i 'telegram|pair|401|auth|error|listening'
```

---

## 🔑 PHASE 11: GATEWAY AUTHENTICATION

Tailscale protects the network layer — only your devices can reach the server. Gateway auth adds a second lock at the application layer. If someone gets onto your tailnet (compromised device, shared account), they still can't control your agent without the auth token. Defense in depth.

### Step 22: Generate and Set the Auth Token

```bash
# Generate a strong random token
openssl rand -hex 32
```

Copy the output and configure it:

```bash
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "YOUR_TOKEN_HERE"
openclaw gateway restart
```

> **Version note:** If `gateway.auth.mode` or `gateway.auth.token` don't work as config key paths, check https://docs.openclaw.ai/gateway/authentication for your installed version's exact syntax. Older versions may use `gateway.authToken`. Run `openclaw config get gateway` after setting to confirm it took.

### Step 23: Verify Auth Is Working (Negative Test)

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

### Step 24: Create a Telegram Bot

1. Open Telegram, search for **@BotFather**
2. Send `/newbot`
3. Choose name: `YourNameAssistant`
4. Choose username: must end in `bot` (e.g., `myopenclawbot`)
5. Copy the **Bot Token** — treat this like a password

### Step 25: Configure Telegram in OpenClaw

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

### Step 26: Approve Your First Message

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

### Step 27: Run Diagnostics

```bash
openclaw doctor
openclaw status
openclaw models status --probe
systemctl --user status openclaw-gateway.service --no-pager | head -n 25
```

Fix anything flagged.

### Step 28: Turn On Reasoning (CRITICAL)

This is the #1 setting people miss. Without it, your agent sounds shallow:

```bash
openclaw config set agents.defaults.thinkingDefault high
openclaw gateway restart
```

### Step 29: Reboot Test

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

### Step 30: Take a Hetzner Snapshot

Capture the fully working state so you can roll back if a future update breaks things:

1. Go to https://console.hetzner.cloud
2. Click your server → **Snapshots** tab
3. Click **Create Snapshot** → name it `post-setup-clean`

Take additional snapshots before major OpenClaw updates or significant config changes.

### Step 31: Create an Encrypted Backup

Everything you just configured — API keys, Telegram tokens, gateway auth, agent settings — lives in `~/.openclaw/`. Back it up, encrypted, before moving on.

```bash
# Create compressed archive
tar -czf ~/openclaw-backup-$(date +%Y%m%d).tar.gz ~/.openclaw

# Encrypt it (set a password — remember it)
openssl enc -aes-256-cbc -salt -pbkdf2 \
  -in ~/openclaw-backup-$(date +%Y%m%d).tar.gz \
  -out ~/openclaw-backup-$(date +%Y%m%d).enc

# Remove the unencrypted version
rm ~/openclaw-backup-$(date +%Y%m%d).tar.gz
```

Download the encrypted backup to your Mac:

```bash
# Run this on your Mac, not the server
scp openclaw@YOUR_TAILSCALE_IP:~/openclaw-backup-*.enc ~/Desktop/
```

**Where to store it:** Don't leave it on your Desktop. Move it to a password manager's file storage, an encrypted drive, or an offline archive.

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

> **Note on SSH tunneling:** You may see guides recommend `PermitTunnel no` and `AllowTcpForwarding no` for further hardening. Be aware that the dashboard SSH tunnel (`ssh -N -L ...` from Phase 14) uses TCP port forwarding. If you set `AllowTcpForwarding no`, the dashboard tunnel will silently stop working. Only disable TCP forwarding if you plan to access the dashboard via direct Tailscale IP instead.

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
| `openclaw: command not found` | Shell not reloaded | `source ~/.bashrc` — if Node isn't installed, see Appendix B |
| Bot doesn't respond | Multiple possible | Check logs (Phase 10), verify pairing, check `openclaw doctor` |
| Telegram says "access not configured" | dmPolicy blocks you | Approve pairing or add your user ID to allowFrom |
| Only one Telegram account starts | Account/binding mismatch | Check `channels.telegram.accounts` and `bindings` config |
| Service doesn't survive reboot | Linger not enabled or XDG issue | `loginctl enable-linger openclaw`, set XDG vars |
| Gateway accepts unauthenticated requests | Auth config didn't take | Re-check key path, restart, re-test (Phase 11) |
| Service broke after Node update | Path changed | Re-run `which node`, update service — see Appendix A |

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
- **Don't auto-update OpenClaw** — update manually, test, confirm nothing broke
- **Take a backup before every update** — see Update Procedure below

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

## 🔄 Update Procedure (Follow This Every Time)

**Never just update and hope.** Back up first, update, verify.

### Before updating:

```bash
# 1. Encrypted backup
tar -czf ~/openclaw-backup-$(date +%Y%m%d).tar.gz ~/.openclaw
openssl enc -aes-256-cbc -salt -pbkdf2 \
  -in ~/openclaw-backup-$(date +%Y%m%d).tar.gz \
  -out ~/openclaw-backup-$(date +%Y%m%d).enc
rm ~/openclaw-backup-$(date +%Y%m%d).tar.gz

# 2. Take a Hetzner snapshot (Console → Snapshots → Create)
#    Name it: pre-update-YYYYMMDD
```

### Run the update:

```bash
# 3. Update OpenClaw — preferred method (official installer, does in-place upgrade):
curl -fsSL https://openclaw.ai/install.sh | bash

# Alternative if you installed via npm:
# npm update -g openclaw

# 4. Restart the gateway
openclaw gateway restart
```

### Verify everything still works:

```bash
# 5. Run diagnostics
openclaw doctor
openclaw models status --probe

# 6. Check service status
systemctl --user status openclaw-gateway.service --no-pager | head -n 25

# 7. Send a test message on Telegram — does it respond?
```

### If something broke:

**First step — isolate optional integrations.** If you have fallback models or other optional integrations enabled, disable them before debugging further:

```bash
openclaw config set agents.defaults.model '{"primary":"anthropic/claude-sonnet-4-6"}'
openclaw gateway restart
openclaw doctor
```

If that fixes it, the issue is in the integration, not OpenClaw itself. If it doesn't, proceed with rollback:

**Snapshot rollback (cleanest):**
```bash
# Hetzner Console → Snapshots → Rebuild from pre-update snapshot
# This rolls back the entire server to the pre-update state
```

**npm version rollback (if you used npm to install):**
```bash
npm install -g openclaw@PREVIOUS_VERSION
openclaw gateway restart
```

**Config restore from encrypted backup:**
```bash
# Decrypt (enter the password you set)
openssl enc -d -aes-256-cbc -pbkdf2 \
  -in ~/openclaw-backup-YYYYMMDD.enc \
  -out ~/openclaw-backup-YYYYMMDD.tar.gz

# Stop the service first
openclaw gateway stop 2>/dev/null
systemctl --user stop openclaw-gateway.service 2>/dev/null

# Extract backup
tar -xzf ~/openclaw-backup-YYYYMMDD.tar.gz -C /

# Fix permissions
chmod 700 ~/.openclaw
chmod 700 ~/.openclaw/secrets 2>/dev/null
chmod 600 ~/.openclaw/secrets/* 2>/dev/null

# Restart
openclaw gateway restart
```

**Only proceed with normal use after all checks pass.**

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
4. Lock down: UFW (Tailscale-only SSH), harden SSH (`sed`-based, copy-paste safe), fail2ban
5. Install OpenClaw via **official installer** (handles Node automatically — see Appendix B if manual Node needed)
6. Run `openclaw onboard --install-daemon` → pick provider → paste API key → loopback mode
7. **Validate locally: `openclaw chat` → `openclaw doctor`**
8. **Lock secrets → migrate with SecretRefs (production default)**
9. Start gateway → verify systemd service → enable linger
10. **Set up gateway auth (`gateway.auth.mode token`) → negative test**
11. Set up Telegram: @BotFather → token → account-based config → approve pairing
12. Turn on reasoning: `thinkingDefault: "high"`
13. **Reboot test → Hetzner snapshot → encrypted backup**
14. Optional: Dashboard via SSH tunnel, SSH hardening extras, Tailscale ACL

Done. Your AI assistant is live 24/7. 🦞

---

## Appendix A: Manual systemd Service (Last Resort)

**You should not need this if `openclaw onboard --install-daemon` worked.** This is a recovery path for when the built-in daemon setup fails or you have custom requirements.

The critical rule: **never hardcode paths.** Always use `which node` and `which openclaw` to get your actual installed paths.

```bash
# Get your real paths first
which node        # e.g., /home/openclaw/.nvm/versions/node/v24.x.x/bin/node
which openclaw    # e.g., /home/openclaw/.nvm/versions/node/v24.x.x/bin/openclaw
```

Create the service file:

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/openclaw-gateway.service << EOF
[Unit]
Description=OpenClaw Gateway
After=network-online.target

[Service]
ExecStart=$(which node) $(which openclaw) gateway --port 18789
Restart=always
RestartSec=5
Environment=HOME=%h
Environment=PATH=$(dirname $(which node)):/usr/local/bin:/usr/bin:/bin

[Install]
WantedBy=default.target
EOF
```

**What's happening here:** The `$(which node)` and `$(which openclaw)` commands insert your actual installed paths at the time you create the file. The `PATH` environment line ensures systemd can find everything — systemd runs in a stripped-down environment that doesn't load `.bashrc`, so without this explicit PATH, nvm's paths aren't available and the service fails silently.

Enable and start:

```bash
systemctl --user daemon-reload
systemctl --user enable openclaw-gateway
systemctl --user start openclaw-gateway
loginctl enable-linger openclaw
```

**If this service breaks after a Node update:** Re-run `which node` to get the new path, recreate the service file, then `systemctl --user daemon-reload && systemctl --user restart openclaw-gateway`.

---

## Appendix B: Manual Node.js Installation (Only if Installer Fails)

**You should not need this if the official OpenClaw installer (Step 15) handled Node automatically.** Use this if the installer couldn't find or install Node, or if you prefer to manage Node versions yourself.

```bash
# Install nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc

# Install Node 24 (recommended) — or Node 22 if you prefer LTS stability
nvm install 24
nvm use 24
nvm alias default 24

# Verify
node --version    # Should show v24.x.x
which node        # Record this path
```

**Why `nvm`:** It lets you install and switch between Node versions without touching the system install. If OpenClaw later requires a different Node version, you can install it alongside the current one without breaking anything.

After installing Node, go back to Step 15 and re-run the OpenClaw installer.
