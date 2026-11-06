# Jane: an AI financial controller that runs your money on autopilot

**The goal.** Jane looks through your bank accounts securely through Plaid and builds a
picture of who you are, such as a college student, a dog owner or a mother of three. Then it
goes through every area of your money:

- subscriptions and bills
- bank fees and idle cash
- cards, loans and debt
- insurance
- taxes
- investing and retirement
- goals and plans

In each area it finds the best plan for you. It then prefills the steps through connectors, so
all you do is tap yes or no.

**Where it is now.** Jane is in a closed, read-only beta at
[jane-dusky.vercel.app](https://jane-dusky.vercel.app). To try it without an account, upload a
bank statement on the landing page. It's analyzed as a temporary visitor and deleted within 24
hours.

> This repository is an overview. Jane's source code is private. Code access is available on
> request for interviews and diligence.

## What the MVP does

- **Reads your money.** Bank accounts link through Plaid, read-only, or you can upload a CSV
  or OFX export.
- **Builds a picture of you.** Jane works out who you are from your money: income and paydays,
  household, pets, vehicles, housing, school and life events. That picture shapes every plan.
- **Finds the best plan in every area.** A detection engine and detectors for each area find
  what each part of your money is costing you and what would be better. Each suggestion shows
  a confidence percentage.
- **Gives you the steps.** Each plan comes as a short card. In the beta, a "yes" gives you the
  exact steps to take yourself. Next, connectors will prefill and carry out those steps for
  you, in stages, starting read-only.
- **Explains and answers.** A chat built on Claude answers questions using tools over your own
  data. Every sentence passes a guardrail check before you see it.
- **Shows the whole picture.** One home page covers net worth, safe-to-spend and what's coming
  up, with a page for each area. Large figures are hidden until you choose to show them.

## How it's built

| Layer     | Choice                                                                                |
| --------- | ------------------------------------------------------------------------------------- |
| App       | Next.js 16 (App Router), React 19, TypeScript in strict mode, Tailwind CSS            |
| Data      | Postgres with Prisma; money stored as integer cents, all times in UTC                 |
| Bank data | Plaid (Transactions, Liabilities), plus CSV and OFX imports parsed in memory          |
| AI        | Anthropic Claude for the chat and explanations, with a guardrail check on every reply |
| Auth      | Auth.js with email magic links and Google sign-in                                     |
| Hosting   | Vercel, with scheduled jobs on Vercel Cron                                            |
| Testing   | Vitest (unit and integration) and Playwright (end-to-end)                             |

```
Plaid / CSV / OFX ──► normalize ──► accounts and transactions (Postgres)
                                          │
                                          ▼
                     profile of the person + detectors for every area
                                          │
                       ┌──────────────────┴──────────────────┐
                       ▼                                     ▼
          plans as cards with the steps     chat: Claude + tools over the person's data
                       │                                     │
                       ▼                                     ▼
          yes / no  (connectors next)                 guardrail check
```

## Engineering highlights

- **Deterministic money math.** The model never calculates a dollar figure. All numbers come
  from tested code; Claude explains them and calls tools to look them up.
- **One person's data stays theirs.** Every user-owned table carries a `user_id`, and every
  query goes through a helper scoped to the signed-in user. An access-proof test checks every
  route: someone holding another person's IDs gets nothing back.
- **Secrets handled carefully.** Bank access tokens are encrypted with AES-256-GCM and never
  logged or returned. Uploaded files are parsed in memory and never stored raw.
- **Guardrails on AI output.** Before anything is shown, Claude's replies are checked for
  unsupported figures, promised outcomes, and investment, tax or legal advice.
- **An audit trail.** Every sensitive event is written to an audit log.
- **Tested heavily.** There are about 9,300 test cases across 1,360 unit and integration
  test files and 111 Playwright specs. They include differential tests, such as the same
  person and scenarios on either side of UTC midnight, to catch time-zone bugs.
- **Safe deploys.** Only commits whose CI has passed reach production. Each deploy is checked
  live, and a failed check rolls back to the last good build.

## How it was built

Jane is built by two student founders working with a team of AI coding agents (Claude Code).
The agents work in parallel, each on its own branch, under a written rulebook and product
spec.

A change reaches users only after passing:

- **CI:** type checks, lint, unit, integration and end-to-end tests.
- **Repository gates:** checks on database migrations, protected test files and an access map
  for every server action.
- **The green-commit deploy:** only commits with passing CI ship, and each is checked live.

Separate agents keep running security-audit and debugging rounds against local builds. Each
fix ships with a test that fails without it.

---

© 2026 The founders of Jane. All rights reserved.
