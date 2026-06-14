# Phased Roadmap + Scaling

## Phase 1 — MVP (10 weeks, ~10 designers)

**Plugin:** SSO, brand/locale pickers. **Copy:** layer selection, variant UI, apply text. **Images:** fill selection, brief input, async generate, poll, preview, apply fill (1024×1024).

**Backend:** Gateway + Brand + Copy + Image (1 instance each). Brand Service: PDF ingest + CRUD. Copy Service: variants + 8 locales. Image Service: DALL-E jobs + S3. Usage logging (N4).

**Admin:** Simple web UI for Creative Ops to manage brands (late MVP, weeks 8–10).

**Infra:** Single region. S3 signed URLs.

**Success:** Copy localization same-day vs 3 days; designers generate on-brand image placeholders without leaving Figma.

**Weeks (outline):**
- **1–2:** Gateway, auth, Brand Service skeleton, plugin shell
- **3–4:** Brand PDF ingest; copy generation + translation
- **5–6:** Copy apply-to-canvas; Image Service + async jobs + plugin poll UI
- **7–8:** Image preview/apply; reference-image support; error handling
- **9–10:** Admin UI, pilot with designers, hardening

---

## Phase 2 — Beta (+2–3 months)

**Plugin:** UX polish — batch frame layers, locale memory per file, regenerate without redoing copy.

**Backend:** Ops usage/cost dashboard; glossary editing in admin; Image Service scaling if queue grows.

**Infra:** Scale Image Service to 2 instances if needed.

**Success:** Stable daily use across full studio; Creative Ops reporting on cost per user/asset.

---

## Phase 3 — Multi-client (~100 users, 12 months)

**Plugin:** Org switcher; plugin version checks.

**Backend:** Per-client onboarding; rate limits per `client_id`.

**Infra:** Load balancer → Gateway; Postgres read replica for reporting; Image Service auto-scale on queue depth.

**Global (if needed):** Stack per region (EU, US) with co-located S3; Gateway routes by `client_id`.

---

## Scaling checklist (concise)

```
MVP          Growth         Multi-client       Global
─────────────────────────────────────────────────────────
4×1 instance → LB           → client isolation → regional stacks
single S3    → scale Image  → read replica     + S3 per region
```

---

## Top risks

| # | Risk | Impact | Mitigation |
|---|------|--------|------------|
| **1** | **10-week MVP scope too large for 2 engineers** — plugin, admin UI, 4 services, copy + 8 locales, image async flow, brand PDF ingest, SSO, metering | High | Split ownership: Eng A → plugin + Gateway + Brand; Eng B → Copy + Image. Monorepo + managed Postgres/S3/Auth0 (no custom infra). Admin UI weeks 8–10 only. **Defer to Beta:** batch layers, locale memory, ops dashboard. Weekly scope review; cut nice-to-haves before core flows slip. |
| 2 | DALL-E too slow for sync (10–20s) | Medium | Async jobs + poll UI; copy completes first; N1 scoped to text ops |
| 3 | 2s copy latency missed | Medium | Parallel GPT calls; short prompts |
| 4 | 4 services = ops overhead | Low | Monorepo, shared CI; Brand Service is small CRUD |
| 5 | OpenAI outage / rate limits | Medium | Retry + queue failed jobs in DB |
| 6 | Multi-tenant data leak | High | `client_id` on all rows from day 1 |

---

## Deploy (N5)

`git push` → CI → staging → prod. Rollback = previous image tag. Target <12h with managed containers.
