# Cadence — Setup Guide

Six importable n8n workflows. Follow this in order. Skipping ahead will cost you more time than it saves.

---

## 1. Create the Google Sheet first

Nothing works until this exists. Create one Google Sheet named **Cadence DB** with five tabs.

**Paste these exact header rows into row 1 of each tab.** The column names must match exactly — the workflows map to them by name.

**Tab `Ideas`**
```
idea_id	found_at	source	raw_signal	angle	keyword	who_its_for	score	why_this_score	status
```

**Tab `Content`**
```
content_id	idea_id	title	blog_markdown	linkedin_post	threads_post	facebook_post	newsletter_section	hashtags	status	validation_notes	created_at
```

**Tab `Posts`**
```
post_id	content_id	platform	title	body	hashtags	publish_at	status	platform_post_id	published_at	error_message
```

**Tab `Stats`**
```
snapshot_date	post_id	content_id	platform	impressions	reactions	comments	shares	clicks	note
```

**Tab `Config`** — optional, for your own notes. The workflows don't read it.

Then copy the Sheet ID from the URL. It's the long string between `/d/` and `/edit`:
`https://docs.google.com/spreadsheets/d/`**`THIS_PART`**`/edit`

---

## 2. Create the credentials in n8n

Create these in n8n **before** importing, so the dropdowns are populated when you go to fix the nodes.

| Credential name to use | Type in n8n | How to fill it |
|---|---|---|
| `Google Sheets — Cadence` | Google Sheets OAuth2 API | Google Cloud project → enable Sheets API → OAuth client (Web app) → paste n8n's redirect URI into Google |
| `Gemini API Key (x-goog-api-key)` | Header Auth | Name: `x-goog-api-key`  Value: your Google AI Studio key |
| `Telegram — Cadence Bot` | Telegram API | Token from @BotFather |
| `LinkedIn — Cadence` | LinkedIn OAuth2 API | LinkedIn app with "Share on LinkedIn" product |
| `Threads Token (Authorization: Bearer)` | Header Auth | Name: `Authorization`  Value: `Bearer YOUR_THREADS_TOKEN` |
| `Facebook Page Token (Authorization: Bearer)` | Header Auth | Name: `Authorization`  Value: `Bearer YOUR_PAGE_TOKEN` |
| `GitHub Token (Authorization: Bearer)` | Header Auth | Name: `Authorization`  Value: `Bearer YOUR_GITHUB_PAT` |
| `Brevo API Key (api-key header)` | Header Auth | Name: `api-key`  Value: your Brevo v3 key |

**You do not need all of these on day one.** Start with the first three. Add the platform ones as you switch each publishing branch on.

> **Facebook tokens expire.** The short-lived token you get first lasts about an hour. Exchange it for a long-lived Page token before you rely on it, or the workflow will quietly stop working in a couple of months.

---

## 3. Import the workflows

In n8n: **Workflows → ⋯ menu → Import from File**. Import in this order:

1. `00-error-handler.json`
2. `01-idea-radar.json`
3. `02-draft-studio.json`
4. `03-scheduler.json`
5. `04-publisher.json`
6. `05-stats-collector.json`
7. `06-weekly-report.json`

---

## 4. After importing each workflow — the same three fixes every time

Imported workflows never carry credentials across. This is normal, not a broken file.

**Fix 1 — the Config node.** Open it and replace every `YOUR_...` placeholder. This is where the Sheet ID, your niche, your voice rules and your platform IDs live. It's the only node you need to edit for normal setup.

**Fix 2 — the credentials.** Any node showing a red triangle needs its credential picked from the dropdown. Open the node, select the credential, save.

**Fix 3 — the Google Sheets document.** Even with the ID set, open each Google Sheets node once and confirm the document and tab resolve. If the "By Name" tab selector doesn't find your tab, switch the selector to "From list" and pick it — that fixes it permanently.

**Then set the error workflow:** open each workflow → **Settings → Error Workflow → "Cadence 00 — Error Handler"**. Ten seconds each. Do not skip this — it's the difference between finding out about a failure immediately and finding out three weeks later.

---

## 5. Turn them on in this order

Do not activate everything at once. Activate one, prove it, then move on.

