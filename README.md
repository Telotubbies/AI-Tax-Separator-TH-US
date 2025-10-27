
# AI Tax Separator TH–US (MVP v0.1)
**Subtitle:** Cross‑Border Personal Tax Intelligence for Thai Residents & US‑Linked Investors  
**Author:** rabbit x ChatGPT (GPT‑5 Thinking)  
**Date:** 2025‑10‑28 (Asia/Bangkok)

---

## 1) Executive Summary (Pitch‑Ready)
**Problem:** Individuals in Thailand who invest in US securities (or receive USD income) face complex, error‑prone tax filings across **two jurisdictions** (Thailand & US). Statements (1099‑DIV/INT/B, broker CSV/PDF, PayPal/Wise) are heterogeneous; treaty rules and foreign tax credits (FTC) are hard to apply correctly; documentation to Thai Revenue Department (TRD) often requires manual reconciliation.

**Solution:** **AI Tax Separator TH–US** — an AI‑assisted, human‑in‑the‑loop platform that **ingests statements**, **classifies transactions**, **applies TH–US tax rules and treaty mapping**, **computes liabilities**, and **generates filing‑ready summaries (draft ภ.ง.ด.90/94 + FTC report)**. Users or advisors review before filing.

**Why Now:** Retail participation in US markets by Thai residents has surged; remote/freelance USD income is rising; regulators emphasize accuracy & cross‑border transparency (FATCA, CRS).

**Outcome:** Lower total tax *legally* through optimization (treaty/credit/structuring), fewer errors, faster filings, clear audit trail.

---

## 2) Personas & JTBD
- **P1 – Thai US‑Stock Investor (B2C):** Holds US shares/ETFs via international brokers; receives dividends; sells with gains/losses. *JTBD:* “Show me exactly what to report in Thailand, how much US tax is creditable, and produce a Thai filing summary.”
- **P2 – Expat Working/Living in Thailand (B2C):** US citizens/Green card holders; have worldwide income obligations. *JTBD:* “Aggregate Thai & US income, apply treaty, and compute residual US tax after FTC.”
- **P3 – CPA/Tax Advisor (B2B/SaaS):** Serves 50–500 clients; drowns in PDFs and CSVs. *JTBD:* “Automate the first 80% (ingest, classify, compute), so I can review and sign‑off the last 20%.”

---

## 3) Problem Detail (Pain Map)
1. **Data Fragmentation:** Broker formats vary (IBKR, Robinhood, TD, eToro, Binance). OCR of PDFs is noisy; CSV headers differ.
2. **Classification Ambiguity:** Dividends vs qualified dividends; short‑ vs long‑term gains; corporate actions (splits, DRIP) and fees.
3. **Jurisdictional Complexity:** Thai PIT ladder, Thai final WHT on domestic dividends (10% option), US capital gains/qualified dividend rates, residency tests, and FTC caps.
4. **FX Treatment:** Date‑based FX for income/withholding; TRD rate vs BOT daily; basis conversion for gain/loss.
5. **Documentation:** Drafting supporting schedules, withholding evidence, and FTC workpapers for both sides.
6. **Human Review:** Need for advisor sign‑off to ensure compliance & risk control.

---

## 4) Value Proposition
- **90% Faster** preparation via ingestion + auto‑classification.
- **Legally Lower Tax** using treaty/credit guidance (no evasion).
- **Fewer Errors** with deterministic rule engine + human verification.
- **Filing‑Ready Output** (Thai draft pack + FTC appendix).
- **Advisor Mode** to scale CPA throughput with consistent quality.

**Key Differentiators**
1. **TH–US Treaty Focus** (underserved niche) with granular rule mapping.
2. **Human‑in‑the‑Loop Workflow** (review/approve/recompute) built‑in.
3. **Statement Adapter Layer** covering popular brokers and e‑wallets.
4. **Explainable Tax Engine** (transparent steps, not a black box).

---

## 5) High‑Level Features
- Upload: PDF/CSV/XLSX for brokers and payment platforms.
- Parsing layer: OCR (PDF), schema mappers (CSV), lot‑matching (FIFO/Specific ID – roadmap).
- Classifier: Transaction typing (DIVIDEND, CAPITAL_GAIN/LOSS, INTEREST, FEE, WHT, ROYALTY).
- Tax Engine: Thailand PIT ladder; Thai dividend options; US simplified rates; **FTC** calculator; **treaty** lookups.
- FX Engine: Daily USD/THB rates; policy toggle (BOT/TRD/avg).
- Reports: **Draft ภ.ง.ด.90/94 summary**, dividend annex, gain/loss schedule, FTC workpaper (US side), audit trail.
- Advisor dashboard: Workqueues, status (Needs‑Data/Needs‑Review/Approved), comments, versioning.
- Security & Compliance: Encryption, PII minimization, audit logs, role‑based access (USER/ADVISOR/ADMIN).

---

