# CarePulse - Diabetes Safety & Doctor Consultation Companion

**MBA ZG583 - Management of AI Products | Experiential Learning Assignment | Team 002 | BITS Pilani WILP**

CarePulse is an AI-assisted outpatient companion for adults with Type 2 Diabetes Mellitus (T2DM) who manage complex medication regimens at home. It helps patients understand their medicines, record possible side effects safely, and arrive at follow-up consultations with a structured summary their clinician can review in under a minute.

> **Intended use:** CarePulse is an educational and communication companion only. It does **not** diagnose, assess causality, recommend treatment, or tell anyone to start, stop, substitute or change the dose of a medicine. All clinical decisions remain with the treating clinician.

---

## The problem

- Diabetes prevalence among Indian adults aged 45+ is about 19.8% (~50.4 million people), and most also manage comorbidities such as hypertension, dyslipidemia and CKD.
- Polypharmacy drives non-adherence, drug-related problems and unguided discontinuation when patients fear side effects.
- Existing apps are mostly reminders or static logs. Information recorded at home does not reach the clinician in a usable form, so consultation time is spent reconstructing medication history.

## The product - three modules

| Module | What it does |
| --- | --- |
| **1. Schedule Builder & Plain-Language Guide** | Converts supported prescriptions and fixed-dose combinations (FDCs) into a daily visual schedule and explains each medicine's ingredients and purpose in plain language. |
| **2. Symptom Tracker & Summarizer** | Captures symptoms with onset, severity and duration; maps free text to a controlled vocabulary; shows *possible* discussion points without ranking or assigning causality. |
| **3. Pre-Consultation Safety Snapshot** | Consolidates adherence, missed doses, supplements, concerns and patient questions into a traceable summary for clinician review. |

---

## Architecture: hybrid rules + retrieval + language model

| Layer | Used for | Why |
| --- | --- | --- |
| **Deterministic rules** | Catalogue lookup, ingredient mapping, schedule generation, duplicate-ingredient check, red-flag phrase screen | Safety-critical; must be reproducible |
| **Retrieval** | Medication guidance from the validated catalogue | Grounds explanations in reviewed sources, not model memory |
| **Language model (API)** | Symptom normalisation, discussion-point phrasing, snapshot condensation | Free text is too varied for rules; model selects from validated content, never originates clinical facts |

**Safety chain (in order):**
1. Red-flag screen runs *before* any model step. On a match, routine processing is bypassed and fixed, pre-approved urgent-care copy is shown.
2. Unmapped medicines are recorded, flagged as unverified and excluded from all AI-supported analysis.
3. Normalisation can return only terms from the curated lay-to-MedDRA vocabulary.
4. Output guardrail screens for diagnosis, start/stop/substitute/dose-change language and unsupported causal claims; matches are suppressed and replaced with static content.
5. Every snapshot line must trace to a source record; unsupported lines are removed, not rewritten.

**Uncertainty labels** (no numeric confidence scores anywhere): `Verified information` / `Possible discussion point` / `Insufficient information`.

**Languages (MVP):** English, Hindi, Telugu, including code-mixed and transliterated input. Safety text is pre-translated and reviewed, never generated at run time.

---

## Repository contents

> Adjust file names below to match the repo.

```
.
|-- README.md
|-- prototype/
|   `-- carepulse_prototype.html        # Self-contained functional prototype
|-- docs/
|   |-- MAIP_TEAM002_PM_Artifact_Final.docx
|   |-- CarePulse_Workflows_Measures_Roadmap   # 6 workflows, 24 measures, version gates
|   `-- cost_model.xlsx                 # Cost, pricing and break-even model
|-- diagrams/                           # Mermaid sources for all swimlane diagrams
|   |-- d1.mmd   # WF-1 Medication setup and schedule
|   |-- d2.mmd   # WF-2 Concern capture to discussion point
|   |-- d3.mmd   # WF-3 Catalogue and duplicate-ingredient check
|   |-- d4.mmd   # WF-4 Snapshot and clinician review
|   |-- d5.mmd   # WF-5 Catalogue and terminology governance
|   `-- d6.mmd   # WF-6 Post-launch monitoring and drift
`-- screenshots/                        # Wireframes, prototype paths, metrics view
```

---

## Running the prototype

No install, account, server or network connection needed.

1. Download `prototype/carepulse_prototype.html`.
2. Open it in any modern browser.

All data is synthetic. Language-model steps are **simulated by a scripted rule engine (Wizard-of-Oz)**, declared on the page.

### Scripted scenario paths

| Path | Scenario | Expected behaviour |
| --- | --- | --- |
| **A** | Mapped, interpretable concern | Normalised term, all candidate medicines listed with start dates, unranked, possible-not-proven wording |
| **B** | Medicine outside the catalogue | Recorded, flagged "Not verified", excluded from analysis |
| **C** | Uninterpretable concern | Original wording retained verbatim, no discussion point generated, carried to clinician |
| **D** | Red-flag concern | Routine processing bypassed before the language step; fixed urgent-care guidance; no medicine named as cause |

Every output carries a **safety trace** naming the layer that produced or blocked it (catalogue, red-flag screen, language step, retrieval, guardrail). The clinician view includes the snapshot, a Relevant / Not relevant control per discussion point, and a live metrics panel.

### What the prototype does not prove

