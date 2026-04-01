# DailyLogicGame

A Wordle-inspired daily logic puzzle game. Each day, a new seating-arrangement puzzle is presented — figure out who sits where before the day resets. Build your streak, share your results, and challenge your friends.

---

## How It Works

1. You're shown a set of seats and a cast of people.
2. Use the provided clues and constraints to deduce the correct seating arrangement.
3. Drag and drop people into their correct seats.
4. Stuck? Watch a short ad to unlock a hint.
5. Come back tomorrow for a new puzzle.

The setting changes daily — one day it's a park bench, the next it's a movie theater. Difficulty varies too, keeping things fresh.

---

## Features

- **One puzzle per day** — same puzzle for everyone, resets at midnight
- **Drag-and-drop interface** — intuitive placement with immediate feedback
- **Difficulty variety** — puzzles range from Easy to Hard across different dimensions
- **Streaks** — track how many days in a row you've solved the puzzle
- **Sharing** — share your result (spoiler-free) to challenge friends
- **Hints** — watch a rewarded ad to unlock a hint when you're stuck
- **Puzzle archive** — access every past puzzle (Premium)
- **No account required** — play anonymously; progress saved locally in your browser

---

## Tiers

| | Free | Premium |
|---|---|---|
| Daily puzzle | Yes | Yes |
| Streaks | Yes | Yes (synced across devices) |
| Sharing | Yes | Yes |
| Hints (rewarded ad) | Yes | — |
| Hints (free per day) | No | Yes |
| Puzzle archive | No | Yes |
| Ads | Yes | No |

Premium access is available via a small subscription through Stripe.

---

## Tech Stack

- **Frontend:** Blazor WebAssembly
- **Backend:** ASP.NET Core Web API
- **Database:** Azure SQL Database
- **Hosting:** Azure Static Web Apps
- **Payments:** Stripe
- **Ads:** Google AdSense + rewarded ad provider

---

## Development Setup

> Setup instructions will be added as the project is scaffolded.

---

## Project Status

Early planning phase. See [PLANNING.md](PLANNING.md) for the development roadmap.
