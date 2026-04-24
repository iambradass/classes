# Handoff — classes.cnattitle (CNAT Education site)
**Date:** 2026-04-24
**Session summary:** Built the CNAT Education class registration platform end-to-end: combined Legal Update I & II registration page + Stripe/GHL workflows with sales rep attribution, all 14 course detail pages with rich content, and prepped migration from `.tech` to `.com` as primary domain.

---

## What Was Done This Session

### Infrastructure & Deploy
- ✅ Fixed recurring Hostinger "not a git repository" deploy error on `classes.cnattitle.tech`
- ✅ Provisioned `classes.cnattitle.com` subdomain on Hostinger (addon, `/home/u269315972/domains/classes.cnattitle.com/public_html`)
- ✅ MyIT added A record: `classes.cnattitle.com → 145.223.104.252` (confirmed 2026-04-24)
- 🟡 **User is wiring git auto-deploy on `.com` right now** — using repo `https://github.com/iambradass/classes` branch `main`

### Registration flow (classes.cnattitle.tech/legal-update-april-2026/)
- Combined Legal I + Legal II into a single upcoming tile on homepage
- Restructured form flow: **Step 1 class choice → Step 2 attendance → Step 3 name/email/phone → Stripe checkout** (auto-redirect same-tab)
- Webhook payload includes all fields needed for GHL: `name, email, phone, attendance, class_choice, class_price, sales_rep, ref_source, source, session, class_name, submitted_at`
- Session field drives dynamic per-month tagging (won't need rebuild for May/June/etc.)
- `client_reference_id` passed to Stripe for session-level tagging on payment side

### GHL workflows (CNAT account, location `9ZNjcEKhpBno9RRXrBA4`)
- ✅ **Legal Update Webhook** (`fc3d1735-4623-4596-b4a8-bc23327c0f87`) — creates/updates contact, dynamic tag `legal-update-{{session}}`, attribution field
- ✅ **Stripe Payment Confirmation** (user-created) — triggered by `checkout.session.completed` webhook, tags `paid-{{session}}`, fires confirmation email
- ✅ Webhook URLs:
  - Registration: `https://services.leadconnectorhq.com/hooks/9ZNjcEKhpBno9RRXrBA4/webhook-trigger/5169cb08-f3f7-4c92-b691-d5eb6b0b1861`
  - Stripe payment: `https://services.leadconnectorhq.com/hooks/9ZNjcEKhpBno9RRXrBA4/webhook-trigger/ab204d48-f60c-4b93-972e-23f2d2f208d8`
- ✅ Stripe event destination created in Stripe Dashboard → Webhooks → sends `checkout.session.completed` to GHL

### Custom fields created in GHL
- Class Choice (dropdown: Legal Update I, Legal Update II, Both Classes)
- Class Session (text)
- Class Registration Status (single options: Registered, etc.)
- Class Registration Date (date)
- Class Payment Status (single options: Paid, Pending Payment, etc.)
- Class Amount Paid (numerical — stores dollars from registration webhook's `class_price`, not Stripe cents)
- Stripe Payment ID (text — `pi_xxx`)
- Stripe Payment Link (text — `plink_xxx`)
- Sales Rep (dropdown: Bradley Patterson, Jami Cortez, Kandi Patterson, Shera Moffitt, Other-N/A)
- Attribution Source (text — `url`, `manual`, or `url-unmatched:xxx`)

### Tags pre-created
- `legal-update-april-2026`, `legal-update-may-2026`
- `paid-april-2026`, `paid-may-2026`
- `rep-jami_cortez`, `rep-shera_moffitt`, `rep-kandi_patterson`, `rep-bradley_patterson` (user renamed from old IG-style slugs to match GHL dropdown values)

### Sales rep attribution
- URL `?ref=` param captures rep in hidden field (with alias map: `jami→jami_cortez`, `shera→shera_moffitt`, etc.)
- Falls back to optional "Referred by" dropdown if no URL param
- `ref_source` tracks whether attribution came from `url` vs `manual`
- User confirmed existing "assign to rep" workflow fires on `rep-{{contact.sales_rep}}` tag

### Per-rep share URLs (already given to reps)
```
Jami Cortez:      https://classes.cnattitle.tech/legal-update-april-2026/?ref=jami
Shera Moffitt:    https://classes.cnattitle.tech/legal-update-april-2026/?ref=shera
Kandi Patterson:  https://classes.cnattitle.tech/legal-update-april-2026/?ref=kandi
Bradley:          https://classes.cnattitle.tech/legal-update-april-2026/?ref=bradley
```

### Social launch
- ✅ Posted to @cnattitle IG + Community National Title LinkedIn via GHL Social Planner
- ⚠️ No FB page connected to GHL — skipped
- User needs to add IG collaborators manually in IG app (@techintitle, @jami_foxx_tx, @shera.dfwrealtor, @closingwithkandi)

### Course pages (all 14)
- ✅ **Legal Update I & II** (`/courses/legal-update/`) — template reference page with full rich content, "Why CNAT" section, RENE numbers, etc.
- ✅ **13 other course pages** rewritten via brand-voice-writer agent following the template:
  - CNAT Agent ONE (#30671-RECE)
  - Contracts & Contractual Competence (#39207-RECE)
  - Create with Canva (no # yet)
  - CRM to ROI (no # yet)
  - Ensuring Insurance Coverage (#30669-RECE)
  - Farm to (Closing) Table (#30672-RECE)
  - Keeping AI on Things (no # yet)
  - RE: Technology (no # yet)
  - Scripty Business (no # yet)
  - Social Media Bootcamp (#30674-RECE)
  - TREC Advertising & Compliance (#30673-RECE)
  - Video Marketing (no # yet)
  - Water Your Rights (no # yet)

### Content fixes applied across pages
- Dallas corporate address (14800 Quorum Dr., STE 150) replacing old Fort Worth footer
- `info@cnattitle.com` replacing `classes@cnattitle.com`
- "Instructor: Bradley Patterson" line removed (TREC wants Provider, not individual)
- Real CNAT logo (`/img/cnat-logo-white.png`) replacing placeholder SVG in nav
- Course descriptions brightened from `#8892a4` to `#d9dee6` for readability
- Catalog filter buttons bumped to 44px for mobile touch targets
- Home page OG tags + images restored (got stripped during course pages rewrite earlier)
- Legal Update page OG tags + client_reference_id for Stripe
- Course numbers added/removed per latest TREC info

---

## Current State

**Working:**
- `classes.cnattitle.tech` — live, all pages deploy on git push, full flow tested end-to-end (registration → Stripe → confirmation email)
- `classes.cnattitle.com` — DNS resolves, subdomain provisioned on Hostinger, **git auto-deploy being wired by user RIGHT NOW**
- GHL workflows active: Legal Update Webhook + Stripe Payment Confirmation, both tested with fake payloads and real Playwright form submissions, contacts deleted after testing

**In progress:**
- 🟡 User is in hPanel setting up `.com` git auto-deploy → this must complete before next-session work proceeds

**Not yet started:**
- 🔴 Task #5: n8n reminder workflow (`gyGoBDZZUR88Xafm`) — must filter by `paid-april-2026` tag (NOT `legal-update-april-2026`) so unpaid registrants don't get reminders. Date triggers for Apr 27/28/29/30 need verification.

---

## Known Issues / Still Broken

- 🟡 **"Amount paid" merge field in confirmation email never rendered correctly** — user deleted that line from the template to ship. Leave for now unless product requires.
- 🟡 **Hostinger git auto-deploy occasionally flakes** — manual "Deploy Now" button in hPanel fixes it. Not root-caused yet. Keep an eye out.
- 🟢 **Real Estate in the Palm of Your Hand vs CNAT Agent ONE** — confirmed same course, mapped #30671-RECE to CNAT Agent ONE
- 🟢 **Water Your Rights vs Legal Update I water rights chapter** — brand-voice-writer agent flagged overlap. User may want to cross-link or differentiate. Left alone for now.
- 🟢 **IG collaborators** — GHL API doesn't support. User adds manually via IG app after posting.

---

## Files Modified / Created

All in repo `https://github.com/iambradass/classes` (branch `main`):

**Homepage + Legal Update registration**
- `index.html` — full site homepage with upcoming class tile, course catalog, footer
- `legal-update-april-2026/index.html` — registration page (class select → attendance → info → Stripe)

**Course detail pages (14)**
- `courses/legal-update/index.html` — template page
- `courses/cnat-agent-one/index.html`
- `courses/contracts-competence/index.html`
- `courses/create-with-canva/index.html`
- `courses/crm-to-roi/index.html`
- `courses/ensuring-insurance/index.html`
- `courses/farm-to-table/index.html`
- `courses/keeping-ai-on-things/index.html`
- `courses/re-technology/index.html`
- `courses/scripty-business/index.html`
- `courses/social-media-bootcamp/index.html`
- `courses/trec-advertising/index.html`
- `courses/video-marketing/index.html`
- `courses/water-your-rights/index.html`

**Assets**
- `img/cnat-logo-white.png` — CNAT white logo (258×82, used in nav)
- `og/og-home.png` — 1200×630 homepage social preview
- `og/og-legal.png` — 1200×630 Legal Update social preview
- `og/ig-legal.png` — 1080×1080 IG square graphic (used in Social Planner post)

---

## Exact Next Steps (in order)

### 1. Verify `.com` is live after user's hPanel deploy (~2 min)
```bash
# Wait for deploy confirmation from user
curl -sIL https://classes.cnattitle.com | head -5
dig classes.cnattitle.com +short  # should show 145.223.104.252
```
If it 200s and serves the same content as `.tech`, proceed.

### 2. Flip canonical + OG URLs from `.tech` to `.com` in all pages (~5 min)
Use Python + `gh api` to pull each file, regex-replace `classes.cnattitle.tech` → `classes.cnattitle.com` ONLY in these meta tags:
- `<link rel="canonical" href="...">`
- `<meta property="og:url" ...>`
- `<meta property="og:image" ...>`
- `<meta name="twitter:image" ...>`
Files to update: `index.html`, `legal-update-april-2026/index.html`, all 14 `courses/*/index.html` pages.

Leave webhook URLs, Stripe URLs, and relative paths alone.

### 3. Add `.htaccess` 301 redirect on `.tech` (preserves path + query string)
Create `.htaccess` at repo root:
```
RewriteEngine On
RewriteCond %{HTTP_HOST} ^classes\.cnattitle\.tech$ [NC]
RewriteRule ^(.*)$ https://classes.cnattitle.com/$1 [R=301,L,QSA]
```
Push to `main`. Both sites auto-deploy from same repo, so `.com` ignores this rule (Host header doesn't match) and `.tech` redirects all traffic.

### 4. Verify old rep links still work (redirect test)
```bash
curl -sIL "https://classes.cnattitle.tech/legal-update-april-2026/?ref=jami" | head -15
# Expected: 301 Location: https://classes.cnattitle.com/legal-update-april-2026/?ref=jami
curl -sL "https://classes.cnattitle.com/legal-update-april-2026/?ref=jami" | grep -i "jami_cortez"
# Expected: dropdown matches jami_cortez as before
```

### 5. Re-scrape OG previews on platforms
User action, not automatable without auth:
- Facebook Debugger: https://developers.facebook.com/tools/debug/
- LinkedIn Post Inspector: https://www.linkedin.com/post-inspector/
- Paste both `classes.cnattitle.com/` and `classes.cnattitle.com/legal-update-april-2026/`
- Click "Scrape Again"

### 6. Update memory files
`/Users/bradleypatterson/.claude/projects/-Users-bradleypatterson/memory/MEMORY.md`:
- Add `classes.cnattitle.com` to the deployment table (row alongside S817, JHL, TitleEdge, ai.cnattitle, boonepdr)
- Update `project_classes_deploy.md` — mark dual-deploy live, `.tech` now redirects, git auto-deploy working on both subdomains
- Create `feedback_dual_domain_deploy.md` capturing the redirect strategy as a pattern for future domain migrations

### 7. (Optional but recommended) Build `/deploy-classes` skill
Following the pattern of `/deploy-s817`, `/deploy-jhl`, `/deploy-titleedge`:
- Path: `/Users/bradleypatterson/.claude/skills/deploy-classes/SKILL.md`
- Description: "Deploy changes to classes.cnattitle.com / classes.cnattitle.tech. Both domains auto-pull from iambradass/classes repo. Use GitHub API for file pushes (git CLI hangs)."

### 8. Move on to Task #5 (n8n reminder workflow)
See dedicated section below.

---

## Task #5 — n8n Reminder Workflow (not yet started)

**Workflow:** `gyGoBDZZUR88Xafm` ("CNAT Class Reminder Engine") on self-hosted n8n at `https://n8n.cnattitle.tech`

**What needs verifying / fixing:**
1. Trigger should query GHL contacts filtered by tag `paid-april-2026` (NOT `legal-update-april-2026` — that includes people who never paid)
2. Date-based sends scheduled:
   - Apr 27 (day before Legal I) — reminder email + SMS
   - Apr 28 (morning of Legal I) — day-of email with Zoom link or address
   - Apr 29 (day before Legal II) — reminder
   - Apr 30 (morning of Legal II) — day-of
3. For Zoom attendees specifically, must include Zoom join link in the day-of email
4. For In-Person attendees, include address + parking notes
5. Use n8n MCP (`n8n - VPS` connector, credentials baked in) to inspect + modify

**Starting point:** Pull workflow with `n8n_get_workflow` tool, review current state, determine what needs changing.

---

## Task #6 — `classes.cnattitle.com` DNS Migration (in progress — this handoff)
See "Exact Next Steps" steps 1-7 above. Will be complete once those finish.

---

## Context / Notes for Next Session

### Repo + deploy
- Single repo `iambradass/classes` serves BOTH subdomains (addon setup, both `.tech` and `.com` pull from main branch)
- Push via GitHub API (git CLI hangs in this environment — memory note exists: `feedback_github_image_deploy.md`)
- Hostinger auto-deploy: usually 30-60s after push; occasionally requires manual "Deploy Now" in hPanel

### Voice/content notes (CNAT)
- Direct, no hype, no emojis in body text
- Treats agents as pros
- Bradley Patterson = Director of Technology at CNAT + TREC instructor (Provider #10034)
- Tagline: `#AlwaysBeClosing · Texas Style`
- Footer email: `info@cnattitle.com`
- HQ: `14800 Quorum Dr., STE 150, Dallas, TX 75254`

### GHL workflow patterns
- Dynamic tagging via `{{contact.class_session}}` — one workflow handles all monthly sessions
- Existing "assign to rep" workflow keys off `rep-{{contact.sales_rep}}` tag
- Amount Paid uses form's `class_price` (already in dollars), NOT Stripe's `amount_total` (which is cents) — this was a deliberate design decision after email merge field bug
- Confirmation email template: Bradley removed the "Amount paid: $X" line because the merge field never rendered — don't bring it back unless there's a real fix

### Testing approach
- Use Playwright for form tests — navigate + fill + submit
- Fire matching Stripe webhook via curl to complete the loop without real payment
- Delete test contacts from GHL after each test (search + delete via MCP)
- Never leave test contacts in production

### Active session IDs for reference
- Last session commits on `iambradass/classes` go through these:
  - Legal Update template: `c30ccc0f`
  - 13 course pages: `0af6a8ce`, `abe7301a`, `14899b19`, `928aaa9e`, `f028e912`, `f6d2ac42`, `2dbbcd43`, `7c59ba8b`, `67a22c8c`, `1b9e0dbd`, `25699a8e`, `4cd3396f`, `4870bbfc`

---

## Deploy Status

- [x] Changes committed (all via GitHub API)
- [x] Pushed to GitHub (main branch)
- [x] Hostinger auto-deploy live on `.tech`
- [ ] Hostinger auto-deploy pending on `.com` (user setting up in hPanel this moment)
- [ ] Canonical URL flip (.tech → .com) — step 2 of next steps
- [ ] `.htaccess` 301 redirect — step 3
- [ ] OG re-scrape on FB/LinkedIn — step 5 (user action)
- [ ] Memory file update — step 6
