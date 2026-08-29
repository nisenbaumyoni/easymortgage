# Product Requirements Document — Mortgage Improvement Platform

| | |
|---|---|
| **Working title** | EasyMortgage |
| **Document status** | Draft v0.1 |
| **Last updated** | 2026-08-29 |
| **Owner** | Product |
| **Related docs** | `../architecture/`, `../technicalsolution/` |

---

## 1. Summary

EasyMortgage helps homeowners and prospective buyers **get a better mortgage**. It
serves two journeys:

1. **Existing mortgage** — the user uploads their mortgage documents. The platform
   extracts the key terms, enriches them with authoritative country data (benchmark
   rates, tax rules, regulator guidance), and uses AI to produce a prioritized,
   plain-language action plan for improving the mortgage (e.g. refinance, product
   switch, overpay, change term, renegotiate).
2. **New mortgage** — the user answers a guided questionnaire. The platform assesses
   affordability and eligibility, then shops comparable products across mortgage
   providers for that country and returns a ranked shortlist with next steps.

The product is **country-agnostic at its core**. Every country-specific rule,
data source, document type, lender catalog, and regulatory constraint lives in a
**Country Plugin**. The platform is **multilingual** and automatically detects the
user's language and locale, while letting them override it.

---

## 2. Problem statement

- Mortgages are the largest financial commitment most households have, yet the
  terms are opaque, the paperwork is dense, and the "is this still a good deal?"
  question is hard to answer without an advisor.
- Switching or renegotiating is under-utilized because the process feels risky and
  the savings are not obvious until someone does the math.
- New borrowers struggle to compare products on a like-for-like basis (rate is
  only part of the true cost — fees, term, flexibility, and incentives matter).
- Solutions that exist are **single-country** and don't generalize; the rules,
  documents, benchmarks, tax treatment, and even the vocabulary differ per country.

---

## 3. Goals and non-goals

### 3.1 Goals

- G1 — Give an existing borrower a **clear, ranked list of concrete improvements**
  with an estimated financial impact and the steps to act on each.
- G2 — Give a new borrower a **ranked shortlist of real, currently-available
  products** matched to their situation, with the true lifetime cost of each.
- G3 — Make adding a **new country** a well-defined plugin task, not a rewrite.
- G4 — Work in the **user's language and locale** out of the box.
- G5 — Be **transparent and compliant**: always show assumptions, data sources,
  timestamps, and the correct regulatory disclaimer for the country.

### 3.2 Non-goals (for now)

- NG1 — Being a licensed mortgage broker or originating loans directly. The MVP
  produces **information and guidance**, and hands off to lenders/brokers.
- NG2 — Real-time binding quotes. Where a country plugin cannot get live
  underwriting decisions, results are clearly labeled "indicative".
- NG3 — Non-mortgage lending (personal loans, credit cards, auto).
- NG4 — Investment / portfolio advice beyond the mortgage itself.

---

## 4. Target users and personas

| Persona | Journey | Needs |
|---|---|---|
| **Homeowner "Dana"** — has a mortgage, suspects it's no longer competitive | Existing | Fast read on whether to act, how much could be saved, what the risks are |
| **Remortgage-ready "Omar"** — fixed period ending in <6 months | Existing | Timeline-aware plan, product-switch vs. remortgage comparison |
| **First-time buyer "Lena"** — has an offer accepted, needs a mortgage | New | Affordability check, shortlist of products she can actually get, document checklist |
| **Mover "Sam"** — porting or taking a new mortgage on a new home | New | Comparison including exit fees on the current deal |
| **Advisor / partner (later)** | Both | White-label or referral flow, audit trail |

---

## 5. Key concepts and terminology

