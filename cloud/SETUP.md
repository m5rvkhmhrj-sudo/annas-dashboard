# Putting Anna's Dashboard on her phone and computer (with sync)

This folder is the deployable, cloud-synced version. Follow these steps once.
Everything here is free.

You do steps 1-3 (accounts). I can finish wiring once you paste the two keys.

---

## Step 1 - Create the Supabase project (the database + login)

1. Go to supabase.com, sign up, click **New project**.
2. Give it a name, set a database password (save it somewhere), pick a region near you, create it. Wait ~2 minutes for it to finish.
3. In the project, open **Project Settings > API**. Copy two values:
   - **Project URL** (looks like `https://abcd1234.supabase.co`)
   - **anon public** key (a long string, safe to put in the page)
4. Open `index.html` in this folder and paste them into the CLOUD CONFIG block near the top of the script:
   ```js
   const SUPABASE_URL='https://YOURPROJECT.supabase.co';
   const SUPABASE_ANON_KEY='your-anon-public-key';
   ```

## Step 2 - Create the data table

1. In Supabase, open **SQL Editor > New query**, paste this, and click **Run**:
   ```sql
   create table public.dashboards (
     user_id uuid primary key references auth.users on delete cascade,
     data jsonb not null default '{}',
     updated_at timestamptz default now()
   );
   alter table public.dashboards enable row level security;
   create policy "Users manage own dashboard" on public.dashboards
     for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
   ```
   This makes one private row per user. The row-level-security policy means each
   person can only ever read or write their own data.

## Step 3 - Email login settings

1. Open **Authentication > Providers** and confirm **Email** is enabled (it is by default).
2. Easiest for personal use: open **Authentication > Sign In / Providers** (or
   **Settings**) and turn **off** "Confirm email." Then sign-up logs her in
   right away with no confirmation email. Leave it on if you prefer the email step.

## Step 4 - Deploy to Vercel (gives you a web address)

Option A, no command line:
1. Go to vercel.com, sign up.
2. **Add New > Project**, and either connect a GitHub repo containing this `cloud`
   folder, or use the Vercel CLI below.

Option B, command line (from inside this `cloud` folder):
```bash
npx vercel        # first run: log in, accept defaults
npx vercel --prod # publishes; prints your live URL
```
You will get a URL like `https://anna-dashboard.vercel.app`.

## Step 5 - Install it on each device

1. Open the Vercel URL on her **laptop** browser and on her **phone** browser.
2. Create the account once (any device), then sign in on the other.
3. Add to home screen / install:
   - iPhone Safari: Share > **Add to Home Screen**.
   - Android Chrome: menu > **Install app** / **Add to Home screen**.
   - Desktop Chrome/Edge: the install icon in the address bar.
4. Now anything she records on one device shows up on the other after a moment.
   The footer shows a small **synced** indicator. If she is offline, it saves on
   the device and syncs when she is back online.

---

## Notes
- The anon key is meant to be public; the row-level-security policy is what keeps
  data private, so do not skip Step 2.
- The app still works offline once installed; changes sync when back online.
- To upgrade the app icon to crisp PNGs later, replace `icon.svg` references in
  `manifest.json` and the `<link rel="apple-touch-icon">` with 192px and 512px PNGs.
