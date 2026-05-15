# Atrium — Softr Build Sheet
*Page-by-page build guide. Open this beside Softr while you build.*

This translates the Atrium prototype (`atrium-app.html`) into concrete Softr pages, Airtable views, blocks, and Make.com scenarios. Work through it top-to-bottom. Each section is short and operational — no theory.

---

## 0. Pre-build checklist (Day 0 — half a day)

Create accounts and grab the keys before touching Softr.

| Tool | Plan to start | What you need |
|---|---|---|
| Airtable | Free → Team ($24/user/mo when you outgrow it) | API key (Personal Access Token) |
| Softr | Basic ($49/mo) | — |
| Make.com | Core ($9/mo) | — |
| Cloudinary | Free (25GB) | Cloud name, API key, API secret |
| Buffer | Essentials ($6/mo) | OAuth connection later via Make |
| Anthropic Claude API | Pay-as-you-go | API key, $20 prepay |
| Tally | Free | — |
| Resend | Free (3k emails/mo) | API key, domain verified |
| Stripe | Pay-as-you-go | API key (test mode first) |
| Domain | — | atrium.studio or similar; point CNAME to Softr |

**Estimated month-1 spend (no studios yet):** ~$80. Most of this is Softr + Make.

---

## 1. Airtable base setup (Day 1)

Create one base named **`Atrium · Production`**. Copy the data model from the architecture doc — 11 tables. Below is the minimum-viable build order with the views Softr needs to bind to.

### Tables to create

1. `Studios` — tenant table
2. `Users` — auth + role per studio
3. `Projects` — canonical project record
4. `Outlets` — pre-seeded library of awards/publications/lectures
5. `Outreach Submissions` — the Kanban pipeline
6. `Packages` — generated outreach packages
7. `Social Uploads` — raw photo/video drops
8. `Social Posts` — composed posts ready to schedule
9. `Publications Archive` — past coverage
10. `Events Archive` — past lectures/exhibitions
11. `Client Shares` — token-based share records

### Critical views to create per table

These are the views Softr lists/grids/kanbans bind to. Name them exactly as below — saves you mistakes later.

**Projects table — create these views:**
- `Grid · All` (default editing view)
- `Gallery · By studio` — grouped by Studio, filtered to current user's studio via Softr
- `Grid · Built only` — filter Status = "Built"
- `Grid · This studio` — filter for Softr-binding

**Outreach Submissions — create these views:**
- `Kanban · By status` — group by Status field
- `Calendar · By submitted date`
- `Grid · This studio` — Softr-binding view

**Social Posts — create:**
- `Grid · Drafts`
- `Grid · Scheduled`
- `Grid · Posted`
- `Calendar · Scheduled at`

**Outlets — create:**
- `Grid · All outlets`
- `Grid · Upcoming deadlines` — formula field for days until deadline

### Key formulas to add

In `Projects`:
- `Hero image URL` = `IF({Hero image}, {Hero image}[0].url, "")` — for embedding in Softr
- `Outreach count` = `COUNTA({Outreach Submissions})`

In `Outreach Submissions`:
- `Days since submitted` = `IF({Submitted date}, DATETIME_DIFF(TODAY(), {Submitted date}, 'days'), "")`
- `Stale flag` = `IF(AND({Status} = "Submitted", {Days since submitted} > 60), "⚠ Follow up", "")`

In `Outlets`:
- `Next deadline (days)` = formula based on next-window date field

---

## 2. Softr app shell setup (Day 2 morning)

1. **Create a new Softr app** named `Atrium`.
2. **Connect Airtable** via Personal Access Token. Grant access to the Atrium base.
3. **Set up user groups** in Softr (Settings → User groups):
   - `Owner` — full access
   - `Editor` — everything except billing & user management
   - `Viewer` — read-only
   - `Client` — only sees their Client Shares
