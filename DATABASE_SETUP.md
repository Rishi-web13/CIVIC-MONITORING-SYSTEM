# Civic Authority - Database Setup Guide

## Overview
Your app now has **persistent cloud storage** using Supabase (a PostgreSQL database). Data is automatically saved and will persist even if users close their browser tabs.

### New Features Implemented
✅ **Real Database** - All issues persist in Supabase, visible to all users  
✅ **1MB File Limit** - Image uploads capped at 1MB  
✅ **Appeal System** - Users can appeal unresolved issues after 30 days with "Appeal Again" button  
✅ **Appeal Tracking** - Admin can see number of appeals per issue in exports

---

## Step 1: Create a Supabase Project

1. Go to [supabase.com](https://supabase.com)
2. Sign up (free account)
3. Create a new project:
   - Click **"New Project"**
   - Name: `civic-authority`
   - Database password: (save this somewhere safe)
   - Region: Choose closest to your users
   - Click **"Create new project"** (takes ~2 mins)

---

## Step 2: Get Your API Keys

1. In Supabase dashboard, go to **Settings** → **API**
2. Copy these two values:
   - **Project URL** (starts with `https://...supabase.co`)
   - **anon public key** (starts with `eyJ...`)

---

## Step 3: Create Database Tables

1. In Supabase, go to **SQL Editor** → click **"New Query"**
2. Copy-paste this SQL:

```sql
CREATE TABLE IF NOT EXISTS issues (
  id TEXT PRIMARY KEY,
  category TEXT NOT NULL,
  address TEXT NOT NULL,
  description TEXT NOT NULL,
  status TEXT DEFAULT 'Pending',
  priority TEXT DEFAULT 'Normal',
  name TEXT DEFAULT 'Anonymous',
  note TEXT DEFAULT '',
  image LONGTEXT,
  timestamp BIGINT NOT NULL,
  appeals INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_status ON issues(status);
CREATE INDEX idx_timestamp ON issues(timestamp DESC);
```

3. Click **"Run"**
4. You should see: `✓ Success. See query results below.`

---

## Step 4: Add Your API Keys to the App

Open `index.html` and find this line (around line 762):

```javascript
const SUPABASE_URL = 'https://YOUR_PROJECT.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR_ANON_KEY';
```

Replace:
- `https://YOUR_PROJECT.supabase.co` with your **Project URL**
- `YOUR_ANON_KEY` with your **anon public key**

Example:
```javascript
const SUPABASE_URL = 'https://abc123xyz.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

---

## Step 5: Test It Works

1. Open `index.html` in your browser
2. Go to **Report Issue** tab
3. Fill out a test report and submit
4. Go back to Supabase dashboard → **Table Editor**
5. Click the **issues** table
6. You should see your report appear!

✅ **If you see it in Supabase, the database is working!**

---

## How It Works

### Data Flow
1. **User submits report** → Saved to Supabase database + localStorage backup
2. **Browser closed** → Data still exists in Supabase cloud
3. **New browser/device** → Opens `index.html` → Loads from Supabase (not localStorage)
4. **No internet?** → Falls back to localStorage (data syncs when online)

### Appeal System
- Issues not resolved after **30+ days** show **"Appeal Again"** button  
- Clicking it increments `appeals` count and re-timestamps the issue
- Admin can see total appeals in CSV export
- Each appeal notifies authorities to review again

### File Upload Limit
- Maximum **1MB** per image
- Enforced in `handleFileUpload()` function

### Admin Features
- View all appeals in **CSV export** (new `appeals` column)
- Track issue response times
- Filter by status, category, or search

---

## Troubleshooting

### "Database not configured" message in console
- Check if SUPABASE_URL and SUPABASE_ANON_KEY are set correctly
- Check browser console (F12) for errors
- App falls back to localStorage automatically

### Data not appearing in Supabase
1. Check browser console (F12) for red errors
2. Verify credentials are copied exactly (no extra spaces)
3. Verify `issues` table exists in Supabase
4. Check Supabase dashboard → **Statistics** to see if data is being saved

### Network errors
- The app gracefully handles offline scenarios
- Data saves locally (localStorage) and syncs when back online
- Users can always access their reports

---

## Security Notes

⚠️ **The `anon` key is for public read/write access**
- Anyone with this key can access/modify issues
- For production with user authentication, use Row-Level Security (RLS)
- Set RLS policies in Supabase dashboard → **Authentication**

### Basic RLS Setup (Optional)
In Supabase SQL Editor, run:
```sql
ALTER TABLE issues ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Anyone can read issues"
  ON issues FOR SELECT
  USING (true);

CREATE POLICY "Anyone can insert issues"
  ON issues FOR INSERT
  WITH CHECK (true);
```

---

## Deployment Options

### Option 1: GitHub Pages (Free, Static)
1. Create GitHub repo
2. Push your files
3. Settings → Pages → Deploy from `main` branch
4. Live at: `yourusername.github.io/civic-authority`

### Option 2: Netlify (Free, Simple)
1. Drop your `index.html` on netlify.com
2. Live in seconds

### Option 3: Your Own Server
- Works with any web host (Apache, Nginx, Node.js, etc.)
- Just serve the `index.html` file

---

## Backup & Export

Admins can download all issues as CSV:
1. Admin Login → Dashboard
2. Click **"Export CSV"** button
3. Includes all data + appeal counts

---

## Support

If you need help:
1. Check browser console (F12) → Console tab for errors
2. Verify Supabase credentials
3. Check [Supabase docs](https://supabase.com/docs) for API help
4. Check [Civic Authority code comments](index.html) for app-specific logic