| Term | Meaning |
|---|---|
| **Country Plugin** | A self-contained module implementing the country contract: locales, documents, data sources, lender catalog, product model, eligibility & tax rules, improvement strategies, cost model, disclaimers. |
| **Mortgage Profile** | Normalized representation of a user's current mortgage (balance, rate, rate type, remaining term, repayment type, product end date, fees, LTV, property value, borrower income). Country-independent schema, country-specific extensions. |
| **Improvement Strategy** | A candidate action (refinance, product transfer, overpay, extend/shorten term, offset, rate lock, renegotiate) with an impact model. |
| **Product Offer** | A mortgage product from a provider: rate, rate type, initial period, fees, incentives, constraints, estimated total cost over a comparison window. |
| **Locale** | Language + region (e.g. `pt-BR`, `he-IL`, `en-GB`) driving translation, formatting, and which Country Plugin loads. |
| **Data Source Connector** | Adapter inside a plugin that fetches authoritative country data (benchmark/policy rate, average market rates, tax authority rules, regulator guidance, government comparison portals). |

---

## 6. Use case 1 — Existing mortgage

### 6.1 Narrative

> Dana signs in, the app is already in her language. She picks "I have a mortgage".
> She uploads her mortgage offer letter and latest annual statement. The app shows
> what it understood — balance, rate, type, term, product end date, fees — and asks
> her to confirm or correct a few fields. It pulls the current benchmark rate and
> average market rates for her country and product type, checks tax treatment, and
> within a minute shows: "You could save an estimated X over the next N years.
> Here are 3 things to consider, ranked." Each item expands into what it is, why it
> applies to her, the estimated impact, the costs and risks, and the steps to do it.

### 6.2 Functional requirements

- **UC1-F1 Document upload** — accept PDF, image (JPG/PNG/HEIC), and common
  scanned formats; multi-file; drag-and-drop and mobile camera capture. Max size
  and page count configurable per plugin.
- **UC1-F2 Document classification** — identify document type against the plugin's
  document taxonomy (e.g. offer letter, annual statement, amortization schedule,
  KFI/ESIS/Loan Estimate, deed, insurance). Unknown documents are flagged, not
  rejected.
- **UC1-F3 Field extraction** — OCR + structured extraction of the fields needed to
  build the Mortgage Profile. Every extracted field carries a **confidence score**
  and a **source reference** (file + page + snippet).
- **UC1-F4 Human-in-the-loop confirmation** — the user reviews extracted fields;
  low-confidence fields are highlighted and required. No analysis runs on
  unconfirmed critical fields.
- **UC1-F5 Country data enrichment** — the active plugin's Data Source Connectors
  fetch: current policy/benchmark rate, representative market rates for the user's
  product type and LTV band, applicable tax rules/incentives, and regulator
  guidance links. Each value is stored with a retrieval timestamp and source URL.
- **UC1-F6 AI analysis** — given the confirmed profile + enrichment + the plugin's
  improvement-strategy rules, produce a ranked set of Improvement Strategies. For
  each: estimated monthly and lifetime impact, upfront costs, break-even point,
  risks, eligibility caveats, and confidence.
- **UC1-F7 Action plan output** — plain-language, translated, printable/exportable
  (PDF). Ordered by net benefit. Includes a "do nothing" baseline for comparison.
- **UC1-F8 Step-by-step instructions** — for the chosen strategy, generate a
  checklist tailored to the country process (who to contact, what to prepare,
  typical timeline, key dates such as the product end date / notice window).
- **UC1-F9 Recheck / monitoring (later phase)** — allow the user to opt in to be
  notified when market conditions or their product end date make it worth
  re-running the analysis.

### 6.3 Analysis inputs → outputs

| Input | Source |
|---|---|
| Current balance, rate, rate type, remaining term, repayment type, product end date, fees, early-repayment terms | Extracted from documents, user-confirmed |
| Property value, LTV, borrower income/affordability signals | Documents + questionnaire |
| Benchmark/policy rate, market rates by product & LTV | Plugin Data Source Connectors |
| Tax treatment (e.g. interest deductibility, incentives) | Plugin tax rules + tax-authority source |
| Country process, fees, timelines, regulator posture | Plugin config |

