# Studio Comms Platform — System Architecture
*A no-code/low-code blueprint for an architectural communications platform*

---

## 1. Product summary

A multi-tenant platform that gives small architectural studios a single place to (a) manage every artifact related to their communications — brand, project assets, publications, events — (b) package projects and send them out to awards/publications/lectures/events with end-to-end tracking, and (c) plan and produce social content from raw uploads, with AI generating studio-specific text and content suggestions.

Pricing positioning: replace ~₹40k–₹2L/month spent on a freelancer or junior agency with a subscription in the ₹3k–₹15k/month range, depending on studio size and usage.

---

## 2. Target user and jobs-to-be-done

**Primary user:** Principal / founder of a 1–10 person architectural studio in India or similar markets, who currently handles their own communications badly or inconsistently and cannot justify an agency retainer.

**Secondary users:** A junior architect or studio manager who actually does the day-to-day uploading and coordination.

**Jobs:**

| Job | Today (painful) | With platform |
|---|---|---|
| Keep brand identity consistent across deliverables | Files scattered on Drive/Dropbox/local | Single brand vault with templates the platform consumes |
| Package a finished project for outreach | Re-edit text/images each time per outlet | Pull from one canonical project; generate variants per outlet |
| Send to awards/publications | Tracked nowhere, missed deadlines | Outreach pipeline with status, calendar, reminders |
| Show client the comms work | Verbal/email back-and-forth | Client-shareable view of outreach + outcomes |
| Post on social regularly | Forgets, posts inconsistently | Upload → AI captions → scheduled → posted |
| Maintain archive (publications, lectures) | Lost over time | Permanent searchable record per studio |

---

## 3. Core modules

The platform decomposes into seven modules. Each can be built as a separate area in the no-code app, sharing the same underlying database.

