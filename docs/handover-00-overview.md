# Rocken Talent — Developer Handover · 00 · Overview & Global Rules

> **This is the foundation doc.** It holds the rules that are true across **every** screen (architecture, auth, matching, notifications, privacy, NFRs). Each per-page spec (Dashboard, Jobs, …) assumes these and only documents what is *page-specific*.
>
> **How to read the spec set** — **Markdown = logic / rules / flows** (the "why & when"); **Figma Dev Mode = visuals** (measurements, tokens, assets). Annotated screenshots use numbered badges ①②③ that map 1:1 to the tables.
>
> ⚠ **FUTURE** = designed but **not part of the MVP** (greyed out in the product). 🔧 **BUILD** = required but **not yet in the prototype**. ❓ **TBD** = open, to confirm with the CRM/backend team.
>
> **Every page spec ends with a "States & edge cases" section** documenting each `if`-case (empty / loading / no-match / guest / blocked …). Reproduce any state live via the **Handover-States** toolbar (bottom-right of the prototype) or a **`?demo=key:val,…`** deep-link. The toggles: `status` · `profilePct` · `processes` · `termine` · `recommended` · `notifs` · `authed` (guest) · `styling` · `cookie`.

---

## 1. Product context & scope
- **Rocken Talent (RT)** is the candidate-facing web app. The **product is real** (will be built); what exists today is a **clickable prototype/demo**.
- **Target platform:** responsive **web only** — no native apps. Desktop **and** a dedicated mobile view.
- **Purpose of this handover:** a **developer** handover for Rocken IT — technical & decoupled from visual design. Not a customer-facing showcase.
- **MVP scope** is still being finalized; ⚠ FUTURE items below are explicitly **out**.

## 2. System architecture & data ownership
The **CRM is Rocken's own system** and is the source of truth for candidates, vacancies, consultants, processes and appointments. RT is a **frontend that integrates via API**.

| Concern | Owner | Note |
|---|---|---|
| Candidate / vacancy / consultant / process data | **CRM** | Rocken IT **builds the API** (it does not exist yet) — this spec defines the data RT *needs*. |
| Recommendations (job-id, match score, `why` text) | **Matching engine** | Separate service/API. |
| Rendering, client state, UX flows | **RT frontend** | This project. |
| Profile edits | **RT → CRM (write-back)** | RT needs **read *and* write** endpoints (see Profile spec). |

## 3. Accounts, auth & activation lifecycle
- **No own auth in RT.** Account creation + login happen **through the CRM**; RT consumes the CRM session/auth.
- A candidate gets an account when they are **activated in the CRM**.
- **Activation lifecycle:** the candidate can **request activation/reactivation** in the app, but **approval always happens in the CRM** (consultant/admin). While inactive → applying & matching are blocked (reactivation CTA shown).
- **Search-status** (e.g. *actively looking / open to offers / in activation / not looking*) — the prototype uses 4 values as a **working assumption**; ❓ the real taxonomy and who controls it (candidate vs. CRM) is **TBD**.
- **Onboarding:** a layover mask exists in Figma but is **not built yet** → 🔧 BUILD.

## 4. Matching & recommendation rules (global)
Three distinct **proposal channels** — keep them separate in data & UI:

| # | Channel | Where it surfaces | Match score shown? |
|---|---|---|---|
| 1 | **Consultant proposal** (consultant hand-picks & pushes) | Home → "Warten auf dich" | **No** |
| 2 | **Self-search** (candidate searches & applies) | Jobs → "Alle Vakanzen" | **No** |
| 3 | **Recommender** (engine suggests) | Home → "Für dich gematcht" · Jobs → "Für dich" | **Yes, if ≥ 70 %** |