## 6) System Architecture (MVP)
```
[Frontend: Vue+Vite] ───► [/api FastAPI]
      │                         │
      │  Upload CSV/PDF         │  Celery jobs (OCR/Parse/FX/Compute)
      ▼                         ▼
  UX + Review UI           [Tax Engine + Rule DB]
      │                         │
      │  PDFs & Summaries       │  Postgres (transactions, lots, withholdings)
      ▼                         ▼
 [WeasyPrint/HTML->PDF] ◄──►  [Redis] (queue)  ──► [S3/minio for files]*
                                     │
                                  [Nginx] ──► Cloudflare (SSL/WAF)
```
\* Optional object storage for statements and generated PDFs.

---

## 7) Technology Stack
**Frontend:** Vue 3, Vite, Pinia, Tailwind, Axios  
**Backend:** FastAPI (Python 3.11), Pydantic v2, SQLAlchemy 2, Uvicorn/Gunicorn  
**Async:** Celery + Redis  
**DB:** PostgreSQL (prod) / SQLite (dev quick)  
**OCR/Parsing:** Tesseract OCR (PDF), Pandas (CSV/XLSX), Broker adapters  
**Docs:** Jinja2 + WeasyPrint (HTML → PDF)  
**Infra:** Docker Compose, Nginx reverse proxy, Cloudflare (Full/Strict), CI/CD (GitHub Actions – roadmap)  
**Security:** JWT (fastapi‑users), at‑rest DB encryption (pgcrypto/extension), secrets in env, audit logs  
**Observability (opt.):** Prometheus + Grafana, Sentry

---

## 8) Data Model (Core Entities)
- **User**(id, email, hashed_password, role)
- **Account**(id, user_id, broker, currency)
- **Transaction**(id, account_id, occurred_at, symbol, type, qty, gross_amount, local_amount_thb, country_source, note)
- **Withholding**(id, txn_id, country, amount)
- **TaxYearSummary**(id, user_id, year, totals..., foreign_tax_paid_thb, thai_tax_due_thb, us_tax_due_thb)
- **FxRate**(date, usd_thb) *(engine table)*
- **AuditLog**(who, when, action, entity, before/after hash)

**Transaction Types:** `DIVIDEND, CAPITAL_GAIN, CAPITAL_LOSS, INTEREST, FEE, WITHHOLDING_TAX, ROYALTY, OTHER`

---

## 9) Statement Adapter Layer
**CSV/PDF Mappers (v0.1):**
- **IBKR** (Activity/Tax statements)
- **Robinhood**, **TD Ameritrade**, **eToro** (essentials)
- **PayPal/Wise** (income transfers)
- **1099‑DIV/INT/B** (US forms – OCR + template cues)

**Mapping Strategy:**
- Header dictionary per provider
- Row‑wise type inference rules (keyword + amount sign + symbol context)
- Corporate action handlers (split, reverse split; DRIP – roadmap)
- Reconciliation: sum(dividends) vs broker annual totals (tolerance ε)

---

## 10) Tax Logic Overview (Simplified for MVP)
### Thailand
- **PIT Ladder (progressive)** for ordinary income.  
- **Dividends (Domestic TH):** Option A include in PIT base; Option B **final WHT 10%** (commonly chosen).  
- **Foreign income timing rule:** If remitted in same year vs later year (policy toggle – roadmap).  
- **Capital gains:** On foreign securities — filing under “other income” per TRD guidance (exact category configurable).  
- **Withholding evidence:** Record `WITHHOLDING_TAX` lines; attach proof.

### United States (Simplified)
- **Residency toggle** (US resident / non‑resident / US citizen abroad).  
- **Qualified dividend & LTCG rate** default 15% for MVP (configurable by bracket).  
- **Foreign Tax Credit (FTC):** `FTC = min(US_tax_due_on_foreign_income, foreign_tax_paid_converted_THB→USD)`.

### Treaty (TH–US)
- WHT reference defaults: Dividend **10%**, Interest **15%**, Royalty **15%** (tune per specific facts).  
- Apply treaty cap when source is Thailand & recipient is US‑linked taxpayer; otherwise standard rules.

> ⚠️ **Note:** MVP logic is intentionally simplified; production requires complete bracket tables, residency tests, qualified‑dividend rules, and Thai remittance timing rules.

---

## 11) FX Engine
- Source: Bank of Thailand daily USD/THB (fallback policy: average/monthly).  
- Apply **transaction‑date FX** to income & withholding; also compute year‑end recon.  
- Policy toggles: TRD rate vs BOT daily; “spot on date” vs “monthly average”.

---

## 12) Human‑in‑the‑Loop Workflow
1. **User Uploads Files** → Ingestion job starts.  
2. **AI Classifies** → Confidence scores per row.  
3. **Advisor Review Screen** → filter by `confidence<0.8`, bulk approve/edit.  
4. **Compute Tax** → Engine runs; show breakdown & explanations.  
5. **Generate PDFs** → Draft pack (Thai summary, FTC worksheet, transaction annex).  
6. **User/Advisor Sign‑Off** → lock version; export pack; e‑file preparation (CSV/XML – roadmap).

