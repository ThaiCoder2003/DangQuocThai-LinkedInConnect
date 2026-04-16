# Setup Guide for cron-job.org (Updated: Cookie-First Strategy)

## Overview
This script runs automatically on cron-job.org to send LinkedIn connection requests on a schedule.

**Smart Login Strategy:**
- ✅ **First run**: Fresh login with credentials, saves cookies
- ✅ **Subsequent runs**: Uses stored cookies (no fresh login = safer!)
- ✅ **Fallback**: If cookies expire, does fresh login again
- ⚠️ **Result**: Fewer logins = Less bot detection

## Prerequisites
1. Repository must be **public** on GitHub (cron-job.org can only access public repos)
2. `data/connect_sheet.csv` with columns: `LinkedIn`, `Email`, `Status`
3. LinkedIn account credentials stored as environment variables

## Initial Setup (Manual - Do This Once)

### Step 1: Generate Initial Cookies

Run locally to authenticate and save cookies:

```powershell
$env:LINKEDIN_USERNAME = "your-email@gmail.com"
$env:LINKEDIN_PASSWORD = "your-password"
python main.py
```

This creates:
- `cookies.pkl` - Session cookies
- `credentials.pkl` - Encrypted credentials
- `data/connect_sheet.csv` - Updated with connection status

### Step 2: Encode Cookies for Environment Variable

Run `read_pkl.py` to get base64-encoded cookies:

```powershell
python read_pkl.py
```

Copy the long string output.

### Step 3: Push to GitHub

Commit and push (base64 cookies in env var):
```bash
git add cookies.pkl credentials.pkl data/connect_sheet.csv
git commit -m "Initial cookies and data setup"
git push
```

---

## Setup on cron-job.org

### Step 1: Create Account
- Go to https://cron-job.org
- Sign up and login

### Step 2: Create New Cron Job

Click **"Create Cron Job"** with these settings:

**Basic Info:**
- **Title**: `LinkedIn Auto Connect`

**Execution:**

Use this command for **Python-based execution**:

```bash
cd /tmp && rm -rf DangQuocThai-LinkedInConnect && \
git clone https://github.com/YOUR_USERNAME/DangQuocThai-LinkedInConnect.git && \
cd DangQuocThai-LinkedInConnect && \
pip install -r requirements.txt && \
export LINKEDIN_USERNAME="your-email@gmail.com" && \
export LINKEDIN_PASSWORD="your-password" && \
export LINKEDIN_COOKIES="<PASTE_BASE64_STRING_HERE>" && \
python main.py
```

**Replace:**
- `YOUR_USERNAME` - Your GitHub username
- `your-email@gmail.com` - Your LinkedIn email
- `your-password` - Your LinkedIn password
- `<PASTE_BASE64_STRING_HERE>` - The base64 string from `read_pkl.py`

### Step 3: Schedule

Choose your cron expression:

**Daily at 9 AM UTC**:
```
0 9 * * *
```

**Every 12 hours**:
```
0 */12 * * *
```

**Every 3 days at 9 AM**:
```
0 9 */3 * *
```

### Step 4: Save & Test

1. Click **"Save"**
2. Click **"Execute now"** to test
3. Check logs for any errors (look for `[CRON]` or `[SESSION]` prefixes)

---

## How It Works

### First Run:
```
[SESSION] 🍪 Attempting to restore session using cookies...
[SESSION] ❌ Cookies expired or invalid
[CRON] 🔐 Cookies invalid/missing. Doing fresh login...
🔐 Starting LinkedIn login process...
✅ SUCCESS: ĐĂNG NHẬP THÀNH CÔNG!
🍪 Cookies SAVED! (for next run)
```

### Subsequent Runs (Ideal):
```
[SESSION] 🍪 Attempting to restore session using cookies...
[SESSION] ✅ Session restored successfully with cookies!
[CRON] ✅ Using cached session - no fresh login needed
[CRON] 👤 Visiting profile: https://linkedin.com/in/xxx
[CRON] ✅ Status: Pending
```

---

## Handling 2FA (Two-Factor Authentication)

### Option 1: Disable 2FA (Easier for Bots)
- Login to LinkedIn
- Settings → Account & privacy → Password and security
- Disable "Two-step verification" temporarily during bot runs

### Option 2: Provide 2FA Code via Environment Variable
If you want to keep 2FA enabled:

Add to your cron command:
```bash
export LINKEDIN_2FA_CODE="123456"
```

(Get the code from your email/authenticator app before scheduling)

### Option 3: First Manual Login
- Do the initial setup on your local machine (handles 2FA manually)
- Cookies will be saved and reused
- Subsequent cron runs skip 2FA since they use existing session

---

## Monitoring & Debugging

### Check Execution Logs
1. Go to your cron job in cron-job.org
2. Click the job → **Execution log**
3. Look for:
   - `[CRON]` - Main execution info
   - `[SESSION]` - Cookie/session restoration
   - `[2FA]` - Two-factor auth issues

### Common Issues

**"Cookies not found"**
- Run locally once to generate them
- Commit cookies.pkl to repo
- Encode and add to env var

**"2FA code required"**
- Either disable 2FA or provide code via `LINKEDIN_2FA_CODE`
- First local run handles it, cookies saved for future runs

**"Chrome not found"**
- Shouldn't happen - `webdriver-manager` auto-installs
- Check your pip installation

**"Still detecting as bot"**
- Increase delay: Change `time.sleep(random.randint(2, 8))` in main.py
- Reduce connection limit: Change `limit = 15` to `limit = 10`
- Run less frequently (every 2-3 days instead of daily)

---

## Alternative: GitHub Actions

If cron-job.org doesn't work for you, use GitHub Actions instead:

1. Secrets already configured in `.github/workflows/linkedin-connect.yml`
2. Go to repo → Settings → Secrets → Add:
   - `LINKEDIN_USERNAME`
   - `LINKEDIN_PASSWORD`
3. Workflow runs automatically every day

---

## Important Notes

⚠️ **Security**: Never share your base64 cookie string publicly  
⚠️ **Rate Limiting**: LinkedIn typically allows 15-20 connections/day  
⚠️ **IP Banning**: Too much bot activity = IP ban (use different times/frequencies)  
⚠️ **ToS**: This is for personal automatio only - check LinkedIn's ToS  
⚠️ **Cookies Expire**: Every 2-8 weeks, cookies expire (you'll need to run locally to refresh)

