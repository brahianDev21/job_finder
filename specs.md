# Job Finder AI — Specs

> **Status:** MVP  
> **Architecture:** Zero-dependency, single-file HTML app with LLM API integration  
> **Storage:** Local-first (JSON files + localStorage)  
> **License:** MIT

---

## 1. Vision & Philosophy

Job Finder AI is an open source, AI-powered job search assistant. The user uploads their CV or describes their experience, the AI builds a structured profile, asks about preferences, suggests platforms, executes research following a well-defined rules file, and populates a job tracker board. The user maintains full control — they can refine the research at any time using natural language feedback.

**Design principles:**
- **Zero dependencies.** One HTML file. Open in a browser and it works. No `npm install`, no build step.
- **Local-first.** All data (profile, preferences, jobs, state) lives on the user's machine. No accounts, no cloud.
- **Bring your own API key.** The user provides their own OpenAI or Claude API key. No proxy, no middleman.
- **AI-augmented, human-controlled.** The AI researches and suggests; the user approves, rejects, and refines.
- **Transparent.** The `rules.md` file is human-readable and fully auditable. Users can modify how research behaves.

---

## 2. Core User Flow

### Phase 1 — Profile Creation

**Entry:** User opens the app for the first time. An onboarding panel is visible at the top.

1. User can either:
   - **Upload CV** (PDF, DOCX, or image) — The file is read client-side via `FileReader`. Its text content is sent to the LLM API with a structured prompt to extract: name, location, contact, skills (with proficiency), work experience (role, company, dates, description), education, languages, and a summary.
   - **Describe manually** — A textarea where the user pastes or types their experience. Sent to the LLM API for the same structured extraction.
2. The LLM returns a JSON object matching the profile schema.
3. The profile is rendered as a preview. The user can edit any field inline.
4. The profile is saved to `profile.json` on disk (or localStorage fallback).

**Key UX rule:** The profile section becomes collapsible after completion, allowing the user to revisit and edit at any time.

### Phase 2 — Preferences

**Entry:** After profile is saved. A conversation-style panel opens.

1. The AI asks a series of questions (driven by the LLM, but guided by a deterministic structure):
   - _"What job titles are you targeting?"_ (free text, multi-value)
   - _"What salary range are you looking for?"_ (currency, min, max)
   - _"Where do you want to work? Cities, regions, or remote-only?"_
   - _"What work modality?"_ (remote / hybrid / onsite / any)
   - _"What seniority level?"_ (junior / mid / senior / lead / any)
   - _"Any keywords or technologies you specifically want or want to avoid?"_
2. Each answer updates `preferences.json`.
3. The preference panel shows a summary card with edit buttons per field.
4. A _"Skip interview"_ button allows power users to fill preferences manually via a form.

### Phase 3 — Platform Setup

**Entry:** After preferences are set. The LLM suggests platforms.

1. Based on the user's role, location, and modality preferences, the LLM returns a list of recommended platforms from the available catalog in `platforms.json`.
2. Each suggestion includes: platform name, description, why it's recommended, and estimated job volume.
3. The user toggles each platform on/off. All are selected by default unless the user deselects them.
4. Users can also manually add custom platforms with a URL search pattern.
5. The approved list is saved back to `platforms.json` (setting `enabled: true/false` per platform).

**Default platform catalog** (shipped with the app):

| Platform | Region | Strengths |
|---|---|---|
| LinkedIn | Global | Largest volume, good filters |
| Indeed | Global | Broad coverage, easy apply |
| Computrabajo | LatAm | Strong in Colombia, Mexico, Peru |
| Glassdoor | Global | Salary data, company reviews |
| RemoteOK | Global | Remote-only jobs |
| WeWorkRemotely | Global | Remote-only, curated |
| GetOnBoard | LatAm | Tech-focused, LatAm companies |
| Torre | Global | AI-matched, profiles over CVs |

### Phase 4 — Research Execution

**Entry:** User clicks "Start Research". The AI follows `rules.md`.

