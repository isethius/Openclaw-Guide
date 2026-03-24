# ClawHQ Marketing Attack Plan v2
## Updated: 2026-03-23 (with reasoning enabled)

---

## WHAT WE ACCOMPLISHED (March 22)
- [x] ~~Fix sitemap.xml~~ → LIVE, returning 200
- [x] ~~Publish 3 blog posts~~ → All 3 LIVE (security, cost, install guide)
- [x] ~~Comparison page~~ → LIVE at /compare
- [x] ~~4 niche pages~~ → LIVE (real estate, crypto, freelancers, local business)
- [x] ~~robots.txt + vercel.json~~ → LIVE
- [x] ~~Update Claude model reference~~ → Now shows Opus 4.6
- [x] ~~Remove cheap loading screen~~ → Done (but hero animation needs verification on mobile)
- Site went from 5 pages → 16 pages in one session

---

## CRITICAL GAPS IDENTIFIED (v2 additions)

### 1. 🔴 Schema Markup Still Missing
Homepage has ZERO JSON-LD structured data. We built the schema blocks but never injected them into index.html. This means:
- No rich snippets in Google (FAQ dropdowns, pricing, star ratings)
- Missing LocalBusiness schema for Tucson/Phoenix local SEO
- Competitors with schema will outrank us in rich results

### 2. 🔴 Homepage Mobile Rendering May Be Broken
The loader removal caused the hero section to not render on mobile (content stays opacity:0). We patched startHero() to fire after 100ms but haven't confirmed the fix works. **Must verify before any marketing push** — can't drive traffic to a broken page.

### 3. 🔴 No Intake Form / Conversion Path
Every page has CTAs pointing to "/" but the homepage has no clear intake form or booking flow. The plan mentioned Tally.so + Calendly but neither is wired up. **We're driving traffic to a page that can't convert.**

### 4. 🟠 Internal Linking Is Weak
New pages link to each other in footers, but the homepage doesn't link to any of them. Need:
- "Built for Your Work" section on homepage → links to 4 niche pages
- "Cities We Serve" footer → links to all city pages
- Blog preview section on homepage
- Each blog post should cross-link to related niche pages

### 5. 🟠 NemoClaw Is a HUGE Content Opportunity
NVIDIA announced NemoClaw at GTC on March 16 — literally 1 week ago. It's an enterprise wrapper around OpenClaw with security/sandbox features. This changes the landscape:
- Blog post: "OpenClaw vs NemoClaw: Which Do You Need?" — ZERO competition right now
- Positioning: NemoClaw is for enterprises with DGX hardware. We're for individuals and small businesses who want OpenClaw without $10K in NVIDIA hardware.
- This is potentially our highest-traffic blog post because people are actively searching

### 6. 🟠 Google Search Console Not Submitted
Sitemap exists but Google doesn't know about it yet. Seth needs to:
- Verify runyourownai.com in Search Console (DNS TXT via Cloudflare)
- Submit sitemap URL
- Request indexing of key pages

---

## REVISED PHASE 0: EMERGENCY FIXES (Today)

### Seth Does (requires his accounts):
- [ ] Submit to Google Search Console (verify via Cloudflare DNS)
- [ ] Create @RunYourOwnAI Twitter/X account
- [ ] Claim r/openclaw subreddit
- [ ] Set up Tally.so intake form OR simple Calendly booking link

### Kito Does:
- [ ] Inject schema markup into homepage (Organization, Service, LocalBusiness, FAQPage)
- [ ] Verify mobile homepage rendering is fixed
- [ ] Add schema to blog posts and niche pages
- [ ] Build 8 city pages (Austin, Denver, Miami, Vegas, San Diego, Nashville, Portland, Raleigh)
- [ ] Write NemoClaw comparison blog post (timely, zero competition)

---

## REVISED PHASE 1: FOUNDATION (This Week)

### Site Improvements
- [ ] Add intake form/booking CTA to homepage (Tally.so embed or Calendly link)
- [ ] Add "Built for Your Work" section to homepage → 4 niche page cards
- [ ] Add "Cities We Serve" to homepage footer
- [ ] Internal linking audit: every page cross-links to 2-3 related pages
- [ ] Add blog preview/listing section to homepage

### New Content (High-Priority)
- [ ] Blog: "OpenClaw vs NemoClaw: Which Do You Need?" (ride the GTC hype)
- [ ] Blog: "SetupClaw vs ClawSetup vs RunYourOwnAI" (already have /compare page, make a blog version for SEO)
- [ ] 8 city pages deployed

### Social (Seth's Manual Work)
- [ ] Twitter profile live with Kito branding
- [ ] Join OpenClaw Discord, start helping in #users-helping-users
- [ ] Reddit: subscribe to target subs, start commenting (no promo yet)

---

## PHASE 2: CONTENT ENGINE (Week 2-3)
*Unchanged from v1 — the strategy is sound. Key addition:*

- [ ] Blog: "OpenClaw for Small Business: 5 Automations That Replace a VA"
- [ ] Blog: "Is OpenClaw Worth It? Honest Review After 3 Months"
- [ ] Track NemoClaw announcements — every update is a blog post opportunity
- [ ] A/B test CTA copy on homepage (track which converts better)

---

## PHASE 3: SCALE (Week 4-8)
*Unchanged from v1. YouTube + expanded social.*

New addition:
- [ ] Create a "Free OpenClaw Security Audit" landing page (lead magnet)
- [ ] Consider NemoClaw setup as premium offering ($2K+) for businesses with NVIDIA hardware

---

## PHASE 4: CONVERT (Week 9-12)
*Unchanged from v1.*

New addition:
- [ ] Referral program: existing clients get $50 credit for referrals
- [ ] "OpenClaw Setup Calculator" interactive tool (input your use case → get recommended plan)

---

## CONTENT PRIORITY STACK (What to Write Next)

| Priority | Content | Why |
|----------|---------|-----|
| 🔴 1 | NemoClaw vs OpenClaw blog | Zero competition, GTC hype, timely |
| 🔴 2 | 8 city pages | SEO compound effect, SetupClaw has 10 |
| 🟠 3 | Homepage sections (niche cards, cities footer) | Internal linking + conversion |
| 🟠 4 | Schema markup injection | Rich snippets in search |
| 🟡 5 | "Is OpenClaw Worth It?" review blog | Long-tail SEO |
| 🟡 6 | Intake form integration | Convert existing traffic |
| 🟢 7 | VPS affiliate content | Revenue diversification |

---

## KEY STRATEGIC INSIGHT (v2)

**NemoClaw changes our positioning.** Before, we were "OpenClaw setup service." Now we can be "OpenClaw setup for humans, NemoClaw consulting for businesses." 

- OpenClaw personal setup: $449-$1,497 (our existing packages)
- NemoClaw enterprise consulting: $2,000-$5,000 (new tier, competes with SetupClaw pricing but with NVIDIA expertise)

This lets us play both ends — affordable personal AI AND premium enterprise — without changing our core offering. The NemoClaw blog post tests this positioning.

---

## METRICS (Same as v1, adding):
- Google Search Console: impressions, clicks, average position
- Page-level conversion tracking (which pages drive form submissions)
- NemoClaw keyword rankings
