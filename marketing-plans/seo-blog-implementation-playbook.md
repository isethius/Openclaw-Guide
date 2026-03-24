# SEO & Blog Implementation Playbook — RunYourOwnAI.com
**Created:** 2026-03-22  
**Status:** Ready to execute  
**Estimated total time:** 6-8 hours

---

## ⚠️ PREREQUISITE: Recover Site Source Files

The `/site/` directory (symlinked to `clawsetup-site`) is **empty** — only `.vercel/project.json` exists. The homepage is live on Vercel, meaning prior deployments used source files that are no longer local.

**Before doing ANYTHING below, you must recover the source:**

```bash
# Option A: Pull from the last Vercel deployment
cd /home/openclaw/.openclaw/workspace/clawsetup-site
vercel pull --yes --token $(cat ~/.openclaw/secrets/vercel.json | jq -r '.token')

# Option B: If vercel pull doesn't restore files, download the live HTML
mkdir -p /home/openclaw/.openclaw/workspace/clawsetup-site
curl -o /home/openclaw/.openclaw/workspace/clawsetup-site/index.html https://runyourownai.com
```

Once you have the source files locally, proceed.

---

## PART 1: Sitemap.xml (Currently 404)

### What to Create

The site is a static HTML site deployed on Vercel. No framework (no Next.js, no Astro). Just HTML files.

**Create file:** `/site/sitemap.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <!-- Homepage -->
  <url>
    <loc>https://runyourownai.com/</loc>
    <lastmod>2026-03-22</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  
  <!-- Existing Pages -->
  <url>
    <loc>https://runyourownai.com/what-is-openclaw</loc>
    <lastmod>2026-03-22</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
  <url>
    <loc>https://runyourownai.com/openclaw-vs-chatgpt</loc>
    <lastmod>2026-03-22</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
  
  <!-- City Pages -->
  <url>
    <loc>https://runyourownai.com/setup/tucson</loc>
    <lastmod>2026-03-22</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
  <url>
    <loc>https://runyourownai.com/setup/phoenix</loc>
    <lastmod>2026-03-22</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
  
  <!-- Blog Posts (add as published) -->
  <url>
    <loc>https://runyourownai.com/blog/is-openclaw-safe</loc>
    <lastmod>2026-03-22</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>
  <url>
    <loc>https://runyourownai.com/blog/openclaw-setup-cost</loc>
    <lastmod>2026-03-22</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>
  <url>
    <loc>https://runyourownai.com/blog/how-to-install-openclaw</loc>
    <lastmod>2026-03-22</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>
</urlset>
```

### Update robots.txt

The existing `robots.txt` is a Vercel/Cloudflare default with AI content signals. You need to **append** a sitemap directive. Create or update `/site/robots.txt`:

```
User-agent: *
Allow: /

Sitemap: https://runyourownai.com/sitemap.xml
```

Keep the existing content-signal section if desired, but ensure the `Sitemap:` line is present.

### Vercel Routing (if needed)

If Vercel doesn't serve `/sitemap.xml` automatically from the file, add a `vercel.json` in the site root:

```json
{
  "cleanUrls": true,
  "trailingSlash": false,
  "headers": [
    {
      "source": "/sitemap.xml",
      "headers": [
        { "key": "Content-Type", "value": "application/xml" }
      ]
    }
  ]
}
```

`cleanUrls: true` also explains why `.html` extensions aren't needed in URLs (Vercel strips them automatically).

### Deploy

```bash
cd /home/openclaw/.openclaw/workspace/clawsetup-site
vercel --yes --prod --token $(cat ~/.openclaw/secrets/vercel.json | jq -r '.token')
```

### Verify
```bash
curl -I https://runyourownai.com/sitemap.xml
# Should return 200 with Content-Type: application/xml
```

---

## PART 2: Schema Markup

Add JSON-LD schema to the `<head>` of each page. Here are the exact scripts.

### 2A. Homepage — Organization + Service + LocalBusiness + FAQPage

Insert before `</head>` in `index.html`:

