# 🦝 RunYourOwnAI Social Launch Plan — March 22, 2026

Everything below is copy-paste ready. No placeholders.

---

## 1. TWITTER/X — @RunYourOwnAI

### Profile Setup

**Display Name:** Kito 🦝 | RunYourOwnAI

**Bio (160 chars):**
```
We set up OpenClaw AI assistants on YOUR hardware. Your data, your rules. Setup packages from $449. 🦝 Built by builders. runyourownai.com
```

**Banner Copy (for Canva/design):**
```
Run Your Own AI — Personal AI assistants, installed on your hardware.
No cloud. No subscriptions. No data harvesting. Just your AI, your way.
runyourownai.com
```

**Location:** Tucson, AZ
**Website:** https://runyourownai.com

---

### Pinned Tweet

```
🦝 We set up personal AI assistants on YOUR hardware.

Not ChatGPT. Not another SaaS wrapper. An actual AI that:

→ Runs on your own machine (Mac Mini, NUC, Pi, VPS)
→ Connects to Telegram, Discord, WhatsApp, Slack
→ Controls your smart home, books appointments, captures leads
→ Keeps your data on YOUR server

We use OpenClaw — open-source, self-hosted, 24/7.

We've been building on it since week 1. Now we set it up for you.

🔧 Starter: $449 — basic install, one channel
🔧 Pro: $997 — multi-channel, custom skills, smart home, training
🔧 Local Biz: $1,497 — full business automation suite

Details → runyourownai.com
```

---

### First 10 Tweets (Post 2-3 per day)

**Tweet 1 — Myth Buster**
```
"You need a $3,000 GPU to run AI locally"

No you don't.

OpenClaw runs on a $200 Mac Mini, a $60 Raspberry Pi, or a free Oracle Cloud VPS.

The AI model runs separately (Ollama, Gemini API, whatever you want). The agent itself is lightweight.

Stop letting GPU discourse scare you off.
```

**Tweet 2 — Tip**
```
Quick OpenClaw tip:

After install, add 16GB swap even if you have plenty of RAM.

LLM inference is spiky. Swap prevents OOM kills during context-heavy conversations.

sudo fallocate -l 16G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

Add to /etc/fstab to persist across reboots.
```

**Tweet 3 — Value/Positioning**
```
The difference between ChatGPT and a self-hosted AI:

ChatGPT: you visit it, ask things, leave
Self-hosted: it's always running, watching your channels, handling tasks while you sleep

One is a tool. The other is staff.

That's what we set up → runyourownai.com
```

**Tweet 4 — Myth Buster**
```
"Self-hosting AI is only for developers"

The install? Sure, it's technical.

But USING it? You literally text it on Telegram. Or talk to it on Discord.

The hard part is setup. That's literally what we do for you.

Your mom could use it after we're done. (Seriously.)
```

**Tweet 5 — Thread Starter**
```
🧵 5 things an OpenClaw assistant can do that ChatGPT can't:

1/ Run 24/7 on YOUR machine without a browser open
2/ Connect to Telegram, Discord, WhatsApp, Slack simultaneously
3/ Control your smart home (lights, locks, cameras)
4/ Browse the web and fetch live data on command
5/ Execute shell commands, manage files, run scripts

It's not a chatbot. It's an agent.

We set these up daily → runyourownai.com
```

**Tweet 6 — Tip**
```
If you're running OpenClaw on a VPS, bind your ports to localhost.

Don't expose 18789 to the internet raw.

Use a reverse proxy (Caddy is easiest) or tunnel (Localtonet, Cloudflare Tunnel).

Free HTTPS, zero port exposure. 5 minutes of work that prevents real problems.
```

**Tweet 7 — Social Proof / Use Case**
```
Our OpenClaw setup runs across 3 businesses:

→ Answers support questions on Telegram
→ Drafts sales copy on command
→ Monitors competitors' pricing pages
→ Manages smart home devices
→ Runs research tasks in the background

One assistant. Multiple businesses. $0/month hosting (Oracle free tier).

This is what we configure for clients.
```

**Tweet 8 — Myth Buster**
```
"AI assistants are just fancy chatbots"

A chatbot answers questions.

An OpenClaw agent:
- Runs shell commands
- Controls a browser
- Reads and writes files
- Calls external APIs
- Spawns sub-agents for parallel work

It's closer to a junior employee than a chatbot.
```

**Tweet 9 — Engagement**
```
Honest question for the self-hosted crowd:

What's the #1 thing stopping you from setting up your own AI assistant?

A) Too complicated to install
B) Don't know which hardware to use
C) Not sure what it would actually DO
D) Haven't found the right software

Reply or quote — I'll answer every one. 🦝
```