| Order | Workflow | Prove it by |
|---|---|---|
| 1 | 00 Error Handler | Break a node on purpose in another workflow, run it, check Telegram |
| 2 | 01 Idea Radar | Run manually. Ideas tab fills with scored rows |
| 3 | 02 Draft Studio | Set one idea's status to `approved`, run manually, check the Content tab |
| 4 | 03 Scheduler | Set one content row to `approved`, run manually, check for 5 rows in Posts |
| 5 | 04 Publisher | Edit one Posts row's `publish_at` to a past time, run manually |
| 6 | 05 Stats Collector | Run manually after something is live |
| 7 | 06 Weekly Report | Run manually — it'll tell you if there's no data yet |

For the publisher, **start with LinkedIn only.** Delete the `platform` rows for the other four in your Posts tab while testing. Add one platform at a time.

---

## 6. How the pieces connect

The workflows never call each other. They coordinate entirely through status columns in the Sheet, which means you can run, debug or rebuild any one of them in isolation.

```
Ideas.status:    new → (you type "approved") → approved → drafted
Content.status:  pending_approval → (you type "approved") → approved → scheduled
Posts.status:    scheduled → published (or failed, with error_message)
```

Your only two jobs are typing `approved` in the Ideas tab and in the Content tab. Everything else is automatic.

---

## 7. Test plan

Run all eight before you call it working. Record the results — the completed table goes in your case study.

| # | Test | How | Expected |
|---|---|---|---|
| 1 | Happy path | Approve one idea, follow it to publication | Live on the platform, `platform_post_id` filled |
| 2 | Bad AI output | In Draft Studio, change the prompt to ask for prose instead of JSON | Row saved with `needs_regeneration`, nothing enters the queue |
| 3 | Rate limit | Temporarily set the Wait node to 0 and process 20 ideas | You see a 429; retry handles it. **Put the Wait back** |
| 4 | Dead credential | Revoke the Facebook token, run the publisher with all 5 platforms queued | Facebook row goes `failed` with an error message, other four publish |
| 5 | Double run | Run the publisher twice in a row | Second run publishes nothing |
| 6 | Over-length | Manually put a 600-character Threads post in the queue | Truncated to 480 before the API call |
| 7 | Empty day | Run Idea Radar with a broken feed URL | Exits cleanly, no error, no blank rows |
| 8 | Error alerting | Break any node deliberately | Telegram alert names the workflow and node |

---

## 8. Design decisions worth being able to explain

These are the things a technical client will ask about. Have the answer ready.

**Why HTTP Request instead of an AI node?** The Gemini REST call is stable across n8n versions and lets me set `responseMimeType: "application/json"`, which makes the model return valid JSON rather than JSON wrapped in markdown fences. Fewer moving parts, fewer parsing failures.

**Why one AI call for five formats?** The free Gemini tier allows roughly 10–15 requests per minute. Five separate calls per piece would hit that ceiling immediately in a loop. One structured call cuts requests by 80% and produces a more consistent voice, because the model writes all five formats together instead of in isolation.

**Why does every loop have a Wait node?** Rate limits are per minute, not per request. The Wait paces the loop under the ceiling. Removing it is the single fastest way to break this system.

**Why separate the Scheduler from the Publisher?** So you can change the posting rhythm without touching any API code, and debug a timing problem without risk of accidentally publishing.

**Why status columns instead of workflows calling each other?** Every stage is independently runnable and independently debuggable. If the publisher fails, the queue is still intact and you can re-run just that stage. Nothing is lost mid-chain.

**Why does nothing publish without approval?** Because a fully autonomous content system eventually embarrasses someone, and it will be the client. The automation handles finding, drafting, formatting, scheduling and publishing. The judgement stays human.

---

## 9. Known limitations — say these before a client finds them

- **LinkedIn personal profiles don't expose per-post analytics via API.** Those rows are logged with `manual_entry_needed`. Organisation pages do expose full analytics.
- **Threads text posts are capped at 500 characters.** The publisher truncates at 480 as a safety margin.
- **Threads and Instagram publishing needs Meta App Review** to post on accounts other than your own. Budget a few weeks.
- **X/Twitter is not included.** X moved to pay-per-use pricing in February 2026 with no free tier — roughly $0.015 per post, and about $0.20 for a post containing a link. For a link-heavy content system that's a real cost, so it's a priced add-on rather than part of the build.
- **Instagram can't accept text-only posts.** It needs an image, which is a separate subsystem. Phase 2.
