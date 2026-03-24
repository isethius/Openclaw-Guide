# Ready-to-Paste Copy — All Channels
## Created: 2026-03-23

---

## 1. FIVERR GIG LISTING

### Gig Title:
`I will install and configure OpenClaw AI assistant on your server`

### Search Tags:
`OpenClaw, AI assistant setup, AI installation, personal AI, self-hosted AI, OpenClaw setup, VPS setup, AI automation, Telegram bot, Discord bot`

### Gig Description:

```
Are you trying to set up OpenClaw and hitting walls?

I've done 20+ OpenClaw installations — on VPS servers, Mac Minis, and Raspberry Pis. I handle the entire setup so you don't have to spend a weekend debugging Node.js versions and gateway configurations.

**What You Get:**

Your own AI assistant, fully configured, secured, and running 24/7. You message it from Telegram, Discord, WhatsApp — whatever you use. It remembers everything, runs automations, and keeps your data on YOUR server (not OpenAI's).

**Every installation includes:**
✅ OpenClaw installed and configured
✅ AI provider set up (Claude, GPT, or Gemini — your choice)
✅ Messaging channel connected (Telegram, Discord, WhatsApp)
✅ Security hardening (SSH keys, firewall, auto-updates)
✅ Tailscale for secure remote access
✅ PM2 auto-restart (your AI survives server reboots)
✅ Post-install support

**What I need from you:**
- A VPS (I can help you pick one — $5-6/month)
- Which AI provider you want (I'll help you decide)
- Which messaging app you want to use
- 15 minutes to walk you through Tailscale setup on your end

**The security stuff matters:** 21,000+ OpenClaw instances were exposed in early 2026 because people skipped security hardening. Every install I do includes proper firewall config, SSH hardening, and gateway protection.

**Why hire me instead of following a YouTube tutorial?**
Because tutorials show you how to get it running. I make sure it's running RIGHT — secured, optimized, and configured for your specific use case. My installs don't end up on the "exposed instances" list.

Message me with your use case and I'll tell you which package fits.
```

### FAQ (for Fiverr):

