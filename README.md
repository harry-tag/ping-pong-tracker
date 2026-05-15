# Office Ping Pong Tracker

ELO-based ping pong leaderboard. Deploys to GitHub Pages with Supabase as the backend.

---

## Setup (5 minutes)

### 1. Create a Supabase project

1. Go to [supabase.com](https://supabase.com) and sign up (free)
2. Click **New project**, give it a name, set a DB password, pick a region
3. Wait ~2 minutes for provisioning

### 2. Create the tables

In your Supabase project, go to **SQL Editor** and run:

```sql
create table players (
  id       uuid primary key default gen_random_uuid(),
  name     text not null unique,
  elo      integer not null default 1000,
  wins     integer not null default 0,
  losses   integer not null default 0,
  streak   integer not null default 0,
  joined_at timestamptz default now()
);

create table games (
  id               uuid primary key default gen_random_uuid(),
  winner_id        uuid references players(id),
  loser_id         uuid references players(id),
  winner_elo_before integer,
  loser_elo_before  integer,
  winner_elo_after  integer,
  loser_elo_after   integer,
  played_at        timestamptz default now()
);

-- Allow public read/write (no auth required)
alter table players enable row level security;
alter table games enable row level security;

create policy "public read players"  on players for select using (true);
create policy "public insert players" on players for insert with check (true);
create policy "public update players" on players for update using (true);

create policy "public read games"  on games for select using (true);
create policy "public insert games" on games for insert with check (true);
```

### 3. Add your credentials to index.html

In **Supabase → Project Settings → API**, copy:
- **Project URL**
- **anon / public** key

Open `index.html` and replace these two lines near the bottom of the `<script>` tag:

```js
const SUPABASE_URL  = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

### 4. Deploy to GitHub Pages

```bash
git init
git add .
git commit -m "init"
# create a new repo on github.com, then:
git remote add origin https://github.com/YOUR_USERNAME/ping-pong-tracker.git
git push -u origin main
```

Then in your GitHub repo: **Settings → Pages → Source → Deploy from branch → main → / (root) → Save**

Your app will be live at `https://YOUR_USERNAME.github.io/ping-pong-tracker/` in ~1 minute.

---

## ELO Details

- Starting ELO: **1000**
- K-factor: **32**
- 🔥 streak shown after 2+ consecutive wins, ❄️ after 2+ consecutive losses
