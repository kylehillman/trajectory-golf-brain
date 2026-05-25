# Golf Simulator Business Project — Full Handoff for Grok

**Doc version:** 2.0
**Prepared:** 2026-05-17
**Owner:** Kyle R. Hillman (50%) + Alexis Kalletta (50%)
**Business:** Trajectory Golf LLC — 24/7 unmanned indoor golf simulator club, Bastrop, TX
**Target opening:** Q1 2027

> This document supersedes the prior `Trajectory_Golf_Handover_to_Grok.md` (May 17 v1.0) where they conflict. Notably: Optix is now the live member backbone, and the Stripe Payment Link / ConvertKit flow described in `trajectory-golf-web/HANDBACK.md` is no longer the live signup path.

---

## Table of Contents

1. [Project Overview & Goals](#1-project-overview--goals)
2. [Current Project State](#2-current-project-state)
3. [Key Documents Summary](#3-key-documents-summary)
4. [Important Decisions & Rationale](#4-important-decisions--rationale)
5. [File Structure Overview](#5-file-structure-overview)
6. [Open Tasks & Next Priorities](#6-open-tasks--next-priorities)
7. [Reusable Context & Preferences](#7-reusable-context--preferences)
8. [Critical Code Snippets](#8-critical-code-snippets)
9. [Risks to Carry Forward](#9-risks-to-carry-forward-do-not-lose)
10. [Handoff Instructions for Grok](#10-handoff-instructions-for-grok)

---

## 1. Project Overview & Goals

**Trajectory Golf LLC** is a 24/7 unstaffed indoor golf simulator membership club opening Q1 2027 in Bastrop, TX. Three premium "launch bays" running TrackMan iO. Members sign up, pay, book, and unlock the door entirely self-service — no on-site staff ever.

**Founder:** Kyle is a SpaceX Starship launch engineer in Bastrop. The brand intentionally borrows aerospace language (*Trajectory · Apex · Approach*) without being a theme park. Cofounder is Kyle's wife Alexis Kalletta (kept maiden name — use **Kalletta** on legal/banking).

**Key model decisions (locked):**

- 3 bays, single site (down from earlier 4-bay / 2-location plans)
- 3 tier pricing: Approach / Trajectory / Apex
- 50 founding members, rate-locked-for-life, $99 refundable deposit
- Independent (rejected Back Nine / X-Golf franchise)
- **No alcohol** → single LLC structure, no TABC, no on-site staff requirement
- SBLOC financing against SpaceX equity (~6.5% interest-only) — not SBA debt
- Owner-operator: Kyle is GC + software builder + fabricator where licensure permits

**Revenue mix (mature):** ~70% memberships / ~10% walk-in / ~8% corporate / ~7% lessons / ~3% leagues / ~2% F&B.

**Hard tripwire:** 25 paying members by M12 or the model fails to launch profitably.

---

## 2. Current Project State

### 2.1 Tech stack & architecture

| Layer | Choice | Status |
|---|---|---|
| Marketing site | Next.js 16 (App Router) + React 19 + TS + Tailwind v4 | **Live**, deployed on Vercel at `trajectory-golf.com` |
| Member backbone (signup, billing, plans, bookings, mobile app) | **Optix** (venue 25863 at `trajectorygolf.optixapp.com`) | **Live** (live-mode Stripe connected) |
| Payments | Stripe via Optix's connected account `acct_1TWPCGPRt0OtMQlg` | **Live mode** |
| Lead capture | Optix `userCreate` mutation (`is_lead: true`) via server action | **Live** |
| Smart access | Kisi (KisiFit, ~$299/mo, 10 doors) | Selected, not yet purchased |
| Cameras | Reolink 4K PoE + 16-ch NVR → Verkada at scale | Spec'd |
| Sim hardware | TrackMan iO + Full Swing Pro **or** Foresight GCHawk enclosure | Tracking decided; enclosure pending |
| Power automation | Architecture B (always-on PCs; only projector/lights via Kisi webhook → Home Assistant → smart plugs) | Decided |
| Mobile app | Vanilla Optix app for Phase 1; white-label Phase 2 (gated on LLC + Apple Org account, +$99/mo) | Phase 1 live |
| Analytics | PostHog (lazy) + Vercel + Meta Pixel + Google Tag + **Plausible** (cookieless; now using private aggregate API + read-only key for total visitors since May 2026) — all no-op w/o env | Code complete (total visitors counter live) |
| Error monitoring | Sentry browser (lazy) | Code complete, needs DSN |
| Hosting | Vercel | Live |
| Domain & email | trajectory-golf.com (Squarespace registrar); `hello@trajectory-golf.com` → Gmail forward | Live |

**Architecture flow (Path A — hosted handoff):**

```
trajectory-golf.com (Next.js)
  → Visitor browses landing page / tier cards
  → "Reserve" CTA deep-links to Optix hosted signup widget with plan pre-selected:
    https://trajectorygolf.optixapp.com/signup/?plans=58551,58552,58553
  → Optix collects card-on-file + $99 deposit + plan enrollment
  → Optix is the source of truth for members, bookings, billing, mobile app
  → Daily Vercel cron (/api/cron/recalc-trial-days) keeps founder plans'
    free_trial_days set so trials end on opening day (2027-01-01)
```

The website never holds member data; Optix does. Server actions are the only thing that ever touches the Optix API token.

### 2.2 What's built and deployed

**Website (`/trajectory-golf-web`):**

- Sticky nav with "Reserve in the member portal" CTA → Optix hosted signup
- Hero with UTM-aware variant (local Bastrop framing vs default)
- Animated trajectory arc SVG + starfield background
- Live spot counter (queries Optix `accountPlans` for plans 58551/52/53, dedupes by `account_id`, caches 60s)
- Tier cards with featured Trajectory middle, each deep-linking to its Optix plan ID
- Sections: Problem, Facility, Coming Soon, Founder Benefits, Founder Note, FAQ, HowItWorks, Updates (newsletter)
- Newsletter signup → Optix lead user via `userCreate(is_lead: true, source: "website-newsletter")`
- Sub-pages: `/pricing`, `/faq`, `/privacy`, `/terms`, `/founder/[n]` (founder shareable URL with dynamic OG image)
- Daily cron at 03:00 UTC recalculates `free_trial_days` on the 3 founder plans so trials always end ~opening day
- `LocalBusiness` + `SportsActivityLocation` JSON-LD
- `next/og` generated 1200×630 social image
- Lighthouse: home `/` Mobile 93/100/96/100 (Perf/A11y/BP/SEO)

**Optix (venue 25863, live):**

- 1 location (`28050`), 3 bays (`632663`, `632755`, `632754`), 11 lockers, 3 add-on products (Extra Hour, Guest Pass, League Season Entry)
- 6 plan templates + 1 Day Pass — see §3.6
- `payment_method_policy: ALWAYS_REQUIRED`
- Standard plans hidden (`onboarding_enabled: false`, `promote: false`) until 50-founder cap fills
- Day Pass temporarily disabled (re-enable at opening)
- **⚠️ Founder deposits temporarily $1** as a safety-net for the first real-card end-to-end test — bump back to $99 after test passes (tracked as live task)

### 2.3 What's NOT yet done

| Area | Status |
|---|---|
| Site / lease | Commercial RE broker **engaged** (2026-05-25); touring not yet started; no lease signed |
| Simulator enclosure decision | Full Swing Pro vs Foresight GCHawk — reference calls outstanding |
| Insurance | Coverage stack defined; quotes not yet collected |
| TX Sales Tax Permit | Not yet filed (Comptroller Webfile, free, ~30 min) |
| Mercury bank account | **Application submitted; in review** as of 2026-05-25 |
| Operating Agreement | Draft exists; needs Chris W. Kirby (Bastrop Law Group) attorney review — email not yet sent |
| Insurance quotes | Stack defined ($2M GL + $1M cyber + $2M umbrella + property + WC); not collected |
| PGA pro recruitment | Planned: ~14 lessons/week, 30% facility share, 1099 |
| Founder presale outreach | Conversations started ad-hoc; formal 100-name target list not yet built |
| Trademark "Trajectory Golf" | TESS search not yet run; Class 41 filing not started |
| Kisi hardware purchase | Pending lease |
| Optix Kisi integration / `member_booking_created` webhook | Optix app manifest currently `{}` |
| Day Pass plan re-enable | Gated on facility opening |
| Bump founder deposit $1 → $99 | After first real card test |
| Apple Developer Org account / white-label Optix app | Phase 2, gated on D-U-N-S (LLC approved 2026-05-13) |

### 2.5 Hermes Agent — Kyle's personal AI surface (new 2026-05-25)

Kyle now runs **Hermes Agent (Nous Research, v0.14.0)** locally on his Mac as a launchd background service.

- Install path: `~/.hermes/hermes-agent/` (venv Python 3.11)
- Service: `ai.hermes.gateway` (launchd plist at `~/Library/LaunchAgents/ai.hermes.gateway.plist`)
- Provider: **xAI Grok OAuth (SuperGrok subscription)** — no API key, OAuth refresh handled automatically
- Active model: **grok-4.3** (confirmed via self-identification)
- Messaging surface: Discord bot **"Trajectory Golf"** in his private "Trajectory Bot" server (`#hermes` channel set as home). `@`-mention or DM works.
- Allowlist: `GATEWAY_ALLOW_ALL_USERS=true` (private server, sole user)
- Credentials in `~/.hermes/.env` (never commit): `DISCORD_BOT_TOKEN`, `DISCORD_CHANNEL_ID`, `GATEWAY_ALLOW_ALL_USERS=true`
- Logs: `~/.hermes/logs/gateway.log` (`tail -f` for live)
- Deferred: X (Twitter) posting via xurl skill — decision pending on Path A (free X dev portal, 500 posts/mo) vs Path B (OpenTweet $5.99/mo)

**This means Grok is reachable from anywhere Kyle has Discord** (phone, web, desktop) — the Grok-in-Hermes brain and the Grok web project brain are the same model; they coordinate via HANDOVER.md.

### 2.4 Stripe integration — important clarifying note

The `HANDBACK.md` in `trajectory-golf-web/` describes a Stripe **Payment Link + ConvertKit** flow that **is no longer the live signup path**. After the 2026-05-15 pivot to Optix, billing moved entirely into Optix's connected Stripe account. The legacy webhook at `app/api/stripe/webhook/route.ts` still handles `checkout.session.completed` (for any lingering Payment Link payments) and `invoice.payment_succeeded` (Optix's billing path), but the **canonical deposit flow now runs through Optix's hosted signup widget, not through Stripe Payment Links on our site**. The webhook signing secret is still test-mode — production needs a live-mode webhook endpoint if Grok decides to revive that path.

---

## 3. Key Documents Summary

### 3.1 Business plan / financial model (v3, working)

| Metric | Value |
|---|---|
| CAPEX | ~$291K (revised from $358K after 4-bay → 3-bay) |
| Financing | SBLOC vs SpaceX equity, ~6.5% interest-only, $250K draw against $400–500K line |
| Annual debt service | ~$14K |
| Mature member target (realistic) | 55–65; optimistic 70–85; stretch 90–100 |
| Member break-even | ~38 members at mature ARPU |
| Realistic mature EBITDA | ~$121K |
| First positive net cash month | M12 |
| Y3 monthly net | ~$11K |
| Personal exposure if business fails | $300–400K |
| Y1 marketing budget | $20K (front-loaded through M6) |
| Personal liquidity reserve required | $15K |

Files (Excel/Word, not in repo — Kyle's local drive):

- `Trajectory_Golf_Model_v3.xlsx`
- `Trajectory_Golf_Stress_Test_v2.xlsx`
- `Bastrop_GolfSim_Gantt_v2.xlsx`
- `Phase1_Execution_Playbook.docx`

### 3.2 Operating Agreement (`files/legal/Trajectory_Golf_LLC_Operating_Agreement.docx`)

13-page Texas-compliant multi-member LLC OA. Articles include community-property acknowledgment (§3.2) and full death/disability/divorce machinery (Article VIII). Needs Chris W. Kirby review before execution.

### 3.3 Site Criteria One-Pager (`files/legal/Trajectory_Golf_LLC_Site_Criteria.docx`)

Broker-ready spec. Hard requirements include **11 ft minimum ceiling height at hitting position** (kills more candidate spaces than anything else), 3,500 SF min / 4,200–4,500 preferred, 200A electrical, ADA, 10+ lighted parking spaces, no competing golf tenant.

### 3.4 Schedule (Gantt v2)

Major milestones:

- May 2026: LLC + EIN + bank + insurance + presale (in progress)
- Aug 2026: target lease signing
- At lease signing: TrackMan pre-order (~18 weeks lead)
- Sep 2026 – Q1 2027: buildout
- Q1 2027: soft open → grand open

### 3.5 Legal entity reference data

| Item | Value |
|---|---|
| Legal name | Trajectory Golf LLC |
| State | Texas |
| SOS file # | 0806599577 |
| SOS Session ID | 051326XE5414 |
| SOS Document # | 1586692250002 |
| Filing date | 2026-05-13 |
| EIN | 42-2596674 |
| Registered agent | Northwest Registered Agent LLC, 5900 Balcones Drive STE 100, Austin TX 78731 |
| BOI/FinCEN | NOT REQUIRED (March 21, 2025 IFR exempts domestic LLCs) |
| First TX Franchise Tax filing | 2027-05-15 ("no tax due" report) |

### 3.6 Optix integration contract

**Plan ID mapping** (live in venue 25863; verify with `planTemplates` query if drift suspected):

| Tier | Founder ID (enabled) | Standard ID (hidden) | Founder $ | Standard $ |
|---|---|---|---|---|
| Approach | 58551 | 58554 | $169 | $199 |
| Trajectory (featured) | 58552 | 58555 | $239 | $279 |
| Apex | 58553 | 58556 | $299 | $349 |
| Day Pass (one-shot, $50) | 58557 | — | — | — |

**Inventory:** Location 28050 · Bays 632663 / 632755 / 632754 · 11 lockers · 3 add-ons (Extra Hour, Guest Pass, League Season Entry).

**Key mutations:**

- `userCreate(is_lead: true, source, notify_user_by_email: false)` — newsletter / lead capture
- `userSignupCommit(plan: { plan_template_id })` — Path B (not currently used; hosted widget handles this)
- `planTemplateCommit(plan_template_id, input)` — used by daily cron to set `free_trial_days`
- `featureSettingsUpdate(input: FeatureSettingsInput)` — toggle `payment_method_policy` etc.

**Endpoint:** `https://api.optixapp.com/graphql` · Auth: `Bearer <OPTIX_ORG_TOKEN>` · Account: `krhillman1234@gmail.com`.

---

## 4. Important Decisions & Rationale

| Decision | Outcome | Why |
|---|---|---|
| Franchise vs independent | Independent | No royalties; full brand control; Kyle has tech/build skill |
| Bays at launch | 3 (down from 4) | Back Nine convergence + Bastrop TAM math |
| Alcohol | **No** | Eliminates TABC complexity, dram shop, staffing requirement |
| Entity structure | Single LLC | No alcohol → no holding/operating split needed |
| Pricing | 3 tiers ($199/$279/$349 std; 15% founder discount) | More wallet share; mirrors Back Nine model |
| App build vs Optix | **Optix wins** — custom Expo app archived 2026-05-15 to `nineteenth-hole-app.archived/` | Don't build software before 50+ paying members; Optix gives members + bookings + plans + billing + mobile app off the shelf |
| Signup flow | Path A: website → Optix hosted signup widget | Avoids holding member PII on our side; Optix owns PCI; faster to launch |
| Financing | SBLOC vs SBA | 6.5% interest-only vs 10.5% amortizing; SpaceX equity over-collateralizes |
| GC | Kyle as GC where licensure permits | Major cost advantage |
| Site count | One location | Validate before expansion; previously rejected 2-location plan |
| F&B | Self-serve fridge + coffee; no bar | Aligns with no-alcohol + no-staff |
| Power automation | Architecture B (always-on PCs; auto only projector/lights) | Avoids Windows-reboot-at-3am failure mode |
| SpaceX direct outreach | Avoid | Kyle does not mix day job and business |
| BOI filing | Skip | Domestic LLC exempt as of 2025-03-21 IFR |
| Spot counter source | Live Optix query, 60s cache, fallback 0 claimed | Always-correct over hand-edited config |
| Marketing site framework | Next.js 16 + Tailwind v4 (CSS-first `@theme`) | Vercel-native, server actions, fast iteration |
| Brand accent token | `--accent` (was `--amber` in handover doc) | Role-named not color-named |
| Apple Dev account | Individual now, transfer to Org post-LLC | App Store work in parallel with LLC; Optix WLA requires Org |

---

## 5. File Structure Overview

```
/Users/kylehillman/golfsimulator/
├── trajectory-golf-website-handover.md     # Original website brief (May 14, 2026)
├── trajectory-golf-web/                    # LIVE Next.js site
│   ├── README.md                           # How to run, edit content
│   ├── HANDBACK.md                         # Build report (PARTIALLY STALE re: Stripe/ConvertKit)
│   ├── AGENTS.md                           # "Next.js 16 has breaking changes — read node_modules docs first"
│   ├── package.json                        # next 16.2.6, react 19.2.4, stripe, zod, posthog
│   ├── vercel.json                         # daily 03:00 UTC cron
│   ├── next.config.ts                      # inlineCss experimental
│   ├── .env.local.example                  # All env vars documented
│   ├── app/
│   │   ├── (marketing)/
│   │   │   ├── layout.tsx                  # Nav + Footer wrapper
│   │   │   ├── page.tsx                    # Landing page (UTM-aware hero variant)
│   │   │   ├── pricing/page.tsx
│   │   │   ├── faq/page.tsx
│   │   │   ├── privacy/page.tsx
│   │   │   └── terms/page.tsx
│   │   ├── founder/[n]/                    # Founder-numbered shareable URL + dynamic OG
│   │   ├── api/cron/recalc-trial-days/     # Daily Optix free_trial_days update
│   │   ├── layout.tsx                      # Root: fonts + analytics
│   │   ├── globals.css                     # Design tokens (CSS-first Tailwind v4 @theme)
│   │   └── opengraph-image.tsx             # 1200x630 OG
│   ├── components/                         # 24 components — hero, tiers, tier-card,
│   │                                       #   facility, problem, founder-note,
│   │                                       #   faq, faq-item, footer, nav,
│   │                                       #   trajectory-arc, starfield, spot-counter,
│   │                                       #   reserve-button, select-tier-button,
│   │                                       #   newsletter-signup, updates-section,
│   │                                       #   how-it-works, coming-soon,
│   │                                       #   posthog-pageview, meta-pixel, google-tag,
│   │                                       #   wordmark, icons
│   ├── lib/
│   │   ├── tiers.ts                        # Source of truth: name, price, perks, Optix plan ID
│   │   ├── faq.ts
│   │   ├── spots.ts                        # Live Optix query for spots claimed/remaining
│   │   ├── optix.ts                        # Optix URL constants + getOptixSignupUrl()
│   │   ├── analytics.ts                    # track() + trackConversion() helpers
│   │   └── structured-data.ts              # JSON-LD
│   ├── actions/
│   │   ├── newsletter.ts                   # subscribeNewsletter() → Optix userCreate
│   │   └── newsletter-types.ts
│   ├── types/window.d.ts                   # fbq, gtag global types
│   └── scripts/set-env.sh
├── nineteenth-hole-app.archived/           # ABANDONED Expo / RN app (last good
│                                           #   commit: 301958b "chore: snapshot before pivot")
├── files/
│   ├── CLAUDE.md                           # STALE — describes the abandoned Expo app
│   │                                       #   under the old "19th Hole" name. Useful as
│   │                                       #   reference for design intent but not for
│   │                                       #   anything currently being built.
│   ├── CLAUDE_CODE_KICKOFF.md              # Original Expo kickoff prompt (stale)
│   ├── legal/
│   │   ├── Certificate of Formation.pdf
│   │   ├── EIN.pdf                         # CP 575 letter
│   │   ├── COC packing Slip - LLC.pdf
│   │   ├── Trajectory_Golf_LLC_Operating_Agreement.docx
│   │   ├── Trajectory_Golf_LLC_Site_Criteria.docx
│   │   ├── PRIVACY_POLICY_DRAFT.md
│   │   ├── TERMS_OF_SERVICE_DRAFT.md
│   │   └── README.md                       # How to swap placeholders + lawyer-review notes
│   └── Grok:Claude Handover/
│       ├── Trajectory_Golf_Handover_to_Grok.md     # v1 (May 17) — superseded
│       └── Trajectory_Golf_Handover_to_Grok_v2.md  # This document
├── Icons/
└── .claude/settings.local.json
```

**Naming note for Grok:** "The 19th Hole" was an *internal* app project name from before the brand was decided. The LLC, brand, and live customer-facing name is **Trajectory Golf**. If you see "19th Hole" in older docs (`files/CLAUDE.md`, the archived Expo folder), that's the same business under the prior internal name — disregard the name, retain the architectural context only if relevant.

---

## 6. Open Tasks & Next Priorities

### Recently completed (2026-05-25)

- ✅ Mercury bank application **submitted** (in review)
- ✅ Commercial RE tenant-rep broker **engaged**
- ✅ Grok web project **provisioned** with Custom Instructions; live HANDOVER fetch confirmed working
- ✅ Hermes Agent installed + gateway running as launchd service; Grok-4.3 via SuperGrok OAuth confirmed; Discord bot live

### Immediate this week (re-ranked 2026-05-25)

1. **Test founder signup end-to-end with a real card** through Optix hosted widget — currently $1 deposit safety-net. Blocks every other founder conversation from converting.
2. After test passes: **bump Optix founder deposits $1 → $99** via `planTemplateCommit`
3. **Email Chris W. Kirby** at Bastrop Law Group (512-240-9565) to book a 1-hour OA review consult; send draft OA ahead of time. 2–3 week calendar lead time → start now.
4. File **Texas Sales Tax Permit** via Comptroller Webfile (free, ~30 min)
5. **TESS trademark search** on "Trajectory Golf" → file Class 41 (~$350 DIY) before signage / merch / lease commitment
6. Open Stripe **live-mode webhook** endpoint (if reviving the deposit-confirmation tagging path) — current webhook secret is test mode

### Founding member campaign (M0–M3)

7. Formalize the 100-name Bastrop-golfer target list (Kyle has started ad-hoc conversations — convert to a tracked pipeline: name, source, stage, last touch, next action)
8. Begin 30 in-person founding-member conversations
9. Capture 50 founders at $99 refundable deposit
10. Weekly founder construction updates (auto via Optix lead emails)
11. Get permission to use 2–3 supporter quotes on the site for social proof

### Site / build (M0–M6)

12. **Tour 3–5 candidate spaces with broker** using Site Criteria one-pager; verify **11 ft ceiling height** at hitting position on each (kills more spaces than anything else)
13. Lease negotiation (DO NOT SIGN before SBLOC is locked — financing gated on SpaceX IPO)
14. TrackMan iO pre-order at lease signing
15. Finalize enclosure: Full Swing Pro vs Foresight GCHawk (reference customer calls — 18-week lead from order)
16. Collect 3 insurance quotes against defined stack (quotes don't bind; gather now)

### Tech (deferred until opening date is firmer)

17. Configure Optix `member_booking_created` / `member_booking_cancelled` webhooks → Kisi door provisioning
18. Purchase Kisi hardware + readers (7 doors)
19. Re-enable Day Pass plan 58557 (`onboarding_enabled: true`)
20. Open standard plans 58554/55/56 once 50-founder cap fills
21. **Apple Developer Org account application + D-U-N-S** (LLC now formed → unlocks this; ~6–10 week wait, start now to parallel the founder presale)
22. Update `OPENING_DATE_ISO` in `app/api/cron/recalc-trial-days/route.ts` when lease signed
23. **Hermes Agent — decide X tweeting path** (Path A: xurl + free X dev portal, 500 posts/mo; Path B: OpenTweet $5.99/mo). Account choice: @KyleHillman1 vs new @TrajectoryGolf handle.

### Operations (M-3 to opening)

24. **PGA pro candidate scouting** (Austin + Bastrop teaching pros — soft outreach to 5–10 candidates now; hire closer to opening)
25. Corporate outreach to Bastrop employers (NOT direct SpaceX)
26. League L1 ready for M+3 post-open; L2 M+6; L3 M+9

---

## 7. Reusable Context & Preferences

### Working style (Kyle)

- **Owner directs, AI codes.** Kyle reviews and decides; the assistant does the writing. Before any major decision (library, schema, breaking change, paid signup) → pause and confirm with one short message. After each meaningful chunk → pause and summarize.
- **Mobile-first short messages.** Many sessions happen between launch ops at SpaceX. Long preambles waste his time.
- **Direct, numbers-focused communication. No hedging.** "It depends" is rarely the right answer. Pick a side and defend it; he will push back if he disagrees. He has explicitly thanked Claude for pushing back rather than validating.
- **Structured tables and scenario comparisons** over prose for any quantitative or trade-off decision.
- **Scenario-based modeling** with explicit break-even thresholds and stress tests. He has rejected assumptions that felt too optimistic (rejected $2,800 prepay model; rejected SBA at 10.5% for SBLOC at 6.5%; rejected 4 bays for 3).
- **Hands-on execution as advantage.** Kyle is GC, software builder, fabricator. Don't suggest contractors for tasks he can do himself unless there's a real reason (licensure, time, expertise gap).
- **Sequencing discipline:** entity → presale validation → lease → equipment → buildout → tech → soft open. Don't pull future steps forward without a reason.
- **Owner wants to ship, not learn.** Don't pad responses with tutorials unless asked.
- **Don't sign up for services on Kyle's behalf.** Wait for credentials. Don't volunteer to build Phase 2+ features (admin dashboard, league engine, TrackMan integration) even if they seem natural.

### Code conventions (web project)

- **Node 20 via nvm.** System Node is 16 (Homebrew). Always `nvm use 20` before `npm`/`npx`/`next`. Repo uses Next.js 16 which requires Node 20+.
- **Next.js 16 has breaking changes** vs older training data. `AGENTS.md` instructs: read `node_modules/next/dist/docs/` before writing Next.js code.
- **Tailwind v4 is CSS-first** — design tokens live in `@theme` blocks in `app/globals.css`. No `tailwind.config.ts`.
- **Brand accent token is `--accent`** (not `--amber` as the handover doc says).
- **TypeScript strict.** No `any`. No `@ts-ignore` without a comment.
- **All Optix API calls go through server actions** in `actions/` or `app/api/`. Never expose `OPTIX_ORG_TOKEN` to the browser.
- **Fail-soft on integration errors.** See `actions/newsletter.ts` — Optix duplicate-email returns a soft success rather than scaring the visitor.
- **Forms: server actions + Zod.** No third-party form backends.
- **Lib data files are user-editable.** `lib/tiers.ts`, `lib/faq.ts` are designed for non-code edits.
- **Don't reintroduce a custom Stripe checkout for the deposit.** Optix's hosted signup owns billing now.

### Brand vocabulary

- "**Launch bays**", never "hitting bays"
- Tier names: **Approach / Trajectory / Apex**
- Type stack: **Fraunces** (serif headlines, italic for emphasis), **Inter** (body), **JetBrains Mono** (data/labels)
- Dark palette, **amber accent** (`#E8A14C`)
- Voice: **honest, not hyped.** No exclamation marks. No "amazing." No "revolutionary." Concrete not abstract. Confident not pleading. Dry humor allowed. Italics on Fraunces are the only emphasis tool — no bold in headlines.

---

## 8. Critical Code Snippets

### 8.1 `lib/tiers.ts` — single source of truth for pricing

```ts
export type TierId = "approach" | "trajectory" | "apex";

export const TIERS: Tier[] = [
  { id: "approach",   name: "Approach",   tagline: "Entry / Get In The Air",
    founderPrice: 169, standardPrice: 199, featured: false,
    optixPlanTemplateId: "58551", perks: [...] },
  { id: "trajectory", name: "Trajectory", tagline: "Mid-Tier / Most Members",
    founderPrice: 239, standardPrice: 279, featured: true,
    optixPlanTemplateId: "58552", perks: [...] },
  { id: "apex",       name: "Apex",       tagline: "Premium / Top Of Arc",
    founderPrice: 299, standardPrice: 349, featured: false,
    optixPlanTemplateId: "58553", perks: [...] },
];
```

### 8.2 `lib/optix.ts` — signup deep-link

```ts
export const OPTIX_VENUE_URL = "https://trajectorygolf.optixapp.com";
export const OPTIX_SIGNUP_URL = `${OPTIX_VENUE_URL}/signup/`;
export const OPTIX_PLAN_DAY_PASS = "58557";   // disabled until opening
export const OPTIX_FOUNDER_PLAN_IDS = ["58551", "58552", "58553"] as const;
export const OPTIX_FOUNDER_SIGNUP_URL =
  `${OPTIX_SIGNUP_URL}?plans=${OPTIX_FOUNDER_PLAN_IDS.join(",")}`;
```

### 8.3 `lib/spots.ts` — live Optix spot counter (60s cache, fail-soft)

Aliased GraphQL query against `accountPlans(plan_template_id:..., limit:200)` for each of 58551/52/53, dedupe by `payer_account.account_id`, exclude `status: "CANCELED"`, clamp at `TOTAL_FOUNDING_SPOTS = 50`. On any fetch failure: defaults to **0 claimed** (under-report > misreport).

### 8.4 `actions/newsletter.ts` — Optix lead capture

```graphql
mutation NewsletterSubscribe($email: String!) {
  userCreate(
    email: $email,
    is_lead: true,
    source: "website-newsletter",
    notify_user_by_email: false
  ) { user_id }
}
```

Duplicate emails return `errors: "Internal server error"` from Optix — treated as soft success.

### 8.5 `app/api/cron/recalc-trial-days/route.ts` — daily trial sync

- Cron: `0 3 * * *` (Vercel `vercel.json`)
- Auth: `Authorization: Bearer $CRON_SECRET`
- `OPENING_DATE_ISO = "2027-01-01T00:00:00Z"` — **edit + redeploy when lease is signed**
- Calls `planTemplateCommit` for each founder plan with computed `free_trial_days`

### 8.6 Env vars (Vercel-managed)

```bash
OPTIX_ORG_TOKEN=<workhorse, ends in "o", server-only>
OPTIX_GRAPHQL_URL=https://api.optixapp.com/graphql
OPTIX_VENUE_ID=25863
OPTIX_PERSONAL_TOKEN=
OPTIX_CLIENT_ID=
OPTIX_CLIENT_SECRET=
NEXT_PUBLIC_POSTHOG_KEY=
NEXT_PUBLIC_POSTHOG_HOST=https://us.i.posthog.com
NEXT_PUBLIC_META_PIXEL_ID=
NEXT_PUBLIC_GA_ID=
NEXT_PUBLIC_SENTRY_DSN=
NEXT_PUBLIC_SENTRY_ENV=production
CRON_SECRET=<openssl rand -hex 32>
```

### 8.7 Run locally

```bash
cd /Users/kylehillman/golfsimulator/trajectory-golf-web
export NVM_DIR="/opt/homebrew/opt/nvm"; source "$NVM_DIR/nvm.sh"; nvm use 20
npm install
cp .env.local.example .env.local   # fill in keys
npm run dev                         # http://localhost:3000
npm run build && npm run lint       # before deploying
```

Site runs fully without keys (form returns a friendly "not connected" message; spot counter defaults to 0 claimed).

---

## 9. Risks to Carry Forward (Do Not Lose)

| Risk | Severity | Mitigation |
|---|---|---|
| **Rent overrun** (#1 historical killer in this category) | High | Walk away above ~$5,500/mo all-in; negotiate hard for TI |
| **Wrong location** | High | Site criteria + 10-min radius + 25K+ VPD + 11ft ceiling verified |
| **Insufficient demand** | High | Hard tripwire 25 paying by M12; 30+ in-person founder conversations before lease |
| **Member churn** (4–7%/mo industry; not in model yet) | High | League-driven retention; founder rate-lock; **Grok should add to next model refresh** |
| **24/7 unmanned incident** | Med-High | Kisi time-bound access; 2 cameras/bay; door-alarm sensors; live within 20 min of facility in Y1 |
| **Equipment downtime** | Med-High | Architecture B always-on PCs; scheduled 4am weekly reboot with alerts |
| **Personal guarantee exposure** | Med-High | Realistic $300–400K, not $75K Kyle initially considered. Accepted. |
| **Premature expansion** | Med | Already single-site; revisit only after 100+ members + 18 mo profitable |
| **App / software distraction** | Med | Optix until 50+ members; deferred custom build |
| **Owner-operator burnout** ("passive" is misleading; Y1 = 25–40 hr/wk not 8) | Med | Honest framing; weekly cleaning slot |
| **Seasonality** (TX summer) | Med | Cooled facility as differentiator; summer + evening leagues |
| **No durable moat** (Back Nine / X-Golf could enter Bastrop) | Med | First-mover 12–18 mo; 10-yr lease + brand + founder loyalty |
| **Cyber / member-data breach** | Low-Med | Optix + Stripe handle PCI; minimal PII on-prem |
| **TX Franchise Tax missed** | Low | Annual May 15 reminder; first due 2027-05-15 |

**Honest counterweights (don't drop these):**

- The IRR figures that swung 41% → 61% mid-modeling were partly methodology change. **Anchor expected return to 35–50% range.**
- The 20-founder presale bar is too low; live target is **30+ in-person conversations + 25 actual signups by M12**.

---

## 10. Handoff Instructions for Grok

### What Grok should do first (in order)

1. **Read this entire document.** Then skim `trajectory-golf-web/HANDBACK.md` (noting it's partially stale on Stripe/ConvertKit per §2.4 above) and `trajectory-golf-website-handover.md` for original design intent.
2. **Verify the live Optix integration is healthy** by curling the spot counter on the live site and confirming `getSpotsClaimed()` returns a number. If 0, that's expected pre-presale-launch.
3. **Help Kyle close the immediate-week task list (§6 items 1–6)** — sales tax permit, bank, lawyer email, $1 → $99 deposit bump test.
4. **Refresh the financial model v3** to include monthly churn (4–7%) which is currently absent.
5. **Prep the Chris Kirby consult agenda** — what to ask the attorney about the OA + Texas-specific liability waiver for unmanned 24/7 facility.
6. **Then move into:** broker outreach support, founder list construction, simulator enclosure decision (Full Swing Pro vs Foresight GCHawk reference calls), insurance quote collection.

### Hard rules

- **Do not** sign up for services on Kyle's behalf. Wait for credentials.
- **Do not** reintroduce a custom Stripe checkout for the deposit. Optix owns billing.
- **Do not** suggest reviving the archived Expo app unless Kyle asks.
- **Do not** suggest direct SpaceX-employee marketing. Kyle does not mix day job and business.
- **Do not** suggest a second location until 100+ members and 18+ months profitable.
- **Do not** push to file BOI/FinCEN. Domestic LLCs are exempt under the 2025-03-21 IFR.
- **Do not** pad responses with tutorials. Direct, numbers-focused, short. One feature per session.
- **Do** pause and confirm before paid signups, library/schema choices, or breaking changes. Pause and summarize after each meaningful chunk.
- **Do** push back on Kyle when his number or assumption looks wrong. He has explicitly thanked Claude for doing this.
- **Do** treat tables, scenario comparisons, and explicit break-even thresholds as the default communication mode for anything quantitative.

### Specific deferred decisions Grok inherits

- **Simulator enclosure:** Full Swing Pro vs Foresight GCHawk (TrackMan iO is the tracking layer regardless). Reference customer calls outstanding.
- **Custom Stripe live-mode webhook:** decide whether to revive deposit-tagging path or rely fully on Optix.
- **Standard-tier opening trigger:** at what exact founder count do we flip plans 58554/55/56 to `onboarding_enabled: true`? Default: when 50/50 founder spots filled.
- **White-label Optix app timing:** start LLC → D-U-N-S → Apple Org transfer process now, or wait until founder presale closes?

### Files Kyle will hand to Grok at session start

In priority order:

1. **This handover document**
2. `Trajectory_Golf_Model_v3.xlsx`
3. `Bastrop_GolfSim_Gantt_v2.xlsx`
4. `Trajectory_Golf_Stress_Test_v2.xlsx`
5. `Phase1_Execution_Playbook.docx`
6. `Trajectory_Golf_LLC_Site_Criteria.docx`
7. `Trajectory_Golf_LLC_Operating_Agreement.docx` (draft, for Kirby consult prep)
8. CP 575 EIN letter PDF (for bank + Stripe onboarding)

---

**Ready for Grok** — This document contains everything needed to pick up development without losing context.

---

## 11. Session Log (Living Memory)

> Append-only changelog of every meaningful Claude or Grok session. Newest entry on top. Each entry is short: date, AI, what changed, what's next. This is how the two assistants stay in sync between sessions. See `README.md` for the workflow.

### 2026-05-26 · Grok 4.3 · Switched visitor counter to cumulative total visitors (Plausible API key)

- User explicitly requested to stop showing live concurrent visitors ("X people viewing right now").
- Goal: show real total unique visitors since we started tracking ("how many people have visited since we started counting").
- Replaced public `current_visitors` endpoint with Plausible Stats `aggregate` API (requires read-only `PLAUSIBLE_API_KEY`).
- New component shows large amber total + "total visitors since May 2026". Falls back gracefully if no key is set.
- Updated `.env.local.example`, privacy page, and §2.2 analytics table.
- Code change committed to web repo (pending deploy). Brain updated.

**Open for next session:** Kyle to create a read-only Plausible API key (Stats scope, trajectory-golf.com only) and add it as `PLAUSIBLE_API_KEY` in Vercel. Then redeploy to make the real total visible.

### 2026-05-26 · Grok 4.3 · Visitor counter upgraded to graphic live readout

- Replaced tiny 0.65rem faint sentence (easy to miss at bottom of #updates) with compact instrument-panel style matching SpotCounter: pulsing amber dot + 0.7rem mono + accent number (tabular-nums) + short label.
- 0 visitors: "Thousands have visited this site"; >0: "X people viewing right now"
- mt-6 flex under newsletter form; now visibly graphic while staying passive/low-key. Build clean.
- Committed 434e6d7 + pushed (only visitors-counter.tsx) to kylehillman/trajectory-golf-web. Vercel auto-deploy in progress.
- **Open for next session:** §6 immediate (real-card founder test at $1, bump to $99 via planTemplateCommit, email Chris Kirby, file TX sales tax permit).

### 2026-05-26 · Grok 4.3 · Plausible script + public visitor counter live on trajectory-golf.com

- Script: exact `https://plausible.io/js/pa-QorOJS5kQlskmTcdfPiDg.js` + init boilerplate (`window.plausible` + `plausible.init()`) confirmed in live HTML (preload link + tags); local HEAD af53e1d matches origin/main
- Counter: `VisitorsCounter` (RSC + 60s revalidate from Plausible public API) renders at bottom of #updates "Stay in touch" section (updates-section.tsx:35); currently "Thousands of people have visited this site." (0 concurrent at verification fetch)
- "X people viewing right now · thousands have visited since we started tracking" when visitors > 0
- Vercel: non-www → www redirect, x-vercel-cache: MISS on curl; user confirmed Plausible dashboard "verify script installation" success
- Updated §2.2 analytics table + this §11 entry
- **Open for next session:** §6 immediate (real-card founder test at $1 safety-net → bump deposits $1→$99 via planTemplateCommit on 58551/52/53; email Chris Kirby for OA review; file TX Sales Tax Permit). Optional: wire Plausible API key for 30-day totals in counter?

### 2026-05-26 · Grok 4.3 · Optix founder plan guest pass allowance investigation + $0 product

- During the real-card founder signup safety-net test (deposits temporarily at $1), a $30 "Day Guest Pass" product sale (#374530 / invoice #00007) was auto-created when the user signed up for Apex Founder, even though the plan deposit itself correctly showed $1.
- Root cause: Guest pass allowances on the founder plans (e.g. 4/month on Apex) were configured to trigger a billable sale of the regular $30 Day Guest Pass product at initial enrollment.
- User explicitly requires: No guest pass charge at the time of founder deposit/signup.
- Created a new $0 "Founder Guest Pass" product.
- User was in the plan template editor (allowance section offers Resource / Product / Check in / All). Decision: Switch founder plan guest pass allowances from the paid product to the actual Guest Pass **resource** (or the new $0 product) so allowances are granted with zero charge.
- Real-card test partially completed (Apex signup succeeded; $30 was refunded on Stripe). Deposits remain at temporary $1 pending clean resolution.
- New local memory note: `memory/optix_founder_guest_passes.md`.

**Open for next session (Claude):** Finish updating the three founder plans (58551/52/53) so guest pass allowances no longer generate a paid product sale at signup. Verify no charge appears on a test (or the existing) founder signup. Once clean, bump the three founder plan deposits $1 → $99 via `planTemplateCommit`. Decide on long-term visibility/restrictions for the $0 Founder Guest Pass product.

### 2026-05-25 · Grok 4.3 · DRY BrandMark extraction on live website (nav + footer)

- Extracted duplicate compact glyph SVG (amber trajectory arc + fairway-green ring + ball) from nav.tsx:41 and footer.tsx:21 (~20 lines dupe) into shared <BrandMark size={n} className={...} /> in components/icons.tsx.
- Updated imports + usages in both files; refined adjacent comments for accuracy. `npm run build` clean (Next 16.2.6 + TS strict).
- Continues post-05-19 branding work (reverts had restored the CSS wordmark + glyph system; new minimalist assets still absent from repo).
- **Open for next session:** Real-card Optix founder test (gates safe presale) or continue website polish (proper X/SVG icon from Icons/, stale web/README + HANDBACK cleanup, or next branding iteration). Permit (§6.1) steps from prior still ready.

### 2026-05-25 · Claude (Opus 4.7, Cowork) · Hermes Agent live + Grok project provisioned + Mercury/broker status flips

- **Mercury bank application submitted** (in review). HANDOVER §2.3 flipped from "Pre-fill done; not submitted" → "Application submitted; in review."
- **Commercial RE tenant-rep broker engaged.** §2.3 flipped; site tours not yet started but the engagement closes the longest-lead unblock on facility search. §6 site/build task re-ranked to put tours first.
- **Stamped Certificate of Formation received** (PDF in `files/legal/Certificate of Formation .pdf`). LLC formally exists as of 2026-05-13 — §3.5 already reflected the filing #; no new edits needed there. EIN already in hand.
- **Hermes Agent (Nous Research v0.14.0) installed and operational** on Kyle's Mac.
  - Provider: xAI Grok OAuth (SuperGrok subscription), active model `grok-4.3`, OAuth tokens at `~/.hermes/auth.json` with auto-refresh
  - Gateway running as launchd service (`ai.hermes.gateway`, plist at `~/Library/LaunchAgents/`) — survives reboot
  - Discord bot "Trajectory Golf" live in Kyle's private "Trajectory Bot" server, `#hermes` channel set as home, `GATEWAY_ALLOW_ALL_USERS=true` (private server)
  - Self-identifies as `grok-4.3, built by xAI` — confirms full provider wiring
  - Logs at `~/.hermes/logs/gateway.log`
  - New §2.5 section added documenting the full setup
- **Grok web project provisioned** with comprehensive Custom Instructions block; live HANDOVER fetch from `raw.githubusercontent.com` confirmed working. The old §6 "blocked on Grok login issues" item is now resolved and removed from blockers.
- **§6 immediate-week re-ranked** to put the real-card Optix test first (it gates every founder conversation from converting safely), Kirby email second (2–3 week calendar lead), then sales tax permit and TESS trademark search.
- **New deferred items added** to §6: TESS trademark search → Class 41 filing; Apple Dev Org + D-U-N-S application (now unblocked by LLC formation); Hermes X tweeting path decision (Path A xurl vs Path B OpenTweet — pending Kyle's call on @KyleHillman1 vs new @TrajectoryGolf handle).
- **Open for next session:** (1) Real-card Optix founder signup test → bump $1 → $99; (2) Send Kirby email and book OA review consult; (3) File TX Sales Tax Permit; (4) Run TESS search on "Trajectory Golf" and decide on Class 41 filing; (5) Schedule first broker site tour; (6) Decide on Hermes X tweeting path. Note for next AI: Kyle can now reach Grok from anywhere via Discord DM to the "Trajectory Golf" bot — coordinate accordingly.

### 2026-05-19 · Grok 4.3 · Pushed branding/positioning updates to live site + prepared X assets

- Made balanced SpaceX engineering emphasis updates (hero eyebrow + deeper founder note per Kyle's "balanced" preference) and added X link in footer.
- Generated refined cleaner minimalist logo/icon variants + X header (using exact brand tokens and dual trajectory motif) via image_gen; assets saved to `Icons/branding-2026-05/` for immediate X use and future code integration.
- Verified `npm run build` clean, committed the three component files, and pushed to `kylehillman/trajectory-golf-web` (new commit eab27f9). Vercel auto-deploy now live.
- Kept changes evolutionary (no core palette/type/voice shift) to maximize presale conversion while amplifying the unique credible positioning.
- **Open for next session:** Upload assets + set bio ("24/7 unmanned... Built by a SpaceX engineer...") on existing @Trajectory_golf X profile; optional deeper SVG logo refresh in dynamic icon/OG generators; monitor Optix signups and X engagement from the updates.

### 2026-05-19 · Grok 4.3 · Set up local Grok project + full workflow in golfsimulator/ workspace

- Created `/Users/kylehillman/golfsimulator/GROK.md` (parallel to CLAUDE.md) with brain-pull-first ritual tailored for this full-access CLI tool: read_file/grep/search_replace for edits, run_terminal_command with nvm ritual, direct git push on brain (no grok-sync paste needed), memory/ navigation, and strict adherence to Kyle's short/tables/pause-for-confirmation style.
- Updated `memory/00_index.md` to document GROK.md and note that both AIs follow the identical HANDOVER-update discipline.
- Pushed the pending local commit (01ee929 "migrated to cowork + built local /memory") so `kylehillman/trajectory-golf-brain` remote is now current with the Cowork + memory system.
- Verified git push works cleanly from the agent shell. The project is now fully dual-AI operational: Claude/Cowork and this Grok CLI stay in lockstep via the public brain repo.
- **Open for next session:** §6 immediate-week items 1–6 (file TX Sales Tax Permit today, submit Mercury/Relay bank app, email Chris Kirby for OA review, real-card founder test through Optix, bump deposits $1 → $99, decide on live Stripe webhook). Also execute handoff §10 item 2: verify live Optix spot counter + API health. Then support broker outreach and founder target list construction.

### 2026-05-18 · Claude (Opus 4.7, Cowork) · Migrated to Cowork + built local /memory + scheduled email digest

- **Switched primary interface from Claude Code CLI to Cowork** (Anthropic desktop app). Folder mounted at `/Users/kylehillman/golfsimulator/`; brain-pull-first ritual unchanged. Installed Cowork plugins covering engineering, finance, small-business, sales, marketing, product-management, design, productivity, and data.
- **Built `/Users/kylehillman/golfsimulator/memory/`** — 5 navigation files (`00_index.md`, `repo_map.md`, `code_conventions.md`, `stale_docs.md`, `quick_commands.md`) so future sessions can find files and patterns without re-scanning the whole tree. Complements HANDOVER (doesn't duplicate it). Root `CLAUDE.md` updated to point at it.
- **Cataloged stale docs** in `memory/stale_docs.md`: `trajectory-golf-web/README.md` + `HANDBACK.md` still describe the dead ConvertKit/Stripe Payment Links flow; `files/legal/README.md` still says "thenineteenthhole.com" / "19th Hole Operations LLC"; `lib/optix.ts` JSDoc references a phantom `memory/project_nineteenth_hole_optix.md`. Drift-watch items: `lib/structured-data.ts` `geo` (Bastrop city center placeholder), `api/cron/recalc-trial-days/route.ts` `OPENING_DATE_ISO`, HANDOVER §2.2 line about $1 founder deposits.
- **Scheduled task in Cowork** — `trajectory-golf-email-digest` runs at 9am/12pm/3pm/5pm/7pm daily, reads `trajectory.golf.llc@gmail.com`, sends phone-friendly digest prioritized by Optix signups → emails from real people → time-sensitive items. Gmail connector connected but not verified against `trajectory.golf.llc@gmail.com` specifically — first run may pause for auth.
- **Open for next session:** §6 immediate-week items 1–6 unchanged (sales tax permit, bank submit, Kirby consult, real-card founder signup test, $1 → $99 deposit bump, live-mode Stripe webhook decision). Cleanup-when-touched: fix the stale README/HANDBACK in `trajectory-golf-web/` and the legal/README "19th Hole" references.

### 2026-05-17 · Claude (Opus 4.7) · Logged blocked Grok-instructions task

- Kyle hit Grok login issues before he could paste the Custom Instructions block into his Grok "Trajectory Golf" project. Logged as a blocked task at the top of §6 so it surfaces on the next session of either AI. The block itself lives in the Claude chat history from 2026-05-17 ("Grok Project Custom Instructions — paste this in").
- **Open for next session:** unblock Grok login → paste Custom Instructions → test that Grok auto-fetches HANDOVER.md at chat start.

### 2026-05-17 · Claude (Opus 4.7) · Added `bin/grok-sync` round-trip helper

- New shell helper at `bin/grok-sync` collapses post-Grok-session work into one command: `git pull --rebase` → save clipboard to `/tmp/grok-paste-*.md` → open `HANDOVER.md` + paste file in `$EDITOR` → on save, commit with auto-derived headline (`session: grok <headline>` from the new §11 entry) → `git push`. Aborts cleanly if `HANDOVER.md` unchanged.
- README updated with install + usage instructions (`alias grok-sync=…` one-liner for `~/.zshrc`).
- **Open for next session:** unchanged — §6 immediate-week items 1–6 (sales tax permit, bank submit, Kirby consult, real-card founder signup test, $1 → $99 deposit bump).

### 2026-05-17 · Claude (Opus 4.7) · Repo flipped public + project instructions added

- Changed visibility of `kylehillman/trajectory-golf-brain` from private to **public** so Grok can fetch `HANDOVER.md` directly via the raw URL. ⚠️ Kyle accepted the tradeoff — EIN, lease walk-away number ($5,500/mo), financial model, and personal-exposure numbers ($300–400K) are now Google-indexable. Sensitive items can be selectively redacted to `[private — ask Kyle]` in a follow-up commit if Kyle changes his mind.
- Updated `README.md` to reflect the public-fetch workflow.
- Wrote `/Users/kylehillman/golfsimulator/CLAUDE.md` so Claude Code auto-loads the brain-repo sync workflow at session start.
- Generated a Grok Project "Custom Instructions" paste-in block (in chat reply, not committed).
- **Open for next session:** §6 immediate-week items 1–6 — sales tax permit, bank submit, Kirby consult, real-card founder signup test, $1 → $99 deposit bump.

### 2026-05-17 · Claude (Opus 4.7) · Initialized shared brain repo

- Created private GitHub repo `kylehillman/trajectory-golf-brain` as the canonical living-memory store for Trajectory Golf.
- Seeded with this `HANDOVER.md` (v2.0, supersedes the earlier `Trajectory_Golf_Handover_to_Grok.md` v1.0).
- Added Session Log section + `README.md` documenting the Claude↔Grok sync workflow.
- **Open for next session:** §6 immediate-week items 1–6 — sales tax permit, bank app submit, Kirby consult, real-card founder signup test, $1 → $99 deposit bump.
