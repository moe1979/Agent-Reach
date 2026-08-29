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

## Company-research workflow — bounded execution

When the user asks to find N LinkedIn companies relevant to a topic, use this bounded workflow. The budgets below are **hard execution limits**, not suggestions.

### Phase A — discovery budget

Maintain an internal `discovery_query_count`, starting at 0.

1. Run **one** `linkedin.search_companies` query that expresses the full intersection of the user's criteria. Increment `discovery_query_count`.
2. Deduplicate returned candidates immediately by canonical LinkedIn company URL/slug.
3. Rank candidates by how directly their returned LinkedIn text appears to satisfy all material criteria.
4. If the first search does not produce enough plausible candidates for verification, run targeted expansion queries using distinct missing concepts.
5. **Normal hard limit: no more than 3 total `linkedin.search_companies` calls.** Once `discovery_query_count == 3`, do not call `search_companies` again during normal discovery.
6. Do not run near-duplicate keyword permutations merely to seek a "stronger" or "better" candidate after enough verified results exist.

### Phase B — candidate verification

1. Select the strongest candidate pool rather than profiling every search result. For a request for N companies, normally verify at most `max(N + 3, 5)` candidates in the first verification batch; cap the first batch at 8 candidates.
2. Call `linkedin.get_company_profile` for independent candidates **in parallel when the runtime supports parallel tool calls**.
3. Request `posts` or `jobs` only when the base profile is insufficient to determine the Evidence Gate. Do not fetch heavy sections by default.
4. Apply the Strict Evidence Gate below to each candidate.
5. Count only `VERIFIED_DIRECT` candidates toward N.

### HARD EARLY STOP

As soon as the number of `VERIFIED_DIRECT` candidates reaches the user's requested N:

**STOP RETRIEVAL IMMEDIATELY AND COMPOSE THE ANSWER.**

Do not:

- search for stronger alternatives;
- search for a better Nth candidate;
- continue discovery to build a reserve list;
- profile additional companies already queued but not needed;
- add adjacent companies when N direct companies are already verified.

The user's requested count is a completion target, not a minimum invitation to continue researching.

### One final expansion round — only when needed

If the normal 3-query discovery budget plus first verification batch yields fewer than N `VERIFIED_DIRECT` companies:

1. You may perform **one final expansion round** consisting of **at most 2 additional `linkedin.search_companies` calls** targeted specifically at the missing criterion or equipment/domain term.
2. Deduplicate against every candidate already seen.
3. Verify only newly discovered plausible candidates, preferably in one parallel batch.
4. Apply the Evidence Gate.
5. Then **STOP regardless of count**. Return the verified subset and state that LinkedIn evidence did not support enough additional direct matches.

Therefore the absolute company-discovery ceiling for one user request is normally **5 `linkedin.search_companies` calls total**: 3 normal + 2 final-expansion calls. Reaching this ceiling requires that fewer than N direct candidates were verified after the normal phase.

### No padding

If the workflow ends with fewer than N verified companies, return fewer than N. Never weaken the Evidence Gate, reinterpret clinical prediction as equipment PdM, or use generic web/model memory merely to fill the requested number.

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
