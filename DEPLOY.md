# Deployment Guide — Boiler Gauge Logger
### Tyre Pyrolysis Plant | Estimated setup time: 25–30 minutes

---

## What you'll set up
- **Worker app** — mobile-friendly page workers use to take photos and log readings
- **Admin dashboard** — manager view with all readings, filters, and Excel export
- **Supabase** — free cloud database (stores all your data)
- **Vercel** — free web hosting (your workers access via a URL)
- **Anthropic API** — the AI that reads gauge images

---

## Step 1 — Set up Supabase (database)

1. Go to **https://supabase.com** and sign up for free
2. Click **"New project"** — give it a name like `pyrolysis-log`
3. Choose a region closest to Malaysia (e.g. Singapore)
4. Wait ~2 minutes for it to finish setting up
5. Go to **SQL Editor** (left sidebar) and paste this, then click Run:

```sql
CREATE TABLE gauge_readings (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  operator text,
  boiler_unit text,
  shift text,
  reading_date date,
  reading_time time,
  temperature_c numeric,
  pressure_bar numeric,
  all_readings jsonb,
  ai_confidence integer,
  image_quality text,
  notes text,
  status text DEFAULT 'normal',
  created_at timestamptz DEFAULT now()
);

-- Allow the app to read and write (no login required)
ALTER TABLE gauge_readings ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Allow all" ON gauge_readings FOR ALL USING (true) WITH CHECK (true);
```

6. Go to **Project Settings → API**
7. Copy these two values — you'll need them shortly:
   - **Project URL** (looks like `https://xxxx.supabase.co`)
   - **anon/public key** (long string starting with `eyJ...`)

---

## Step 2 — Get your Anthropic API key

1. Go to **https://console.anthropic.com**
2. Sign in or create an account
3. Go to **API Keys** and click **Create Key**
4. Copy the key (starts with `sk-ant-...`) — save it somewhere safe

---

## Step 3 — Deploy to Vercel

1. Go to **https://vercel.com** and sign up (free) — use GitHub login if possible
2. Click **"Add New Project"**
3. Choose **"Upload"** (drag and drop your project folder) OR connect via GitHub:
   - If using GitHub: push this folder to a new GitHub repo, then import it on Vercel
   - If uploading: zip the entire `pyrolysis-app` folder and upload it
4. Before deploying, add these **Environment Variables** in Vercel:
   - Click **"Environment Variables"** in the deploy screen
   - Add: `ANTHROPIC_API_KEY` = your key from Step 2
5. Click **Deploy**
6. Vercel will give you a URL like `https://pyrolysis-log.vercel.app`

---

## Step 4 — Update the config in your HTML files

Open `index.html` and `admin.html` and replace these two placeholder values:

```js
const SUPABASE_URL  = 'YOUR_SUPABASE_URL';    // → paste your Project URL
const SUPABASE_ANON = 'YOUR_SUPABASE_ANON_KEY'; // → paste your anon key
```

Then redeploy (push to GitHub or re-upload to Vercel).

---

## Step 5 — Share with your team

- **Workers** open: `https://your-app.vercel.app/`
- **Admin dashboard**: `https://your-app.vercel.app/admin`

You can also set a custom domain in Vercel settings if you want something like `log.yourcompany.com`.

---

## Tips

- On mobile (Android/iPhone), workers can add the URL to their home screen for an app-like experience:
  - **iPhone**: Safari → Share → "Add to Home Screen"
  - **Android**: Chrome → menu → "Add to Home Screen"
- The admin dashboard auto-refreshes every 60 seconds
- Excel export respects whatever filters you've applied
- AI confidence above 80% = reliable reading; below 50% = worker should double-check

---

## Costs (all free to start)

| Service | Free tier |
|---------|-----------|
| Vercel | 100GB bandwidth/month, unlimited deploys |
| Supabase | 500MB database, 2GB bandwidth/month |
| Anthropic API | Pay per use — ~$0.003 per gauge photo read |

For 5 workers logging 4 readings/day = ~600 readings/month ≈ **~$1.80/month** in API costs.

---

## Need help?

If you get stuck on any step, share the error message and I can help you fix it.