```html
<!-- Schema: Organization -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Run Your Own AI",
  "alternateName": "ClawHQ",
  "url": "https://runyourownai.com",
  "description": "Professional OpenClaw AI assistant setup service. Remote worldwide, in-person in Tucson & Phoenix AZ.",
  "foundingDate": "2026",
  "founder": {
    "@type": "Person",
    "name": "Seth",
    "jobTitle": "Founder"
  },
  "areaServed": [
    {"@type": "Country", "name": "United States"},
    {"@type": "City", "name": "Tucson", "containedInPlace": {"@type": "State", "name": "Arizona"}},
    {"@type": "City", "name": "Phoenix", "containedInPlace": {"@type": "State", "name": "Arizona"}}
  ],
  "contactPoint": {
    "@type": "ContactPoint",
    "contactType": "sales",
    "availableLanguage": "English"
  }
}
</script>

<!-- Schema: Service (one per tier) -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Service",
  "serviceType": "AI Assistant Setup",
  "provider": {
    "@type": "Organization",
    "name": "Run Your Own AI"
  },
  "name": "OpenClaw Professional Setup",
  "description": "Professional installation and configuration of OpenClaw AI assistant on your own server.",
  "offers": [
    {
      "@type": "Offer",
      "name": "Starter Setup",
      "price": "449",
      "priceCurrency": "USD",
      "description": "Basic OpenClaw install, 1 AI provider, 1 messaging channel, 7-day support"
    },
    {
      "@type": "Offer",
      "name": "Pro Setup",
      "price": "997",
      "priceCurrency": "USD",
      "description": "Full setup with 2+ channels, custom skills, API optimization, 14-day hypercare, 1-hour training"
    },
    {
      "@type": "Offer",
      "name": "Local Setup",
      "price": "1497",
      "priceCurrency": "USD",
      "description": "In-person white-glove installation in Tucson & Phoenix AZ, 2-hour training, 30-day support"
    },
    {
      "@type": "Offer",
      "name": "Monthly Support",
      "price": "149",
      "priceCurrency": "USD",
      "priceSpecification": {
        "@type": "UnitPriceSpecification",
        "billingDuration": "P1M"
      },
      "description": "Ongoing maintenance, updates, troubleshooting, up to 5 support requests per month"
    }
  ]
}
</script>

<!-- Schema: LocalBusiness (for Tucson/Phoenix local SEO) -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Run Your Own AI",
  "description": "OpenClaw AI assistant setup service in Tucson & Phoenix, Arizona",
  "url": "https://runyourownai.com",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "Tucson",
    "addressRegion": "AZ",
    "addressCountry": "US"
  },
  "areaServed": ["Tucson", "Phoenix", "Scottsdale", "Mesa", "Chandler", "Tempe"],
  "priceRange": "$449-$1497"
}
</script>

<!-- Schema: FAQPage (pulls from the FAQ section on homepage) -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is OpenClaw?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "OpenClaw is an open-source AI assistant that runs on your own server. It connects to Telegram, WhatsApp, Discord, and other apps, with persistent memory and the ability to control your computer — all while keeping your data private."
      }
    },
    {
      "@type": "Question",
      "name": "How much does OpenClaw setup cost?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Our setup starts at $449 for a basic install (Starter), $997 for a full setup with multiple channels and training (Pro), and $1,497 for in-person installation in the Tucson/Phoenix area (Local). Ongoing hosting costs $5-20/month for a VPS."
      }
    },
    {
      "@type": "Question",
      "name": "Is OpenClaw safe and secure?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "OpenClaw itself is secure, but improper setup can expose your server. Every installation we do includes SSH hardening, firewall configuration, and security best practices. The recent security incidents (ClawHavoc, exposed instances) were all due to misconfigured self-installs, not OpenClaw vulnerabilities."
      }
    },
    {
      "@type": "Question",
      "name": "Do I need technical skills to use OpenClaw?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No. We handle all the technical setup — server provisioning, installation, configuration, and security. Once installed, you interact with your AI through apps you already use (Telegram, WhatsApp, Discord). We also provide training to get you comfortable."
      }
    },
    {
      "@type": "Question",
      "name": "What's the ongoing cost after setup?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "After the one-time setup fee, your only costs are VPS hosting ($5-20/month, or free if you run on your own hardware) and AI API usage ($5-35/month depending on how much you use it). We optimize your model routing to keep costs as low as possible."
      }
    }
  ]
}
</script>
```

### 2B. City Pages (setup/tucson.html, setup/phoenix.html)

Add to each city page's `<head>`:

```html
<!-- Schema: LocalBusiness (City-Specific) -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Run Your Own AI — [CITY] OpenClaw Setup",
  "description": "Professional OpenClaw AI assistant setup in [CITY], Arizona. In-person or remote installation.",
  "url": "https://runyourownai.com/setup/[city-slug]",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "[CITY]",
    "addressRegion": "AZ",
    "addressCountry": "US"
  },
  "areaServed": {"@type": "City", "name": "[CITY]"},
  "priceRange": "$449-$1497",
  "parentOrganization": {
    "@type": "Organization",
    "name": "Run Your Own AI",
    "url": "https://runyourownai.com"
  }
}
</script>
```

Replace `[CITY]` and `[city-slug]` per page.

### 2C. Blog Posts — Schema Templates

**Security post** — use `Article` + `FAQPage`:
```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Is OpenClaw Safe in 2026? Security Guide After the Breaches",
  "author": {"@type": "Person", "name": "Seth"},
  "publisher": {"@type": "Organization", "name": "Run Your Own AI"},
  "datePublished": "2026-03-22",
  "description": "Complete guide to OpenClaw security in 2026, covering the ClawHavoc breach, CVE-2026-25253, and how to harden your installation.",
  "mainEntityOfPage": "https://runyourownai.com/blog/is-openclaw-safe"
}
</script>
```

