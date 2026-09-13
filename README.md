# 💼 LinkedIn Jobs Scraper — Bulk Job Data, FREE TO USE

Scrape LinkedIn job postings in bulk and get back a clean, spreadsheet-ready dataset: job title, company, the **exact** posting date, location, published salary range, seniority, employment type, industry, applicant count, the full description, and — when the posting names one — **the recruiter who posted it, with their profile link**.

No login. No cookies. No session tokens. No per-result fee.

---

## ✨ Why this one

Most LinkedIn job scrapers charge per result and still hand back columns that are empty or wrong. This one is free, and it fills the columns the others leave blank.

| | This Actor | Typical paid Actor |
|---|---|---|
| **Price** | **Free** — you pay only your own Apify platform usage | $0.40 – $2.00 per 1,000 jobs |
| **Salary** | Parsed into `salary_min`, `salary_max`, currency and pay period | Usually empty |
| **Recruiter contact** | Name, job title and profile URL, included | Empty, or sold as a paid add-on |
| **Posting date** | The posting's **exact** date — `2026-09-03` | Often reconstructed from "5 months ago" and rounded |
| **Apply link** | Reported as `null`, honestly — LinkedIn shows it only to signed-in users | Sometimes filled with the LinkedIn URL, which is not an apply link |
| **Applicant count** | Exact number, with a flag when LinkedIn only gave a range | Missing, or a bucket reported as exact |
| **Input** | Paste a URL **or** describe the search in plain words | Usually one or the other |
| **Filters LinkedIn ignores** | Applied to the real data, so they actually work | Passed upstream and silently dropped |
| **Minimum run size** | 1 job | One competitor forces a 150-job minimum |

---

## 🔗 Three ways to tell it what you want

**1 — Paste a URL.** Copy any LinkedIn jobs URL straight out of your browser. Every filter already in it is kept.

```
https://www.linkedin.com/jobs/search/?keywords=Software%20Engineer&location=London&geoId=100495523&f_TPR=r2592000
```

**2 — Paste a single job link.** Get one posting in full.

```
https://www.linkedin.com/jobs/view/4466429011
```

**3 — Just describe it.** No URLs, no ID lookups:

```json
{
  "keywords": ["Software Engineer", "Backend Engineer"],
  "locations": ["London", "Berlin"],
  "datePosted": "pastWeek"
}
```

Type a location in plain words and the Actor resolves it to the right place on LinkedIn for you. Same for companies — `"companyNames": ["Stripe"]` works; you never have to hunt down a numeric ID.

You can mix all three in one run. Everything is merged and de-duplicated, and each row records the search that delivered it. When several of your searches match the same job, the full overlap is recorded in the run summary.

---

## 🎯 Why scrape LinkedIn jobs?

- **Recruiting & sourcing** — track which companies are hiring, for what, and how fast their roles fill.
- **Sales prospecting** — a company hiring five Salesforce admins is a company buying Salesforce tooling. Hiring is the earliest public buying signal there is.
- **Market & salary research** — build real pay benchmarks from published ranges, by role, level and city.
- **Job boards & aggregators** — keep a niche board stocked without maintaining scrapers.
- **Job seekers** — monitor new postings in your niche and reach the named recruiter before the queue builds.
- **Competitive intelligence** — watch a competitor's headcount plans before they announce anything.

---

## 📦 What data does it extract?

### 💼 The Job
- `job_id` — LinkedIn's permanent ID for the posting; the key to join on across runs
- `title` — the role as the employer titled it
- `job_url` — the public posting, with per-visit tracking stripped so it is stable run to run
- `location` — where the role is based
- `posted_date` — the **exact** calendar date, not a rounded "3 weeks ago"
- `posted_relative` — how LinkedIn phrased its age
- `is_recently_posted` — whether LinkedIn still flags it as new

### 🏢 The Company
- `company_name`, `company_id`, `company_url`, `company_logo`

### 🏷️ Classification
- `employment_type` — Full-time, Contract, Internship…
- `seniority_level` — Entry level, Associate, Mid-Senior, Director, Executive
- `job_function`, `industries`
- `workplace_type` — Remote / Hybrid / On-site, read from what the employer wrote

