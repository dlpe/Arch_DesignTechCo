# Three Goals → Figma Plugin + Backend

Maps each Superside objective to **what the plugin must do** and **which service handles it**.

---

## Brand identity → **Brand Service** (CRUD)

**Pain:** 25 brands, each with different tone, glossary, and visual rules. Creative Ops must manage these without developer involvement.

**Not in the Figma plugin** — managed via a simple **admin web UI** (Creative Ops). Plugin only **reads** the brand list for the picker.

### Admin UI (build on Brand Service)

| Feature | API |
|---------|-----|
| List / create / edit brands | `GET/POST/PUT /brands` |
| Upload guideline PDF (F3) | `POST /brands/{id}/pdf` → extract text → store |
| Edit tone of voice | `PUT /brands/{id}` → `tone_of_voice` field |
| Edit visual identity | `PUT /brands/{id}` → `visual_guidelines` field |
| Manage glossary (locked terms) | `PUT /brands/{id}/glossary` |

### What Brand Service stores (per brand)

| Field | Used by |
|-------|---------|
| `tone_of_voice` | Copy Service |
| `visual_guidelines` | Image Service |
| `glossary` (forbidden/preferred terms) | Copy Service |
| `source_pdf_url` | Archive in S3 |

### Who calls Brand Service

| Caller | Call | Purpose |
|--------|------|---------|
| **Plugin** | `GET /brands` | Populate brand picker |
| **Copy Service** | `GET /brands/{id}` (internal) | Fetch tone + glossary before GPT call |
| **Image Service** | `GET /brands/{id}` (internal) | Fetch visual guidelines before DALL-E |
| **Admin UI** | CRUD endpoints | Creative Ops manages identity |

Brand Service **owns all writes** to brand data. Copy and Image are read-only consumers.

---

## 1. Replace or localise copy (on-brand tone)

**Pain:** 3-day manual localization; inconsistent voice across 25 brands.

### Figma plugin

| Feature | Purpose |
|---------|---------|
| **Select text layers** | Read selected nodes; extract `characters`, font, layer name |
| **Brand picker** | Choose which brand's guidelines apply (`brand_id`) |
| **Locale picker** | Target language (8 locales) |
| **Generate button** | POST selected layers to Copy Service |
| **Variant picker UI** | Show 2–3 options per layer; designer picks winner |
| **Apply to canvas** | `loadFontAsync` → set `node.characters`; preserve text styles |
| **Layer role tag** (optional) | Headline / CTA / body — improves prompts; infer from name or dropdown |

### Backend → **Copy Service**

- Fetches tone + glossary from **Brand Service** (`GET /brands/{id}`)
- Call GPT-4o: variants + translate in one step per layer
- Return JSON keyed by layer ID
- Log usage + estimated cost per user (N4)

**MVP:** Full plugin flow above.

---

## 2. On-brand image placeholders

**Pain:** Designers leave Figma for Midjourney/stock; breaks focus and version control.

### Figma plugin

| Feature | Purpose |
|---------|---------|
| **Select image fill layers** | Detect layers with image fills or empty placeholders |
| **Export current fill** (optional) | If layer has an image, send thumbnail to API — keeps layout/subject when replacing |
| **Brief input** | Short description per layer — supplements or replaces empty fills |
| **Generate images** | POST to Image Service; receive `job_id` immediately |
| **Progress UI** | Poll job status; spinner per layer — see **Risk: DALL-E latency** below |
| **Thumbnail preview** | Show result in plugin before applying |
| **Apply as fill** | Fetch PNG → `figma.createImage()` → set `fills` on layer (1024×1024) |
| **Regenerate** | Retry one layer without redoing copy |

### Backend → **Image Service**

On-brand images need **two inputs** (plus optional reference):

| Input | Source | Role |
|-------|--------|------|
| **Brand visual identity** | **Brand Service** — `visual_guidelines` from PDF/manual edit | Keeps output on-brand |
| **User brief** | Plugin text field | Subject/scene ("urban runner at dawn") |
| **Current fill image** (optional) | Plugin exports layer's existing image → Image Service | Preserve composition when iterating |