| Output | Description |
|---|---|
| Savings estimate vs. baseline | Range, with assumptions listed |
| Ranked Improvement Strategies | Impact, cost, break-even, risk, eligibility, confidence |
| Action checklist | Country-specific steps and key dates |
| Sources & assumptions appendix | Every number traceable to a source + timestamp |
| Disclaimer | Country-correct regulatory wording |

---

## 7. Use case 2 — New mortgage

### 7.1 Narrative

> Lena picks "I need a mortgage". A guided questionnaire asks about the property,
> price, deposit, income, employment, existing debts, and preferences (rate
> certainty vs. lowest cost, expected time in the home, flexibility to overpay).
> The app runs an affordability and eligibility pass using the plugin's rules,
> then queries the country's mortgage providers for matching products. She gets a
> ranked shortlist with the true cost over her expected horizon, what she'd likely
> qualify for, a document checklist, and a referral/hand-off to each provider or a
> broker.

### 7.2 Functional requirements

- **UC2-F1 Guided intake** — dynamic questionnaire driven by the plugin (fields,
  order, validation, help text all localized). Save & resume.
- **UC2-F2 Affordability & eligibility** — apply the plugin's affordability model
  (income multiples / DTI / stress tests as relevant to the country) to produce a
  borrowing range and a qualification likelihood.
- **UC2-F3 Product sourcing** — query the plugin's Lender Catalog / provider
  adapters for products matching amount, LTV, term, property type, and borrower
  profile. Support both **live API** providers and **rate-table** providers
  (clearly labeled indicative).
- **UC2-F4 Like-for-like comparison** — normalize every Product Offer to a **total
  cost over a user-chosen comparison window** (default = expected time in home),
  including fees, incentives (cashback, free valuation), and the reversion rate
  after any initial period.
- **UC2-F5 Ranked shortlist** — ordered by total cost (default) with alternative
  sorts (lowest monthly, most flexible, fastest to complete). Each entry shows why
  it ranked where it did.
- **UC2-F6 Document checklist & readiness** — what the user needs to gather, based
  on the plugin's document taxonomy and the selected providers.
- **UC2-F7 Hand-off** — deep link / referral to the provider or a partner broker,
  with the user's consent, carrying over the collected data where the integration
  allows.
- **UC2-F8 AI assist** — explain trade-offs in plain language, answer "what if"
  questions (bigger deposit, longer term, fixed vs. variable), and surface risks
  the user didn't ask about.

---

## 8. Country Plugin architecture (product requirements)

The core platform must not contain any country-specific logic. A Country Plugin is
the unit of extension and must be independently developed, tested, versioned, and
deployed.

### 8.1 What a plugin provides (the country contract)

| Capability | Description |
|---|---|
| **Metadata** | Country code, supported locales, currency, default comparison window, activation status |
| **Regulatory profile** | Whether output is "information", "guidance", or "advice"; required disclaimers; data-handling constraints; whether hand-off must go through a licensed entity |
| **Document taxonomy** | List of recognized document types, expected formats, and the fields each yields |
| **Document parsers / extractors** | Rules/models to classify documents and extract fields, with confidence |
| **Data Source Connectors** | Adapters for benchmark/policy rate, market rate references, tax-authority rules, regulator guidance, government comparison portals |
| **Mortgage product model** | Country product types (e.g. fixed, variable, tracker, offset, mixed), initial periods, reversion behavior, early-repayment rules |
| **Lender catalog + provider adapters** | Providers operating in the country and how to query them (API or rate table) |
| **Affordability & eligibility rules** | Income multiples / DTI / stress tests / LTV limits as applicable |
| **Tax rules** | Interest deductibility, buyer incentives, transfer taxes, subsidies |
| **Improvement strategy rules** | Which strategies are legal/available, their cost models, and their constraints |
| **Cost model** | Valuation, legal, arrangement, exit/early-repayment, government fees |
| **Localization pack** | Translations, formatting rules, terminology glossary, help content |
| **Process & timelines** | Steps, actors, typical durations, notice periods, key-date logic |

### 8.2 Platform responsibilities (country-independent)