1. A progress panel opens showing:
   - Current step (e.g., _"Searching LinkedIn for 'React Developer remote'..."_)
   - Jobs found so far (counter)
   - Platform progress (check marks)
   - Elapsed time
2. The research runs in rounds — one per platform × one per search query combination.
3. For each search:
   - The LLM constructs an optimal search URL for the platform.
   - The app fetches the page content (via a proxy or direct, depending on CORS).
   - The LLM extracts job data from the HTML: title, company, location, link, salary (if present), description snippet, posted date.
   - Each extracted job is checked for duplicates against the existing `jobs.json`.
   - New jobs are scored for relevance (0-100) using the LLM comparing description against profile + preferences.
   - Jobs above a configurable threshold are added to the job board in real-time.
4. The job board (same card UI as the current `job-tracker.html`) updates incrementally.
5. When all platforms are exhausted, the progress panel shows a summary.

**Research behavior is governed by `rules.md`** — see Section 7.

### Phase 5 — Feedback & Refinement Loop

**Entry:** At any point during or after research. A _"Refine"_ bar is always visible.

1. User types natural language feedback into the refine bar:
   - _"exclude junior roles, I want mid-level only"_
   - _"don't search in Bogotá"_
   - _"add Python to my skills"_
   - _"only show jobs posted in the last 7 days"_
   - _"I want at least $3M COP"_
2. The LLM parses the feedback and:
   - Updates `preferences.json` with the new constraints.
   - Flags existing jobs that no longer match.
   - Optionally re-runs research with the updated parameters.
3. Flagged jobs are dimmed/deprioritized on the board but not deleted. The user can unflag them.

**Key UX rule:** Feedback must feel conversational and forgiving. The user never loses data.

---

## 3. File Structure

```
job_finder/
├── index.html           # Main application (HTML + CSS + JS)
├── profile.json         # User profile (parsed from CV or manual input)
├── preferences.json     # Job search criteria
├── platforms.json       # Platform catalog + user enabled/disabled states
├── rules.md             # Research skill file (structured AI instructions)
├── jobs.json            # Discovered job listings database
├── state.json           # Runtime state (tracking statuses, flags, notes)
└── specs.md             # This file
```

### File Schemas

**`profile.json`**
```jsonc
{
  "version": 1,
  "personal": {
    "name": "Jane Doe",
    "location": "Medellín, Colombia",
    "email": "jane@example.com",
    "phone": "+57 300 000 0000",
    "linkedin": "https://linkedin.com/in/janedoe",
    "portfolio": "https://janedoe.dev"
  },
  "skills": [
    { "name": "React", "proficiency": "advanced" },
    { "name": "TypeScript", "proficiency": "intermediate" },
    { "name": "Node.js", "proficiency": "intermediate" }
  ],
  "experience": [
    {
      "role": "Frontend Developer",
      "company": "TechCorp",
      "start": "2022-03",
      "end": "2025-06",
      "description": "Built React dashboards and design system...",
      "highlights": ["Led migration to TypeScript", "Reduced bundle size by 40%"]
    }
  ],
  "education": [
    {
      "degree": "B.S. Computer Science",
      "institution": "Universidad Nacional",
      "year": 2021
    }
  ],
  "languages": ["Spanish (native)", "English (B2)"],
  "summary": "Frontend developer with 3 years of experience...",
  "raw_cv_text": "Original CV text for reference..."
}
```

**`preferences.json`**
```jsonc
{
  "version": 1,
  "roles": ["Frontend Developer", "React Developer", "Full Stack Developer"],
  "salary": {
    "currency": "COP",
    "min": 4000000,
    "max": 8000000,
    "period": "monthly"
  },
  "location": {
    "cities": ["Medellín"],
    "regions": ["Antioquia"],
    "remote_ok": true,
    "hybrid_ok": true,
    "onsite_ok": false
  },
  "seniority": "mid",
  "modality": ["remote", "hybrid"],
  "keywords": {
    "include": ["React", "TypeScript", "Next.js", "frontend"],
    "exclude": ["junior", "trainee", "internship"]
  },
  "max_posted_days": 30,
  "filters": {
    "min_match_score": 60,
    "require_salary": false,
    "exclude_companies": []
  }
}
```

