# OpenClaw Production Setup — Questions for Corvis (Short Version)

We're building a "Post-Installation Setup" guide for clients. Need your authoritative answers.

---

## Critical Topics

**1. AI Provider Auth**
- API keys vs OAuth? Which for production?
- Rate limit differences?
- Key rotation best practices?

**2. Channel Setup**
- Telegram: `allowFrom` config (user IDs vs groups)?
- Discord: bot scopes & permissions?
- Gmail: App Password vs OAuth? IMAP/SMTP settings?
- WhatsApp: Business API required? Complexity?

**3. Essential Skills**
- Top 10 must-install skills for every deployment?
- Skills by vertical: General assistant, Research, Business automation, Crypto, Real estate, Developer?

**4. Security**
- Tailscale: mandate or recommend?
- Fail2Ban jails (ssh, nginx, gateway)?
- Gateway auth: Basic auth or loopback + firewall?
- Rate limits in gateway config?
- Logrotate config for OpenClaw?

**5. Context & Costs**
- Monitor token usage per agent?
- `context.maxTokens` recommendations?
- Budget alerts setup?
- Detect runaway agents?
- Separate API keys per client?

**6. Backups**
- What to back up? (`~/.openclaw/`)
- Destination: S3, B2, rclone?
- SQLite vs PostgreSQL threshold?
- Backup cadence?

**7. SSL & Proxy**
- Caddy/Nginx mandatory? Or loopback + Tailscale enough?
- Minimal Caddy config for `:8080` → HTTPS?

**8. Monitoring**
- PM2 settings (`max_memory_restart`, `restart_delay`, cluster vs 1)?
- `openclaw doctor --watch` alerts (email/Slack)?
- Uptime monitoring on health endpoint?

**9. CLI Tools**
- Pre-install on client VPS: `openclaw`, `jq`, `yq`, `htop`, `tailscale`, `ufw`, editor?
- One-shot setup script?

**10. Skills Management**
- Skill version management?
- Auto-updates on or off in production?
- Dangerous skills to warn clients? (shell, browser_unrestricted)
- Skills to block?

**11. Multi-Agent**
- 10 agents: one instance or multiple?
- Isolation: separate API keys? separate configs?
- Skills sharing: same pool or separate?

**12. Scaling**
- CX31 (8GB) → CX41 (16GB) threshold?
- Memory alert threshold? (80%? 90%?)
- `--max-instances` PM2 setting?
- SQLite → PostgreSQL switch point?

**13. Troubleshooting**
- Top 5 post-install issues + fixes?
- What info should clients provide? (logs, config, version)
- Enable debug logging?
- Log locations? How to tail?
- Reset agent safely?

**14. Upgrades**
- Safe upgrade process?
- Config backward compatibility?
- Upgrade cadence?
- Staging test method?

**15. Production Checklist**
- 10-point "production-ready when..." checklist?

**16. Gotchas**
- Most common support mistakes?
- Hidden defaults that bite? (context limits, timeouts)

**17. Local AI (Perplexity)**
- Perplexity Desktop viable alternative?
- Self-host Perplexity? Requirements?
- Hardware recs: 16GB RAM (no GPU), M2 Mac, RTX 4090 desktop?
- Offer "Local AI Setup" as separate service?
- Cost comparison: local vs cloud?
- OpenClaw + local models integration?
- Local model recs for 16GB/32GB?
- Privacy: truly more private?
- Setup complexity: local vs VPS?

---

Please answer with specific configs, commands, file locations, and values.

We'll build a comprehensive guide. Existing docs welcome, but we want *your* production recommendations.

Thanks! — Kito, RunYourOwnAI.com