- Locale detection and plugin resolution.
- Identity, consent, session, and secure document storage.
- Orchestration of the two journeys (upload → extract → confirm → enrich → analyze → present).
- The AI analysis engine and prompt/rule orchestration, parameterized by plugin data.
- Normalized schemas (Mortgage Profile, Improvement Strategy, Product Offer).
- Comparison math and total-cost normalization.
- Presentation, export, notifications, and audit trail.
- Plugin lifecycle: registration, versioning, capability discovery, health checks,
  graceful degradation when a connector is unavailable.

### 8.3 Plugin quality bar (Definition of Done for a new country)

- All contract capabilities implemented or explicitly marked "not supported".
- Regulatory profile reviewed and signed off by legal/compliance.
- At least one working provider integration (API or rate table).
- Document extraction validated against a labeled sample set (target accuracy per
  critical field defined per country, e.g. ≥95% on balance/rate/term).
- Localization complete for at least one locale; no untranslated strings in the
  journey.
- End-to-end test passing for both journeys with representative fixtures.
- Data sources documented with owner, refresh cadence, and licensing/ToS clearance.

### 8.4 Degradation rules

- If a Data Source Connector fails, use the last cached value and show its age; if
  none, mark the dependent output "unavailable" rather than guessing.
- If no provider integration returns results, still show the affordability range
  and a generic checklist.
- A plugin can be toggled to "read-only / analysis-only" if provider or hand-off
  integrations are down.

---

## 9. Internationalization and localization

- **I18N-1 Language detection** — detect language from, in priority order: explicit
  user setting → account preference → `Accept-Language` header → IP-based country
  hint → fallback to a configured default. The detected choice is always shown and
  editable.
- **I18N-2 Locale vs. country** — language and country are separate. A user may
  read in English while their mortgage is in another country. The **Country Plugin
  is chosen by the mortgage's country**, not the UI language.
- **I18N-3 Full translation** — all UI, generated action plans, checklists, emails,
  and PDF exports are localized. AI-generated text is produced in the user's
  language (not translated after the fact) using the plugin's terminology glossary.
- **I18N-4 Formatting** — numbers, currency, dates, percentages, and address
  formats follow the locale. Currency of the mortgage is always shown explicitly.
- **I18N-5 RTL support** — layout, components, and PDF output support right-to-left
  scripts.
- **I18N-6 Content fallback** — if a string is missing in the target locale, fall
  back to the plugin default locale, then platform default, and log the gap.
- **I18N-7 Translation operations** — externalized string catalogs, pseudo-locale
  for testing, and a process for professional review of legally sensitive strings
  (disclaimers, risk warnings) — these are never machine-translated without review.

---

## 10. AI and analysis requirements

- **AI-1 Grounding** — the model works from the confirmed Mortgage Profile and the
  plugin's fetched data and rules. It must not invent rates, fees, tax rules, or
  legal steps. Missing data is stated as missing.
- **AI-2 Explainability** — every recommendation lists the inputs it used, the
  assumptions, and the calculation basis. Numbers come from the deterministic
  comparison engine, not from the model's free generation.
- **AI-3 Ranking** — strategies/offers are ranked by a transparent scoring function
  (net financial benefit over the comparison window, adjusted for risk and
  eligibility confidence). The model explains the ranking; it doesn't set it alone.
- **AI-4 Guardrails** — no definitive legal/tax assertions beyond the plugin's
  vetted content; always attach the country disclaimer; flag when a licensed
  professional is required.
- **AI-5 Confidence & uncertainty** — outputs carry confidence; low-confidence
  results are labeled and de-emphasized.
- **AI-6 Human review hooks** — support an optional expert-review step before the
  plan is shown, per plugin/regulatory profile.
- **AI-7 Evaluation** — a regression suite of scenario fixtures with expected
  outcomes; track recommendation quality, extraction accuracy, and hallucination
  rate per release and per plugin.
- **AI-8 Privacy** — documents and personal data are not used to train external
  models; any model provider terms must permit zero-retention / no-training use.

---

## 11. Data, documents, and privacy

