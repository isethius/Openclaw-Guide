# ClawHQ.com Intake Form — Complete Specification
**Research Date:** 2026-03-07  
**Purpose:** Primary conversion point for OpenClaw installation service ($349–$997)  
**Goal:** Qualify leads, give Seth pre-call intel, close more deals

---

## 1. RESEARCH FINDINGS SUMMARY

### Multi-Step vs. Single Page — Verdict: **3-Step Multi-Step Wins**
- Multi-step forms average **13.85% vs 4.53%** conversion rate for complex lead-gen (+206%)
- HubSpot data: multi-step converts **86% higher** on average than single-page
- Real case study: 11-field single-page → 3 steps = **0.96% to 8.1%** (+743%)
- For $300–$1,000 services with 7+ fields needed: **3-step is the clear winner**
- Optimal: Step 1 ultra-light → Step 2 qualify → Step 3 schedule/confirm

### Progressive Disclosure — Verdict: **Show Pricing Before the Form**
- Hiding pricing increases unqualified leads and wastes Seth's time
- Atlassian transparent pricing: **25% more qualified leads + faster close**
- Pre-form pricing lets people self-qualify → only serious buyers fill it out
- Strategy: Show tiers (Starter/Pro/Elite) on the page, THEN the form below
- Clears the air — people who fill it out already know what they're getting into

### Phone Number — Verdict: **Optional, Not Required**
- Required phone number: **18.7% drop in conversions** (largest friction field)
- 4.1% decrease per additional field — every field costs you leads
- Make phone OPTIONAL with "fastest way to reach you" framing
- At Step 3 (post-commitment), phone friction is much lower than at Step 1

### Calendly vs. Manual Follow-up — Verdict: **Hybrid (Form → Calendly Redirect)**
- Embedded Calendly after form: **30–70% more meetings booked**
- Manual follow-up: introduces delay, people lose interest or book elsewhere
- Best pattern for ClawHQ: Short form → Thank you page with Calendly embed
- Keeps Seth in control while converting while interest is HOT

### Form Platform — Verdict: **Tally (free) or Typeform (premium)**
- **Tally**: Free, unlimited responses, Notion-style, conditional logic, Zapier support — **use this to launch fast**
- **Typeform**: Premium "one question at a time" UX, better for paid traffic funnels, ~$29/mo — upgrade if conversion matters more than cost
- **Formspree**: Developer backend only — skip unless building custom HTML form
- **Native HTML + Formspree**: Good if ClawHQ has a real developer and wants zero branding constraints

### Competitor — SetupClaw.com Analysis
- Targets: founders, exec teams, agencies, VCs — higher-ticket than ClawHQ
- NO intake form visible — just a Calendly link (`cal.com/michaelsf/setup-clawdbot`)
- After purchase → within 48hrs kickoff, Slack channel, live setup session
- Weakness: High friction to start (buy first, then figure it out)
- **ClawHQ opportunity**: Form BEFORE money = lower commitment, more leads

---

## 2. FORM STRUCTURE RECOMMENDATION

### Architecture: 3-Step with Calendly Redirect

```
Landing Page
  └── Pricing Section (visible before form)
      └── CTA: "Apply for Setup"
          └── STEP 1: Who Are You? (2 fields + device choice)
              └── STEP 2: Your Setup (channels + use cases + tech level)
                  └── STEP 3: Contact + Optional Phone
                      └── THANK YOU PAGE: Calendly embed + confirmation
                          └── AUTO-EMAIL: Sent immediately
```

---

## 3. COPY & FIELDS — COMPLETE SPECIFICATION

---

### PRE-FORM: PRICING SECTION (on landing page, above form)

**Headline:**  
> *"Choose your starting point — we'll customize everything on the call."*

| Tier | Price | Best For |
|------|-------|----------|
| **Starter** | $349 | Single device, Telegram + 1 channel, core setup |
| **Pro** | $597 | VPS + 2 channels + email + calendar + 3 workflows |
| **Elite** | $997 | VPS or Mac Mini + all channels + full automation + 14-day care |

*All plans include: 14-day hypercare, security hardening, and a working bot on day one.*

