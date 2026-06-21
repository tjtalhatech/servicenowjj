# ServiceNow Leads

Auto-collects ServiceNow job postings from Adzuna + JSearch twice a day and
shows them on a simple dashboard (GitHub Pages). Free, no server needed.

## Setup (10 minutes)

### 1. Push this folder to a new GitHub repo
```bash
cd servicenow-jobs
git init
git add .
git commit -m "Initial setup"
git branch -M main
git remote add origin https://github.com/<your-username>/servicenow-jobs.git
git push -u origin main
```

### 2. Get API keys (use either or both)

**Adzuna** (recommended, very generous free tier):
1. Sign up at https://developer.adzuna.com/
2. Create an app → you'll get an `app_id` and `app_key`

**JSearch / RapidAPI** (covers LinkedIn, Indeed, Glassdoor, ZipRecruiter):
1. Sign up at https://rapidapi.com/letscrape-6bRBa3QguO5/api/jsearch
2. Subscribe to the free tier → copy your `X-RapidAPI-Key`

### 3. Add keys as GitHub Secrets
In your repo: **Settings → Secrets and variables → Actions → New repository secret**

Add:
- `ADZUNA_APP_ID`
- `ADZUNA_APP_KEY`
- `RAPIDAPI_KEY`

(You can add just one provider's keys to start — the script skips whichever is missing.)

### 4. Enable GitHub Pages
**Settings → Pages → Source → GitHub Actions**

### 5. Run it
Go to **Actions → Fetch ServiceNow Jobs → Run workflow** to trigger it manually
the first time. After that it runs automatically at 08:00 and 20:00 UTC.

Your dashboard will be live at:
`https://<your-username>.github.io/servicenow-jobs/`

## Sources used

| Source | Coverage | Cost | Notes |
|---|---|---|---|
| Adzuna | Aggregates Indeed, LinkedIn, company sites, more | Free tier (generous) | Queried for 5 specific role variants (Developer, Admin, Architect, Consultant, Implementation) across 5 countries by default |
| JSearch | Aggregates via Google for Jobs (LinkedIn, Indeed, Glassdoor, ZipRecruiter) | Free tier ~200 req/month | Only runs on the **08:00 UTC** schedule to conserve quota |
| RemoteOK | Remote jobs | Free, no key needed | Filters for "servicenow" in title/tags/description |
| Arbeitnow | Global jobs, strong in Europe | Free, no key needed | Filters for "servicenow" in title/tags/description |

## Customizing

- **Change schedule**: edit the two `cron` lines in
  `.github/workflows/fetch-jobs.yml` (uses UTC time)
- **Change which countries Adzuna searches**: set a repo **variable**
  (Settings → Secrets and variables → Actions → Variables tab) called
  `ADZUNA_COUNTRIES`, e.g. `us,gb,in,pk,ae` (Adzuna doesn't cover every
  country — check https://developer.adzuna.com/docs/countries)
- **Change role keywords searched**: edit `ROLE_VARIANTS` in `fetch_jobs.py`
- **Adjust JSearch quota usage**: change the `SKIP_JSEARCH` condition in the
  workflow if you want it to run on both schedules, or neither
- **Run locally** to test before pushing:
  ```bash
  export ADZUNA_APP_ID=xxx ADZUNA_APP_KEY=xxx RAPIDAPI_KEY=xxx
  python fetch_jobs.py
  open docs/index.html
  ```

## How it works

```
GitHub Actions (cron, 2x/day)
  → fetch_jobs.py calls Adzuna + JSearch APIs
  → de-dupes, merges with history, saves docs/data/jobs.json
  → commits the file + deploys docs/ to GitHub Pages
  → dashboard (docs/index.html) reads jobs.json and renders cards
```
