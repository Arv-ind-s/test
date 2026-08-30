# spec.md — Covenant Radar

**AI-Powered Dynamic Covenant Monitoring & Early Warning · Commercial Banking · Credit Risk**
Version 0.1 · 2026-08-29 · written at T+0 for hand-over at T+3. Product of a 24-hour AI hackathon global final. Sections §0–§31; IDs used: `R-nn` (functional requirements), `N-nn` (non-functional), `R-nn.a` (checks), `F-nn` (flows), `NG-nn` (not building), `RISK-nn` (risks), `[OPEN-nn]` (open questions, §27), `[ASSUMED-nn]` (assumptions, §12), `LATER:` (real-product deferrals), outcome labels `MEASURED` / `USER-TESTED` / `ESTIMATED` / `PROJECTED`.

---

## §0 Summary

Covenant Radar is for the relationship, credit and risk teams watching corporate loan accounts in Indian banks. Today covenants are checked when the borrower's quarterly statements arrive, so a company can slide for months while its file still says compliant. The slide surfaces late, when choices have narrowed and recovery is poorest — lenders took a 67% haircut on claims resolved under insolvency till September 2025. Covenant Radar watches the account daily and warns 30, 60 and 90 days before a covenant is likely to break, with evidence attached.

It works in ordered stages, and most are plain computation, not AI. At setup a language model reads the covenant text, but code re-verifies its proposal and a person confirms it before it counts. From then on, code computes the ratios, tests each covenant, scores daily payment, utilisation and cash signals as sustained or noise, and projects each covenant's headroom to a dated breach risk with named drivers. The model writes only the final warning memo, from those computed facts. The user walks away with a ranked watchlist, a committee-ready memo, and an audit trail that reconstructs every warning.

The rest of the page, briefly:

