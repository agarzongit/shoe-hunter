# Daily Shoe Hunter — Cloud Setup (Free)

Runs every morning in GitHub's cloud. Emails you when there are hits.
No machine of yours needs to be on. Total cost: $0.

## What it does

Each morning at 7:00 AM ET it searches Google Shopping for these shoes in
size 11.5 Wide (2E), filters to anything ≤ $120, and emails you a summary
with direct links. Anything ≤ $100 is flagged green as a deal.

| Brand  | Model            |
|--------|------------------|
| Brooks | Adrenaline GTS   |
| Brooks | Ghost            |
| Brooks | Beast            |
| Brooks | Glycerin GTS     |
| Hoka   | Gaviota          |
| Hoka   | Arahi            |
| Asics  | Gel-Kayano       |

It also commits a dated Excel file to your repo so you have a running history
of every hit you've ever found.

## One-time setup — ~15 minutes

### Step 1: Make a GitHub account (skip if you have one)

Go to <https://github.com/signup>. Free tier is fine.

### Step 2: Create the repo

1. Click the **+** icon top-right → **New repository**
2. Name it `shoe-hunter` (or whatever)
3. **Private** (recommended — keeps your email address out of public view)
4. Don't check any of the "initialize with README" boxes
5. Click **Create repository**

### Step 3: Upload the files

The easiest way without using the command line:

1. On your new empty repo page, click **uploading an existing file**
2. Drag the entire contents of this folder (everything in the zip):
   - `shoe_hunter.py`
   - `requirements.txt`
   - `.gitignore`
   - `.github/workflows/daily.yml`  ← IMPORTANT, this is what schedules the job
   - `README.md`
3. Scroll down and click **Commit changes**

**Note about the `.github` folder:** GitHub's web uploader sometimes drops dotfolders.
If after uploading you don't see a `.github/workflows/daily.yml` file in your repo,
you'll need to create it manually:
- Click **Add file** → **Create new file**
- In the filename box type exactly: `.github/workflows/daily.yml`
- Paste the contents of `daily.yml` from the zip
- Commit

### Step 4: Create a Gmail App Password

You need this so the script can send you emails. **Do NOT use your regular
Gmail password** — Google blocks that. Use an App Password.

1. Make sure 2-factor auth is ON for your Google account:
   <https://myaccount.google.com/security>
2. Go to <https://myaccount.google.com/apppasswords>
3. App name: "Shoe Hunter" → **Create**
4. Google shows you a 16-character password like `abcd efgh ijkl mnop`
5. **Copy it now** — Google only shows it once. (No spaces when you paste it later.)

If you don't use Gmail, the script also works with Outlook, iCloud, etc. —
look up "[your email provider] SMTP settings" and adjust `SMTP_SERVER` /
`SMTP_PORT` in `.github/workflows/daily.yml`.

### Step 5: Add your credentials as repo Secrets

1. In your repo, click **Settings** (top right of the repo page)
2. Left sidebar: **Secrets and variables** → **Actions**
3. Click **New repository secret**, and add these four one at a time:

| Name         | Value                                              |
|--------------|----------------------------------------------------|
| `EMAIL_FROM` | your.email@gmail.com                              |
| `EMAIL_TO`   | your.email@gmail.com  (same address is fine)      |
| `EMAIL_USER` | your.email@gmail.com                              |
| `EMAIL_PASS` | the 16-char App Password from Step 4 (no spaces)  |

### Step 6: Test it

1. Click the **Actions** tab at the top of the repo
2. If you see a "Workflows aren't being run on this repository" banner,
   click **I understand my workflows, go ahead and enable them**
3. Left sidebar: click **Daily Shoe Hunter**
4. Click **Run workflow** (right side) → **Run workflow**
5. Wait ~2 minutes. Refresh. You should see a green checkmark.
6. Check your email — you should have a "Shoe Hunter — N hit(s)" message.

If anything failed, click the failed run to see the error log.

### Step 7: You're done

It now runs automatically at 7:00 AM ET every day. You'll get an email
each morning whether or not there were hits.

To **stop** it: Settings → Actions → General → "Disable Actions for this repository"

To **change the time**: edit `.github/workflows/daily.yml`. The cron string
`'0 12 * * *'` means 12:00 UTC = 7am ET. Change the first two numbers
(`minute hour`) — note GitHub uses UTC, so subtract 4 for EDT (summer) or
5 for EST (winter) from your desired local time.

## Customizing

- **Add/remove shoes**: edit `TARGET_SHOES` near the top of `shoe_hunter.py`
- **Change price cap**: edit `PRICE_CAP = 120.00`
- **Change deal threshold (green highlight)**: edit `GOOD_DEAL_THRESHOLD = 100.00`

Push the edits to the repo and the next scheduled run picks them up.

## Troubleshooting

**"Email not arriving"**
- Check spam folder first
- Verify App Password is correct (no spaces, no quotes)
- Verify 2FA is enabled on your Google account
- Look at the Actions run logs — if `send_email` failed, the error is there

**"Google blocked the search (CAPTCHA page)"**
- This rarely happens with one run/day, but if it does, the run will show
  zero hits with no error. Just wait — the next day's run usually works.
  If it persists, switch to SerpAPI free tier (100 searches/month, plenty
  for daily use). Ping me if you want help wiring that up.

**"Zero hits, every day, for a week"**
- Genuinely possible — stability shoes in 11.5 Wide under $120 are rare.
- Try raising the cap to $140 in `shoe_hunter.py` to confirm the search
  itself works.

## What this does NOT do

- Does NOT buy shoes for you. Email gives you links — you click and purchase.
- Does NOT guarantee 11.5 Wide is in stock at the listed price. Always
  verify on the retailer page before buying. Rows marked "Verify" in the
  Excel are listings where the size couldn't be confirmed from the snippet.
- Does NOT log into Amazon, Brooks, or Zappos directly. It uses Google
  Shopping, which aggregates them.