**`platforms.json`**
```jsonc
{
  "version": 1,
  "platforms": [
    {
      "id": "linkedin",
      "name": "LinkedIn",
      "enabled": true,
      "search_url_template": "https://www.linkedin.com/jobs/search/?keywords={query}&location={location}&f_TPR=r{days}",
      "region": "global",
      "job_types": ["local", "hybrid", "remote"],
      "extraction_rules": {
        "container": ".jobs-search__results-list > li",
        "title": ".job-card-list__title",
        "company": ".job-card-container__company-name",
        "location": ".job-card-container__metadata-item",
        "link": "a.job-card-list__title",
        "date": "time"
      },
      "notes": "May require login for full results. Consider using public listing pages."
    },
    {
      "id": "indeed",
      "name": "Indeed",
      "enabled": true,
      "search_url_template": "https://co.indeed.com/jobs?q={query}&l={location}&fromage={days}",
      "region": "global",
      "job_types": ["local", "hybrid", "remote"],
      "extraction_rules": {
        "container": ".job_seen_beacon",
        "title": ".jobTitle",
        "company": ".companyName",
        "location": ".companyLocation",
        "link": "a.jcs-JobTitle",
        "date": ".date"
      }
    },
    {
      "id": "computrabajo",
      "name": "Computrabajo",
      "enabled": true,
      "search_url_template": "https://co.computrabajo.com/trabajo-de-{query}",
      "region": "latam",
      "job_types": ["local", "hybrid"],
      "extraction_rules": {
        "container": ".box_border",
        "title": ".tO",
        "company": ".dO a",
        "location": ".w_ado",
        "link": "a.js-o-link",
        "date": ".fs13"
      }
    },
    {
      "id": "linkedin",
      "name": "LinkedIn",
      "enabled": true,
      "search_url_template": "https://www.linkedin.com/jobs/search/?keywords={query}&location={location}",
      "region": "global",
      "job_types": ["local", "hybrid", "remote"]
    },
    {
      "id": "getonboard",
      "name": "GetOnBoard",
      "enabled": true,
      "search_url_template": "https://www.getonbrd.com/empleos-{query}",
      "region": "latam",
      "job_types": ["remote"]
    },
    {
      "id": "remoteok",
      "name": "RemoteOK",
      "enabled": false,
      "search_url_template": "https://remoteok.com/remote-{query}-jobs",
      "region": "global",
      "job_types": ["remote"]
    },
    {
      "id": "weworkremotely",
      "name": "We Work Remotely",
      "enabled": false,
      "search_url_template": "https://weworkremotely.com/remote-jobs/search?term={query}",
      "region": "global",
      "job_types": ["remote"]
    }
  ]
}
```

**`jobs.json`**
```jsonc
[
  {
    "id": "uuid-v4",
    "title": "React Developer",
    "company": "TechCorp",
    "location": "Medellín, Antioquia",
    "location_type": "hybrid",
    "platform": "LinkedIn",
    "link": "https://www.linkedin.com/jobs/view/12345",
    "salary": "$5M–7M COP/mes",
    "posted": "2026-06-28",
    "discovered": "2026-07-02T10:30:00Z",
    "tags": ["React", "TypeScript", "Next.js"],
    "description_snippet": "We are looking for a React developer with 3+ years...",
    "match_score": 87,
    "warning": null,
    "top_pick": false,
    "easy_apply": true,
    "status": "pending",
    "flagged": false,
    "notes": ""
  }
]
```

**`state.json`**
```jsonc
{
  "version": 1,
  "last_research_at": "2026-07-02T10:25:00Z",
  "research_history": [
    {
      "id": "rh-001",
      "started_at": "2026-07-02T10:00:00Z",
      "ended_at": "2026-07-02T10:25:00Z",
      "platforms_scanned": ["linkedin", "computrabajo"],
      "jobs_found": 47,
      "queries_used": ["React developer Medellín", "Frontend developer remoto Colombia"]
    }
  ],
  "filters": {
    "active_tab": "all",
    "show_applied": false,
    "show_skipped": false,
    "search_query": "",
    "sort_by": "match_score",
    "sort_order": "desc"
  }
}
```

