# OpenClaw Production Setup — Questions for Corvis

**Context:** We run `RunYourOwnAI.com` — professional OpenClaw setup service for clients. We need authoritative answers on production deployment best practices to build a "Post-Installation Setup" guide that's bulletproof.

We've built a site, guides, and are deploying client instances. Before we finalize our checklist, we want your official recommendations.

---

## 1. AI Provider Authentication

**Q:** For Anthropic/OpenAI/Google/Groq — should clients use **API keys** or **OAuth flows**? Which is recommended for self-hosted production deployments?

**Q:** Are there rate limit differences between key types that matter for 10+ agent deployments?

**Q:** Best practices for rotating keys without downtime?

---

## 2. Channel Setup

### Telegram
**Q:** Recommended `channels.telegram.allowFrom` configuration:
- Specific user IDs only?
- Allow groups?
- Any security gotchas?

### Discord
**Q:** What permissions/scopes should the bot invite link include?
- DM only or server + DM?
- Recommended gateway mode (loopback vs public)?

### Gmail/Email
**Q:** App Password vs OAuth — which is more reliable for self-hosted Gmail channel?
- Any specific IMAP/SMTP settings we must configure?
- Do app passwords work with 2FA enabled?

### WhatsApp
**Q:** Can we use personal WhatsApp or is Business API required?
- What's the setup complexity and cost?
- Are there official OpenClaw skills for WhatsApp?

---

## 3. Essential Skills by Use Case

**Q:** What are your **top 10 must-install skills** for every new deployment?

**Q:** Is there an "essential skills pack" or curated list from ClawHub?

**Q:** For these verticals, which skills are critical vs nice-to-have?

| Vertical | Essential Skills | Nice-to-have |
|----------|------------------|--------------|
| General assistant | ? | ? |
| Research | ? | ? |
| Business automation | ? | ? |
| Local business | ? | ? |
| Crypto/trading | ? | ? |
| Real estate | ? | ? |
| Developer | ? | ? |

---

## 4. Security Hardening (Beyond SSH/UFW)

**Q:** Tailscale — should we mandate it for all clients or just recommend?

**Q:** Fail2Ban — which jails are critical? (ssh, nginx, gateway?)

**Q:** Gateway authentication — Basic auth or just loopback + firewall?

**Q:** Rate limiting — should we enable per-channel rate limits in gateway config?

**Q:** Log rotation — what's your recommended `logrotate` config for OpenClaw logs?

**Q:** Automatic updates — is `unattended-upgrades` sufficient or need custom cron for OpenClaw itself?

---

## 5. Context Management & Cost Control

**Q:** How to monitor token usage per agent/usage in real-time?

**Q:** Recommended `context.maxTokens` global setting? Per-skill overrides?

**Q:** Best practice: Daily/weekly budget alerts via which method?

**Q:** How to identify runaway agents (infinite loops, runaway contexts)?

**Q:** Should each client environment use separate AI provider keys? Or shared keys with separate billing?

---

## 6. Backups & Persistence

**Q:** What files/directories MUST be backed up? (`~/.openclaw/` includes DB, configs, skills — confirm)

**Q:** Recommended backup destination: S3, Backblaze B2, rclone to another server?

**Q:** SQLite vs PostgreSQL — when should clients switch to Postgres? (agent count threshold?)

**Q:** Backup cadence: Daily? Hourly? What's your production recommendation?

---

## 7. SSL & Reverse Proxy

**Q:** Should every production OpenClaw sit behind Caddy/Nginx with SSL?

**Q:** Or is loopback + Tailscale sufficient (no public ports)?

**Q:** If using Caddy, what's the minimal config to proxy `:8080` → HTTPS with auto-SSL?

---

## 8. Monitoring & Alerts

**Q:** PM2 config — what are your recommended settings?
- `max_memory_restart`?
- `restart_delay`?
- `instances` (cluster mode) or 1?

**Q:** Should we use `systemd` timers for health checks instead of PM2?

**Q:** How to configure `openclaw doctor --watch` to alert on failures? (Email? Slack webhook?)

**Q:** Uptime monitoring — do you recommend UptimeRobot/StatusCake hitting the gateway health endpoint?

---

## 9. CLI Tools (Client & Setup)

**Q:** Which CLIs should we pre-install on every client VPS?
- `openclaw` (obvious)
- `jq`, `yq` (config manipulation)
- Editors (`nano`/`vim`)
- Monitoring (`htop`, `iotop`)
- Network (`tailscale`, `ufw`)

**Q:** Should we ship a setup script that installs all these + OpenClaw in one go?

---

## 10. Skills Management Best Practices

**Q:** How to manage multiple skill versions? (ClawHub handles versioning but client concerns?)

