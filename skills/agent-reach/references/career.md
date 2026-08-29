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
mcporter call linkedin.search_people keywords="AI engineer" location="Germany"
mcporter call linkedin.search_companies keywords="predictive maintenance medical imaging"
mcporter call linkedin.get_company_profile company_name="company-slug"
mcporter call linkedin.get_company_profile company_name="company-slug" sections="posts,jobs"
mcporter call linkedin.get_company_employees company_name="company-slug"
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

A company may be labeled **Verified Relevant** only when retrieved LinkedIn evidence supports BOTH the requested domain and the requested operational capability.

For **predictive maintenance (PdM) of medical equipment or medical imaging equipment**, require:

- **Medical-equipment domain evidence:** the retrieved LinkedIn profile/posts/jobs explicitly concern physical medical devices/equipment or imaging equipment such as MRI, CT, X-ray, ultrasound, PET/SPECT, radiotherapy systems, patient monitors, ventilators, pumps, laboratory analyzers, or comparable maintainable healthcare assets; AND
- **Equipment-maintenance evidence:** the retrieved LinkedIn content explicitly concerns predicting, detecting, or preventing degradation/failure of the **equipment, component, subsystem, or asset itself**. Valid concepts include predictive maintenance, condition-based maintenance, equipment health, machine health, component failure prediction, remaining useful life (RUL), anomaly detection on equipment telemetry, service prediction, remote equipment diagnostics, reliability analytics, uptime/downtime optimization, digital twins for asset health, or maintenance scheduling based on equipment condition.

### Critical semantic exclusion: PdM is not clinical prediction

**Predictive maintenance is an asset/equipment reliability concept. It is NOT prediction about a patient, disease, diagnosis, or clinical outcome.**

The following must NOT be counted as equipment PdM unless separate retrieved evidence explicitly connects them to equipment/component health or maintenance:

- disease-risk prediction;
- early diagnosis or early detection of disease/inflammation;
- preventive healthcare;
- patient deterioration prediction;
- cardiac-event prediction;
- ECG/EEG/patient monitoring analytics;
- medical image interpretation, segmentation, classification, reconstruction, or diagnostic AI;
- PACS/DICOM analytics aimed at diagnosis;
- clinical decision support;
- medical scribe/EMR/RCM AI;
- computer vision for diagnosis;
- data annotation for medical AI;
- predictive analytics whose prediction target is a patient or clinical outcome.

**Test the prediction target:** ask "What is being predicted?" If the answer is patient/disease/diagnosis/outcome, it is clinical prediction and must be rejected for an equipment-PdM query. If the answer is equipment failure/degradation/component health/service need/remaining life/downtime, it may qualify.

Likewise, predictive maintenance for automotive, aerospace, manufacturing, energy, or generic industrial assets does not qualify merely because the same company also has a medical-imaging business. Retrieved evidence must connect maintenance/reliability capability to medical equipment or a clearly applicable medical-device offering.

### Evidence labels

Classify each candidate internally as:

- `VERIFIED_DIRECT` — explicit medical-equipment domain + explicit equipment-health/maintenance prediction evidence.
- `VERIFIED_ADJACENT` — relevant medical-equipment company with reliability/monitoring capability that is plausible but not explicitly tied to PdM in retrieved LinkedIn evidence, or an equipment-PdM company whose medical-device applicability is not explicit. Show only under "Adjacent / needs verification" and never count toward the verified total.
- `CLINICAL_PREDICTION_NOT_PDM` — predicts disease, patient risk, diagnosis, imaging findings, or clinical outcomes rather than equipment health. Reject for equipment-PdM requests.
- `REJECTED` — otherwise unsupported.

## Claim discipline

For every included company:

- `What they do` must paraphrase retrieved LinkedIn evidence, not general knowledge.
- `Why relevant` must state the retrieved evidence connecting the company to **equipment reliability/maintenance**, not merely AI or healthcare.
- LinkedIn URL must come from the MCP result; never synthesize it.
- Quantitative claims are allowed only when retrieved LinkedIn content contains the number.
- Do not claim "actively developing", "market leader", "#1", market share, installed base, accuracy, downtime reduction, customer count, or similar unless explicitly retrieved.
- Medical-image annotation does not equal predictive maintenance.
- Disease prediction does not equal predictive maintenance.
- Preventive medicine does not equal preventive/predictive equipment maintenance.

## People-research workflow

1. Use `linkedin.search_people` for discovery.
2. Use `linkedin.get_person_profile` for selected results.
3. Request heavy sections separately when needed.
4. Do not expose contact information unless specifically required and legitimately returned.

## Evidence and URL rules

- Never invent a LinkedIn profile/company/job URL.
- Never claim an entity was "verified on LinkedIn" unless LinkedIn MCP returned it in this run.
- Deduplicate by canonical LinkedIn URL/slug before counting results.
- If LinkedIn MCP returns no useful results or errors, report the failure and then use the fallback below. Do not silently replace LinkedIn research with model memory.

## Read-only safety boundary

Default to read/search operations. Connection requests and messages are write operations; do not invoke them unless the user explicitly requests that action and confirms intended recipient/content when required.

Reading conversations can mark items as read. Do not use inbox/conversation tools unless explicitly required.

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