- **Match score (%)** is rendered **only on recommender matches** and **only when ≥ 70 %**. Never on consultant proposals or self-search.
- **Two proposal texts:** `why` (why-you-match) = **LLM/engine-generated**; `note` (quote incl. the consultant's name) = **written by the consultant** (CRM field).
- **No recommendations** when: profile < 70 % complete **OR** account not activated **OR** the engine finds no matching internal vacancy.

## 5. Interest → application flow (global)
1. Candidate clicks **"Interesse melden"** (single, unified CTA) on a vacancy → interest goes **to the responsible consultant**, *not* directly to the company.
2. Dossier is sent to the company **only after the candidate allows submission** ("Einreichen erlauben", consent). **The consultant submits**, not the candidate.
3. **Pipeline stages** are advanced by **consultants in the CRM** (process status); "who's at bat" is derived from CRM status.

## 6. Consultant model
- Every process has a **responsible consultant = whoever created the process in the CRM**. For a **self-search / self-application** it is the **consultant responsible for the vacancy**. (These can differ.)
- **Chat** messages go **into the CRM**.
- Each consultant has an **"About me"** maintained in their **CRM user profile**; there is no separate company text.

## 7. Notifications
- **Channels:** **E-mail** (triggered via CRM/Marketing) **+ an in-app notification center** (bell, read/unread) — 🔧 the in-app center is **not in the prototype yet**.
- **Triggers (only these):**

| Trigger | Notify? |
|---|---|
| New consultant proposal | ✅ |
| Application status changed | ✅ |
| Interview scheduled / changed | ✅ |
| Account activated / reactivated | ✅ (🔧 new) |
| New chat message from consultant | ❌ |
| New recommender matches | ❌ |

## 8. Global display & privacy rules
- **Guest masking:** logged-out visitors **do not see company data** — company name/logo/exact location are hidden (region only). The **"confidential" flag is defined in the CRM**.
- **Company logos** are stored **in the CRM**.
- **Score rules** as in §4 (≥ 70 %, recommender only).
- **Cookie consent** overlays on first visit until dismissed (global, all screens). Full-width bottom bar; choices: *Einstellungen · Nur notwendige · Alle akzeptieren*.
- **No timeframe / deadline promises (content rule).** RT must **never** state a guaranteed response time or a deadline for the application process — no *"Rückmeldung bis …"*, no *"Antwort bis …"*, no *"meldet sich innerhalb von 24 h"*. Rocken cannot guarantee that something happens within a fixed window. Use status text **without** time commitments (e.g. *"meldet sich bei dir"*, *"Wir prüfen deine Vorschläge — du musst nichts tun"*). Real, scheduled **appointment dates** (e.g. an interview on 18 Jun) are fine — those are events, not process guarantees.

## 9. Non-functional requirements (NFRs)
| Area | MVP requirement |
|---|---|
| **Language** | **German only.** No i18n required for MVP, but keep strings extractable for later multi-language. |
| **Data protection (DSGVO)** | RT covers **only consent** (dossier submission + cookie banner). **Data storage, deletion ("right to be forgotten") and access requests live in the CRM.** |
| **Accessibility** | **Best-effort** (contrast, keyboard) — **no** formal WCAG audit/certification target. |
| **Devices / browsers** | **Modern browsers** (current Chrome/Safari/Firefox/Edge); **desktop + mobile-web** with a **dedicated mobile view** (`mobile.html`), not just a responsive squeeze. No legacy/IE fallbacks. |

## 10. Glossary
| Term | Meaning |
|---|---|
| **RT** | Rocken Talent — the candidate-facing web app (this project). |
| **CRM** | Rocken's own system; source of truth for all core data + auth. |
| **Consultant** | Rocken recruiter who owns processes and pushes proposals. |
| **Proposal / Vorschlag** | A vacancy a consultant hand-picks for a candidate (channel 1). |
| **Recommender** | The matching engine that algorithmically suggests vacancies (channel 3). |
| **Dossier** | The candidate's application package sent to a company after consent. |
| **Vakanz** | A job vacancy (from the CRM). |
| **`why` / `note`** | Engine-generated match rationale / consultant-written quote. |
| **"Warten auf dich"** | Home section: open tasks + consultant proposals. |

## 11. Open questions — ❓ TBD (confirm with CRM/backend team)
- **Account-status taxonomy** — the CRM likely has more than the 4 prototype statuses; who controls each?
- **Profile-completeness formula** — which fields, what weighting, computed in CRM or frontend? (Gates matching at ≥ 70 %.)
- **CV source** — RT has no document handling in MVP, yet skills are parsed from the CV → the CV lives/enters in the **CRM**; confirm.
- **Sort order within lists** — proposal order; matches by score desc?

## 12. New build items — 🔧 (required, not yet in prototype)
- **In-app notification center** (bell + read/unread list).
- **Account-activation notification** (email + in-app).
- **Onboarding layover** (Figma exists, not built).

## 13. FUTURE — ⚠ out of MVP
- **Karriere-Hub** (entire page): market-value / Career-Radar, article feed.
- **Zeugnis-Decoder / Generator.**
- **Interview preparation** (incl. the "Vorbereiten" / prep-guide buttons in Termine).
- **Rocken networking events** (e.g. Career Night) in the Termine list — MVP shows **interviews only**.

## 14. Page index
| # | Screen | Spec |
|---|---|---|
| 1 | Home / Dashboard | [`handover-dashboard.md`](handover-dashboard.md) |
| 2 | Jobs & Suche | [`handover-jobs.md`](handover-jobs.md) |
| 3 | Bewerbungen | *(to come)* |
| 4 | Profil | *(to come)* |
| 5 | Vakanz-Detail | *(to come)* |
| 6 | Termine | *(to come)* |
| 7 | Login / Onboarding | *(to come)* |

---
*Live prototype: https://dennistodesco-star.github.io/handover_proto_Talent/ · Source: `index.html` (`index-redesign.html`)*