4. **Domain** — point your custom domain. Softr handles SSL.
5. **Set the design system** in Softr's design panel:
   - Primary color: `#1c1a17`
   - Accent: `#b8542e`
   - Background: `#f5f2ec`
   - Heading font: DM Serif Display
   - Body font: Inter

---

## 3. Page-by-page Softr build

For each screen, you'll find: the Softr page name, who can see it, the Airtable view it binds to, and the blocks to drop in.

### 3.1 `/onboarding` — Studio onboarding wizard

Mirrors `onboarding.html`. Softr handles this with a **9-step form block** (or 9 separate pages with Next buttons).

| Block | Type | Source | Notes |
|---|---|---|---|
| Step 1 — Welcome | Static text + button | — | Heading + intro |
| Step 2 — Basics | Form | Writes to `Studios` | Studio name, founded, location, principal name |
| Step 3 — What you build | Form | Updates `Studios` | Multi-select chips for typology + materials |
| Step 4 — Philosophy | Form | Updates `Studios` | Two long-text fields |
| Step 5 — Train your voice | Form + file upload | Writes to `Studios.Voice samples` (attachment field) | Up to 5 files or pasted URLs |
| Step 6 — Identity | Form | Updates `Studios` | Logo upload, palette hexes, font names |
| Step 7 — Synthesise | Static loading page | Calls Make webhook | Webhook URL goes here — see scenario 1 |
| Step 8 — Voice doc | Detail page | Reads `Studios.Brand voice notes` | Editable rich-text field |
| Step 9 — Done | Static page | — | Three CTA cards + "Open studio" button → `/dashboard` |

**Access:** logged-in user; only visible if `Studios.Onboarded` is unchecked.

**Conditional logic:** after step 8, set `Studios.Onboarded = true`, redirect to `/dashboard`.

---

### 3.2 `/dashboard` — Studio home

Mirrors the Dashboard screen.