1. **Studio Profile & Brand Vault** — onboarding wizard captures scale, project types, design philosophy, values, tone. Logo, colors, typography, templates are stored here. Drives the AI layer.
2. **Project Library** — canonical record of each built/unbuilt project: text (concept, narrative, technical), graphics (drawings, diagrams), photography, videography, technical metadata (area, location, year, client, materials).
3. **Project Packaging Studio** — generates an outreach-ready package from a Project, tailored to a specific Outlet (an award form, a magazine's submission spec, a lecture brief). AI rewrites and re-trims to fit.
4. **Media Outreach Tracker** — a Kanban/calendar of submissions: Outlet, deadline, status (planned → sent → under review → won/published/rejected). Client-shareable.
5. **Social Media Studio** — uploads + AI caption generation + scheduling to Instagram/LinkedIn primarily.
6. **Publications & Events Archive** — print scans, links, talk recordings, photographs of lectures and events.
7. **Client View (read-only share)** — a curated subset the studio shares with its client for transparency.

---

## 4. Recommended stack

### Primary recommendation: Softr + Airtable + Make.com + Cloudinary + Buffer + Anthropic Claude API

| Layer | Tool | Why |
|---|---|---|
| Frontend / app UI | **Softr** | Fastest way to build a multi-tenant, role-based web app on top of a database. Strong template library, decent design control. |
| Database / backend | **Airtable** | Tables map naturally to your domain (Studios, Projects, Outreach, etc.). Easy to model, easy to evolve. Has views (Kanban, Calendar, Gallery) which Softr can embed. |
| Automation / glue | **Make.com** | Cheaper and more powerful than Zapier for media-heavy, branching workflows. Native Anthropic + OpenAI modules. |
| Media storage + delivery | **Cloudinary** | Hosts high-res architectural photos and video, with on-the-fly resizing/format conversion. Cheaper egress than Drive at scale; better URLs than Airtable attachments. |
| AI text + image generation | **Anthropic Claude API** (text) + **OpenAI / fal.ai / Replicate** (image variants if needed) | Claude is well-suited to longform brand-voice rewrites. Image gen is optional in MVP. |
| Social scheduling | **Buffer** or **Publer** (Buffer's API is simpler) | Schedules to IG, LinkedIn, X, FB. Native Make module. |
| Email (outreach, notifications) | **Resend** or **Postmark** | Cheap, deliverability-friendly. |
| Auth & billing | Softr built-in + **Stripe** (via Softr's Stripe integration) | Avoids building this yourself. |
| Forms (project intake, outlet briefs) | **Tally** or **Fillout** | Embeds in Softr, writes to Airtable. |

**Total monthly cost at MVP scale (you + 5 paying studios):** roughly $80–$150/month.

### Why not the alternatives

- **Bubble** — more powerful and one tool instead of two, but you'd be building auth, tables, and views from scratch. Slower to MVP. Move here in v2 if Softr feels limiting.
- **Webflow + Memberstack + Xano** — beautiful frontend, but Xano backend is more complex than Airtable for a solo no-code builder. Reserve for when design is a wedge.
- **Bare Notion / Notion + Super** — tempting but bad fit: weak multi-tenancy, weak media handling, weak forms.
- **Glide** — strong mobile-first, but the heavy media review and outreach Kanban are better on desktop.

---

## 5. Data model (Airtable schema)

Eleven tables. Field types abbreviated: `T` text, `LT` long text, `S` single select, `M` multi-select, `A` attachment, `D` date, `L` link to another table, `F` formula, `U` URL, `N` number, `C` checkbox.

### Studios
| Field | Type | Notes |
|---|---|---|
| Name | T | Studio name |
| Slug | T | URL-safe id |
| Scale | S | Solo / 2–5 / 6–10 / 11+ |
| Project types | M | Residential, Institutional, Adaptive reuse, Interior, Landscape, etc. |
| Design philosophy | LT | Free text, used by AI |
| Studio values | LT | Free text, used by AI |
| Brand voice notes | LT | Generated + editable |
| Logo | A | |
| Color palette | T | Hex codes, comma-separated |
| Primary typeface | T | |
| Plan | S | Free / Solo / Studio / Practice |
| Owner email | T | |

### Users
Standard auth fields plus `Studio` (L → Studios) and `Role` (S: Owner / Editor / Viewer / Client).

### Projects
| Field | Type | Notes |
|---|---|---|
| Title | T | |
| Studio | L → Studios | Tenant key |
| Status | S | Concept / In progress / Built / Published |
| Year | N | Completion year |
| Location | T | |
| Area (sqm) | N | |
| Client | T | |
| Typology | M | Same options as Studio.project types |
| Materials | M | |
| Concept narrative | LT | Author writes; AI can expand |
| Technical description | LT | |
| Short blurb (50 words) | LT | AI-generated, human-edited |
| Hero image | A | |
| All photos | A | Up to 50 |
| Drawings | A | Plans, sections, axos |
| Videos (URLs) | U | Cloudinary or YouTube/Vimeo |
| Credits | LT | Photographer, contractor, consultants |

### Outlets
A library of awards/publications/lecture venues. Pre-seeded by you with ~200 popular outlets (Dezeen, ArchDaily, Stir World, IndianArchitect&Builder, IIA awards, etc.) and extensible per studio.

| Field | Type | Notes |
|---|---|---|
| Name | T | |
| Type | S | Award / Publication / Lecture / Event / Exhibition |
| Region | M | India, Asia, Global |
| Submission URL | U | |
| Submission spec | LT | Word counts, image specs, deadline cadence |
| Typical deadline window | T | "March each year" |
| Fee | N | INR |
| Notes | LT | |

### Outreach Submissions
The pipeline. One record per Project × Outlet submission.

| Field | Type | Notes |
|---|---|---|
| Project | L → Projects | |
| Outlet | L → Outlets | |
| Studio | L → Studios | Derived via lookup |
| Status | S | Planned / Drafting / Submitted / Under review / Accepted / Rejected / Withdrawn |
| Submitted date | D | |
| Decision date | D | |
| Outcome notes | LT | |
| Package | L → Packages | The package version sent |
| Shared with client | C | Visible in Client View |

### Packages
A snapshot of a Project tailored to one Outlet.

| Field | Type | Notes |
|---|---|---|
| Project | L → Projects | |
| Outlet | L → Outlets | |
| Generated text (per word-count) | LT | |
| Selected images | A | Subset of project images, sized to outlet spec |
| Selected drawings | A | |
| Credits block | LT | |
| Generated at | D | |
| Locked | C | When sent, freeze |

### Social Uploads
Raw uploads the studio dumps in.

| Field | Type | Notes |
|---|---|---|
| Studio | L → Studios | |
| Project | L → Projects | Optional |
| Media | A | Single image or video |
| Note from studio | LT | "This is the courtyard shot we love" |
| Status | S | New / Drafted / Scheduled / Posted / Rejected |
| Channel | M | Instagram / LinkedIn / X |
| Scheduled at | D | |

### Social Posts
The composed post ready to go to Buffer.

| Field | Type | Notes |
|---|---|---|
| Uploads | L → Social Uploads | One or more |
| Caption | LT | AI-drafted, human-edited |
| Hashtags | LT | |
| Channel | S | |
| Scheduled at | D | |
| Buffer ID | T | After scheduling |
| Status | S | Draft / Scheduled / Posted |
| Posted URL | U | |

### Publications Archive
| Project (L), Outlet (L), Type (S: Print scan / Online), URL (U), Scans (A), Date (D) |

### Events Archive
| Studio (L), Event name (T), Type (S: Lecture / Panel / Exhibition / Workshop), Date (D), Venue (T), Recording URL (U), Photos (A), Notes (LT) |

### Client Shares
Token-based shareable links to a Client View. `Studio`, `Token`, `Projects shown` (L → Projects, multiple), `Expires`, `Access count`.

---

## 6. AI / content generation layer

This is the wedge that justifies the price. Treat it as four named "generators," each a prompt + a Make.com scenario.

### 6.1 Brand voice synthesizer
- **Trigger:** Studio completes onboarding (philosophy, values, project types, sample texts they like).
- **Action:** Single Claude call returns a structured brand voice doc: 3 voice attributes, do/don't list, sample sentence transforms, vocabulary preferences. Saved to `Studios.Brand voice notes`.
- **Reuse:** Every other generator pulls this as a system message preamble.

### 6.2 Project narrative writer
- **Trigger:** User clicks "Generate blurb" on a Project.
- **Action:** Claude receives `Studio brand voice` + structured project fields + raw concept notes. Returns three lengths (50 / 150 / 400 words) and a one-line tagline. User picks/edits.

### 6.3 Package adapter
- **Trigger:** User clicks "Generate package for [Outlet]" on a Project.
- **Action:** Claude receives brand voice + project narrative + the Outlet's submission spec (word count, angle, image count). Returns a tailored text block, suggested image order, and a credits block. Cloudinary is then called via Make to produce images at the outlet's required dimensions/format.

### 6.4 Social copy generator
- **Trigger:** User uploads to Social Uploads and clicks "Draft posts."
- **Action:** Claude receives brand voice + project context + image metadata (or the studio's note) and returns 2–3 caption variants per channel with different angles (process / poetic / technical). Hashtag set tuned per channel.

### Prompt scaffolding (reusable)

```
SYSTEM
You are writing for {{Studio.Name}}, an architectural studio.
Voice attributes: {{Studio.Brand voice notes}}.
Always use British/Indian English. Never use the words "elevate", "unlock", "delve".
Avoid generic architecture-speak ("seamless", "stunning", "harmonious blend").
Reference specific materials, dimensions, and design choices wherever possible.

USER
Project: {{Project.Title}}
Year: {{Project.Year}} | Location: {{Project.Location}} | Area: {{Project.Area}} sqm
Typology: {{Project.Typology}}
Materials: {{Project.Materials}}
Concept narrative (raw):
{{Project.Concept narrative}}

Task: {{task-specific instruction}}
Constraints: {{word count / format}}
Return JSON with keys: ...
```

### Cost shape
At Claude Sonnet pricing, an average project package generation runs about 2k–4k input tokens + 1k output, ~$0.02–$0.05 per generation. A studio doing 20 packages and 100 social drafts/month costs you <$5 in AI. Bake $1–$3 of AI cost into each plan as headroom.

---

## 7. Media storage strategy

Architectural photography is heavy (RAW 20–80MB, JPEG 5–15MB, video files multi-GB). Airtable attachments are fine for thumbnails and small docs but **not** for hero photography or video — they don't transform images and Airtable's CDN is slow.

**Recommended:**
- All uploaded photos go to **Cloudinary** via a Make.com upload step. Airtable stores only the Cloudinary public ID + thumbnail.
- Cloudinary delivers per-outlet resized versions on demand (`w_1600,q_auto,f_auto` for web, `w_3000` for print).
- Videos: Cloudinary up to ~100MB; for heavier reels, **Bunny Stream** or **Vimeo** with link stored in Airtable.

**Capacity & cost (small scale):**
- Cloudinary free: 25 credits/month ≈ 25GB storage + 25GB bandwidth + transforms. Enough for 5 paying studios.
- Cloudinary Plus: $89/month, 225 credits. Comfortable for ~30 paying studios.
- For tens of GBs of video, add Bunny Stream (~$1/month base + $0.005/GB delivered).

**Retention policy:** never auto-delete. Studios are paying for the archive.

---

## 8. Automation flows (Make.com scenarios)

Map each scenario to a button or status change in Softr/Airtable.

1. **New project asset upload** — Tally form → Make → upload to Cloudinary → write Cloudinary URL back to Airtable Project record.
2. **Generate project blurb** — Airtable button → Make → Claude API → write 3 lengths back to Project.
3. **Build outreach package** — Airtable button on Outreach Submission → Make: pull Project + Outlet specs → Claude for text → Cloudinary for resized images zipped → Dropbox/Drive deposit + email link to user → set Package.Locked = true.
4. **Submission status change** — Airtable automation: if Outreach Submission.Status changes to "Submitted," set Submitted date = today and notify client (if Shared with client). If "Accepted," fire a celebration email and create a Publications Archive record.
5. **Social draft pipeline** — Social Upload created → Make → Claude generates 2–3 caption variants → creates Social Post drafts → notifies user.
6. **Schedule to Buffer** — Social Post.Status changes to "Scheduled" → Make pushes to Buffer with Cloudinary asset URL → on success writes Buffer ID back.
7. **Deadline radar** — Daily scheduled scenario: for each Outlet with typical deadline in next 30 days, check if studio has unsubmitted relevant Projects → email suggestion ("You haven't submitted Project X to Dezeen — their next window opens in 21 days").
8. **Weekly digest to client** — Sunday: for each Studio with active Client Shares, send a summary of activity (new submissions, decisions received, posts published).

---

## 9. Integrations checklist

| Integration | Purpose | Cost at small scale |
|---|---|---|
| Anthropic Claude API | All AI text | $5–$30/mo |
| Cloudinary | Media storage + delivery | $0–$89/mo |
| Buffer | Social scheduling | $6–$18/mo per connected channel set |
| Resend | Transactional email | $0 (free tier 3k/mo) |
| Stripe | Subscriptions | 2.9% + fixed fee per charge |
| Tally / Fillout | Forms | $0–$29/mo |
| Make.com | Automations | $9–$29/mo (depends on ops) |
| Softr | App UI | $49–$139/mo |
| Airtable | Database | $0–$24/user/mo (start free, move to Team) |

**Realistic monthly burn at 0–5 paying studios:** ~$80/mo.
**At 30 paying studios:** ~$350/mo.

---

## 10. Multi-tenancy, permissions, security

- Every record carries a `Studio` link or lookup. Softr's user filters enforce row-level isolation: a user only sees records where `Studio = their studio`.
- Roles: Owner (full), Editor (everything except billing), Viewer (read-only internal), Client (sees only records flagged `Shared with client = true` plus Client Shares table).
- Client Shares use a tokenised public URL with optional expiry — built as a public Softr page that filters by token.
- Backups: Airtable's snapshots + a weekly Make export to Google Drive (CSV per table). Cloudinary holds originals.
- Sensitive: avoid storing client financial data; if you do, encrypt the field or keep it off-platform.

---

## 11. Pricing model (suggested)

| Plan | Target | INR/month | What it includes |
|---|---|---|---|
| Free / Starter | Solo trial | ₹0 | 1 project, 1 outlet/month submission, 10 social drafts, branded watermark on shares |
| Solo | Solo practitioner | ₹2,500 | 10 projects, unlimited submissions, 30 social drafts/mo, no watermark |
| Studio | 2–5 person studio | ₹6,000 | Unlimited projects, 100 social drafts/mo, client share, archive |
| Practice | 6+ person studio | ₹12,000 | Multi-user, priority outlet seeding, on-demand human review (you) of one package/mo |

Start with all features in one plan during private beta, then split. Founding studio pricing: 50% off lifetime for the first 10.

---

## 12. Phased build roadmap

### Phase 0 — Validation (week 1–2)
- Interview 8–10 small studios. Show them this architecture as a deck. Ask what's missing, what's overkill, what they'd pay.
- Lock the v1 feature list based on feedback.

### Phase 1 — MVP build (week 3–8)
**Goal:** one studio can run their full comms loop end to end. You operate it semi-manually.

Build order:
1. Airtable schema (all 11 tables) — half a day.
2. Softr app shell: auth, Studio Profile, Project Library (list + detail). Two days.
3. Tally upload form → Cloudinary → Airtable. One day.
4. Make.com scenario: project blurb generator (Claude). One day.
5. Outreach Submissions Kanban view in Softr. One day.
6. Social Uploads + caption generator + Buffer scheduling. Two days.
7. Manual onboarding for 1–2 friendly studios.

### Phase 2 — Private beta (week 9–14)
**Goal:** 5 paying studios. Smooth the rough edges.
- Package adapter for top 20 outlets pre-seeded.
- Client Share view.
- Stripe + plan gating.
- Deadline radar.
- Publications & Events archive.

### Phase 3 — Open beta (month 4–6)
**Goal:** 30 paying studios. Reduce your manual ops time per studio to <30 min/week.
- Self-serve onboarding wizard with AI brand voice synthesis.
- Outlet library reaches 200.
- Weekly digest emails.
- Referral / founding-studio program live.

### Phase 4 — Scale decisions (month 6+)
Pick one based on signal:
- **Productize harder** — invest in design polish, add image-gen features, build mobile app via Glide companion.
- **Service-augment** — offer a "we-do-it-for-you" tier with you/contractors writing/curating.
- **Vertical expand** — adapt for adjacent fields (interior designers, landscape architects).

---

## 13. Risks, open questions, and what could kill it

| Risk | Mitigation |
|---|---|
| Studios won't trust AI-generated text for their craft | Always present AI output as a draft; show side-by-side; never auto-publish. The studio remains the author. |
| Awards/publications change submission specs constantly | Outlet records need a maintainer (initially you). Build a "flag stale" button per outlet. |
| Heavy media + cheap plans = margin compression | Track per-studio storage; introduce overage at 10GB/studio. |
| Instagram API restrictions on auto-posting from third parties | Buffer/Publer abstract this, but require the studio to grant access; build a clear onboarding for it. |
| Studios prefer working with humans, not platforms | Lead with the archive + outreach tracker (clear value); soften the AI angle in messaging. |
| You become a bottleneck operating it for early studios | That's fine for the first 5 — it's how you learn. Document everything, build automation from observed manual steps. |

**Open questions to resolve before week 3:**
- Which 1–2 studios will be your first design partners?
- What is the single Outlet whose package format you'll perfect first? (Recommend Dezeen — well-defined spec, high-value to studios.)
- Will you operate India-only or global from day one? (Recommend India-only for v1; outlet library and pricing are localised.)
- Do you have rights to use studio content in marketing the platform itself? (Add to ToS.)

---

## 14. What I'd build first this week

If you have one week and want to feel real progress:

1. **Day 1** — Set up Airtable with the 11 tables. Wire one Studio + one real Project (use your own studio or a friend's).
2. **Day 2** — Stand up Softr, point it at Airtable, get login + Studio Profile + Project list pages working.
3. **Day 3** — Add Tally form for project asset uploads → Make scenario → Cloudinary → back to Airtable.
4. **Day 4** — Build the "Generate blurb" Claude scenario. End-to-end: button in Airtable → Make → Claude → text back into the Project record.
5. **Day 5** — Add the Outreach Submissions Kanban. Manually log 3 past submissions for your test studio.
6. **Day 6** — Walk a real studio through it. Note every place they get confused or want more.
7. **Day 7** — Decide if you keep building or pivot the scope.

By end of week one you'll either have proof that the core loop is valuable, or you'll have learned what's wrong cheaply.

---

*Document v0.1 — refine as you build. Pin this in your Comms folder alongside the Airtable base.*
