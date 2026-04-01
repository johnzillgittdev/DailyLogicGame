# PLANNING.md — DailyLogicGame

Detailed technical and product planning document. Updated as decisions are made.

---

## Product Vision

A browser-based daily logic puzzle game modeled after Wordle's daily cadence and social sharing loop. Each day, all users receive the same seating-arrangement puzzle. Solving it maintains a streak. The game is free to play anonymously; a premium subscription unlocks the puzzle archive and removes ads.

---

## Core Game Loop

```
User visits site
    → Today's puzzle loads (from API or IndexedDB cache)
    → User reads scenario and constraints
    → User drags people into seats
    → User submits arrangement
        → Correct: celebration animation, streak updated, share prompt shown
        → Incorrect: feedback given, user continues trying
    → (Optional) User watches rewarded ad to unlock a hint
    → User shares result
    → Puzzle locks until tomorrow
```

---

## Puzzle Design

### Data Model

```
Puzzle
├── Id (int)
├── ScheduledDate (date) — the day this puzzle is served
├── Difficulty (enum: Easy | Medium | Hard)
├── Setting (string) — e.g., "ParkBench", "BusSeats", "MovieTheater"
├── ScenarioText (string) — narrative description of the setup
├── People (JSON array)
│   └── [{ "id": "alice", "displayName": "Alice", "avatarKey": "..." }, ...]
├── Seats (JSON array)
│   └── [{ "id": "seat1", "label": "Window", "position": 1 }, ...]
├── Constraints (JSON array)
│   └── [{ "text": "Alice does not sit next to Bob." }, ...]
├── Solution (JSON object) — { "seat1": "alice", "seat2": "bob", ... }
├── HintSequence (JSON array) — ordered list of hints to reveal
├── GeneratedAt (datetime)
└── IsActive (bool)
```

**Note:** The `Solution` field must never be returned to the client. Solution validation is handled server-side.

### Difficulty Dimensions

Any combination of the following may make a puzzle easier or harder:

| Dimension | Easy | Hard |
|---|---|---|
| People/seat count | 3–4 | 6–8 |
| Constraint count | Many (easy deductions) | Few (requires elimination) |
| Constraint clarity | Direct ("Alice sits first") | Indirect ("Alice is not adjacent to Bob") |
| Red herrings | None | Possible |

### Settings (examples — expand over time)
- Park bench
- City bus seats
- Movie theater row
- Classroom desks
- Dinner table
- Airplane row
- Roller coaster car

---

## Monetization Plan

### Ad Strategy
- **Rewarded ads:** Primary ad interaction. User explicitly opts in to watch a short ad in exchange for one hint. This respects the user experience while generating revenue.
- **Banner ads:** Small, non-intrusive banner shown to free users. Removed for Premium subscribers.
- **Ad provider:** Google AdSense for banners. Rewarded ad provider TBD — needs web support (not just mobile). Candidates: Google Ad Manager, Playwire, or a custom interstitial approach.

### Stripe Integration
- One subscription tier: **Premium**
- Managed via Stripe Billing (monthly or annual)
- Webhook listener on the backend updates user subscription status
- Stripe Customer Portal for users to manage/cancel

### Premium Features
- Puzzle archive (all past puzzles, playable on-demand)
- No ads
- Free hints (limited per day, e.g., 2 per puzzle — exact number TBD)
- Cross-device streak sync (requires account)

---

## User System

### Anonymous Users
- No sign-up required
- Game state stored in **IndexedDB**:
  - Today's puzzle state (current placement, hints used, solved status)
  - Completion history (puzzle ID + date + solved bool)
  - Streak data (current streak, longest streak, last played date)
- Data is local only — clearing browser data resets everything

### Registered Users (Free Account)
- Email + password via ASP.NET Core Identity
- Streak and completion history synced to server
- Required for Premium upgrade

### Premium Users
- All registered user features
- Stripe subscription active
- Archive access unlocked on backend
- Hint entitlement tracked server-side

---

## API Design (planned endpoints)

```
GET  /api/puzzle/today          → Returns today's puzzle (without solution)
GET  /api/puzzle/{id}           → Returns a specific puzzle (archive, Premium only)
GET  /api/puzzle/archive        → Returns list of past puzzle metadata (Premium only)
POST /api/puzzle/{id}/validate  → Validates a submitted solution, returns correct/incorrect
POST /api/puzzle/{id}/hint      → Returns next hint for a puzzle (records hint use server-side)

POST /api/auth/register         → Create account
POST /api/auth/login            → Login
POST /api/auth/logout           → Logout

GET  /api/user/me               → Get current user profile + subscription status
GET  /api/user/stats            → Get streak + completion stats

POST /api/subscription/create-session   → Create Stripe Checkout session
POST /api/subscription/webhook          → Stripe webhook handler
GET  /api/subscription/portal           → Create Stripe Customer Portal session
```

---

## Data Storage Strategy