**Cost post** — use `Article`:
```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "OpenClaw Setup Cost 2026: Complete Price Breakdown",
  "author": {"@type": "Person", "name": "Seth"},
  "publisher": {"@type": "Organization", "name": "Run Your Own AI"},
  "datePublished": "2026-03-22",
  "description": "Full breakdown of OpenClaw costs in 2026: DIY vs professional setup, VPS hosting, API fees, and which service gives the best value."
}
</script>
```

**Install guide** — use `HowTo`:
```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "How to Install OpenClaw: The Complete Guide",
  "description": "Step-by-step guide to installing OpenClaw on a VPS, Mac, or Raspberry Pi in 2026.",
  "totalTime": "PT2H",
  "estimatedCost": {
    "@type": "MonetaryAmount",
    "currency": "USD",
    "value": "0"
  },
  "step": [
    {"@type": "HowToStep", "name": "Choose Your Hardware", "text": "Select a VPS provider, Mac Mini, or Raspberry Pi for your OpenClaw server."},
    {"@type": "HowToStep", "name": "Set Up the Server", "text": "Provision your VPS, create a user, configure SSH keys, and set up a firewall."},
    {"@type": "HowToStep", "name": "Install Node.js", "text": "Install Node.js v22 using nvm or your system package manager."},
    {"@type": "HowToStep", "name": "Install OpenClaw", "text": "Run npm install -g openclaw to install the OpenClaw CLI globally."},
    {"@type": "HowToStep", "name": "Configure API Keys", "text": "Set up your AI provider API keys (Anthropic, OpenAI, or Google) and messaging channel tokens."},
    {"@type": "HowToStep", "name": "Connect Your Channels", "text": "Configure Telegram, Discord, WhatsApp, or Signal integration."},
    {"@type": "HowToStep", "name": "Harden Security", "text": "Enable firewall rules, set up auto-updates, configure SSH hardening."},
    {"@type": "HowToStep", "name": "Test and Launch", "text": "Send a test message through your connected channel and verify all features work."}
  ]
}
</script>
```

---

## PART 3: Google Search Console Submission

### Step-by-Step (Seth does this manually)