**Q:** Should we disable auto-updates (`skill-updates` off) in production and update manually?

**Q:** How to audit skill permissions? Which skills are **dangerous** for beginners? (shell, browser_unrestricted, file_write outside allowed dirs?)

**Q:** Any skills that should be **blocked** from installation on client systems?

---

## 11. Multi-Agent & Team Setup

**Q:** For 10 agents, should they all run on one OpenClaw instance or separate?

**Q:** How to isolate agents between each other? Separate API keys? Separate config directories?

**Q:** Recommended pattern: one OpenClaw per business unit vs one per use case?

**Q:** Skills sharing — same skill pool or separate per agent?

---

## 12. Scaling Considerations

**Q:** At what agent count/memory usage do you recommend upgrading from CX31 (8GB) to CX41 (16GB)?

**Q:** What memory threshold should trigger an upgrade alert? (80%? 90%?)

**Q:** Should we set `--max-instances` on PM2 to avoid memory exhaustion?

**Q:** Database performance — SQLite fine up to how many agents? When switch to PostgreSQL?

---

## 13. Troubleshooting & Support

**Q:** Top 5 most common issues after install and their fixes?

**Q:** What information should clients provide when asking for help? (logs location, config snippets, version output)

**Q:** How to enable debug logging? (`LOGGING=debug openclaw gateway start`)

**Q:** Where are logs located by default? How to tail them live?

**Q:** How to safely reset an agent to defaults without losing global config?

---

## 14. Version Upgrades

**Q:** How to upgrade OpenClaw safely? (`npm update -g openclaw` then config migration?)

**Q:** Are configs backward compatible? Should we backup before every upgrade?

**Q:** Recommended upgrade cadence? Monthly? Quarterly?

**Q:** How to test upgrade in staging before production? (clone config, run upgrade, validate)

---

## 15. Production Readiness Checklist (Summary)

**Q:** Can you give us a **10-point checklist** that says "Your OpenClaw is production-ready when..."?

Example:
- [ ] AI provider keys configured and tested
- [ ] At least 1 channel operational
- [ ] Security hardened (SSH keys only, firewall, fail2ban)
- [ ] Backups scheduled and tested restore
- [ ] Monitoring alerts configured
- [ ] ... etc

We'll build our entire post-install guide around this checklist.

---

## 16. Gotchas & Recurring Support Issues

**Q:** What are the **most common mistakes** you see repeatedly in support tickets?

**Q:** What do clients usually misunderstand or misconfigure that leads to failures?

**Q:** Any hidden defaults that bite people? (context limits, timeouts, rate limits)

---

## 17. Optional: Advanced Features

**Q:** When should clients consider:
- Separate PostgreSQL?
- Caddy reverse proxy with SSL?
- Multiple OpenClaw instances vs single instance?
- Custom skill development vs ClawHub skills?

---

## 18. Perplexity & Local Computer Setups

**Context:** Some clients want to run AI on their own desktop/laptop instead of a VPS. Perplexity Desktop is one option. How does this fit into the OpenClaw ecosystem?

**Q:** Is Perplexity Desktop a viable alternative to OpenClaw for clients? What are the trade-offs?

**Q:** Can Perplexity be self-hosted on a local machine? What are the requirements (OS, RAM, GPU)?

**Q:** Hardware recommendations for running local AI (Perplexity or other models) on:
- Windows laptop (16GB RAM, no GPU)
- MacBook Pro M2 (16GB unified)
- Desktop with RTX 4090

**Q:** Should we offer "Local AI Setup" as a separate service tier? What does it include?

**Q:** Cost comparison:
- VPS ($10/mo) + API ($40/mo) = $50/mo total
- Local (hardware upfront + no monthly API if using local models)
- When does local become cheaper vs cloud?

**Q:** If client wants to run models locally (Ollama, LM Studio), can OpenClaw integrate with that?

**Q:** What's the recommended local model for 16GB RAM? For 32GB?

**Q:** Privacy comparison: Is local AI truly more private than cloud APIs? (metadata, logging concerns)

**Q:** Setup complexity: How much harder is local AI for a non-technical client vs VPS-based OpenClaw?

**Q:** Should we create a separate "Local AI Setup" guide for clients who want to avoid monthly API costs?

---

**Total:** ~70 questions across 18 categories.

**Please answer with specific configurations, commands, file locations, and recommended values.** We'll turn this into a comprehensive "Post-Installation Setup" guide for our clients and include it in our GitHub repo.

Also: If there's existing documentation that covers these topics, point us to it and we'll summarize. But we want *your* production recommendations specifically.

Thanks! — Kito (RunYourOwnAI.com)

---

**Last updated:** 2026-03-25
**Repo:** https://github.com/beckelspreadsheets-ui/OpenClaw-Guide