**How it works (not img2img):** DALL-E 3 is text-to-image. If a reference image exists, Image Service sends it to **GPT-4o vision** first: *"Describe this scene + apply Brand X visual rules"* → refined text prompt → DALL-E. Without a reference, prompt = brand visual rules + brief only.

> **Tone of voice** is mainly for copy. For images, use **visual identity** from the same PDF (palette, photography style, forbidden visuals).

- Async job: DALL-E 3 → store PNG in S3 → return signed URL
- Plugin polls `GET /jobs/{id}`

### Risk: DALL-E latency — sync or async?

| Question | Answer |
|----------|--------|
| **How long does DALL-E 3 take?** | Typically **10–20 seconds** per image. With GPT-4o vision (reference image → prompt) add **~2–5s** more. |
| **Can it be synchronous?** | **No** — fails **N1** (≤2s), blocks the Figma plugin UI, and risks iframe/network timeouts. |
| **Does N1 apply to images?** | N1 (≤2s) targets single-sample **copy** operations (F1/F2). F4 image generation is async by design — DALL-E latency is incompatible with a synchronous path. |
| **UX mitigation** | Return `job_id` immediately; poll every 2s; per-layer spinners; designer reviews copy while images generate; partial results as each layer completes. |
| **Alternative if instant images are required** | Pre-cached brand placeholders (not generative), or a faster lower-fidelity model — product scope decision. |

**Model note:** Assessment constraint specifies GPT-4o for foundation models. F4 requires 1024×1024 raster output; **DALL-E 3** (OpenAI image API) is used for placeholders. GPT-4o handles copy, translation, and optional vision on reference images.

**MVP:** Full image flow — brief, async generate, poll, preview, apply fill (F4).

---

## 3. Internal tool now → multi-client, ~100 users later

**Pain:** Must pilot with 10 designers; architecture should not block licensing to other companies.

### Figma plugin

| Feature | MVP (10 users) | Later (~100 users, multi-client) |
|---------|----------------|----------------------------------|
| **SSO login** | OAuth iframe → JWT stored in plugin | Same; supports client IdPs |
| **Session refresh** | Silent token refresh before API calls | Same |
| **Org / client context** | Hardcoded or env config | Org switcher in plugin settings |
| **Error states** | Auth expired, API down, rate limit — clear messages | Same + status link |
| **Plugin version header** | Send `X-Plugin-Version` on every request | Backend can deprecate old versions |
| **No secrets in plugin** | All calls via Gateway; no OpenAI keys | Same |

### Backend → **Gateway** + shared data model

| Concern | MVP | Scale path |
|---------|-----|------------|
| Auth | Auth0 JWT validation at Gateway | Per-client IdP config in admin |
| Tenancy | `client_id` on all DB rows (single client in prod) | Row isolation; admin UI to onboard clients |
| Metering | Log `user_id`, `client_id`, tokens per request | Ops dashboard (Beta) |
| Deploy | 4 small services, 1 instance each | See [04-roadmap.md](./04-roadmap.md) scaling ladder |

**Design principle:** The plugin is a thin client. Business logic and brand rules live on the server. Plugin version + JWT support multi-tenant routing without a client rewrite.

---

## Plugin UI layout (one screen)

```
┌─────────────────────────────────┐
│  AI Creative Studio    [Login]  │
├─────────────────────────────────┤
│  Brand: [Brand X ▼]             │
│  Locale: [de-DE ▼]              │
├─────────────────────────────────┤
│  TEXT (3 layers selected)       │
│  [Generate copy]                │
│  ▸ Headline: "Erobere..."  [✓]  │
│  ▸ CTA: "Jetzt shoppen"    [ ]  │
├─────────────────────────────────┤
│  IMAGES (1 layer selected)      │
│  Brief: [urban runner...]       │
│  [Generate placeholder] ⏳      │
│  [thumbnail preview]            │
├─────────────────────────────────┤
│  [Apply selected to canvas]     │
└─────────────────────────────────┘
```