Real model behaviour on free text at scale; mapping accuracy beyond the six loaded regimens; lay-to-MedDRA performance against a gold set; clinician usefulness in a real clinic; privacy, security or regulatory readiness; cost at volume.

---

## MVP scope

- **15 literature-derived regimen patterns** (Embase search of Indian real-world and prescribing studies) covering DPP-4i, SGLT2i, sulfonylurea, TZD and metformin combinations plus ARB/telmisartan-based antihypertensive therapy. The prototype loads 6 of these.
- Only exact product records that pass source verification, independent review and the M-01 test gate become active catalogue entries.
- **Out of scope:** diagnosis, causality assessment, dose or regimen changes, treatment recommendations, EHR integration, predictive analytics, emergency monitoring.

## Configuration model

All clinical content (catalogue, mappings, synonyms, red-flag patterns, prompts, model choice, languages, targets) is versioned configuration, not code. Each item sits in a control tier:

| Tier | Examples | Gate |
| --- | --- | --- |
| **T1 Operational** | Targets, escalation routing, pricing | Second reviewer |
| **T2 Clinical** | Catalogue, ingredient mapping, MedDRA synonyms, prompts, model, languages | Named test set must pass first |
| **T3 Safety-controlled** | Red-flag list, unsafe-output screen | Deployed bundle, unreachable from any screen; full adversarial re-run |
| **T4 Scope policy** | Intended-use boundary | Not configurable; requires change of intended use |

---

## Key launch thresholds

| ID | Measure | Target |
| --- | --- | --- |
| M-01 | Regimen mapping accuracy | 100% (300 tests) |
| M-05 | Red-flag escalation, per language | 100% (150 tests each) - launch blocker |
| M-06 | Unsafe medication advice rate | 0% (600 adversarial prompts) - launch blocker |
| M-08 / M-09 | Summary accuracy / unsupported statements | >= 95% / 0% |
| M-14 | Medication schedule understanding | >= 80% (62 participants) |
| M-15 | Consultation preparedness (North Star) | >= 80% scoring 4-5 of 5 |
| M-17 | Clinician usefulness rating | >= 80% rating 4-5 of 5 |
| M-18 | Medication reconciliation time | >= 30% reduction (~4 min) |
| M-20 | Subgroup task success | No group > 15 points below overall |

Full definitions, methods and sample-size derivations are in `CarePulse_Workflows_Measures_Roadmap`.

---

## Roadmap

| Phase | Versions | Weeks | Outcome |
| --- | --- | --- | --- |
| 0 Discovery | v0.1 | Done | Problem, workflow, safety boundaries, prototype |
| 1 Evaluation MVP | v0.2 - v0.6 | 1-23 | Verified catalogue, terminology layer, 3 languages, guardrails, clinician workflow |
| 2 Usability evaluation | v0.7 | 24-28 | Understanding and preparedness evidence |
| 3 Pilot preparation | v0.9 | 29-34 | Privacy, security, DPDP clearance; governance signed |
| 4 Controlled pilot | v1.0 | 35-60 | 100 patients, 6 months, one site; first revenue |
| 5 Scale decision | v1.1 | 61-63 | Go / no-go on expansion |

**Recommendation:** Proceed to evaluation MVP development. Do not pilot or scale until safety, workflow and commercial gates are met.

## Business model (planning estimates, not validated)

- **Buyer:** hospitals, diabetes clinics and chronic-care programmes; patients pay nothing.
- **Proposed test price:** Rs 132-179 per enrolled patient-month by volume tier; pilot at Rs 4,50,000 for 100 patients / 6 months.
- **Funding to first revenue:** ~Rs 37.6 lakh (build + 8 months running cost).
- **Key insight:** inference is ~0.1% of running cost; clinical review, support and governance are the real cost drivers. The economic case depends on ~4 clinician minutes saved per consultation (break-even at 3.96).

---

## Team 002

| Name | BITS ID | GitHub | Role |
| --- | --- | --- | --- |
| Ramyatha Munnangi | 2025MB26036 | [@2025MB26036-glitch](https://github.com/2025MB26036-glitch) | Product Strategy and Research |
| Aishwarya M | 2025MB26172 | [@2025mb26172](https://github.com/2025mb26172) | Clinical Workflow and Governance |
| Subhasree Dutta | 2025MB26011 | [@Subhashree7dutta](https://github.com/Subhashree7dutta) | UX and Prototype |
| Anushka Tripathi | 2025MB26168 | [@Atripa1803](https://github.com/Atripa1803) | PRD and Evaluation |
| Sumathi Senthil Kumar | 2025MB26068 | [@WonderWoman8](https://github.com/WonderWoman8) | Business and Scaling |

## AI collaboration

Claude (Anthropic) was used to code the functional prototype from the PM artifact, draft the symptom-terminology design, research free drug-reference APIs, verify references and format documents. Prototype code and design suggestions were checked against the PRD, intended-use boundary and safety rules. Scope, risk acceptance, thresholds, pricing and the final recommendation were decided by the team.

## Reference data sources

CDSCO approved product information; Indian Pharmacopoeia Commission (monographs, PvPI communications); Jan Aushadhi product list; RxNorm/RxNav (US NLM); openFDA; WHO ATC/DDD; MedDRA (ICH MSSO, licensed). Full literature references are in the PM artifact.

## Disclaimer

Academic project. All data shown is synthetic. Not a medical device, not validated for clinical use, and not for use with real patient data.
