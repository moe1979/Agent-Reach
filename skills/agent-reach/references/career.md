# LinkedIn / Career Intelligence

Use this reference whenever the user explicitly asks for LinkedIn, LinkedIn profiles, companies on LinkedIn, people search, company employees, jobs, hiring posts, or LinkedIn company intelligence.

## Mandatory routing rule

For an explicit LinkedIn request, **LinkedIn MCP is the primary source**. Do not start with Exa, Google, Bing, generic curl scraping, GitHub, Python scripts, or model memory.

On Hermes/Windows, use the installed `mcporter` command directly from PATH. Do not reconstruct executable paths and do not install another LinkedIn or Agent Reach package.

**There is no `scripts/search.js` or generic Agent Reach LinkedIn script in this Hermes skill. Never try to run one.** LinkedIn operations are direct `mcporter call linkedin.*` commands.

### First-choice commands

```text
mcporter list linkedin
mcporter call linkedin.get_my_profile
mcporter call linkedin.get_person_profile linkedin_username="username"
mcporter call linkedin.get_person_profile linkedin_username="username" sections="experience,education,skills"
mcporter call linkedin.search_people keywords="AI engineer" location="Germany"
mcporter call linkedin.search_companies keywords="predictive maintenance medical imaging"
mcporter call linkedin.get_company_profile company_name="company-slug"
mcporter call linkedin.get_company_profile company_name="company-slug" sections="posts,jobs"
mcporter call linkedin.get_company_employees company_name="company-slug"
mcporter call linkedin.get_company_employees company_name="company-slug" keywords="AI"
mcporter call linkedin.search_jobs keywords="predictive maintenance" location="Germany" max_pages=2
mcporter call linkedin.get_job_details job_id="JOB_ID"
mcporter call linkedin.search_posts keywords="predictive maintenance medical imaging" date_posted="past-month" max_pages=2
mcporter call linkedin.get_company_posts company_name="company-slug"
```

## Company-research workflow

When the user asks to find N LinkedIn companies relevant to a topic:

1. Run `linkedin.search_companies` with one semantically useful query.
2. If fewer than N plausible candidates appear, run at most 2 additional targeted LinkedIn queries using distinct domain terms. Do not switch to Google/Bing merely to fill N.
3. Deduplicate candidates by canonical LinkedIn company URL/slug.
4. For each plausible candidate, call `linkedin.get_company_profile` using the returned slug/URL. Request `posts` or `jobs` only when needed to prove relevance.
5. Apply the Evidence Gate below before including the company.
6. Stop when N verified companies are found or the candidate pool is exhausted. If fewer than N pass, return the smaller verified set and state that LinkedIn evidence did not support additional entries.

## Strict Evidence Gate

A company may be labeled **Verified Relevant** only when retrieved LinkedIn evidence supports BOTH sides of the user's requested intersection.

For a query such as "AI-based predictive maintenance for medical equipment or medical imaging", require:

- **Domain evidence:** LinkedIn profile/posts/jobs explicitly support medical devices, medical equipment, healthcare equipment, diagnostic imaging, radiology, MRI, CT, X-ray, ultrasound, or a comparably direct medical-technology domain; AND
- **Capability evidence:** LinkedIn profile/posts/jobs explicitly support predictive maintenance, condition monitoring, equipment-health monitoring, failure prediction, anomaly detection for equipment, asset monitoring, digital twins for equipment health, reliability analytics, or a comparably direct maintenance/reliability capability.

Merely having "AI", "machine learning", "healthcare", "medical scribe", "computer vision", "IoT", "data annotation", or "medical imaging" is **not sufficient** to infer predictive maintenance.

Merely having predictive maintenance for automotive/industrial assets is **not sufficient** to infer medical-equipment predictive maintenance.

### Evidence labels

Classify each candidate internally as one of:

- `VERIFIED_DIRECT` — both domain and capability are explicitly supported by LinkedIn evidence.
- `VERIFIED_ADJACENT` — one side is explicit and the other is strongly adjacent but not explicit. May be shown only in a clearly separated "Adjacent / needs verification" section, never counted toward the requested verified total.
- `REJECTED` — evidence does not support the requested intersection.

Do not expose internal scoring mechanics unless useful, but preserve the distinction in the final wording.

## Claim discipline

For every included company:

- `What they do` must paraphrase retrieved LinkedIn evidence, not general knowledge.
- `Why relevant` must identify the exact retrieved evidence connecting it to the user's requested topic.
- LinkedIn URL must come from the MCP result; never synthesize it from a company name.
- Quantitative claims are allowed only when the retrieved LinkedIn content itself contains the number.
- Do not claim "actively developing", "market leader", "#1", market share, installed base, accuracy, downtime reduction, customer count, or similar unless explicitly retrieved.
- Do not transform a weak adjacent capability into a direct claim. For example, medical-image annotation does not equal predictive maintenance.

## People-research workflow

1. Use `linkedin.search_people` for discovery.
2. Use `linkedin.get_person_profile` for details on selected results.
3. Request heavy sections such as posts or certifications separately when needed.
4. Do not expose contact information unless the user's task specifically requires it and the tool returned it legitimately.

## Evidence and URL rules

- Never invent a LinkedIn profile/company/job URL.
- Never claim an entity was "verified on LinkedIn" unless LinkedIn MCP returned it in this run.
- Deduplicate by canonical LinkedIn URL/slug before counting results.
- If LinkedIn MCP returns no useful results or errors, report the failure and then use the fallback below. Do not silently replace LinkedIn research with model memory.

## Read-only safety boundary

Default to read/search operations. Connection requests and messages are write operations; do not invoke them unless the user explicitly requests that action and confirms the intended recipient/content when required.

Reading conversations can mark items as read. Do not use inbox/conversation tools unless the user's task explicitly requires messages.

## Authentication

Preferred Windows setup uses a persistent browser profile explicitly created by the user:

```text
uvx mcp-server-linkedin@latest --login --no-headless --no-auto-import
uvx mcp-server-linkedin@latest --status --no-auto-import
```

Typical mcporter registration:

```text
mcporter config add linkedin --command uvx --arg "mcp-server-linkedin@latest" --arg "--no-auto-import" --scope home
```

Do not auto-import browser sessions when explicit profile isolation is configured.

## Fallback

Use Jina Reader only for a **known public LinkedIn URL** when LinkedIn MCP is unavailable or fails:

```text
curl.exe -sL "https://r.jina.ai/https://www.linkedin.com/in/username/"
curl.exe -sL "https://r.jina.ai/https://www.linkedin.com/company/company-slug/"
```

Jina Reader is a public-page fallback, not a substitute for authenticated LinkedIn search. Google/Bing scraping, guessed LinkedIn URLs, and hand-written Python company lists are not acceptable substitutes for a request explicitly scoped to LinkedIn.
