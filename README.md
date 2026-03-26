# 🛡 Riskcheck

A mobile-first web app for digital risk and incident reporting, built as a single HTML file with a [Supabase](https://supabase.com) backend. Designed to replace paper-based risk checklists in industrial and shift-based work environments.

> 🇸🇪 This app is in Swedish. All forms, labels, and checklist questions are in Swedish, and data is stored in Swedish regardless of the user's device language.

---

## Features

- **Two report types**
  - *Risk/Incident Report* — log hazards, near-misses, root causes, and proposed actions
  - *Risk Inventory* — pre-task safety checklist with structured yes/no/N/A responses
- **Smart validation** — mandatory comment fields appear automatically when a critical answer is given (e.g. "Yes, this risk exists" or "No, I can't find the fire equipment")
- **Follow-up questions** — context-sensitive sub-questions, such as lone worker communication checks
- **Login with name + PIN** — no email or account required
- **Admin view** — see all reports across all users, filter by type, view details
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

create policy "Läs användare" on anvandare for select using (true);
create policy "Skapa rapport" on rapporter for insert to anon with check (true);
create policy "Se rapporter" on rapporter for select to anon using (true);
create policy "Radera rapport" on rapporter for delete to anon using (true);

grant usage on schema public to anon;
grant select on anvandare to anon;
grant select, insert, delete on rapporter to anon;
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

Upload `index.html` to GitHub Pages (or any static host) and share the URL with your team.

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

---

## Security Notes

- The Supabase `anon` key is embedded in the HTML file. Since the repo is public, the key is visible in source code.
- Row Level Security (RLS) is enabled on all tables, limiting what the key can access.
- Staff must authenticate with a valid name + PIN before any data can be read or written.
- For higher security requirements, consider rate limiting login attempts via a Supabase Edge Function.

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
