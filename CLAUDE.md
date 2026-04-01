# CLAUDE.md — DailyLogicGame

This file provides context for AI agents working on this project. Read it fully before writing any code or making suggestions.

---

## Project Overview

DailyLogicGame is a Wordle-inspired daily web game where users solve a seating-arrangement logic puzzle each day. The setting varies (park bench, bus, movie theater, etc.) and difficulty can vary across multiple dimensions. Completed puzzles earn the user a streak. Users can share results with friends. An archive of past puzzles is available behind a small paywall.

---

## Core Behaviors for AI Agents

- **Always ask for clarification** if requirements, design decisions, or implementation choices are ambiguous. Do not make assumptions and proceed unilaterally.
- Do not add features, refactoring, comments, or error handling beyond what is explicitly asked.
- Do not create new files unless absolutely necessary — prefer editing existing ones.
- Do not introduce security vulnerabilities (XSS, SQL injection, CSRF, etc.).
- Match the scope of your changes to what was actually requested.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Blazor WebAssembly (.NET) |
| Backend | ASP.NET Core Web API (.NET) |
| Database | Azure SQL Database (serverless tier) |
| Hosting | Azure Static Web Apps (frontend + API together) |
| Local Storage | IndexedDB (anonymous user game state and streaks) |
| Authentication | ASP.NET Core Identity |
| Payments | Stripe |
| Ads | Google AdSense (banner) + rewarded ad provider (TBD) |
| AI Puzzle Gen | TBD — Claude Haiku or GPT-4o-mini (evaluated at implementation time) |

---

## Architecture Overview

```
[Blazor WASM Frontend]
        |
        | HTTP (REST)
        |
[ASP.NET Core Web API]
        |
   +----+----+
   |         |
[Azure SQL]  [AI Service]
             (batch puzzle generation)
```

- The frontend is a Blazor WASM SPA hosted on Azure Static Web Apps.
- Anonymous users store all game state (current puzzle state, streak, completion history) in **IndexedDB** in the browser.
- Authenticated/paid users have their data backed up to the server so it persists across devices.
- Puzzles are **pre-generated in batches** (approximately 50 at a time) by an AI service and stored in the database. They are NOT generated on-demand at runtime.
- Each puzzle has a scheduled date. The API serves today's puzzle based on the current date.

---

## Puzzle Design

### Core Concept
Each puzzle presents a set of seats and a set of people. The user must place all people into the correct seats using drag-and-drop. Constraints (implicit in the scenario description) guide the solution.

### Difficulty Dimensions
Difficulty can vary along multiple axes — any combination may be used on a given day:
- **Seat/person count** — more people and seats increases complexity
- **Constraint clarity** — harder puzzles have less obvious starting deductions
- **Constraint count** — fewer explicit constraints per person makes deduction harder
- Difficulty levels: **Easy**, **Medium**, **Hard**

### Settings
The visual setting of the puzzle changes daily (examples: park bench, bus seats, movie theater row, classroom desks, dinner table). Settings are cosmetic but should be consistent with the puzzle's scenario text.

### Interaction
- Drag-and-drop to place people into seats
- Open-ended — no move limit, user keeps trying until correct
- **Hints:** Users can watch a short rewarded ad to unlock a hint. Paid users receive a set number of free hints per day without watching an ad.

---

## Monetization

| Feature | Free (Anonymous) | Free (Account) | Premium (Paid) |
|---|---|---|---|
| Daily puzzle | Yes | Yes | Yes |
| Streaks | Yes (local only) | Yes (synced) | Yes (synced) |
| Sharing | Yes | Yes | Yes |
| Hints (rewarded ad) | Yes | Yes | — |
| Hints (free) | No | No | Yes (limited per day) |
| Puzzle archive | No | No | Yes |
| Ads | Yes | Yes | No |

- **Rewarded ads:** Users watch a short ad in exchange for a hint. This is the primary ad interaction — more natural and less intrusive than persistent banners. Small passive banner ads may also be present for free users.
- **Payments:** Stripe.
- **Ad network for rewarded ads:** TBD — needs to support web (not just mobile). Options include Google Ad Manager, AdSense for content, or a web-compatible rewarded ad SDK.

---

## Key Conventions

- Use standard .NET naming conventions (PascalCase for classes/methods, camelCase for local variables).
- API routes follow RESTful conventions under `/api/`.
- Blazor components go in `Components/`, pages in `Pages/`.
- All database access goes through a repository pattern — no raw SQL in controllers.
- **Puzzle solutions must never be sent to the client in plaintext.** Validate solutions server-side via API call.

---

## Development Phases

See [PLANNING.md](PLANNING.md) for the full phased roadmap.

---

## Open Questions (as of project start)

- Exact AI service for puzzle generation (Claude Haiku vs GPT-4o-mini) — evaluate at implementation.
- Database schema for puzzle constraints (JSON column vs normalized table).
- Sharing mechanic format (emoji grid, text summary, or image card).
- Whether difficulty is user-selectable per day or fixed by the daily puzzle schedule.
- Rewarded ad provider that supports web (not just mobile apps).
- Number of free hints per day for paid users.