### 💰 Pay
- `salary` — the range exactly as published
- `salary_min`, `salary_max` — as numbers, ready to sort and average
- `salary_currency`, `salary_period` — so an hourly rate is never averaged against an annual salary
- `salary_source`, `salary_note` — who provided the range, and their caveat

### 🧑 The Recruiter
- `poster_name`, `poster_title`, `poster_profile_url`, `poster_photo`

### 📊 Competition
- `applicants_count` — how many have applied
- `applicants_is_estimate` — `true` when LinkedIn gave a bucket rather than a number
- `benefits` — badges such as "Actively Hiring" and "Be an early applicant"
- `apply_type` — `EASY_APPLY` or `EXTERNAL`
- `is_easy_apply`

### 📝 The Description
- `description_text` — full text, readable, with list bullets and paragraphs intact
- `description_html` — the same with the employer's original formatting

### 🧾 Run Metadata
- `search`, `found_by` — which of your searches delivered it
- `detail_scraped`, `detail_error`, `scraped_at`, `success`, `error_type`, `source_url`

---

## ⚙️ How to use it

### Where to search

#### `searchUrls` (Array)
LinkedIn job search URLs or single job links, pasted from the browser. Filters already in the URL are kept.

#### `keywords` (Array)
Search terms, one per line. Each keyword is searched in each location, so 3 keywords × 2 locations runs 6 searches.

#### `locations` (Array)
Plain words — `"London"`, `"New York"`, `"Germany"`. Looked up on LinkedIn automatically; no `geoId` needed.

#### `distanceMiles` (Integer) — default `25`
`0` (exact location), `5`, `10`, `25` or `50`. Any other number rounds up to the next one LinkedIn accepts.

#### `datePosted` (String) — default `pastMonth`
`anyTime`, `past24Hours`, `pastWeek` or `pastMonth`.

#### `postedWithinHours` (Integer)
A custom freshness window in hours, overriding the dropdown. LinkedIn's own filter offers only 24 hours / 1 week / 1 month — this takes any window, so you can poll hourly for brand-new postings (`1`) or scan the last three days (`72`).

#### `companyNames` / `companyIds` (Arrays)
Restrict to specific employers, by name or by numeric ID.

#### `easyApply` / `under10Applicants` (Booleans)
Only jobs you can apply to on LinkedIn, and only jobs with fewer than ten applicants so far.

### Refine the results

LinkedIn's public job search accepts several filters and then ignores them — a request that asks for internships comes back with everything. Rather than pass them along and hope, this Actor applies them to the data it actually collected:

- `jobTypes` — Full-time, Part-time, Contract, Temporary, Internship, Volunteer, Other
- `experienceLevels` — Internship … Executive
- `workplaceTypes` — Remote, Hybrid, On-site
- `titleIncludes` / `titleExcludes` — e.g. exclude `Senior`, `Principal`
- `companyExcludes` — drop staffing agencies
- `locationExcludes`
- `descriptionIncludes` / `descriptionExcludes` — e.g. require `visa sponsorship`, exclude `security clearance`
- `requireSalary` — only jobs with a published pay range
- `requirePoster` — only jobs with a named recruiter to contact
- `postedAfter` — a hard date cutoff, using each job's exact date

If every job is filtered out, the run tells you **which filter** removed them, so you can loosen it.

### Output and limits

#### `maxItems` (Integer) — default `100`
Total jobs across every search.

#### `maxItemsPerSearch` (Integer)
Cap each search so one broad keyword cannot consume the whole run.

#### `scrapeJobDetails` (Boolean) — default `true`
On: full description, salary, seniority, employment type, industry, applicant count and recruiter. Off: only the search-card fields — roughly 10× faster and cheaper, ideal for monitoring which jobs exist.

#### `includeDescriptionHtml` (Boolean) — default `true`
#### `deduplicateJobs` (Boolean) — default `true`
#### `maxConcurrency` (Integer) — default `5`

---

## 📊 Sample output