---

## 13) API Surface (MVP)
```
POST /api/upload/csv      -> queue parse; returns job_id
POST /api/upload/pdf      -> queue OCR; returns job_id
GET  /api/transactions    -> list/filter (year, type, symbol)
POST /api/tax/compute     -> {year, salary_thb, dividend_thb, gain_thb, loss_thb, foreign_tax_paid_thb}
GET  /api/report/summary  -> returns PDF (draft ภ.ง.ด.90/94 summary)
GET  /api/report/ftc      -> returns PDF FTC worksheet
```
Auth: JWT; Roles: USER / ADVISOR / ADMIN

---

## 14) Security & Compliance
- **PII Minimization:** store only essential fields; mask where possible.  
- **Encryption:** TLS (Cloudflare Full/Strict), DB at‑rest encryption (pgcrypto), encrypted file storage.  
- **Auditability:** action logs; versioned computations; checksum of outputs.  
- **Access Control:** role‑based, least privilege; advisor‑client binding.  
- **Data Residency:** choose TH DC (if using cloud); configurable retention.  
- **Legal:** This software provides computational assistance, **not legal/tax advice**; final responsibility after human review.

---

## 15) Output Pack (Draft for Filing)
- **Thai Annual Summary (ภ.ง.ด.90/94 – draft)**: salary/other income, net gain/loss, dividend option chosen, foreign tax paid, Thai tax due.  
- **Dividend Annex:** per symbol & date; withheld tax evidence refs.  
- **FTC Worksheet (US):** foreign income by category, foreign taxes paid, limitation calc.  
- **Broker Reconciliation:** broker totals vs system totals (diff ε).  
- **Audit Log Appendix** (optional).

---

## 16) Roadmap (Quarterly)
- **Q1 – MVP:** CSV/PDF ingestion, core engine, draft PDFs, single user.  
- **Q2 – Advisor Mode:** multi‑client, comments, bulk approval, stronger US rules.  
- **Q3 – FX/Lots:** BOT integration, FIFO/Specific ID, corporate actions.  
- **Q4 – e‑Filing:** TRD export templates (CSV/XML), integrations with accountant tools.

---

## 17) KPIs
- Time‑to‑prepare (TTPrep) per user/file.  
- Auto‑classification accuracy (top‑1).  
- % rows requiring manual review.  
- Reduction in total tax **legally** (vs baseline self‑prep).  
- Advisor throughput (files per day per reviewer).

---

## 18) Business Model & Pricing (Draft)
- **B2C:** THB 1,490/year (Standard), THB 2,990/year (Pro with advisor chat).  
- **B2B (Advisors):** THB 990/client/year (volume tiers).  
- **Add‑ons:** White‑label API for fintech/banks; custom adapters.

---

## 19) Risks & Mitigations
- **Regulatory drift:** keep rule packs versioned; update cadence pre‑season.  
- **Data sensitivity:** encryption, zero‑trust access, breach response playbook.  
- **Parsing noise:** hybrid OCR + schema hints + human review threshold.  
- **Edge cases:** explicit override UI and annotation for training.

---

## 20) Go‑to‑Market (Thailand First)
- Partnerships with **CPA firms** and **wealth platforms**.  
- Community education: webinars “ยื่นภาษีรายได้ต่างประเทศอย่างถูกต้อง”.  
- Early‑adopter offer for US‑stock communities; referral program.

---

## 21) Minimal Dev Plan (Week‑by‑Week)
**W1:** Repo + Docker skeleton; DB schema; upload endpoints.  
**W2:** CSV parsers (IBKR, Robinhood); basic classifier.  
**W3:** Thai engine (PIT ladder + dividend 10% final option); FX mock.  
**W4:** PDF draft pack (WeasyPrint).  
**W5:** Advisor review UI + comments; confidence filter.  
**W6:** FTC v0; policy toggles; alpha pilot.  
**W7–8:** Hardening; monitoring; beta with 5 advisors.

---

## 22) Appendix – Example Computation (Illustrative)
**Inputs (THB):**  
- Salary: 0  
- Dividends: 120,000 (foreign)  
- Net capital gain: 200,000  
- Foreign tax paid (converted): 18,000

**Thai side (simplified):**  
- PIT on 200,000 gain ≈ 7,500 (ladder simplified)  
- Dividend option (final 10%): 12,000  
- **Thai payable ≈ 19,500**

**US side (simplified):**  
- US tax on (div + gain) at 15% ≈ (120,000+200,000)*0.15 = 48,000  
- **FTC = min(18,000, 48,000) = 18,000 → US payable ≈ 30,000**

> *Figures are illustrative; production engine will use exact brackets and residency rules.*

---

## 23) Legal & Ethical Note
This document is for **product design**. The software provides computational assistance and document generation; **final filing requires human review** (advisor/CPA). No unlawful tax evasion is supported.