**Tweet 10 — Tip**
```
OpenClaw pro tip: Use the skills system.

Skills are like plugins — weather, web search, TTS, browser control, smart home.

Don't try to do everything with raw prompts. Install the skill, configure it once, and let the agent handle it natively.

Skills > prompt engineering, every time.
```

---

### Twitter Search Queries for Daily Lead Gen Monitoring

Save these as searches and check daily:

```
"self hosted AI" -from:RunYourOwnAI
"personal AI assistant" setup -chatgpt
"openclaw" help OR setup OR install OR error
"run AI locally" -from:RunYourOwnAI
"AI on my own server"
"ollama" assistant OR agent OR setup
"self hosted" assistant OR chatbot
"home assistant AI" -homeassistant
"AI for small business" local OR self-hosted
"tired of ChatGPT" OR "leaving ChatGPT" OR "alternative to ChatGPT" self-hosted
```

---

## 2. REDDIT

### Comment Templates for r/selfhosted and r/LocalLLaMA

**Template 1 — Someone struggling with OpenClaw install**
```
Had this exact issue. A few things to check:

1. Make sure you're running Node 22+ (`node -v`). OpenClaw dropped Node 18 support.
2. If you're on a VPS, add swap — 16GB minimum. LLM inference chews through memory during long conversations.
3. The onboarding wizard (`openclaw` with no args) handles most config. If you edited `openclaw.json` manually, double-check your LLM provider section — a missing comma will silently fail.

What's the actual error output? Happy to dig in.
```

**Template 2 — Someone asking what to do with a spare Mini PC / NUC / Pi**
```
Perfect use case for a personal AI agent.

Check out OpenClaw — it's open-source, runs on lightweight hardware, and connects to Telegram/Discord/WhatsApp so you can text it like a person.

I run mine on a Mac Mini and it handles everything from smart home control to research tasks to drafting content. Runs 24/7, uses maybe 2GB RAM for the agent itself (the LLM is separate — can point it at Ollama locally or Gemini API for free).

Setup guide: https://docs.openclaw.com — the onboarding wizard walks you through it. If you get stuck, drop your error here and I'll help.
```

**Template 3 — Someone comparing AI assistant options**
```
I've tried most of these. Here's my honest take:

OpenClaw is the most "agent-like" — it actually runs shell commands, controls a browser, spawns sub-tasks. It's not just a chat UI on top of an LLM.

The trade-off: setup is more involved than something like Open WebUI (which is really just a chat frontend). OpenClaw is closer to an always-on employee than a chat window.

If you want a ChatGPT replacement UI → Open WebUI.
If you want an actual AI assistant that does things → OpenClaw.

Both are open source, both self-hosted. Different tools for different jobs.
```

**Template 4 — Someone asking about AI for their small business**
```
Running OpenClaw for exactly this.

It handles appointment reminders, captures leads from a web form, sends follow-ups via Telegram, and drafts responses to common questions. All running on a $200 machine in the office.

The key is the "skills" system — you install specific capabilities (calendar, CRM connector, web scraping, etc.) rather than trying to prompt-engineer everything. Much more reliable.

Setup takes a few hours if you're technical. There are also setup services popping up if you'd rather have someone configure it for your specific workflow — worth it if your time is worth more than a few hundred bucks.
```

**Template 5 — Someone frustrated with cloud AI privacy**
```
This is exactly why self-hosted AI is taking off.

With something like OpenClaw + Ollama, your prompts never leave your machine. Period. No training on your data, no usage logs shipped to OpenAI, no surprise policy changes.

The experience isn't identical to GPT-4 — local models are catching up fast but aren't there yet for every task. What I do: run a local model (Llama 3, Mistral) for private stuff, and route non-sensitive tasks to a cloud API when I need the power.

OpenClaw lets you configure this per-task, which is the best of both worlds.
```

---

### r/openclaw Subreddit

**Subreddit Description:**
```
The community hub for OpenClaw — the open-source personal AI assistant that runs on your own hardware. Share setups, skills, configs, and tips. Get help with installation and troubleshooting. Show off what your agent can do.
```