- **D-1 Sensitivity** — uploaded documents contain financial and identity data.
  Treat all of it as highly sensitive by default.
- **D-2 Encryption** — encryption in transit and at rest; per-user or per-document
  key isolation where feasible.
- **D-3 Retention & deletion** — user-configurable retention; one-click delete of
  all documents and derived data; automatic purge after a configurable period of
  inactivity.
- **D-4 Data residency** — support storing a country's user data in-region where a
  plugin's regulatory profile requires it.
- **D-5 Minimization** — only extract and keep the fields the journeys need.
- **D-6 Consent** — explicit, granular, revocable consent for: document analysis,
  querying external data sources, contacting providers, and any data sharing on
  hand-off.
- **D-7 Access & portability** — user can view everything held about them and
  export it.
- **D-8 Auditability** — immutable audit log of what data was fetched, from where,
  when, and what was shown to the user (needed for disputes and compliance).
- **D-9 Compliance baseline** — GDPR-equivalent handling everywhere; per-country
  additions (e.g. financial-data rules) come from the plugin.

---

## 12. Integrations

| Integration | Journey | Notes |
|---|---|---|
| Document OCR / extraction | Existing | Pluggable; on-prem option for data-residency countries |
| Benchmark / policy rate sources | Both | Central bank or equivalent, per plugin |
| Market rate references | Both | Regulator data, aggregators, or provider tables |
| Tax authority rules | Both | For deductibility/incentives; versioned |
| Government comparison portals | Both | Where they exist (link out + import where allowed) |
| Mortgage provider APIs | Both | Live eligibility/products where available |
| Provider rate tables | Both | Fallback; labeled indicative; refresh cadence tracked |
| Broker / partner hand-off | Both | Consent-gated; passes collected data where permitted |
| Identity / KYC (later) | New | Only if the product moves toward origination |
| Notifications (email/push) | Both | Localized templates |

All external calls must be resilient (timeout, retry, cache, circuit-break) and
must record source + timestamp for anything shown to the user.

---

## 13. Non-functional requirements

- **NFR-1 Security** — OWASP ASVS-aligned; pen-tested before each new country
  launch; secrets management; least-privilege access to document stores.
- **NFR-2 Performance** — existing-mortgage analysis returns a first result within
  ~60s of document confirmation for a typical 2–4 document set; new-mortgage
  shortlist within ~10s of questionnaire completion (live-API dependent).
- **NFR-3 Availability** — 99.9% for the core journeys; per-connector degradation
  must not take down the journey.
- **NFR-4 Scalability** — horizontal scaling of extraction and analysis workloads;
  per-country isolation so one plugin's load or outage doesn't affect others.
- **NFR-5 Observability** — structured logs, tracing across the orchestration
  pipeline, per-plugin dashboards (extraction accuracy, connector health,
  conversion).
- **NFR-6 Accessibility** — WCAG 2.2 AA for all user-facing surfaces, including PDF
  exports.
- **NFR-7 Portability of plugins** — a plugin is packaged and versioned
  independently; the platform supports running multiple plugin versions
  concurrently during migrations.
- **NFR-8 Testability** — every plugin ships fixtures; the platform provides a
  plugin conformance test kit.

---

## 14. Trust, compliance, and regulatory posture

- Mortgage advice is regulated in most jurisdictions. The **plugin's regulatory
  profile** declares whether the product may present "information", "guidance", or
  "regulated advice" in that country, and what entity must stand behind a hand-off.
- The MVP posture is **information + guidance with hand-off**, not regulated advice.
- Every output screen and export carries the country-correct disclaimer, the data
  sources with timestamps, and the assumptions used.
- Marketing and in-product copy must not imply guaranteed savings or approval.
- A compliance sign-off gate is part of every plugin's Definition of Done and every
  country launch.

---

## 15. Success metrics

