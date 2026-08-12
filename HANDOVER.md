# Golf Simulator Business Project — Full Handoff for Grok

**Doc version:** 2.0
**Prepared:** 2026-05-17
**Owner:** Kyle R. Hillman (50%) + Alexis Kalletta (50%)
**Business:** Trajectory Golf LLC — 24/7 unmanned indoor golf simulator club, Bastrop, TX
**Target opening:** Q1 2027

> This document supersedes the prior `Trajectory_Golf_Handover_to_Grok.md` (May 17 v1.0) where they conflict. Notably: Optix is now the live member backbone, and the Stripe Payment Link / ConvertKit flow described in `trajectory-golf-web/HANDBACK.md` is no longer the live signup path.

---

---

## 0. Current truth (2026-08-12) — supersedes older sections where they conflict

> Personal financing line size, rate, and collateral values are **intentionally omitted** from this public doc. Ask Kyle or Chief of Staff privately.

| Item | Current |
|---|---|
| Configuration | **2 launch bays, ~1,700 SF** (was 3 / ~2,400–2,500) |
| Go site | **1065 Lovers Lane, Texas State Business Park, Bastrop** (flex/industrial) |
| Rejected | The Colony retail (FM 969) — Del Webb free-with-HOA sim + cost + 24/7 friction |
| Shell / open | Shell **Q4 2026** · target open **Q1 2027** |
| Landlord | David Alexander, NEWCOR — david@newcorcre.com |
| Tenant rep | Nathan K. Smith, Austin Tenant Advisors — nathan@austintenantadvisors.com |
| Base rent | Ask $20/SF · target $18/SF |
| NNN | **$3.30/SF** ($0.27/SF/mo) confirmed |
| TI | $25/SF **verbal only** — must be in LOI |
| Modeled all-in | ~$26.45/SF (~$3,747/mo) — small-suite $/SF premium **unverified** (if $26 base → BE ~35) |
| Hard CAPEX | **~$145K** (was ~$242K / older ~$291K) |
| Peak business draw | **~$84K** (was ~$262K) |
| Breakeven | **~30 members** (~25% fill); headroom ~2.5–3.0× |
| 10-yr FCF (2-bay) | **~$578K** standalone; ~$1,039K with bay 3 + site 2 |
| Financing | Morgan Stanley securities-based LOC **approved**; drawable ~**Dec 2026** after employee equity lockup. Details private. Funds: MS → Kyle personal → LLC **member loan** (Kirby) → business. Draw in tranches. |
| Presale gate | **20 paid $99 deposits within 60 days** of address announce; walk if under 10; founder discount 25 spots / 15% / 12-mo |
| Sim mix | TrackMan as **flagship** bay; second **value** unit under consideration (not all-TrackMan) |
| Critical path | Column grid + clear height from David → dual quote 1,700 + 2,400 SF → HVAC quotes → OA + member note (Kirby) → LOI (hard expansion option) → announce → deposits → draw |

**Non-negotiable LOI terms:** hard expansion option on adjacent suite · rent abatement 6 mo (floor 4) from **shell delivery** · TI $25/SF in writing · capped PG with burn-down · CAM capped · renewal caps · assignment rights · explicit 24/7 unmanned use.


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

**Trajectory Golf LLC** is a 24/7 unstaffed indoor golf simulator membership club opening Q1 2027 in Bastrop, TX. Two premium launch bays at entry (~1,700 SF), with hard expansion option for a third. Members sign up, pay, book, and unlock the door entirely self-service — no on-site staff ever.

**Founder:** Kyle is a SpaceX Starship launch engineer in Bastrop. The brand intentionally borrows aerospace language (*Trajectory · Apex · Approach*) without being a theme park. Cofounder is Kyle's wife Alexis Kalletta (kept maiden name — use **Kalletta** on legal/banking).

**Key model decisions (locked):**