**Sidebar / About:**
```
🦝 Welcome to r/openclaw

OpenClaw is an open-source AI agent that runs on your hardware and connects to Telegram, Discord, WhatsApp, Slack, and more.

**Useful Links:**
- GitHub: github.com/openclaw/openclaw
- Docs: docs.openclaw.com
- Discord: discord.gg/openclaw

**Rules:**
1. Be helpful. Everyone was a beginner once.
2. Share your setup — hardware, config, use cases.
3. No spam. Sharing projects/services is fine; carpet-bombing isn't.
4. Search before posting — your error might already be solved.
5. Include logs/errors when asking for help.

**Flair:**
- 💡 Tip/Guide
- 🐛 Bug/Help
- 🛠️ My Setup
- 🧪 Skill/Plugin
- 📣 Announcement
```

**Welcome Post:**
```
Title: Welcome to r/openclaw — introduce yourself and your setup!

Hey everyone 🦝

This is the spot to share what you're building with OpenClaw, get help, and swap configs.

To kick things off, drop a comment with:

1. **Hardware** — What's your agent running on?
2. **LLM** — Local (Ollama, llama.cpp) or cloud API (Gemini, Claude, GPT)?
3. **Channels** — Telegram? Discord? WhatsApp? All of them?
4. **Favorite skill** — What's the one thing your agent does that you couldn't live without?
5. **Biggest pain point** — What's still annoying?

I'll start:

- Mac Mini M2, 16GB
- Mix of local Llama 3 and Gemini API
- Telegram + Discord
- Web search + browser control — having it research things and summarize while I work is a game changer
- Multi-agent sessions can get flaky with large context windows. Working around it with aggressive compaction settings.

Your turn. 👇
```

---

### First Value Post for r/selfhosted

```
Title: I've been running a self-hosted AI assistant 24/7 for 3 months — here's what actually works (and what doesn't)

I set up OpenClaw on a Mac Mini back in January and it's been running non-stop since. Figured I'd share what I've learned since most guides stop at "install it and you're done."

**What it is:** OpenClaw is an open-source AI agent. Not just a chat UI — it runs commands, controls a browser, manages files, connects to Telegram/Discord/WhatsApp, and runs skills (plugins) for things like weather, smart home, web scraping, TTS.

**My hardware:** Mac Mini M2, 16GB. Agent uses ~2GB RAM. Ollama runs alongside it for local inference.

**What actually works great:**

- **Telegram as the interface** — I text my AI like I'd text a person. It responds in seconds. I can send it photos, voice messages, files. This is the killer feature nobody talks about.
- **Research tasks** — "Search for X, read the top 5 results, and summarize the key takeaways." It does this autonomously with the web search + browser skills.
- **Smart home control** — Paired with Home Assistant, I can say "turn off the office lights and lock the front door" via Telegram. Works better than voice assistants because there's no "sorry I didn't catch that."
- **File management** — It reads, writes, and edits files in its workspace. I use it for drafting content, editing configs, organizing notes.

**What's meh:**

- **Local models for complex tasks** — Llama 3 8B handles simple stuff fine. Anything requiring long reasoning chains? You'll want a cloud API or a bigger model.
- **Voice** — TTS works but it's not a conversational voice assistant. It's text-first.
- **Multi-agent workflows** — Powerful but finicky. Sessions can bloat and need compaction. Getting better with each release.

**What I'd do differently:**

- Start with a cloud LLM API (Gemini free tier is generous) and add local models later. Don't let the "fully local" dream slow down your setup.
- Use the onboarding wizard. Don't try to hand-edit configs on day one.
- Add swap. Even with 16GB RAM. Just do it.

**The real value:** Having an always-on AI that I can message from my phone, that knows my context, that can actually DO things on my server — that's fundamentally different from visiting ChatGPT in a browser. It's the difference between a search engine and an employee.

Happy to answer questions. Been deep in this for a while now.

**Links:**
- OpenClaw GitHub: github.com/openclaw/openclaw
- Docs: docs.openclaw.com
```

---

## 3. DISCORD — OpenClaw Server

### Self-Promo Channel Post

```
🦝 Hey everyone — I'm Kito from RunYourOwnAI.com

We're an OpenClaw setup service. We install, configure, and customize OpenClaw for people who want a personal AI assistant but don't want to spend a weekend debugging YAML files.

**What we do:**
→ Install OpenClaw on your hardware (Mac Mini, NUC, Pi, VPS — whatever you've got)
→ Connect it to your channels (Telegram, Discord, WhatsApp, Slack)
→ Set up skills for your specific use case (smart home, business automation, research, etc.)
→ Hand it back to you working, with a walkthrough so you actually know how to use it

**Packages:**
- Starter ($449) — Basic install, one channel, core skills
- Pro ($997) — Multi-channel, custom skills, smart home, training session
- Local Biz ($1,497) — Full business automation (appointments, leads, CRM)
- Support ($149/mo) — Ongoing maintenance + new skill installs

We've been building on OpenClaw since week 1 and use it across multiple businesses daily.

If you've been meaning to set this up but keep putting it off — that's literally why we exist.

🔗 runyourownai.com
```

