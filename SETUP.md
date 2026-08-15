# Setup — Ritika's job alerts

This is a fork of [rishabhsabnavis/job-alerts](https://github.com/rishabhsabnavis/job-alerts),
retuned from **AI/ML + software engineering** to **finance, banking, consulting,
and data/business analytics** — the roles that match a Business Analytics, AI &
Finance degree with commercial banking and data analyst internships behind it.

It costs nothing to run: GitHub Actions runs the script on a schedule, and
[ntfy.sh](https://ntfy.sh) delivers the push notification. No server, no API
keys, no subscriptions.

**Right now it tracks 214 companies and matches 387 currently-open roles.**
That 387 is the baseline it will "arm" itself with — after that you only hear
about genuinely new postings.

---

## Part 1 — Get it running (about 15 minutes)

### Step 1: Install ntfy on your phone

1. Install **ntfy** from the iOS App Store or Google Play.
2. Open it, tap **+** to subscribe to a topic.
3. Invent an unguessable topic name — treat it like a password, because anyone
   who knows the string can read your alerts. Something like:

   ```
   ritika-jobs-4kz91mqx7
   ```

4. Subscribe to exactly that string. Nothing else to configure.

### Step 2: Test it locally before touching GitHub

From this folder:

```bash
cd ~/Desktop/job-alerts && python3 poll.py --verify
```

That probes every feed and prints a table of what resolved and how many roles
matched. It sends no notifications and writes no files. A full run takes about
50 seconds. Every row should say `ok` — if a company says `HTTP 404`, its board
moved (see "Fixing a broken company" below).

Now prove the phone path works end to end:

```bash
NTFY_TOPIC=ritika-jobs-4kz91mqx7 python3 poll.py --test-notify
```

Replace the topic with yours. Your phone should buzz within a couple of seconds.
If it doesn't, the problem is the phone subscription, not the script — fix it
here before going further.

### Step 3: Put it in your own GitHub repo

The folder still points at the original author's repo, so repoint it at yours.

1. Create a new **public** repo on GitHub named `job-alerts`. Do not add a
   README, .gitignore, or license — you want it empty.

   *Why public:* GitHub Actions minutes are unlimited on public repos. On a
   private repo you get 2,000 minutes/month, and this schedule burns roughly
   2,900. Nothing personal lives in the repo — your ntfy topic is stored as an
   encrypted GitHub secret, not in the code. If you'd rather keep it private,
   see "Running it private" at the bottom.

2. Point the folder at your repo and push:

```bash
cd ~/Desktop/job-alerts && git remote set-url origin https://github.com/YOUR-USERNAME/job-alerts.git
```

```bash
cd ~/Desktop/job-alerts && git add -A && git commit -m "Retune for finance, banking, consulting and analytics roles" && git push -u origin HEAD
```

### Step 4: Add your topic as a repository secret

In your repo on GitHub: **Settings → Secrets and variables → Actions → New
repository secret**

- Name: `NTFY_TOPIC`
- Value: `ritika-jobs-4kz91mqx7` (your topic)

### Step 5: Turn on Actions and do the first run

1. Go to the **Actions** tab and enable workflows if prompted.
2. Select **job-alerts** in the sidebar → **Run workflow**.

The first run **seeds**: it records all ~387 currently-open roles as "already
seen" and sends you one single *"Job alerts armed"* ping instead of 387
notifications. From then on it runs every 30 minutes and only pings you about
postings that are genuinely new.

That's it. You're done.

---

## Part 2 — What was tuned for you, and how to change it

Everything lives in **`sources.json`**. Two sections matter: `filters` (what
kind of role) and `companies` (where to look).

After **any** edit, run `python3 poll.py --verify` to check nothing broke, then
commit and push — GitHub runs whatever is on your default branch.

### The role filter

A posting is kept only if its title clears three gates:

1. **A role keyword matches** — `analyst`, `investment banking`, `credit`,
   `fp&a`, `wealth management`, `underwriting`, `actuarial`, `business
   intelligence`, `consulting`, `treasury`, `valuation`, `tax`, `audit`, and so
   on. This list is deliberately broad; the level gate does the real narrowing.
2. **It reads as entry-level** — `intern`, `summer analyst`, `new grad`,
   `analyst program`, `rotational`, `class of 2027`, `Analyst I`, etc.
3. **No exclude keyword matches** — this is the one I added for you. Finance
   titles collide badly with technical ones: "analyst" alone pulls in *Security
   Analyst*, *QA Analyst*, *Systems Analyst*, *SOC Analyst*. So
   `exclude_keywords` vetoes those, plus recruiting, clinical, retail and
   commission-sales roles — while deliberately **keeping** sales & trading,
   underwriting, and actuarial work, which a blunt "drop anything with 'sales'"
   rule would have killed.

**To narrow to a specific track** (say you decide you only want IB and credit),
cut `role_keywords` down to just the ones you want:

```json
"role_keywords": ["investment banking", "credit", "capital markets",
                  "financial analyst", "equity research", "m&a"]
```

**To stop seeing a category**, add words to `exclude_keywords`. For example, if
Big 4 audit/tax internships are noise to you:

```json
"tax intern", "audit intern", "assurance"
```

**Graduation-year handling:** you graduate December 2026, so `exclude_grad_years`
is set to `["2024", "2025"]` only — 2026-dated full-time roles are still live
for you and are kept. Once you've signed somewhere for 2027, add `"2026"` and
`"2027"` to that list to silence everything but later classes. Internships are
never dropped by this rule, only full-time postings.

### The company list

214 rows, in three flavors:

**1. Direct feeds (127 companies)** — real-time reads of the company's own
applicant tracking system, the same data their careers page renders. These are
the good ones. Verified working for you: JPMorgan Chase, Wells Fargo, Citi,
Morgan Stanley, Truist, PNC, U.S. Bank, KeyBank, Fifth Third, Huntington, M&T,
Regions, Texas Capital, Frost Bank, USAA, BMO, TD, RBC, Santander, Barclays,
Fidelity, Vanguard, BlackRock, State Street, T. Rowe Price, Invesco, Franklin
Templeton, PIMCO, Blackstone, Houlihan Lokey, Baird, Raymond James, Moelis, S&P
Global, Nasdaq, CME, Morningstar, PwC, Crowe, RSM, BDO, Huron, Travelers,
Allstate, Nationwide, Northwestern Mutual, Prudential, Visa, Mastercard, Capital
One, PayPal, Fiserv, FIS, Global Payments, McKesson, JLL, Sabre, Southwest, AT&T,
Kimberly-Clark, plus ~45 fintech/investment firms on Greenhouse/Ashby/Lever
(Stripe, Ramp, Brex, Robinhood, Point72, AQR, William Blair, Charles River
Associates, Sixth Street…).

For 15 of them I added a **second row pointing at their campus/early-careers
board** — Raymond James, Houlihan Lokey, Blackstone, BlackRock, Baird, Fidelity,
Invesco, KeyBank, Texas Capital, M&T, BMO, RBC, Citi, Visa, Moelis. These
boards are where the 2027 analyst classes actually get posted, and they're
invisible on the main careers page. Raymond James' campus board alone matched 48
roles where its main board matched 3. Duplicate company names are safe — roles
collapse on `(company, title)`, so you never get two pings for one job.

**2. Aggregator-only (87 companies)** — these run bespoke careers software with
no readable feed: **Goldman Sachs, Bank of America, Deloitte, EY, KPMG, Grant
Thornton, Accenture, Charles Schwab, Comerica, American Express, Discover,
McKinsey, Bain, BCG, Texas Instruments, American Airlines, Toyota, ExxonMobil,
PepsiCo, Vistra, CBRE**, and others. Their names are still in the file so that
the aggregator repos can match them, but coverage is partial.

⚠️ **Be honest with yourself about this gap.** Several of these are prime
targets for you — Comerica (you interned there), Goldman's Dallas office, the
Big 4, Schwab in Westlake. **Set up native email job alerts on those companies'
own careers sites**, and treat this tool as covering everything else. That's a
15-minute afternoon of work that closes the biggest hole in the system.

**3. Aggregator feeds** — the SimplifyJobs and vanshb03 new-grad/internship
repos. Worth keeping (they're where the Citadel, Jane Street and Goldman rows
above came from), but know that they're maintained by CS students for CS roles,
so their finance coverage is thin and skewed toward quant. Your direct feeds do
the heavy lifting.

### Adding a company

Most companies are on one of five job boards, and there's a script that finds
which:

```bash
cd ~/Desktop/job-alerts && python3 discover.py "Comerica" "Charles Schwab"
```

It prints a paste-ready row. **Always eyeball the job titles it reports before
trusting it** — slugs collide, and it will confidently hand you a veterinary
clinic when you asked for Boston Consulting Group (it did, to me). Then paste
into the `companies` list in `sources.json`:

```json
{"name": "Cadence Bank", "ats": "greenhouse", "slug": "cadencebank"}
```

For a company on **Workday** (most banks and Fortune 500s), you need the tenant,
data center and site name. Find them from the careers-page URL
`https://TENANT.wdN.myworkdayjobs.com/SITE`, or read them straight off
`https://TENANT.wdN.myworkdayjobs.com/robots.txt`, which lists every site the
tenant runs — including hidden campus ones. Then:

```json
{"name": "Zions Bancorporation", "ats": "workday", "slug": "",
 "tenant": "zions", "site": "External", "dc": "wd5",
 "queries": ["analyst", "intern"], "max": 60}
```

A wrong site name returns HTTP 404; a wrong tenant or data center returns 422.

If a company has no feed at all, add it anyway so aggregator listings match:

```json
{"name": "Comerica", "ats": "none", "slug": "", "aliases": ["Comerica Bank"]}
```

### Fixing a broken company

If `--verify` shows `HTTP 404` for a row, that company changed boards. Re-run
`discover.py` for it, or check its careers URL, and update the row. One broken
company never takes down the run — the script catches it and keeps going.

### Turning down the volume

- **Too many pings at once:** lower `max_pings` (currently 12). Everything past
  that arrives as a single digest ping instead.
- **Too much noise overall:** the fastest wins are trimming `role_keywords` and
  dropping `"ats": "none"` companies — those match by name against every
  aggregator row, so a big employer contributes a lot on its own.
- **Run too slow:** lower `max_workers` (currently 12) if a host starts
  rate-limiting you; raise it if you add another hundred companies.

---

## Things worth knowing

**Matching is title-only.** A role posted as plain "Financial Analyst" with no
level word, on a company's own board, won't match — nothing in the title says
it's entry-level, and reading descriptions roughly triples false positives. The
`Analyst I` / `Analyst 1` pattern is treated as entry-level to catch the most
common case.

**Accenture is deliberately off.** Its Workday board returns no location on any
posting, so the US filter can't work and it floods you with European
internships. To re-enable it, delete its `"ats": "none"` row and add:

```json
{"name": "Accenture", "ats": "workday", "slug": "", "tenant": "accenture",
 "site": "accenturecareers", "dc": "wd103", "queries": ["analyst", "intern"],
 "max": 80}
```

**Running it private.** If you'd rather the repo not be public, edit
`.github/workflows/poll.yml` and change the cron from `*/30 * * * *` to
`0 * * * *` (hourly). That lands around 1,400 minutes/month, inside the free
2,000. You'll hear about roles within an hour instead of half an hour.

**GitHub's scheduler runs late.** Cron on Actions is best-effort and can drift
10–15 minutes under load. It's a fine tradeoff for free.

**Your ntfy topic is a password.** Anyone with the string can read your alerts.
Don't paste it into screenshots, and if it leaks, just pick a new one: change
the phone subscription and the `NTFY_TOPIC` secret.

**Check `TRACKING.md`** any time to see exactly what's being watched, which
endpoint each company is read from, and every role currently matching. Regenerate
it with:

```bash
cd ~/Desktop/job-alerts && python3 poll.py --report
```
