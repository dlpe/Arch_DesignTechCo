# MVP Epics & Sprints

10-week MVP plan for **2 engineers**, **2-week sprints**. Import epics into Jira (Epic → Story) or Trello (Sprint lists → cards).

**Ownership:** Eng A — Figma plugin, Gateway, Brand Service, Admin UI · Eng B — Copy Service, Image Service, shared infra/CI

---

## Sprint overview

| Sprint | Weeks | Goal | Demo |
|--------|-------|------|------|
| **S1** | 1–2 | Platform, Gateway, auth, plugin setup | Login from Figma; services on staging |
| **S2** | 3–4 | Brand ingest + copy generation | On-brand localized copy for one layer |
| **S3** | 5–6 | Copy E2E + image jobs started | Apply copy to canvas; image job created |
| **S4** | 7–8 | Image E2E + hardening | Full flow: copy + image placeholder applied |
| **S5** | 9–10 | Admin UI + prod readiness | Creative Ops manages brands; MVP live |

---

## Epic map (MVP)

| Epic | Scope | Sprints |
|------|-------|---------|
| **E1** Platform, Gateway & Auth | Monorepo, CI, deploy, Auth0, API Gateway | S1, S4–S5 |
| **E2** Brand Service & Admin UI | Brand CRUD, PDF ingest, admin web app | S1–S2, S5 |
| **E3** Figma Plugin | Designer UI: auth, copy, images, apply to canvas | S1–S4 |
| **E4** Copy Service | GPT-4o variants + 8 locales | S2–S3 |
| **E5** Image Service | DALL-E async jobs, S3 | S3–S4 |
| **E6** Usage & metering | Usage events, cost per user (N4) | S3–S5 |

---

## S1 — Foundation (weeks 1–2)

### E1 Platform, Gateway & Auth
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E1-1 | Monorepo setup (4 services + shared types) | Both | 3 |
| E1-2 | CI pipeline: lint, test, build 4 Docker images | Both | 5 |
| E1-3 | Dev/staging environments (managed containers + Postgres + S3) | A | 5 |
| E1-4 | Auth0 tenant + SSO app configuration | A | 2 |
| E1-5 | Gateway: JWT validation + route to Brand / Copy / Image | A | 5 |
| E1-6 | Internal service auth (service token or private network) | A | 3 |

### E2 Brand Service & Admin UI (API skeleton)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E2-1 | Postgres schema v1 (`clients`, `brands`, `jobs`, `usage_events`) | B | 3 |
| E2-2 | `GET /brands` list endpoint (seed data) | A | 3 |
| E2-3 | `GET /brands/{id}` — tone, visual, glossary schema | A | 3 |
| E2-4 | S3 bucket + signed URL helper | A | 2 |

### E3 Figma Plugin (setup & auth)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E3-1 | Plugin project setup (TypeScript + React UI in Figma) | A | 3 |
| E3-2 | OAuth login flow in plugin panel | A | 5 |
| E3-3 | API client with JWT + error handling | A | 3 |
| E3-4 | Brand picker (loads `GET /brands`) | A | 2 |

**S1 exit criteria:** Designer logs in via SSO; brand list loads in plugin; all services deploy to staging.

---

## S2 — Brand ingest + copy generation (weeks 3–4)

### E2 Brand Service & Admin UI (ingest)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E2-5 | `POST /brands/{id}/pdf` — upload to S3, extract text | A | 5 |
| E2-6 | Parse PDF into `tone_of_voice` + `visual_guidelines` | A | 5 |
| E2-7 | `PUT /brands/{id}/glossary` — locked terms | A | 3 |

### E3 Figma Plugin (copy prep)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E3-5 | Read selected text layers (`characters`, font, id) | A | 5 |
| E3-6 | Locale picker (8 locales) | A | 2 |
| E3-7 | Layer role tag (headline / CTA / body) — optional | A | 2 |

### E4 Copy Service
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E4-1 | Fetch brand from Brand Service; build GPT-4o prompt | B | 5 |
| E4-2 | `POST /copy/generate` — 2–3 variants per layer | B | 5 |
| E4-3 | Translate to target locale (8 locales) in same call | B | 5 |
| E4-4 | Parallel requests for multi-layer (latency target N1) | B | 5 |

**S2 exit criteria:** POST copy for one layer returns localized on-brand variants using ingested PDF.

---

## S3 — Copy E2E + image jobs (weeks 5–6)

### E4 Copy Service (complete)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E4-5 | Error handling: timeout, missing brand, OpenAI retry | B | 3 |
| E4-6 | Unit + integration tests for prompt assembly | B | 3 |

### E3 Figma Plugin (copy flow)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E3-8 | Variant picker UI (per layer) | A | 5 |
| E3-9 | Apply selected variant to canvas (`loadFontAsync`, `characters`) | A | 5 |
| E3-10 | Copy generate button wired to Copy Service | A | 3 |