| Block | Type | Source | Filter |
|---|---|---|---|
| Hero text | Static + dynamic | Reads `Studios.Name`, `Users.First name` | "Good evening, {Name}" — Softr supports user data tokens |
| KPI cards (×4) | Stat blocks | Airtable counters | Active projects (`Projects` count where Studio = logged-in user's studio), Open submissions, This week posts, YTD wins |
| Recent activity | List block | `Activity log` view (create a new table for this) | Last 10 entries, filtered to studio |
| Upcoming deadlines | List block | `Outlets · Upcoming deadlines` view | Filtered to studio's relevant outlets |

**Access:** logged-in user with completed onboarding.

**Critical:** every list block must have a **user filter** = `Studio = Logged-in user's Studio`. This is Softr's row-level multi-tenancy. Configure under Block Settings → Filter → "Use logged-in user's data" → field `Studio`.

---

### 3.3 `/projects` — Project library

Mirrors the Projects screen.

| Block | Type | Source |
|---|---|---|
| Page header + "+ New project" button | Static + button | Button → opens `/projects/new` form |
| Filter chips | Inline filters | Filter by Status, Typology |
| Project grid | List block (gallery layout) | `Projects · Gallery · By studio` view |

Each card displays:
- Hero image (Cloudinary URL field)
- Title
- Location · Year · Area
- Up to 3 tags (Typology, Materials)
- Click → `/projects/{record_id}`

**User filter:** Studio = logged-in user's Studio.

---

### 3.4 `/projects/new` — New project form

A modal or full page.

| Block | Type |
|---|---|
| Form block writing to `Projects` table | Form |

Fields: Title, Status, Location, Year, Area, Typology (multi), Concept narrative (long), Hero image (file).

**On submit:** triggers Make scenario `Project narrative generator` (see §4.2) → user lands on `/projects/{new_id}` and sees the AI panel filling in.

---

### 3.5 `/projects/{id}` — Project detail

Mirrors the Project Detail screen. This is the most block-heavy page.

| Block | Type | Source |
|---|---|---|
| Back to projects link | Static link | — |
| Project header | Detail block | `Projects` record | Hero image full width |
| Meta grid (location, year, area, client, materials, photographer) | Detail block | `Projects` record |
| Outreach so far panel | List block | `Outreach Submissions` filtered by `Project = current record` |
| "Build outreach package" button | Action button | Opens modal → `Packages · New` form |
| "Share with client" button | Action button | Creates `Client Shares` record + copies link |
| Tabs (Narrative, Media, Drawings, Technical, Outreach) | Tabbed section | — |
| Narrative card × 4 (tagline, 50w, 150w, 400w) | Detail block | `Projects` rich-text fields |
| "Regenerate variants" button | Action button | Webhook → Make scenario 2 |

**Access:** Studio members (Owner, Editor, Viewer).

---

### 3.6 `/outreach` — Outreach Kanban

Mirrors the Outreach screen.

| Block | Type | Source |
|---|---|---|
| Filter chips (Awards, Publications, Lectures, Project) | Inline filters | — |
| Kanban block | Kanban list block | `Outreach Submissions · Kanban · By status` |
| Calendar toggle button | Page link | → `/outreach/calendar` |

**Card shows:** Outlet name (big), Project name (small), Deadline date, Type badge.

**Drag behavior:** Softr's Kanban natively updates the `Status` field when a card is dragged. Set up an Airtable automation: when Status → "Submitted", set `Submitted date = TODAY()`; when → "Accepted", create a Publications Archive record.

---

### 3.7 `/outreach/new` — New submission

| Block | Type |
|---|---|
| Form writing to `Outreach Submissions` | Form |

Linked-record pickers for `Project` and `Outlet`. Submit → triggers Make scenario 3 (package builder).

---

### 3.8 `/social` — Social Studio

Mirrors the Social screen.

| Block | Type | Source |
|---|---|---|
| Upload zone | File upload form | Writes to `Social Uploads` |
| Tabs (Drafts, Scheduled, Posted) | Inline filter | Filter `Social Posts.Status` |
| Post list | List block | `Social Posts · Grid · Drafts` (etc.) |
| Composer side panel | Form + detail block | Updates a specific `Social Posts` record |
| AI variants button | Action button | Webhook → Make scenario 4 |
| Schedule button | Action button | Webhook → Make scenario 5 (sends to Buffer) |

---

### 3.9 `/brand` — Brand Vault

Mirrors the Brand Vault screen. Mostly a single editable detail page bound to the logged-in user's `Studios` record.

| Block | Type | Source |
|---|---|---|
| Studio profile detail | Detail block | `Studios` record (user-bound) |
| Identity panel (logo, palette, type) | Detail block | `Studios` record |
| Voice doc display | Rich text detail | `Studios.Brand voice notes` |
| "Re-synthesise voice" button | Action button | Webhook → Make scenario 1 |
| "Edit" buttons | Inline editor | Softr's edit-in-place |

---

### 3.10 `/archive` — Publications & Events

| Block | Type | Source |
|---|---|---|
| Filter chips | Inline filter | — |
| List block | List | `Publications Archive · By studio` |
| List block | List | `Events Archive · By studio` |

Each row shows Outlet/Event name, sub-detail, date.

---

### 3.11 `/client-shares` — Client Shares management

Studio-side view of created shares.

| Block | Type | Source |
|---|---|---|
| "+ Create share" button | Button → form | Writes to `Client Shares` |
| List of shares | List block | `Client Shares · By studio` view |
| Each row: Project, Recipient, Token, Last viewed, Copy link button | List item config | — |

---

### 3.12 `/share/{token}` — Public client view

Mirrors `client-share.html`. **This page must be public** (no login required) and filtered by the URL token parameter.

| Block | Type | Source |
|---|---|---|
| Studio mark header | Static (dynamic from token) | Reads `Studios` via `Client Shares.Studio` |
| Hero | Detail block | `Projects` linked via `Client Shares.Project` |
| Pipeline section | List block | `Outreach Submissions` filtered by Project AND `Shared with client = true` |
| Wins grid | List block | `Publications Archive` filtered by Project |
| Numbers panel | Stat block | Aggregated metrics (manual fields in `Client Shares` for now) |
| Social tiles | List block | `Social Posts · Posted` filtered by Project |
| What's next | List block | Mix of upcoming submissions + scheduled posts |
| Signature | Static (dynamic) | Reads `Studios.Principal name` |

**Access:** public, URL-parameter filtered. Softr supports this via "Filter by URL parameter" on the page.

**Security:** the token in the URL is the only auth. Make it 32+ chars. Add an `Expires` date and an `Access count` counter (incremented via Airtable automation on page view — track via a logged hit table).

---

## 4. Make.com scenarios

Build these in order. Each scenario starts with either an Airtable webhook trigger (button press) or a scheduled trigger.

### 4.1 Scenario 1 — Brand voice synthesiser
**Trigger:** Webhook (called from onboarding step 7).
**Modules:**
1. Webhook receives `studio_id`.
2. Airtable: Get `Studios` record by id. Read voice samples.
3. (For each sample attachment) HTTP module: download file, parse text (use `mammoth` for docx, plain read for txt, OCR for PDF if needed — or pre-extract on the form).
4. Anthropic Claude module:
   - System: "You're synthesising a brand voice for an architectural studio. Output JSON with: attributes (3 named), do_list (4), dont_list (4), vocabulary_keep, vocabulary_avoid."
   - User: studio philosophy + values + 3 samples concatenated.
   - Model: `claude-sonnet-4` (or whichever is current at build time).
5. Parse JSON.
6. Airtable: Update `Studios.Brand voice notes` (rich text), `Voice version` += 1.
7. Webhook response: 200 OK.

**Average run cost:** $0.04. **Runtime:** ~12 sec.

### 4.2 Scenario 2 — Project narrative generator
**Trigger:** Webhook (called from project detail "Regenerate variants" button or after new project save).
**Modules:**
1. Webhook receives `project_id`.
2. Airtable: Get `Projects` record.
3. Airtable: Get linked `Studios` record (for brand voice).
4. Claude module — system message includes brand voice doc; user message includes structured project fields + raw concept narrative.
5. Request returns JSON: `{tagline, short_50, mid_150, long_400}`.
6. Airtable: Update `Projects` rich-text fields.
7. Webhook response.

**Cost:** $0.03 per call.

### 4.3 Scenario 3 — Package adapter
**Trigger:** Webhook (called from "Build outreach package" button).
**Modules:**
1. Receives `project_id`, `outlet_id`.
2. Airtable: Get Project, Outlet, Studio (for brand voice).
3. Claude module — system: brand voice + outlet submission spec; user: project narrative + materials + photos list. Returns: outlet-specific text body, suggested image order (array of asset IDs), credits block, cover-letter draft.
4. For each image in suggested order: Cloudinary API — transform to outlet's required dimensions/format. Collect URLs.
5. Airtable: Create `Packages` record with generated text, image URLs, locked=false.
6. Resend: Email user with "Your Dezeen package is ready" + link to review.

**Cost:** $0.05–$0.08 per call.

### 4.4 Scenario 4 — Social caption generator
**Trigger:** Webhook (called when a `Social Upload` is created or "Draft posts" button pressed).
**Modules:**
1. Receives `upload_id`.
2. Airtable: Get upload + linked Project + Studio.
3. Claude — generate 3 caption variants per selected channel (Instagram, LinkedIn) tuned for different angles (process, poetic, technical) + hashtags.
4. Airtable: Create 3 `Social Posts` draft records linked to the upload.
5. Notify user (in-app + email).

**Cost:** $0.02 per call.

### 4.5 Scenario 5 — Schedule to Buffer
**Trigger:** Airtable record updated — `Social Posts.Status` changes to "Scheduled".
**Modules:**
1. Get Social Post + linked Upload.
2. Cloudinary: get the image URL (already there).
3. Buffer module: schedule post to selected channels at `Scheduled at`.
4. Airtable: write `Buffer ID` back to record; status stays "Scheduled".
5. After post goes live (Buffer webhook back to Make), set status = "Posted" + `Posted URL`.

### 4.6 Scenario 6 — Deadline radar
**Trigger:** Scheduled, daily at 7 AM IST.
**Modules:**
1. For each `Studios` record (active subscription):
2. Iterator: get `Outlets · Upcoming deadlines` view (within 30 days).
3. Compare against studio's `Projects` — flag relevant ones not yet submitted.
4. Build digest email (Resend module).
5. Send if 1+ suggestions.

### 4.7 Scenario 7 — Submission status auto-actions
**Trigger:** Airtable record updated in `Outreach Submissions`.
**Modules:**
- If new status = "Submitted" → set `Submitted date = TODAY`.
- If new status = "Accepted" → create `Publications Archive` row, send celebration email to studio, log activity.
- If new status = "Rejected" → log activity, optional email.

### 4.8 Scenario 8 — Weekly client digest
**Trigger:** Scheduled, Sunday 6 PM IST.
**Modules:** For each active `Client Share` viewed in last 30 days → compose summary of activity → email to client recipients.

### 4.9 Scenario 9 — Project asset upload (Cloudinary)
**Trigger:** Tally form submission (or direct file upload field on Softr).
**Modules:**
1. Receive file + project ID.
2. Cloudinary: upload (with folder `studios/{studio_slug}/projects/{project_slug}/`).
3. Airtable: append URL to `Projects.All photos` or `Hero image`.

---

## 5. Build order — first three weeks

### Week 1 — Skeleton

| Day | Build |
|---|---|
| Day 1 | Airtable: create 11 tables, key fields only, 1 seed Studio + 1 seed Project. |
| Day 2 | Softr app shell: connect Airtable, design system, user groups, domain. |
| Day 3 | Pages: `/dashboard`, `/projects`, `/projects/{id}` (read-only first). |
| Day 4 | Make scenario 9 (Cloudinary upload) + Tally form. Test end-to-end. |
| Day 5 | Make scenario 2 (project narrative) + wire "Regenerate" button. |
| Day 6 | Walk a real studio through it. Capture every moment of confusion. |
| Day 7 | Triage feedback. Decide what to fix vs defer. |

### Week 2 — Outreach & Brand

| Day | Build |
|---|---|
| Day 8 | Pages: `/outreach`, `/outreach/new`, Kanban block. |
| Day 9 | Pre-seed `Outlets` table with 30 outlets for your region. |
| Day 10 | Make scenarios 3 (package builder) + 7 (status automations). |
| Day 11 | Page: `/brand`. Make scenario 1 (brand voice synthesiser). |
| Day 12 | Page: `/onboarding` — 9-step flow. Wire to scenario 1. |
| Day 13 | Polish + first paying user runs full onboarding to package end-to-end. |
| Day 14 | Test with second studio. |

### Week 3 — Social & Client share

| Day | Build |
|---|---|
| Day 15 | Page: `/social`. Composer + tabs. |
| Day 16 | Make scenarios 4 + 5 (caption gen + Buffer scheduling). |
| Day 17 | Page: `/client-shares` (admin) + `/share/{token}` (public). |
| Day 18 | Make scenarios 6 (deadline radar) + 8 (weekly client digest). |
| Day 19 | Stripe + plan gating. |
| Day 20 | QA pass + invite 5 founding studios. |
| Day 21 | Day-zero of private beta. |

---

## 6. Testing checklist (before opening beta)

Tick every box.

### Multi-tenancy
- [ ] Logged in as Studio A, can you see Studio B's projects? (Should be NO.)
- [ ] Create a second studio, repeat for every list page.
- [ ] Try editing a URL to access another studio's project ID directly.

### Permissions
- [ ] Viewer cannot see "+ New project" button.
- [ ] Viewer cannot drag Kanban cards.
- [ ] Client role only sees Client Shares page.

### Onboarding
- [ ] Brand voice synthesis completes in under 30 seconds for 3 samples.
- [ ] Voice doc is editable post-synthesis.
- [ ] User can skip optional steps.

### Project flows
- [ ] Upload 10 photos via Tally — all land in Cloudinary + linked in Airtable.
- [ ] "Generate blurb" returns 3 lengths, distinct from each other.
- [ ] Regenerate produces different output each time.

### Outreach
- [ ] Drag a card from Drafting → Submitted — Submitted date stamps automatically.
- [ ] Build package → Email arrives within 60 seconds.
- [ ] Package has correct image dimensions for the outlet spec.

### Social
- [ ] Upload triggers caption generation within 30 sec.
- [ ] Schedule → Buffer → post goes live at the set time.
- [ ] After posting, status flips to "Posted" with URL filled in.

### Client share
- [ ] Open share URL in incognito — no login prompt.
- [ ] Verify only flagged Outreach Submissions show.
- [ ] Token expiry works.
- [ ] Page view count increments.

### Money
- [ ] Stripe test: subscribe to Studio plan → user lands on dashboard.
- [ ] Cancel → access reverts to read-only after period ends.

---

## 7. Going-live checklist

| Item | Done |
|---|---|
| Custom domain SSL active | ☐ |
| Resend domain verified (DKIM, SPF) | ☐ |
| Stripe webhook → Softr live mode | ☐ |
| Privacy policy + ToS pages live | ☐ |
| Backup: weekly Make scenario exports all tables to Drive | ☐ |
| Status page or maintenance banner ready | ☐ |
| Founding studio pricing in Stripe | ☐ |
| Loom walkthrough recorded (5 min) | ☐ |
| First 5 invites sent | ☐ |

---

## 8. Where this build will hit a wall (and how to handle it)

### Wall 1 — Softr's image handling for high-res architectural photos
**Symptom:** Cards load slowly with big Cloudinary images.
**Fix:** Always use Cloudinary URL transformations in the image field (`w_800,q_auto,f_auto`). Set this in your formula field, not at upload time, so original stays archived.

### Wall 2 — Make.com operation limits on Core plan
**Symptom:** Hitting 10k operations/month at 8 studios.
**Fix:** Bump to Pro ($16) or move heavy daily-radar scenario to a different platform (n8n self-hosted, ~$5/mo VPS).

### Wall 3 — Buffer's free plan only allows 1 channel set
**Symptom:** Studio wants IG + LinkedIn + X.
**Fix:** Move to Buffer Essentials ($6/mo per studio) — bake into your Studio-plan price.

### Wall 4 — Kanban drag is too slow with 100+ submissions
**Symptom:** Lag.
**Fix:** Add a "Closed" archive view; only show last 6 months in main Kanban.

### Wall 5 — Onboarding wizard in Softr can feel clunky
**Symptom:** Multi-step form blocks aren't as fluid as `onboarding.html`.
**Fix:** Either accept it (it's a one-time experience) or build the onboarding as a custom embedded code block — Softr supports it. The HTML in `onboarding.html` can be embedded as-is, with the final step calling a Softr webhook to update the user record.

### Wall 6 — When Softr stops being enough
**Trigger:** You want custom interactions Softr's blocks don't support (drag-and-drop within a package builder, drawing annotation, etc.).
**Fix:** Move to **Bubble** (4–6 weeks rebuild) or **WeWeb + Xano** (faster if you already have Airtable structure). Don't rebuild prematurely — Softr should hold to ~100 paying studios.

---

## 9. Cheat-sheet: which file to look at for what

| Question | Open this |
|---|---|
| What does the dashboard / projects / outreach look like? | `atrium-app.html` |
| What does the client see in a share? | `client-share.html` |
| What's the onboarding experience? | `onboarding.html` |
| What's the data model and overall architecture? | `studio-comms-architecture.md` |
| How do I actually build this in Softr? | This file (`softr-build-sheet.md`) |

---

*Build sheet v0.1 — update as you learn. Anything you discover during Week 1 belongs in section 8.*