```json
{
  "job_id": "4458752240",
  "title": "Senior Software Engineer",
  "company_name": "Novatus",
  "company_id": "20337423",
  "company_url": "https://uk.linkedin.com/company/novatus-global",
  "location": "London Area, United Kingdom",
  "posted_date": "2026-08-25",
  "posted_relative": "2 weeks ago",
  "employment_type": "Full-time",
  "seniority_level": "Mid-Senior level",
  "job_function": "Engineering",
  "industries": "Software Development and Financial Services",
  "workplace_type": "Hybrid",
  "applicants_count": 90,
  "applicants_is_estimate": false,
  "benefits": ["Actively Hiring"],
  "apply_type": "EXTERNAL",
  "is_easy_apply": false,
  "apply_url": null,
  "poster_name": "Sam Cainer",
  "poster_title": "People & Talent Partner | Novatus",
  "poster_profile_url": "https://uk.linkedin.com/in/samcainer",
  "description_text": "Novatus is a Series B scale-up RegTech SaaS provider…",
  "job_url": "https://uk.linkedin.com/jobs/view/senior-software-engineer-at-novatus-4458752240",
  "found_by": ["\"Software Engineer\" in London last 30d"],
  "success": true
}
```

With a published pay range:

```json
{
  "title": "Registered Nurse",
  "company_name": "Encompass Health",
  "location": "Colorado Springs, CO",
  "salary": "$39.00/hr - $54.00/hr",
  "salary_min": 39,
  "salary_max": 54,
  "salary_currency": "$",
  "salary_period": "HOURLY",
  "salary_source": "Encompass Health provided pay range"
}
```

---

## ❓ Frequently asked questions

**Do I need a LinkedIn account, cookies or a session token?**
No. The Actor reads only what LinkedIn publishes publicly. Nothing to log into, nothing to expire, nothing to get your account restricted.

**What does it cost?**
The Actor itself is free. You pay only your own Apify platform usage, which is covered by the free plan's monthly credit for typical runs.

**Why is `apply_url` always empty?**
LinkedIn reveals the employer's application link only to signed-in users. Rather than fill the column with the LinkedIn job URL — which would look like an apply link while pointing back to LinkedIn — it is reported as `null`. Use `apply_type` to tell an on-LinkedIn application from one that hands off to the employer, and `job_url` to open the posting.

**Why do some jobs have no salary?**
Most employers do not publish one. Where a range exists it is captured and parsed; where none exists the field is `null` rather than guessed. Set `requireSalary` to keep only jobs that published one.

**How many jobs can one search return?**
LinkedIn's public job search will not page past **1,000 results** for any single query, however broad it is — a search matching 11,000 jobs still yields at most 1,000. The Actor warns you when a search exceeds this. To collect more, split the run into narrower searches: a shorter `postedWithinHours` window, a tighter location, or one search per company.

**Can I run it on a schedule?**
Yes. Combine a Schedule with `postedWithinHours` to poll for genuinely new postings, and `job_id` to tell new from already-seen.

**Why did I get fewer jobs than `maxItems`?**
Either the search ran out of matches, or your filters removed them. The run log and `RUN_SUMMARY` report exactly which filter rejected how many.

---

## 📋 Dataset views

The dataset ships with five ready-made views, so you rarely need to wrangle columns:

- **Overview** — title, company, location, date, salary, level, applicants, link
- **Recruiter contacts** — every job that names the person who posted it
- **Salary data** — jobs with a published range, parsed into comparable numbers
- **Full descriptions** — the complete job text
- **Problems** — anything that could not be scraped, and why

---

## ⚖️ Legal & compliance

This Actor collects only publicly available job postings — the same pages any visitor can view without signing in. It does not log in, does not use anyone's account credentials, and does not access private or member-only data.

You are responsible for how you use the data, including compliance with GDPR, CCPA and any other law that applies to you. Job postings routinely name a recruiter; where they do, that name is personal data and you should have a lawful basis before storing or contacting them.

---

## 🤝 Support

### Get help
- **Issues & feature requests** — open an issue on the Actor's page
- **Questions** — use the Apify Actor chat

### Social
- GitHub: [FlowExtractAPI](https://github.com/FlowExtractAPI)

---

*Built and maintained by FlowExtract API.*