| Data | Where stored | Why |
|---|---|---|
| Today's puzzle state | IndexedDB | Works offline, no login needed |
| Streak (anonymous) | IndexedDB | Local, immediate |
| Streak (registered) | Server DB | Cross-device sync |
| Puzzle solutions | Server DB only | Never sent to client |
| User accounts | Server DB | Auth + subscription |
| Subscription status | Server DB (Stripe webhook updates) | Source of truth |

---

## Puzzle Generation Strategy

Puzzles are generated in **batches of ~50** by an AI service, then stored in the database. This avoids runtime AI costs and allows for human review before puzzles go live.

### Generation Process (planned)
1. A scheduled job or manual trigger calls the AI service with a generation prompt.
2. The AI returns a structured puzzle JSON (scenario, people, seats, constraints, solution, hints).
3. A validation step checks that the puzzle has exactly one valid solution.
4. Valid puzzles are stored in the database with `IsActive = false`.
5. A human reviewer approves puzzles and sets `ScheduledDate`, flipping `IsActive = true`.

### AI Service
- **Candidates:** Claude Haiku (Anthropic), GPT-4o-mini (OpenAI)
- Selection criteria: cost per puzzle, accuracy of constraint generation, single-solution guarantee
- Evaluate both at implementation time with sample prompts

---

## Sharing Mechanic

Post-solve, the user is shown a shareable result. Format TBD — options:

1. **Text summary:** "Daily Logic Game #42 ✅ — Solved! 🪑🧍🧍🧍"
2. **Emoji grid:** Visual representation of the seating (spoiler-free)
3. **Image card:** Generated share card with puzzle number, difficulty, and result

The share button copies to clipboard and optionally opens native share dialog on mobile.

---

## Hosting Plan

### Near-term (development & testing)
- **Azure Static Web Apps** (free tier)
  - Hosts Blazor WASM frontend as static files
  - Hosts ASP.NET Core API via the integrated Azure Functions runtime
  - Free tier includes SSL, custom domain support, and GitHub Actions CI/CD

### Database (development)
- **Azure SQL Database — Serverless tier**
  - Auto-pauses when not in use (minimizes cost during development)
  - Scales up automatically under load

### Long-term (production)
- Evaluate moving API to Azure App Service if Azure Functions limitations are hit
- Azure SQL Database — General Purpose tier for production load
- Azure CDN for static asset caching
- Application Insights for monitoring and error tracking

---

## Development Phases

### Phase 1 — Core Game (Frontend Only)
- [ ] Scaffold Blazor WASM project
- [ ] Build puzzle display component (seats + people)
- [ ] Implement drag-and-drop placement
- [ ] Hardcode one sample puzzle for development
- [ ] Implement solution validation (client-side for now)
- [ ] Streak tracking via IndexedDB
- [ ] Basic share mechanic (copy to clipboard)

### Phase 2 — Backend & Real Puzzles
- [ ] Scaffold ASP.NET Core Web API
- [ ] Set up Azure SQL Database and EF Core models
- [ ] Implement puzzle endpoints (today, validate)
- [ ] Move solution validation server-side
- [ ] Connect frontend to API
- [ ] Deploy to Azure Static Web Apps

### Phase 3 — AI Puzzle Generation
- [ ] Design and test puzzle generation prompt
- [ ] Evaluate Claude Haiku vs GPT-4o-mini
- [ ] Build generation script/job
- [ ] Build single-solution validation logic
- [ ] Build admin review interface (simple)
- [ ] Populate initial puzzle queue

### Phase 4 — User Accounts & Premium
- [ ] ASP.NET Core Identity setup
- [ ] Register/login UI in Blazor
- [ ] Sync streak/history to server for logged-in users
- [ ] Stripe integration (subscription, webhook, portal)
- [ ] Archive endpoint + UI (Premium gate)
- [ ] Remove ads for Premium users

### Phase 5 — Ads & Hints
- [ ] Integrate Google AdSense banner (free users)
- [ ] Research and integrate rewarded ad provider (web-compatible)
- [ ] Build hint reveal UI
- [ ] Rewarded ad → hint unlock flow
- [ ] Premium hint entitlement (free hints per day)

### Phase 6 — Polish & Launch
- [ ] Mobile responsiveness pass
- [ ] Accessibility audit
- [ ] Performance audit (Lighthouse)
- [ ] Custom domain setup
- [ ] Privacy policy + cookie consent (GDPR/CCPA)
- [ ] Analytics (basic, privacy-respecting)
- [ ] Launch

---

## Open Questions

| Question | Status |
|---|---|
| AI service for puzzle generation (Haiku vs GPT-4o-mini) | Decide at Phase 3 |
| Puzzle constraint storage (JSON column vs normalized) | Decide at Phase 2 |
| Sharing mechanic format (text, emoji, image card) | Decide at Phase 1 |
| Is difficulty user-selectable or fixed per day | Undecided |
| Rewarded ad provider for web | Research needed |
| Number of free hints per day for Premium users | Undecided |
| Puzzle archive pricing (monthly vs one-time) | Undecided |
