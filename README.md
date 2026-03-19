# Il Cliente Indimenticabile

**Privacy-first CRM desktop app for wellness professionals who work 1-to-1 with clients.**

> Built for coaches, therapists, yoga teachers, and personal trainers who manage 15–50 recurring clients and need a fast, offline tool to track sessions, automate follow-ups, and never forget a detail.

![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS-blue)
![Stack](https://img.shields.io/badge/stack-Electron%20%2B%20React%20%2B%20Express%20%2B%20SQLite-purple)
![License](https://img.shields.io/badge/license-Proprietary-red)
![Status](https://img.shields.io/badge/status-v1.0.0%20%7C%20Pre--release-orange)

---

## The Problem

Wellness professionals juggle client notes across WhatsApp chats, Google Docs, paper notebooks, and memory. Follow-ups get forgotten. Patterns go unnoticed. Existing CRMs are cloud-based, subscription-heavy, and built for sales teams — not for someone who needs to remember that *Marta always struggles with work-life balance after holiday breaks*.

## The Solution

A **single desktop application** that lives on the coach's computer. No cloud. No subscription. No data leaving the machine. Just a clean, Notion-inspired interface where every client interaction is captured in under 5 minutes.

---

## Core Features

| Feature | Description |
|---------|-------------|
| **Client Profiles** | Rich profiles with personal triggers, communication preferences, goals, and status tracking |
| **Session Recording** | Post-session notes in ~5 min: mood tracking (1-10), topics, breakthroughs, assigned tasks |
| **Smart Follow-ups** | Automated desktop notifications when follow-ups are due (configurable delay, default 48h) |
| **Communication Hub** | Send pre-compiled messages via Email (SMTP/mailto) or WhatsApp with merge fields (`{nome}`, `{tema}`, `{compito}`) |
| **Analytics** | Mood trend graphs, tag frequency analysis, automatic pattern recognition per client |
| **Calendar** | Weekly availability slots, monthly/weekly views, .ical export to Google/Apple/Outlook |
| **PDF Reports** | Exportable client progress reports (last 10 sessions + mood graphs) |
| **Auto-save Drafts** | Session forms auto-save to localStorage — no data loss on accidental close |
| **Guided Onboarding** | 4-step wizard: profession → client count → import first clients → set availability |

---

## Tech Stack

### Architecture

```
┌─────────────────────────────────────────────────┐
│              COACH'S COMPUTER                    │
│                                                  │
│   Electron Shell (window, menus, notifications)  │
│        │                                         │
│        ▼                                         │
│   React SPA ──HTTP──▶ Express API                │
│   (port 5173)         (port 3001)                │
│                           │                      │
│                           ▼                      │
│                    SQLite Database                │
│                   (local file only)               │
│                                                  │
│   External (via shell):                          │
│   • mailto: → default email client               │
│   • wa.me/  → WhatsApp                          │
│   • SMTP    → email provider (optional)          │
└─────────────────────────────────────────────────┘
```

### Frontend

| Technology | Purpose |
|-----------|---------|
| **React 18** | Component-based UI |
| **Vite 5** | Build tool & HMR dev server |
| **TailwindCSS 3** | Utility-first styling with custom Notion-inspired palette |
| **shadcn/ui** | Accessible, unstyled component primitives |
| **React Router 6** | Client-side routing |
| **React Query (TanStack) 5** | Server state, caching & background sync |
| **Zustand 4** | Lightweight global state (auth, drafts) |
| **React Hook Form 7 + Zod 3** | Form handling with schema validation |
| **Recharts 2** | Data visualization (mood trends, tag frequency) |
| **Framer Motion 11** | Animations & skeleton loaders |
| **Lucide React** | Icon system |

### Backend

| Technology | Purpose |
|-----------|---------|
| **Node.js 20 LTS** | Runtime |
| **Express 4** | REST API with structured error handling |
| **better-sqlite3 9** | Synchronous SQLite driver (recompiled for Electron) |
| **JWT + bcryptjs** | Stateless authentication |
| **node-cron 3** | Scheduled follow-up checks |
| **Nodemailer 8** | SMTP email delivery |
| **Helmet 7** | HTTP security headers |

### Desktop

| Technology | Purpose |
|-----------|---------|
| **Electron 33** | Cross-platform desktop shell |
| **Electron Forge 7** | Build system → `.exe` / `.dmg` installers |
| **electron-updater 6** | Auto-update via GitHub Releases |
| **IPC (preload.js)** | Secure bridge between renderer and main process |

---

## Database Schema

10+ relational tables in SQLite:

```
users ─┐
       ├──▶ clients ──▶ sessions ──▶ session_tags ──▶ tags
       │        │
       │        ├──▶ goals
       │        ├──▶ communication_log
       │        └──▶ calendar_events
       │
       ├──▶ calendar_slots
       └──▶ automation_templates
```

Key design decisions:
- **Single-user architecture** — one coach per installation, no multi-tenancy overhead
- **Soft deletes** — client data is never permanently lost by accident
- **Denormalized analytics** — pre-computed mood averages for fast dashboard rendering

---

## Security & Privacy

This application was designed with **GDPR compliance** as a first-class concern:

- **Zero cloud transmission** — all data stays on the local filesystem
- **Electron security hardening** — `contextIsolation: true`, `nodeIntegration: false`, whitelisted IPC channels only
- **Password hashing** — bcrypt with salt rounds
- **JWT auth** — stateless tokens, no session storage on server
- **Helmet.js** — security headers on all API responses
- **No telemetry** — the app does not phone home

---

## Project Structure

```
├── electron/               # Desktop shell & IPC handlers
│   ├── main.js             # Window creation, server bootstrap
│   ├── preload.js          # Secure IPC bridge
│   ├── forge.config.js     # Build configuration
│   └── ipc/                # Handler modules (backup, PDF, comms)
│
├── server/                 # Express REST API
│   ├── routes/             # Endpoint definitions
│   ├── controllers/        # Request handlers
│   ├── services/           # Business logic & DB queries
│   ├── middleware/          # Auth, validation, error handling
│   ├── config/             # Constants, DB connection, SMTP
│   └── database/           # Schema & migrations
│
├── client/                 # React SPA
│   └── src/
│       ├── pages/          # Full-page views
│       ├── components/     # Domain-organized UI components
│       ├── hooks/          # React Query data hooks
│       ├── store/          # Zustand stores
│       ├── lib/            # API client, utilities, query keys
│       └── styles/         # Global CSS & design tokens
│
└── data/                   # SQLite database (created at first run)
```

---

## Design System

Custom Notion-inspired palette optimized for long working sessions:

| Token | Hex | Usage |
|-------|-----|-------|
| Powder Blush | `#FF6B6B` | Primary actions, CTAs |
| Eggshell | `#F5DEB3` | Warm backgrounds, cards |
| Icy Aqua | `#00D9D9` | Success states, positive mood |
| Light Blue | `#D0E8F2` | Sidebar, secondary surfaces |
| Blue Slate | `#475569` | Text, borders, icons |

---

## Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| **SQLite over PostgreSQL** | Single-user desktop app — no need for a database server. Simpler deployment, zero configuration |
| **better-sqlite3 (sync) over sqlite3 (async)** | Synchronous API simplifies Electron IPC handlers and avoids callback nesting |
| **Zustand over Redux** | Minimal boilerplate for a small global state surface (auth + draft) |
| **React Query for server state** | Automatic cache invalidation, background refetch, optimistic updates — eliminates manual loading/error state management |
| **Express embedded in Electron** | Decouples frontend from data layer. Could be extracted to a standalone server if needed in the future |
| **No TypeScript (v1)** | Velocity trade-off for MVP. Zod schemas provide runtime type safety at API boundaries |

---

## Distribution

| Platform | Format | Build Tool |
|----------|--------|-----------|
| Windows | `.exe` installer | Electron Forge (Squirrel) |
| macOS | `.dmg` installer | Electron Forge (DMG) |

Auto-updates supported via `electron-updater` connected to GitHub Releases.

---

## Status

This is a **commercial product** — source code is not open source.

This repository serves as a **technical portfolio showcase** demonstrating:
- Full-stack desktop application architecture
- Electron + React + Express integration patterns
- Offline-first data management with SQLite
- Privacy-by-design engineering
- Production-grade project structure and security practices

---

## About

Built by **LMG Soluzioni Digitali** — digital products for the Italian wellness market.

Part of the *Ecosistemi Digitali e Notion Aesthetic* product line.

📧 For licensing or inquiries, visit our website.

---

<sub>Built with Electron · React · Express · SQLite · TailwindCSS · shadcn/ui</sub>
