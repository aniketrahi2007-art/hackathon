# hackathon
# 🔄 Complete End-to-End Pipeline (How the System Works)

This project implements a **controlled auto-apply agent pipeline**.  
Below is the full workflow from user input to job application attempts, step by step.

---

## STEP 0 — System Setup (One-time)

Before any run:
- Google Sheets acts as the **database**
- Each tab represents a logical table
- Google Apps Script is deployed as a **Web App**
- Colab (or UI) triggers backend execution

No background workers or cron jobs are assumed.

---

## STEP 1 — Candidate Profile Creation (`STUDENT_PROFILE`)

**Input source:** Web UI or API  
**Output:** A single candidate record

User provides:
- Name
- Email
- Location
- Remote preference
- Start date

Backend behavior:
- Generates a unique `student_id` (primary key)
- Normalizes profile into a structured object
- Stores it in `STUDENT_PROFILE`
- This `student_id` becomes the join key across the entire system

📌 Nothing else runs unless a valid `student_id` exists.

---

## STEP 2 — Skills & Evidence Capture (`BULLET_BANK`)

**Input source:** Web UI or API  
**Output:** Multiple skill bullet rows

User adds skills as short natural-language bullets, e.g.:
- “Python automation using APIs”
- “Backend scripting and data pipelines”

Backend behavior:
- Each bullet is stored as its own row
- Bullets are never hallucinated or generated
- These bullets are later used for keyword matching

📌 If no explicit skill list exists, skills are **derived from bullet text** automatically.

---

## STEP 3 — Job Dataset (`JOBS`)

**Input source:** Preloaded dataset (manual / scraped / imported)

Each job contains:
- Job ID
- Company
- Title
- Location
- Remote flag
- Requirements text
- Description text

This table is **read-only** during execution.

---

## STEP 4 — Policy Loading (`APPLY_POLICY`)

Before matching begins, policy rules are loaded:

Policy controls include:
- Kill switch (stop all applications instantly)
- Minimum match score
- Max applications per run
- Blocked companies
- Blocked role keywords

📌 These rules apply globally and override all other logic.

---

## STEP 5 — Auto-Search & Matching Engine

**Triggered by:** API / UI / Colab  
**Function:** `run_autosearch_apply`

For each job:
1. Extract keywords from requirements + description
2. Remove stopwords and noise
3. Measure keyword coverage against:
   - Candidate skills
   - Bullet text
4. Compute a **match score (0 → 1)**

Score adjustments:
- Bonus if job is remote and candidate allows remote
- Bonus if job location matches candidate location

Each job is assigned:
- Match score
- Decision (`selected` or `skipped`)
- Reason for decision

---

## STEP 6 — Ranked Queue Generation (`APPLY_QUEUE`)

All jobs (selected and skipped) are written to `APPLY_QUEUE` with:
- Match score
- Decision
- Reason
- Timestamp

This creates a **fully explainable ranking layer**.

📌 Nothing is hidden — even skipped jobs are logged.

---

## STEP 7 — Auto-Apply Execution (Optional)

If auto-apply is enabled:
- Top N selected jobs (based on policy limits) are attempted
- Each attempt builds a structured “application package”
- The package contains:
  - Candidate facts
  - Match score
  - Metadata (no fabricated claims)

Applications are attempted sequentially.

---

## STEP 8 — Application Logging (`APPLICATIONS`)

Every attempt is logged:
- Successful submissions → status = `submitted`
- Failures → status = `failed` with reason

Failure reasons may include:
- Simulated rate limits
- Temporary failures
- Policy blocks

📌 This table acts as the **audit trail**.

---

## STEP 9 — Post-Run Summary

After execution, the system can return:
- Number of jobs scored
- Number selected
- Number attempted
- Number submitted
- Number failed
- Latest application receipts

This enables monitoring and debugging.

---

## Key Design Principles

- **Explainability:** Every decision has a reason
- **Control:** Policy layer overrides automation
- **Auditability:** Every action is logged
- **Safety:** Kill switch + caps prevent runaway behavior
- **Modularity:** Apply logic can be replaced with real integrations

---

## What This Demonstrates

This pipeline demonstrates:
- A complete agent loop (profile → match → decide → act → log)
- Responsible automation patterns
- Clear separation of data, logic, and policy
- A system that can be extended to real-world integrations

This is not a mock UI — it is a working execution pipeline.