- **Business case served:** the single case in `problem.md` — AI-driven covenant breach early warning over 30/60/90-day horizons for commercial lending (§1).
- **Why we think it works:** SMA-2 stress is a documented lead indicator of NPA slippage about two quarters ahead, and manual quarterly monitoring is the documented gap (§2 rows 5, 8; §3 reasoning, `ESTIMATED` until §17's test runs).
- **Three things a judge will remember:** dragging the horizon and watching a covenant break 63 days out; a pasted sanction clause becoming a live, code-verified covenant; a why-panel that opens every stage of the decision, not just the model call (§5, §15, §17).
- **The few things it does:** digitise covenants, test them deterministically, score signals for persistence and materiality, forecast dated breach risk with drivers, rank the portfolio, draft grounded intervention memos, keep a full audit trail (§8 in-scope; §10).
- **What it will not do:** touch a live core-banking system, scrape real news, authenticate users, decide credit actions, retrain models, read PDFs, send alerts, or ship a second UI language (§8, NG-01…NG-09).
- **Built with (copied from §13):** Python + Flask + SQLite + pandas on one locked-down laptop; the TCS GenAI Lab gateway (`https://genailab.tcs.in`) running `azure_ai/genailab-maas-DeepSeek-V3-0324` for the two model stages.
- **How we tell it worked:** §17's 20-example set scored twice (product vs naive baseline) at T+20 on the judging machine; §6's five metrics, each with a failure reading that would mean the idea is wrong.

---

## §1 Purpose

### Scorecard 1 — business cases in `problem.md`

`problem.md` states exactly one business case, so this table has one row and nothing was struck; the real selection pressure is in Scorecard 2. Scores 1–5, from Step 2's research, not from the brief alone.

| Business case | Impact | Data | Needs AI | Originality room | Demo clarity | 24-h fit | Verdict |
|---|---|---|---|---|---|---|---|
| Covenant breach early warning, 30/60/90-day horizons, commercial credit | 5 — late detection costs are documented in ₹ lakh crore (§2 rows 2–4) | 5 — synthetic portfolio is mandated by `problem.md`, so data is fully reachable | 4 — extraction and narrative genuinely need a model; forecasting is best done by code (§3) | 3 — crowded field (§2 rows 6–7, §4), but the deterministic-registry + dated-horizon + audit combination is open | 5 — a countdown to breach is visibly dramatic on screen | 4 — fits only if narrowed hard (§8) | **Chosen** |

### Scorecard 2 — three product concepts for that case

Mechanisms as ordered stages, ≤8 words per stage; ✱ marks a stage decided by code rather than by a prompt.

| # | Concept | Mechanism stages | Judge watches | Prior-art gap (per §2/§4) | Undeliverable if | Impact | Data | AI-need | Orig. | Demo | 24-h | Verdict |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| C1 | **Covenant Radar** — registry-first monitoring pipeline | 1 model proposes covenants, code re-verifies✱, human confirms · 2 compute ratios, test covenants✱ · 3 score signals: sustained vs noise✱ · 4 project headroom to dated breach risk✱ · 5 rank portfolio by urgency✱ · 6 model drafts grounded memo | Queue reorders; horizon scrubbed to a dated breach; why-panel opens each stage | Incumbents track dates or alert on external data; none found combining deterministic recompute, dated horizon and per-stage audit (§4) | Synthetic generator eats the hours — mitigated by authored scenario templates (R-01) | 5 | 5 | 4 | 4 | 5 | 4 | **Chosen** |
| C2 | **Case-File Copilot** — ask-the-loan-file conversational analyst | 1 ingest and index borrower documents✱ · 2 retrieve evidence for a question✱ · 3 model answers with citations · 4 log Q&A to case file✱ | A chat answering credit questions with citations | Thin — 9fin ships exactly this shape for debt documents (§4); the pattern is the most common AI demo in any room | The 30/60/90 numbers would have to come from the model, which §17's grounding rule forbids | 3 | 4 | 3 | 1 | 2 | 4 | Loses on score; decisive defects: originality 1, and forecast-by-model conflicts with `problem.md`'s deterministic-calculation requirement |
| C3 | **Stress War-Room** — simulation-first breach war-gaming | 1 compute ratios, test covenants✱ · 2 model maps stress narrative to shocks, code applies✱ · 3 simulate paths to breach dates✱ · 4 re-simulate under candidate interventions✱ · 5 model drafts scenario memo | Typed stress story visibly bends the breach date | Stress-testing exists at portfolio level; borrower-level covenant war-gaming is rarer | Shock calibration is hand-waving inside 24 h; simulation UI is heavy | 4 | 4 | 3 | 4 | 4 | 2 | Loses on score; decisive defect: 24-h fit 2, and 30/60/90 monitoring becomes a side effect rather than the core |

**Struck out, and under which rule:** no business case was struck (only one exists). No concept was struck by the depth floor — all three resolve to ≥3 ordered stages with ≥1 decided by code; the floor's real work here was shaping C1, whose stages 2–5 are all code-decided. C2 and C3 lose on the scorecard, with their decisive defects named in the table. The naive fourth concept every room builds — one prompt behind a chat box — was not designed, because the depth floor eliminates it by definition.

**Did Step 2's research change the ranking?** Yes. At hour 0 the instinctive winner was C2, the chat-with-the-loan-file copilot; the research demoted it (9fin already is that product, and model-generated probabilities would break grounding) and promoted the evidence-ledger stage of C1, because the alert-fatigue literature (§2 row 8) showed noise control is the actual unsolved half of `problem.md`'s core challenge.

**What is being built and why, plainly:** Indian banks monitor loan covenants from borrower-submitted quarterly statements (QIS/stock statements) reviewed by hand; deterioration between reporting cycles is caught late, and the later it is caught the less is recovered — a 67% haircut on admitted claims once accounts reach insolvency (§2 rows 3, 5). The banks pay, then depositors and the exchequer via recapitalisation. Covenant Radar gives the credit team a daily, evidence-backed, dated warning — 30/60/90 days ahead — so intervention happens while a covenant conversation is still cheap. It serves the single business case `problem.md` states.

**What the problem is not** (nearby problems deliberately unsolved):

- Not retail or MSME collections and dunning.
- Not fraud adjudication — Radar surfaces signals a Red-Flagged-Account process could consume (§2 law block), but classifying fraud is a human, regulated act.
- Not loan origination, underwriting or credit scoring at sanction time.
- Not an external credit rating or a replacement for one.
- Not a document-management or covenant-certificate collection workflow.
- Not market, liquidity or operational risk monitoring.
- Not IFRS-9/ECL provisioning computation.
- Not a chat assistant over bank policy.

**(judged 1, 4)**

---

## §2 Background and context

Research run 2026-08-29 with live web access; 41 distinct searches and page-opens (log printed in chat with this spec). Findings that change this spec:

| # | What we found | The number | Source (link) | Year | What it changes here |
|---|---|---|---|---|---|
| 1 | Gross NPAs of scheduled commercial banks are at a multi-decadal low — the pitch is keeping them low, not fighting a blaze | GNPA 2.3% (Mar 2025); 2.15% domestic ops (Sep 2025) | RBI FSR via [Business Standard](https://www.business-standard.com/finance/news/indian-scheduled-commercial-banks-gross-npa-ratio-at-15-year-low-125070101394_1.html); [PIB](https://www.pib.gov.in/PressReleasePage.aspx?PRID=2225442&reg=3&lang=1) | 2025 | §1 framing; §6 metrics measure prevention, not cleanup |
| 2 | Loan-portfolio frauds surged even as case counts fell — late detection of borrower misbehaviour is the expensive kind | Loan-related frauds ₹33,148 crore in FY25 (of ₹36,014 crore total, up 194%) | RBI Annual Report via [Business Standard](https://www.business-standard.com/finance/news/bank-fraud-amount-triples-in-fy25-despite-drop-in-number-of-cases-rbi-125052900696_1.html) | 2025 | §6 value anchor; justifies treasury-flow and diversion signals in R-05 |
| 3 | Once stress reaches insolvency, recovery is poor — earliness has direct rupee value | 67% haircut on admitted claims; ₹3.99 lakh crore realised vs ₹12.31 lakh crore claimed (to Sep 2025) | IBBI data via [Business Standard](https://www.business-standard.com/industry/news/ibc-haircuts-2025-creditor-recovery-cirp-delays-ibbi-data-125112300278_1.html) | 2025 | §0/§6 headline value; G2's avoided-loss arithmetic |
| 4 | Wilful default has grown ~10× in a decade — borrower-behaviour signals, not just financials, matter | ₹3,83,264 crore outstanding, 18,318 wilful defaulters (Mar 2025) | Parliament/TransUnion CIBIL via [Moneylife](https://www.moneylife.in/article/from-39369-crore-to-383264-crore-in-10-years-indias-wilful-default-crisis-laid-bare-in-parliament-abg-shipyard-leads-at-6695-crore/79963.html) | 2025 | R-05 includes payment-behaviour and treasury-pattern evidence types |
| 5 | How banks cope today: borrower-submitted stock statements, QIS Forms I–III, MSOD returns, stock audits, and IBA-empanelled Agencies for Specialised Monitoring for ₹250-crore+ exposures; RBI's own supervision found post-sanction monitoring weak (end-use lapses, reliance on CA certificates); EASE reforms push PSBs toward IT-based EWS | QIS applies to fund-based working-capital limits ≥ ₹5 crore; ASM threshold ₹250 crore (IBA framework, Oct 2018) | [CAclubindia on QIS/QMS](https://www.caclubindia.com/articles/beyond-loan-sanction-understanding-qms-in-working-capital-finance-and-credit-monitoring-55833.asp); [RBSA on ASM](https://rbsa.in/risk-consulting/strategic-risk-advisory/agency-for-specialized-monitoring/); [Business Standard on RBI findings](https://www.business-standard.com/article/finance/rbi-finds-lapses-in-monitoring-of-fund-diversion-111011500043_1.html); [PIB on EASE 8.0](https://www.pib.gov.in/PressReleasePage.aspx?PRID=2237483&reg=3&lang=1) | 2018–2025 | Defines the "before" in §22's before/after; F-01's manual baseline; §6 hours-saved metric |
| 6 | An Indian incumbent exists and is credible — differentiation cannot be "we alert on borrower risk" | Crediwatch EWS: 200+ configurable alerts, 9–12-month lead claims, used by SBI, IndusInd, Central Bank of India | [Crediwatch EWS](https://about.crediwatch.com/about/product/early-warning-systems); [Azure Marketplace listing](https://marketplace.microsoft.com/en-us/product/saas/tchinformationanalyticsprivatelimited1654155358283.crediwatch_1?tab=overview) | 2025 | §4 differentiation; §22 judge question 6 |
| 7 | The global covenant-software field tracks dates or extracts documents; the sorting question in the trade is "does the tool test the covenant or only store the number a human entered" | Moody's, nCino (2,700+ FIs), Finastra LoanIQ + Eigen (100+ extracted fields), BankStride, Cync, Teslar | [Aloan category guide](https://aloan.ai/guides/best-covenant-monitoring-software); [Moody's loan monitoring](https://www.moodys.com/web/en/us/solutions/lending/loan-monitoring.html); [Finastra–Eigen](https://www.finastra.com/applications/eigen-technologies) | 2026 | §4's centre of gravity: Radar *tests* the covenant deterministically and dates the breach |
| 8 | Early-warning systems drown analysts in false positives; alert fatigue makes them ignore alerts — noise control is the unsolved half | ~95% false-positive rates reported in bank transaction monitoring; legacy EWS indicators described as producing overwhelming false alerts (worldwide figures — stand-ins; no India-specific rate found) | [Risk Hub](https://riskhub.org/blogs/early-warning-signals-of-credit-deterioration); [Flagright](https://www.flagright.com/post/understanding-false-positives-in-transaction-monitoring) | 2024–2025 | Promotes persistence/materiality scoring (stage 3, R-05) and §17 thresholds T3/T4 from nice-to-have to core |
| 9 | SMA-2 stress among large borrowers is a documented lead indicator — rising SMA-2 translates into NPA slippage about two quarters later; large borrowers carried 54.6% of loans and 83.4% of GNPA at end-Sep 2018 | SMA-2 +58.6% Q1→Q2 FY19 preceded slippage | RBI FSR Dec 2018 via [Shankar IAS summary](https://www.shankariasparliament.com/current-affairs/financial-stability-report-rbi-1) | 2018 (dated; the mechanism, not the level, is what we use) | §3's causal argument that behavioural stress precedes breach; horizon design and §17's threshold rationale |
| 10 | No granted or pending patent specifically claiming ML-based loan-covenant breach-horizon prediction surfaced in our search; adjacent art exists (transaction-risk prediction; income-risk scoring) | Closest: [US6119103](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/6119103), [US20110078073A1](https://patents.google.com/patent/US20110078073A1/en); class [705/38](https://patents.justia.com/patents-by-us-classification/705/38) | USPTO / Google Patents / Justia | 2000–2011 | §4's originality claim is combination-novelty, asserted against a shallow search — carried as **[OPEN-01]** (professional prior-art search, CPC G06Q 40/03) |
| 11 | Public Indian data exists to calibrate the synthetic portfolio's realism — aggregate corporate performance and company master data, not entity financial statements in bulk | RBI DBIE: listed non-govt non-financial company aggregates; data.gov.in company master CSV/API; MCA V3 filings at ₹100/company/year | [RBI DBIE](https://data.rbi.org.in/); [data.gov.in MCA catalog](https://www.data.gov.in/ministrydepartment/ministry-corporate-affairs); [MCA guide](https://knowledgebase.bison.co.in/view_article.php?id=352) | 2025–2026 | R-01's generator calibrates ratio ranges to DBIE aggregates; §14 lists these as build-time references only |

### Indian law and regulators that bind this product

- **RBI Master Directions on Fraud Risk Management (commercial banks & AIFIs), effective 15 Jul 2024** — mandate a board-approved Early Warning Signals and Red-Flagged Accounts framework, EWS integration with core banking, Data Analytics Units, RFA reporting on CRILC within 7 days, fraud-or-not decision within 180 days. *Makes us:* produce an auditable, reconstructable warning trail (R-11) whose outputs a bank's EWS/RFA process could consume; keep human decisioning outside the tool (advisory-only, §7/§16). Source: [SCC Online summary](https://www.scconline.com/blog/post/2024/07/17/rbi-revises-master-directions-on-fraud-risk-management-in-regulated-entities-legal-news/), [ELP full-text note](https://elplaw.in/wp-content/uploads/2024/07/RBI-Revised-Master-Directions-on-Fraud-Risk-Management-July-2024.pdf) (2024).
- **RBI Prudential Framework for Resolution of Stressed Assets, 7 Jun 2019** — SMA-0/1/2 classification by days overdue (1–30/31–60/61–90), CRILC monthly reporting plus weekly default reports for aggregate exposure ≥ ₹5 crore, 30-day review period on default. *Makes us:* speak this vocabulary natively (SMA bands on screen, ≥ ₹5 crore facilities in the synthetic book) and treat 30/60/90 not as arbitrary but as the supervisory cadence. Source: [RBI notification](https://www.rbi.org.in/Scripts/NotificationUser.aspx?Id=11580&Mode=0) (2019).
- **Digital Personal Data Protection Act, 2023** — a corporate borrower is not a data principal, but its guarantors, promoters, directors and signatories are; penalties to ₹250 crore per contravention. *Makes us:* class personal fields (§14), mask them before anything leaves the machine (N-04, §16), even though this build is synthetic-only. Source: [Cyril Amarchand FIG paper](https://corporate.cyrilamarchandblogs.com/2023/12/fig-paper-no-28-data-law-series-2-implications-of-digital-personal-data-protection-act-2023-on-indian-banks/) (2023).
- **RBI FREE-AI Committee report, 13 Aug 2025** — seven sutras for AI in the financial sector including Explainability, Accountability, Fairness; board-approved AI policy; proportionate oversight. Advisory, not yet binding rules. *Makes us:* per-stage explainability (§17 why-panel) and human-in-the-loop advisory outputs — designed in, not bolted on. Source: [Dvara Research summary](https://dvararesearch.com/summary-of-the-rbi-free-ai-committee-report/), [KPMG note](https://kpmg.com/in/en/insights/2025/08/rbi-free-ai-committee-report-on-framework-for-responsible-and-ethical-enablement-of-artificial-intelligence.html) (2025).
- **RBI Master Direction on Outsourcing of IT Services, 10 Apr 2023 (effective 1 Oct 2023)** — cloud/AI services used by a bank are material IT outsourcing: the bank keeps responsibility for data confidentiality, needs audit rights, incident reporting from provider so RBI is told within 6 hours, exit strategy. *Makes us:* treat the model gateway as an outsourced processor — minimum necessary data crosses to it (masking, N-04), every call logged (§20), a no-gateway fallback exists (§22). Source: [Lexology summary](https://www.lexology.com/library/detail.aspx?g=b38be29c-a67e-4eb1-b084-3b99f960313a), [RBI MD PDF](https://fidcindia.org.in/wp-content/uploads/2023/04/RBI-OUTSOURCING-OF-IT-SERVICES-10-04-23.pdf) (2023).
- **CERT-In Directions, 28 Apr 2022** — cyber incidents reported within 6 hours; ICT logs retained 180 days within India. Binds a production deployment's operator, not this synthetic local demo. `LATER:` production runbook includes CERT-In reporting and 180-day log retention in India. Source: [Trilegal note](https://trilegal.com/news-insights/how-to-comply-with-cert-ins-new-six-hour-time-frame-to-report-cyber-incidents/) (2022).
- **Checked and dropped as not applying:** Aadhaar/eKYC limits (no retail identity verification), NPCI/UPI rules (no payments rail), GST rules (no tax workflow; GST data as a future signal source is `LATER:`), TRAI (no telecom), SEBI (no securities issuance or advice — covenant data here is private credit), IRDAI (no insurance).

**Searches run: 41 distinct (25 field, 16 stack-and-look).** The three findings that changed our mind: (1) GNPA at a 15-year low flipped the framing from cleanup to prevention — the value anchor became the 67% haircut and ₹33,148 crore of loan frauds, not an NPA mountain; (2) Crediwatch plus the Moody's/nCino/Eigen field killed the generic alert-dashboard idea and located the open ground at deterministic covenant *testing* with a dated horizon and per-stage audit; (3) the alert-fatigue literature promoted persistence/materiality control from a feature to the core of the mechanism — `problem.md`'s "meaningful deterioration vs temporary noise" is the industry's known unsolved half. **(judged 2, 3)**

---

## §3 The idea, and why we think it works

**The solution in plain sentences.** Covenant Radar keeps a live, machine-readable register of every covenant a borrower signed, recomputes each one daily from the freshest data, and scores the everyday behavioural signals — payments, utilisation, cash flows — for whether they are sustained deterioration or passing noise. It projects each covenant's headroom forward and tells the credit team, with evidence, which borrower is likely to break which covenant in 30, 60 or 90 days, what is driving it, and what to do first. A person can open "why" on any number and see the whole chain that produced it.

**The mechanism (written once, here — the spine of this document).** Six ordered stages. Stages 2–5 are decided by code; stage 1 is model-proposed but code-verified and human-confirmed; stage 6 is the model writing prose over computed facts.

| Stage | In | Out | Decided by |
|---|---|---|---|
| 1. Covenant digitisation | Sanction-letter covenant text (pasted) | Structured covenant records: definition, threshold, direction, testing frequency, exceptions — proposed by the model, re-verified by code (schema check + independent recomputation of the definition against the ratio library + range sanity), confirmed by a person; invalid proposals are rejected, never silently accepted | Model proposes → **code verifies (fails closed)** → human confirms |
| 2. Covenant engine | Confirmed registry + borrower financial periods | Each covenant tested at its frequency: current value, threshold, headroom %, pass/fail, exceptions applied | **Code** |
| 3. Evidence ledger | Daily signal events (payments, utilisation, treasury flows, concentration, industry/news markers) | Typed evidence items with persistence score, materiality score and decay state; each classified sustained vs transient against §17's T3/T4 | **Code** |
| 4. Horizon forecast | Stage-2 headroom series + stage-3 sustained evidence | Per covenant per horizon (30/60/90): breach probability, confidence, projected breach date where trajectory crosses threshold, top drivers with contribution shares | **Code** (transparent trend projection + signal-pressure adjustment + calibrated mapping; formula inspectable in the why-panel) |
| 5. Portfolio triage | All stage-4 forecasts + exposure | Borrowers/facilities ranked by urgency = probability × exposure × confidence, banded against §17's T1/T2 | **Code** |
| 6. Intervention memo | Stages 2–5 outputs for one borrower + action catalogue | A warning memo: situation, drivers, evidence citations, recommended interventions for RM/credit/risk — every figure copied from records, never generated; shape-checked before display | **Model** (grounded, checked by code) |

**The reasoning, one step at a time.** Because RBI's own data says behavioural stress (SMA-2) precedes NPA slippage by about two quarters (§2 row 9), signals that exist *between* reporting cycles carry usable early information. Because today's monitoring is quarterly, borrower-submitted and manual (§2 row 5), that information is currently thrown away. Because covenant terms are contractual and computable, the breach condition itself needs no AI — computing it deterministically makes the warning auditable, which the 2024 Fraud Directions effectively require (§2 laws). Because analysts ignore noisy alerting systems (§2 row 8), a persistence-and-materiality filter must sit between signals and escalation, or the tool joins the ignored pile. Therefore: deterministic covenant testing + filtered behavioural evidence + horizon projection should surface true deterioration weeks before a scheduled test date, without drowning the desk. **What would prove us wrong:** if, on §17's example set, the full mechanism finds deteriorating borrowers no earlier or no more precisely than the naive distance-to-threshold rule (the baseline arm), the pipeline's extra stages are decoration and the idea fails. This is an argument, not a result — nothing has run at hour 3 — so the expectation is labelled `ESTIMATED`, and the head-to-head is funded and scheduled as §17's baseline arm (N-01), first scored at T+8, printed beside pass marks at T+20 (§23).

**Where AI earns its place, and where it does not.** AI earns stage 1 (covenant clauses are unbounded natural language; a rule parser handles only the formats it anticipated — §17's extraction examples test exactly this against a regex baseline) and stage 6 (turning twenty computed facts into a memo an RM acts on is language work). AI is deliberately *refused* stages 2–5: ratios, tests, persistence scoring, projection and ranking are arithmetic where a model would add error and subtract auditability. Judged on §17's examples, both ways.

**Coverage of the problem statement** (every part of `problem.md`; a part with no answer is named):

| `problem.md` part | What answers it | Carried by |
|---|---|---|
| Monitor covenant thresholds with financial/behavioural signals | Stages 2–3 | R-02, R-04, R-05 |
| Forecast breach risk at 30/60/90 days | Stage 4 | R-06 |
| Explain drivers of deterioration | Stage 4 outputs + why-panel | R-06, R-09 |
| Recommend prioritized interventions | Stages 5–6 | R-07, R-10 |
| Distinguish deterioration from noise (core challenge) | Stage 3 persistence/materiality + §17 T3/T4 | R-05, N-03 |
| Ingest financial statements, calculate ratios | Generator + engine | R-01, R-04 |
| Extract/represent covenant definitions, thresholds, frequency, exceptions | Stage 1 | R-03, R-02 |
| Monitor account activity, payments, utilization | Stage 3 | R-01, R-05 |
| Evaluate treasury flows and cash-pattern changes | Stage 3 treasury evidence type | R-05 |
| Concentration exposure + synthetic industry/news signals | Stage 3 evidence types (synthetic feed per `problem.md`) | R-01, R-05 |
| Predict breach probability at 30/60/90 | Stage 4 | R-06 |
| Identify primary drivers | Stage 4 attribution | R-06 |
| Rank borrowers/facilities by urgency and impact | Stage 5 | R-07 |
| Recommend RM/credit/risk interventions (advisory, human review) | Stage 6 + role-tagged catalogue | R-10 |
| Auditable warning trail (data, trends, calculations, reasoning) | Every stage writes audit events | R-11, R-09 |
| Covenant condition kept distinct from behavioural/external indicators | Registry (stage 2) and ledger (stage 3) are separate records and separate screens | R-02, R-05 |
| Risk view revisable when later evidence changes interpretation | Evidence supersession + override capture | R-11 |
| Confidence and materiality visible | Stage 4 confidence + stage 3 materiality on every warning | R-06, R-05 |
| Local, no live core-banking integration, synthetic data | Everything runs on one laptop from a seeded synthetic book | R-01, NG-01, NG-02 |

No part is unanswered; no part needed an `NG` to escape.

**Assumptions this reasoning stands on** (each also in §12's register): that behavioural signals lead covenant breaches in the synthetic world as they do in RBI's real one (§2 row 9 — if wrong, stage 3 adds nothing over stage 2); that covenant text in Indian sanction letters is regular enough for ≥90% field extraction after validation (§17 tests it; if wrong, stage 1 narrows to assisted manual entry); that a team of five with the skills in **[ASSUMED-01]** — all five write Python, two are comfortable building web UI, one knows credit basics — exists as assumed.

**Why this is buildable in §11's hours, honestly:** because it is narrow. One synthetic book of 12 borrowers, ~8 covenant types, 5 signal types, 3 authored storylines; one journey (§9 F-01) polished, three screens, one model, no logins, no PDFs, no integrations. Stages 2–5 are a few hundred lines of arithmetic over a small SQLite database — the kind of code five people write and test in a day. The two model stages are bounded: one extraction prompt, one memo prompt, both shape-checked. What makes it fit is everything §8 refuses. **(judged 5, 6, 7, 10, 11)**

---

## §4 What is new here

One line per thing Step 2 actually found, with how Radar differs. Links are the ones the searches returned.

**Products:**
- [Moody's Lending Suite / CreditLens](https://www.moodys.com/web/en/us/solutions/lending/loan-monitoring.html) — automates covenant *document* collection, validation and testing schedules; Radar differs by projecting a dated breach horizon forward and exposing per-stage reasoning, not compliance status alone.
- [nCino commercial lending](https://www.ncino.com/solutions/commercial-lending) — covenant due-date notifications and compliance records inside an origination platform; Radar tests covenants from data and forecasts, rather than reminding humans to.
- [Finastra LoanIQ + Eigen](https://www.finastra.com/applications/eigen-technologies) — extracts 100+ fields from compliance certificates; Radar's stage 1 differs by *re-verifying the extraction in code against a ratio library and failing closed* before anything is registered.
- [BankStride](https://www.bankstride.com/covenant-tracking-monitoring) — covenant tracking, document collection, exception workflow; date-and-exception management, no forecast.
- [Cync Software](https://aloan.ai/guides/best-covenant-monitoring-software) and [Teslar](https://aloan.ai/guides/best-covenant-monitoring-software) (per the category guide) — tracking-layer tools; same difference as BankStride.
- [Abrigo](https://aloan.ai/guides/best-commercial-lending-software) — covenant features inside a broader lending platform; not horizon-forecasting.
- [9fin](https://9fin.com/features/covenants) — AI covenant analysis with cited answers over debt documents for buy-side credit teams; a research tool over documents, not a monitoring pipeline over live borrower data; its existence is why concept C2 was demoted (§1).
- [Crediwatch EWS](https://about.crediwatch.com/about/product/early-warning-systems) — Indian incumbent, 200+ alerts on public/private data with 9–12-month-lead claims; alert-centric, not covenant-registry-centric: it flags entity risk, Radar tests the signed contract and dates the breach.
- [Lyzr covenant-monitoring agent blueprint](https://www.lyzr.ai/blueprints/banking/loan-covenant-monitoring-agent/), [metricsIQ](https://www.metricsiq.ai/industries/financial-services/covenant-monitoring/), [Anaptyss CovenAce](https://www.anaptyss.com/blog/ai-powered-covenant-monitoring-financial-institutions/), [Uptiq](https://www.uptiq.ai/glossary/what-is-covenant-tracking) — 2024–26 AI covenant products/marketing; none we found combines deterministic recompute, persistence-filtered evidence, dated horizon and per-stage explainability in one auditable trail.

**Schemes and regimes in use today:**
- RBI's [EWS/RFA framework under the 2024 Fraud Directions](https://www.scconline.com/blog/post/2024/07/17/rbi-revises-master-directions-on-fraud-risk-management-in-regulated-entities-legal-news/) — the compliance regime Radar's audit trail is shaped to feed; Radar is a tool under it, not a substitute for it.
- [QIS/stock-statement/MSOD monitoring](https://www.caclubindia.com/articles/beyond-loan-sanction-understanding-qms-in-working-capital-finance-and-credit-monitoring-55833.asp) — the borrower-reported quarterly regime; Radar's difference is daily computation between those reports.
- [Agencies for Specialised Monitoring](https://rbsa.in/risk-consulting/strategic-risk-advisory/agency-for-specialized-monitoring/) — human outsourced monitoring above ₹250 crore; Radar is the software layer below and beside that threshold.
- [EASE-programme IT EWS in PSBs](https://www.pib.gov.in/PressReleasePage.aspx?PRID=2237483&reg=3&lang=1) — mandate exists, adoption is uneven; Radar is a concrete shape for it.

**Papers:**
- [Altman 1968 Z-score](https://en.wikipedia.org/wiki/Altman_Z-score) — the interpretable distress baseline; Radar's baseline arm philosophy comes from here, and Radar differs by working at covenant level with contract-specific thresholds, not firm-level bankruptcy scores.
- [Barboza et al., ML vs Z-score](https://www.sciencedirect.com/science/article/abs/pii/S0957417417302415) — ML beats classic ratios on accuracy but loses traceability; Radar takes the opposite trade deliberately (code forecasts, model writes).
- [Demerjian–Owens, covenant-violation probability](https://www.sciencedirect.com/science/article/abs/pii/S0165410115000725) — measures violation probability from Dealscan-style data; Radar differs by holding the actual contract definition and recomputing it.
- [CovenantAI, Saunders et al. 2025](https://sbfc.sydney.edu.au/2025/papers/SBFC2025_6B2_P261.pdf) — ML detection of violations from SEC filings text; detection after the fact, not a forward horizon on live signals.
- [CUAD, Hendrycks et al. 2021](https://ar5iv.labs.arxiv.org/html/2103.06268) and [ContractEval 2025](https://arxiv.org/pdf/2508.03080) — clause extraction is well-studied with known error rates; exactly why stage 1's code re-verification exists, and why extraction alone is not our novelty claim.
- [Systematic review of EWS in finance](https://arxiv.org/pdf/2310.00490) — surveys the field; no covenant-registry-plus-dated-horizon system is described in what we read of it.
- [Networked-guarantee loan default prediction](https://arxiv.org/pdf/1702.04642) — graph risk propagation; adjacent, out of our scope (NG territory).
- [Predictive stress-testing model for covenant breach detection (IJSRCSEIT)](https://ijsrcseit.com/home/issue/view/article.php?id=CSEIT25113360) — 2025 conceptual model closest in intent; conceptual, and without the persistence-filter or per-stage audit design.

**Patents:** no granted or pending patent specifically claiming covenant-breach-horizon prediction surfaced in our search — said in those words, and carried as **[OPEN-01]** because our search was a web-scale one, not a professional freedom-to-operate search. Closest adjacent art found: [US6119103](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/6119103) (transaction-based financial risk prediction, 2000) and [US20110078073A1](https://patents.google.com/patent/US20110078073A1/en) (income-risk credit scoring, 2011); the relevant class is [US 705/38](https://patents.justia.com/patents-by-us-classification/705/38).

**The one mechanism we would call genuinely new, stated carefully:** the *dated-horizon evidence ledger* — fusing (a) a code-verified covenant registry that recomputes the contract itself, (b) a persistence-and-materiality-scored evidence ledger that decays noise, and (c) a projection that outputs a *date* a specific covenant crosses its specific threshold, every stage of which is openable in an audit panel. Each part is known art (Altman-style ratios; EWS alerting; CUAD-style extraction; trend projection). The combination — and the failing-closed extraction check, which inverts the usual "AI extracts, human hopefully reviews" into "AI proposes, code disproves" — is what we did not find anywhere above. It is not obviously the next step for someone skilled in the art, because the field's economics push vendors toward either document workflow (Moody's, Eigen) or external-data alerting (Crediwatch), not toward re-implementing the contract's arithmetic. Is it patentable? Possibly as a system claim on the verified-extraction-plus-dated-horizon pipeline, but we assert only combination novelty and Indian-regime fit: the parts are known; the combination, the fails-closed verification, and the SMA/CRILC-native framing are ours. A professional search (**[OPEN-01]**) decides the rest. **(judged 9)**

---

## §5 The three standout features

Funded first in §11, built first in §28, never on §8's cut list. Three different capabilities; working-by hours are distinct (T+13, T+15, T+17 — only one lands on T+17).

**S1 — Paste a sanction clause, get a live covenant** *(carried by stage 1 of §3's mechanism)*
- What the user can now do: turn the covenant paragraph of a sanction letter into a registered, tested, monitored covenant in under a minute — without trusting the AI, because code re-verifies the proposal and rejects what it cannot reproduce.
- Why it beats today: today covenant terms live in PDFs and Excel; Eigen-style extraction exists abroad but registers what the model says — Radar registers only what code can independently recompute (§2 row 7, §4).
- Built by: R-03 (+R-02 registry) — 4.5 hours.
- Working end to end by: **T+15**.
- Narrower version if hours run short: extraction runs on two pre-tested clause texts chosen from the seed set instead of arbitrary pasted input; registry entry stays manual for the rest.

**S2 — The 30/60/90 breach radar with a draggable horizon** *(carried by stages 2–5 of §3's mechanism — the mechanism itself on stage)*
- What the user can now do: see, for every borrower, which covenant breaks first, on what projected date, at what probability and confidence, driven by which signals — and drag time forward to watch headroom deplete instead of reading a static report.
- Why it beats today: quarterly QIS review discovers breaches after test dates (§2 row 5); incumbent tools track dates or raise undated alerts (§2 rows 6–7); nothing we found *dates* the breach and shows the trajectory.
- Built by: R-04, R-05, R-06 (+R-07 queue, R-08 screen) — 16 hours across them.
- Working end to end by: **T+13** (first standout working; the thinnest path through these stages already runs at T+8).
- Narrower version: scrubber becomes three fixed stops (30/60/90) instead of continuous drag; forecast table and dated breach remain.

**S3 — Committee-ready warning memo with a reconstructable audit trail**
- What the user can now do: generate an intervention memo whose every figure click-resolves to the record that produced it, log an override with reasons, and later reconstruct why any warning was raised, from source data to recommendation — the Fraud-Directions-shaped trail (§2 laws).
- Why it beats today: RBI's own inspections found post-sanction monitoring reliant on borrower certificates and weak documentation (§2 row 5); an auditable trail is the compliance artefact banks are told to have.
- Built by: R-10, R-11 (+R-09 why-panel) — 7 hours across them.
- Working end to end by: **T+17**.
- Narrower version: memo for the hero borrower only; audit reconstruction shown as the filtered event log plus why-panel rather than a dedicated screen.

Three different capabilities — digitise a contract, forecast its breach, defend the warning — not one capability shown three ways. None is a chart, tile, filter or view of data the user already had; each exists whether or not anyone is watching and would be in the product with no judges in the room. **(judged 12)**

---

## §6 Goals, business value and success metrics

| # | Goal | Who is better off, by how much | The number that says it happened | Today's figure (from §2) | First measured | Measurement source | Result that would mean the idea is wrong | Label |
|---|---|---|---|---|---|---|---|---|
| G1 | Warn before the breach, not after | Credit/risk team gains ≥30 days of intervention room per deteriorating account | ≥60% of deteriorating synthetic borrowers flagged ≥30 days before their covenant test date crosses | 0 — today's discovery is at/after the quarterly test (§2 row 5) | T+20, on the judging machine | §17 example-set backtest, printed in §23 | Radar flags no earlier than the naive headroom rule (baseline arm) | Synthetic run: `MEASURED` at T+20; real-bank outcome: `PROJECTED` |
| G2 | Convert earliness into recovery value | The bank; anchor arithmetic: at the 67% average haircut on claims (§2 row 3), even a 1-percentage-point recovery improvement on one ₹50-crore exposure (a mid-size CRILC-band facility we chose for the synthetic book) is ₹50 lakh | ₹ value = exposure × recovery-point gain; demo prints the arithmetic per scenario | ₹3.99 lakh crore realised vs ₹12.31 lakh crore claimed (§2 row 3) | Post-hackathon pilot (`LATER:` real measurement needs a real portfolio) | IBBI/bank recovery records in a pilot | Early warning produces no measurable recovery delta in pilot | `PROJECTED` |
| G3 | Do not drown the desk (noise control) | Risk officers; alert volume stays actionable | ≤1 of the 4 authored noisy-transient borrowers escalated; ≤30% of portfolio in amber-or-worse at any day of the backtest | Legacy EWS false-positive rates reported near 95% (worldwide stand-in, §2 row 8) | T+20 | §17 backtest, thresholds T1–T4 in force | Radar escalates the noise borrowers — it has rebuilt alert fatigue | `MEASURED` at T+20 (synthetic) |
| G4 | Usable by a stranger unaided | An RM/credit officer new to the tool completes §9 F-01 without help | ≤3 minutes, ≤2 stalls, zero explanations from us | No comparable figure exists for the manual QIS route; not claimed | T+13 (§15's timing run) | Observed unaided run by a person not on the team (§15 owns the open question of who) | Stranger cannot reach the memo without being rescued | `USER-TESTED` if §15's outside user materialises; else stays `ESTIMATED` and says so |
| G5 | Every shown figure defensible | Auditors/judges; trust in the trail | 20/20 sampled on-screen figures click-resolve to their source record via why-panel or audit trail | RBI inspections found documentation reliance on borrower/CA certificates (§2 row 5) | T+20 | §23 acceptance sweep, sampled by P5 | Any figure that resolves nowhere | `MEASURED` at T+20 |

**Why these metrics and not the easy ones:** alerts raised, borrowers scanned, dashboards viewed and model calls made would all look excellent while the product failed — a system that alerts on everything maximises them (§2 row 8). G1/G3 are a pair on purpose: earliness *and* quiet, because either alone is trivially gamed by tuning thresholds up or down. **(judged 14, 16)**

---

## §7 Users and stakeholders

In the 24-hour build, every human role below is played by one analyst on one laptop — that is a deliberate, stated decision (NG-03 covers the missing login; §16 gives the role-action table this build does and does not enforce). Corporate-credit documentation and workflow are in English (**[ASSUMED-02]**), so the UI is English with Indian number formats (₹, lakh/crore), IST dates and Indian FY quarters.

| Who | What they need | What they may do | What they must never be able to do | Signs off |
|---|---|---|---|---|
| Relationship Manager (RM) | An early, specific, evidence-backed heads-up on their accounts, days before the covenant test date | View portfolio and case files; read memos; record an action taken | Change covenant definitions, thresholds or forecasts; delete evidence | P5 (delivery lead) per §23 |
| Credit Officer | Covenant intake without re-keying, and proposals they can trust because code re-verified them | Paste clause text; review/correct/confirm stage-1 proposals; view everything RMs see | Confirm a covenant that failed code verification (the confirm control never appears for a failed proposal) | P5 |
| Risk Officer | A ranked, noise-controlled watchlist; the power to disagree | Everything above, plus: generate memos; override a risk view with reasons; mark evidence disputed | Silently override — every override is captured (R-11) | P5 |
| Auditor / reviewer | Reconstruction of any warning | Read-only everything, including audit trail, model-call log, thresholds and their history | Change anything | P5 |
| Operator (delivery lead in-build) | Working system, sane thresholds | Edit thresholds.json (logged); reseed; start/stop; run gates | Edit the audit log | P5 |
| Judges (audience, not users) | A legible 8-minute run — §22's demo is their flow | Watch; ask §22's questions | — | — |
| TCS GenAI Lab gateway (external system) | Well-formed requests within its limits | Receive masked prompts (N-04); return completions for stages 1 and 6 | Receive personal-class fields or raw identifiers; be trusted for figures (§17) | — |

---

## §8 Scope

**In (one line each):** one synthetic Indian commercial-lending book (12 borrowers, ≥ ₹5-crore facilities, 8 quarters financials, 180 days of daily signals, 3 authored storylines) · covenant registry with definition/threshold/frequency/exceptions · model-assisted, code-verified covenant intake from pasted clause text · deterministic covenant engine (~8 ratio types) · evidence ledger over 5 signal types (payments, utilisation, treasury flows, concentration, industry/news markers — synthetic feed) with persistence/materiality scoring · 30/60/90 forecast with probability, confidence, dated crossing, drivers · portfolio triage queue · grounded intervention memo with role-tagged advisory actions · override/feedback capture · append-only audit trail · why-panel over every stage · thresholds in an editable file · model-call observability log · §17 example set with dual-arm scoring harness.

**Out:**

- **NG-01 — No live core-banking/LOS/credit-system integration.** `problem.md` says none is required; 24 hours buys none. `LATER:` read-only CBS/LOS connectors (account conduct, utilisation) and CRILC-format export.
- **NG-02 — No real news/industry feeds.** `problem.md` permits synthetic external signals; scraping and entity-matching real news is a project of its own. `LATER:` licensed news API with entity resolution to borrower CINs.
- **NG-03 — No authentication or per-role authorization in-build.** One analyst, one laptop, server bound to 127.0.0.1 (§16). `LATER:` bank SSO (AD/SAML), §16's role-action matrix enforced server-side, maker-checker on intake and overrides.
- **NG-04 — No automated credit decisions, waivers or escalations.** All outputs advisory with human review, per `problem.md` and the Fraud Directions' human decisioning (§2). `LATER:` workflow hooks into the bank's committee/RFA process. This NG is permanent product posture, not a deferral of automation.
- **NG-05 — No learning/retraining inside the 24 hours.** Corrections are captured (R-11), not learned from. `LATER:` §17's loop-back paragraph (cadence, approval, rollback, drift watch).
- **NG-06 — No non-English UI.** Corporate credit runs in English (**[ASSUMED-02]**); hours go to the mechanism. `LATER:` Hindi and regional-language UI strings for wider desk use.
- **NG-07 — No alert delivery channels (email/WhatsApp/SMS).** The demo is pull-based on screen. `LATER:` digest emails and escalation pings per role.
- **NG-08 — No PDF/scan ingestion or OCR.** Stage 1 takes pasted text/.txt only. `LATER:` layout-aware PDF extraction for sanction letters and compliance certificates.
- **NG-09 — No dark theme.** §15 commits to one paper-light look, judged on a projector. `LATER:` a dark variant tokenised from the same palette.

**Cut list if behind** (decision hour against each; §5's three features and §15's protected screen are not on this list and never join it — when hours bind they *narrow* per §5/§11, in the order §11 states):

1. **T+13:** drop the day-advance animation — day picker becomes a plain select (saves polish hours in R-08's orbit, not the screen itself).
2. **T+15:** drop inline-edit on intake review — accept/reject/edit-in-form only (R-03 UX trim).
3. **T+17:** drop the dedicated audit screen — F-04 runs on the event-log table plus why-panel (R-11 view trim).
4. **T+19:** drop R-12 (what-if preview, a `should`) if it was started from reserve and is not finished.

**(judged 4)**

---

## §9 User flows and use cases

**F-01 — Morning triage (the main flow; §15's protected screen lives here).** Risk officer opens Radar → portfolio queue, ranked by urgency, SMA-style bands visible → opens the top borrower's case file → sees covenants with headroom, the 30/60/90 forecast, drivers, evidence ledger → drags the horizon scrubber and watches DSCR cross its threshold at a projected date → opens the why-panel on the forecast stage → clicks *Generate memo* → reviews the grounded memo, logs "RM call scheduled" as the action taken. **Bad ends:** gateway down at memo step (told plainly; §22 fallback path; everything before the memo still works) · forecast confidence below T2 (screen shows "insufficient evidence — watching", never a bare probability) · officer disagrees (override captured with reason, view revised, both states kept — R-11).

**F-02 — Covenant intake.** Credit officer opens a borrower → *Add covenant* → pastes the sanction-letter covenant paragraph → stage 1 proposes structured fields with the model, code re-verifies (schema, recomputation against the ratio library, range sanity) → officer corrects the testing frequency, confirms → covenant is live and immediately tested. **Bad ends:** verification fails (proposal shown struck-through with the failing check named; confirm control absent; officer may hand-enter) · pasted text is empty/enormous/instruction-like (§19: refused with a plain message) · model unavailable (hand-entry form is the same screen, minus the proposal column).

**F-03 — A signal day (noise vs deterioration, revision).** The RM (played in-build by the analyst, like every role — §7) advances the demo day → new signal events land (a delayed payment; a utilisation tick-up) → evidence ledger scores them; a transient blip decays visibly without escalating; a sustained pattern crosses T3 and lifts the forecast → the borrower moves in the queue with a "what changed" note → later evidence contradicts (payment made): risk view revises down, prior state preserved in the trail.
 **Bad ends:** a signal contradicts financials (both shown, flagged "unreconciled", confidence reduced — never silently merged) · duplicate signal file re-seeded (idempotent ingestion, §19).

**F-04 — Audit and administration.** Auditor picks any past warning → reconstruction view: source data, covenant calculation, trend, evidence in force, thresholds in force, forecast, memo, overrides — the §2 Fraud-Directions trail → operator (separately) edits T1 in the thresholds file and reloads; the change appears in the audit trail with before/after. **Bad ends:** thresholds file malformed (refused at load with line number; last good values stay in force) · reconstruction of a warning from before a reseed (told plainly: demo retention is build-lifetime, §14).

---
## §10 Requirements

Every stage of §3's mechanism is carried by at least one requirement: stage 1 → R-02/R-03; stage 2 → R-04; stage 3 → R-05; stage 4 → R-06; stage 5 → R-07; stage 6 → R-10; cross-stage audit → R-09/R-11. Hours here are the estimates §11 adds up.

### R-01 · Synthetic Indian portfolio generator
Flow: F-01, F-03 · Priority: must · Hours: 3
Why: without a believable book there is nothing to monitor, forecast or demo.
Does: a deterministic, seeded generator produces 12 corporate borrowers with Indian names/CIN-format IDs (clearly fictional), facilities ≥ ₹5 crore, 8 quarters of financial statements calibrated to public aggregate ranges (§2 row 11), and 180 days of daily signal events across the 5 signal types. Three authored storylines: sustained deteriorator (hero), noisy-transient, stable; the rest templated. Regenerating from the same seed gives identical data.
Checks:
- R-01.a Given the seed command, when run twice from scratch, then both runs produce byte-identical databases.
- R-01.b Given the generated book, when each borrower's ratios are recomputed by hand for one quarter (sampled), then values match the stored statements.
- R-01.c (bad path) Given a truncated/malformed seed file, when seeding runs, then it stops with a named error and writes no partial book.

### R-02 · Covenant registry
Flow: F-02, F-01 · Priority: must · Hours: 1.5
Why: the contract's terms are the product's ground truth; unversioned terms make every later number indefensible.
Does: stores covenants per facility: definition (from a closed library of ~8 ratio/condition types), threshold, direction, testing frequency, applicable exceptions, source text, version, status. Versions are immutable once a test has run against them; edits create a new version. Registry is populated by seed and by F-02 intake.
Checks:
- R-02.a Given a registered covenant, when its terms are edited, then a new version exists, the old version remains, and past test results still reference the old version.
- R-02.b (bad path) Given a covenant whose definition names a ratio outside the library, when registration is attempted, then it is refused with the unknown-definition error.

### R-03 · Model-assisted, code-verified covenant intake
Flow: F-02 · Priority: must · Hours: 3
Why: covenant terms live in prose; manual re-keying is today's error-prone path (§2 row 7).
Does: pasted clause text goes to the model (stage 1), which proposes structured fields; code independently re-verifies — schema validity, definition recomputable against the ratio library, threshold within sane range for that ratio type, frequency from the allowed set; failures strike the proposal and name the failing check. A person confirms before anything registers. The confirm control does not exist for a failed proposal.
Checks:
- R-03.a Given a well-formed clause from the example set, when proposed and verified, then all fields match the hand-labelled answer and confirmation registers a live covenant.
- R-03.b (bad path) Given a clause with a deliberately impossible threshold (e.g., DSCR ≥ 40), when verified, then the proposal is struck with the range check named, and no confirm control renders.
- R-03.c (bad path) Given pasted text containing instructions to the model ("ignore prior instructions…"), when processed, then output is either a valid proposal about covenant fields or a refusal — never any other behaviour; the attempt is logged (§17 guardrails).

### R-04 · Deterministic covenant engine
Flow: F-01, F-03 · Priority: must · Hours: 3
Why: the contractual condition must be computed exactly, kept distinct from behavioural indicators (`problem.md` control 1).
Does: computes the ratio library (leverage, DSCR, interest cover, current ratio, TOL/TNW, utilisation %, receivable days, minimum-net-worth) from stored financials; tests each covenant at its frequency; applies registered exceptions; outputs value, threshold, headroom %, pass/fail, computed-at. All arithmetic in code; no model involvement.
Checks:
- R-04.a Given the 12 hand-worked engine cases in §17's set, when the engine runs, then 12/12 exact matches.
- R-04.b Given a covenant with an exception in force (e.g., threshold relaxed for two quarters), when tested inside and outside the window, then the exception applies only inside it, and the test record names the exception used.
- R-04.c (bad path) Given a borrower with a missing quarter, when tested, then the covenant is marked "stale — data to Q(n)", no value is invented, and forecast confidence for it drops (feeds R-06).
- R-04.d (bad path) Given negative EBITDA making a ratio undefined, when tested, then the defined behaviour is shown ("not computable — treated as breach-direction worst case, flagged for review"), never a division error or a silent skip.

### R-05 · Evidence ledger with persistence and materiality scoring
Flow: F-03, F-01 · Priority: must · Hours: 3.5
Why: `problem.md`'s core challenge — deterioration vs noise — lives here; without it Radar rebuilds alert fatigue (§2 row 8).
Does: ingests daily signal events (payments, utilisation, treasury flows, concentration, industry/news markers); each becomes a typed evidence item scored for persistence (T3) and materiality (T4) with decay; classified sustained vs transient; transient items remain visible but do not escalate; superseding evidence revises, never deletes. Concentration and industry markers enter as slower-moving typed items from the seed.
Checks:
- R-05.a Given the noisy-transient storyline, when 180 days replay, then its blip evidence decays without the borrower ever crossing T1-amber.
- R-05.b Given the deteriorator storyline, when its payment delays persist past T3, then the evidence flips to sustained on the exact day the rule says, and the flip is in the audit trail.
- R-05.c Given contradicting later evidence (payment made), when ingested, then the risk view revises, and both the prior and revised states are reconstructable.
- R-05.d (bad path) Given the same signal file seeded twice, when ingested, then no duplicate evidence items exist.

### R-06 · 30/60/90 horizon forecast with drivers
Flow: F-01, F-03 · Priority: must · Hours: 4
Why: this is the product's promised answer — probability, confidence and a *date*, per covenant, per horizon.
Does: projects each covenant's metric trajectory (trend over recent periods, adjusted by sustained-evidence pressure), finds where it crosses the threshold, maps distance-velocity-pressure to a breach probability per horizon with a confidence score; attributes top drivers with contribution shares (T5). Pure code; formula parameters live with §17's thresholds; every intermediate is stored for the why-panel.
Checks:
- R-06.a Given the deteriorator, when forecast at the demo day, then direction is deteriorating, the 60-day probability exceeds T1, and the projected crossing date is within ±10 days of the storyline's authored breach.
- R-06.b Given the stable borrower, when forecast, then all horizons sit below T1-amber.
- R-06.c Given any forecast, when its why-panel opens, then inputs, formula terms, thresholds compared and driver shares are all displayed and sum consistently.
- R-06.d (bad path) Given confidence below T2, when the case file renders, then "insufficient evidence — watching" appears in place of a probability.

### R-07 · Portfolio triage queue
Flow: F-01 · Priority: must · Hours: 1.5
Why: `problem.md` requires ranking by urgency, confidence and exposure; the desk needs an ordered morning.
Does: ranks borrowers/facilities by urgency = probability × exposure × confidence at the worst covenant-horizon; bands by T1 (act/amber/watch); shows what changed since the prior day. Pure code sort.
Checks:
- R-07.a Given the seeded book at demo day, when the queue renders, then the deteriorator ranks first and ordering matches the recomputed urgency values.
- R-07.b (bad path) Given two borrowers with equal urgency, when ranked, then the tie-break (larger exposure first) applies and is stated in the why-panel.

### R-08 · Borrower case file with horizon scrubber (the protected screen)
Flow: F-01, F-03 · Priority: must · Hours: 4
Why: this is §15's protected screen and §22's opening hook; the mechanism becomes visible here.
Does: one screen per borrower: covenant strip with headroom, forecast panel with the draggable 30/60/90 horizon (dragging recomputes displayed headroom/probability from stored daily paths — display reads stage-4 outputs, it does not re-model), evidence ledger margin, memo and action area. Empty/loading/error/offline states designed (§15).
Checks:
- R-08.a Given the hero borrower, when the horizon is dragged from today to 90, then covenant lines, probability, dated crossing and lit drivers update coherently at every stop, within §18's interaction budget.
- R-08.b (bad path) Given a borrower with no evidence and one covenant, when opened, then the designed empty states render (no blank panels, no console errors).

### R-09 · Why-panel over every stage
Flow: F-01, F-04 · Priority: must · Hours: 3
Why: explainability is judged, regulated (§2 FREE-AI), and the depth claim's proof — a why-panel that only explains the model call cannot explain stages 2–5.
Does: from any shown decision, opens the run behind it: each §3 stage in order, what it received, what it produced, whether code or the model decided it, which thresholds were compared and which side the value fell; for model stages, the prompt version and returned text; for code stages, the rule and the numbers. (Full contract in §17.)
Checks:
- R-09.a Given the hero's 60-day warning, when why is opened, then all six stages are present, each opening to its inputs/outputs/decider, and every threshold named shows its value and the comparison result.
- R-09.b (bad path) Given a stage that did not run (memo not yet generated), when why is opened, then that stage shows "not run" rather than fabricated content.

### R-10 · Grounded intervention memo
Flow: F-01 · Priority: must · Hours: 2.5
Why: the actionable artefact — what the RM, credit and risk teams actually take away (`problem.md` capability 9).
Does: on demand per borrower, the model (stage 6) drafts a memo over a fixed template: situation, covenant position, drivers, evidence citations, recommended interventions drawn only from a role-tagged action catalogue (RM/credit/risk), advisory wording. Every figure is injected from records; the model's own text is labelled as drafted; output is shape-checked (all figure slots resolve, no unlisted actions, length ≤ T6) — fail → one retry → refuse (§17).
Checks:
- R-10.a Given the hero borrower, when a memo generates, then every number in it click-resolves to a record and every recommended action exists in the catalogue with the right role tag.
- R-10.b (bad path) Given a forced malformed model response (test hook), when shape-check fails twice, then the user sees the refusal message and no memo — never a partial memo.
- R-10.c (bad path) Given the gateway down, when memo is requested, then the plain unavailable message appears and everything else on the screen still works.

### R-11 · Audit trail, override and feedback capture
Flow: F-01, F-03, F-04 · Priority: must · Hours: 1.5
Why: the Fraud-Directions-shaped trail (§2) and the product's only learning signal (§17 loop).
Does: append-only events for every stage run, threshold change, intake confirmation, memo, override, action logged — each carrying actor label, timestamp (IST), prompt/model versions where relevant, thresholds in force. Reconstruction view per warning (F-04). Overrides capture what was shown, what the user did instead, which stage produced it, and why.
Checks:
- R-11.a Given any warning in the demo book, when reconstructed, then source data, calculation, trend, evidence, thresholds, forecast, memo and any overrides are all reachable from one view.
- R-11.b (bad path) Given an attempt to edit or delete an audit event via any exposed route, then no route exists that can (the store is append-only by construction) and the attempt (if via file tampering) is out of threat model, stated in §16.

### R-12 · What-if intervention preview
Flow: F-01 · Priority: should (reserve-funded only) · Hours: 2
Why: lets a risk officer see which intervention buys the most headroom — the C3 idea's cheapest slice.
Does: applies a catalogue intervention's stated effect (e.g., ₹5 crore limit reduction) to the stored trajectory and re-renders the forecast panel side-by-side.
Checks: R-12.a Given the hero and one intervention, when previewed, then the delta shown equals the recomputed forecast difference.

### R-13 · Nightly batch scoring and alert digest
Flow: F-01 (it pre-computes the queue that flow opens on) · Priority: later (specced, not built) · Hours: —
Why: a real desk gets its queue at 9 a.m. without opening anything. `LATER:` scheduled nightly stage-2–5 run over the full book with a digest per role (NG-07's channels).
Checks: R-13.a Given the scheduler runs, when morning comes, then every borrower's forecast is fresher than 24 hours.

### Non-functional requirements

### N-01 · The example set and dual-arm harness
Flow: F-01, F-02 · Priority: must · Hours: 3
Why: the one number separating "it works well" from every other team claiming the same (§17 owns the content).
Does: 20 examples (§17) live as files; a harness runs the product arm and the baseline arm on all of them, prints both scores beside pass marks. Run at T+8 (first score) and T+20 (§23's `MEASURED` run).
Checks:
- N-01.a Given the harness, when run on the judging machine, then both arms' scores print for all 20 examples with pass/fail per mark.
- N-01.b (bad path) Given one malformed example file, when the harness runs, then it names the file and continues with the rest.

### N-02 · Observability — app log and AI log, separate
Flow: F-04 · Priority: must · Hours: 1
Why: "what did that call cost and why did it say that" must be answerable an hour later (§20 owns the content).
Checks:
- N-02.a Given any demo run, when the AI log is read, then every model call shows prompt version, model, tokens in/out, latency, computed cost field, check verdict, retries.
- N-02.b (bad path) Given the log file unwritable, when a request runs, then the user-facing flow still completes and the failure is surfaced on the ops panel, not swallowed.

### N-03 · Thresholds externalized
Flow: F-04 · Priority: must · Hours: 0.5
Why: every deciding number must be changeable without a code change and auditable when changed (§17 owns the table).
Checks:
- N-03.a Given a T1 change in the thresholds file, when reloaded, then the queue re-bands accordingly and an audit event holds before/after.
- N-03.b (bad path) Given a malformed thresholds file, when reload is attempted, then the last good values remain in force and the error names the line.

### N-04 · Privacy masking and key hygiene
Flow: F-01, F-02 · Priority: must · Hours: 1
Why: the gateway is another company's computer (§2 IT-outsourcing MD; DPDP classes in §16).
Does: prompts to the gateway are built from templates that admit only whitelisted, non-personal fields (ratios, headrooms, evidence types, clause text scrubbed of names/ID-patterns by rule); the API key comes only from the environment; neither key nor raw personal fields ever appear in code, logs or prompts.
Checks:
- N-04.a Given the full demo run, when both logs and all outbound request bodies (captured by the test hook) are scanned, then zero personal-class fields (§14) and zero key material appear.
- N-04.b (bad path) Given a clause text containing a guarantor name and PAN-format string, when sent to stage 1, then the outbound prompt shows them masked, and the stored registry keeps the original locally.

### N-05 · Performance to §18's targets
Flow: F-01 · Priority: must · Hours: 0.5
Checks:
- N-05.a Given the seeded book on the judging machine, when §18's measured runs execute at T+20, then p95 screen, scrubber and memo targets are met, and the numbers print in §23.
- N-05.b (bad path) Given 10× the book (120 borrowers, script-generated), when the daily scoring runs, then it completes within §18's 10× budget — or the miss is printed honestly in §23.

Every solution requirement in `problem.md` maps to at least one requirement above (§3's table gives the row-by-row mapping).

---

## §11 Constraints

**The clock.** Five people (**[ASSUMED-01]**) × (24 − 4 spec/setup − **5 sleep** − 3 rehearsal/demo) = 5 × 12 = **60 working hours**. The sleep figure is the single number 5 (between 4 and 6), and §28 schedules exactly 5 hours per person.

- **working hours: 60**
- **`must` hours: 36.5** (R-01 3 + R-02 1.5 + R-03 3 + R-04 3 + R-05 3.5 + R-06 4 + R-07 1.5 + R-08 4 + R-09 3 + R-10 2.5 + R-11 1.5 = 30.5 functional; N-01 3 + N-02 1 + N-03 0.5 + N-04 1 + N-05 0.5 = 6.0 non-functional)
- **support hours: 12** (20% of working hours): the two gates 2 · merges/integration 3 · T+20 acceptance sweep and the fixes it finds 3.5 · demo data freeze, reset scripting, preflight 1 · design review at T+17 1 · screenshot capture incl. the human fallback 1 · §15's unaided-user timing 0.5
- **reserve: 11.5** = 60 − 36.5 − 12; floor is max(2, 15% × 60 = 9) = 9 ✓. The reserve is not slack: it absorbs what nobody thought of at hour 3; R-12 may be started from it only after T+13 is green.

**Funding order, said plainly because at some hour it will bind:** §5's three standout features are priced first — S2 (16 h across R-04/05/06/07/08), then S1 (4.5 h across R-02/R-03), then S3 (7 h across R-09/R-10/R-11) — and §15's protected screen (R-08's 4 h, inside S2's total) immediately after them; everything else is funded from what remains. When hours bind, the order of giving way is: first the narrower versions §5 already wrote, in the order S1 → S3 → S2 (extraction narrows first because its fallback loses least; the mechanism narrows last); the protected screen holds its fidelity while the *other* screens compress (§15); none of the four is ever dropped.

**The rest of what is fixed, whose rule, and what it forbids:**

- **Laws (§2's block; the regulators' rules):** advisory-only outputs with human review; auditable trail; personal-data masking to the gateway; no fraud classification by the tool.
- **The laptop (TCS IT policy):** no admin rights, no Docker/VMs/services; project folder only; packages only from the approved registry (**[ASSUMED-03]**: it is a standard TCS PyPI mirror carrying §13's packages — verified at the T+4 gate); no CDN fonts/scripts; no model weights or GitHub downloads; local web server on a high port; assume the proxy blocks everything else.
- **Money:** zero budget beyond the provided gateway key; every paid alternative is out.
- **Team:** five people, skills per **[ASSUMED-01]**; nobody joins mid-run.
- **The demo slot:** **[ASSUMED-04]** 8 minutes total — 5 minutes of demo, up to 3 of questions inside the slot (`problem.md` gives no slot; §22's beats are timed against this and fit with spare).
- **Submission:** **[ASSUMED-05]** repo link/zip, this spec, a ≤10-slide deck and a backup screen recording, all due at T+24; `problem.md` names no earlier deadline, so no marker moves (§28 carries this).
- **Judging machine:** **[ASSUMED-06]** one of our own build laptops drives the projector (§28's T+4 gate note).
- **Hour markers:** **[ASSUMED-07]** `problem.md` states none, so the brief's markers stand: T+0 start · T+3 spec · T+4 environment gate · T+8 thinnest path · T+13 first standout · T+17 freeze · T+20 demo on judging machine · T+22 code freeze · T+24 judged.
- **Systems it must fit inside:** the GenAI Lab gateway (§12/§13/§14) is the only external system; its limits are [OPEN-02]/[OPEN-03] and its outage behaviour is §22's.

**(judged 6, 11)**

---

## §12 Dependencies, assumptions and critical path

**Outside things this needs; owner; if late or missing:**

| Dependency | Owner | If late/missing | Critical path? |
|---|---|---|---|
| GenAI Lab gateway reachable with our key; envelope numbers — rate limit, quota, context window, latency, token accounting, price per call and per month — all unknown at hour 3: **[OPEN-02]**, answered at the T+4 gate | Organizers / P4 tests | Stages 1 and 6 fall back to §22's replay cache; stages 2–5 (the mechanism's core) run regardless — demo survives, S1 narrows | **Yes** — for S1/S3's live form; the pipeline itself is not hostage to it |
| Structured-output/JSON mode and streaming support on the gateway: **[OPEN-03]**, T+4 | P4 | Own-code shape check is the plan of record anyway (§17); streaming is cosmetic | No |
| Fallback chat alias `azure/genailab-maas-gpt-4o` answering: **[OPEN-04]**, T+4 (the only alias beyond the evidenced `azure_ai/genailab-maas-DeepSeek-V3-0324` this spec commits to; both copied character-for-character from the organizer list) | P4 | Single-model build; §22's endpoint-dies fallback loses one rung | No |
| TLS verification: the handed sample sets `verify=False`; whether verification-on works on this network is **[OPEN-05]**, tested first at the T+4 gate with verification left on — if it succeeds we ship it that way (§16 states the posture) | P4 | Carry the sample's setting, stated plainly in §16 | No |
| Python ≥3.10 present per-user on the laptop (python.org is outside the approved registry, so presence is assumed-not-installable): **[OPEN-06]**, T+4; default if absent: the IT self-service portal's approved Python, requested at T+4 sharp | P1 | Everything is stuck until it exists — the one true single point of failure | **Yes** |
| Approved registry carries every package §13 pins: **[OPEN-07]**, T+4 | P1 | §13's standard-library fallback path — ugly but demonstrable | **Yes** until the gate; No after |
| Judging-room network lets the demo laptop reach the gateway: **[OPEN-08]**, checked at T+20's first run in the room | P5 | Replay mode for stages 1/6, labelled as replay on screen (§22) | No |
| An outside-team person for §15's unaided timing at T+13 (owner P5 asks the organizers by T+6; §15 carries the open question) | P5 | G4 stays `ESTIMATED` and says so | No |

Nothing else is on the critical path: data is self-generated (R-01), the stack is local, and no approval, account or human outside the room is needed for the mechanism to run. The two critical-path items ([OPEN-06], [OPEN-07]) are both resolved inside the first hour of build time at the T+4 gate, which is why that gate exists.

**Assumptions register** (each labelled where it arose; consequences if wrong):

- **[ASSUMED-01]** (§3) Team of five; all write Python, two build web UI, one knows credit basics. Wrong → §11's arithmetic reruns with the true count; below four people, S1 ships in its narrow form from the start.
- **[ASSUMED-02]** (§7) Corporate-credit users work in English; sanction letters are English. Wrong → NG-06 flips into scope and costs reserve.
- **[ASSUMED-03]** (§11) The approved registry is a standard TCS PyPI mirror. Wrong → [OPEN-07]'s fallback path.
- **[ASSUMED-04]** (§11) 8-minute slot, questions inside. Wrong → §22's beat table re-times; beats are modular and cut from the middle.
- **[ASSUMED-05]** (§11) Submission bundle due T+24, nothing earlier. Wrong (earlier deadline) → §28 moves freeze markers ahead of it and says which.
- **[ASSUMED-06]** (§11) Judging machine is our build laptop. Wrong → T+20 becomes install-and-verify on the real machine; §25's from-nothing path is the plan and its ≤10-minute budget is why it is rehearsed.
- **[ASSUMED-07]** (§11) Hour markers as the brief's nine. Wrong → §28 re-anchors; the dependency order of markers is unchanged.
- **[ASSUMED-08]** (here; used by §30) `problem.md` gives no criteria weighting, so we ranked them ourselves: 9, 12, 13, 14 decide; 1, 2, 4 are table stakes; the rest mid-weight. Wrong → §30's opening line re-weights; section content stands.

**(judged 8, 19)**

---

## §13 Architecture overview

Everything below is chosen here, not before (rule 2). Versions and evidence are from Step 4's searches on 2026-08-29, with links; "installs without admin" means per-user `pip install --user` from the approved registry into the project's venv, or already present on Windows 11.

| Layer | Technology (version) | Why production-grade (evidence, link) | Installs without admin | Fallback if the proxy/registry blocks it |
|---|---|---|---|---|
| What the user touches | Server-rendered HTML via Jinja2 3.1.6 templates + hand-written CSS + vanilla JS; charts as inline SVG rendered by our own templates (no chart library — §15's look is hand-drawn and the proxy kills CDNs) | Jinja2: Pallets-maintained, security release Mar 2025-era line, current 3.1.6 on [PyPI](https://pypi.org/project/Jinja2/) (2026); vanilla JS/CSS/SVG: platform standards, nothing to maintain | Yes — Jinja2 arrives as Flask's dependency | Python stdlib `string.Template` + static HTML (ships inside CPython, maintained under the PSF release policy — [devguide](https://devguide.python.org/versions/)) |
| Runs the logic | Python 3.12+ (whatever ≥3.10 the laptop has — [OPEN-06]; CPython maintained under the PSF release policy, [devguide](https://devguide.python.org/versions/)) · Flask 3.1.3 · served by waitress 3.0.2 | Flask 3.1.3 released 19 Feb 2026, Pallets, the most widely deployed Python microframework ([PyPI](https://pypi.org/project/Flask/)); waitress 3.0.2, Pylons Project, pure-Python WSGI server that "supports Windows directly", recommended by [Flask's own deployment docs](https://flask.palletsprojects.com/en/stable/deploying/waitress/) | Yes — both pure-Python from the registry | FastAPI 0.141.1 + uvicorn 0.52.4 (both current Aug 2026, massive production adoption — [FastAPI release notes](https://fastapi.tiangolo.com/release-notes/), [uvicorn PyPI](https://pypi.org/project/uvicorn/)) if Flask/waitress specifically are missing or broken on the mirror; stdlib `http.server` is deliberately *not* the fallback — its own docs say not for production ([docs](https://docs.python.org/3/library/http.server.html)) — and a registry empty of every web framework is [OPEN-07]'s escalation case in §12, not a component choice |
| Numeric work | pandas 3.0.5 (+ its numpy) | pandas 3.0.5 released 22 Jul 2026, 15-billion-download library run by the PyData core team ([PyPI](https://pypi.org/project/pandas/), [release notes](https://pandas.pydata.org/docs/whatsnew/index.html)) | Yes — registry wheels for Windows | Python stdlib `statistics` + `csv` + hand math (stdlib, same PSF policy as above; fine at 12 borrowers) |
| Where data is kept | SQLite via Python stdlib `sqlite3` (SQLite bundled with CPython; exact bundled version read at the T+4 gate) | SQLite: the most widely deployed database engine, public-domain, maintained ([sqlite.org](https://www.sqlite.org/mostdeployed.html)); the `sqlite3` module is stdlib ([docs](https://docs.python.org/3/library/sqlite3.html)) — nothing to install | Yes — nothing to install | JSON files via stdlib `json` (same no-install property) |
| Background work | None in-build (day-advance is synchronous; R-13 is `later`) | — | — | — |
| Outside services: model gateway | Endpoint `https://genailab.tcs.in` (fact, handed over); chat model **we choose**: `azure_ai/genailab-maas-DeepSeek-V3-0324` (the one evidenced alias); fallback alias `azure/genailab-maas-gpt-4o` [OPEN-04]. Client **we choose**: langchain-openai 1.5.1 (primary — matches the handed sample, so it is the evidenced working path) | langchain-openai 1.5.1 current on PyPI with ~69 M monthly downloads ([pypistats](https://pypistats.org/packages/langchain-openai)); openai SDK 3.5.0 current, official ([PyPI](https://pypi.org/project/openai/)); httpx 0.28.1, ~871 M monthly downloads ([PyPI](https://pypi.org/project/httpx/)) | Yes — pip from registry | The gateway speaks the OpenAI protocol, so three doors exist and one is needed: fallback 1 the raw `openai` SDK 3.5.0; fallback 2 a plain HTTPS POST to the chat-completions path with httpx 0.28.1 (already a dependency of the handed sample). If the SDK majors mis-handshake with the gateway, the pin drops to the last 2.x at the gate — noted under [OPEN-02]'s gate test |
| Build-time: test runner | pytest 9.1.1 | Released 19 Jun 2026, the standard Python test runner ([PyPI](https://pypi.org/project/pytest/)) | Yes — pure-Python install | Python stdlib `unittest` (PSF-maintained) |
| Build-time: screenshot capture | Microsoft Edge headless CLI — `msedge.exe --headless --disable-gpu --window-size=W,H --screenshot=out.png URL`; Edge ships with Windows 11, so nothing downloads; whether policy allows headless on this laptop is **[OPEN-09]**, tested at T+4 | Edge is Chromium, Microsoft-shipped and serviced; the headless screenshot flags are documented Chromium behaviour ([swimburger guide](https://swimburger.net/blog/web/hidden-gem-take-screenshots-using-built-in-commands-in-chrome-edge), [Edge CLI notes](https://textslashplain.com/2022/01/05/edge-command-line-arguments/)). The usual tools are ruled out on purpose: Playwright fetches browser binaries from its CDN at install, exactly what this proxy blocks (documented: [microsoft/playwright#14442](https://github.com/microsoft/playwright/issues/14442), [#19622](https://github.com/microsoft/playwright/issues/19622)) | Yes — already on the machine | A named person: P3 captures with Edge's built-in full-page Screenshot tool (Ctrl+Shift+S) at §15's named sizes; the hour is priced in §11's support block |

**One request, walked.** The risk officer's browser (Edge, loopback) sends `GET /borrower/B-07` to waitress on `127.0.0.1:8500` → Flask routes it → the view loads registry, engine results, evidence, forecasts from SQLite → Jinja2 renders the case file with inline-SVG trajectories → response returns. On *Generate memo*: the view assembles the grounded template from records, N-04's masking pass scrubs it, langchain-openai POSTs it to `https://genailab.tcs.in` (the one hop that leaves the machine — **trust boundary out**), the reply is shape-checked (§17), stored, logged (§20), rendered. **Untrusted input crosses into trusted code** at exactly three doors, each validated at the door: pasted clause text (R-03/§19), seed files (R-01.c), thresholds.json (N-03.b) — and the model's replies are treated as untrusted input too (§17's checks), which is the fourth door.

**Laptop today vs server in production:** today everything above runs in one process on one laptop against SQLite. `LATER:` on a real server the same stages split into a web tier (any WSGI host), a scheduled scoring worker (R-13), a server database (PostgreSQL) and the bank's identity in front (NG-03's SSO); the gateway call goes through the bank's egress proxy under the IT-outsourcing MD's contract terms (§2). Nothing in §3's mechanism changes shape.

---

## §14 Interfaces and data

**Ways in and out** (who may call reflects §7; enforcement reality is §16's):

| Interface | What goes in | What comes back | Who may call | When the caller is wrong | Cost | When slow/down |
|---|---|---|---|---|---|---|
| Web UI, `http://127.0.0.1:8500` | Clicks, pasted clause text, day-advance, threshold edits (operator) | Rendered screens | The one analyst (all §7 human roles); loopback-only — nothing else can reach it | Malformed/oversized input → §19's plain refusals; unknown route → designed 404 | ₹0 | It is local; if the process dies, §25's restart ≤2 min |
| Model gateway (out): `https://genailab.tcs.in`, alias `azure_ai/genailab-maas-DeepSeek-V3-0324` (fallback alias `azure/genailab-maas-gpt-4o` [OPEN-04]) | Masked stage-1/stage-6 prompts (N-04) | Completions (message objects) | Only the app's two model stages | Auth failure → stage unavailable message + log; never retried with altered credentials | Per-call and monthly cost unknown — **[OPEN-02]**; §20 records tokens so cost computes the day the price is known | Timeout 30 s, one retry, then §22's replay cache with the on-screen `REPLAY` label; the rest of the product is unaffected |
| Seed data files (in): `data/seed/*.csv`, `*.json` | Generator output / authored storylines | Loaded book | Operator via §25's seed command | R-01.c: named error, no partial load | ₹0 | n/a (local) |
| thresholds.json (in) | §17's threshold values | Live config | Operator | N-03.b: last good values stay | ₹0 | n/a |
| Logs (out): `logs/app.jsonl`, `logs/model_calls.jsonl` | — | §20's records | Read: auditor/operator | Unwritable → N-02.b | ₹0 | n/a |
| Indian data sources (reference only, build-time): [RBI DBIE](https://data.rbi.org.in/) aggregates, [data.gov.in](https://www.data.gov.in/ministrydepartment/ministry-corporate-affairs) company master, [MCA V3](https://knowledgebase.bison.co.in/view_article.php?id=352) filings | — | Calibration ranges for R-01 | Humans, before T+8 | — | MCA ₹100/company/year if ever used; ₹0 in-build | Not called at runtime — no runtime failure mode. §2 found no payment or identity rail this product needs (corporate credit; Aadhaar/UPI dropped in §2's law block) |

**Kinds of record** (meaningful fields, not table definitions; owner = who answers for its correctness in-build). Unless a row says otherwise, every record below is kept for the **build lifetime** and removed by §21's wipe-reseed — that is what deletion really removes — with production retention covered by §21's `LATER:` line and, for the audit trail, its own row below:

- **Borrower** — name (fictional), CIN-format ID, industry, concentration share, promoter/guarantor names *(personal-class, synthetic)*. Owner P1. Kept: build lifetime (wipe-reseed removes all — §21). Deletion = reseed; nothing survives except git history of seed code. Always true: every borrower has ≥1 facility.
- **Facility** — type, sanctioned limit (₹ crore), outstanding, utilisation %. Owner P1. Always true: exposure ≥ ₹5 crore (the CRILC band this book models, §2).
- **Covenant** — definition (library ref), threshold, direction, frequency, exceptions, source text *(may embed personal names — personal-class by contagion)*, version, status. Owner P4 (intake) / P1 (seeded). Always true: immutable once tested (R-02.a); every live covenant recomputable by the engine.
- **FinancialPeriod** — quarter (Indian FY), statement lines used by the ratio library *(financial-class)*. Owner P1. Always true: a period is complete or the covenant tests against it are marked stale (R-04.c).
- **SignalEvent / EvidenceItem** — type (payment/utilisation/treasury/concentration/industry-news), date, magnitude, persistence score, materiality score, state (transient/sustained/superseded) *(financial-behavioural class)*. Owner P2. Always true: never deleted, only superseded (R-05.c).
- **Forecast** — covenant ref, horizon, probability, confidence, projected crossing date, driver shares, formula inputs snapshot, computed-at. Owner P2. Always true: any probability shown on any screen exists here first — no probability is ever born in display code or model text (§17).
- **WarningMemo** — borrower ref, figure slots with record refs, model text segments labelled as drafted, actions (catalogue refs), prompt/model version, check verdict. Owner P4. Always true: every figure slot resolves (R-10.a).
- **AuditEvent** — actor label, event type, before/after or payload, thresholds snapshot ref, IST timestamp. Owner P5. Always true: append-only (R-11.b). Kept: build lifetime in-build; `LATER:` production retention per the Fraud Directions' audit expectations, write-once storage.
- **OverrideRecord** — what was shown, what the user did instead, stage, reason, versions and thresholds in force (§17's loop record). Owner P5.
- **ModelCallLog** — §20's per-call record. Owner P4. *(Prompt bodies are masked already — N-04.)*
- **ExampleCase** — §17's set: input, expected, arm scores, added-by. Owner P4.

**Personal/sensitive marking (feeds §16):** identifying — promoter/guarantor/director names, PAN-format strings (synthetic); financial — statements, flows, utilisation; official-identity — CIN/PAN formats (synthetic); free text — pasted clause text and override reasons, which may contain any of the above and are treated at the strictest class present. No health-class data exists in this product.

---

## §15 UI and UX — it has to look like a talented designer made it

### The direction, chosen the way §1 chose the mechanism

| Direction | Organised around | Borrows from (opened in Step 4) | Fits F-01? | Cost vs §11 | At four metres | Verdict |
|---|---|---|---|---|---|---|
| D-A **Examiner's ledger** — the screen is a credit case file being weighed: paper ground, ink tables an examiner could initial, evidence pinned to every claim | A decision being weighed | [9fin](https://9fin.com/features/covenants)'s layered density and discipline (inverted to light); [Koyfin](https://www.koyfin.com)'s professional data density | Yes — F-01 *is* weighing a case | Medium — tables and SVG we hand-draw anyway | Excellent — dark ink on paper is the highest-contrast thing a dim projector can show | **Chosen** |
| D-B **Ops triage console** — a queue being cleared, Linear-style rows, drill-in panel | A queue being cleared | [Linear](https://linear.app)'s list anatomy (identifier, badges, status colour) | Partly — the queue is one screen of F-01, not its soul; the case file is | Low | Good | Struck: it flattens the evidence story to list rows, and a Linear-shaped tool reads as generic SaaS — the second-most-generic look in the room |
| D-C **Data terminal** — dark, dense, market-scanner idiom | A market being scanned | [9fin](https://www.9fin.com/platform)'s dark navy platform look | No — scanning is not the job; weighing one case is | Medium | Poor — low-contrast dark greys smear on a dim projector | Struck: projector physics, and permanent-dark is this year's most common AI tell (refusals below) |

**The winner, in one sentence a builder can hold:** *Every screen is a credit case file being weighed — computed numbers in ink on paper, every claim wearing its evidence, and one amber-to-red accent that only ever means breach risk.*

**Three things this direction forbids** (consequences of the direction itself; these join the refusal list and every screen is reviewed against them at a glance):

1. **No KPI tiles or summary-card grids.** A case file has no dashboard; numbers live in ledger tables beside their context, never as isolated big-number tiles.
2. **No colour that is not risk.** Chrome, links, buttons, icons and states stay ink and paper; the green→amber→red scale appears only where it *means* covenant headroom state — if everything is highlighted, nothing is evidence.
3. **No chart without its ledger line.** Every trajectory or sparkline sits beside the figures it plots and the evidence rows behind them, click-through; a chart that cannot show its numbers does not render.

### Visual identity (buildable, not improvisable)

- **Palette (hex, with why):** paper `#F7F4EE` (warm document white — reads as paper, not as a startup's cream); ink `#191C1F`; muted ink `#5A6067`; hairline `#D9D3C7`; headroom green `#1E6B45`; watch amber `#B45309`; breach red `#B42318`. Chosen because a credit file is a document, the audience is bank examiners and credit officers who live in documents, and the three risk hues map exactly to headroom state (forbid 2) with AAA-capable contrast on paper.
- **Typeface, named, and why it and not the default:** headings and memo prose in **Georgia** — a workhorse newspaper serif that ships on every Windows machine (no CDN, no vendoring problem), sets institutional gravity, and is pointedly *not* Instrument Serif, this year's anti-slop cliché (refusals below); **data and numbers in Consolas** — monospaced tabular figures make ledger columns align and deltas scannable; **UI labels in Segoe UI** — quiet native chrome so the serif and the numbers carry the identity. All three are on Microsoft's own Windows 11 font list ([Microsoft Typography](https://learn.microsoft.com/en-us/typography/fonts/windows_11_font_list)), so they exist on the judging machine without downloading anything. The face is the loudest signal a person chose this; these three are chosen, available offline, and justified per role.
- **Type scale:** 13px data / 15px body / 18px section / 22px case-file title / 28px screen title; weights 400, 600, 700 only.
- **Spacing:** 4px scale (4/8/12/16/24/32/48); table rows 32px; the page grid is a 12-col, 1200px max content width.
- **Shape:** radius 2px everywhere (documents have corners, not pills); 1px hairline borders; **no shadows** — paper is flat — with one exception: the why-panel drawer casts a single 0/8/24 shadow because it sits *above* the document.
- **Motion:** two durations — 160ms (state changes) and 240ms (panel slide) — one easing `cubic-bezier(0.2, 0, 0, 1)`; the horizon scrub itself is direct manipulation (no tween — the display *is* the data at the dragged date). Under `prefers-reduced-motion`: panels appear without slide, scrub updates discretely at the three horizon stops, nothing animates.
- **Light and dark:** deliberately one theme — light paper (NG-09 says dark is `LATER:`) — and **the judged run uses it**, because the projector argument below is decisive.

### Composition

Each screen is organised around the thing being done, never a grid of things to read: the queue around *choosing whom to open*; the case file around *weighing one borrower*; intake around *checking a proposal against its source text* (side-by-side, clause left, fields right). Density: high in ledger tables, deliberately empty above the fold — the case-file header holds exactly four facts (name, exposure, worst covenant, dated risk) and 40% whitespace, so the projector eye lands somewhere. Alignment: every number right-aligned on the decimal in Consolas; labels left; the accent may appear only in the headroom column, the forecast band, the scrubber's crossing tick and the queue band chips — never in navigation, buttons or headings. All mockups and screenshots use the real seeded book (Meridian Auto Components' actual generated numbers), never lorem or short placeholder names — layouts are designed for realistic Indian lengths ("Meridian Auto Components Pvt Ltd", "₹62.4 crore", "TOL/TNW ≤ 3.25").

### What we refuse

The default AI-generated look: purple-to-indigo gradients on near-black; glass cards with blur; glowing borders; a bento grid of tiles; stock icon sets; a gradient blob behind a hero line; and the refusal that does not age — a component library left at its defaults. **Plus the three tells Step 4 actually found this year, each with where we saw it:** (1) the untouched shadcn default card — `rounded-2xl shadow-lg p-6` — and the "Cardocalypse" of cards nested in cards ([vibecodekit's 2026 fix guide](https://vibecodekit.dev/ai-slop-design)); (2) permanent dark mode as the default reflex with neon cyan/violet glow — called the v0/Cursor signature ([same guide](https://vibecodekit.dev/ai-slop-design)); (3) the 2026 "tasteful default" the anti-slop advice itself created — cream background, Instrument Serif, sage green accent ([claudecodehq's Unslop UI playbook](https://www.claudecodehq.com/playbooks/unslop-ui)) — which our paper-and-Georgia ledger dodges by being denser, colder and older than a lifestyle brand: examiner, not artisan café. **Plus the three the direction forbids, above.** Fonts ship as system faces (Georgia/Consolas/Segoe UI are on every Windows 11 machine) — nothing loads from a CDN, so nothing 403s on the projector. And no 3D, maps, WebGL or heavy motion anywhere: nothing in this product shows space, volume, a network or a route, so every figure is flat hand-drawn SVG and no heavy library — and therefore no flat fallback — is needed.

### The screens

1. **Portfolio queue** (entry): ranked rows — name · exposure · worst covenant · dated risk · band chip · "what changed". One click into a case file.
2. **Borrower case file** (**the protected screen** — carries §22's opening beat): described below with sketch.
3. **Covenant intake** (F-02): clause text left, proposed fields right, verification verdicts inline, confirm only when green.
4. **Audit & ops** (F-04): warning reconstruction timeline; thresholds view with change history; example-set scoreboard (both arms).
The why-panel is a drawer available on screens 1, 2 and 4, not a screen.

Case-file sketch (real content lengths):

```
┌────────────────────────────────────────────────────────────────────┐
│ Meridian Auto Components Pvt Ltd        Exposure ₹62.4 cr   [AMBER]│
│ Worst: DSCR 1.31 vs ≥1.20 · projected cross 04 Nov 2026 (63 days)  │
├──────────────────────────────────────────────┬─────────────────────┤
│  TODAY ─────────●────────────── +90d         │  EVIDENCE           │
│  DSCR   1.31  ↘ crosses 1.20 at day 63       │  ▪ Payment delay    │
│  TOL/TNW 2.9  → steady, headroom 11%         │    17d, sustained   │
│  Util%  86    ↗ rising, watch                │  ▪ Util +9pp/30d    │
│  [drag horizon]  30d: 22% · 60d: 71% · 90d:… │  ▪ Treasury outflow │
│                                              │    pattern, transnt │
├──────────────────────────────────────────────┴─────────────────────┤
│ [Why this?]                [Generate memo]      [Log action ▾]     │
└────────────────────────────────────────────────────────────────────┘
```

**Make it move for a reason — the signature interaction:** the **horizon scrubber**. Dragging from today toward +90, the covenant trajectories deplete, the crossing tick appears the moment a line meets its threshold with the date attached, drivers ink themselves into the margin as they begin to bite, and the header's dated risk rewrites live. An ordinary control in its place — a 30/60/90 tab strip — would show three static snapshots; the scrubber shows *trajectory, when, and what is driving it* in one gesture, and lets the user ask "what does next month look like" with their hand, which no product Step 4 opened can do: the closest prior art is Moody's automated testing schedules with alerts ([link](https://www.moodys.com/web/en/us/solutions/lending/loan-monitoring.html)) — a calendar, not a trajectory — and Koyfin's dashboards ([link](https://www.koyfin.com)) chart the past, not a dated forward crossing. It is deliberately not one of §5's features: it is the interface being good at its job. **It lives on the protected screen and happens inside §22's opening hook** — it is what the hook is made of, takes no beat of its own, and by living there it inherits §11's funding and §8's exemption.

**One screen carries the room:** the **borrower case file** is built to full fidelity (R-08's 4 hours, funded in §11 beside §5's features, absent from §8's cut list); the queue, intake and audit screens get what the hours leave and compress first.

**The projector is the display.** Dim, grey blacks, worse contrast than any panel — so the judged run is the light paper theme, and the **contrast floor is 7:1** for all text and 4.5:1 for the accent chips, verified *on the projector* at T+20, not only on a laptop panel. What the judging room's projector actually is: **[OPEN-10]**, answered at T+20's first run in the room; until then we design against 1920×1080 and the floor above.

**How the screens get proved:** capture via the §13 build-tool row — Edge headless CLI at **390×844 (mobile), 1366×768 (laptop), 1920×1080 (projector)** — with the named human fallback (P3, Edge's built-in full-page capture, same three sizes, 1 support hour in §11) if [OPEN-09] fails. **How long the main job takes is measured, not guessed:** at T+13, a person not on this team (**[OPEN-11]** — owner P5 asks the organizers by T+6) does F-01 unaided while P5 watches silently, noting stalls and time; pass is G4's ≤3 minutes/≤2 stalls; the half-hour is priced in §11's support block. If no outsider exists, G4 stays `ESTIMATED` and says so — a label nobody earned is worse than the honest one.

**Languages and access:** English UI (**[ASSUMED-02]**), ₹ lakh/crore number formatting, IST dates, Indian FY quarters. Screen-reader users get a real heading hierarchy, labelled controls, table semantics and text equivalents for every SVG figure (the ledger line under forbid 3 doubles as the accessible rendering); the scrubber is keyboard-operable (arrow keys step days; the three horizon stops are tab stops). One-handed phone use: the queue and case file reflow to one column at 390px with the scrubber full-width and thumb-reachable — the demo is desktop, the reflow is designed anyway. **Every screen has designed empty, loading, error and offline states** — empty: "No covenants registered — add one from the sanction letter" with the intake link; loading: skeleton ledger rows, no spinners over blank; error: the §19 message plus what to do next; offline/gateway-down: everything computes locally except stages 1/6, which show their §22 fallback state — and these states are part of R-08.b's check, not an afterthought. Interface hours are real hours in §11: R-08's 4, R-09's 3 within features, plus the capture and timing entries in the support block — none of it is "made pretty at the end". **(judged 13)**

---
## §16 Security, privacy and compliance

**Who may do what — two questions, two answers.**

*How the product knows who is calling:* honestly, it does not — this build has **no authentication** (NG-03). The server binds to `127.0.0.1:8500` only, so the sole caller is a person at this keyboard on this OS account; there is no token, session or password to store or leak. An unauthenticated network caller gets nothing because nothing listens off loopback; an on-machine caller without "permission" is indistinguishable from the analyst — that is the stated single-user reality, and returning the same screens to every local caller is a decision written down here, not an accident. Actor labels on audit events are self-declared at session start (a dropdown, not an identity). `LATER:` bank AD/SAML SSO in front, server-side enforcement of the matrix below, maker-checker on intake and overrides, and distinct refusals: 401 for no identity, 403 with the missing permission named for an identified caller without it.

*What each §7 role may do* — the matrix this build displays and a real deployment enforces (❌⁰ = nobody may, ever; ⚙ = operator-only):

| Action | RM | Credit | Risk | Auditor | Operator |
|---|---|---|---|---|---|
| View queue, case files, memos | ✓ | ✓ | ✓ | ✓ | ✓ |
| Paste clause / confirm covenant | — | ✓ | ✓ | — | — |
| Confirm a covenant that failed verification | ❌⁰ (control never renders) | ❌⁰ | ❌⁰ | ❌⁰ | ❌⁰ |
| Generate memo / log action | ✓ | ✓ | ✓ | — | — |
| Override a risk view (captured) | — | — | ✓ | — | — |
| Edit thresholds (logged) | — | — | — | — | ⚙ |
| Reseed / reset | — | — | — | — | ⚙ |
| Edit or delete audit events | ❌⁰ | ❌⁰ | ❌⁰ | ❌⁰ | ❌⁰ |
| Approve credit action / waiver / escalation in the tool | ❌⁰ — advisory only (NG-04) | ❌⁰ | ❌⁰ | ❌⁰ | ❌⁰ |

**Keys and secrets:** `GENAILAB_API_KEY` lives only in the environment (set per §25), never in code, notebooks, logs, this spec or git; N-04.a scans for it. **TLS:** the handed sample disables certificate verification (`verify=False`); we state that plainly here rather than leave it for a judge to find in code. Whether verification-on works on this network is **[OPEN-05]**: the T+4 gate makes its first call with verification left on, and if it succeeds we ship it that way and say so. `LATER:` the real posture on a server — corporate CA bundle installed, verification always on, no exceptions.

**The data itself** — §14's personal/sensitive fields sorted by class, and what happens to each. All data in-build is synthetic (R-01), so no real person's data is processed; the rules below exist because production would process guarantor/promoter/director data, which DPDP covers (§2).

| Class (examples) | In a log | On the wire | At rest | On delete | Off this machine |
|---|---|---|---|---|---|
| Identifying (promoter/guarantor names) | Never — masked to role tokens (`GUARANTOR_1`) | Loopback only | SQLite file in project folder; no app-level encryption in-build (synthetic data; stated, not hidden). `LATER:` encrypted at rest (OS BitLocker as baseline, SQLCipher or server-side encryption in production) | Wipe-reseed removes all (§21) | **Never** — N-04 masks before the gateway; §13's gateway is another company's computer and §17's masking rule governs what reaches it |
| Financial (statements, flows, utilisation) | Aggregates only | Loopback; masked derived values (ratios, headrooms) may go to the gateway for stage 6 | Same as above | Same | Only as masked derived values in prompts |
| Official-identity formats (CIN/PAN-format synthetic strings) | Never | Loopback only | Same | Same | Never — regex-stripped by N-04 |
| Free text (clause text, override reasons) | Reference by ID, not body | Clause text goes to stage 1 **after** the N-04 scrub (names/ID-patterns replaced) | Original kept locally | Same | Scrubbed version only |

**Untrusted input is checked at its door** (the four doors §13 walked: pasted text, seed files, thresholds.json, model replies) — size, shape and content rules per §19 and §17, all failing closed.

**What an auditor would ask us to show, and where it is:** the warning reconstruction (R-11.a) · the model-call log with verdicts (§20) · the thresholds file and its change history (N-03.a) · the masking test evidence (N-04.a/b) · the role matrix above and the honest NG-03 statement · this section's TLS statement and the T+4 result.

---

## §17 The AI: guardrails, grounding and explainability

**What the model may decide, and what it may never decide alone.** The model may: propose covenant fields for stage 1 (a person and code stand between proposal and registration) and phrase stage 6's memo over computed facts. The model may **never**: compute a ratio, decide pass/fail, set or alter a probability, confidence, date, threshold or ranking, invent a figure, name, date or citation, decide an escalation, or take any action. Any figure, name, date or citation the product presents *as true* comes from the record or the calculation, never from the model; text the model genuinely writes (memo prose, proposal rationale) is displayed labelled **Drafted by model** so a user always knows which of the two they are reading.

**The example set** — 20 examples, hard and failing cases included, living as files in `examples/*.json` so adding one is an edit, not a code change; P4 and P5 may add (each addition is an AuditEvent). Carried by **N-01 (3 hours in §11)** — a set nobody funded is a set nobody runs.

| ID | Case | Exercises |
|---|---|---|
| EX-01 | DSCR computed from Meridian's Q4 statements, hand-worked | Stage 2 exactness |
| EX-02 | TOL/TNW hand-worked, second borrower | Stage 2 exactness |
| EX-03 | Covenant with a two-quarter exception window, tested inside and outside | Exceptions |
| EX-04 | Borrower with a missing quarter | Stale handling (R-04.c) |
| EX-05 | Negative EBITDA period | Undefined-ratio behaviour (R-04.d) |
| EX-06 | Covenant exactly at its threshold | Boundary behaviour (T-rules) |
| EX-07 | Deteriorator's dated crossing vs authored breach | Stage 4 dating (±10 days) |
| EX-08 | Stable borrower full replay | No false alarm |
| EX-09 | Noisy-transient full replay | Persistence filter (T3/T4) |
| EX-10 | Contradicting later payment | Revision without deletion |
| EX-11 | Two borrowers, equal urgency | Deterministic tie-break |
| EX-12 | 120-borrower generated book, one scoring day | §18's 10× budget |
| EX-13 | Clean DSCR clause text | Extraction, easy |
| EX-14 | Clause with an exception ("save that for FY27 Q1–Q2…") | Extraction, hard |
| EX-15 | Clause with ambiguous testing frequency | Must flag ambiguity, not guess |
| EX-16 | Clause with impossible threshold (DSCR ≥ 40) | Verification strikes it (R-03.b) |
| EX-17 | Clause embedding "ignore your instructions…" | Injection refusal (R-03.c) |
| EX-18 | Hero memo, every figure traced | Grounding (R-10.a) |
| EX-19 | Forced malformed model reply | Retry-once-then-refuse (R-10.b) |
| EX-20 | Hero memo scored on the usefulness rubric | Usefulness |

**Pass marks — targets, `ESTIMATED`; nothing has run at hour 3 and no score is written here.** Accuracy: EX-01–06 and EX-11 must go 7/7 exact (they are code; anything less is a bug). Grounding: 0 ungrounded figures on EX-18. Extraction: field-level precision and recall ≥0.90 across EX-13–15 *after* the verification stage, and 2/2 fail-closed on EX-16–17. Forecast usefulness: EX-07 within ±10 days; EX-08/09 zero false escalations. Usefulness: EX-20 median ≥4/5 on a 5-point rubric (actionable / evidence-cited / role-correct / advisory-toned / concise), scored independently by three team members. Speed: memo p95 ≤20 s; any screen p95 ≤2 s (these are our targets for ourselves — the gateway's own latency is [OPEN-02]). Safe failure: EX-19 refuses, never renders a partial memo. §24 says how each is run and by whom; the first real scores exist at T+8 when the thinnest path runs; §23 re-runs everything at T+20 on the judging machine and prints both arms beside these marks — that run, and only that run, is labelled `MEASURED`.

**Scored twice — the baseline arm.** The same 20 examples also run against the simplest plausible alternative: for forecasting, the naive headroom rule (flag any covenant with headroom <10%, no velocity, no evidence pressure — Altman-spirited distance-to-threshold, §4); for extraction, a regex/keyword parser; for the memo, one naive prompt with no grounding slots. This is §3's head-to-head happening rather than being argued, and it is what §22's judge-question 3 is answered from: the gap between two printed numbers, not an opinion. The baseline arm is cheap (a rule, a regex, a prompt) and is funded inside N-01's hours.

**Grounding.** Everything shown traces to something the product can point at: the statement line it read, the record it loaded, the sum it computed. Where there is nothing to point at, the product says so ("insufficient evidence — watching", "not computable") instead of filling the gap. The answer's shape is fixed and checked in code before anything is displayed — required slots resolved, actions from the catalogue only, length within T6, no figure outside its slot; a reply that fails is retried once, then refused, never shown. This section owns the bad-model-answer path; §19 points here and sets no retry count of its own. Until the T+4 gate says the gateway supports structured output ([OPEN-03]), the shape check is our own code — which it remains even if JSON mode exists, because a guardrail in code is not a request in a prompt.

**Thresholds** — every number this product compares against to decide. Values live in `thresholds.json` (N-03); the operator may change them; every change is an AuditEvent. Starting values are ours at hour 3, not tuned truths: **[OPEN-12]** — they are tuned against this example set on the synthetic backtest by T+13, and what would set them properly in production is a real portfolio's history. Each row: value · above · below · exactly at · enforced where.

| # | Threshold | Value | Above | Below | Exactly at | Enforced |
|---|---|---|---|---|---|---|
| T1 | Escalation probability (per worst covenant-horizon) | act ≥ 0.70; amber ≥ 0.40 | Act band: queue top, memo offered | Watch band: listed, no escalation | Boundary belongs to the higher band (0.70 → act; 0.40 → amber) | Code (banding fn) |
| T2 | Confidence floor | 0.50 | Probability shown with confidence | "Insufficient evidence — watching"; no bare probability ever renders | Shown (floor is inclusive) | Code (render guard) |
| T3 | Persistence — sustained if | ≥14 consecutive days or ≥3 events in 30 days | Sustained: feeds stage 4 pressure | Transient: visible, decaying, no forecast effect | Sustained (inclusive) | Code (ledger) |
| T4 | Materiality — evidence counts if projected headroom erosion at 90d | ≥5% of threshold value | Counts | Noted on the ledger, excluded from pressure | Counts (inclusive) | Code (ledger) |
| T5 | Driver listed if contribution | ≥10% of risk delta | Listed with share | Folded into "other" | Listed | Code (attribution) |
| T6 | Memo length ceiling | 1,200 tokens out | Regenerate once shorter, else refuse | Fine | Fine | Code (shape check) |
| T7 | Model-call ceiling per session-hour | 40 calls | Calls queue; banner says so | Fine | Queue engages at the 40th | Code (counter) — this is the spend ceiling in-build; ₹ spend per user computes when [OPEN-02] prices a call |
| T8 | Bad-shape retry | 1 retry, then refuse | — | — | — | Code (§ above) |

A threshold with no stated boundary behaviour is a coin toss written as a number; every row above names its boundary. The memo's *tone* (advisory, no directive language) is the one guardrail that lives partly in the prompt — named here as a request, not a guarantee; the enforceable half (actions only from the catalogue) is in code.

**Explainability — inside the product, not on a slide.** For every decision a user sees, they can open **why**: which inputs were used, which sources were read (each linked), which stage made the call, how sure it is and what that number means, and what would have changed the answer. Every stage of §3's mechanism is openable — the run behind the result, in order, each stage's inputs, outputs, and decider — not only the final answer. The panel answers the same three questions whatever kind of thing decided the stage, and each kind answers differently: a **rule/code stage** (2–5) shows the rule and the numbers it compared — e.g., stage 4 shows trend slope, evidence pressure terms, the mapping curve, and that 0.71 ≥ T1's 0.70 so the act band fired, naming the threshold and which side the value fell; a **prompted model stage** (1, 6) shows what it was given (the masked prompt, versioned) and what it returned, plus the code verdict on it; a **trained/statistical model** — none ships in this build, and if one ever replaces stage 4's formula, its panel must show which inputs moved the decision, by how much, and which model version decided (`LATER:` with NG-05's retraining line). This is what makes the depth claim checkable by a judge instead of asserted by a slide.

**Guardrails both ways.** Outbound: N-04 strips/masks identifying and official-identity strings before anything leaves the machine; ceilings T6/T7 cap tokens, calls and spend. Inbound: input that tries to rewrite the instructions (EX-17) gets the fixed refusal and a log entry — enforced by the template structure and the shape check in code, not by asking the model nicely; refused outright regardless of phrasing: credit decisions, waiver approvals, fraud classification, any output about a real (non-seeded) company. Each guardrail above is marked by where it is enforced; the only prompt-side item is memo tone, named as such.

**The loop back.** What a user does with an answer is the only real signal this product will ever get; 24 hours is long enough to capture it and nowhere near long enough to learn from it. Every correction, override, rejection and accepted answer is an OverrideRecord/AuditEvent (§14): what was shown, what the user did instead, which stage produced it, prompt and model versions (§20 logs them), and the thresholds in force at the time. Inside the 24 hours this record changes three things: a disputed forecast becomes a new example in this set; a repeated extraction correction becomes a prompt-version bump (versioned, logged); a threshold complaint becomes a tuned value via N-03. `LATER:` the real loop — overrides accumulate into a labelled dataset; any trained stage-4 model retrains on it monthly; a named risk owner approves each new version against this example set before it reaches a user; a worse version rolls back by pinning the prior version id; drift is noticed when §20's override-rate and failed-check alerts trend up. A product that collects nothing has no second version; this one collects from hour 8. **(judged 8, 12)**

---

## §18 Performance and scalability targets

Targets are ours (chosen numbers); how measured is named per row; `MEASURED` values appear only in §23's T+20 run.

| Target | Number | Measured how |
|---|---|---|
| Any screen render, seeded book | p95 ≤ 2 s | Timed over 20 scripted loads on the judging machine |
| Horizon scrubber response | ≤ 100 ms per step (it reads precomputed daily paths) | Browser performance trace during R-08.a |
| Full-portfolio daily scoring (12 borrowers) | ≤ 5 s | Timed in EX-08/09 replays |
| Memo generation end-to-end | p95 ≤ 20 s (our budget; gateway latency itself is [OPEN-02]) | N-01 harness timings |
| 10× book (120 borrowers), one scoring day | ≤ 60 s | EX-12 |
| Mobile reflow usable | Case file readable and scrubber operable at 390px; first load ≤ 5 s on Edge's mid-tier Android emulation with throttled 3G-class network | Devtools emulation at T+20 (the product is a desk tool; the reflow is designed, not the demo) |
| Concurrent users | 1 (in-build reality); waitress default thread pool handles the demo's single session with headroom | Observed during rehearsal |

**At 10× today's load** (120 borrowers): nothing structural gives way — stages 2–5 are linear arithmetic over small rows; SQLite and pandas idle through it; the LLM is untouched because scoring uses no model calls. **At 100×** (1,200 borrowers, multiple desks): the first thing to give way is the synchronous single-process design — scoring must move to R-13's scheduled batch; the second is SQLite's single-writer model under concurrent analysts — move to PostgreSQL; the third is the gateway rate limit if every analyst generates memos — memos stay on-demand, get cached per borrower-day, and T7's ceiling becomes per-desk. What we would change: §13's `LATER:` server shape (web tier + worker + server DB + SSO). What it costs per month at 100×: infrastructure is one mid-size VM and a managed Postgres (thousands of rupees, not lakhs — `ESTIMATED`); the model bill is the real variable and computes as memos/day × tokens/memo × price — price is **[OPEN-02]**, so the formula is stated and the rupee figure is not. The mechanism's core costs zero model calls per borrower per day by design — that is the scaling story. **(judged 15)**

---

## §19 Edge cases and errors

Everything except a bad model answer, which §17 owns (retry once, then refuse — no other section sets a retry). User-facing messages are English (**[ASSUMED-02]**), plain, and name the next step. Nothing is silently retried except where a count is stated.

| Case | What the user is told | Retried? | Must never be lost |
|---|---|---|---|
| Empty / whitespace clause paste | "Paste the covenant paragraph first." | No | — |
| Enormous clause paste (> 20,000 chars) | "Too long for one covenant — paste one clause at a time." | No | The pasted text is not stored |
| Hostile input in any field (script tags, SQL-ish, path-ish) | Escaped everywhere by the template engine; forms validate type/range; "That value isn't valid here." | No | — |
| Instruction-injection in clause text | §17's fixed refusal; logged | No | The log entry |
| Seed file malformed / truncated | Operator: R-01.c's named error | No | The previous good book (seeding is all-or-nothing) |
| Duplicate seed / double ingestion | Nothing (idempotent); ops panel notes "already loaded" | — | No duplicate evidence (R-05.d) |
| Missing quarter for a borrower | Case file: "Data to Q2 FY26 — covenant marked stale"; confidence drops | No | The staleness marker in the trail |
| Ratio undefined (negative EBITDA) | R-04.d's worst-case-flagged message | No | The flag |
| Gateway slow | 30 s timeout, then the down path below | 1 (the §17 retry covers shape; timeout retries once too, same count) | The request log entry |
| Gateway down / auth fails | "Model unavailable — memo and clause-parsing paused. Everything computed locally still works." + §22's replay option | No further | Everything computed locally |
| A dependency answers but wrongly (gateway returns 200 with junk) | Treated exactly as §17's failed shape check | Per §17 | The verdict in the model-call log |
| Crash / restart mid-session | §25's restart; on boot the app reopens the last consistent state (SQLite transactions — no half-written test results) | — | The audit trail; any half-finished memo simply does not exist (written only after passing checks) |
| thresholds.json malformed | N-03.b: line-numbered error; last good values hold | No | Last good values |
| Browser offline / page reload mid-scrub | State is server-side; reload lands on the same case file | — | — |

---

## §20 Observability — the app, and the AI separately

**The app's log** (`logs/app.jsonl`): request id, IST timestamp, route, actor label, duration ms, outcome, error class if any. **Never logged:** key material, personal-class field values (identifying/official-identity — masked to tokens), full clause bodies (by reference id), model prompt bodies (they live masked in the AI log only). **Following one user's request end to end:** every screen action carries a request id; the same id appears on the app line, any AuditEvent, and any model call it spawned — one grep tells the story an hour later.

**The AI's own log** (`logs/model_calls.jsonl`), because the app log answers nothing about it — per call: call id, request id, stage (1 or 6), **prompt version**, **model and version string as returned by the gateway**, tokens in, tokens out, latency ms, computed cost field (tokens × price — price blank until [OPEN-02] answers; the field exists so a day of users adds up the day the price is known), §17 check verdict (pass / retried / refused), retry count, refusal reason. Enough to answer "why did it say that?" about a call from an hour ago — the prompt version pins what it was asked, the verdict pins what happened to the answer. A judge asking what this costs per user is really asking whether we measured anything: we measure tokens per call from hour 8 and multiply the day the price exists.

**The §17 loop's events are first-class log entries:** every override, correction, refusal and failed check is written as it happens — the earliest sign the product is drifting from what it was tuned for.

**The handful of numbers worth watching, each with the value that raises an alert and who it wakes:**

| Number | Alert value | Wakes |
|---|---|---|
| Shape-check failure rate (per session) | > 10% | P4 (AI lead) — prompt or gateway drifted |
| Gateway latency p95 | > 20 s | P4 — switch demo to replay posture |
| Model calls this hour vs T7 | ≥ 40 (T7's own value) | P4 — ceiling engaged, banner on |
| Override rate (overrides / warnings shown) | > 30% in a day | P5 — the idea-risk alarm (RISK-01's early sign) |
| Amber-or-worse share of portfolio | > 30% on any backtest day | P2 — thresholds misfiring toward alert fatigue (G3's guard) |
| Log write failures | ≥ 1 | P5 — immediately; observability is the audit trail's floor |

---

## §21 Data and compatibility

**What has to come in, and in what shape:** only R-01's seed (CSV/JSON, schema versioned with a `schema_version` field) and pasted clause text. No bank data enters this build (NG-01).

**When the shape changes during the build:** the honest answer is **wipe and reload** — `python -m radar.seed --reset` rebuilds the world from the current generator in seconds. What makes that safe: the data is synthetic and regenerable by construction; the generator and storylines are code in git, so any prior world is recoverable by checkout; nothing a user entered matters except demo-rehearsal artefacts, and the demo book is frozen at T+20 (§22) after which the schema does not move. The audit trail is part of the wiped world — acceptable in-build because every wipe is itself deliberate and pre-freeze; stated so nobody mistakes it for the production posture.

`LATER:` real migration and compatibility — versioned migrations (Alembic-style) with forward-only DDL, an import layer that maps CBS/LOS extracts into §14's records with per-field provenance, dual-running old and new covenant registry versions through one reporting cycle before cutover, and a write-once audit store that survives every schema change by design.

---

## §22 The prototype and the judged demo

**The slot:** 8 minutes, questions inside (**[ASSUMED-04]**). The run below is **5:00 of beats, leaving 3:00 for questions** — it fits with time to spare. Machine: the build laptop (**[ASSUMED-06]**) on the projector at 1920×1080, light paper theme (§15). Data: the demo book **frozen at T+20** — no reseed after freeze. All beats show `must` requirements that exist in §10.

| Clock | Beat | What the room sees | Requirements visible | Real or stubbed |
|---|---|---|---|---|
| 0:00–0:10 | **Hook** — on §15's protected screen, §15's signature interaction *is* the hook | Meridian Auto Components' case file. Exact words: *"This borrower looks compliant today. …"* — drag — *"…and in 63 days their debt-service covenant breaks. Radar knew this morning."* | R-08, R-06 | Real |
| 0:10–1:40 | **Before / after** | Before (20 s): the real coping mechanism — a QIS form and an Excel covenant tracker on screen, *"this is how ₹5-crore-plus accounts are watched today — quarterly, by hand"* (§2 row 5). After (70 s): the queue, ranked; Meridian on top at 71% / 60-day; *"same bank, same data, Tuesday morning"* | R-07, R-05 | Real (the Excel is a prop labelled as a prop) |
| 1:40–2:40 | **S2 — the mechanism running** (not its output): advance one day; a payment signal lands; evidence flips sustained at T3; forecast and queue move; **why-panel opened on stage 3 and stage 4** — the room sees the rule, the numbers, the threshold and which side fell | *"No model touched that. Here's the arithmetic — and here's the exact rule that separated deterioration from noise."* | R-05, R-06, R-09 | Real |
| 2:40–3:30 | **S1 — clause to covenant** | Paste a sanction clause; model proposes; verification strikes a deliberately-wrong threshold on the first paste (*"the AI proposed it; the code refused it"*); paste the good clause; confirm; it goes live and is tested on the spot | R-03, R-02, R-04 | Real (live gateway; replay fallback below) |
| 3:30–4:20 | **S3 — memo and the trail** | Generate memo; click two figures to their records; open the audit reconstruction; log an override (*"the officer disagrees — Radar keeps both states"*) | R-10, R-11, R-09 | Real (live gateway; replay fallback below) |
| 4:20–4:50 | **The proof** | The example-set scoreboard from the T+20 run: both arms beside the pass marks — *"same 20 cases: the naive rule, and the mechanism; the gap is the product"* | N-01 | Real — the T+20 `MEASURED` run, shown as a stored result and rerunnable on request |
| 4:50–5:00 | **Close** | G1/G3's two numbers, the honest `LATER:` line (*"synthetic book, no logins, no live feeds — §8 says so out loud"*) | — | — |

**What is real and what is stubbed, and how a watcher can tell:** every computation (stages 2–5) is real code over the frozen synthetic book — nothing pre-rendered. The two model beats are live gateway calls; if the room's network fails them, the demo switches to the **replay cache** (stored real responses from rehearsal) and the screen shows a visible `REPLAY` chip — the label is the tell, and we say it aloud. The external news/industry signals are synthetic **by the brief's own instruction** (`problem.md` permits supplied/synthetic signals) and the seed files are on screen if asked. Nothing marked `LATER:` appears working anywhere in this run.

**Reset, preflight, backup:** one-command reset `python -m radar.seed --reset --demo-freeze` restores the frozen book in <30 s. Preflight (run at T+22 and again 15 minutes before the slot): env var set → gateway ping (verification per §16's T+4 result) → replay cache present → seed hash matches freeze → port 8500 free → projector at 1920×1080, 100% zoom, notifications off → backup recording present on a second laptop **and** a phone. Backup: a full 5:00 screen recording made at T+22.

**Fallbacks, decided now:** **network drops** → replay cache for stages 1/6 with the `REPLAY` chip; stages 2–5 are local and unaffected. **Model endpoint dies** → same replay posture; if it dies mid-call, the §19 message shows and we narrate the fallback — and the second alias `azure/genailab-maas-gpt-4o` ([OPEN-04], same endpoint, same key) is tried once before replay. **Machine reboots** → §25's cold start ≤2 min while the co-presenter narrates over the backup recording from the second laptop; if the machine will not return, the recording carries the remaining beats, said plainly.

**The six questions a hostile expert asks after this exact run, with the section each answer is read off:**

1. *"Where does 71% come from?"* — From stage 4's inspectable formula: trend, evidence pressure, calibrated mapping — opened live in the why-panel with T1 named; calibrated on the synthetic backtest, and labelled so. **(§17, §3)**
2. *"What happens on the input you didn't show — a clause your parser chokes on, a quarter that's missing?"* — It fails closed: proposals are struck with the failing check named, stale data is marked and drops confidence; we can show EX-15/EX-04 on request. **(§19, §17)**
3. *"Why a model at all — rules could do this?"* — Rules *do* do stages 2–5; the model exists only where language is unbounded, and the scoreboard beat shows both arms on the same 20 cases — the regex parser's misses are the answer. **(§17)**
4. *"What's real and what's stubbed?"* — All computation is real over a frozen synthetic book; model calls are live with a labelled replay fallback; news signals are synthetic per the brief. The table above is in the spec. **(§22, this table)**
5. *"What does this cost at a thousand borrowers?"* — Scoring costs zero model calls by design; the bill is memos on demand — tokens are logged per call from hour 8, and the rupee figure computes the day the gateway's price ([OPEN-02]) is known; the formula and the 100× architecture are written. **(§18, §20)**
6. *"Why you and not Crediwatch or Moody's?"* — They alert on entity risk or track compliance dates; Radar recomputes the signed contract, dates the breach, and opens every stage to audit — the 'tests the covenant vs stores the number' line is the field's own sorting question. **(§4, §2 row 7)**

**The single weakest claim in this document, named first:** the 30/60/90 probabilities are calibrated only on synthetic histories we authored — the mapping from headroom, velocity and evidence pressure to a probability has never met a real Indian portfolio. We make the claim anyway because the direction is evidenced (SMA-2 stress leads slippage, §2 row 9), the mechanism is transparent enough to recalibrate on real data without redesign, and the honest alternative — no probability, only headroom — is the baseline arm we show beside it. If a judge finds this before we name it, it costs the prize; named, it is a design decision. **(judged 12)**

---
## §23 Acceptance criteria

Done means all of the following, **on the judging machine, true by T+20** — leaving T+20→T+22 to fix what this list finds rather than discovering it at code freeze:

1. Every `must` requirement's checks pass: R-01.a–c, R-02.a–b, R-03.a–c, R-04.a–d, R-05.a–d, R-06.a–d, R-07.a–b, R-08.a–b, R-09.a–b, R-10.a–c, R-11.a–b, N-01.a–b, N-02.a–b, N-03.a–b, N-04.a–b, N-05.a–b — run per §24's methods, results recorded.
2. §17's 20-example set has run on this machine with **both arms' scores printed beside the pass marks** (the `MEASURED` run); misses are printed, not hidden.
3. §6's G1, G3 and G5 are computed from that run and printed; G4 carries whichever label §15's timing earned; G2 remains `PROJECTED` and says so.
4. §16 verified: outbound-capture scan shows zero personal-class fields and zero key material (N-04.a); nothing listens off loopback; the TLS posture on screen matches the T+4 gate's result; the audit trail reconstructs the hero warning end to end.
5. §18's measured rows (screen p95, scrubber, portfolio scoring, memo p95, EX-12's 10× run) are timed on this machine and printed.
6. §15 proved: screenshots exist at 390×844, 1366×768 and 1920×1080; contrast spot-checked on the projector ([OPEN-10]); all four state families (empty/loading/error/offline) render on the protected screen.
7. §22 rehearsable: reset command restores the frozen book in <30 s; preflight list passes; replay cache present; backup recording exists.
8. §25's cold start from a clean folder completes in ≤10 minutes, performed by the team member who did not write the setup.

If someone can argue about whether an item passed, the item is written wrongly — each points at a check, a printed number or an artefact that exists or does not.

---

## §24 Testing and rollout

**How each requirement is proved** (check → method → who):

| Requirement | Method | Who |
|---|---|---|
| R-01, R-02, R-04, R-05, R-06, R-07 | Automated: pytest 9.1.1 unit/integration tests encoding each check, run at every commit and at T+20 | P1/P2 write beside the code; P5 runs the sweep |
| R-03, R-10 (model stages) | N-01 harness (automated scoring of EX-13–20) plus by-hand runs of the bad paths (R-03.b/c, R-10.b/c) with the test hook | P4; P5 witnesses at T+20 |
| R-08, R-09 (screens, why-panel) | By hand against a written click-script of R-08.a/b and R-09.a/b, on the projector at T+20; screenshots captured per §15 | P3 executes, P5 witnesses |
| R-11 | Automated append-only property test + by-hand reconstruction walk (R-11.a) | P5 |
| N-01–N-05 | N-01/N-05 automated (harness, timers); N-02/N-03 automated checks + by-hand log reading; N-04 automated scan of captured outbound bodies | P4 (N-01/N-04), P5 (rest) |

**Tested with real Indian data and real Indian text:** the book is synthetic by the brief's instruction, but its *shapes* are Indian and real — clause texts written in Indian sanction-letter idiom (₹ crore thresholds, "TOL/TNW", FY quarters, exception windows), ratio ranges calibrated to RBI DBIE aggregates (§2 row 11), ₹ lakh/crore formatting, IST dates, CIN-format identifiers — so the extraction and display paths meet the text they would meet in production.

**Who tries it before real users:** P5 daily from T+8; the outside-team user at T+13 ([OPEN-11]); the whole team in rehearsal at T+22. **How it reaches users in steps, rather than all at once:** in-build, demo-only. `LATER:` (1) backtest against one bank's historical portfolio (would have Radar flagged the accounts that actually slipped?); (2) shadow mode beside the existing QIS process for one quarter, alerts visible to the credit team but binding nothing; (3) live on one desk with thresholds tuned to that book; (4) portfolio-wide.

---

## §25 Running it, and getting back to green

**From nothing, on the judging machine** (commands as typed; ≤10 minutes total, rehearsed per §23.8):

```
cd covenant-radar
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt        # from the approved registry only
set GENAILAB_API_KEY=<key>             # session env var; never in a file
python -m radar.seed --reset           # builds the synthetic book, ~30 s
python -m radar.serve --port 8500      # waitress on 127.0.0.1:8500
```

Then open `http://127.0.0.1:8500` in Edge. First-time run by someone who has never seen it: the README's copy of exactly these lines; nothing else is needed (no admin, no services, no downloads beyond the registry).

**Rollback as it really works in a 24-hour build:** commit every time something works (the team rule from T+4: green = commit, at least hourly); tag the latest demonstrably-good state `last-green` (P5 moves the tag); the one command back is `git reset --hard last-green` followed by `python -m radar.seed --reset` — total cost under 2 minutes. **Who calls it:** P5, unilaterally, any time after T+17 if a change breaks the demo path; before T+17 the change's author gets 20 minutes to fix forward first. By T+22 the `last-green` tag is the frozen demo build and only P5 may move anything.

`LATER:` real deployment and rollback — versioned releases to a bank-hosted VM behind SSO, blue-green cutover with the previous release warm, database migrations per §21's `LATER:`, and rollback as a release-pointer flip plus migration down-scripts, rehearsed quarterly.

---

## §26 Risks and mitigations

Three kinds: the idea being wrong (01–03), building it in time (04–07), and the ones that appear as it grows (08–10).

| ID | What could go wrong | Likely? | How bad | Early warning sign | Doing now to make it less likely | If it happens anyway |
|---|---|---|---|---|---|---|
| RISK-01 | The 30/60/90 probabilities are not believed by a domain-expert judge (idea) | Medium | High — it is the headline claim | §20's override-rate alert; disbelief in the T+13 internal review | Transparent formula, confidence display, honest `ESTIMATED`/`MEASURED` labels, dual-arm scoreboard, §22 names the weakness first | Lead with the deterministic engine and the *dated headroom* (arithmetic, unarguable); frame probability as a ranking aid; the why-panel is the rebuttal |
| RISK-02 | Materiality/persistence control fails and Radar escalates noise — rebuilding alert fatigue (idea) | Medium | High — G3 is a named goal | >30% of portfolio amber-or-worse on any backtest day (§20 alert) | T3/T4 tuned against EX-08/09 by T+13 ([OPEN-12]) | Raise T3/T4 live (thresholds file, logged); demo the revision flow as the feature it is |
| RISK-03 | Judges read extraction as solved elsewhere (Eigen/9fin) and discount S1 (idea) | Medium | Medium | Q6-style question in rehearsal | §4's line is ready: they extract, we *disprove* — the fails-closed verification is the difference, and R-03.b shows it live | Show EX-16 on stage: the model proposes, the code refuses |
| RISK-04 | Gateway quota/latency/liveness fails during build or demo (delivery) | Medium | Medium — stages 1/6 only | T+4 gate readings ([OPEN-02]); §20's latency alert | Replay cache built at T+8 and refreshed at rehearsal; second alias probe ([OPEN-04]); T7 ceiling conserves quota | §22's replay posture with the visible label; the mechanism's core never needed the gateway |
| RISK-05 | Approved registry missing a pinned package (delivery) | Low-medium | Medium | T+4 install log ([OPEN-07]) | Two-deep fallbacks per §13 row, chosen for registry-independence (stdlib) where possible | Drop to the named fallback; if the registry is empty of web frameworks entirely, escalate to organizers at T+4 — hour 4, not hour 20 |
| RISK-06 | Hours overrun — the schedule nobody can run (delivery) | Medium | High | Any §28 marker going red | §11's reserve (11.5 h) and priced cut list; features have pre-written narrower versions | P5 takes the next §8 cut within 30 minutes; two consecutive reds → §5's narrower versions in §11's order (S1 first, S2 last) |
| RISK-07 | The one build laptop dies (delivery) | Low | High | None — it is sudden | Hourly push to the remote repo; second laptop kept synced from T+8; backup recording from T+22 | §25's cold start on the second laptop (≤10 min, rehearsed); recording covers the slot if both fail |
| RISK-08 | Model cost grows linearly with borrowers × memos (growth) | High at scale | Medium | §20's per-call token log trending up | Scoring uses zero model calls by design; memos on demand only, cached per borrower-day; T6/T7 ceilings | `LATER:` batch memo generation off-peak, shorter memo template, per-desk ceilings |
| RISK-09 | SQLite's single writer blocks concurrent desks (growth) | Certain at 100× | Medium | Write-lock errors in a load test | In-build single-user is inside its envelope; §18 names it as the second thing to give way | `LATER:` PostgreSQL per §13's server shape — the schema is small and the mechanism is unchanged |
| RISK-10 | Gateway rate limits bind at portfolio scale (growth) | Medium | Medium | [OPEN-02]'s numbers × §18's 100× arithmetic | The model sits only at the edges (intake, memo) — the daily scoring loop scales with CPU, not with quota | `LATER:` provisioned throughput or an in-bank hosted model for stage 6; stage 1 is human-paced and never binds |

**(judged 8, 18)**

---

## §27 Open questions and decisions

**Every open question** — who answers, what is stuck until they do, the default if no answer comes:

| ID | Question | Who answers | Stuck until answered | Default if no answer |
|---|---|---|---|---|
| [OPEN-01] | Does any granted/pending patent claim covenant-breach-horizon prediction? (Our web search found none; professional search of CPC G06Q 40/03 pending) | P5 post-event / organizer IP cell | §4's patentability judgment only | Claim combination-novelty only; assert no patent |
| [OPEN-02] | Gateway envelope: rate limits, quota, context window, latency, token accounting, price per call and per month | P4, at the T+4 gate | §18/§20's rupee cost math; latency budgets stay ours | Assume tight: small prompts, T7 ceiling, cost field logged but blank |
| [OPEN-03] | Does the gateway support structured output / JSON mode and streaming? | P4, T+4 | Nothing structural — own-code shape check ships regardless | Assume no; code checks stand |
| [OPEN-04] | Does `azure/genailab-maas-gpt-4o` answer on this key? | P4, T+4 | First rung of §22's endpoint-dies ladder | Single-model build; replay is the fallback |
| [OPEN-05] | Does the gateway call succeed with TLS verification left on? | P4, T+4 (first call is made verification-on) | §16's shipped posture line | Carry the handed sample's `verify=False`, stated plainly in §16 |
| [OPEN-06] | Is Python ≥3.10 present per-user on the laptop? | P1, T+4 | Everything — the single true blocker | Request via IT self-service at T+4 sharp; escalate to organizers within the hour |
| [OPEN-07] | Does the approved registry carry every package §13 pins? | P1, T+4 | §13's primary rows | Fall to §13's named fallbacks; empty-registry case escalates per RISK-05 |
| [OPEN-08] | Does the judging-room network reach the gateway? | P5, at T+20's first in-room run | Live vs replay posture for the two model beats | Replay with the visible label |
| [OPEN-09] | Does Edge headless run under this laptop's policy? | P3, T+4 | §15's automated capture | The named human fallback (P3, priced) |
| [OPEN-10] | What is the judging room's projector (resolution, real contrast)? | P5, T+20 | §15's contrast verification | Design to 1920×1080 and the 7:1 floor; verify the moment we plug in |
| [OPEN-11] | Will a person outside the team be available at T+13 for the unaided timing? | P5 asks organizers by T+6 | G4's label | G4 stays `ESTIMATED` and says so |
| [OPEN-12] | What properly calibrates T1–T5 (and stage 4's mapping)? In-build: the synthetic backtest by T+13. Properly: a real portfolio's history | P2 (in-build); a pilot bank (`LATER:`) | Defensibility of the starting values — they are labelled ours | Ship hour-3 values, tuned on EX-07–09, labelled |

**Decisions already made** — what was chosen, what else was considered, why, and what would change our mind:

- **Concept C1 over C2/C3 (§1):** chosen for auditability, originality against §2's field, and demo clarity. Mind-changer: if EX-13–15 shows extraction below the 0.90 floor even after verification, S1 narrows and C1 continues — C2 never becomes right because its probabilities would be model-made.
- **Code-decided forecast over an ML model (§3/§17):** transparency, 24-hour fit, and the Barboza-line trade (accuracy vs traceability) taken deliberately toward traceability. Mind-changer: real portfolio data and time — `LATER:` an ML stage 4 behind the same why-panel contract.
- **Flask + waitress over FastAPI + uvicorn (§13):** both are boring and proven; Flask's synchronous simplicity fits a single-user server-rendered app; FastAPI is the named fallback. Mind-changer: registry gap at T+4.
- **`azure_ai/genailab-maas-DeepSeek-V3-0324` as the one chat model (§13):** it is the alias the handed sample evidences; one model = one gate call = fewer blanks. Mind-changer: a failed gate ([OPEN-04] promotes the fallback).
- **Light paper theme, one theme only (§15):** projector physics and the refusal list. Mind-changer: none inside 24 hours; NG-09 holds.
- **SQLite over a server database (§13):** file-based, stdlib, zero-install under the laptop's constraints. Mind-changer: nothing in-build; §18's 100× line is where it changes.
- **Synthetic-only data (R-01):** mandated by `problem.md` and the only DPDP-clean option in a hackathon. Mind-changer: a sanctioned pilot with a bank's own historical data (§24's `LATER:` rollout).

---

## §28 Milestones, ownership and the clock

**People:** P1 data & engine (R-01, R-02, R-04) · P2 signals & forecast (R-05, R-06, R-07, threshold tuning) · P3 UI (R-08, screens, states, §15 capture) · P4 AI (R-03, R-10, N-01, N-04, gateway) · P5 delivery (R-09, R-11, N-02, N-03, N-05, gates, demo, cuts). Skills per [ASSUMED-01].

**The T+4 environment gate — on the judging laptop, which is the laptop we build on ([ASSUMED-06]), so there is no later hand-off surprise.** Prove, in one sitting, with results written into the repo: package install from the approved registry ([OPEN-07]) · Python version ([OPEN-06]) · app starts and serves a page · seed loads · AI access: one call to `azure_ai/genailab-maas-DeepSeek-V3-0324` made with TLS verification on ([OPEN-05]), then the fallback alias probe ([OPEN-04]) · latency and any visible quota/limit headers recorded ([OPEN-02], [OPEN-03]) · Edge headless capture probe ([OPEN-09]) · offline fallback: kill the network, confirm the app's local paths and the replay stub behave. If this laptop were *not* the judging machine, the gate would re-run on the real one the hour we get it — written here because [ASSUMED-06] could be wrong.

| Marker | Demonstrably true when it passes | Standout features working | Primarily on duty |
|---|---|---|---|
| T+0 | Challenge starts; repo, venv, this spec's hand-off prepared | — | All |
| T+3 | Spec handed over; roles agreed; [OPEN-11] request sent by T+6 | — | P5 |
| T+4 | The gate above passes; every red gate item has its fallback engaged *today* | — | P1, P4 |
| T+8 | Thinnest end-to-end path on real (seeded) input: seed → engine → one forecast → one screen → one live memo; replay cache first filled; N-01 harness runs with first scores | S2 thin | P1, P2, P4 |
| T+13 | **S2 end to end** (queue → case file → scrubber → why-panel); thresholds tuned on backtest ([OPEN-12]); outside-user timing run happens ([OPEN-11]); internal review of RISK-01 | S2 | P2, P3 |
| T+15 | **S1 end to end** (paste → propose → verify → confirm → live test), incl. EX-16/17 bad paths | S2, S1 | P4 |
| T+17 | **Feature freeze + design review.** **S3 end to end** (memo → grounding clicks → audit reconstruction → override). All three working; nothing new after this, only fixing. Screens reviewed against §15's sentence and forbids | S2, S1, S3 | P5 leads review |
| T+20 | Demo runs start to finish on the judging machine; §23's whole list executed; both arms' scores printed; projector checks ([OPEN-08], [OPEN-10]); demo book frozen | All three | P5, P3 |
| T+22 | Code freeze; `last-green` is the demo build; full rehearsal ×2 with timing; backup recording made; preflight rehearsed | All three | All awake |
| T+23:30 | Submission uploaded ([ASSUMED-05]): repo link/zip, spec, deck, recording — 30 minutes before the deadline, not at it | — | P5 |
| T+24 | Judged; §22's run | — | Presenters P3+P5 |

**Sleep — 5 hours per person (§11's number), scheduled:** P5 sleeps T+10.5–15.5 · P1 T+12–17 · P4 T+12–17 · P3 T+13.5–18.5 · P2 T+14–19. At least three people are awake at every moment; everyone is awake for T+20 through T+24; each person's block sits after their heaviest build window and before their heaviest verification window.

**When a marker goes red:** P5 decides within 30 minutes, and the decision is the next item on §8's cut list — not a debate. Two consecutive red markers → §5's narrower versions engage in §11's order (S1 narrows first, then S3, then S2). A red T+4 on [OPEN-06]/[OPEN-07] is the exception: escalate to the organizers immediately while engaging fallbacks, because hour 4 is the last cheap hour for that. Hours come from §11's arithmetic; no marker assumes work the reserve has not got.

**(judged 17)**

---

## §29 Change history

| Date | Version | What changed | Who |
|---|---|---|---|
| 2026-08-29 | 0.1 | Initial spec written at T+0 from `problem.md` + 41-search research pass; all sections §0–§31 | Spec lead (requirements & architecture) |

---

## §30 How this is judged

**Weighting:** `problem.md` gives none, so we ranked them ourselves ([ASSUMED-08]): **9 (originality), 12 (working prototype), 13 (ease of use), 14 (business value) decide the room; 1, 2, 4 are table stakes; the rest carry middle weight** — the pages went where the deciding happens.

| # | Criterion | Section | The line that answers it |
|---|---|---|---|
| 1 | Problem stated clearly, specifically, unambiguously | §1 | "Indian banks monitor loan covenants from borrower-submitted quarterly statements… deterioration between reporting cycles is caught late, and the later it is caught the less is recovered." |
| 2 | Background and context | §2 | The 11-row research table with rupee figures, sources and years, plus the law block. |
| 3 | Problem backed by industry research and evidence | §2 | Every row carries the number, the named source, the link and the year; 41 searches printed. |
| 4 | Boundaries and limits defined | §1 + §8 | The eight-item "what the problem is not" list, and NG-01…NG-09 each with a reason. |
| 5, 10 | Reasoning and logic behind the solution | §3 | "Because RBI's own data says behavioural stress precedes slippage… therefore… and here is what would prove us wrong" — labelled `ESTIMATED` with the head-to-head funded. |
| 6, 11 | Practical and achievable with actual resources | §11 (+§3 close) | "working hours: 60 · `must` hours: 36.5 · support hours: 12 · reserve: 11.5" — built on this laptop, this registry, these five people. |
| 7 | Addresses the core parts of the problem | §3 | The coverage table: every `problem.md` part → its answering mechanism stage → its requirement; no row empty. |
| 8 | Risks and assumptions named | §12 + §26 + §17 | Eight labelled assumptions with consequences; ten risks with early warnings; "what the model may never decide alone." |
| 9 | Originality, creativity, patentability | §4 | "The parts are known; the combination, the fails-closed verification, and the SMA/CRILC-native framing are ours" — measured against 20+ named products, papers and patents, with [OPEN-01] for the professional search. |
| 12 | Prototype shows key features working | §22 (+§5) | The 5:00 beat table: hook, mechanism running with the why-panel open, all three standout features live, real-vs-stubbed declared. |
| 13 | Ease of use, interface, experience | §15 | "Every screen is a credit case file being weighed…" — a chosen direction with forbids, one protected screen, and G4's *measured* unaided run. |
| 14 | Business benefit and value | §6 | G2: at the 67% haircut, one percentage point of recovery on one ₹50-crore exposure is ₹50 lakh — with labels that admit what is projected. |
| 15 | Growth to meet increased demand | §18 | Honestly mostly `LATER:` in 24 hours — the 10×/100× ladder names what gives way first (sync process, then SQLite, then gateway limits) and the server shape that answers each. |
| 16 | Right metrics for business impact | §6 | "Alerts raised… would all look excellent while the product failed" — G1/G3 paired so neither can be gamed. |
| 17 | Clear, realistic, achievable milestones | §28 | Nine markers, each with what is demonstrably true, which feature works, who is on duty, and the red-marker rule. |
| 18 | Scaling risks and what is done | §26 | RISK-08/09/10 — cost growth, the single-writer, the rate limit — each with an early sign and an answer. |
| 19 | Dependencies and critical path acknowledged | §12 | The dependency table with two items marked critical-path and "nothing else is on the critical path" said in words. |

**(judged: all)**

---

## §31 Checks

Run against the file as written, read back section by section; cited, not narrated; where a check covers many items, the thinnest is quoted.

1. **OK** — thinnest §2 row: row 9 (SMA-2 lead indicator) is a 2018 figure and says so in its Year cell ("2018 (dated; the mechanism, not the level, is what we use)"); every row carries source, link, year and a "what it changes" cell.
2. **OK** — thinnest trace: §22's before-beat claim "this is how ₹5-crore-plus accounts are watched today — quarterly, by hand" cites §2 row 5; chosen numbers (T1–T8, §18 targets, retention, versions) are exempt as the spec's own.
3. **OK** — every law, patent and paper is one the searches returned, each with its link: e.g., the least-known citation, "Predictive stress-testing model for covenant breach detection (IJSRCSEIT)", links to its article page in §4.
4. **OK** — §3's coverage table maps all ten `problem.md` capabilities plus the six controls to R-01…R-11/N-03; the business case built for is the single case `problem.md` lists (§1 Scorecard 1).
5. **OK** — thinnest bad-path: R-07.b covers the tie/boundary case rather than an outright failure (the queue has no external failure mode); every other `must` has a refusal/outage/malformed-input check (e.g., R-10.c gateway down), and every requirement carries a Why line naming the user's loss.
6. **OK** — every requirement names an existing flow (thinnest: R-13 → F-01, as the queue's pre-computation); F-01–F-04 all carry requirements; §7's roles each appear (RM F-03, Credit F-02, Risk F-01, Auditor/Operator F-04, gateway F-01/F-02, judges via §22 as §7 states); no chosen technology appears before §13 — §0's stack line says "copied from §13", and §12 names only "every package §13 pins".
7. **OK** — §3's table has a row per problem part and no row answers "out of scope"; none needed an NG.
8. **OK** — §3: "if… the full mechanism finds deteriorating borrowers no earlier or no more precisely than the naive distance-to-threshold rule… the idea fails"; §4 measures its claim against 20+ named and linked art items and takes [OPEN-01] where the search was shallow.
9. **OK** — thinnest §13 row: screenshot capture — primary Edge headless (version machine-resident, probed at T+4 under [OPEN-09], Chromium evidence linked) with the named human fallback (P3, priced 1 h in §11's support block) that check 29 accepts; every other row names version + evidence + no-admin install + fallback, with the web layer's fallback upgraded to FastAPI 0.141.1 + uvicorn 0.52.4 precisely because `http.server`'s own docs refuse production use.
10. **OK** — nothing requires Docker, VMs, admin, services or out-of-registry downloads: pip `--user`/venv installs, stdlib SQLite, Edge already present, system fonts; §13's server notes are marked `LATER:` and exempt.
11. **OK** — every §6 goal has a value, number, measurement source and first-measured date (thinnest: G2, honestly `PROJECTED` to a pilot); every §18 target has a number and a measured-how cell; a text search of the file finds fast/secure/scalable/reliable/robust standing bare nowhere outside quoted titles and section headings.
12. **OK** — every §14 interface row has a failure column entry (thinnest: seed files, whose failure is R-01.c's named error); the record list opens with the blanket ownership/retention/deletion sentence and per-record owners and sensitivity marks.
13. **OK** — §22 shows only R-02…R-11 and N-01, all existing `must` requirements, each beat marked Real or replay-labelled; the six questions cite §17, §19, §17, §22, §18/§20, §4 respectively, and each cited section says what the answer claims; the weakest claim (synthetic-only calibration of the probabilities) is the document's actual headline exposure, not a decorative one.
14. **OK** — §26 holds three idea risks (01–03), four delivery (04–07), three growth (08–10), each with an early-warning cell and an if-anyway cell; §12 marks two critical-path items and states "nothing else is on the critical path" with the reason.
15. **OK** — [OPEN-01…12] and [ASSUMED-01…08] are unique, sequential in order of first appearance (§2 → §11 → §12 → §13 → §15 → §17), each present where it arose and re-listed in §27 (opens) and §12 (assumptions).
16. **OK** — §11 prints "**working hours: 60** · **`must` hours: 36.5** · **support hours: 12** · **reserve: 11.5**"; 12 = 20% of 60; 11.5 = 60 − 36.5 − 12 ≥ max(2, 9); the sleep figure is the single number 5 and §28 schedules exactly 5 per person; §8's cut list carries T+13/T+15/T+17/T+19 against its four items.
17. **OK** — §28 schedules five named sleep blocks and the red-marker rule ("P5 decides within 30 minutes… the next item on §8's cut list"); §22 names all three: network drops → labelled replay; endpoint dies → alias probe then replay; reboot → cold start ≤2 min over the backup recording; §28 carries the [ASSUMED-05] submission at T+23:30 with the upload list, and no marker falls after it.
18. **OK** — thinnest `LATER:`: NG-07's ("digest emails and escalation pings per role") still names the real thing; §22 states "nothing marked `LATER:` appears working anywhere in this run", and §23 counts only built items.
19. **OK** — §17 names what the model may never decide alone, the grounding rule ("any figure… the product presents as true comes from the record or the calculation, never from the model"), the fail path (retry once, refuse, never shown), code-vs-prompt marking per guardrail (memo tone is the one named prompt-side item), the why-panel contract for rule, prompted and trained deciders each naming the threshold and the side the value fell, and the OverrideRecord (§14) with retraining as NG-05/§17's `LATER:`.
20. **OK** — §20's AI log lists prompt version, model version string, tokens in/out, latency, computed cost field and check verdict per call, separate from the app log; all six watched numbers carry alert values and a named waker; overrides, corrections, refusals and failed checks are first-class entries.
21. **OK** — §15 has hex+reasons, three named faces with reasons and the Microsoft font-list link, the type/spacing/shape scales, motion values with the reduced-motion answer, system fonts (no CDN), the standing refusal list plus the three 2026 tells with where seen plus the three direction-forbids, the signature scrubber with what an ordinary tab strip could not do and its placement inside §22's hook on the protected screen, the explicit no-3D line, four state families per screen inside R-08.b, and the timing run (who: P5 watches an outside user; when: T+13; priced; [OPEN-11] with the `ESTIMATED` fallback label).
22. **OK** — §5: three different capabilities (digitise / forecast / defend), each with requirements, hours, working-by hour and a narrower version; none is a chart/tile/filter/view; S2 is carried by mechanism stages 2–5; none appears in §8's cut list; beats 4, 3 and 5 of §22 are theirs; working-by hours T+13/T+15/T+17 are distinct with only S3 on T+17.
23. **OK** — §0 opens with two paragraphs of continuous prose, 187 words total, no headings/bullets/IDs/anchors, naming that code computes ratios, tests covenants, scores signals and projects headroom while the model reads clause text and writes the memo; everything in §0 restates later sections; IDs run in order with none repeated or skipped (R-01–13, N-01–05, F-01–04, NG-01–09, RISK-01–10, OPEN-01–12, ASSUMED-01–08); §29 carries the 2026-08-29 v0.1 row.
24. **OK** — §1 prints both scorecards (one business case; three concepts differing in mechanism, stages phrased ≤8 words with code marks) with struck-out items and rules ("no concept was struck by the depth floor — the floor's real work was shaping C1") and the research-changed-the-ranking line; §3's AI-vs-rules argument is labelled `ESTIMATED` with the baseline arm funded; §17 has 20 examples and pass marks; §28's T+4 gate proves install/start/data/AI/latency/quota/offline; §22 has the 10-second hook, the 90-second before/after (0:10–1:40), beats summing to 5:00 inside the 8:00 slot, reset, preflight and backup recording.
25. **OK** — every outcome carries one of the four labels (G1–G5, §3's reasoning, §17's marks, §18's cost line); no estimate or projection is written as achieved — §17 states "no score is written here" and §23 is where `MEASURED` happens.
26. **OK** — `https://genailab.tcs.in` appears verbatim in §13 and §14; the only aliases are `azure_ai/genailab-maas-DeepSeek-V3-0324` and `azure/genailab-maas-gpt-4o`, both character-for-character from the organizer list; no client library is named before §13; §16 states the `verify=False` inheritance, the verification-on gate test, and the environment-only key; rate limits, quota, context window, structured output, streaming, latency, token counts and price are nowhere stated as known — they live in [OPEN-02]/[OPEN-03] — while §18's response targets and §17's speed mark and T6/T7 ceilings are set as targets.
27. **OK** — §3 states six ordered stages; stages 2–5 are decided by code and §3's table marks each decider; every stage is carried by named requirements (§10's opening map) and §17's why-panel opens each ("every stage of §3's mechanism is openable"); no stage's content is "the model is asked to do it" — even stage 6 is model-drafted but code-checked against records.
28. **OK** — §2 prints "41 distinct (25 field, 16 stack-and-look)" and the three mind-changing findings; the single business case was researched before Step 3 chose concepts; every §13 version cites a Step-4 search result link, including the late FastAPI/uvicorn and stdlib/font sources fetched to close exactly this gap; web access worked, so no failure notice was owed in chat.
29. **OK** — §15 chose its direction first: three directions scored, each borrowing from a product actually opened with its link (9fin, Koyfin, Linear), two struck with reasons (list-flattening genericness; projector physics + the dark-default tell); the winner is one holdable sentence; under it sit three quoted forbids a reviewer can hold a screen against, none a restatement of the generic list; composition rules give the organising principle, density, alignment, deliberate emptiness and the accent's allowed homes; the case file is named as carrying §22's opening beat, priced via R-08 in §11, absent from §8's cut list; the contrast floor is 7:1 verified on the projector ([OPEN-10]); the judged theme is light paper; capture has its §13 row and its named human fallback with an hour against it.
30. **OK** — the six: **observability** → N-02 (1 h in §11); **thresholds** → N-03 (0.5 h); **data security & privacy** → N-04 (1 h); **authentication & authorization** → NG-03 in §8 with its reason (one analyst, one laptop, loopback) and its `LATER:` (SSO + enforced role matrix); **feedback loop** → R-11 (1.5 h); **explainability** → R-09 (3 h). Every technology the six need is already a §13 row passing check 9 (Flask/SQLite/stdlib for logs and stores; no session or token library exists to need a row, which is NG-03's point).
31. **OK** — §17's set is carried by N-01 with 3 hours in §11; the examples live in `examples/*.json` and P4/P5 may add (each addition an AuditEvent); the baseline arm (naive headroom rule, regex parser, naive prompt) runs on the same 20 examples against the same marks; §23 item 2 runs both arms at T+20 and prints both; no score anywhere in this file is written as already measured.

*Honest residuals worth naming even under OK verdicts:* check 5's thinnest bad-path (R-07.b) is a boundary case, not an outage — the queue's real outage modes live upstream in R-05/R-06 and are checked there; check 9's Edge row carries no pinned version by design (the browser is machine-resident and policy-gated, hence [OPEN-09]); and check 2 rests on §2 row 8's false-positive figures being worldwide stand-ins, which row 8 itself declares.

*(End of spec.md — version 0.1, 2026-08-29.)*
