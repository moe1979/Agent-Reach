# LinkedIn / Career Intelligence

Use this reference whenever the user explicitly asks for LinkedIn, LinkedIn profiles, companies on LinkedIn, people search, company employees, jobs, hiring posts, or LinkedIn company intelligence.

## Mandatory routing rule

For an explicit LinkedIn request, **LinkedIn MCP is the primary source**. Do not start with Exa, Google, Bing, generic curl scraping, GitHub, Python scripts, or model memory.

On Hermes/Windows, use the installed `mcporter` command directly from PATH. Do not reconstruct executable paths and do not install another LinkedIn or Agent Reach package.

### First-choice commands

```text
# Discover/verify the configured LinkedIn MCP only when needed
mcporter list linkedin

# Authenticated user's own profile
mcporter call linkedin.get_my_profile

# Specific person
mcporter call linkedin.get_person_profile linkedin_username="username"
mcporter call linkedin.get_person_profile linkedin_username="username" sections="experience,education,skills"

# Search people
mcporter call linkedin.search_people keywords="AI engineer" location="Germany"

# Search companies by domain/topic
mcporter call linkedin.search_companies keywords="predictive maintenance medical imaging"

# Get a company profile; company_name should be the LinkedIn slug when known
mcporter call linkedin.get_company_profile company_name="company-slug"
mcporter call linkedin.get_company_profile company_name="company-slug" sections="posts,jobs"

# Company employees/demographics
mcporter call linkedin.get_company_employees company_name="company-slug"
mcporter call linkedin.get_company_employees company_name="company-slug" keywords="AI"

# Jobs
mcporter call linkedin.search_jobs keywords="predictive maintenance" location="Germany" max_pages=2
mcporter call linkedin.get_job_details job_id="JOB_ID"

# LinkedIn posts/content search
mcporter call linkedin.search_posts keywords="predictive maintenance medical imaging" date_posted="past-month" max_pages=2

# Company posts
mcporter call linkedin.get_company_posts company_name="company-slug"
```

## Company-research workflow

When the user asks to find N LinkedIn companies relevant to a topic:

1. Run `linkedin.search_companies` with a semantically useful topic query.
2. Select distinct company results returned by LinkedIn. Do not invent or normalize slugs from memory.
3. For each selected company, call `linkedin.get_company_profile` using the returned LinkedIn company slug/URL.
4. If relevance depends on recent activity, optionally call `linkedin.get_company_posts` or `linkedin.search_posts`.
5. Return only facts supported by the actual MCP results. If fewer than N genuinely relevant companies are returned, say how many were verified rather than padding the list.

## People-research workflow

1. Use `linkedin.search_people` for discovery.
2. Use `linkedin.get_person_profile` for details on selected results.
3. Request heavy sections such as posts or certifications separately when needed.
4. Do not expose contact information unless the user's task specifically requires it and the tool returned it legitimately.

## Evidence and URL rules

- Never invent a LinkedIn profile/company/job URL.
- Never claim a company or person was "verified on LinkedIn" unless the LinkedIn MCP returned that entity in this run.
- Never fabricate market share, customer counts, installed base, revenue, AI accuracy, downtime reduction, or other quantitative claims.
- A company being a medical-imaging manufacturer does not by itself prove it uses AI predictive maintenance. Relevance must be supported by returned company/about/posts/jobs content or clearly labeled as an inference.
- Deduplicate entities by canonical LinkedIn URL/slug before counting them toward the requested total.
- If LinkedIn MCP returns no useful results or errors, report the failure and then use the fallback below. Do not silently replace LinkedIn research with model memory.

## Read-only safety boundary

Default to read/search operations. The MCP also exposes write-capable actions such as connection requests and sending messages; do not invoke them unless the user explicitly requests that action and confirms the intended recipient/content when required.

Reading conversations can mark items as read. Do not use inbox/conversation tools unless the user's task explicitly requires messages.

## Authentication

The preferred Windows setup uses a persistent browser profile created explicitly by the user, for example:

```text
uvx mcp-server-linkedin@latest --login --no-headless --no-auto-import
uvx mcp-server-linkedin@latest --status --no-auto-import
```

A typical mcporter registration is:

```text
mcporter config add linkedin --command uvx --arg "mcp-server-linkedin@latest" --arg "--no-auto-import" --scope home
```

Do not auto-import browser sessions when the user has chosen explicit login/profile isolation.

## Fallback

Use Jina Reader only for a known public LinkedIn URL when LinkedIn MCP is unavailable or fails:

```text
curl.exe -sL "https://r.jina.ai/https://www.linkedin.com/in/username/"
curl.exe -sL "https://r.jina.ai/https://www.linkedin.com/company/company-slug/"
```

Jina Reader is a public-page fallback, not a substitute for authenticated LinkedIn search. Google/Bing scraping and hand-written Python company lists are not acceptable substitutes for a request explicitly scoped to LinkedIn.