---

## 4. UI Layout

### Full layout (top to bottom)

```
┌──────────────────────────────────────────────────────┐
│  HEADER                                              │
│  Job Finder AI                    [⚙ Settings]      │
│                                                      │
│  STATS BAR                                           │
│  Total: 47  |  Applied: 12  |  Skipped: 5  |  New: 30│
├──────────────────────────────────────────────────────┤
│  ONBOARDING (collapsible, visible only when needed)  │
│  ┌─ Profile ──────────────────────────────────────┐  │
│  │ [📎 Upload CV]  or  [✏ Manual input]           │  │
│  │ ┌─────────────────────────────────────────────┐│  │
│  │ │ Name: Jane Doe  │  Location: Medellín       ││  │
│  │ │ Skills: React, TS, Node  │  Yrs exp: 3      ││  │
│  │ │ [ Edit profile ]                            ││  │
│  │ └─────────────────────────────────────────────┘│  │
│  └────────────────────────────────────────────────┘  │
│  ┌─ Preferences ──────────────────────────────────┐  │
│  │ Role: Frontend Dev  |  Salary: $4M–8M COP      │  │
│  │ Location: Medellín, Remote  |  Level: Mid      │  │
│  │ [ Edit preferences ]                           │  │
│  └────────────────────────────────────────────────┘  │
│  ┌─ Platforms ────────────────────────────────────┐  │
│  │ [✓] LinkedIn  [✓] Indeed  [✓] Computrabajo    │  │
│  │ [ ] RemoteOK  [✓] GetOnBoard  [ Manage ]      │  │
│  └────────────────────────────────────────────────┘  │
├──────────────────────────────────────────────────────┤
│  RESEARCH CONTROLS                                   │
│  [▶ Start Research]  [⏸ Pause]  [🔄 Re-run]        │
│  ┌─ Progress ─────────────────────────────────────┐  │
│  │ Step 2/3: Searching Indeed...  ✓ LinkedIn      │  │
│  │ ████████████░░░░░░░░  62%  Jobs found: 34      │  │
│  └────────────────────────────────────────────────┘  │
│  Refine: [________________________] [Apply]         │
│  💡 Try: "exclude junior roles" or "only remote"    │
├──────────────────────────────────────────────────────┤
│  TABS                                                │
│  [All 47] [Local 12] [Hybrid 15] [Remote 20]        │
├──────────────────────────────────────────────────────┤
│  CONTROLS                                            │
│  🔍 [Search jobs...________]  [Pending] [Applied] [Skip]│
│  Sort: [Match score ▼]                              │
├──────────────────────────────────────────────────────┤
│  JOB GRID (CSS Grid, same card design as current)    │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│  │ ● pending    │ │ ● pending    │ │ ● pending    │ │
│  │              │ │              │ │              │ │
│  │ React Dev    │ │ Frontend Eng │ │ Full Stack   │ │
│  │ TechCorp     │ │ Startup S.A. │ │ BigCo        │ │
│  │ [Hybrid]     │ │ [Remote] [⚡]│ │ [Local]      │ │
│  │ [LinkedIn]   │ │ [GetOnBoard] │ │ [Indeed]     │ │
│  │ $5M-7M COP   │ │              │ │ $6M COP      │ │
│  │              │ │              │ │              │ │
│  │ React TS     │ │ Vue 3 JS     │ │ React Node   │ │
│  │ Next.js      │ │ Tailwind     │ │ Python       │ │
│  │              │ │              │ │              │ │
│  │ 87% match 💎 │ │ 72% match    │ │ 64% match    │ │
│  │ Posted 2d    │ │ Posted 5d    │ │ Posted 1w    │ │
│  │ [⚡Assist]   │ │ [⚡Assist]   │ │ [⚡Assist]   │ │
│  │ [View Job]   │ │ [View Job]   │ │ [View Job]   │ │
│  └──────────────┘ └──────────────┘ └──────────────┘ │
├──────────────────────────────────────────────────────┤
│  EMPTY STATE (shown when no jobs match filters)      │
│  No jobs found. Try adjusting your filters or        │
│  running research with different preferences.        │
└──────────────────────────────────────────────────────┘
```