- **2 bays** at entry (~1,700 SF), single site; hard expansion option for bay 3; site: 1065 Lovers Lane
- 3 tier pricing: Approach / Trajectory / Apex
- Founder program: 25 discounted spots (15% lifetime, 12-mo commit); 50 public spots; $99 refundable deposit; presale gate 20/60 days
- Independent (rejected Back Nine / X-Golf franchise)
- **No alcohol** → single LLC structure, no TABC, no on-site staff requirement
- Morgan Stanley securities-based LOC approved (details private); drawable ~Dec 2026 post-lockup; not SBA for site 1
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
- Founder deposits set to **$99** with `initial_invoice_due: PLAN_PURCHASE` so Optix auto-charges the card-on-file immediately at signup (recurring monthly still deferred to trial-end via `start_billing_on: FOLLOWING_BILLING_DATE` + `free_trial_days`). End-to-end $1 test verified clean 2026-05-25 (Invoice #15 paid via Stripe); final $99 sanity test recommended before sending founder conversations the signup link.
- Founder guest-pass allowance: AccessTemplate `35048` ("Guest Pass") is bound to product `53471` ("Founder Guest Pass", $0) instead of the paid Guest Pass product (`53149`, $25). This prevents the prior bug where signing up auto-charged the paid guest-pass product at enrollment. **Optix API gotcha:** `access_template_id` on an existing Access row is immutable — `planTemplateCommit` accepts the field in input but silently ignores it. To change what an allowance binds to, mutate the template's `rules` via `accessTemplateCommit`, not the plan's accesses array.

### 2.3 What's NOT yet done

| Area | Status |
|---|---|
| Site / lease | Commercial RE broker **active** — Nathan K Smith (Austin Tenant Advisors); Aug 7 tour done; location pricing Excel pending from Nathan; no lease signed |
| Simulator enclosure decision | Full Swing Pro vs Foresight GCHawk — reference calls outstanding |
| Insurance | Coverage stack defined; quotes not yet collected |
| TX Sales Tax Permit | Not yet filed (Comptroller Webfile, free, ~30 min) |
| Mercury bank account | **Live** (IO card shipped 2026-06); LAL vs ~$1MM SpaceX collateral expected imminently (2026-08-12) |
| Operating Agreement | Draft exists; needs Chris W. Kirby (Bastrop Law Group) attorney review — email not yet sent |
| Insurance quotes | Stack defined ($2M GL + $1M cyber + $2M umbrella + property + WC); not collected |
| PGA pro recruitment | Planned: ~14 lessons/week, 30% facility share, 1099 |
| Founder presale outreach | Conversations started ad-hoc; formal 100-name target list not yet built |
| Trademark "Trajectory Golf" | TESS search not yet run; Class 41 filing not started |
| Kisi hardware purchase | Pending lease |
| Optix Kisi integration / `member_booking_created` webhook | Optix app manifest currently `{}` |
| Day Pass plan re-enable | Gated on facility opening |
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
- ✅ **Founder signup flow fixed end-to-end** — guest-pass allowance rebound to $0 product; deposit-timing bug fixed (`initial_invoice_due: PLAN_PURCHASE`); deposits bumped $1 → $99; $1 test signup verified clean ($1 charged on Stripe, Invoice #15 paid in Optix, no spurious guest-pass charge)

### Immediate this week (re-ranked 2026-05-25)

1. **Final $99 sanity test** — one real-card signup at the live $99 deposit before sending founder conversations the signup link. Last verification step before presale goes hot.
2. **Email Chris W. Kirby** at Bastrop Law Group (512-240-9565) to book a 1-hour OA review consult; send draft OA ahead of time. 2–3 week calendar lead time → start now.
3. File **Texas Sales Tax Permit** via Comptroller Webfile (free, ~30 min)
4. **TESS trademark search** on "Trajectory Golf" → file Class 41 (~$350 DIY) before signage / merch / lease commitment
5. Open Stripe **live-mode webhook** endpoint (if reviving the deposit-confirmation tagging path) — current webhook secret is test mode

### Founding member campaign (M0–M3)

6. Formalize the 100-name Bastrop-golfer target list (Kyle has started ad-hoc conversations — convert to a tracked pipeline: name, source, stage, last touch, next action)
7. Begin 30 in-person founding-member conversations
8. Capture 50 founders at $99 refundable deposit
9. Weekly founder construction updates (auto via Optix lead emails)
10. Get permission to use 2–3 supporter quotes on the site for social proof

### Site / build (M0–M6)

11. **Tour 3–5 candidate spaces with broker** using Site Criteria one-pager; verify **11 ft ceiling height** at hitting position on each (kills more spaces than anything else)
12. Lease negotiation (DO NOT SIGN before SBLOC is locked — financing gated on SpaceX IPO)
13. TrackMan iO pre-order at lease signing
14. Finalize enclosure: Full Swing Pro vs Foresight GCHawk (reference customer calls — 18-week lead from order)
15. Collect 3 insurance quotes against defined stack (quotes don't bind; gather now)

### Tech (deferred until opening date is firmer)

16. Configure Optix `member_booking_created` / `member_booking_cancelled` webhooks → Kisi door provisioning
17. Purchase Kisi hardware + readers (7 doors)
18. Re-enable Day Pass plan 58557 (`onboarding_enabled: true`)
19. Open standard plans 58554/55/56 once 50-founder cap fills
20. **Apple Developer Org account application + D-U-N-S** (LLC now formed → unlocks this; ~6–10 week wait, start now to parallel the founder presale)
21. Update `OPENING_DATE_ISO` in `app/api/cron/recalc-trial-days/route.ts` when lease signed
22. **Hermes Agent — decide X tweeting path** (Path A: xurl + free X dev portal, 500 posts/mo; Path B: OpenTweet $5.99/mo). Account choice: @KyleHillman1 vs new @TrajectoryGolf handle.

### Operations (M-3 to opening)

23. **PGA pro candidate scouting** (Austin + Bastrop teaching pros — soft outreach to 5–10 candidates now; hire closer to opening)
24. Corporate outreach to Bastrop employers (NOT direct SpaceX)
25. League L1 ready for M+3 post-open; L2 M+6; L3 M+9

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

### 2026-08-12 · Chief of Staff (Grok Bot) · Consulting feasibility pack — CONDITIONAL GO

- Delivered consulting feasibility pack: **CONDITIONAL GO** on 2-bay Lovers Lane owner-op path.
- Underwrite blended membership ARPU **~$265** (Club ~$249–259 / Apex ~$329 / Hour Bank replacing Approach unlimited); **NO-GO** on $169-led stack.
- Structural: **$249 credited reservation** preferred over $99; gate **25 qualified OR 20@≥$249 / 60d** (walk <10); LOI economics before address announce; hybrid 90-day soft-open; **Optix+Kisi+HA only**; fix live 3-bay site copy before announce.
- Capital: net CAPEX **~$150.6K**, peak business draw **~$84–95K**; no lease until drawable financing + Kirby member-loan docs.

**Open for next session:** (1) Nathan Excel follow-up drafted not sent; (2) column grid + dual SF quote; (3) Kirby restart; (4) price cards + site copy.

### 2026-08-12 · Chief of Staff (Grok Bot) · Redacted public sync from Claude Aug 12 handoff

- Added **§0 Current truth** banner: 2-bay / ~1,700 SF, go site **1065 Lovers Lane**, Colony rejected, CAPEX/BE/FCF/presale/LOI path updated.
- **Personal financing numbers redacted** (line size, rate, collateral) — private to Kyle / CoS memory only. Public note: MS SBLOC approved, drawable ~Dec 2026, member-loan fund flow via Kirby.
- Soft-patched §1 locked-model bullets where they still said 3-bay / old SBLOC framing.
- Open conflicts still need Kyle call: deposit stack (Optix vs Stripe/ConvertKit) and site hosting (Next/Vercel vs Squarespace) — not resolved in this commit.

**Open for next session:** (1) Column grid + dual SF quote from David; (2) clarify live deposit + hosting stack; (3) OA/member note with Kirby after capital access path clear; (4) Nathan Excel chase if still outstanding.

### 2026-08-12 · Chief of Staff (Grok Bot) · Catch-up from Kyle + email RE/legal/finance audit

- **Financing:** LAL loan against ~$1MM SpaceX collateral in process; expected to fund in the next few days (supersedes older SBLOC-only framing for near-term cash). Mercury account already live.
- **Legal:** Chris Kirby / Bastrop Law Group ready to engage on OA once funded. $200 consult paid 2026-06-01. Engagement Adobe Sign was canceled (expired link); Lori will resend when ready. Open $5,600 trust-account payment request (2026-06-02) still unread in inbox.
- **RE:** Nathan K Smith (Austin Tenant Advisors) is active tenant rep. Tour Aug 7 with Alexis. Nathan owes Excel of pricing/terms for locations discussed by phone this week — **not in email yet**. Prior email pricing: 1065 Lovers Ln ($25 retail / $20 flex, NNN ~$0.28/sf/mo); Newcor Texas State Business Park Flex ~$20/sf + OPEX ~$0.27/sf (David Alexander). Budget hard constraint under $25/SF.
- Cannot ingest grok.com / Claude project chat histories from here — email + HANDOVER + Kyle verbal are the catch-up sources.

**Open for next session:** (1) Chase Nathan for location Excel once Kyle OKs; (2) on LAL funding → fund Kirby trust + restart engagement docs; (3) reconcile model to LAL terms; (4) site shortlist scorecard vs 11ft ceiling / rent walk-away.

### 2026-08-12 · Chief of Staff (Grok Bot) · Stood up multi-agent operating team

- Created Grok Bot agent team under Chief of Staff for Trajectory Golf startup → multi-site automation.
- Lanes live now: **Launch Ops**, **Founder Growth**, **Product & Automation**, **Finance & Legal**. CoS owns orchestration, morning digest, and brain commits.
- Operating model: specialists pull `HANDOVER.md` first; Optix remains billing source of truth; no SpaceX-employee marketing; pause on paid/breaking changes; sequencing entity → presale → lease → equipment → buildout → open → clone.
- Phase B deferred: Facility Build, Site Ops Autopilot, Multi-Site Expansion.
- Noted ~2.5 months of brain quiet since May 2026 sessions — status reconciliation still needed before hard execution.

**Open for next session:** (1) Kyle status check — what moved since May; (2) assign first wave of §6 work to Launch Ops + Finance & Legal; (3) Founder Growth pipeline scaffold; (4) Product & Automation health check on live site/Optix.

### 2026-05-25 · Claude (Opus 4.7, Cowork) · Founder signup flow fixed end-to-end (guest-pass allowance + deposit timing + $99 bump)

Three bugs found and fixed in one session. Founder presale is now operational at the real $99 deposit.

**1) Guest-pass allowance bug (continued from Grok's 2026-05-26 session).** All three founder plans (58551/52/53) shared `AccessTemplate 35048` ("Guest Pass") for their guest-pass allowance. That template was bound to the paid Guest Pass product (`53149`, $25, formerly $30), so every founder signup auto-charged the paid product at enrollment. Tried Option A first — created a new AccessTemplate (`35196`) and tried to swap each plan's guest-pass `access_template_id` via `planTemplateCommit`. **Mutation returned no errors but the swap silently didn't take.** Confirmed by re-reading the plans: access rows still pointed at 35048. Conclusion: **`access_template_id` is immutable on existing Access rows in Optix's API** — `planTemplateCommit` accepts it in input but ignores it. Pivoted to Option B: edit template 35048's `rules.product_id` from `[53149]` → `[53471]` (the new $0 "Founder Guest Pass" product Grok set up). One `accessTemplateCommit` mutation. Plans inherit automatically because they still reference 35048. Verified by reading `rules.product_id`. Orphan template 35196 deleted as cleanup. New API gotcha captured in §2.2.

**2) Deposit timing bug (NEW DISCOVERY — silently broken since plan creation).** Test signup showed no $30 charge (guest-pass fix worked) but also no $1 deposit charge. Investigated and found: all three founder plans had `initial_invoice_due: FOLLOWING_BILLING_DATE`, which means the deposit invoice is generated on the *next* billing cycle. Combined with `free_trial_days: 221`, the deposit was scheduled for ~Dec 28, 2026 — trial-end. Worse: looking at older CANCELED test signups (215200/202/203) from before the trial was wired in, even those had `initial_invoice_due_timestamp` set ~1 week after signup, not at signup. **The "$99 refundable deposit at enrollment" mechanic has never actually worked.** Fix: flipped `initial_invoice_due` from `FOLLOWING_BILLING_DATE` → `PLAN_PURCHASE` on all 3 plans via single multi-aliased `planTemplateCommit`. Verified by fresh test signup — $1 hit Stripe immediately, Invoice #15 paid in Optix.

**3) Deposit bumped $1 → $99.** Per §6 task, after the end-to-end $1 test verified clean. Single multi-aliased `planTemplateCommit`. All 3 plans now at deposit: 99 with initial_invoice_due: PLAN_PURCHASE.

**§2.2 updated** with new state + the API gotcha. **§2.3 + §6 cleaned** of completed items ($1→$99 bump done; real-card test ✅ at $1). **§6 immediate-this-week re-ranked** — only outstanding founder-flow item is the recommended final $99 sanity test.

**Local memory updated** at `memory/optix_founder_guest_passes.md` with the full resolved state including the API gotcha.

**Scripts left in `trajectory-golf-web/scripts/`** (all idempotent, read-only or atomic; safe to re-run):
- `optix-discover-allowances{,-2,-3,-4}.sh` — schema introspection (kept for future debugging)
- `optix-fix-founder-guest-passes.sh` — v1, deprecated (the silently-failing attempt; kept as documentation of the API gotcha)
- `optix-fix-founder-guest-passes-v2.sh` — Option B, the working fix
- `optix-diagnose-deposit.sh` + `optix-check-alexis-plan.sh` — deposit-timing diagnosis
- `optix-fix-deposit-timing.sh` — the initial_invoice_due flip
- `optix-bump-deposits-to-99.sh` — $1 → $99 bump
- `optix-delete-orphan-template.sh` — orphan template 35196 cleanup

**Open for next session:** (1) Final $99 sanity test (real card, fresh email) before sending founder conversations the signup link — strongly recommended even though $1 verified clean end-to-end; (2) cancel any IN_TRIAL test signups still hanging around (Alexis's Approach 216265, the Apex test 216256) since their billing timestamps are baked in pre-fix; (3) Kirby email + TX Sales Tax Permit + TESS trademark search per §6 immediate-this-week.

### 2026-05-26 · Grok 4.3 · Ended session (social graphics + counter fix)

- Added prominent graphic X and Instagram buttons (with custom icons) in the Updates / "Stay in touch" section.
- Plausible cumulative visitors counter now fully working in production (fixed `date` parameter issue).
- All web changes committed and pushed.
- Brain updated and pushed.
- Session closed at user's request.

**Open for next session:** Continue with §6 priorities (real-card Optix founder test at $1, bump to $99, Kirby email, TX sales tax permit, etc.). Social links are now live and prominent.

### 2026-05-26 · Grok 4.3 · Made X and Instagram more prominent with graphics

- Added graphic social buttons (with custom XIcon + InstagramIcon) in the UpdatesSection ("Stay in touch") — much more visible than just footer text links.
- New icons follow the same stroke style as other brand icons (currentColor, 1.75 weight).
- Cleaned up previous debug code from visitor counter in same pass.
- Web changes committed as 7c8468e and pushed. Vercel deploying now.
- Build clean.

### 2026-05-26 · Grok 4.3 · Added X (@Trajectory_Golf) and Instagram (@trajectory.golf) links + finalized visitor counter

- Added Instagram link in footer (https://www.instagram.com/trajectory.golf/) next to existing X link.
- Updated X link to user's preferred handle casing (Trajectory_Golf).
- Removed temporary debug code from visitors-counter.tsx now that the Plausible total visitors counter is working in production.
- Build verified clean.

### 2026-05-26 · Grok 4.3 · Switched visitor counter to cumulative total visitors (Plausible API key)

- User explicitly requested to stop showing live concurrent visitors ("X people viewing right now").
- Goal: show real total unique visitors since we started tracking ("how many people have visited since we started counting").
- Replaced public `current_visitors` endpoint with Plausible Stats `aggregate` API (requires read-only `PLAUSIBLE_API_KEY`).
- New component shows large amber total + "total visitors since May 2026". Falls back gracefully if no key is set.
- Updated `.env.local.example`, privacy page, and §2.2 analytics table.
- Web changes committed as 3c6bb50 + pushed to kylehillman/trajectory-golf-web (only the counter + docs files).
- Brain updated.

**Open for next session:** Kyle to create a read-only Plausible API key (Stats scope, trajectory-golf.com only) and add it as `PLAUSIBLE_API_KEY` in Vercel → redeploy. Counter will then show real cumulative total.

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