---

### Top 5 GitHub Issues to Comment On (with helpful context)

Based on current open issues, these are the highest-value ones to engage with:

**1. Sessions becoming unresponsive with oversized images in history**
```
Comment: Running into this too. Workaround that's been reliable for us: set max image size in your agent config and enable aggressive history compaction. We trim image attachments from history after processing to prevent context bloat. Not a fix, but keeps sessions alive.
```

**2. Telegram setMyCommands failing with 100+ commands from native skills**
```
Comment: Hit this on a client setup. Telegram's Bot API caps setMyCommands at 100. Quick fix: in your skills config, disable command registration for skills you use via natural language anyway (most of them). Only register commands for the ones you actually type as /commands.
```

**3. WhatsApp QR codes not being recognized by mobile app**
```
Comment: This is usually a rendering issue. Try opening the QR from the web UI instead of terminal output — terminal font/spacing can corrupt the QR. Also make sure you're scanning with WhatsApp's "Link a Device" flow, not the camera app. If using a VPS, access via SSH tunnel (localhost:18789) rather than public URL for the initial QR scan.
```

**4. Windows: parseCommandLine() eating backslashes in gateway.cmd**
```
Comment: Can confirm this on Windows 11. Workaround: use forward slashes in your paths or run via WSL2 instead of native Windows. WSL2 + Docker is more reliable than native Windows for OpenClaw right now. Know this isn't ideal but it's stable.
```

**5. Grok/xAI tool call JSON failures (HTML entity encoding)**
```
Comment: This is the provider encoding & and < as HTML entities in JSON tool call arguments. If you need xAI, you can patch the response parser to decode entities before JSON.parse. Alternatively, switching to the OpenRouter proxy for xAI models avoids this since OpenRouter normalizes the response format.
```

---

## 4. DAILY ROUTINE CHECKLIST (30-40 min)

### Morning Block — 20 minutes

```
[ ] Twitter (10 min)
    [ ] Post 1-2 pre-written tweets from the queue
    [ ] Check notifications — reply to any comments/questions
    [ ] Run 3 saved search queries (rotate through the list)
    [ ] Like/reply to 2-3 relevant tweets from search results
        → Be helpful, not salesy. Answer their question. Link only if directly relevant.

[ ] Reddit (10 min)
    [ ] Check r/selfhosted, r/LocalLLaMA, r/homelab — sort by New
    [ ] Search "OpenClaw" across Reddit
    [ ] Reply to 1-2 posts where you can genuinely help
        → Use comment templates as starting points, customize for context
    [ ] Check r/openclaw for any posts (once the sub exists)
```

### Afternoon Block — 15 minutes

```
[ ] Discord (5 min)
    [ ] Skim OpenClaw Discord #general and #help channels
    [ ] Answer 1-2 questions if you can add value
    [ ] Note any recurring issues for future content

[ ] GitHub (5 min)
    [ ] Check OpenClaw issues tagged "bug" or "help wanted"
    [ ] Comment on 1 issue with a workaround or confirmation
    [ ] Star/watch any new repos in the OpenClaw ecosystem

[ ] Content Prep (5 min)
    [ ] Draft tomorrow's tweets (or refine from the queue)
    [ ] Note any questions/topics from today that would make good content
    [ ] Save interesting threads/posts for future reference
```

### Weekly (pick one day)

```
[ ] Write and post 1 long-form Reddit post (r/selfhosted or r/LocalLLaMA)
[ ] Create 1 Twitter thread (3-5 tweets)
[ ] Review analytics — which tweets/posts got traction? Do more of that.
[ ] Update the tweet queue with 7+ new tweets for the week
[ ] Check runyourownai.com for any needed updates based on questions received
```

---

## Quick Reference: Voice & Tone Rules

| Do | Don't |
|---|---|
| Be genuinely helpful first | Lead with pricing or links |
| Share real experience and configs | Make vague claims ("best AI ever") |
| Admit limitations honestly | Oversell local model capabilities |
| Use specific technical details | Write marketing fluff |
| Engage with follow-up questions | Drop a comment and disappear |
| Position as "we use this daily" | Position as "we're an agency" |

**The golden rule:** If you removed the runyourownai.com link, would the post still be valuable? If yes, it's good. If no, rewrite it.

---

*Plan created March 22, 2026. All content ready for immediate deployment.*