### Responsive behavior

- **Desktop (>1024px):** 3-column grid, full stats bar
- **Tablet (768–1024px):** 2-column grid, stacked stats
- **Mobile (<768px):** 1-column grid, simplified header, collapsible filters

### Color system (CSS custom properties, inherited from current design)

```css
:root {
  --bg: #0d1117;
  --surface: #161b22;
  --surface-hover: #1c2129;
  --border: #30363d;
  --text: #e6edf3;
  --text-muted: #8b949e;
  --accent: #58a6ff;
  --success: #3fb950;
  --warning: #d29922;
  --danger: #f85149;
  --applied: #3fb950;
  --skip: #f85149;
  --pending: #58a6ff;
  --local: #a371f7;
  --hybrid: #d29922;
  --remote: #3fb950;
  --match-high: #3fb950;
  --match-mid: #58a6ff;
  --match-low: #8b949e;
}
```

---

## 5. API Integration

### LLM Provider Configuration

The app supports two providers, selectable in settings:
- **OpenAI** (gpt-4o / gpt-4o-mini)
- **Claude** (claude-sonnet-4-20250514 / claude-3-5-haiku)

The API key is stored in localStorage (never transmitted anywhere except the provider's API). A settings panel allows switching providers and models.

### API Call Patterns

| Call | Endpoint | Frequency | Structured output? |
|---|---|---|---|
| `parse_cv_to_profile` | Chat completions | Once per CV upload/manual | Yes (JSON schema) |
| `conduct_preference_interview` | Chat completions | Once per onboarding | Streaming chat |
| `suggest_platforms` | Chat completions | Once per onboarding | Yes (JSON array) |
| `parse_refinement` | Chat completions | Per user feedback | Yes (JSON) |
| `construct_search_queries` | Chat completions | Per research run | Yes (JSON array) |
| `extract_jobs_from_html` | Chat completions | Per platform page | Yes (JSON array) |
| `score_job_match` | Chat completions | Per discovered job | Yes (score + reasoning) |
| `generate_cover_letter` | Chat completions | On demand (Assist) | Plain text |

### Prompt Engineering Strategy

Each API call uses:
1. **System prompt** — loaded from `rules.md` or hardcoded per call type.
2. **Context injection** — relevant slices of `profile.json`, `preferences.json`, and the HTML/scraped content.
3. **Structured output** — JSON schema enforcement where applicable to guarantee parsable results.

### Rate Limiting & Costs

- Research runs are intentionally sequential (not parallel across platforms) to control API costs.
- A cost estimator is shown before research starts: _"Estimated API calls: ~120. Estimated cost: ~$0.50 USD"_
- The user can set a max cost/budget per research run.
- Max `extract_jobs_from_html` calls per minute: 6 (10s delay between requests to respect platform servers).

### Web Fetch Strategy

Since many job platforms block CORS requests from browsers, the app uses one of two strategies:
1. **AllOrigins-style public proxy** — For MVP simplicity. The app prepends a proxy URL to fetch blocked pages.
2. **Local proxy mode** — For advanced users, a small local server script (Python/Node) that the app can spawn.

The chosen proxy is configurable in settings.

---

## 6. `rules.md` — Research Skill File

This is the **most critical file** in the app. It is human-readable markdown that also serves as the system prompt for the AI during research. Users can (and should) customize it.

```markdown
# Research Rules — Job Finder AI

## 1. Core Principles

- You are a job search assistant. Your goal is to find relevant, high-quality
  job listings that match the user's profile and preferences.
- Be thorough but respectful. Never overload platforms with requests.
- Prioritize quality over quantity. A curated list of 20 great matches
  is better than 200 mediocre ones.
- Always respect the user's exclusion criteria. If they say "no junior roles,"
  do not return any job with "junior" in the title.

## 2. Pre-Search Phase

Before searching any platform:

1. **Read the profile** (`profile.json`):
   - Understand their skills, experience level, and career trajectory.
   - Note their location and language abilities.

2. **Read the preferences** (`preferences.json`):
   - Target roles, salary range, location constraints, modality.
   - Keywords to include and exclude.
   - Minimum match score threshold.

3. **Read the platform catalog** (`platforms.json`):
   - Only search platforms marked `enabled: true`.
   - Use the `search_url_template` for each platform.
   - Respect each platform's `job_types` — don't search remote-only platforms
     for onsite roles.

## 3. Query Construction

For each enabled platform, construct search queries by combining:

- **Role + Location** (e.g., `React Developer Medellín`)
- **Role + Modality** (e.g., `Remote Frontend Developer Colombia`)
- **Key skill + Role** (e.g., `TypeScript Developer`)

Rules:
- Generate 2–5 queries per platform. Don't spam.
- Use the platform's native search syntax when known.
- For platforms with advanced filters, construct URLs that pre-apply
  date filters (e.g., posted in last 30 days).
- Prefer the platform's API/search page over scraping Google.
- URL-encode all query parameters properly.

## 4. Job Extraction

When extracting jobs from a page:

1. Look for individual job cards/list items — each one is a candidate.
2. For each candidate, extract:
   - **title** — Clean text, no HTML entities, trimmed.
   - **company** — Company name only, no location metadata.
   - **location** — City/region/remote indicator.
   - **link** — Full URL to the job posting page.
   - **salary** — If visible on the listing. Format as "$X COP/mes".
   - **description_snippet** — First 200 characters of the description, or
     visible summary text.
   - **posted** — Date string. Prefer ISO format (YYYY-MM-DD).
   - **tags** — Extract all mentioned technologies, skills, and frameworks.
     Normalize names (e.g., "react.js" → "React", "typescript" → "TypeScript").
   - **easy_apply** — True if the listing mentions one-click apply, LinkedIn
     Easy Apply, or similar.
3. If a job card is missing critical fields (title or link), skip it.
4. If a job card is clearly not a real job (sponsored content, "promoted"
   without a real position, training/course ads), skip it.

### Location Type Detection

Classify each job into one of:
- **remote** — Mentions "remote", "remoto", "trabajo desde casa", "work from anywhere",
  or "teletrabajo".
- **hybrid** — Mentions "hybrid", "híbrido", "mixed", "2–3 days in office",
  or similar partial-remote language.
- **local** — Specifies a physical location with no remote modifier.

If unclear, default to `local`.

## 5. Deduplication

Before adding a job to `jobs.json`:

1. Check if a job with the exact same `link` already exists. If so, skip.
2. Check if a job with the same `title`, `company`, and `location` already
   exists from another platform. If so, merge tags and note both platforms,
   but keep only one entry. Add a `cross_posted` field listing all platforms.
3. If the same title+company appears but is clearly a different opening
   (different location, seniority, or department), keep both.

## 6. Match Scoring

For each new job, assign a `match_score` (0–100) based on:

| Factor | Weight | How to evaluate |
|---|---|---|
| Title match | 25% | Does the role match the user's target roles? Exact match = 25. Adjacent role = 15. Unrelated = 0. |
| Skill overlap | 30% | What % of the user's skills appear in the job description/tags? 80%+ = 30. 50–79% = 20. 20–49% = 10. <20% = 0. |
| Location fit | 15% | Is the location compatible with preferences? Exact city match = 15. Same region = 10. Remote matches remote_ok = 15. Mismatch = 0. |
| Salary fit | 15% | Within range? At or above min = 15. Slightly below = 8. Way below = 0. No salary listed = 8 (neutral). |
| Seniority fit | 10% | Does the job level match the user's desired seniority? Match = 10. One level off = 5. Two+ levels off = 0. |
| Keyword signals | 5% | Bonus for matching include keywords, penalty for exclude keywords appearing. |

> **Hard rule:** If any excluded keyword appears in the title, set match_score = 0
> and flag the job with a warning. The user may still want to see it.

## 7. Quality Filtering

Apply these filters before adding jobs:

1. `match_score < min_match_score` → Skip entirely (don't add).
2. `posted > max_posted_days` ago → Skip (stale job).
3. Title contains "internship", "trainee", "pasantía", "becario" → Skip
   unless the user's preferences explicitly allow junior roles.
4. Company in `exclude_companies` → Skip.

## 8. Rate Limiting & Ethics

- Wait at least 10 seconds between requests to the same platform.
- Do not attempt to bypass login walls, paywalls, or CAPTCHAs.
- Do not scrape pages that require authentication.
- Respect `robots.txt` and platform terms of service.
- If a platform returns an error or blocks the request, mark it as failed
  in the progress panel and move on. Don't retry more than once.

## 9. Output Format

Every extracted job must be returned in this exact JSON structure:

```json
{
  "title": "string",
  "company": "string",
  "location": "string",
  "location_type": "remote|hybrid|local",
  "platform": "string",
  "link": "string",
  "salary": "string|null",
  "posted": "string (YYYY-MM-DD)",
  "tags": ["string"],
  "description_snippet": "string",
  "match_score": 0-100,
  "easy_apply": false,
  "warning": "string|null",
  "top_pick": false,
  "cross_posted": ["platform_id"]
}
```
```

---

## 7. MVP Feature Checklist

- [ ] CV upload (PDF, DOCX, PNG/JPG) — client-side text extraction
- [ ] Manual experience input (textarea + AI parsing)
- [ ] Profile generation (JSON output, editable inline)
- [ ] AI-driven preference interview (chat-style)
- [ ] Manual preference form (skip-interview fallback)
- [ ] Platform catalog display + toggle on/off
- [ ] Custom platform addition
- [ ] Research execution (sequential, one platform at a time)
- [ ] Real-time job board population during research
- [ ] Progress panel with step tracking
- [ ] Match score display per job card
- [ ] Natural language refinement bar
- [ ] `rules.md` — shipped with the app, editable by user
- [ ] Job status tracking (pending / applied / skip) — same UX as current
- [ ] Job filtering (all / local / hybrid / remote tabs)
- [ ] Search within results
- [ ] Sort by match score, date, salary
- [ ] "Assist" button — generate cover letter via LLM
- [ ] "View Job" link — opens job posting in new tab
- [ ] Stats bar (total, applied, skipped, pending)
- [ ] Data export (download `jobs.json` + `profile.json`)
- [ ] Data import (restore from exported files)
- [ ] API key configuration (settings panel)
- [ ] Provider/model selection (OpenAI / Claude)
- [ ] Cost estimation before research
- [ ] Responsive design (desktop, tablet, mobile)
- [ ] Dark theme (inherit from current design)
- [ ] All data stored locally (JSON files + localStorage)

### Out of MVP Scope (v2+)

- Multiple user profiles
- Scheduled recurring research (daily/weekly auto-scan)
- Email/SMS alerts for new matches
- Auto-fill job applications (browser automation)
- Company research (Glassdoor integration, culture analysis)
- Salary negotiation tips per offer
- Interview question prediction per job
- Collaborative multi-user mode
- Cloud sync / multi-device support
- PWA / offline support
- Chrome extension for LinkedIn one-click save

---

## 8. Implementation Notes

### Tech Stack

| Concern | Choice |
|---|---|
| UI | Vanilla HTML + CSS + JS (single file) |
| LLM API | OpenAI GPT-4o / Claude Sonnet (configurable) |
| CV parsing | PDF.js (for PDF), Mammoth.js (for DOCX), Tesseract.js (for images) — all loaded from CDN only when needed |
| Storage | `localStorage` for state + app state; JSON file downloads for import/export |
| Web fetching | `fetch()` with a public CORS proxy (configurable) |
| File handling | `FileReader` API for CV upload; Blob/download for export |

### Loading third-party libraries

To keep the app a "single file that works when opened," heavy libraries (PDF.js, Mammoth.js) are:
1. Not bundled — they are loaded dynamically from CDN only when the user uploads a file of that type.
2. Indicated in the UI: _"Loading PDF parser..."_ with a spinner.

### LLM Provider Flexibility

The app is designed to be **LLM-agnostic**. No provider is hardcoded. Instead:

- A single `provider.js` configuration module defines API endpoint, model name, headers, and request/response format.
- Switching between OpenAI, Claude, Groq, Ollama, or any OpenAI-compatible endpoint is done by editing that config or using the settings UI.
- The app auto-detects the provider format based on the endpoint URL and adjusts request/response parsing accordingly.
- Default provider is OpenAI-compatible API format (most widely supported). Claude-native format is also supported.

### Web Fetching Strategy

Two proxy modes, user-selectable in settings:

1. **Public proxy** (default) — Uses `allorigins.win` or `corsproxy.io`. Zero setup. Works out of the box. Good for casual use.
2. **Local proxy** — Ships with a `proxy.py` (30-line Python script). Run `python proxy.py` in a terminal, the app connects to `localhost:8765`. More private, no rate limits. Good for power users.
3. **Direct** — Attempts direct fetch. Works only on platforms that don't block CORS (RemoteOK, WeWorkRemotely).

### CV File Size

Maximum CV file size: **10 MB**. If the file exceeds this, a warning is shown and the user is prompted to reduce the file size or paste the text manually.

### Research Concurrency

The user configures concurrency in settings:

- **Sequential** (1 platform at a time): Slower, uses fewer API calls, easier to debug. Default.
- **Balanced** (2 platforms at a time): Moderate speed and cost.
- **Aggressive** (all platforms simultaneously): Fastest, highest parallel API cost.

The user can change this at any time, even mid-research (affects the next batch).

### Security

- **API key never leaves the browser** except via direct HTTPS calls to the LLM provider's API.
- **No telemetry, no analytics, no tracking.** The app has zero external requests except: LLM API calls, CORS proxy (for job platforms), and CDN loads for optional libraries.
- **CV data stays local.** The raw CV text is sent to the LLM for parsing, but the file itself never leaves the browser.
- Users are informed: _"Your CV content will be sent to the LLM API for parsing. Nothing is stored on intermediate servers (we don't run any)."_

---

## 9. Development Phases

### Phase A — Core Shell (mirrors current job-tracker)
Build `index.html` with the same UI skeleton, styling, card system, and filter logic as the current `job-tracker.html`. This gives us a working job board with static data.

### Phase B — Data Layer
Implement JSON file management: load/save/export `profile.json`, `preferences.json`, `platforms.json`, `jobs.json`, `state.json`. Add localStorage persistence for quick state.

### Phase C — AI Integration
Add API key config, provider selection, and all prompt templates. Implement CV parsing, profile generation, preference interview, and platform suggestion.

### Phase D — Research Engine
Implement the research pipeline: query construction, web fetching via proxy, job extraction, deduplication, match scoring, and real-time board population. Wire up `rules.md` as the driving prompt.

### Phase E — Refinement & Polish
Add the refinement bar, sort options, assist button enhancements, export/import, and responsive breakpoints.

---

## 10. Resolved Decisions

| Question | Decision |
|---|---|
| LLM provider | No specific provider. App is LLM-agnostic with configurable endpoint, model, and format (OpenAI-compatible + Claude-native). |
| Web proxy | Both public proxy (default, zero setup) and local proxy (`proxy.py`, more private). User-selectable in settings. |
| CV file size max | 10 MB. |
| Match score threshold | 60 (configurable by user). |
| Research concurrency | User-configurable: sequential, balanced (2x), or aggressive (all-at-once). Settable in settings at any time.
