# Il Cliente Indimenticabile

**Privacy-first CRM desktop app per professioniste del benessere — offline-first, no cloud, no subscription.**

![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS-blue)
![Stack](https://img.shields.io/badge/stack-Electron%20%2B%20React%20%2B%20Express%20%2B%20SQLite-purple)
![License](https://img.shields.io/badge/license-Proprietary-red)
![Status](https://img.shields.io/badge/status-v1.0.0%20%7C%20Pre--release-orange)

---

## Overview

CRM desktop distribuito come installer (`.exe` / `.dmg`) per coach, terapeute, yoga teacher e personal trainer che gestiscono 15–50 clienti ricorrenti in sessioni 1-to-1. Tutta la logica applicativa gira in locale: Express su `localhost:3001`, database SQLite in `userData`, nessuna richiesta in uscita eccetto invio comunicazioni (mailto/wa.me/SMTP).

---

## Architettura
┌─────────────────────────────────────────────────┐
│ COMPUTER UTENTE │
│ │
│ Electron 33 (shell, IPC, notifiche native) │
│ │ │
│ ▼ │
│ React SPA ──HTTP──▶ Express API │
│ (Vite, port 5173) (port 3001) │
│ │ │
│ ▼ │
│ SQLite (better-sqlite3) │
│ app.getPath('userData')/db.sqlite │
│ │
│ Comunicazioni esterne (solo su richiesta): │
│ - mailto: → client email di sistema │
│ - wa.me/ → WhatsApp Desktop o Web │
│ - SMTP → Nodemailer (credenziali locali) │
└─────────────────────────────────────────────────┘


### Principi chiave
- **Single-user by design** — un'installazione = una coach, zero multi-tenancy
- **Nessun server remoto** — Express è un processo figlio di Electron, mai esposto a internet
- **IPC hardened** — `contextIsolation: true`, `nodeIntegration: false`, whitelist esplicita in `preload.js`
- **Comunicazioni manuali** — l'app prepara i messaggi, l'utente li invia; nessun invio automatico silenzioso

---

## Stack

### Desktop Shell
| Tecnologia | Versione | Note |
|-----------|---------|------|
| Electron | 33.x | Shell cross-platform |
| Electron Forge | 7.x | Build `.exe` (Squirrel) + `.dmg` |
| electron-updater | 6.x | Auto-update via GitHub Releases |

### Frontend
| Tecnologia | Versione | Note |
|-----------|---------|------|
| React | 18.x | SPA con React Router 6 |
| Vite | 5.x | Build tool + HMR |
| TailwindCSS | 3.x | Palette custom con 5 scale di colore + token semantici |
| shadcn/ui | latest | Componenti accessibili unstyled (WCAG AA) |
| React Query | 5.x | Server state, optimistic updates, cache invalidation |
| Zustand | 4.x | Stato globale: auth + bozze sessione (persist) |
| React Hook Form + Zod | 7.x + 3.x | Form + schema validation condiviso frontend/backend |
| Recharts | 2.x | Mood trend (line) + tag frequency (bar) |
| Framer Motion | 11.x | Skeleton loaders + transizioni pagina |

### Backend (embedded in Electron)
| Tecnologia | Versione | Note |
|-----------|---------|------|
| Node.js | 20.x LTS | Runtime |
| Express | 4.x | REST API con error handling centralizzato |
| better-sqlite3 | 9.x | Driver sincrono — ricompilato per Electron via `electron-rebuild` |
| Drizzle ORM | latest | ORM type-safe + migrations |
| bcryptjs | 2.x | Hash password (puro JS, no native addon) |
| jsonwebtoken | 9.x | JWT stateless per auth locale |
| node-cron | 3.x | Scheduler follow-up → notifiche native OS |
| Nodemailer | 6.x | Invio SMTP opzionale |
| html-pdf-node | 1.x | Export PDF report clienti (generato localmente) |
| Helmet | 7.x | Security headers su tutte le risposte API |

---

## Schema Database (SQLite)

10 tabelle relazionali:
users ─┐
├──▶ clients ──▶ sessions ──▶ session_tags ──▶ tags
│ │
│ ├──▶ goals
│ ├──▶ communication_log
│ └──▶ calendar_events
│
├──▶ calendar_slots
└──▶ automation_templates


**Decisioni di schema rilevanti:**
- `settings` su `users` è un campo JSON serializzato (timezone, SMTP config, notification prefs) — evita migration per ogni preferenza
- `session_tags` many-to-many con `ON DELETE CASCADE`
- `calendar_slots` definisce la disponibilità settimanale ricorrente (pattern), `calendar_events` le istanze concrete
- Nessun soft delete esplicito in v1 — considerare per v2

---

## Sicurezza & Privacy (GDPR by design)

- **Zero cloud transmission** — nessun dato lascia il filesystem locale
- **contextIsolation + IPC whitelist** — il renderer non accede mai a Node.js direttamente
- **bcrypt** per hash password con salt
- **JWT** stateless — nessuna sessione lato server
- **Helmet.js** su tutte le risposte Express
- **Nessuna telemetria** — l'app non "telefona a casa"
- DB salvato in `app.getPath('userData')` — mai nella cartella di installazione (che su Windows è read-only post-install)

---

## Decisioni Architetturali

| Decisione | Razionale |
|-----------|-----------|
| SQLite invece di PostgreSQL | Single-user desktop — zero config, zero processo DB separato da avviare |
| `better-sqlite3` (sync) invece di `sqlite3` (async) | Semplifica gli IPC handler Electron, evita callback hell e race conditions |
| Express embedded invece di Electron IPC puro | Separa il data layer dall'UI; il server potrebbe essere estratto in futuro senza refactor massiccio |
| Zustand invece di Redux | Superficie di stato globale ridotta (auth + draft) — Redux sarebbe over-engineered |
| Nessun TypeScript in v1 | Trade-off velocità per MVP; Zod compensa la type safety a runtime sui boundary API |
| React Query per server state | Cache automatica, background refetch, optimistic updates — elimina il boilerplate di loading/error manual |

---

## Pattern Critici (da rispettare)

**1. Ricompila `better-sqlite3` per Electron**
```js
// forge.config.js
hooks: {
  packageAfterPrune: async (config, buildPath) => {
    execSync('electron-rebuild -f -w better-sqlite3', { cwd: buildPath })
  }
}
```
Senza questo l'app crasha silenziosamente all'avvio su Windows.

**2. Unico punto di connessione DB**
Tutta l'app passa per `server/config/database.js`. Mai connessioni dirette altrove.

**3. Zod schema condiviso**
Stesse validazioni importate sia nelle route Express che nei form React. Zero discrepanze frontend/backend.

**4. Response shape uniforme**
```js
// Sempre e comunque:
{ success: true, data: {...} }
{ success: false, error: "Messaggio", code: 400 }
```

**5. Query keys centralizzate**
Tutti i React Query keys in `client/src/lib/queryKeys.js` — evita cache stale dopo mutations.

**6. Auto-save bozza con Zustand persist**
`clearDraft()` chiamato SOLO dopo conferma di salvataggio avvenuto. L'utente non perde mai una sessione compilata a metà.

---

## Struttura Progetto
├── electron/ # Main process, preload, IPC handlers, forge config
├── server/ # Express API (routes → controllers → services → DB)
│ ├── routes/
│ ├── controllers/
│ ├── services/ # Business logic
│ ├── middleware/ # Auth, validation, error handling
│ └── database/ # Schema Drizzle + migrations SQL
├── client/src/
│ ├── pages/ # Viste full-page
│ ├── components/ # Componenti per dominio (clients, sessions, calendar…)
│ ├── hooks/ # React Query hooks per dominio
│ ├── store/ # Zustand (auth, ui, draft)
│ └── lib/ # api.js, queryKeys.js, utils, electron bridge
└── data/ # .gitkeep — il .sqlite viene creato a runtime

---

## Status

Prodotto commerciale — source code non open source. Questo repository è un **portfolio showcase** dell'architettura e delle scelte ingegneristiche.

Costruito da **LMG Soluzioni Digitali** - https://lmgsoluzionidigitali.it/
