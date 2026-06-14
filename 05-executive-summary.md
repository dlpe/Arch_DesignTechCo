# Executive Summary

## Problem

~10 designers, 25 brands, Figma-first workflow. Copy localization: **3 days**. Images: off-platform. Inconsistent brand voice.

## Solution

**Two client applications** and **four backend services**:

### Clients

1. **Figma plugin** — designers generate and apply localized copy and image placeholders inside Figma
2. **Admin UI** — Creative Ops manages brand identity (CRUD, PDF upload, glossary)

### Backend

1. **Gateway** — auth, routing
2. **Brand Service** — brand identity storage, PDF ingest (F3); serves Admin UI and AI services
3. **Copy Service** — on-brand variants + 8 locales (reads from Brand Service)
4. **Image Service** — async placeholders (reads visual guidelines from Brand Service)

| Goal | Clients | Backend |
|------|---------|---------|
| Localise on-brand copy | Figma plugin: layer select, variant UI | Copy Service ← Brand Service |
| Image placeholders | Figma plugin: fill select, preview | Image Service ← Brand Service |
| Brand governance (F3) | Admin UI: CRUD, PDF upload | Brand Service |
| Internal → multi-client | Both clients: SSO; plugin brand picker | Brand Service owns `client_id` on all brand data |

## Scale path

| Stage | Add |
|-------|-----|
| ~10 users | 4 services × 1 instance |
| ~100 users | Scale Image Service; load balancer on Gateway |
| Multi-client | Brand admin per client |

## MVP (10 weeks, 2 engineers)

Copy + translate + brand PDF ingest + **image placeholders** + SSO. All four backend services in monorepo. Admin UI for brand CRUD in weeks 8–10.

**Top risk:** Scope is aggressive for two engineers — mitigated by parallel ownership, managed infra, and deferring Beta-only features (batch layers, ops dashboard).