**CTA Button:**  
> `→ Apply for Your Setup`

---

### STEP 1 OF 3: "Let's start with the basics"

**Page headline:**  
> "Tell us about yourself"

**Subtext:**  
> "This takes 90 seconds. We'll reach out within 24 hours to schedule your setup."

**Progress indicator:** ● ○ ○ (Step 1 of 3)

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| First name | Text | ✅ | Keep it human |
| Email address | Email | ✅ | Primary contact |
| Which device do you want OpenClaw on? | Radio/Select | ✅ | See options below |
| Which tier are you most interested in? | Radio | ✅ | Pre-qualifies budget |

**Device options (radio buttons with icons):**
- 🖥️ Cloud VPS (most popular — we set it up remotely, ~$5-10/mo)
- 🍎 Mac Mini (need iMessage, prefer local hardware — ~$600 at cost)
- 💻 My laptop/desktop (I already have a machine I want to use)
- 🤔 Not sure — help me decide

**Tier options:**
- Starter ($349) — Just get me started
- Pro ($597) — Full setup + workflows ← **Mark as "Most Popular"**
- Elite ($997) — All in, full automation
- Not sure yet — I want to discuss

**CTA:** `Continue →`

**Abandonment note:** After email capture, run an abandon sequence even if they don't finish.

---

### STEP 2 OF 3: "Your Setup"

**Page headline:**  
> "How do you want to talk to your AI — and what should it do?"

**Subtext:**  
> "Pick everything that applies. No wrong answers — we'll dial it in on the call."

**Progress indicator:** ● ● ○ (Step 2 of 3)

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| Which messaging channels? | Checkboxes | ✅ | Multi-select |
| What do you want your AI to do? | Checkboxes | ✅ | Multi-select |
| How technical are you? | Radio | ✅ | Helps Seth calibrate |
| Anything else you want us to know? | Textarea | ❌ | Optional, short |

**Messaging channels (checkboxes — pick all that apply):**
- 📱 Telegram (most popular, works anywhere)
- 💬 Discord (great for community/team setups)
- 📲 WhatsApp (requires phone number on account)
- 🍎 iMessage (Mac Mini or Mac required)
- 🔔 Slack (team environments)
- 📧 Email only (no chat app needed)