**Q: Do I need technical skills?**
A: No. I handle everything technical. You just need to set up a VPS account (I'll walk you through it) and install Tailscale on your computer (5 minutes). After that, you interact with your AI through Telegram/Discord — apps you already know.

**Q: What's the ongoing cost after setup?**
A: VPS hosting: $5-20/month. AI API calls: $5-35/month depending on usage. No subscriptions, no per-seat fees. You own it.

**Q: Can you set it up on my existing server?**
A: Yes, as long as it runs Linux and has at least 2 CPU cores and 4GB RAM. Mac also works.

**Q: How long does setup take?**
A: Typically 24-48 hours from when I get access. I'll message you when it's ready and schedule a walkthrough.

**Q: Is OpenClaw safe?**
A: Yes — when properly configured. The security incidents in 2026 were all from misconfigured DIY installs, not OpenClaw bugs. Every install I do includes full security hardening.

---

## 2. UPWORK PROFILE

### Title:
`OpenClaw AI Assistant Setup & Configuration Specialist`

### Overview:
```
I specialize in one thing: setting up OpenClaw AI assistants that actually work.

20+ installations completed on VPS servers, Macs, and Raspberry Pis. Every setup includes security hardening, API optimization, and channel configuration (Telegram, Discord, WhatsApp).

What makes my installs different:
• Security-first: SSH hardening, firewall, Tailscale — your instance won't end up on the "exposed" list
• Cost-optimized: Model routing that cuts your API costs 40-60%
• Actually configured: Not just installed — persona, skills, memory, channels all set up for YOUR use case

Services:
• Full OpenClaw installation and configuration ($449-$1,497)
• Security audit of existing installations ($149)
• Monthly maintenance and support ($149/month)
• Custom skill development ($199+)

I respond within 2 hours during business hours. Let's talk about what you need.
```

---

## 3. DISCORD INTRO (for OpenClaw server)

### Bio:
`OpenClaw setup specialist | 20+ installs | runyourownai.com`

### First message in #introductions (if they have one):
```
Hey everyone 👋 I'm Seth. I've been setting up OpenClaw since the early days — done about 20 installs now on everything from $5 VPS boxes to Raspberry Pis.

I run a setup service (runyourownai.com) but mostly I'm here to help. If you're stuck on something, drop it in #users-helping-users and I'll take a look. The security/gateway stuff is my specialty — I've seen every way an install can go sideways.

Happy to be here. 🤙
```

---

## 4. DISCORD HELP RESPONSE TEMPLATES

### Template 1: Node.js Version Issue
```
That's a Node version issue — OpenClaw needs v22+. If you installed via apt, it probably gave you v18 or v20.

Quick fix:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc
nvm install 22
nvm use 22
nvm alias default 22

Then reinstall OpenClaw: npm install -g openclaw

Run `node --version` to verify you're on v22. That should clear it up.
```

### Template 2: Gateway Won't Start
```
Check if something else is using port 3000:
ss -tlnp | grep 3000

If yes, either kill that process or change OpenClaw's port:
openclaw config set gateway.port 3001

Also run `openclaw doctor` — it'll tell you exactly what's wrong. 9 times out of 10 it's a port conflict or a Node version issue.
```

### Template 3: Exposed Gateway / Security
```
If your gateway is bound to 0.0.0.0, it's exposed to the internet. Fix it now:

openclaw config set gateway.bind 127.0.0.1

Then use Tailscale for remote access instead of exposing ports:
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

This way your instance is only accessible through your private Tailscale network. Free for personal use.

Also strongly recommend: disable SSH password auth, set up UFW firewall, and install fail2ban. The 21K exposed instances from earlier this year were all from skipping this stuff.
```

### Template 4: Telegram Bot Not Responding
```
Three things to check:

1. Token correct? Double-check with @BotFather on Telegram
2. allowFrom set? Run: openclaw config get channels.telegram.allowFrom — this should be YOUR Telegram user ID (number, not username)
3. Gateway running? Check: openclaw gateway status

If all three look good, try increasing the timeout. The default can be too short on slower VPS boxes:
openclaw config set channels.telegram.timeout 15000

Restart after: openclaw gateway restart
```

### Template 5: After Helping Someone (Natural Service Mention)
```
Glad that fixed it! The setup can be tricky — I've hit every one of these issues across my installs.

If you ever want the full thing configured properly (security, multiple channels, custom skills, etc.) without the debugging marathon, I do that professionally at runyourownai.com. No pressure though — happy to keep helping here. 🤙
```

---

## 5. GITHUB ISSUE COMMENT TEMPLATES

### Template: Temp Directory Error (#27329)
```
Hit this on a few installs. Quick fix:

mkdir -p /tmp/openclaw-$(whoami)
chmod 700 /tmp/openclaw-$(whoami)
openclaw gateway restart

The issue is that after updates, the expected temp directory either doesn't exist or has wrong permissions. Creating it manually with proper ownership resolves it.

If you're on a shared hosting environment, you might also want to check that your user owns the ~/.openclaw directory:
ls -la ~/.openclaw/

Everything should be owned by your user, not root.
```

### Template: Fresh Install Getting Old Version (#31442)
```
If you're getting 2026.2.26 instead of latest:

npm cache clean --force
npm install -g openclaw@latest

Verify with: openclaw --version

If you installed via a package manager instead of npm, that's likely the issue — the package repos lag behind. Always install via npm directly for the latest version.
```

---

## 6. YOUTUBE COMMENT TEMPLATES

### On Tutorial Videos (when someone says "stuck"):
```
Had the same issue on a few installs. The fix for [specific problem] is:
[specific fix]

If you're still stuck, check out runyourownai.com — I do professional OpenClaw setups and can save you the debugging headaches. But try the fix above first, should work. 👍
```

### On "Is OpenClaw Worth It?" Videos:
```
Been using OpenClaw daily for months now. The install is the hard part — once it's running and configured properly, it's genuinely better than ChatGPT for personal use because it actually remembers everything and works through your existing messaging apps.

The security setup is critical though. A lot of the "horror stories" are from people who skipped firewall/SSH config. Proper setup takes about 2 hours if you know what you're doing.
```

---

## 7. BUDDY REFERRAL MESSAGE

```
Hey man — remember when you paid me to set up OpenClaw? I actually turned that into a legit service. Got a whole website up at runyourownai.com.

Two favors if you're down:
1. Would you mind writing me a quick 2-sentence testimonial about the setup? Something like what you liked about it, that it works, whatever feels honest. Would help a ton.
2. Know anyone else who'd want their own AI assistant set up? I'll give you $50 for anyone you send my way who signs up.

No pressure on either. Appreciate you being the first customer lol 🤙
```

---

## 8. PRODUCT HUNT LISTING

### Tagline:
`Professional OpenClaw setup — your own AI assistant, done in 48 hours`

### Description:
```
RunYourOwnAI is a professional setup service for OpenClaw — the open-source AI assistant with 318K+ GitHub stars.

We handle the entire installation: server provisioning, security hardening, channel configuration (Telegram, Discord, WhatsApp), custom AI persona, skills, and API optimization.

Why? Because OpenClaw is powerful but the setup is brutal. 21,000 instances were exposed in 2026 from misconfigured DIY installs. We make sure yours isn't one of them.

• Starter: $449 — Basic install, 1 channel, security
• Pro: $997 — Full setup, multi-channel, training
• Local: $1,497 — In-person (Tucson/Phoenix AZ)

Every install includes security hardening, API cost optimization, and post-install support.
```

### First Comment (as maker):
```
Hey PH! 👋 I'm Seth. I started RunYourOwnAI after a friend paid me $200 to set up OpenClaw for him — then I found out a company in SF charges $6,000 for the same thing.

I've done 20+ installs now and turned it into a proper service. We're one of the first OpenClaw setup specialists and we actually use it daily across all our businesses.

Happy to answer any questions about OpenClaw, self-hosted AI, or the setup process!
```