### E5 Image Service (start)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E5-1 | `jobs` table + status machine (pending → done / failed) | B | 3 |
| E5-2 | `POST /images/jobs` — enqueue, return `job_id` | B | 5 |
| E5-3 | Background worker: DALL-E 3 → S3 → update job | B | 8 |
| E5-4 | Fetch visual guidelines from Brand Service; build prompt | B | 5 |
| E5-5 | `GET /images/jobs/{id}` — poll status + signed URL | B | 3 |

### E6 Usage & metering (start)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E6-1 | Log usage event on copy request (user, tokens, cost) | B | 3 |
| E6-2 | Log usage event on image job complete | B | 2 |

**S3 exit criteria:** Designer localizes copy and applies to Figma; image job creates and returns pending status.

---

## S4 — Image E2E + hardening (weeks 7–8)

### E5 Image Service (complete)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E5-6 | GPT-4o vision: reference image → refined DALL-E prompt | B | 5 |
| E5-7 | Failed job handling + retry | B | 3 |
| E5-8 | 1024×1024 output validation | B | 2 |

### E3 Figma Plugin (image flow)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E3-11 | Select image fill layers + brief input | A | 3 |
| E3-12 | Export current fill as reference thumbnail | A | 5 |
| E3-13 | Poll UI — spinner per layer | A | 3 |
| E3-14 | Thumbnail preview in plugin | A | 3 |
| E3-15 | Apply PNG to fill (`createImage`, set fills) | A | 5 |
| E3-16 | Regenerate single layer without redoing copy | A | 3 |

### E1 / E3 Hardening
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E1-7 | Rate limiting at Gateway | A | 3 |
| E3-17 | Plugin error states (auth expired, API down) | A | 3 |
| E6-3 | Basic usage query for ops (SQL or API stub) | B | 3 |

**S4 exit criteria:** Full flow — localise copy + generate on-brand image placeholder + apply both to canvas.

---

## S5 — Admin UI + prod readiness (weeks 9–10)

### E2 Brand Service & Admin UI (complete)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E2-8 | Admin web app setup + SSO (ops role) | A | 3 |
| E2-9 | Brand list + create / edit UI | A | 5 |
| E2-10 | PDF upload UI → Brand Service | A | 3 |
| E2-11 | Edit tone of voice + visual guidelines UI | A | 3 |
| E2-12 | Glossary editor (forbidden / preferred terms) | A | 5 |
| E2-13 | Seed 3–5 brands from real guideline PDFs | Both | 3 |

### E1 Platform, Gateway & Auth (prod)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E1-8 | Production deploy + rollback tested (<12h, N5) | A | 3 |
| E1-9 | Deploy runbook | A | 2 |

### E3 Figma Plugin (release)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E3-18 | Publish plugin to DesignTechCo (private org) | A | 2 |
| E3-19 | Bug fixes from designer feedback | Both | 8 |

### E6 Usage & metering (complete)
| ID | Story | Owner | Points |
|----|-------|-------|--------|
| E6-4 | Usage report: cost per user for Creative Ops | B | 5 |

**S5 exit criteria:** MVP live for 10-person studio; Creative Ops manages brands without engineering.

---

## Sprint board layout (Trello)

```
Backlog | S1 | S2 | S3 | S4 | S5 | Done
```

Label cards: `Eng-A` (blue), `Eng-B` (green), `Both` (purple).

---

## Jira import tips

| Jira field | Value |
|------------|-------|
| **Epic Name** | E1 Platform Gateway Auth, E2 Brand Admin, E3 Figma Plugin, … |
| **Story ID** | E4-2, E3-15, … (prefix in summary) |
| **Components** | Platform, Brand, Admin, Plugin, Copy, Image |
| **Fix Version** | MVP-S1 … MVP-S5 |

---

## Phase 2 backlog (post-MVP)

| Epic | Stories (summary) |
|------|-------------------|
| **P2-E1 Plugin UX** | Batch all text layers in frame; locale memory per file |
| **P2-E2 Ops dashboard** | Usage & cost charts by user / brand / asset |
| **P2-E3 Scale** | Image Service 2nd instance; LB on Gateway |
| **P2-E4 Governance** | Guideline versioning; approval before brand goes live |
| **P2-E5 Multi-client prep** | Org switcher in plugin; client onboarding in admin |

---

## Requirements traceability

| Req | Epics |
|-----|-------|
| F1 Copy variants | E3, E4 |
| F2 8 locales | E3, E4 |
| F3 Brand PDFs | E2 |
| F4 Image placeholders | E3, E5 |
| N1 ≤2s copy latency | E4 |
| N2 SSO | E1, E3 |
| N3 Scale path | P2-E3 |
| N4 Metering | E6 |
| N5 Deploy <12h | E1 |

---

## Capacity note

~**120–140 story points** across 5 sprints. If S3–S4 slip, cut **E3-7 (layer role tag)** and **E5-6 (reference image vision)** first — defer to Phase 2.
