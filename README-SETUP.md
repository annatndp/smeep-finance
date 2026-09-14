# Smeep Finance — Setup & Deploy Guide (v1)

One-file PWA (index.html) + Supabase (email/password auth, realtime sync) + GitHub Pages. Same formula as your Kho ý tưởng app.

## 1. Supabase — create the tables

Open your Supabase project → **SQL Editor** → paste and run this whole block:

```sql
-- Expense transactions (Add tab / Dashboard)
create table expenses (
  id uuid primary key default gen_random_uuid(),
  amount numeric not null check (amount > 0),
  category text not null default 'groceries',   -- groceries | dining | shopping | amex
  note text default '',
  spent_on date not null default current_date,
  created_at timestamptz default now()
);

-- Monthly checklist ticks (Oversikt tab)
create table oversikt_checks (
  id uuid primary key default gen_random_uuid(),
  month text not null,      -- 'YYYY-MM'
  item text not null,       -- leie, laan, billsp, billsh, transport, bsu, notouch, poker, asia, wedding
  person text not null,     -- 'I' or 'N'
  created_at timestamptz default now(),
  unique (month, item, person)
);

alter table expenses enable row level security;
alter table oversikt_checks enable row level security;

create policy "authenticated full access" on expenses
  for all to authenticated using (true) with check (true);
create policy "authenticated full access" on oversikt_checks
  for all to authenticated using (true) with check (true);

-- Instant sync between phones
alter publication supabase_realtime add table expenses;
alter publication supabase_realtime add table oversikt_checks;
```

Note: everyone who can sign in to this project sees the same shared data — that's intentional for the two of you.

## 2. Create the login user(s)

Supabase → **Authentication → Users → Add user** → enter email + password (tick "Auto confirm user"). Create one account, or two (one for Isaac, one for Nga) — both see the same data either way.

If you don't want strangers signing themselves up: **Authentication → Sign In / Up → disable "Allow new users to sign up"**.

## 3. config.js

Copy `config.example.js` → rename to `config.js` → paste your values from **Project Settings → API**:

```js
window.SUPABASE_URL = "https://xxxx.supabase.co";
window.SUPABASE_ANON_KEY = "eyJ...";  // the "anon public" key
```

The anon key is designed to be public (RLS + auth protect the data), so a public repo is fine.

If you're reusing the same Supabase project as Kho ý tưởng, config.js is identical to that repo's — just copy it over. The new tables live happily next to the `videos` table.

## 4. GitHub Pages

1. Create a new **public** repo, e.g. `smeep-finance`.
2. Upload these 7 files via the GitHub web UI:
   `index.html`, `sw.js`, `manifest.webmanifest`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`, and your `config.js`.
3. **Settings → Pages** → Source: Deploy from a branch → `main`, folder `/ (root)` → Save.
4. After ~1 minute the app is live at `https://annatndp.github.io/smeep-finance/`.
5. Open it on each phone → **Add to Home Screen** → it runs fullscreen like a native app.

If a Pages deploy fails with 503: it's GitHub's side — Actions tab → Re-run failed jobs.

## 5. Verifying updates

A tiny **v1** marker sits next to the sync status under the app title. Whenever you edit index.html, bump that number so you can confirm the new version is live. The service worker is network-first for HTML, so it never gets stuck on an old cached version.

Sync status meanings: **● Synced** (green) = connected · **Connecting…** = loading · **Connection error** = check config/tables · **Not configured** = config.js missing.

## 6. Where the numbers live (easy to edit later)

All plan numbers are plain constants near the top of the `<script type="module">` in index.html:

- `BUDGET` — the 8 000 kr food budget (amount + which categories count).
- `CATS` — the 4 expense categories (name, emoji, color).
- `PLAN` — the full oversikt: salaries, every fixed cost, every saving, per person, including the notes. Change a salary or add a saving item here, bump the version marker, re-upload — done.

Checklist ticks are stored per month, so past months keep their history when the plan changes.

## 7. What's synced vs local

- Expenses, deletions, and oversikt ticks → Supabase, realtime on both phones.
- Dark mode choice → saved on each device (localStorage), so you can have dark and Isaac light.