**What do you want your AI to do? (checkboxes):**
- 📬 Monitor and summarize my email
- 📅 Manage my calendar / schedule meetings
- 🌐 Web search on demand
- 💻 Help me code / debug projects
- 📝 Draft messages, emails, and responses
- 📊 Track tasks and to-dos
- 🔔 Morning briefings (weather, news, agenda)
- 🤖 Run automated workflows while I sleep
- 🏠 Wholesale real estate / lead research
- 💰 Crypto / market monitoring
- 📣 Social media monitoring / posting
- 🧠 Something custom (I'll explain on the call)

**Technical experience level (radio):**
- 🟢 Non-technical — I've never touched a terminal
- 🟡 Somewhat technical — I can follow instructions
- 🔵 Technical — I'm comfortable with command line / servers
- 🔴 Developer — I just want the hard parts done

**CTA:** `Almost done →`

---

### STEP 3 OF 3: "One last thing"

**Page headline:**  
> "How should we reach you?"

**Subtext:**  
> "Seth personally reviews every application and typically responds within a few hours."

**Progress indicator:** ● ● ● (Step 3 of 3)

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| Phone number | Tel | ❌ | Optional — see copy below |
| Best time to reach you | Select | ❌ | Optional qualifier |
| How did you hear about ClawHQ? | Select | ❌ | Attribution, optional |

**Phone field label copy:**  
> *"Phone number (optional) — if you'd prefer a quick text or call over email"*

**Best time options:**
- Morning (8am–12pm)  
- Afternoon (12pm–5pm)  
- Evening (5pm–8pm)  
- Whenever — I'm flexible

**How did you hear options:**
- Twitter / X
- Friend or referral
- Google search
- Reddit
- YouTube
- OpenClaw community / Discord
- Other

**CTA button:**  
> `Submit My Application →`

**Below CTA (trust signal):**  
> 🔒 *Your info is never sold or shared. Seth reads every submission personally.*

---

### THANK YOU PAGE

**Headline:**  
> "You're in the queue, [First Name]. 🎉"

**Subtext:**  
> "Seth reviews every application personally and typically responds within a few hours. While you wait — grab a time on his calendar to skip the line:"

**Calendly embed** (inline, not redirect)  
→ 20–30 min "ClawHQ Setup Call" booking

**Below Calendly:**  
> "Prefer to wait for Seth to reach out? No problem — you'll hear from him at [email] within 24 hours."

**Social proof row** (3 short testimonials or "as seen in" logos):
- Fast Company, IBM, Cisco (from OpenClaw's press)
- One-liner quotes if available

---

## 4. AUTO-RESPONSE EMAIL SPECIFICATION

**Subject:** `Your ClawHQ application is confirmed ✅`  
**From:** `Seth <seth@clawhq.com>` ← NOT no-reply  
**Send trigger:** Immediate on form completion

---

**Body:**

> Hey [First Name],
>
> Got it — your application is in. I personally review every submission and I'll be in touch within a few hours (often sooner).
>
> **What you told us:**
> - Device: [their device answer]  
> - Tier interest: [their tier answer]  
> - Key use cases: [their top 2–3 checkboxes]
>
> **What happens next:**
> 1. I'll review your setup and put together a plan
> 2. I'll reach out to [email] (or [phone] if provided) to set up your call
> 3. We do a quick 20-min call, answer your questions, and book your installation day
>
> **Want to skip the queue?**  
> [→ Grab a time on my calendar] ← Calendly link (button)
>
> If you have questions in the meantime, just reply to this email. I read everything.
>
> — Seth  
> Founder, ClawHQ  
> [phone if he wants to share] | [twitter/X handle]

---

**Email design notes:**
- Plain text preferred — feels personal, not automated
- Alternatively: minimal HTML with logo at top, no heavy graphics
- No unsubscribe link needed (transactional, not marketing)
- Mobile-first — short paragraphs, one CTA button

---

## 5. FORM AS A SELLING TOOL — UX STRATEGY

Each step of the form should REINFORCE the value proposition, not just collect data.

### Step 1 Sells: The Device Choice
- Showing device options (VPS, Mac Mini, laptop) teaches the user that this is real infrastructure
- The pricing tiers remind them of the value before they're even committed
- Subtext: "We set it up remotely" removes a major objection (I'd have to be technical)

### Step 2 Sells: The Feature List
- Showing all the things the AI can do (email, calendar, crypto, real estate) is a micro-pitch
- Each checkbox is a tiny "yes" — psychological commitment to the product
- Seeing the full list = "wait, it can do ALL that?" moment
- The use case list should match Seth's target audience's actual pain points

### Step 3 Sells: Seth's Personal Touch
- "Seth personally reviews every application" = humanizes the brand
- Makes them feel like they're applying for something valuable, not just filling out a contact form
- Trust signals (lock icon, no-share promise) reduce drop-off anxiety

### Thank You Page Sells: Urgency + Social Proof
- Calendly on the thank you page = immediate next step while they're most excited
- "Skip the line" framing = FOMO / scarcity
- Press logos (Fast Company, IBM, Cisco) = legit credibility

---

## 6. WHAT TO AVOID (CONVERSION KILLERS)

| ❌ Don't Do This | ✅ Do This Instead |
|---|---|
| Required phone number on Step 1 | Make phone optional, move to Step 3 |
| Generic "Contact Us" form | "Apply for Your Setup" — frame as application |
| Ask for company name (B2C) | Skip it — this is personal AI, not B2B |
| Open-ended "tell us about yourself" | Checkboxes with specific options |
| "Submit" button | "Submit My Application →" |
| No-reply confirmation email | Seth's personal email as from address |
| 5+ steps | Max 3 steps — you're losing people at step 4 |
| No progress bar | Always show "Step X of 3" |
| Redirect to generic page | Thank you page WITH Calendly (instant booking) |
| Asking budget before building rapport | Budget pre-qualified by tier choice in Step 1 |

---

## 7. MINIMUM VIABLE FORM (if starting TODAY)

If Seth needs something live in an hour, here's the absolute minimum:

**Tool:** Tally.so (free)  
**Fields (single page):**
1. Name
2. Email
3. Device interest (radio)
4. What do you want it to do? (checkboxes, same list)
5. Phone (optional)

**After submit:** Redirect to Calendly booking page

This sacrifices ~30% of conversion vs. the 3-step version but is infinitely better than nothing. Build the 3-step version when there's capacity.

---

## 8. PLATFORM RECOMMENDATION

### Primary Recommendation: **Tally.so**
- ✅ Free forever for basic use
- ✅ Multi-step support with progress bars
- ✅ Conditional logic (show/hide fields based on answers)
- ✅ Embed on ClawHQ.com as iframe or popup
- ✅ Email notifications to Seth on each submission
- ✅ Zapier integration for CRM/automation
- ✅ Custom thank you page / redirect
- ⚠️ Less "wow factor" than Typeform

### Upgrade Path: **Typeform**
- ✅ Premium one-question-at-a-time UX (higher perceived quality)
- ✅ Feels like a premium product — matches a $349-$997 offer
- ✅ 300+ integrations
- ⚠️ Starts at $29/mo, response limits at each tier
- 📌 Worth it once traffic is consistent — looks significantly better

### Custom HTML + Formspree (if developer available)
- ✅ Zero branding constraints — fully match ClawHQ design
- ✅ Formspree handles submissions ($0 free tier, $10/mo pro)
- ✅ Can do multi-step with JavaScript (or use a library like Alpine.js)
- ⚠️ Takes dev time
- 📌 Best long-term if ClawHQ wants full brand control

---

## 9. COMPETITOR ANALYSIS — SetupClaw.com

| Feature | SetupClaw.com | ClawHQ.com Opportunity |
|---------|---------------|------------------------|
| Intake method | Direct Calendly link | Form → Then Calendly = More leads captured |
| Target market | Founders, exec teams, VCs | Broader (individuals, small biz) |
| Pricing | Not shown publicly | Show tiers openly — builds trust |
| Positioning | "White-glove enterprise" | "Personal AI for real people" |
| Lead capture | Buy first, ask questions later | Form first = lower barrier |
| Differentiator | Slack channel, 14-day hypercare | Seth's personal touch, faster/cheaper |

**SetupClaw weakness:** Zero form means zero async lead capture. If someone visits at 2am, they either book a call immediately or leave forever. ClawHQ's form captures them regardless of time zone.

---

## 10. IMPLEMENTATION PRIORITY

### Week 1 (Launch)
- [ ] Build 3-step Tally form with all fields above
- [ ] Set up email notification to Seth on each submission
- [ ] Create thank you page with Calendly embed
- [ ] Write and configure auto-response email (can be manual at first — just do it fast)
- [ ] Test on mobile

### Week 2 (Optimize)
- [ ] Add conditional logic (if device = Mac Mini, add iMessage to channel suggestions)
- [ ] Connect Tally → Zapier → Google Sheet (lead tracking)
- [ ] Set up abandon email if Step 1 complete but Step 2/3 not done
- [ ] A/B test: "Apply for Setup" vs "Get Started" CTA copy

### Week 3+ (Scale)
- [ ] Consider Typeform upgrade if conversion data justifies it
- [ ] Build custom HTML form if brand feels mismatched
- [ ] Add retargeting pixel on thank you page (Meta/Google)
- [ ] Add video testimonial near form (Loom-style, Seth talking to camera)

---

## APPENDIX: Quick Copy Reference

**Form title options (test these):**
- "Apply for Your OpenClaw Setup"
- "Let's Build Your Personal AI"
- "Get Your AI Running in 24 Hours"

**Step 1 CTA options:**
- "Continue →"
- "Next Step →"
- "Let's Go →"

**Final CTA options:**
- "Submit My Application →"
- "Book My Spot →"
- "Let's Do This →"

**Thank you headline options:**
- "You're in the queue, [Name]. 🎉"
- "Got it. Seth will be in touch within a few hours."
- "Application received. You're one step away from your AI."
