# Data Flow: "Localise Copy & Replace Images"

## Sequence

```mermaid
sequenceDiagram
  actor D as Designer
  participant P as Figma Plugin
  participant GW as Gateway
  participant BRAND as Brand Service
  participant COPY as Copy Service
  participant IMG as Image Service
  participant DB as PostgreSQL
  participant OAI as OpenAI
  participant S3 as S3

  D->>P: Select layers, pick brand + locale

  P->>GW: GET /brands (on open)
  GW->>BRAND: list brands
  BRAND-->>P: brand picker options

  P->>GW: POST /copy/generate (JWT)
  GW->>COPY: forward
  COPY->>BRAND: GET /brands/{id}
  BRAND-->>COPY: tone + glossary
  COPY->>OAI: GPT-4o per layer (parallel)
  COPY-->>P: copy variants (~1-2s)

  P->>D: Pick variants in UI

  P->>GW: POST /images/jobs
  GW->>IMG: forward
  IMG->>BRAND: GET /brands/{id}
  BRAND-->>IMG: visual_guidelines
  IMG->>DB: create job (pending)
  IMG-->>P: job_id

  IMG->>OAI: DALL-E (async)
  IMG->>S3: store PNG

  loop poll
    P->>GW: GET /images/jobs/{id}
    IMG-->>P: status + signed URL when done
  end

  D->>P: Apply to canvas
  P->>P: set text + image fills
```

## Step-by-step flow

| # | Plugin | Backend |
|---|--------|---------|
| 1 | Read selected text layers + image fills | — |
| 2 | User picks brand, locale | — |
| 3 | `POST /copy/generate` | Copy Service → Brand Service for tone/glossary → GPT-4o |
| 4 | Show variant picker | — |
| 5 | User selects winners | — |
| 6 | `POST /images/jobs` with briefs + optional reference image | Image Service → Brand Service for visual rules → DALL-E |
| 7 | Poll + show thumbnails | Image Service → DALL-E → S3 |
| 8 | Apply text + fills to Figma nodes | — |

**Latency (N1):** Copy path sync (~2s). Image path async — never block copy on DALL-E.

## Example payloads

**Copy request:**
```json
{
  "client_id": "designtechco",
  "brand_id": "brand_x",
  "locale": "de-DE",
  "layers": [{ "id": "123:456", "text": "Run your world", "role": "headline" }]
}
```

**Copy response:**
```json
{
  "123:456": ["Erobere deine Welt", "Deine Welt. Dein Tempo."]
}
```

**Image job response:**
```json
{ "job_id": "job_abc", "status": "pending" }
```

**Poll (done):**
```json
{
  "status": "done",
  "images": { "123:789": "https://cdn.../job_abc.png" }
}
```
