# 🛡 Riskcheck

A mobile-first web app for digital risk and incident reporting, built as a single HTML file with a [Supabase](https://supabase.com) backend. Designed to replace paper-based risk checklists in industrial and shift-based work environments. The app is in Swedish — all forms, labels, and checklist questions are in Swedish, and data is stored in Swedish regardless of the user's device language.

---

## Features

- **Two report types**
  - *Risk/Incident Report* — log hazards, near-misses, root causes, and proposed actions
  - *Risk Inventory* — pre-task safety checklist with structured yes/no/N/A responses
- **Smart validation** — mandatory comment fields appear automatically when a critical answer is given (e.g. "Yes, this risk exists" or "No, I can't find the fire equipment")
- **Color-coded responses** — green indicates a safe answer, red indicates a risk or concern, making post-task reviews quick and intuitive
- **Follow-up questions** — context-sensitive sub-questions, such as lone worker communication checks
- **Login with name + PIN** — no email or account required, with automatic lockout after 3 failed attempts (unlocks after 30 minutes or manually by admin)
- **Admin view** — see all reports across all users, filter by type and by year/month with a period summary showing report counts, delete reports
- **User management** — admins can add new users, remove existing ones, toggle roles, reset PIN codes and optionally force a PIN change at next login
- **PIN self-service** — users can change their own PIN at any time from the nav bar
- **Statistics** — charts showing report counts, root cause frequency, and reports per person
- **Export** — download all data as CSV (semicolon-separated, UTF-8 BOM for Excel compatibility) or JSON
- **Three themes** — Dark, Classic, and Light, saved per device
- **PWA-ready** — can be added to the home screen on iOS and Android for a native app feel
- **Works offline for viewing** — data syncs with Supabase when connected

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Single-file HTML + vanilla JS + CSS |
| Database | Supabase (PostgreSQL) |
| Hosting | GitHub Pages |
| Fonts | Google Fonts (DM Sans, DM Mono) |

No build tools, no frameworks, no dependencies to install.

---

## Getting Started

### 1. Set up Supabase

Create a free project at [supabase.com](https://supabase.com), then run the following SQL in the **SQL Editor**:

```sql
create table anvandare (
  id uuid primary key default gen_random_uuid(),
  namn text not null,
  pin text not null,
  roll text not null default 'personal',
  tvinga_pin_byte boolean default false,
  sparad timestamptz default now()
);

create table rapporter (
  id uuid primary key default gen_random_uuid(),
  anvandare_id uuid references anvandare(id),
  anvandare_namn text,
  typ text not null,
  datum date,
  tid time,
  plats text,
  beskrivning text,
  orsaker jsonb,
  atgarder text,
  tankt jsonb,
  risker jsonb,
  varfinns jsonb,
  kommentar text,
  sparad timestamptz default now()
);

alter table anvandare enable row level security;
alter table rapporter enable row level security;

grant usage on schema public to anon;
grant select, insert, update, delete on anvandare to anon;
grant select, insert, delete on rapporter to anon;

create policy "Läs användare" on anvandare for select using (true);
create policy "Skapa användare" on anvandare for insert with check (true);
create policy "Uppdatera användare" on anvandare for update to anon using (true);
create policy "Ta bort användare" on anvandare for delete to anon using (true);

create table inloggningar (
  id uuid primary key default gen_random_uuid(),
  anvandare_id uuid references anvandare(id) on delete cascade,
  namn text not null,
  typ text not null,
  sparad timestamptz default now()
);

alter table inloggningar enable row level security;

grant select, insert on inloggningar to anon;

create policy "Läs inloggningar" on inloggningar for select using (true);
create policy "Skapa inloggning" on inloggningar for insert with check (true);
```

### 2. Add users

```sql
-- Admin user
insert into anvandare (namn, pin, roll) values ('Admin', 'your-pin', 'admin');

-- Regular users
insert into anvandare (namn, pin, roll) values ('First Last', '1234', 'personal');
```

### 3. Configure the app

Open `index.html` and update the following constants near the top of the `<script>` tag:

```javascript
const SUPABASE_URL = 'https://your-project.supabase.co';
const SUPABASE_KEY = 'your-publishable-api-key';
```

### 4. Deploy

Upload both `index.html` and `sw.js` to GitHub Pages (or any static host) — both files must be in the same directory. Share the URL with your team.

The app includes a Service Worker that caches resources automatically. When you upload a new version of `index.html`, users will receive the update the next time they close and reopen the app — no reinstallation needed.

---

## Usage

### Staff
1. Open the URL in a mobile browser
2. Tap **Share → Add to Home Screen** to install as an app
3. Log in with your name and PIN
4. Fill in either a Risk Inventory or a Risk/Incident Report and save

### Admin
Log in with an admin account to access:
- All reports from all users
- Statistics and charts
- CSV/JSON export
- User management — add, remove, change roles, reset PIN codes and force PIN change at next login

---

## Security Notes

- The Supabase `anon` key is embedded in the HTML file. Since the repo is public, the key is visible in source code.
- Row Level Security (RLS) is enabled on all tables, limiting what the key can access.
- Staff must authenticate with a valid name + PIN before any data can be read or written.
- Accounts are automatically locked for 30 minutes after 3 failed login attempts. Admins can unlock accounts manually in the Users tab.

---

## Checklist Logic

| Section | Triggers mandatory comment |
|---|---|
| Have you considered... | **No** answer |
| Is avoidance needed? | **Yes** answer |
| Risks present | **Yes** answer |
| Can you locate... | **No** answer |
| Lone worker → communication | Sub-question: **No** answer |

---

## License

© 2026 CARÅ. All rights reserved.
