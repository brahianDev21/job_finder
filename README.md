# Job Finder AI

> AI-powered job search assistant. Upload your CV, set preferences, and let AI find your next job.  
> **Zero dependencies. One HTML file. Works in any browser.**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![No dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)]()
[![LLM](https://img.shields.io/badge/LLM-agnostic-purple)]()

---

## Quick Start

1. **Open `index.html`** in any modern browser. No `npm install`, no server needed.
2. **Set up your API key** — The API Dashboard panel opens automatically on first visit. Choose your provider (OpenAI, Claude, Ollama, etc.), paste your key, and click **Test Connection**.
3. **Upload your CV** — Drop a PDF/DOCX/TXT file in the Profile panel and click **Parse with AI**. The LLM extracts your skills, experience, and education into a structured profile.
4. **Click Start Research** — The AI searches your approved platforms, extracts job listings, scores them against your profile, and populates the board in real-time.

That's it. You're job hunting with AI.

---

## How It Works

```
 ┌──────────┐    ┌──────────────┐    ┌───────────┐    ┌──────────┐    ┌────────────┐
 │  Profile │───▶│ Preferences  │───▶│ Platforms │───▶│ Research │───▶│  Job Board │
 │  (CV→AI) │    │ (role, $,    │    │ (toggle,  │    │ (query→  │    │ (apply,    │
 │          │    │  location)   │    │  suggest) │    │  fetch→  │    │  skip,     │
 │          │    │              │    │           │    │  extract) │    │  filter)   │
 └──────────┘    └──────────────┘    └───────────┘    └──────────┘    └────────────┘
                                                                           │
                                                                    ┌──────▼──────┐
                                                                    │  Refinement │
                                                                    │  (natural   │
                                                                    │  language)  │
                                                                    └──────┬──────┘
                                                                           │
                                                                  "exclude junior
                                                                   roles, remote
                                                                   only, +Python"
```

1. **Profile** — Your CV is parsed by an LLM into a structured JSON profile (skills, experience, education, location).
2. **Preferences** — You define what you want: job titles, salary range, location, work modality, seniority, keywords to include/exclude. An optional AI interview guides you through it.
3. **Platforms** — Toggle which job sites to search (LinkedIn, Indeed, Computrabajo, RemoteOK, etc.). The AI can suggest platforms based on your profile and preferences.
4. **Research** — The AI follows `rules.md` (the research skill file) to construct search queries, fetch job listings from each platform, extract structured job data via the LLM, score each job against your profile, deduplicate, and populate the board.
5. **Job Board** — Track each job as Pending, Applied, or Skipped. Filter by location type, search within results, sort by match score or date. Use the **Assist** button to generate cover letter tips.
6. **Refinement** — Type natural language feedback at any time: *"no more junior roles"*, *"only remote"*, *"salary above $3M COP"*. The LLM updates your preferences and re-runs research.

---

## Features

| Category | Feature |
|---|---|
| **Profile** | CV upload (PDF, DOCX, TXT), AI-powered parsing, manual input, inline JSON editing |
| **Preferences** | AI interview (conversational), manual form, salary range, location, modality, keywords, seniority |
| **Platforms** | 8 built-in platforms (LinkedIn, Indeed, Computrabajo, Glassdoor, GetOnBoard, RemoteOK, WeWorkRemotely, Torre), enable/disable per platform, AI platform suggestion |
| **Research** | Sequential/platform-by-platform execution with progress tracking, pause/cancel, rate limiting (10s between requests), cost estimation before start |
| **Job Board** | Pending/Applied/Skipped status tracking, filter by Local/Hybrid/Remote tabs, text search, sort by match/date/salary/title |
| **Match Scoring** | 0-100 score per job based on title match, skill overlap, location fit, salary fit, seniority fit, and keyword signals |
| **Refinement** | Natural language feedback bar, LLM-parsed preference updates, auto-flags jobs that no longer match |
| **Assist** | AI-generated cover letter tips per job: key points, skills to emphasize, potential interview questions |
| **Data** | Export all data as JSON backup, import/restore from backup, all data in localStorage |
| **LLM** | Provider-agnostic — OpenAI, Anthropic (Claude), OpenRouter, Ollama (local), any OpenAI-compatible endpoint, streaming support |
| **Proxy** | 3 web-fetch modes: public proxy (zero setup), local proxy script, direct fetch |
| **UI** | Dark theme, responsive (desktop/tablet/mobile), collapsible panels, connection status indicator, toast notifications |

---

## Usage Guide

### 1. API Dashboard

The first thing you see on opening the app. Configure your LLM connection here.

| Field | Description |
|---|---|
| **Provider** | OpenAI, Anthropic (Claude), OpenRouter, Ollama (local), or Custom OpenAI-compatible endpoint |
| **Model** | Model name (e.g. `gpt-4o`, `claude-sonnet-4-20250514`, `llama3`) |
| **API Endpoint** | Base URL for the provider's API. Auto-filled when you select a provider. Only editable for **Custom** mode |
| **API Key** | Your provider's API key. Stored **only** in your browser's localStorage. Never sent anywhere except the provider's API |
| **Web Fetch Proxy** | How the app fetches job platform pages (see Proxy Modes below) |
| **Concurrency** | Sequential (1 platform at a time), Balanced (2 at a time), or Aggressive (all at once) |
| **Match Threshold** | Minimum match score (0-100) for a job to be added to your board. Default: 60 |
| **Temperature** | LLM creativity (0-2). Lower = more precise. Default: 0.3 |
| **Stream responses** | Enable real-time text streaming from the LLM |

Click **Save** to persist your settings. Click **Test Connection** to verify everything works. A green dot appears in the header when connected.

#### Supported LLM Providers

| Provider | Endpoint | Recommended Model | Notes |
|---|---|---|---|
| **OpenAI** | `https://api.openai.com/v1` | `gpt-4o` | Best structured output reliability |
| **Anthropic** | `https://api.anthropic.com/v1` | `claude-sonnet-4-20250514` | Native Anthropic API format |
| **OpenRouter** | `https://openrouter.ai/api/v1` | `openai/gpt-4o` | Multi-model gateway, pay-per-token |
| **Ollama** | `http://localhost:11434/v1` | `llama3` | Local, free, no internet needed |
| **Custom** | Any URL | Any | Any OpenAI-compatible endpoint (Groq, DeepSeek, local LM Studio, etc.) |

#### Web Proxy Modes

| Mode | How it works | Pros | Cons |
|---|---|---|---|
| **Public** | Uses `allorigins.win` as CORS proxy | Zero setup, works out of the box | Rate-limited, less private |
| **Local** | Requires running `python proxy.py` on port 8765 | Fully private, no rate limits | Requires Python, extra setup |
| **Direct** | Fetches directly from the browser | Simplest, no proxy | Most job sites block CORS — many won't work |

---

### 2. Creating Your Profile

The **Profile** panel handles your professional identity.

**Option A — Upload CV:**
1. Drag a PDF, DOCX, or TXT file onto the drop zone (max 10 MB)
2. Click **Parse with AI**
3. The LLM extracts your name, skills, experience, education, and languages into a structured profile
4. Review the preview. Click **Edit Profile** to directly edit the JSON if needed

**Option B — Manual input:**
1. Click **Enter experience manually**
2. Paste or type your background in the prompt
3. Click **Parse with AI**

Your profile is saved to localStorage and persists across sessions. Use **Export** to download it as JSON.

> **Note:** Your CV text is sent to the LLM API for parsing. The file itself never leaves your browser. Nothing is stored on any intermediate server (we don't run any).

---

### 3. Setting Job Preferences

The **Job Preferences** panel defines what kind of jobs you want.

**Option A — AI Interview:**
1. Click **AI Interview**
2. The LLM asks you 6 questions one at a time (role, salary, location, modality, seniority, keywords)
3. Answer conversationally. At the end, preferences are auto-saved as JSON

**Option B — Manual form:**
1. Click **Fill manually**
2. Fill in: target job titles, seniority level, salary range (min/max + currency), location, work modality (remote/hybrid/onsite), include keywords, exclude keywords, max posting age (days)
3. Click **Save Preferences**

Preferences affect what the research engine searches for, how jobs are scored, and which results are filtered out.

---

### 4. Selecting Platforms

The **Platforms** panel shows 8 built-in job platforms. Toggle each on/off to control where the AI searches.

Click **Suggest platforms for my profile** to let the AI recommend which platforms to enable based on your role, location, and preferences.

The enabled count and estimated API cost are shown in the research bar.

---

### 5. Running Research

Click **Start Research** in the research bar. The pipeline runs as follows:

```
Construct queries ──▶ Fetch platform pages ──▶ Extract jobs (LLM) ──▶ Score & filter ──▶ Deduplicate ──▶ Add to board
```

- A progress bar shows the current step and per-platform progress
- Jobs appear on the board in real-time as they're discovered
- You can **Pause** (and resume later) or **Cancel** at any time
- Research waits 10 seconds between platforms to respect rate limits
- A cost estimate is shown before you start (~$0.001 per API call)

After research completes, stats update and a summary toast appears.

---

### 6. Managing Jobs on the Board

The job board inherits the UI from the original job-tracker:

| Action | How |
|---|---|
| **Mark as Applied** | Click the circle button on a card. Turns green |
| **Skip** | Click the ✕ button on the right. Card dims |
| **Undo** | Click again to toggle back |
| **Filter** | Use the All / Local / Hybrid / Remote tabs |
| **Search** | Type in the search bar to filter by title, company, or tags |
| **Sort** | Use the dropdown to sort by match score, date, salary, or title |
| **Show Applied/Skipped** | Click the Applied or Skipped buttons to reveal them below pending jobs |
| **View Job** | Click **View Job →** to open the posting in a new tab |
| **Assist** | Click **⚡ Assist** to generate AI-powered cover letter tips |

Each card shows:
- Job title with match score badge (green ≥85, blue ≥60, gray <60)
- Company name
- Location type badge (Remote / Hybrid / Local)
- Platform source and salary (if available)
- Technology tags
- Description snippet
- Posting date or discovery time
- New badge for jobs found in the last 24 hours

---

### 7. Refining Your Search

The **Refine** bar lets you update your preferences using natural language:

| You type | What happens |
|---|---|
| *"exclude junior roles"* | Adds "junior" to exclude keywords, flags matching jobs |
| *"only remote"* | Sets modality to remote only |
| *"add Python to my skills"* | Updates profile with Python |
| *"salary above $3M COP"* | Sets min salary to 3,000,000 COP |
| *"don't search in Bogotá"* | Removes Bogotá from location preferences |

The LLM parses your feedback, updates preferences, and flags jobs that no longer match. Click **Start Research** again to find new jobs with the updated criteria.

---

### 8. Export & Import

- **Export** — Downloads a `jobfinder_backup_YYYY-MM-DD.json` file containing your profile, preferences, platforms, jobs, statuses, and settings
- **Import** — Restore everything from a previously exported file. Useful for backups, switching devices, or sharing configuration

Buttons are in the header bar next to the tabs.

---

### 9. Assist — Cover Letter Tips

On any job card, click **⚡ Assist** to generate AI-powered application tips:
- Key points to highlight in your cover letter
- Skills from the job description to emphasize
- Potential interview questions based on the role

The result is shown in a prompt. Confirm to copy to clipboard.

---

## Technical Details

### Architecture

```
job_finder/
│
├── index.html       ← Single file: HTML + CSS + JS (1990+ lines)
├── rules.md         ← Research skill file (also the AI system prompt)
└── specs.md         ← Full product specification
```

The entire application is a **single HTML file** with embedded CSS and JavaScript. There is no build step, no framework, no npm dependencies, and no backend server. It runs entirely in the browser.

### Tech Stack

| Layer | Technology |
|---|---|
| UI | HTML5 + CSS3 (custom properties, grid, flexbox, dark theme) |
| Logic | Vanilla JavaScript (ES6+) |
| Storage | localStorage (browser Web Storage API) |
| LLM API | Fetch API → OpenAI-compatible or Anthropic endpoints |
| Web fetching | Fetch API via CORS proxy |
| PDF parsing | PDF.js (dynamically loaded from CDN only when needed) |
| File handling | FileReader API |
| Dependencies | **None** (PDF.js loaded on-demand) |

### Data Flow

```
                     ┌─────────────┐
                     │ localStorage │
                     └──────┬──────┘
           ┌────────────────┼────────────────┐
           │                │                │
    ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
    │  SETTINGS   │ │   JOBS[]    │ │  STATUSES{} │
    │  (provider, │ │  (title,    │ │  (link →     │
    │   model,    │ │   company,  │ │   pending/   │
    │   key, etc) │ │   score...) │ │   applied/   │
    └─────────────┘ └─────────────┘ │   skip)      │
                                    └──────────────┘
    ┌─────────────┐ ┌─────────────┐ ┌──────────────┐
    │  PROFILE    │ │  PREFS      │ │   STATE      │
    │  (skills,   │ │  (roles,    │ │  (filters,   │
    │   exp, edu) │ │   salary,   │ │   history)   │
    └─────────────┘ │   location) │ └──────────────┘
                    └─────────────┘
```

All data is stored in `localStorage` with the `jf_` prefix. Use **Export** to download everything as JSON and **Import** to restore.

### Research Pipeline (detailed)

```
1. CONSTRUCT QUERIES   →  LLM generates 2-3 search queries per enabled platform
                           based on profile + preferences
2. FETCH PAGES         →  For each query, construct a platform search URL
                           and fetch via configured proxy mode
3. EXTRACT JOBS        →  LLM parses HTML, extracts structured job data
                           (title, company, location, link, salary, tags, description)
4. SCORE MATCHES       →  LLM scores each job 0-100 against profile + preferences
5. FILTER              →  Remove jobs with match_score < threshold
                           Remove stale jobs (posted > max_posted_days)
                           Remove jobs with excluded keywords in title
6. DEDUPLICATE         →  Check for duplicate links and title+company combinations
7. ADD TO BOARD        →  New jobs are prepended to the jobs array and rendered
                     
⏱ 10-second delay between platforms
```

### Rules.md — The Research Skill File

`rules.md` is both human-readable documentation and the AI's system prompt during research. It defines:

- **Core principles** — quality over quantity, respect exclusion criteria
- **Pre-search setup** — what data the AI reads before searching
- **Query construction** — how to build search URLs per platform
- **Job extraction** — what fields to extract, what to skip
- **Deduplication** — how to avoid duplicate entries
- **Match scoring** — weighted algorithm (title 25%, skills 30%, location 15%, salary 15%, seniority 10%, keywords 5%)
- **Quality filtering** — score threshold, date cutoff, excluded keywords
- **Rate limiting & ethics** — 10s between requests, respect robots.txt
- **Output format** — exact JSON schema

You can edit `rules.md` to customize how the AI researches. Changes take effect on the next research run.

### Storage Keys

| Key | Content |
|---|---|
| `jf_settings` | Provider, model, API key, proxy, concurrency, threshold, temperature |
| `jf_profile` | Parsed CV data (personal info, skills, experience, education) |
| `jf_preferences` | Job search criteria (roles, salary, location, modality, keywords) |
| `jf_platforms` | Platform catalog with enabled/disabled states |
| `jf_jobs` | Discovered job listings array |
| `jf_tracking` | Job statuses keyed by URL (pending / applied / skip) |
| `jf_app_state` | UI state (filters, sort, research history, last research time) |

---

## Privacy & Security

- **Your API key never leaves your browser** except via direct HTTPS calls to your LLM provider's API endpoint. No proxy, no middleman.
- **No telemetry, no analytics, no tracking.** The app makes zero external requests except: LLM API calls (to your configured provider), web page fetches (to job platforms via your chosen proxy), and CDN loads for PDF.js (only when you upload a PDF).
- **Your CV data stays local.** The raw CV text is sent to the LLM for parsing, but the file itself is never uploaded anywhere.
- **No accounts, no servers, no database.** Everything lives in your browser's localStorage.
- **Open source.** You can audit every line of code. The app is a single HTML file.

---

## FAQ

### Do I need a backend server?
No. The app is a single HTML file. Open it in a browser and it works. The only external service is your LLM provider's API.

### Does it work offline?
Partially. The UI, data management, and job tracking work offline. Research requires internet (LLM API + job platform fetching).

### How much does research cost?
Depends on your LLM provider, model, and how many platforms you enable. A typical research run with 3 platforms and GPT-4o uses ~30-50 API calls and costs ~$0.05-$0.15 USD. An estimate is shown before you start.

### Can I use a local LLM?
Yes. Select **Ollama** in the provider dropdown and point it to `http://localhost:11434/v1` with any model you have pulled (e.g. `llama3`, `mistral`). This is free and works offline.

### Can I use a different LLM provider not listed?
Yes. Select **Custom** and enter any OpenAI-compatible endpoint URL (e.g. Groq, DeepSeek, LM Studio, vLLM, Together AI).

### Why do I need a web proxy?
Most job platforms block cross-origin requests from browsers (CORS policy). The proxy acts as a middleman to fetch pages on your behalf. You can use the free public proxy or run your own local one.

### What platforms are supported?
LinkedIn, Indeed, Computrabajo, Glassdoor, GetOnBoard, RemoteOK, WeWorkRemotely, and Torre. You can extend this by editing the `platforms` array in the code or via the AI suggestion feature.

### Can I add custom platforms?
Yes. Edit the `loadPlatforms()` function's default array in `index.html`, or modify the platforms in the UI and export/import them. The `search_url` field supports `{query}` and `{location}` placeholders.

### How do I backup my data?
Click **Export** in the header. This downloads a JSON file with all your data. Click **Import** to restore from a backup.

### What if research finds no jobs?
Check that: (1) at least one platform is enabled, (2) your preferences aren't too restrictive, (3) the match score threshold isn't too high, (4) your API key is valid and the connection test passes. Try lowering the threshold or broadening your search criteria.

### Does it support multiple profiles?
Not in the MVP. This is planned for v2. For now, use Export/Import to switch between different profiles.

### How do I update the app?
Download the latest `index.html` and replace your existing one. Export your data first, then import it into the new version.

---

## File Structure

```
job_finder/
├── index.html       # Main application (HTML + CSS + JS)
├── rules.md         # Research skill file (AI system prompt + docs)
├── specs.md         # Full product specification
├── README.md        # This file
└── job-tracker.html # Original job tracker (used as UI reference)
```

---

## Customization

### Changing the Research Behavior

Edit `rules.md` to modify:
- How the AI constructs search queries
- How jobs are extracted and scored
- Match scoring weights
- Quality filtering rules
- Rate limiting behavior

### Adding a Platform

Edit the default array in `loadPlatforms()` (inside `index.html`), or modify platforms in the UI:

```javascript
{
  id: 'myplatform',           // unique ID
  name: 'My Platform',        // display name
  enabled: true,              // enabled by default?
  region: 'global',           // global, latam, eu, etc.
  job_types: ['remote'],      // supported types
  search_url: 'https://myplatform.com/jobs?q={query}&l={location}'
}
```

### Changing Appearance

All colors are CSS custom properties in the `:root` block. Change `--accent`, `--bg`, `--surface`, etc. to match your preferred theme.

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

Built with ❤️ for job seekers everywhere. PRs welcome.