| Metric | Why |
|---|---|
| **Activation** — % of users who complete confirmation (existing) or questionnaire (new) | Funnel health |
| **Analysis completion** — % who receive an action plan / shortlist | Core value delivered |
| **Recommendation acceptance** — % who mark a strategy/offer as "I'll do this" | Perceived usefulness |
| **Verified outcome (later)** — % who confirm they switched/renegotiated and the realized saving | True impact |
| **Extraction accuracy** per critical field, per plugin | Quality & trust |
| **Time to first result** (both journeys) | Experience |
| **Connector uptime / staleness** per plugin | Data trust |
| **Hand-off conversion** to providers/brokers | Business model |
| **Locale coverage** — % of sessions fully localized (no fallback strings) | I18N quality |
| **New-country lead time** — calendar time to ship a plugin to DoD | Scalability of the model |

---

## 16. Scope and phasing

### Phase 0 — Foundations
- Core orchestration, normalized schemas, plugin runtime & conformance kit.
- I18N framework, language/locale detection, RTL.
- Secure document store, consent, audit log.
- AI analysis engine with grounding, explainability, and eval harness.

### Phase 1 — MVP (1 launch country)
- One reference Country Plugin, both journeys end-to-end.
- Existing: upload → extract → confirm → enrich → ranked action plan + checklist + PDF.
- New: questionnaire → affordability → product shortlist (at least one live or
  rate-table provider) → document checklist → hand-off.
- One primary locale + English.

### Phase 2 — Second country + hardening
- Add a structurally different country (different product types, tax rules,
  regulator posture) to prove the plugin boundary.
- Add locales; professional review workflow for legal strings.
- Monitoring/recheck opt-in for existing mortgages.

### Phase 3 — Scale
- Plugin SDK + docs so new countries can be built with less core involvement.
- More provider integrations; broker marketplace.
- Optional expert-review tier.

---

## 17. Risks and open questions

| # | Risk / question | Notes |
|---|---|---|
| R1 | Regulatory classification varies and can force "advice" rules | Resolve per country in the regulatory profile before launch; legal in the loop early |
| R2 | Provider data access — many lenders have no API and restrictive ToS | Rate-table fallback; partnerships; label indicative clearly |
| R3 | Document variety is huge; extraction accuracy on messy scans | Human-in-the-loop confirmation is mandatory; per-field confidence; labeled sample sets |
| R4 | Authoritative data source licensing / ToS for reuse | Clear each source during plugin DoD; cache policy must respect terms |
| R5 | AI hallucination on legal/tax specifics | Grounding + vetted plugin content + disclaimers + eval suite |
| R6 | Data residency and cross-border storage | In-region storage option driven by plugin |
| R7 | Savings estimates that don't materialize → trust damage | Always show ranges, assumptions, and break-even; "verified outcome" tracking |
| Q1 | Do we monetize via hand-off referral, subscription, or both? | Affects hand-off design and disclosure |
| Q2 | Which launch country, and who owns its compliance sign-off? | Blocks Phase 1 |
| Q3 | Build vs. buy for OCR/extraction, and can it run in-region? | Affects Phase 0 |
| Q4 | How much expert human review is required in the loop at launch? | Cost and latency trade-off |

---

## 18. Appendix — illustrative country differences

These illustrate *why* the plugin boundary exists; they are not implementation specs.

| Dimension | Example variation across countries |
|---|---|
| Product types | Fixed-for-life; short fixed + reversion to variable; tracker linked to policy rate; offset; mixed |
| Rate reset / switching | Free product transfers vs. full remortgage with legals; notice windows; early-repayment charge structures |
| Affordability test | Income multiple caps; DTI limits; regulator stress-rate add-ons |
| Tax | Mortgage interest fully/partly deductible or not; first-time-buyer relief; property transfer taxes |
| Key documents | Offer letter + annual statement; ESIS/KFI; Loan Estimate/Closing Disclosure; notarized deed; amortization table |
| Authoritative data | Central bank policy rate; regulator average-rate publications; government comparison portal; tax authority guidance |
| Process actors | Bank direct; mortgage broker; notary; government registry |
| Language / script | Single vs. multiple official languages; LTR vs. RTL |

---
