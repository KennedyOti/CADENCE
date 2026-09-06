# CADENCE
### A Text-Content Publishing Engine Built Entirely on Free Tiers

**Portfolio project plan + proposal guide**
Author: Benjamin | Version 1.0 | Scope: text-based content only, zero recurring cost

---

## PART 1 — THE PROBLEM

Before the tools, the problem. If you can't state this in 30 seconds, the demo won't sell anything.

### The problem, stated plainly

Solo consultants, agencies and small B2B service businesses know that publishing consistently brings in leads. Almost none of them do it consistently. Not because they can't write — they know their subject better than anyone — but because of two specific blockages:

**1. The blank page.** "What do I write about this week?" costs them 30–60 minutes of scrolling before a single word is written. Most weeks they give up before they start.

**2. The repurposing tax.** One good idea has to be rewritten four or five separate times — long for the blog, punchy for LinkedIn, short for Threads, casual for the Facebook page, personal for the newsletter. Each rewrite is a context switch. Five context switches is where the week dies.

The result is the pattern every consultant recognises: three posts in one week, then silence for a month. And inconsistent publishing is worse than none, because the audience never forms a habit around you.

### What Cadence does about it

Cadence removes both blockages and leaves the human in charge of the part that actually matters — judgement.

- Every morning it finds and scores 10–15 content ideas from sources your audience actually reads, so the blank page never appears.
- When you approve an idea, **one AI call** turns it into a complete publishing pack: a blog article, a LinkedIn post, a Threads post, a Facebook post and a newsletter section — all in your voice, all from the same idea.
- You review and approve in one place.
- It publishes each piece to each platform on schedule, spaced properly.
- It collects the numbers daily and sends you a plain-English report every Monday.

**Human time per week: about 90 minutes** (reviewing ideas, editing drafts, tapping approve). Everything else runs on its own.

**Recurring cost: $0.**

### Why this is a good portfolio project

- It solves a problem every prospective client already has, and recognises instantly.
- It is text-only, so no video rendering, no expensive APIs, no editing software.
- It runs on your own accounts, so you can leave it running for 60 days and show **real published output** rather than screenshots.
- Free tiers impose real constraints (rate limits, quotas), and working within them is genuine engineering you can talk about in an interview. This is a feature, not a compromise.

---

## PART 2 — THE FREE STACK

Everything here has a permanent free tier. Not a trial. Verified September 2026 — but check each one before you build, because these change often.

