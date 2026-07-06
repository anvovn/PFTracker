# CLAUDE.md

This file is the shared source of truth for this project. Both Claude Cowork and Claude Code should read this before doing any work. Update it whenever a real decision changes — architecture, scope, priorities, or tech stack.

Last updated: 2026-07-06

---

## Project purpose

An all-in-one personal finance app targeting **freelancers and variable-income users** as the primary underserved demographic. Connects checking, savings, investment accounts, and credit cards in one place.

The core bet: most finance apps optimize for engagement (dashboards, categorization). This app optimizes for **outcomes** — forward-looking cash flow forecasting and actionable AI that recommends specific steps and tracks whether the user took them.

## Competitive landscape (researched)

Reviewed: YNAB, Monarch Money, Rocket Money, Empower, Quicken Simplifi, EveryDollar. None of them combine deep cash flow forecasting + proactive SMS alerts + a genuine freelancer/variable-income mode + a real free tier. That combination is the gap this app is built to fill.

## The 8 differentiation angles (prioritized)

1. **Cash flow forecasting** (top moat) — 3 tiers: (1) recurring detection + daily calendar view, (2) predictive alerts on a 7-10 day horizon, (3) "what-if" scenario engine (new debt, extra payoff, income change, big purchase)
2. **Gamification** — white-hat mechanics (streaks, badges, XP, quests, anonymous leaderboards) tied to real outcomes, not just points
3. **SMS / proactive alerts** — six alert types via Twilio, fires before problems happen, not after
4. **Variable income / freelancer mode** — flex budgets as % of income, auto tax set-aside, deduction tracking, lean-month planning, income averaging
5. **Multi-user / family accounts** — tiered permissions (view-only, collaborator, owner) for couples, parent/teen, roommates, adult-child/aging-parent
6. **Actionable AI** — closes the loop: names a specific action, executes it in one tap, tracks whether the user followed through
7. **Credit score improvement roadmap** — personalized, tied to the user's actual connected accounts (e.g. "pay down this card by $X for ~Y points")
8. **Transparent flat pricing with a real free tier** — free tier genuinely useful (not crippled); Pro ~$6-7/mo; Family ~$10-12/mo for up to 6 users

## Tech stack

**Full-stack framework:** Next.js (App Router) for web — serves both the UI and the API. No standalone Express backend; `apps/web/app/api/` route handlers are the backend.

**Database:** PostgreSQL, accessed via Drizzle ORM (`drizzle-kit` for migrations).

**External services:**
- Plaid — account aggregation, link token exchange, cursor-based transaction sync, webhooks
- Twilio — SMS for proactive alerts

**Auth:** JWT + bcrypt.

**Mobile:** React Native via Expo. Separate app, shares logic (not UI) with web through shared packages.

**Monorepo:** pnpm workspaces (`pnpm-workspace.yaml` listing `apps/*` and `packages/*`). **Turborepo was evaluated and deliberately removed** — not needed at this stage with only two apps and no build-caching pain yet. Revisit if/when builds get slow or a team joins.

**Shared packages** (`packages/`):
- `@finance/types` — shared TypeScript interfaces (Account, Transaction, Forecast, Debt, CreditScore, etc.)
- `@finance/utils` — pure functions, no React, no platform deps (cash flow calcs, debt payoff math, credit score estimation, currency/date helpers, freelancer tax calcs)
- `@finance/api` — typed fetch client wrapping all `/api/*` routes
- `@finance/hooks` — TanStack Query hooks used identically by both apps (`useCashFlow`, `useDebtPlan`, `useCreditRoadmap`, `useAuth`, etc.)
- `@finance/store` — Zustand global state (auth, UI state)

**Web-only:** Tailwind CSS, Recharts, react-plaid-link, `middleware.ts` for route protection.

**Mobile-only:** React Navigation, react-native-plaid-link-sdk, Victory Native (charts), expo-secure-store, expo-local-authentication, Expo Notifications.

**Deployment:** Vercel (web + API + Vercel Cron for scheduled jobs — daily forecast recompute, SMS alert dispatch). Mobile via EAS to the App Store and Google Play.

## Key architecture decisions and why