1. **Go to** [Google Search Console](https://search.google.com/search-console/)
2. **Sign in** with the Google account you want to manage the property
3. **Add Property** → Select "URL prefix" → Enter `https://runyourownai.com`
4. **Verify ownership** — Recommended method for Vercel: **HTML file upload**
   - Download the `google[code].html` verification file Google provides
   - Place it in `/site/google[code].html`
   - Deploy to Vercel: `vercel --yes --prod --token <token>`
   - Click "Verify" in GSC
   - **Alternative:** DNS TXT record via Cloudflare (since domain is on Cloudflare):
     - Go to Cloudflare → DNS → Add TXT record with the value Google provides
     - This doesn't require a deploy
5. **Once verified:**
   - Go to **Sitemaps** → Submit `https://runyourownai.com/sitemap.xml`
   - Go to **URL Inspection** → Inspect the homepage → Click "Request Indexing"
   - Do the same for each existing page:
     - `https://runyourownai.com/what-is-openclaw`
     - `https://runyourownai.com/openclaw-vs-chatgpt`
     - `https://runyourownai.com/setup/tucson`
     - `https://runyourownai.com/setup/phoenix`
6. **Set up email alerts** — GSC will notify you of indexing issues, coverage errors, etc.
7. **Check back in 48-72 hours** for initial indexing data

### Bonus: Bing Webmaster Tools
- Go to [Bing Webmaster Tools](https://www.bing.com/webmasters/)
- Can import directly from GSC (one click)
- Submit same sitemap

---

## PART 4: Blog Post File Structure & Conventions

### URL Pattern
```
/blog/[slug]
```
Maps to file: `/site/blog/[slug].html`

### Directory Structure
```
/site/
├── index.html
├── sitemap.xml
├── robots.txt
├── vercel.json
├── what-is-openclaw.html
├── openclaw-vs-chatgpt.html
├── setup/
│   ├── tucson.html
│   └── phoenix.html
└── blog/
    ├── is-openclaw-safe.html
    ├── openclaw-setup-cost.html
    └── how-to-install-openclaw.html
```

### Blog HTML Template

Each blog post should follow this template (matching the existing site's design language — dark theme, Phosphor icons, same fonts/CSS):

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[META TITLE]</title>
  <meta name="description" content="[META DESCRIPTION]">
  <meta name="keywords" content="[TARGET KEYWORDS]">
  <link rel="canonical" href="https://runyourownai.com/blog/[slug]">
  
  <!-- Open Graph -->
  <meta property="og:title" content="[META TITLE]">
  <meta property="og:description" content="[META DESCRIPTION]">
  <meta property="og:url" content="https://runyourownai.com/blog/[slug]">
  <meta property="og:type" content="article">
  <meta property="og:site_name" content="Run Your Own AI">
  
  <!-- Schema (Article or HowTo - see Part 2C) -->
  <script type="application/ld+json">
  { ... }
  </script>
  
  <!-- Same CSS/fonts as main site -->
  [LINK TO EXISTING STYLESHEETS]
</head>
<body>
  <!-- Same nav as main site -->
  <nav>...</nav>
  
  <article class="blog-post">
    <header class="blog-header">
      <p class="blog-date">March 22, 2026</p>
      <h1>[POST TITLE]</h1>
      <p class="blog-subtitle">[SUBTITLE/HOOK]</p>
    </header>
    
    <div class="blog-content">
      <!-- Post body with H2/H3 structure -->
    </div>
    
    <!-- CTA Section -->
    <div class="blog-cta">
      <h2>Skip the Headache. We'll Set It Up for You.</h2>
      <p>Professional OpenClaw setup starting at $449. Running in 24 hours.</p>
      <a href="/#contact" class="btn-primary">Get My AI Assistant →</a>
    </div>
  </article>
  
  <!-- Same footer as main site -->
  <footer>...</footer>
</body>
</html>
```

---

## PART 5: Blog Post #1 — "Is OpenClaw Safe in 2026?"

### File
`/site/blog/is-openclaw-safe.html`

### URL
`https://runyourownai.com/blog/is-openclaw-safe`

### Target Keywords
- Primary: `is openclaw safe`, `openclaw security`
- Secondary: `openclaw security issues 2026`, `openclaw safe to use`, `openclaw breach`, `openclaw clawhavoc`
- Long-tail: `is openclaw safe after the breaches`, `openclaw security best practices 2026`, `openclaw exposed instances`

### Meta Title
`Is OpenClaw Safe in 2026? Security Guide After the Breaches | Run Your Own AI`

### Meta Description
`The ClawHavoc breach exposed 21,000+ instances. Is OpenClaw still safe? Yes — if set up correctly. Complete 2026 security guide with hardening checklist and what went wrong.`

### Word Count Target
**2,500–3,000 words**

### Internal Linking Strategy
- Link to homepage pricing section (CTA: "professional setup = secure setup")
- Link to `/blog/how-to-install-openclaw` (when mentioning DIY risks)
- Link to `/blog/openclaw-setup-cost` (when mentioning cost of getting it wrong)
- Link to `/what-is-openclaw` (for readers who landed here first)

### Full Outline

```
H1: Is OpenClaw Safe in 2026? The Complete Security Guide After the Breaches

  Introduction (200 words)
  - Hook: "21,000 exposed OpenClaw instances. Headlines screaming about breaches. So is OpenClaw actually safe?"
  - Thesis: OpenClaw is safe. Bad setups aren't. Here's the difference.
  - What this guide covers

H2: What Actually Happened: The 2026 OpenClaw Security Incidents
  H3: The ClawHavoc Breach (March 2026)
    - What happened: thousands of instances exposed to the internet with no auth
    - Root cause: default configs left unchanged, no firewall, no SSH hardening
    - NOT an OpenClaw vulnerability — a user configuration failure
  H3: CVE-2026-25253: The Real Vulnerability
    - What it was (brief, factual)
    - How it was patched
    - How quickly the OpenClaw team responded
  H3: The 21,000 Exposed Instances
    - Shodan scan results
    - Most were hobby installs with zero security
    - What "exposed" actually means vs. "hacked"

H2: Is OpenClaw Itself Secure?
  H3: Open Source = Auditable
    - 135K+ GitHub stars, massive community review
    - OpenAI acquisition adds security resources
  H3: The Architecture Is Sound
    - Runs on YOUR server (not a shared cloud)
    - No data leaves your machine unless you configure it to
    - Model API calls are the only external communication
  H3: NemoClaw: NVIDIA's Enterprise Security Layer
    - Brief mention of NemoClaw (launched March 17)
    - What it adds: sandboxing, guardrails
    - Who it's for (enterprise vs. personal)

H2: The 10-Step OpenClaw Security Hardening Checklist
  H3: 1. Never Use Default Credentials
  H3: 2. SSH Key Authentication Only (Disable Password Login)
  H3: 3. Configure UFW/iptables Firewall
  H3: 4. Run OpenClaw Behind a Reverse Proxy (Nginx/Caddy)
  H3: 5. Enable Automatic Security Updates
  H3: 6. Use a Dedicated Non-Root User
  H3: 7. Set Up Fail2Ban
  H3: 8. Keep OpenClaw Updated (npm update)
  H3: 9. Limit API Key Permissions & Spending Caps
  H3: 10. Regular Backup Schedule
  (Each step: 2-3 sentences explaining what and why)

H2: DIY vs. Professional Setup: The Security Gap
  - Most DIY installs skip 7+ of the 10 steps above
  - Time investment: 4-8 hours for someone who knows what they're doing
  - Cost of getting it wrong: data exposure, API key theft, compute hijacking
  - [SOFT CTA] "We do all 10 steps on every install, plus monitoring. Starting at $449."

H2: Who Should NOT Self-Install OpenClaw
  - If you don't know what SSH is
  - If you've never configured a firewall
  - If you're storing sensitive data (business, medical, legal)
  - If you don't have time to maintain and update it
  - [LINK to /blog/how-to-install-openclaw]: "If you DO want to try it yourself, here's our complete guide"

H2: The Bottom Line: Safe Product, Risky Installs
  - Summary: OpenClaw is as safe as your setup
  - The breaches were all configuration failures
  - Professional setup eliminates the #1 risk factor

  CTA Section:
  - "Don't become a statistic. Get a properly secured OpenClaw install."
  - Button: "Get Secure Setup — Starting at $449 →" → /#contact
```

---

## PART 6: Blog Post #2 — "OpenClaw Setup Cost 2026"

### File
`/site/blog/openclaw-setup-cost.html`

### URL
`https://runyourownai.com/blog/openclaw-setup-cost`

### Target Keywords
- Primary: `openclaw setup cost`, `openclaw cost`, `how much does openclaw cost`
- Secondary: `openclaw pricing 2026`, `openclaw monthly cost`, `openclaw api cost`, `openclaw setup price`
- Long-tail: `openclaw setup cost breakdown 2026`, `openclaw vps cost`, `openclaw running costs`, `cheapest way to run openclaw`

### Meta Title
`OpenClaw Setup Cost 2026: Complete Price Breakdown (DIY vs Pro) | Run Your Own AI`

### Meta Description
`What does OpenClaw actually cost in 2026? Full breakdown: DIY ($0 setup + $10-35/mo), professional setup ($249-$6,000), VPS hosting, API fees, and hidden costs most guides skip.`

### Word Count Target
**2,800–3,200 words**

### Internal Linking Strategy
- Link to homepage pricing section (anchor to #pricing)
- Link to `/blog/is-openclaw-safe` (when discussing security costs/risks of cheap setup)
- Link to `/blog/how-to-install-openclaw` (when covering DIY path)
- Link to `/what-is-openclaw` (early mention for context)
- Link to `/setup/tucson` and `/setup/phoenix` (when mentioning local setup option)

### Full Outline

```
H1: OpenClaw Setup Cost 2026: The Complete Price Breakdown

  Introduction (200 words)
  - Hook: "Everyone asks how much OpenClaw costs. Nobody gives a straight answer. Here's every dollar, broken down."
  - Promise: DIY costs, professional setup costs, ongoing monthly costs, hidden costs
  - No affiliate links, no BS

H2: The Short Answer (TL;DR Table)
  - Table format:
    | Path | Setup Cost | Monthly Cost | Total Year 1 |
    | DIY | $0 | $10-35 | $120-420 |
    | Freelancer (Fiverr) | $75-400 | $10-35 | $195-820 |
    | ClawSetup.ai | $249-1,199 | $10-35 | $369-1,619 |
    | Run Your Own AI | $449-1,497 | $10-35 | $569-1,917 |
    | SetupClaw.com | $3,000-6,000 | $10-35 | $3,120-6,420 |
  - "Now let's break down what you actually GET for each price."

H2: Cost #1: The Server (VPS Hosting)
  H3: VPS Options & Pricing
    - Hetzner: $4-8/mo (best value, EU/US)
    - DigitalOcean: $6-12/mo (good docs)
    - Linode/Akamai: $5-10/mo
    - AWS Lightsail: $5-10/mo
    - Oracle Cloud Free Tier: $0 (limited)
  H3: Run It on Your Own Hardware (Free)
    - Mac Mini: ~$599 one-time, $0/mo
    - Old laptop: $0
    - Raspberry Pi 5: ~$80 one-time, $0/mo
    - Tradeoff: uptime, reliability, noise, power cost (~$3-5/mo electricity)
  H3: Our Recommendation
    - Hetzner CX22 ($4.50/mo) for most people
    - Mac Mini if you want iMessage integration
    - [SOFT CTA]: "We provision and configure your VPS as part of every setup"

H2: Cost #2: AI Provider API Fees
  H3: How AI Pricing Works (Tokens Explained Simply)
    - What a token is (in plain English)
    - Input vs output pricing
  H3: Model Pricing Comparison (March 2026)
    - Table: Claude Opus 4 / Claude Sonnet 4 / GPT-4.1 / GPT-4.1 mini / Gemini 2.5 Pro
    - Cost per 1M tokens for each
  H3: What Real Monthly Usage Looks Like
    - Light user (10-20 messages/day): $5-12/mo
    - Medium user (30-50 messages/day): $12-25/mo
    - Power user (100+ messages/day): $25-50/mo
  H3: The Model Routing Trick That Saves 60-80%
    - Route simple tasks to cheap models (Sonnet, GPT-4.1 mini)
    - Route complex tasks to expensive models (Opus, GPT-4.1)
    - OpenRouter as a single API for all providers
    - [SOFT CTA]: "We configure model routing on every Pro and Local install"

H2: Cost #3: Setup — DIY vs. Professional
  H3: DIY Setup: $0 (But 6-20 Hours of Your Time)
    - What you need to know: Linux, SSH, Node.js, API configuration
    - Realistic time investment: 6-20 hours for first-timers
    - Hidden cost: your time × your hourly rate
    - Risk: security gaps if you miss hardening steps
    - [LINK to /blog/how-to-install-openclaw]
  H3: Freelancer/Fiverr: $75-400
    - Pros: cheap upfront
    - Cons: inconsistent quality, no ongoing support, security unknown
    - Risk: who did you just give server access to?
  H3: ClawSetup.ai: $249-1,199
    - NYC-based, multiple tiers
    - Profile-specific setups (EA, Content Creator, etc.)
    - Limited blog/documentation
  H3: Run Your Own AI (Us): $449-1,497
    - What each tier includes (brief — link to full pricing page)
    - Seth does it personally
    - 100% money-back guarantee
    - Security hardening included on every tier
    - [CTA]: "See our full pricing →" → /#pricing
  H3: SetupClaw.com: $3,000-6,000
    - SF-based, enterprise-focused
    - "Collison Install" (they come to you)
    - Great service, 3-6x the price for similar result
    - Makes sense if you're running a 50-person team

H2: Hidden Costs Nobody Talks About
  H3: Domain & SSL (Optional but Recommended)
    - $10-15/year for a domain
    - SSL is free with Cloudflare or Let's Encrypt
  H3: Backup & Monitoring
    - VPS backups: $1-3/mo extra
    - Uptime monitoring: free (UptimeRobot)
  H3: Maintenance & Updates
    - OpenClaw updates frequently
    - Security patches need to be applied
    - Time cost: 30-60 min/month if DIY
    - [SOFT CTA]: "Our $149/mo support plan handles all maintenance"
  H3: The Cost of Getting It Wrong
    - Exposed API keys = surprise $500+ bill
    - Unsecured server = cryptominer
    - [LINK to /blog/is-openclaw-safe]

H2: Total Cost of Ownership: Year 1 Scenarios
  - Scenario 1: Budget DIY — $120-180/year
  - Scenario 2: DIY + Good VPS — $200-420/year
  - Scenario 3: Professional Setup (us) + VPS — $569-1,177/year
  - Scenario 4: Premium Setup (us) + Support — $1,266-2,325/year
  - "Compare to ChatGPT Plus at $240/year — and OpenClaw does 10x more"

H2: Which Option Is Right for You?
  - Decision tree / flowchart (text-based):
    - "Are you comfortable with Linux?" → Yes → DIY might work → but read the security guide first
    - "Is your time worth more than $449?" → Yes → hire us
    - "Do you need it for business?" → Yes → Pro or Local tier
    - "Are you in Tucson/Phoenix?" → Yes → Local tier, we come to you

  CTA Section:
  - "Know what you need? Let's get started."
  - Button: "See Pricing & Book Setup →" → /#pricing
```

---

## PART 7: Blog Post #3 — "How to Install OpenClaw"

### File
`/site/blog/how-to-install-openclaw.html`

### URL
`https://runyourownai.com/blog/how-to-install-openclaw`

### Target Keywords
- Primary: `how to install openclaw`, `openclaw installation guide`
- Secondary: `openclaw setup guide`, `openclaw tutorial`, `install openclaw 2026`, `openclaw setup tutorial`
- Long-tail: `how to install openclaw on vps`, `openclaw step by step guide`, `openclaw beginner guide 2026`, `openclaw ubuntu setup`

### Meta Title
`How to Install OpenClaw: The Complete 2026 Guide (Step-by-Step) | Run Your Own AI`

### Meta Description
`Step-by-step OpenClaw installation guide for 2026. Covers VPS setup, Node.js, API keys, Telegram/Discord, security hardening. Works on Ubuntu, Mac, and Raspberry Pi.`

### Word Count Target
**3,000–3,500 words** (longest post — needs to be genuinely comprehensive to rank)

### Internal Linking Strategy
- Link to homepage pricing section (CTAs throughout: "Want us to do this for you?")
- Link to `/blog/is-openclaw-safe` (when covering security hardening section)
- Link to `/blog/openclaw-setup-cost` (when mentioning VPS costs, API costs)
- Link to `/what-is-openclaw` (early context link)
- Link to `/setup/tucson` and `/setup/phoenix` (local setup alternative)

### Full Outline

```
H1: How to Install OpenClaw: The Complete 2026 Guide

  Introduction (250 words)
  - Hook: "OpenClaw is the most capable AI assistant you can self-host. Installing it? That's the hard part."
  - What this guide covers: full install from zero to working AI on your phone
  - Time estimate: 2-4 hours for someone comfortable with terminals; 6-20 hours for beginners
  - Prerequisites: a computer with internet access, a credit card for API keys
  - [SOFT CTA box]: "Want to skip all this? We install OpenClaw for $449. Running in 24 hours." → /#contact

H2: Before You Start: What You'll Need
  H3: Hardware Options
    - VPS (recommended): Hetzner, DigitalOcean, Linode — $5-12/mo
    - Mac Mini / old laptop (free, but less reliable uptime)
    - Raspberry Pi 5 (works but slower)
    - Minimum specs: 2GB RAM, 2 vCPU, 20GB storage
    - [LINK to /blog/openclaw-setup-cost for full cost breakdown]
  H3: Software Requirements
    - OS: Ubuntu 22.04+ (recommended), macOS, Debian
    - Node.js v22+
    - npm
    - A terminal/SSH client
  H3: Accounts You'll Need
    - AI provider: Anthropic (Claude) OR OpenAI (GPT-4) OR Google (Gemini) — pick one
    - OpenRouter (optional but recommended — single API for all providers)
    - Telegram account (for your bot) OR Discord server
    - (Optional) Cloudflare account for DNS

H2: Step 1 — Provision Your Server
  H3: Option A: VPS (Recommended)
    - Sign up for Hetzner/DigitalOcean
    - Create a server: Ubuntu 22.04, 2GB RAM, nearest region
    - Note the IP address
    - SSH in: `ssh root@YOUR_IP`
  H3: Option B: Mac or Local Machine
    - Open Terminal
    - Ensure you're running macOS 13+ or Ubuntu 22.04+
  H3: Option C: Raspberry Pi
    - Flash Ubuntu Server 22.04 to SD card
    - Boot and SSH in

H2: Step 2 — Secure the Server (Don't Skip This)
  H3: Create a Non-Root User
    ```bash
    adduser openclaw
    usermod -aG sudo openclaw
    su - openclaw
    ```
  H3: Set Up SSH Key Authentication
    - Generate key pair on your local machine
    - Copy public key to server
    - Disable password authentication in sshd_config
  H3: Configure the Firewall
    ```bash
    sudo ufw allow OpenSSH
    sudo ufw enable
    sudo ufw status
    ```
  H3: Enable Automatic Updates
    ```bash
    sudo apt install unattended-upgrades
    sudo dpkg-reconfigure unattended-upgrades
    ```
  - [LINK to /blog/is-openclaw-safe]: "See our full 10-step hardening checklist"
  - [SOFT CTA]: "This is the step most DIY installs skip — and why 21,000 instances got exposed."

H2: Step 3 — Install Node.js
  ```bash
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
  source ~/.bashrc
  nvm install 22
  node --version  # Should show v22.x.x
  ```
  - Troubleshooting: common PATH issues, nvm not found

H2: Step 4 — Install OpenClaw
  ```bash
  npm install -g openclaw
  openclaw --version
  ```
  H3: Initial Configuration
    ```bash
    openclaw init
    ```
  - Walk through the interactive setup prompts
  - What each option means

H2: Step 5 — Configure Your AI Provider
  H3: Option A: Anthropic (Claude) — Recommended
    - Go to console.anthropic.com → Create account → Generate API key
    - Add to OpenClaw config
    - Set spending limit ($10-25/mo recommended to start)
  H3: Option B: OpenAI (GPT-4)
    - Go to platform.openai.com → API keys
    - Add to config
  H3: Option C: OpenRouter (Access All Models)
    - Go to openrouter.ai → Create account → API key
    - Benefits: one key, all providers, automatic failover
  H3: Model Routing (Save Money)
    - Configure cheap models for simple tasks
    - Reserve expensive models for complex reasoning
    - [LINK to /blog/openclaw-setup-cost]: "See our full cost optimization breakdown"

H2: Step 6 — Connect Your Messaging Channel
  H3: Option A: Telegram (Most Popular)
    - Open Telegram → message @BotFather
    - `/newbot` → follow prompts → get bot token
    - Add token to OpenClaw config
    - Start a chat with your bot
  H3: Option B: Discord
    - Go to Discord Developer Portal
    - Create application → Add bot → Get token
    - Set intents (Message Content, etc.)
    - Invite bot to your server
    - Add token to config
  H3: Option C: WhatsApp
    - Requires WhatsApp Business API or Baileys integration
    - More complex setup — brief overview + link to docs

H2: Step 7 — Start OpenClaw
  ```bash
  openclaw start
  ```
  H3: Running as a Background Service (PM2)
    ```bash
    npm install -g pm2
    pm2 start openclaw -- start
    pm2 save
    pm2 startup
    ```
  - Verify it starts on reboot
  H3: Verify Everything Works
    - Send a message to your bot on Telegram/Discord
    - Check logs: `pm2 logs openclaw`
    - Common issues and fixes

H2: Step 8 — Customize Your Assistant
  H3: Set Up Your Persona
    - Edit SOUL.md for personality
    - Edit USER.md with your info
  H3: Enable Skills
    - Web search
    - Browser control
    - File management
    - Memory system
  H3: Configure Memory
    - How OpenClaw's memory works
    - Setting up persistent context

H2: What Can Go Wrong (And How to Fix It)
  H3: "npm ERR! permission denied"
    - Solution: use nvm, never `sudo npm install -g`
  H3: "Error: Cannot find module 'openclaw'"
    - Solution: PATH issues, nvm activation
  H3: "Telegram bot not responding"
    - Solution: check bot token, check firewall, check PM2 status
  H3: "High API costs"
    - Solution: model routing, spending caps
    - [LINK to /blog/openclaw-setup-cost]
  H3: "Server exposed / security concerns"
    - Solution: go back to Step 2
    - [LINK to /blog/is-openclaw-safe]

H2: Want to Skip All This?
  - Honest summary: this guide works, but it's 2-4 hours minimum
  - Most people get stuck on Steps 2, 5, or 6
  - "We've done this hundreds of times. We can have you running in 24 hours."
  - Pricing: $449 Starter, $997 Pro, $1,497 Local (Tucson/Phoenix)
  - Every install includes the full security hardening checklist

  CTA Section:
  - "Your time has a cost. Our setup fee pays for itself."
  - Button: "Get Professional Setup — $449 →" → /#contact
  - Secondary: "Or keep going — you've got this. 💪"
```

---

## PART 8: Deployment Checklist

After creating all files, deploy and verify in this order:

### Deploy Steps
```bash
cd /home/openclaw/.openclaw/workspace/clawsetup-site

# 1. Verify all files exist
ls -la sitemap.xml robots.txt vercel.json index.html
ls -la blog/is-openclaw-safe.html blog/openclaw-setup-cost.html blog/how-to-install-openclaw.html

# 2. Deploy
vercel --yes --prod --token $(cat ~/.openclaw/secrets/vercel.json | jq -r '.token')

# 3. Verify sitemap
curl -s https://runyourownai.com/sitemap.xml | head -20

# 4. Verify robots.txt has sitemap directive
curl -s https://runyourownai.com/robots.txt | grep -i sitemap

# 5. Verify each blog post returns 200
curl -I https://runyourownai.com/blog/is-openclaw-safe
curl -I https://runyourownai.com/blog/openclaw-setup-cost
curl -I https://runyourownai.com/blog/how-to-install-openclaw

# 6. Validate schema markup (paste each URL into):
# https://validator.schema.org/
# https://search.google.com/test/rich-results

# 7. Submit to Google Search Console (manual — see Part 3)
```

### Post-Deploy
- [ ] Sitemap returns 200 with XML content
- [ ] robots.txt includes Sitemap directive
- [ ] All 3 blog posts return 200
- [ ] Schema validates with zero errors in Rich Results Test
- [ ] GSC property verified
- [ ] Sitemap submitted in GSC
- [ ] All new URLs submitted for indexing via URL Inspection
- [ ] Update `sitemap.xml` with any additional pages (city pages, etc.)

---

## Quick Reference: URL Slug Map

| Content | URL | File Path | Priority |
|---------|-----|-----------|----------|
| Homepage | `/` | `/site/index.html` | 1.0 |
| What is OpenClaw | `/what-is-openclaw` | `/site/what-is-openclaw.html` | 0.8 |
| OpenClaw vs ChatGPT | `/openclaw-vs-chatgpt` | `/site/openclaw-vs-chatgpt.html` | 0.8 |
| Tucson Setup | `/setup/tucson` | `/site/setup/tucson.html` | 0.8 |
| Phoenix Setup | `/setup/phoenix` | `/site/setup/phoenix.html` | 0.8 |
| Blog: Security Guide | `/blog/is-openclaw-safe` | `/site/blog/is-openclaw-safe.html` | 0.7 |
| Blog: Cost Breakdown | `/blog/openclaw-setup-cost` | `/site/blog/openclaw-setup-cost.html` | 0.7 |
| Blog: Install Guide | `/blog/how-to-install-openclaw` | `/site/blog/how-to-install-openclaw.html` | 0.7 |

---

*This playbook is ready to execute. Start with the prerequisite (recover source files), then work through Parts 1-8 in order. Total time: 6-8 hours for all 3 blog posts + technical SEO fixes.*