| Layer | Tool | Free tier reality | Watch out for |
|---|---|---|---|
| **Automation** | n8n Community Edition, self-hosted | Free, unlimited workflows and executions | Some features (multi-user, SSO, variables) are paid-only. Doesn't matter for this. |
| **Server** | Oracle Cloud Always Free ARM VM | Free forever. Recently reduced to about 2 CPU / 12 GB — still far more than n8n needs | ARM capacity is often unavailable in popular regions; you may have to retry over a few days |
| **Server (fallback)** | Your own PC or laptop, Docker | Free, always available | Only runs when the machine is on |
| **AI** | Google AI Studio (Gemini API) | Free. Flash and Flash-Lite models only | Roughly 10–15 requests/min and a daily request cap. **This shapes the whole design — see Part 4.** Pro models were removed from the free tier in April 2026. |
| **Database** | Google Sheets | Free, effectively unlimited rows for this | Slower than a real database; fine at this volume |
| **Database (nicer)** | Airtable free plan | Free, ~1,000 records per base | You'll hit the record cap in about a year. Sheets doesn't have this problem |
| **Documents** | Google Docs + Drive | 15 GB free | Text files are tiny; you'll never hit it |
| **Approvals & alerts** | Telegram bot | Free, no limits that matter | — |
| **Blog** | GitHub Pages + GitHub API | Free hosting, free API, custom domain supported | Static site — you commit a markdown file and it rebuilds |
| **Blog (alternative)** | Hashnode or Dev.to | Free, both have a free publishing API | Dev.to audience is developers; Hashnode is more flexible |
| **LinkedIn** | LinkedIn API | Free | Personal profile posting needs `w_member_social`. Company pages need extra permissions |
| **Threads** | Threads API (Meta) | Free, no paid tier, 250 posts per 24h | Two-step publish. App review needed to post on *other people's* accounts |
| **Facebook Page** | Meta Graph API | Free | Needs a Page, not a personal profile |
| **Newsletter** | Brevo free plan | Free, ~300 emails/day, API included | Daily send cap, not a subscriber cap |
| **Newsletter (alt)** | MailerLite free plan | Free up to ~1,000 subscribers | — |
| **Dashboard** | Looker Studio | Free | Connects straight to your Google Sheet |
| **Research sources** | RSS feeds, Hacker News Algolia API, Reddit JSON, Google News RSS, YouTube Data API | All free, most need no key at all | Reddit and HN want a polite request rate |
| **Domain / HTTPS** | DuckDNS + Caddy (Let's Encrypt) | Free subdomain, free certificate | Only needed if you use Telegram approvals — see Part 5 |

### What I deliberately left out, and why

Say this part out loud in interviews. Knowing what *not* to use is the more senior skill.

**X / Twitter — excluded.** X ended free API access. As of February 2026 new developers get pay-per-use only: roughly $0.015 per post, and about $0.20 for any post containing a link. A link-heavy content system would cost real money for the one platform with the worst organic reach for B2B. Not worth it. If a client insists, the honest answer is "that's a paid line item, here's the maths."

**Instagram — excluded from v1.** Instagram's API requires a media file; you cannot publish a plain text post. Adding it means generating quote-card images, which is a whole extra subsystem. It's in the roadmap as v2, not v1.

**All video — excluded.** Out of scope by design. Text is where the free tiers actually work.

**Medium — excluded.** Its publishing API was retired.

**Reddit auto-posting — excluded.** The API is free, but automated posting is the fastest way to get an account banned and a client angry. It stays a *research* source only. Mention this — it shows you think about consequences, not just capability.

**Paid AI (GPT, Claude) — not needed here.** Gemini Flash handles structured content generation well. The build is model-agnostic, so swapping in a paid model later is a one-node change. Say exactly that.

### One licensing note

n8n is fair-code licensed. Self-hosting it for your own use, or setting it up on a client's own server for their internal use, is fine. Reselling hosted n8n as your own product is not — that needs a commercial licence. Know this before a client asks you to "host it for us as a service."

---

## PART 3 — WHAT YOU'RE BUILDING

Five workflows. That's it. Small enough to actually finish, big enough to be impressive.

```
                    ┌──────────────────────────────────┐
                    │   GOOGLE SHEET  "Cadence DB"     │
                    │   Ideas · Content · Posts · Stats│
                    └──────────────────────────────────┘
                       ▲        ▲         ▲         ▲
      ┌────────────────┘        │         │         └──────────────┐
      │                         │         │                        │
┌─────┴────────┐    ┌───────────┴──┐  ┌───┴────────────┐  ┌────────┴──────┐
│ WF1          │    │ WF2          │  │ WF3            │  │ WF4           │
│ Idea Radar   │───▶│ Draft Studio │─▶│ Publisher      │  │ Pulse         │
│ daily 06:00  │    │ on approval  │  │ every 30 min   │  │ daily + Mon   │
│              │    │ ONE AI call  │  │ LI·TH·FB·Blog  │  │ stats+report  │
└──────────────┘    └──────────────┘  └────────────────┘  └───────────────┘

              ┌──────────────────────────────────────────┐
              │ WF0  Approval + Error Handler (Telegram) │
              └──────────────────────────────────────────┘
```

**The rule that makes it safe:** nothing publishes without a human approving it. Every piece sits at `pending_approval` until you say yes.

---

## PART 4 — THE FREE-TIER CONSTRAINT THAT SHAPES EVERYTHING

Read this before you build anything. It's the single most important design decision in the project, and it's the thing that will impress a technical client.

**The constraint:** Gemini's free tier gives you roughly 10–15 requests per minute and a daily request ceiling in the hundreds to low thousands, depending on which model you use. Check your actual numbers in Google AI Studio — they differ by model and change over time.

**The naive design** makes one AI call per platform per piece of content:
- 1 blog + 1 LinkedIn + 1 Threads + 1 Facebook + 1 newsletter = **5 calls per idea**
- Process 10 ideas in a loop = 50 calls fired as fast as n8n can loop
- Result: `429 Too Many Requests` before the tenth call. The workflow dies halfway, you have three published posts and two missing, and the Sheet is now in an inconsistent state.

**The Cadence design** makes **one AI call per idea**, returning all five formats in a single structured JSON response.

- 10 ideas = 10 calls instead of 50
- 80% fewer calls, and the voice stays consistent across all five formats because the model wrote them together
- Add a **Wait node of 6–8 seconds** inside every loop to stay under the per-minute cap
- Enable **Retry on Fail** (3 tries, 5000ms) on every AI node to survive the occasional 429
- Keep a daily counter row in the Sheet, and have the workflow stop and alert you at 80% of quota rather than failing mid-run

**Three sentences you can say in any interview:**

> "The free Gemini tier caps you at around 15 requests a minute, so I designed the generation step to produce all five platform formats in a single structured call instead of five separate ones. That cut API calls by 80% and made the voice more consistent across platforms, because the model writes them together rather than in isolation. The same architecture scales to a paid model with a one-node change."

That answer is worth more than any number of nodes on a canvas.

---

## PART 5 — SETUP

### Step 1: Get n8n running (2–3 hours)

Pick one route.

**Route A — Oracle Cloud Always Free (recommended: it runs 24/7 and costs nothing)**
1. Sign up at cloud.oracle.com. A card is required for identity verification; a small temporary hold appears and is released.
2. Choose your home region carefully — **you cannot change it later**. Pick a large region with several availability domains.
3. Create a Compute instance: Ubuntu 24.04, shape `VM.Standard.A1.Flex` (ARM), allocate the full free allowance.
4. If you get "Out of capacity," that's normal. Try a different availability domain, or retry over a few days. It eventually works.
5. Open ports 80 and 443 in both the OCI Security List **and** in `iptables` on the machine itself. Oracle images ship with restrictive local firewall rules and this catches everyone.
6. Install Docker, then run n8n with Docker Compose using PostgreSQL, following n8n's official self-hosting docs.

**Route B — your own machine (start here if Oracle is being difficult)**
1. Install Docker Desktop.
2. `docker run -it --rm -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n`
3. Open `http://localhost:5678`. Done in ten minutes.
4. Limitation: workflows only run when the machine is on. Fine for building; move to Route A before your 30-day proof run.

### Step 2: Decide if you need a public URL

This is a simplification most beginners miss and it can save you a whole day.

**Every trigger in Cadence is a schedule, not a webhook.** So if you use **Sheet-based approval** — you type `approve` in a column and a scheduled workflow picks it up 15 minutes later — you need **no public URL at all**. No domain, no HTTPS, no tunnel. Nothing.

If you want the nicer **Telegram approval buttons** (tap ✅ on your phone), the Wait node needs a public HTTPS URL. Free route: DuckDNS for a free subdomain, Caddy in front of n8n for an automatic free Let's Encrypt certificate.

**My recommendation:** build v1 with Sheet approval. Add Telegram in week 3 once everything else works. Don't let infrastructure block you from building the actual product.

### Step 3: Set these environment variables

```
N8N_ENCRYPTION_KEY=<long random string — BACK THIS UP>
GENERIC_TIMEZONE=Africa/Nairobi
N8N_DEFAULT_LOCALE=en
EXECUTIONS_DATA_PRUNE=true
EXECUTIONS_DATA_MAX_AGE=168
```

The last two matter: without pruning, execution history grows until the database chokes. Losing the encryption key means every saved credential becomes unrecoverable — back it up somewhere outside the server.

### Step 4: Credentials

**Gemini** — Go to Google AI Studio, create an API key. In n8n, create a "Google Gemini (PaLM) API" credential. Two minutes. Easiest one you'll do.

**Google Sheets / Docs / Drive** — Google Cloud Console → new project → enable Sheets, Docs and Drive APIs → OAuth consent screen (External, add yourself as a test user) → create an OAuth client ID (Web application) → paste n8n's redirect URI into Google, then the client ID and secret into n8n → Connect.

**Telegram** — Message @BotFather, `/newbot`, copy the token. Message your own bot once, then visit `https://api.telegram.org/bot<TOKEN>/getUpdates` to find your numeric chat ID.

**LinkedIn** — developers.linkedin.com → create an app → request the "Share on LinkedIn" and "Sign In with LinkedIn using OpenID Connect" products → n8n has a built-in LinkedIn node, use it rather than raw HTTP.

**Facebook Page** — developers.facebook.com → create an app → add Facebook Login → get a Page Access Token with `pages_manage_posts` and `pages_read_engagement`. Convert it to a long-lived token, otherwise it expires in about an hour and your workflow silently stops working in two months.

**Threads** — same Meta developer account. Scopes: `threads_basic`, `threads_content_publish`, `threads_manage_insights`. Publishing is two steps: create a container at `POST /{threads-user-id}/threads`, then publish it with `POST /{threads-user-id}/threads_publish`. Text posts are capped at 500 characters — validate this before sending or the call fails.

**GitHub (for the blog)** — Settings → Developer settings → fine-grained personal access token with Contents: Read and write on one repository. Publishing an article = committing a markdown file. GitHub Pages rebuilds the site automatically.

---

## PART 6 — THE DATABASE

One Google Sheet called **Cadence DB**, four tabs. Build this first — every workflow just reads a row, does one thing, writes the status back.

**Tab 1 — `Ideas`**
`idea_id` | `found_at` | `source` | `raw_signal` | `angle` | `keyword` | `who_its_for` | `score` | `why_this_score` | `status`

`status` values: `new`, `approved`, `rejected`, `drafted`

**Tab 2 — `Content`** (one row per idea that became content)
`content_id` | `idea_id` | `title` | `blog_markdown` | `linkedin_post` | `threads_post` | `facebook_post` | `newsletter_section` | `hashtags` | `status` | `created_at`

`status` values: `pending_approval`, `approved`, `rejected`, `scheduled`, `published`

**Tab 3 — `Posts`** (one row per platform per piece — this is your publishing queue)
`post_id` | `content_id` | `platform` | `body` | `publish_at` | `status` | `platform_post_id` | `published_at` | `error_message`

**Tab 4 — `Stats`** (one row per post per day)
`snapshot_date` | `post_id` | `platform` | `impressions` | `reactions` | `comments` | `shares` | `clicks`

Add a fifth tab, **`Config`**, holding your niche, audience description, tone rules and links. Every AI prompt reads from it. When a client wants a different voice, you change one cell instead of editing five prompts.

---

## PART 7 — BUILD, PHASE BY PHASE

### WF0 — Error Handler (build first, 30 minutes)

```
Error Trigger
  → Set: workflow name, node name, error message, execution URL
  → Telegram: "🔴 {{workflow}} failed at {{node}}: {{message}}"
```

Then open **every** workflow → Settings → Error Workflow → select this one. Ten seconds each. This single habit is the clearest visible difference between a hobbyist build and a professional one, and clients notice it.

---

### WF1 — Idea Radar (1–2 days)

**Runs:** daily at 06:00.
**Produces:** 10–15 scored, ranked ideas waiting in the Sheet before you wake up.

```
Schedule Trigger (daily 06:00)
  → RSS Read × 3–5   (blogs and newsletters your audience reads)
  → HTTP Request → Hacker News Algolia API
        http://hn.algolia.com/api/v1/search?tags=story&numericFilters=points>100
        (completely free, no key needed)
  → HTTP Request → Reddit
        https://www.reddit.com/r/{subreddit}/top.json?t=week&limit=25
        (free; set a User-Agent header, don't hammer it)
  → HTTP Request → Google News RSS
        https://news.google.com/rss/search?q={your+keyword}
  → Merge (append all branches)
  → Code: normalise everything to {source, headline, url, engagement}
  → Remove Duplicates (on headline)
  → Google Sheets: read existing Ideas → Filter out anything already captured
  → Limit: 15 items          ← protects your AI quota
  → Loop Over Items (batch size 1)
       → Wait 7 seconds       ← stays under the rate limit
       → Google Gemini: scoring prompt
       → Code: strip ```json fences, parse safely
  → Google Sheets: append rows to Ideas
  → Sort by score desc → Limit 10
  → Telegram: the morning digest
```

**The scoring prompt:**
```
You are a content strategist for {NICHE}.
Audience: {AUDIENCE}. Their biggest frustrations: {PAIN_POINTS}.

Here is a raw signal found online today:
Source: {source}
Headline: {headline}

Do four things:
1. Turn it into a specific content idea for THIS audience, with an angle
   that is not just a summary of the source.
2. Name the one person this is for (e.g. "an operations manager at a
   30-person firm who still approves invoices by email").
3. Give the search phrase they'd type to find it.
4. Score it 0-100: relevance to the audience (40%), how specific and
   non-obvious it is (30%), evergreen value (20%), how easy it is to
   write well (10%).

Be harsh. Most ideas are mediocre and should score under 60.
Only genuinely sharp, specific ideas score above 80.

Return ONLY valid JSON, no markdown fences, no commentary:
{"angle":"","who_its_for":"","keyword":"","score":0,"why_this_score":""}
```

**Two things that will go wrong:**
- The model wraps its JSON in ```` ```json ```` fences. Always strip them in a Code node with a try/catch before parsing. Never trust raw LLM output straight into a parser.
- Everything scores 85+. Fix it in the prompt: "most ideas should score under 60" calibrates it immediately.

**Done when:** you wake up, open Telegram, and ten ranked ideas with reasons are waiting.

---

### WF2 — Draft Studio (2 days — the heart of the system)

**Runs:** every 15 minutes, looking for ideas you marked `approved`.
**Produces:** five finished drafts from one AI call.

```
Schedule Trigger (every 15 min)
  → Google Sheets: read Ideas where status = "approved"
  → IF no rows → stop
  → Google Sheets: read the Config tab (niche, voice, links)
  → Loop Over Items (batch size 1)
       → Wait 8 seconds
       → Google Gemini: the multi-format prompt below
       → Code: strip fences, parse, validate all 5 fields exist
       → Google Sheets: append to Content (status = pending_approval)
       → Google Docs: create a doc with the blog article
       → Google Sheets: update the Idea row → status = "drafted"
  → Telegram: "3 new drafts ready for review" + link to the Sheet
```

**The multi-format prompt — the most important prompt in the project:**
```
You write for {NICHE}. Audience: {AUDIENCE}.
Voice rules: {VOICE_RULES}
Never use: "in today's fast-paced world", "unlock", "leverage",
"game-changer", "dive deep", "it's important to note".

The idea:
Angle: {angle}
Written for: {who_its_for}
Search phrase: {keyword}

Produce FIVE versions of this one idea. Same core insight, genuinely
different shape for each platform — do not just truncate the blog post.

1. BLOG (700-900 words, markdown): specific title, opening that names
   a concrete situation the reader recognises, 3-4 H2 sections, at least
   one real example with numbers, one clear takeaway. No conclusion that
   just restates the intro.

2. LINKEDIN (150-220 words): first line must work alone as the preview,
   short paragraphs with line breaks, one specific story or number,
   ends with a real question. No hashtag wall — maximum 3.

3. THREADS (under 480 characters, hard limit): the single sharpest
   point from the idea. Conversational. No hashtags.

4. FACEBOOK (80-120 words): warmer and plainer than LinkedIn.
   Assume the reader is scrolling and not in work mode.

5. NEWSLETTER (200-300 words): written to one person, second person,
   ends with one specific thing they could do this week.

Return ONLY valid JSON, no markdown fences:
{"title":"","blog_markdown":"","linkedin_post":"","threads_post":"",
 "facebook_post":"","newsletter_section":"","hashtags":[]}
```

**Validation step you must not skip.** After parsing, a Code node checks: all five fields present and non-empty, Threads post under 480 characters, blog over 400 words. If any check fails, write `status = needs_regeneration` and alert yourself instead of pushing broken content into the queue. Silent bad data is worse than a loud failure.

---

### WF3 — Publisher (2 days)

**Runs:** every 30 minutes.
**Does:** takes approved content, splits it into scheduled posts, publishes each one at its time.

**Part A — scheduling (runs when content is approved):**
```
Google Sheets: read Content where status = "approved"
  → Code: build 5 rows in Posts, one per platform, with staggered times
        blog       → next weekday 09:00
        linkedin   → same day 11:00
        threads    → same day 14:00
        facebook   → next day 10:00
        newsletter → Friday 08:00
  → Google Sheets: append to Posts (status = "scheduled")
  → Update Content status = "scheduled"
```

**Part B — publishing (every 30 minutes):**
```
Schedule Trigger (every 30 min)
  → Google Sheets: read Posts where status="scheduled" AND publish_at <= now
  → Loop Over Items (batch size 1)
      → Switch on platform:
          ├─ linkedin  → LinkedIn node → create post
          ├─ threads   → HTTP: POST /{user-id}/threads (media_type=TEXT)
          │            → Wait 20s
          │            → HTTP: POST /{user-id}/threads_publish
          ├─ facebook  → HTTP: POST /{page-id}/feed
          ├─ blog      → GitHub: create file
          │              path: _posts/{{date}}-{{slug}}.md
          │              content: front-matter + blog_markdown
          └─ newsletter→ Brevo API: create and send campaign
      → Google Sheets: update status="published", store platform_post_id
      → Wait 10 seconds
  → Telegram: "Published 3 posts"
```

**Four rules to build in:**
1. **Set "Continue On Fail"** on each platform branch, and write the error text into `error_message`. One platform failing must never stop the other four.
2. **Always store the returned `platform_post_id`.** Without it, WF4 cannot track performance and half the system is dead weight.
3. **Never publish anything with `status = published` already.** This is your protection against double-posting when a run overlaps.
4. **Threads is two calls, not one,** with a wait between them. Skipping the wait is the most common cause of failed Threads posts.

---

### WF4 — Pulse (1 day)

**Part A — daily stats collection, 02:00:**
```
Schedule Trigger (daily 02:00)
  → Google Sheets: read Posts published in the last 30 days
  → Switch by platform:
      ├─ threads   → GET /{media-id}/insights
      │              (views, likes, replies, reposts)
      ├─ facebook  → GET /{post-id}/insights
      └─ linkedin  → personal-profile stats aren't fully exposed by the
                     API; log what's available and note the gap
  → Code: normalise into one shape
  → Google Sheets: append rows to Stats
```

Then connect **Looker Studio** to the Stats tab and build five charts: posts published per week, engagement by platform, best-performing topics, best posting hour, week-over-week trend. Free, looks professional, and gives you a shareable link for your portfolio.

**Be honest about the LinkedIn gap.** Personal-profile analytics are limited via API. Say so in your proposal — "LinkedIn organisation pages expose full analytics; personal profiles don't, so those numbers are entered weekly or pulled from the export." Naming a limitation before a client discovers it is how you build trust.

**Part B — Monday 08:00 report:**
```
Schedule Trigger (Monday 08:00)
  → Google Sheets: read last 7 days of Stats + the 7 days before
  → Code: compute deltas
  → Gemini: analyst prompt
  → Telegram + Gmail: send it
```

```
You are a content performance analyst. Here is this week's data and
last week's: {data}

Write a short brief for a busy founder:
1. The three numbers that matter most, with % change
2. The best performing piece, and a specific reason why it worked
3. The worst performer, and a specific reason why it didn't
4. One pattern across the data
5. Three specific topics to write next week, each justified by
   something in the data

No praise, no filler. If performance dropped, say so and say why.
Under 350 words.
```

---

### WF0b — Telegram Approval (optional, add in week 3)

Only build this once everything else works, and only if you've set up a public URL.

```
Execute Workflow Trigger (row_id, summary, tab)
  → Telegram: send message with inline keyboard [✅ Approve] [❌ Reject]
  → Wait (Resume: On Webhook Call, timeout 24h)
  → Switch on callback data → update the Sheet accordingly
```

Until then, approval is: open the Sheet, type `approved` in the status column. Genuinely fine. Some clients actually prefer it.

---

## PART 8 — TESTING

Run every one of these before you call it done. Write down the result. The completed table goes in your case study — it's evidence of process, which is rarer and more valuable than evidence of skill.

| # | Test | How to run it | Expected result |
|---|---|---|---|
| 1 | Happy path | Add one idea manually, approve it, follow it to publication | Live on all 5 destinations |
| 2 | Broken AI output | Temporarily change the prompt to return prose instead of JSON | Code node catches it, error alert fires, nothing corrupts |
| 3 | Rate limit hit | Remove the Wait nodes and run 20 items | You see the 429, then confirm retry logic handles it. Put the Waits back |
| 4 | Bad credential | Revoke the Facebook token | Facebook branch fails and logs, other 4 still publish |
| 5 | Double run | Trigger the publisher twice within a minute | Second run publishes nothing |
| 6 | Over-length content | Force a 600-character Threads post | Caught by validation before the API call |
| 7 | Empty day | Run Idea Radar with all sources returning nothing | Exits cleanly, no error, no empty rows |
| 8 | Recovery | Stop the server mid-run, restart it | Nothing lost, next run picks up where it left off |

---

## PART 9 — TIMELINE

Realistic for evenings and weekends alongside your day job.

| Week | Build | Done when |
|---|---|---|
| **1** | n8n running, Gemini + Sheets + Telegram credentials, the Sheet built, WF0 error handler | An error in a test workflow pings your phone |
| **2** | WF1 Idea Radar + WF2 Draft Studio | Ideas arrive daily; approving one produces five drafts |
| **3** | WF3 Publisher — LinkedIn and Threads first, then Facebook, blog, newsletter | One idea goes live on all five |
| **4** | WF4 Pulse + Looker Studio dashboard + all 8 tests + Telegram approvals | Dashboard shows real numbers |
| **5–8** | **Run it for real on your own content.** Fix what breaks. | 30 days of published output |
| **9** | Demo video, case study, GitHub repo | Portfolio-ready |

**If you only have two weeks:** WF0 + WF1 + WF2, and publish to LinkedIn only. "Finds and scores ideas daily, then turns an approved idea into five platform-ready drafts in one AI call" is already a complete, demonstrable product.

---

## PART 10 — PROVING IT WORKS

This is what separates you from every other freelancer who says they build automations.

**Run it on yourself for 30 days.** Point Cadence at your own positioning — automation and AI for small businesses, or Business Central for mid-size firms, whichever you want inbound leads for. Let it publish three times a week.

At the end you can say: **"This system has published 40 pieces across four platforms over 30 days, on free infrastructure, and here's the live dashboard."**

That sentence closes deals. "I know n8n" does not. And there's a compounding benefit: while it demonstrates your skill, it's also building your own audience and generating your own leads. The demo *is* the marketing.

**Then package five things:**

1. **A 3-minute demo video.** The problem (20s) → the architecture diagram (20s) → one idea flowing end to end (100s) → the dashboard (20s). Your own voice. This is your highest-value asset.
2. **A one-page case study PDF.** Problem, approach, diagram, the free-stack table, the 90-minutes-vs-full-week comparison, screenshots of real published posts.
3. **The live blog.** A real URL where the automated articles actually live.
4. **A public GitHub repo.** Workflow JSON with credentials stripped, plus this document trimmed as the README.
5. **A view-only Looker Studio link.**

---

## PART 11 — HOW TO PITCH IT

### Position it as a product, not as hours

Don't sell "n8n automation." Sell **"a content publishing system that gets you from three posts a month to twelve, without adding a hire."**

### Your three-tier offer

| Tier | What they get | Typical price |
|---|---|---|
| **Starter** | Idea Radar + Draft Studio + LinkedIn publishing, on their own free accounts, plus a handover walkthrough | Fixed fee, 1 week |
| **Standard** | All five workflows, four platforms, dashboard, documentation, 30 days of support | Fixed fee, 2–3 weeks |
| **Ongoing** | Monthly retainer: monitoring, prompt tuning as their voice evolves, new platforms as they need them | Monthly |

Price in your own market. The structure matters more than the numbers: **fixed scope, fixed fee, clear boundary between build and maintenance.**

### The four questions every client will ask, and your answers

**"Will it sound like AI?"**
No — because you're not asking a chatbot for a post. The system carries your voice rules, your audience, your banned phrases and your examples into every generation, and nothing publishes without your approval. The first two weeks are spent tuning the voice against your actual writing.

**"What if it posts something embarrassing?"**
It can't. Every piece stops at approval. The automation does the finding, drafting, formatting, scheduling and publishing. The judgement stays with you. That's deliberate.

**"Why free tools? Are they reliable?"**
They're the same APIs the paid tools use — you're just skipping the middleman's subscription. The free tiers have limits, and I've designed around them explicitly: one AI call per piece instead of five, request pacing to stay under rate limits, and automatic retries. If you outgrow them, upgrading is a credit-card change, not a rebuild.

**"What happens if you disappear?"**
You own the server, the accounts and the workflows. I hand over a written runbook and a recorded walkthrough. Nothing is locked to me.

### The upgrade path (say this early — it builds trust and it sells the retainer)

- More volume than the free AI tier allows → move to paid Gemini or Claude, one node change
- Instagram → add quote-card image generation
- X/Twitter → possible but now a paid line item, roughly $0.015 per post and about $0.20 per post containing a link
- Video → a separate project with real costs
- Team approvals → move from Sheets to Airtable or Notion

---

## PART 12 — PROPOSAL TEMPLATE

Adapt the bracketed parts. If the job post asks for a specific opening line, use it exactly — that's the first thing they check.

---

**AI WORKFLOW**

Hi [Name],

Most content systems fail for one of two reasons: nobody knows what to write on Monday morning, or one good idea has to be rewritten five times for five platforms and the week runs out before it happens.

I built a system that removes both. Here's how it works and what I'd build for you.

**What it does**

Every morning it pulls signals from the sources your audience actually reads — industry blogs, Hacker News, relevant subreddits, news feeds — and an AI scores each one on relevance, specificity and how easy it is to write well. You get a ranked shortlist before you start work.

You approve the ideas you like. Each approved idea then becomes five finished drafts in a single AI call — a blog article, a LinkedIn post, a Threads post, a Facebook post and a newsletter section. Same insight, genuinely different shape for each platform, all in your voice, using your rules and your banned phrases.

You review, approve, and it publishes on schedule. Then it tracks performance daily and sends you a Monday morning report in plain English: what worked, what didn't, and three specific topics to write next.

Your time: about 90 minutes a week. Everything else runs on its own.

**Why I build it this way**

Two design decisions I'd want you to understand before we start.

*Nothing publishes without your approval.* The automation handles finding, drafting, formatting, scheduling and publishing. The judgement stays with you. I've seen fully autonomous content systems and they eventually embarrass someone.

*Everything runs on free infrastructure.* Self-hosted n8n, Gemini's free API tier, Google Sheets as the database, and the platforms' own free APIs. There's no monthly software cost. That means working within real constraints — the free AI tier allows around 15 requests a minute, so I designed the generation step to produce all five formats in one structured call rather than five separate ones. That's 80% fewer API calls, and the voice comes out more consistent because the model writes them together. If you outgrow the free tier, moving to a paid model is a one-node change, not a rebuild.

**What I've built**

- **Cadence** — the system described above. I've been running it on my own content for [30] days: [40] pieces published across [4] platforms, fully automated apart from approvals. [Demo video link] · [Live dashboard link] · [Blog link]
- **A WhatsApp lead-capture system** in n8n — inbound messages classified by AI for intent and urgency, hot leads pushed to the owner immediately, everything else logged to a CRM and enrolled in follow-up.
- **A voice QA harness** for a medical AI phone agent using Twilio and the OpenAI Realtime API — automated call testing and response validation.
- **My day job** is building API integrations for Microsoft Dynamics 365 Business Central for clients in the Netherlands and Kenya. OAuth flows, rate limits, retry logic and making sure a job that runs twice doesn't do the work twice are things I handle every week, not things I've read about.

**Platforms I specialise in**

n8n, self-hosted — chosen because it handles branching, custom code and AI orchestration properly, and doesn't bill per task, which matters when one idea fans out into twenty operations. I've also worked with Make.com and GoHighLevel where a client is already committed to them.

**Two things I'd tell you upfront**

X/Twitter no longer has a free API tier — as of February 2026 it's pay-per-use, roughly $0.015 per post and around $0.20 for any post containing a link. For a link-heavy content system that's a real cost for the platform with the weakest organic reach in B2B. I'd leave it out unless you specifically want it, and then it's a priced line item.

Instagram can't accept plain text posts through the API — it needs an image. It's straightforward to add once the text pipeline is live, but it's a second phase, not part of the first build.

I'd rather tell you both now than have you find out in week three.

**How I'd start**

A 30-minute call to look at your current process and hear how you actually write. Then a fixed-scope, fixed-price first phase — idea engine plus drafting plus your single most important platform — live within a week, so you see it working before committing to the rest.

Every build I hand over includes a written runbook, a recorded walkthrough, and error alerts that reach you the moment something breaks. You own the accounts and the server. Nothing is locked to me.

[Name]
[Portfolio link]

---

## PART 13 — QUICK GLOSSARY

**API** — how one program talks to another. Instead of clicking "post," your workflow sends LinkedIn a message asking it to post.

**Rate limit** — how many requests you may make per minute. Exceed it and you get a `429` error.

**Quota** — the daily ceiling on requests. Resets each day.

**429 error** — "too many requests." Wait and retry; don't retry immediately or you make it worse.

**Structured output** — forcing the AI to reply in strict JSON so a program can read it reliably.

**OAuth** — the "Sign in with Google" permission flow. You approve once, your app gets a token, the token lets it act for you.

**Access token** — the temporary key an API gives you after OAuth. Expires. Short-lived Facebook tokens must be exchanged for long-lived ones.

**Webhook** — a URL that waits for something to happen and starts a workflow. Cadence mostly avoids these, which is why it needs no public server.

**Idempotency** — making sure running the same thing twice doesn't produce two results. Critical for anything that publishes.

**Container (Threads/Instagram)** — Meta's two-step publish. Create a container holding the content, then publish it with a second call.
