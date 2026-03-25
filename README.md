# Receipt Tracker PWA

Snap receipts → Claude reads them → expenses logged to Supabase.

## Quick Start

### 1. Create a GitHub repo and deploy

```bash
# Create a new repo on GitHub called "receipt-tracker"
# Then push this folder:
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/receipt-tracker.git
git push -u origin main
```

Go to your repo → **Settings** → **Pages** → set source to "Deploy from a branch" → select `main` / `/ (root)` → Save.

Your app will be live at: `https://YOUR_USERNAME.github.io/receipt-tracker/`

### 2. Set up Supabase (free tier works fine)

1. Go to [supabase.com](https://supabase.com) and create a new project
2. Go to **SQL Editor** and run this:

```sql
-- Transactions table
CREATE TABLE transactions (
  id TEXT PRIMARY KEY DEFAULT gen_random_uuid()::text,
  type TEXT NOT NULL CHECK (type IN ('income', 'expense')),
  date DATE NOT NULL,
  amount NUMERIC(10,2) NOT NULL,
  source_or_store TEXT NOT NULL,
  category TEXT NOT NULL,
  notes TEXT DEFAULT '',
  line_items JSONB,
  thumbnail TEXT,
  is_recurring BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Index for fast date queries
CREATE INDEX idx_transactions_date ON transactions(date DESC);

-- Enable Row Level Security
ALTER TABLE transactions ENABLE ROW LEVEL SECURITY;

-- Allow all operations for anon key (this is a personal app)
CREATE POLICY "Allow all for anon" ON transactions
  FOR ALL USING (true) WITH CHECK (true);
```

3. Go to **Settings** → **API** and copy:
   - **Project URL** (e.g., `https://xxxxx.supabase.co`)
   - **anon public** key (starts with `eyJ...`)

### 3. Get an Anthropic API key

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Create an API key
3. Add some credits ($5-10 is plenty to start — each receipt scan costs ~$0.01)

### 4. Configure the app

1. Open your deployed app URL on your phone
2. Tap **Setup** in the bottom nav
3. Paste in your three keys (Anthropic API key, Supabase URL, Supabase anon key)
4. Tap **Save Settings**
5. Set up your recurring income rule (e.g., weekly paycheck)

### 5. Add to home screen

**iPhone:** Open the URL in Safari → Share button → "Add to Home Screen"
**Android:** Open in Chrome → three-dot menu → "Add to Home Screen"

## How it works

- **Scan**: Opens your camera, snaps a receipt photo, sends it to Claude Sonnet which extracts store, date, total, line items, and suggests a category. You confirm with one tap.
- **Add**: Manual entry for income or non-receipt expenses.
- **Log**: Scrollable list of all transactions, grouped by month. Tap to expand and see details.
- **Stats**: Monthly income vs expenses with category breakdown. Yearly view with month-by-month bars.
- **Setup**: API keys, recurring income rules, CSV export.

## Data storage

- Primary: Supabase (your own instance, you own the data)
- Backup: localStorage (works offline, syncs when Supabase is available)
- Receipt thumbnails: stored as base64 in the transaction record

## Cost estimate

- **Supabase**: Free tier handles thousands of transactions
- **Claude API**: ~$0.01-0.02 per receipt scan (Sonnet model)
- **GitHub Pages**: Free
- **Total**: Maybe $1-2/month if you scan a receipt every day

## Files

- `index.html` — The entire app (single file, no build step)
- `manifest.json` — PWA manifest for home screen install
- `icon-192.png` / `icon-512.png` — App icons
- `README.md` — This file
