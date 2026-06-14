# Technology Choices & Trade-offs

## Stack

| Piece | Choice |
|-------|--------|
| Plugin | TypeScript, Figma Plugin API, React UI |
| Admin UI | Simple web app (React) — brand CRUD for Creative Ops |
| Backend | **4 services:** Gateway, Brand, Copy, Image (monorepo) |
| Database | PostgreSQL — Brand Service owns `brands`; Copy/Image own jobs + usage |
| Files | S3 — PDFs (Brand Service), PNGs (Image Service) |
| AI | GPT-4o (required: copy, translation, vision); DALL-E 3 (F4 image placeholders — OpenAI API, not named in brief) |
| Auth | Auth0 at Gateway |
| Hosting | Managed containers — 1 instance each at MVP |

---

## Trade-off 1: Four microservices vs fewer

| | **4 services (chosen)** | 3 services (Brand inside Copy) |
|--|-------------------------|--------------------------------|
| Pros | Brand identity is shared by Copy + Image without duplication; clear CRUD boundary; Creative Ops changes don't touch AI services | One fewer deploy |
| Cons | One more service | Copy Service owns brand data Image also needs — coupling |
| **Pick** | **Gateway + Brand + Copy + Image** — Brand is small, mostly CRUD + PDF ingest |


---

## Trade-off 2: Brand Service API vs shared DB reads

| | **Brand Service API (chosen)** | Copy/Image query `brands` table directly |
|--|--------------------------------|--------------------------------------------|
| **Pick** | Copy/Image call `GET /internal/brands/{id}` — Brand Service owns all writes; enforces `client_id` in one place |

---

## Trade-off 3: Sync copy vs async images

| | **Split paths (chosen)** | All sync |
|--|--------------------------|----------|
| **Pick** | Copy ~2s sync. Image returns `job_id`; plugin polls. |

---

## Scaling (when to add what)

| Trigger | Add |
|---------|-----|
| Image queue backs up | Scale Image Service only |
| Designers in multiple geographies | Regional deployment + **S3 bucket in same region** as services |
| 100+ users | LB → Gateway |
| Multiple clients | Brand Service admin for onboarding |

Brand Service stays 1 instance — CRUD is low volume.

---

## Security (N2)

- JWT at Gateway; admin routes require `ops` role
- Brand Service is only writer for brand data
- OpenAI keys in Copy + Image only
- Usage logged per user (N4)