- **Next.js over Vite**: wanted SSR/SSG for marketing pages (SEO) and a single deployable app instead of a separate API server. Trade-off: dashboard pages need `"use client"` since they use hooks/state — accepted, since this is an authenticated app, not a content site.
- **No Express**: folding the backend into Next.js route handlers removes a whole app from the monorepo. `server/` inside `apps/web/` holds the actual business logic (Plaid sync, forecasting engine, Twilio dispatch); route handlers stay thin and import from there.
- **No Turborepo yet**: pnpm workspaces alone provide the actual monorepo linking (`@finance/*` packages resolve via `pnpm-workspace.yaml`). Turborepo only adds task orchestration + caching, which isn't needed with 2 apps and no expensive build step yet.
- **UI is NOT shared between mobile and web.** Only logic is shared (hooks, types, utils, API client, state). Each platform gets its own component library — attempts to force-share UI (e.g. React Native Web) add complexity that isn't worth it pre-launch.
- **Free tier is strategic, not charity**: real free tier (aggregation, basic categories, net worth, credit score) is meant to drive acquisition; forecasting/what-if/SMS/freelancer mode/credit roadmap are the paid differentiators.

## Current state

- Backend logic (Plaid integration, DB schema design, alert types, dashboard endpoints) has been scoped in detail.
- Full monorepo file skeleton has been generated — all directories and empty files exist per the structure below, but **no code has been written into them yet.**
- Root config files (`package.json`, `pnpm-workspace.yaml`, individual package `package.json` files) are also currently empty and need `name` fields at minimum before `pnpm install` will link packages correctly.

## Directory structure (current skeleton)

```
financeapp/
├── package.json                  # root — needs name + scripts
├── pnpm-workspace.yaml            # needs packages: [apps/*, packages/*]
├── tsconfig.base.json
├── .env.example                   # DATABASE_URL, PLAID_*, TWILIO_*, JWT_SECRET
├── apps/
│   ├── web/                       # Next.js — UI + API
│   │   ├── app/
│   │   │   ├── (marketing)/       # public pages, SSG
│   │   │   ├── (auth)/            # login, register
│   │   │   ├── (dashboard)/       # authenticated app, client components
│   │   │   └── api/               # route handlers = the backend
│   │   │       ├── auth/
│   │   │       ├── plaid/         # link-token, exchange, webhook
│   │   │       ├── accounts/
│   │   │       ├── transactions/
│   │   │       ├── cashflow/      # forecast, scenario, recurring
│   │   │       ├── debt/          # payoff-plan
│   │   │       ├── credit/        # score, roadmap
│   │   │       ├── alerts/
│   │   │       └── cron/          # daily-forecast, send-alerts (Vercel Cron)
│   │   ├── server/
│   │   │   ├── db/                # Drizzle schema, connection, migrations
│   │   │   └── services/          # plaid.ts, twilio.ts, forecast.ts, alerts.ts, auth.ts
│   │   └── components/
│   └── mobile/                    # Expo React Native
│       └── src/
│           ├── navigation/
│           ├── screens/
│           ├── components/
│           └── lib/                # notifications, biometrics, storage
└── packages/
    ├── types/
    ├── utils/
    ├── api/
    ├── hooks/
    └── store/
```

## Next steps (in order)

1. Fill in root `package.json`, `pnpm-workspace.yaml`, and every package `package.json` with `name` fields so `pnpm install` links the workspace.
2. Set up Drizzle schema in `apps/web/server/db/schema.ts` (accounts, transactions, credit liabilities, investment holdings, webhook events — already designed).
3. Implement Plaid Link flow end-to-end: `link-token` route → client Plaid Link → `exchange` route → webhook handler.
4. Build the cash flow forecasting engine (Tier 1: recurring detection + calendar view) as the first real differentiator feature.
5. Wire up the shared `@finance/hooks` + `@finance/api` layer so both web and mobile can call the same endpoints identically.

## Working preferences

- Prefers short, efficient prompts; expects full context carried forward without re-explanation.
- Prefers complete, production-ready code over conceptual walkthroughs.
- In active build mode — sessions are execution-focused.
- Cowork is used for planning/research/specs/docs. Claude Code is used for implementation, debugging, and git. Decisions get written back into this file, not left in chat history.
