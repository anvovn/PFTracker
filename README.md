# PFTracker

An all-in-one personal finance app built for **freelancers and variable-income users** — a demographic most finance apps underserve. PFTracker connects checking, savings, investment, and credit card accounts in one place, then goes further than the typical dashboard-and-categorization app: it forecasts cash flow forward in time and turns insight into action.

Most finance apps optimize for engagement. PFTracker optimizes for **outcomes** — it tells you what's coming, recommends a specific next step, and tracks whether you actually took it.

## Why this app

We reviewed YNAB, Monarch Money, Rocket Money, Empower, Quicken Simplifi, and EveryDollar. None combine deep cash flow forecasting, proactive SMS alerts, a genuine freelancer/variable-income mode, and a real free tier in one product. That gap is what PFTracker is built to fill.

## Core features

1. **Cash flow forecasting** (the core moat) — three tiers of increasing depth:
   - Recurring transaction detection with a daily calendar view
   - Predictive alerts on a 7–10 day horizon
   - A "what-if" scenario engine for modeling new debt, extra debt payoff, income changes, or big purchases

2. **Gamification** — streaks, badges, XP, quests, and anonymous leaderboards, all tied to real financial outcomes rather than vanity points.

3. **Proactive SMS alerts** — six alert types delivered via Twilio, designed to fire *before* a problem happens (e.g. low balance ahead of a due date), not after.

4. **Variable income / freelancer mode** — flexible budgets set as a percentage of income, automatic tax set-aside, deduction tracking, lean-month planning, and income averaging.

5. **Multi-user / family accounts** — tiered permissions (view-only, collaborator, owner) to support couples, parents and teens, roommates, or an adult child helping an aging parent.

6. **Actionable AI** — closes the loop by naming a specific action, executing it in one tap, and tracking whether the user followed through.

7. **Credit score improvement roadmap** — personalized recommendations tied to the user's actual connected accounts (e.g. "pay down this card by $X for roughly Y points").

8. **Transparent flat pricing** — a genuinely useful free tier (aggregation, basic categories, net worth, credit score) drives acquisition; forecasting, what-if scenarios, SMS, freelancer mode, and the credit roadmap are paid differentiators. Target pricing: Pro ~$6–7/mo, Family ~$10–12/mo for up to 6 users.

## How it's built

- **Web:** Next.js (App Router) serves both the UI and the API — `apps/web/app/api/` route handlers are the backend, with business logic living in `apps/web/server/` (Plaid sync, forecasting engine, Twilio dispatch). There's no separate Express server.
- **Database:** PostgreSQL via Drizzle ORM.
- **Account aggregation:** Plaid (link token exchange, cursor-based transaction sync, webhooks).
- **Alerts:** Twilio SMS.
- **Auth:** JWT + bcrypt.
- **Mobile:** React Native via Expo, sharing logic (not UI) with web through shared packages — each platform keeps its own component library.
- **Monorepo:** pnpm workspaces (`apps/*`, `packages/*`). Turborepo was evaluated and deliberately left out for now — not needed with just two apps and no build-caching pain yet.
- **Deployment:** Vercel for web/API, with Vercel Cron running the daily forecast recompute and SMS alert dispatch. Mobile ships via EAS to the App Store and Google Play.

Shared packages: `@finance/types` (shared interfaces), `@finance/utils` (cash flow math, debt payoff, credit score estimation, freelancer tax calcs), `@finance/api` (typed fetch client), `@finance/hooks` (TanStack Query hooks used by both apps), `@finance/store` (Zustand global state).

## Project status

The monorepo skeleton exists — all directories and empty files per the structure below — but implementation hasn't started yet. Root and package `package.json` files still need `name` fields before `pnpm install` will link the workspace.

Build order:

1. Fill in root `package.json`, `pnpm-workspace.yaml`, and package `package.json` files with `name` fields.
2. Set up the Drizzle schema (accounts, transactions, credit liabilities, investment holdings, webhook events).
3. Implement the Plaid Link flow end-to-end (link-token → client Link → exchange → webhook).
4. Build the cash flow forecasting engine, Tier 1 first (recurring detection + calendar view).
5. Wire up `@finance/hooks` + `@finance/api` so web and mobile call the same endpoints identically.

## Repo structure

```
pftrackerapp/
├── apps/
│   ├── web/                       # Next.js — UI + API
│   │   ├── app/
│   │   │   ├── (marketing)/       # public pages, SSG
│   │   │   ├── (auth)/            # login, register
│   │   │   ├── (dashboard)/       # authenticated app
│   │   │   └── api/               # route handlers = the backend
│   │   │       ├── auth/
│   │   │       ├── plaid/
│   │   │       ├── accounts/
│   │   │       ├── transactions/
│   │   │       ├── cashflow/
│   │   │       ├── debt/
│   │   │       ├── credit/
│   │   │       ├── alerts/
│   │   │       └── cron/
│   │   ├── server/
│   │   │   ├── db/                # Drizzle schema, connection, migrations
│   │   │   └── services/          # plaid.ts, twilio.ts, forecast.ts, alerts.ts, auth.ts
│   │   └── components/
│   └── mobile/                    # Expo React Native
│       └── src/
│           ├── navigation/
│           ├── screens/
│           ├── components/
│           └── lib/
└── packages/
    ├── types/
    ├── utils/
    ├── api/
    ├── hooks/
    └── store/
```

See `pftrackerapp/CLAUDE.md` for the full architecture rationale, decision log, and current priorities — it's the source of truth for this project and should be kept in sync with any real decisions made during implementation.
